
![normal](/static/images/lambda/lambda_flow.png)

## How to deploy Docker Image in ECR?

In the previous post we have seen how to build a docker image and persist it in ECR. In the post, I will explore how to deploy that image.

There are many ways to deploy the docker image in cloud.

![normal](/static/images/lambda/deploy.png)

Since there are different ways to deploy the image, I will explore deployment using [`Serverless - Lambda`](<(https://aws.amazon.com/lambda/?nc1=h_ls)>).

Some of the other serverless architecture deployment providers are:

- [Google Cloud Functions](https://cloud.google.com/functions/)

- [Azure Functions](https://azure.microsoft.com/en-us/services/functions/)

- [IMB Cloud Functions](https://www.ibm.com/cloud/functions)

In this post, I will be going through the following topics:

- `Basics of Serverless`

- `Basics of AWS Lambda`

- `Triggering Lambda with API Gateway`

- `Deploying Container using Lambda`

- `Automating deployment to Lambda using Github Actions`

## Basics of Serverless

Typical model deployment looks like:

- Need to setup and maintain the servers (ec2 instances) in which models are deployed

- Need to manage scaling: How many servers are required?

- Need to manage time to deploy models

- How can a single developer manage a complex service?

This is where the `serverless architecture` comes into picture.

### What is Serverless Architecture?

A serverless architecture is a way to build and run applications and services without having to manage infrastructure. The application still runs on servers, but all the server management is done by third party service (AWS). We no longer have to provision, scale, and maintain servers to run the applications, databases, and storage systems.

### Why use serverless architectures?

By using a serverless architecture, developers can focus on their core product instead of worrying about managing and operating servers or runtimes, either in the cloud or on-premises. This reduced overhead lets developers reclaim time and energy that can be spent on developing great products.

#### The advantages of using a serverless architecture are:

- It abstracts away the server details and lets you serve your code or model with few lines of code

- It will handle the provising of servers

- It will scale the machines up and down depending on usage

- Does the load balancing

- No cost when the code is not running

#### There are some downsides also to serverless architecture.

- `Response latency`: Since the code is not running readily and will only run once it is called, there will be latency till the code is up and running (loading a model in case of ML).

- `Not useful for long running processes`: Serverless providers charge for the amount of time code is running, it may cost more to run an application with long-running processes in a serverless infrastructure compared to a traditional one.

- `Difficult to debug`: Debugging is difficult since the developer cannot have the access(ssh) to the machine where the code is running.

- `Vendor limitations`: Setting up a serverless architecture with one vendor can make it difficult to switch vendors if necessary, especially since each vendor offers slightly different features and workflows.

## Basics of AWS Lambda

Let's create a basic function.

- Sign in to the AWS Management Console and open the Amazon Lambda console at [https://console.aws.amazon.com/lambda/home](https://console.aws.amazon.com/lambda/home)

- Click on `Create Function`

![normal](/static/images/lambda/lambda_1.png)

- Provide some name to the function and choose python as the runtime.

![normal](/static/images/lambda/lambda_2.png)

- Go to the `Test` section and click on `Test` button

![normal](/static/images/lambda/lambda_3.png)

- Test logs

![normal](/static/images/lambda/lambda_4.png)

_Note: The lambda handler function expects two variables: event, context_

## Triggering Lambda with API Gateway

Now that we have created a basic function, we need a way to call / trigger it. There are different ways to trigger the lambda. Let's see how to do it using `API Gateway`.

API Gateway handles all the tasks involved in accepting and processing up to hundreds of thousands of concurrent API calls, including traffic management, CORS support, authorization and access control, throttling, monitoring, and API version management.

Let's a create a simple API.

- Sign in to the AWS Management Console and open the Amazon API Gateway console at [https://us-west-2.console.aws.amazon.com/apigateway](https://us-west-2.console.aws.amazon.com/apigateway)

- Click on `Build` button under HTTP API

![normal](/static/images/lambda/api_1.png)

- Add `Integration` to the API. Select the Integration type as `Lambda` and select the lambda we have created

![normal](/static/images/lambda/api_2.png)

- Once the API is created, refersh the Lambda page. You will see `API Gateway` under the triggers of lambda

![normal](/static/images/lambda/api_3.png)

- Go to configuration section and API Gateway details, there will be a `API endpoint`. Open that link in new tab

![normal](/static/images/lambda/api_4.png)

- You will see the response returned by lambda in the browser

![normal](/static/images/lambda/api_5.png)

## Deploying Container using Lambda

We have created a docker image and persisted it in `ECR` in the last post. Let's see how to use that image and run it using Lambda.

- Create a new lambda function with name `MLOps-Basics` and choose the type as `Container Image`

![normal](/static/images/lambda/lambda_5.png)

- Select the corresponding image and tag from `ECR` after clicking on `Browse Images` option

![normal](/static/images/lambda/lambda_6.png)

- Since the model size is more than 100MB and it will take some time to load, go the `Configuration` section and update the memory size and timeout

![normal](/static/images/lambda/lambda_7.png)

- Now let's configure the `Test` so that the lambda can be tested. Go to the `Test` section and configure the test event.

![normal](/static/images/lambda/lambda_8.png)

- Click on `Test` button, after a while logs can be visible as following:

![normal](/static/images/lambda/lambda_9.png)

Now that the lambda is setup, let's trigger it using API Gateway. Before doing that some code changes needs to be done so that the lambda knows what function to call when it is triggered.

Create a file called `lambda_handler.py` in the root directory. The contents of the file are as follows:

```python
import json
from inference_onnx import ColaONNXPredictor

inferencing_instance = ColaONNXPredictor("./models/model.onnx")

def lambda_handler(event, context):
	"""
	Lambda function handler for predicting linguistic acceptability of the given sentence
	"""

	if "resource" in event.keys():
		body = event["body"]
		body = json.loads(body)
		print(f"Got the input: {body['sentence']}")
		response = inferencing_instance.predict(body["sentence"])
		return {
			"statusCode": 200,
			"headers": {},
			"body": json.dumps(response)
		}
	else:
		return inferencing_instance.predict(event["sentence"])
```

Let's understand what's happening here:

- When lambda is triggered with API it recevies some information regarding the request. A sample looks like the following:

```json
{
  "resource": "/prediction",
  "path": "/prediction",
  "httpMethod": "POST",
  "headers": {
    "Accept": "*/*",
    "Accept-Encoding": "gzip, deflate, br",
    "Content-Type": "application/json",
    "Host": "ttaq1i1kvi.execute-api.us-west-2.amazonaws.com",
    "Postman-Token": "f5b1d731-7903-431e-a96f-b9353e7e9ec6",
    "User-Agent": "PostmanRuntime/7.28.2",
    "X-Amzn-Trace-Id": "Root=1-6106f00c-564e363f6389eaff72b07cfb",
    "X-Forwarded-For": "157.47.15.62",
    "X-Forwarded-Port": "443",
    "X-Forwarded-Proto": "https"
  },
  "multiValueHeaders": {
    "Accept": ["*/*"],
    "Accept-Encoding": ["gzip, deflate, br"],
    "Content-Type": ["application/json"],
    "Host": ["ttaq1i1kvi.execute-api.us-west-2.amazonaws.com"],
    "Postman-Token": ["f5b1d731-7903-431e-a96f-b9353e7e9ec6"],
    "User-Agent": ["PostmanRuntime/7.28.2"],
    "X-Amzn-Trace-Id": ["Root=1-6106f00c-564e363f6389eaff72b07cfb"],
    "X-Forwarded-For": ["157.47.15.62"],
    "X-Forwarded-Port": ["443"],
    "X-Forwarded-Proto": ["https"]
  },
  "queryStringParameters": None,
  "multiValueQueryStringParameters": None,
  "pathParameters": None,
  "stageVariables": None,
  "requestContext": {
    "resourceId": "nfb8ha",
    "resourcePath": "/prediction",
    "httpMethod": "POST",
    "extendedRequestId": "DZpx8FzXPHcFWAQ=",
    "requestTime": "01/Aug/2021:19:03:40 +0000",
    "path": "/deploy/prediction",
    "accountId": "246113150184",
    "protocol": "HTTP/1.1",
    "stage": "deploy",
    "domainPrefix": "ttaq1i1kvi",
    "requestTimeEpoch": 1627844620249,
    "requestId": "c4d0a77b-acd1-42c7-9304-8b1183c6a32f",
    "identity": {
      "cognitoIdentityPoolId": None,
      "accountId": None,
      "cognitoIdentityId": None,
      "caller": None,
      "sourceIp": "157.47.15.62",
      "principalOrgId": None,
      "accessKey": None,
      "cognitoAuthenticationType": None,
      "cognitoAuthenticationProvider": None,
      "userArn": None,
      "userAgent": "PostmanRuntime/7.28.2",
      "user": None
    },
    "domainName": "ttaq1i1kvi.execute-api.us-west-2.amazonaws.com",
    "apiId": "ttaq1i1kvi"
  },
  "body": "{\n    \"sentence\": \"this is a sample sentence\"\n}",
  "isBase64Encoded": False
}
```

- Since the input for the model is under `body`, we need to parse the event and get that information. That's what the `lambda_handler` function is doing.

Now let's modify the Dockerfile to include these changes.

```
FROM amazon/aws-lambda-python

ARG AWS_ACCESS_KEY_ID
ARG AWS_SECRET_ACCESS_KEY
ARG MODEL_DIR=./models
RUN mkdir $MODEL_DIR

ENV TRANSFORMERS_CACHE=$MODEL_DIR \
    TRANSFORMERS_VERBOSITY=error

ENV AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
    AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY

RUN yum install git -y && yum -y install gcc-c++
COPY requirements_inference.txt requirements_inference.txt
RUN pip install -r requirements_inference.txt --no-cache-dir
COPY ./ ./
ENV PYTHONPATH "${PYTHONPATH}:./"
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8
RUN pip install "dvc[s3]"
# configuring remote server in dvc
RUN dvc init --no-scm
RUN dvc remote add -d model-store s3://models-dvc/trained_models/

# pulling the trained model
RUN dvc pull dvcfiles/trained_model.dvc

RUN python lambda_handler.py
RUN chmod -R 0755 $MODEL_DIR
CMD [ "lambda_handler.lambda_handler"]
```

Most of the contents of the Dockerfile stays the same. Changes are the following:

- Changed the base image to `amazon/aws-lambda-python`

- Added the Transformers cache directory as `models`

- Added a sample run `python lambda_handler.py`. This will download the tokenizer required and saves it in the cache

- Given permissions to the `models` directory

- Modified the default CMD to run as `lambda_handler.lambda_handler`. Now the lambda knows what function to invoke when the container is created.

### Adding API Gateway trigger to the Lambda

- Go to the API Gateway and create a `New API`

![normal](/static/images/lambda/api_6.png)

- Select the API type as `REST API`

![normal](/static/images/lambda/api_7.png)

- Give it a name

![normal](/static/images/lambda/api_8.png)

- Now let's create a resource. Resource is like endpoint.

![normal](/static/images/lambda/api_9.png)

- Give it a name. Here I am naming it as `predict`.

![normal](/static/images/lambda/api_10.png)

- Now let's create a method for that resource

![normal](/static/images/lambda/api_11.png)

- Type of the method. I am creating method type as `POST`. Since we are connecting the API to Lambda, choose the Integration type as Lambda function and select the correct Lambda function. Make sure to check the box `Use Lambda Proxy Integration`

![normal](/static/images/lambda/api_12.png)

- In order to be able to use the API, it has to be deployed.

![normal](/static/images/lambda/api_13.png)

- Create a stage name for the API. I am creating the stage name as `deploy`

![normal](/static/images/lambda/api_14.png)

- Navigate to the `Stages` section and the `post` method of the resource `predict`. A `Invoke URL` will be present.

![normal](/static/images/lambda/api_15.png)

- Go the Lambda and refresh. `API Gateway` trigger is enabled. Click on the API Gateway to check the configuration

![normal](/static/images/lambda/lambda_10.png)

Now that the API Gateway is integrated, let's call it. Go to `Postman` and create a POST method with the Invoke URL and body containing `sentence` parameter.

![normal](/static/images/lambda/postman.png)

## Automating deployment to Lambda using Github Actions

Now whenever we want to change some code or update model, the lambda also needs to be updated with the latest image. And it becomes hectic to do all the things manually. So let's create a Github Action for updating the Lambda function whenever the ECR image is updated.

Go to the `.github/workflows/build_docker_image.yaml` file and add the following:

```yaml
- name: Update lambda with image
  run: aws lambda update-function-code --function-name  MLOps-Basics --image-uri 246113150184.dkr.ecr.us-west-2.amazonaws.com/mlops-basics:latest
```

This step will now update the lambda function `MLOps-Basics` with the image `246113150184.dkr.ecr.us-west-2.amazonaws.com/mlops-basics:latest`. (Modify this according to the image tag)

## 🔚

This concludes the post. We have seen how to create serverless application using `AWS Lambda` from the docker image and how to invoke it using `API Gateway`.

Complete code for this post can also be found here: [Github](https://github.com/graviraja/MLOps-Basics)

## References

- [AWS Serverless Architecture Intro](https://aws.amazon.com/lambda/serverless-architectures-learn-more/)

- [CloudFlare documentation on serverless](https://www.cloudflare.com/en-in/learning/serverless/why-use-serverless/)


---
---
**Note: The purpose of the project to explore the libraries and learn how to use them. Not to build a SOTA model.**

## Requirements:

This project uses Python 3.8

Create a virtual env with the following command:

```
conda create --name project-setup python=3.8
conda activate project-setup
```

Install the requirements:

```
pip install -r requirements.txt
```

## Running

### Training

After installing the requirements, in order to train the model simply run:

```
python train.py
```

### Monitoring

Once the training is completed in the end of the logs you will see something like:

```
wandb: Synced 5 W&B file(s), 4 media file(s), 3 artifact file(s) and 0 other file(s)
wandb:
wandb: Synced proud-mountain-77: https://wandb.ai/raviraja/MLOps%20Basics/runs/3vp1twdc
```

Follow the link to see the wandb dashboard which contains all the plots.

### Versioning data

Refer to the blog: [DVC Configuration](https://www.ravirajag.dev/blog/mlops-dvc)

### Exporting model to ONNX

Once the model is trained, convert the model using the following command:

```
python convert_model_to_onnx.py
```

### Inference

#### Inference using standard pytorch

```
python inference.py
```

#### Inference using ONNX Runtime

```
python inference_onnx.py
```

## S3 & ECR

Follow the instructions mentioned in the [blog post](https://www.ravirajag.dev/blog/mlops-container-registry) for creating S3 bucket and ECR repository. 

### Configuring dvc

```
dvc init (this has to be done at root folder)
dvc remote add -d model-store s3://models-dvc/trained_models/
```

### AWS credentials

Create the credentials as mentioned in the [blog post](https://www.ravirajag.dev/blog/mlops-container-registry)

**Do not share the secrets with others**

Set the ACCESS key and id values in environment variables.

```
export AWS_ACCESS_KEY_ID=<ACCESS KEY ID>
export AWS_SECRET_ACCESS_KEY=<ACCESS SECRET>
```

### Trained model in DVC

Sdd the trained model(onnx) to dvc using the following command:

```shell
cd dvcfiles
dvc add ../models/model.onnx --file trained_model.dvc
```

Push the model to remote storage

```shell
dvc push trained_model.dvc
```

### Docker

Install the docker using the [instructions here](https://docs.docker.com/engine/install/)

Build the image using the command

```shell
docker build -t mlops-basics:latest .
```

**The default command in dockerfile is modified to support the lambda. If you want to run without lambda use the last weeks dockerfile.**

Then run the container using the command

```shell
docker run -p 8000:8000 --name inference_container mlops-basics:latest
```

(or)

Build and run the container using the command

```shell
docker-compose up
```

### Pushing the image to ECR

Follow the instructions mentioned in [blog post](https://www.ravirajag.dev/blog/mlops-container-registry) for creating ECR repository.

- Authenticating docker client to ECR

```
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 246113150184.dkr.ecr.us-west-2.amazonaws.com
```

- Tagging the image

```
docker tag mlops-basics:latest 246113150184.dkr.ecr.us-west-2.amazonaws.com/mlops-basics:latest
```

- Pushing the image

```
docker push 246113150184.dkr.ecr.us-west-2.amazonaws.com/mlops-basics:latest
```

Refer to `.github/workflows/build_docker_image.yaml` file for automatically creating the docker image with trained model and pushing it to ECR.

### Serveless - Lambda

Refer to the [Blog Post here](https://www.ravirajag.dev/blog/mlops-serverless) for detailed instructions on configuring lambda with the docker image and invoking it using a API.


### Running notebooks

I am using [Jupyter lab](https://jupyter.org/install) to run the notebooks.

Since I am using a virtualenv, when I run the command `jupyter lab` it might or might not use the virtualenv.

To make sure to use the virutalenv, run the following commands before running `jupyter lab`

```
conda install ipykernel
python -m ipykernel install --user --name project-setup
pip install ipywidgets
```
