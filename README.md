# AWS Lambda Adapter

A tool to run web applications on AWS Lambda

AWS Lambda Adapter allows you to build web app (http api) with familiar frameworks (e.g. Express.js, Flask, SpringBoot, and Laravel, anything speaks HTTP 1.1/1.0) and run it on AWS Lambda.
The exact same docker image can run on AWS Lambda, Amazon EC2, AWS Fargate, and local computers.

AWS Lambda adapter is developed as a Lambda extension (since v0.2.0). For details, checkout its [design](docs/design.md) and [development](docs/development.md) documents.

![Lambda Adapter](docs/images/lambda-adapter-overview.png)

## Usage

AWS Lambda Adapter work with Lambda functions packaged as both docker images and Zip packages.

### Lambda functions packaged as Docker Images or OCI Images

To use Lambda Adapter with docker images, package your web app (http api) in a Dockerfile, and add one line to copy Lambda Adapter binary to /opt/extensions inside your container.
By default, Lambda Adapter assumes the web app is listening on port 8080. If not, you can specify the port via [configuration](#Configurations).

```dockerfile
COPY --from=public.ecr.aws/awsguru/aws-lambda-adapter:0.3.2 /lambda-adapter /opt/extensions/lambda-adapter
```

Pre-compiled Lambda Adapter binaries are provided in ECR public repo: [public.ecr.aws/awsguru/aws-lambda-adapter](https://gallery.ecr.aws/awsguru/aws-lambda-adapter).
Multi-arch images are also provided in this repo. It works on both x86_64 and arm64 CPU architecture.

Below is a Dockerfile for [an example nodejs application](examples/expressjs).

```dockerfile
FROM public.ecr.aws/docker/library/node:16.13.2-stretch-slim
COPY --from=public.ecr.aws/awsguru/aws-lambda-adapter:0.3.2 /lambda-adapter /opt/extensions/lambda-adapter
EXPOSE 8080
WORKDIR "/var/task"
ADD src/package.json /var/task/package.json
ADD src/package-lock.json /var/task/package-lock.json
RUN npm install --production
ADD src/ /var/task
CMD ["node", "index.js"]
```

This works with any base images except AWS managed base images. To use AWS managed base images, you need to override the ENTRYPOINT to start your web app.

### Lambda functions packaged as Zip package for AWS managed runtimes

AWS Lambda Adapter also works with AWS managed Lambda runtimes. You need to do three things:

1. attach Lambda Adapter layer to your function. 
   1. x86_64: `arn:aws:lambda:${AWS::Region}:753240598075:layer:LambdaAdapterLayerX86:2`
   2. arm64: `arn:aws:lambda:${AWS::Region}:753240598075:layer:LambdaAdapterLayerArm64:2`
2. configure Lambda environment variable `AWS_LAMBDA_EXEC_WRAPPER` to `/opt/bootstrap`. 
3. set function handler to your web application start up script. e.g. `run.sh`.

For details, please check out [the example nodejs application](examples/expressjs-zip).


### Configurations

The readiness check port/path and traffic port can be configured using environment variables. These environment variables can be defined either within docker file or as Lambda function configuration.

|Environment Variable| Description                                   | Default      |
|--------------------|-----------------------------------------------|--------------|
|READINESS_CHECK_PORT| readiness check port                          | 8080         |
|READINESS_CHECK_PATH| readiness check path                          | /            |
|PORT                | traffic port                                  | 8080         |
|REMOVE_BASE_PATH    | the base path to be removed from request path | empty string |

**REMOVE_BASE_PATH** - The value of this environment variable tells the adapter whether the application is running under a base path.
For example, you could have configured your API Gateway to have a /orders/{proxy+} and a /catalog/{proxy+} resource.
Each resource is handled by a separate Lambda functions. For this reason, the application inside Lambda may not be aware of the fact that the /orders path exists.
Use REMOVE_BASE_PATH to remove the /orders prefix when routing requests to the application. Defaults to empty string. Checkout [SpringBoot](examples/springboot) example.

## Examples

- [Flask](examples/flask)
- [Express.js](examples/expressjs)
- [Express.js in Zip](examples/expressjs-zip)
- [SpringBoot](examples/springboot)
- [nginx](examples/nginx)
- [php](examples/php)

## Acknowledgement

This project was inspired by several community projects.

- [re:Web](https://github.com/apparentorder/reweb)
- [Serverlessish](https://github.com/glassechidna/serverlessish)

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This project is licensed under the Apache-2.0 License.
