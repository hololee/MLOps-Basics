
![normal](/static/images/ecr/ecr_flow.png)

## What is Container Registry ?

A container registry is a place to store container images. A container image is a file comprised of multiple layers which can execute applications in a single instance. Hosting all the images in one stored location allows users to commit, identify and pull images when needed.

There are many tools with which we can store the container images. The prominent ones are:

- [Docker Hub](https://hub.docker.com/)

- [Amazon Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/)

- [JFrog Container Registry](https://jfrog.com/container-registry/)

- [Google Container Registry](https://cloud.google.com/container-registry)

- [Azure container Registry](https://azure.microsoft.com/en-in/services/container-registry/#features)

and many more...

I will be using `AWS ECR`.

In this post, I will be going through the following topics:

- `Basics of S3`

- `Programmatic access to S3`

- `Configuring AWS S3 as remote storage in DVC`

- `Basics of ECR`

- `Configuring GitHub Actions to use S3, ECR`

## Basics of S3

### What is S3?

Amazon Simple Storage Service (S3) is a storage for the internet. It is designed for large-capacity, low-cost storage provision across multiple geographical regions.

Amazon S3 provides developers and IT teams with Secure, Durable and Highly Scalable object storage.

### How is data organized in S3?

Data in S3 is organized in the form of buckets.

- A Bucket is a logical unit of storage in S3.

- A Bucket contains objects which contain the data and metadata.

Before adding any data in S3 the user has to create a bucket which will be used to store objects.

### Creating bucket

- Sign in to the AWS Management Console and open the Amazon S3 console at [https://console.aws.amazon.com/s3/](https://console.aws.amazon.com/s3/)

- Click on `Create Bucket`

![normal](/static/images/ecr/create_bucket.png)

- Bucket name details

![normal](/static/images/ecr/bucket_name.png)

- Created bucket

![normal](/static/images/ecr/bucket.png)

- Uploading a sample file

![normal](/static/images/ecr/upload_1.png)

- Select any sample and upload it. After uploading, the bucket looks like

![normal](/static/images/ecr/upload_2.png)

Now that we have seen how to create a bucket and upload files, let's see how to access s3 programmatically.

## Programmatic access to s3

We can access s3 either via cli or any programming language. Let's see both ways.

Credentials are required to access any aws service. There are different ways of configuring credentials. Let's look at a simple way.

- Go to `My Security Credentials`

![normal](/static/images/ecr/key1.png)

- Navigate to `Access Keys` section and click on `Create New Access Key` button.

![normal](/static/images/ecr/key2.png)

This will download a csv file containing the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`

**Do not share the secrets with others**

Set the ACCESS key and id values in environment variables.

```
export AWS_ACCESS_KEY_ID=<ACCESS KEY ID>
export AWS_SECRET_ACCESS_KEY=<ACCESS SECRET>
```

### Accessing s3 using CLI

[Download the AWS CLI package and install it from here](https://aws.amazon.com/cli/)

`aws cli` comes with a lot of commands. [Check the documentation here](https://docs.aws.amazon.com/cli/latest/index.html)

Let's see what all present in s3 bucket using cli.

```
aws s3 ls s3://models-dvc/
```

Output looks like

```
(base) ravirajas-MacBook-Pro » aws s3 ls s3://models-dvc/
2021-07-24 12:39:21         22 sample.txt
```

For all the available list of commands, refer to the [documentation here](https://docs.aws.amazon.com/cli/latest/index.html)

### Accessing s3 using Python

Install the `boto3` library which is AWS SDK for Python

```
pip install boto3
```

The following code prints the contents of s3 bucket.

```python
import boto3

s3 = boto3.resource('s3')
bucket = s3.Bucket('models-dvc')
for obj in bucket.objects.all():
    print(obj.key)
```

Output looks like:

```
sample.txt
```

For all the available list of commands, refer to the [documentation here](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)

## Configuring AWS S3 as remote storage in DVC

Let's see how to configure s3 as the remote storage in DVC where trained models can be pushed.

Let's create a folder called `trained_models` in s3 which will be used for storing the trained models.

In order to use dvc for s3 make sure you install dvc with s3 support.

```shell
pip install "dvc[s3]"
```

Initialise the dvc (if not initialised) using the following command:

```
dvc init
```

Configure the remote storage to s3 location.

Get the s3 url as mentioned below

![normal](/static/images/ecr/url.png)

Add the s3 folder as remote storage for models in dvc.

```
dvc remote add -d model-store s3://models-dvc/trained_models/
```

Make sure the AWS credentials are set in ENV.

Now let's add the trained model to dvc using the following command:

```shell
cd dvcfiles
dvc add ../models/model.onnx --file trained_model.dvc
```

Push the model to remote storage

```shell
dvc push trained_model.dvc
```

Once the model is pushed to dvc, refresh the s3.

![normal](/static/images/ecr/model.png)

## Basics of ECR

In the previous week, we have build container using CICD, but the image is not persisted anywhere for further usage. This is where `Container Registry` comes into the picture.

![normal](/static/images/ecr/ecr.png)

Search for ECR and click on `Get Started`

![normal](/static/images/ecr/ecr1.png)

Create a repository when prompted with name `mlops-basics`

![normal](/static/images/ecr/ecr2.png)

Let's build the docker image and push it to ECR.

Before building the docker image need to modify the `Dockfile`. Till now I am using Google Drive as the remote storage. That needs to be changed to S3.

The dockerfile looks like:

```
FROM huggingface/transformers-pytorch-cpu:latest

COPY ./ /app
WORKDIR /app

ARG AWS_ACCESS_KEY_ID
ARG AWS_SECRET_ACCESS_KEY


# aws credentials configuration
ENV AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
    AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY


# install requirements
RUN pip install "dvc[s3]"	# since s3 is the remote storage
RUN pip install -r requirements_inference.txt

# initialise dvc
RUN dvc init --no-scm

# configuring remote server in dvc
RUN dvc remote add -d model-store s3://models-dvc/trained_models/

RUN cat .dvc/config

# pulling the trained model
RUN dvc pull dvcfiles/trained_model.dvc

ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

# running the application
EXPOSE 8000
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

Build the docker image using the command:

```shell
docker build --build-arg AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID --build-arg AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY  -t inference:test .
```

Now let's push the image to ECR.

Commands required to push the image to ECR can be found in the ECR itself

![normal](/static/images/ecr/push.png)

Following the commands there:

- Authenticating docker client to ECR

```
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 246113150184.dkr.ecr.us-west-2.amazonaws.com
```

- Tagging the image

```
docker tag inference:test 246113150184.dkr.ecr.us-west-2.amazonaws.com/mlops-basics:latest
```

- Pushing the image

```
docker push 246113150184.dkr.ecr.us-west-2.amazonaws.com/mlops-basics:latest
```

## Configuring GitHub Actions to use S3, ECR

Now let's see how to configure S3, ECR in Github Actions

We need AWS credentials for fetching the model from S3, Pushing the image to ECR. We can't share this information publicly. Fortunately GitHub Actions has a way to store these information securely.

It's called `Secrets`.

- Go to the `settings` tab of the repository

![normal](/static/images/ecr/secret1.png)

- Go to `Secrets` section and click on `New repository secret`

![normal](/static/images/ecr/secret2.png)

- Save the following secrets:

  - `AWS_ACCESS_KEY_ID`

  - `AWS_SECRET_ACCESS_KEY`

  - `AWS_ACCOUNT_ID` (this is the account id of profile)

These values can be used in GitHub Actions in the following manner:

- AWS_ACCESS_KEY_ID: `{{secrets.AWS_ACCESS_KEY_ID}}`
- AWS_SECRET_ACCESS_KEY: `{{secrets.AWS_SECRET_ACCESS_KEY}}`
- AWS_ACCOUNT_ID: `{{secrets.AWS_ACCOUNT_ID}}`

Let's modify the workflow file.

[GitHub Actions Marketplace](https://github.com/marketplace?type=actions) comes with lot of predefined actions which are useful for us.

- [`aws-actions/configure-aws-credentials@v1`](https://github.com/aws-actions/configure-aws-credentials) will be useful to configure AWS credential environment variables for use in other GitHub Actions.

- [`jwalton/gh-ecr-push@v1`](https://github.com/jwalton/gh-ecr-push) will be useful to push/pull the image to ECR.

```yaml
name: Create Docker Container

on: [push]

jobs:
  mlops-container:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./week_7_ecr
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          ref: ${{ github.ref }}
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-west-2
      - name: Build container
        run: |
          docker build --build-arg AWS_ACCOUNT_ID=${{ secrets.AWS_ACCOUNT_ID }} \
                       --build-arg AWS_ACCESS_KEY_ID=${{ secrets.AWS_ACCESS_KEY_ID }} \
                       --build-arg AWS_SECRET_ACCESS_KEY=${{ secrets.AWS_SECRET_ACCESS_KEY }} \
                       --tag mlops-basics .
      - name: Push2ECR
        id: ecr
        uses: jwalton/gh-ecr-push@v1
        with:
          access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          region: us-west-2
          image: mlops-basics:latest
```

Let's understand what happening here:

- Jobs will run on `ubuntu-latest` runner

- Clones the code and navigates to `week_7_ecr` directory

- Sets the AWS environment variables using `aws-actions/configure-aws-credentials@v1` action

- Builds the image and tag it with `mlops-basics` tag

- Push the image to ECR using `jwalton/gh-ecr-push@v1` action.

Output will look like:

In actions tab Github:

![normal](/static/images/ecr/ga.png)

In the ECR:

![normal](/static/images/ecr/img.png)

## 🔚

This concludes the post. We have seen how to automatically create a docker image using GitHub Actions and save it to ECR. How to use S3 as the remote storage.

Complete code for this post can also be found here: [Github](https://github.com/graviraja/MLOps-Basics)

## References

- [AWS S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)

- [AWS ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)


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


### Running notebooks

I am using [Jupyter lab](https://jupyter.org/install) to run the notebooks.

Since I am using a virtualenv, when I run the command `jupyter lab` it might or might not use the virtualenv.

To make sure to use the virutalenv, run the following commands before running `jupyter lab`

```
conda install ipykernel
python -m ipykernel install --user --name project-setup
pip install ipywidgets
```
