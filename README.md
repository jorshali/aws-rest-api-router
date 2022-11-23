
# aws-rest-api-router

A simple Typescript-based REST API router for use with AWS Lambda functions.

## Introduction

A simple way to route your REST API requests and eliminate boilerplate without the need for extensive libraries or configuration.  The implementation has a very small footprint.

Provides the following:

- HTTP Method / Path routing
- Access to path parameters
- Body parsing to JSON
- Error handling
- Eliminates boilerplate

## Installation

* `npm i aws-rest-api-router`

## Basic usage

This module provides a simple routing mechanism for defining REST APIs within AWS Lambda functions.  Codes speaks better than words, so the following Lambda function definition demonstrates how the `AwsFunctionRouter` can be used to define a group of REST API routes:

```
import { APIGatewayProxyEvent, APIGatewayProxyResult, Context } from 'aws-lambda';

import { AwsFunctionRouter, AwsRequestContext } from 'aws-rest-api-router';

import { PostRepository } from './PostRepository';

const postRepository = new PostRepository();

// define the router with a basePath which should match the URI for your Lambda's API Gateway
const router = new AwsFunctionRouter<Post>({
    basePath: '/post-service',
    includeCORS: true
  })
  .get('', async (route) => {
    // GET http://my-aws-url/post-service
    return route.okResponse(await postRepository.findAll());
  })
  .get('/:id', async (route, requestContext) => {
    // GET http://my-aws-url/post-service/1
    const id = route.getPathParams(requestContext).id;
    const post = await postRepository.findPostById(id);

    if (!post) {
      return route.errorResponse(StatusCodes.NOT_FOUND);
    }

    return route.okResponse(post);
  })
  .post('', async (route, requestContext) => {
    // POST http://my-aws-url/post-service
    // { "title": "My first post", "content": "This is my first post" }
    const post = await postRepository.save(route.parseBody(requestContext));

    return route.okResponse(post);
  })
  .update('/:id', async (route, requestContext) => {
    // PUT http://my-aws-url/post-service/1
    // { "id": "1", "title": "My first post", "content": "This is my first post" }
    const id = route.getPathParams(requestContext).id;
    const post = await postRepository.findPostById(id);

    if (!post) {
      return route.errorResponse(StatusCodes.NOT_FOUND);
    }

    const postUpdates = route.parseBody(requestContext);

    post.title = postUpdates.title;
    post.content = postUpdates.content;

    postRepository.update(post);

    return route.okResponse(post);
  });

export const lambdaHandler = async (event: APIGatewayProxyEvent, context: Context): Promise<APIGatewayProxyResult> => {
  return await router.handle(new AwsRequestContext(event, context));
};
```

Note that getting path params relies on using an [AWS proxy resource](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-set-up-simple-proxy.html) within your API Gateway definition.

You can find a working example in [aws-sam-rest-api-starter](https://github.com/jorshali/aws-sam-rest-api-starter).