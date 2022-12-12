
![normal](/static/images/monitor/kibana_flow.png)

## What is the need of monitoring?

Monitoring systems can help give us confidence that our systems are running smoothly and, in the event of a system failure, can quickly provide appropriate context when diagnosing the root cause.

Things we want to monitor during and training and inference are different. During training we are concered about whether the loss is decreasing or not, whether the model is overfitting, etc.

But, during inference, We like to have confidence that our model is making correct predictions.

There are many reasons why a model can fail to make useful predictions:

- The underlying data distribution has shifted over time and the model has gone stale. i.e inference data characteristics is different from the data characteristics used to train the model.

- The inference data stream contains edge cases (not seen during model training). In this scenarios model might perform poorly or can lead to errors.

- The model was misconfigured in its production deployment. (Configuration issues are common)

In all of these scenarios, the model could still make a `successful` prediction from a service perspective, but the predictions will likely not be useful. Monitoring machine learning models can help us detect such scenarios and intervene (e.g. trigger a model retraining/deployment pipeline).

The scope of this post to understand the basics of monitoring predictions.

There are different tools available for monitoring:

- [Prometheus](https://prometheus.io/)

- [Grafana](https://grafana.com/)

- [Fiddler](https://www.fiddler.ai/ml-monitoring)

- [EvidentlyAI](https://evidentlyai.com/)

- [Kibana](https://www.elastic.co/kibana/)

and many more...

I will be using `Kibana`.

In this post, I will be going through the following topics:

- `Basics of Cloudwatch Logs`

- `Creating Elastic Search Cluster`

- `Configuring Cloudwatch Logs with Elastic Search`

- `Creating Index Patterns in Kibana`

- `Creating Kibana Visualisations`

- `Creating Kibana Dashboard`

## Basics of Cloudwatch Logs

### What is Cloudwatch logs?

`Amazon CloudWatch Logs` is a service that collects and stores logs from your application and infrastructure running on AWS, provides the same features expected of any log management tool: real-time monitoring, searching and filtering, and alerts.

Let's see how to check the logs for the `lambda` we implemented in the last week.

- Go to the `MLOps-Basics` Lambda. Navigate to the `Monitor` section and `Logs` part. There will be a button indicating `View logs in cloudwatch`. Click on that.

![normal](/static/images/monitor/cwl_1.png)

- Now a new window will open (Cloudwatch) which contains the logs of the Lambda. This is the `Log group` corresponding to the lambda. Click on the top one.

![normal](/static/images/monitor/cwl_2.png)

- The logs corresponding to the latest stream will be visible as follows.

![normal](/static/images/monitor/cwl_3.png)

It will be hard to monitor the predictions using these logs. Having a dashboard containing the predictions and counts will be helpful to monitor. Let's see how to stream these logs to `Kibana` and visualise there.

## Creating Elastic Search Cluster

Let's create an `Elastic Search Cluster` which will be used to stream the logs.

- Sign in to the AWS Management Console and open the Elasticsearch Service at [https://console.aws.amazon.com/es/home](https://console.aws.amazon.com/es/home). Create a `New Domain`

- Choose the domain type as `Development and Testing` (change according to the needs)

![normal](/static/images/monitor/es_1.png)

- Give the domain a name. `mlops-cluster`

![normal](/static/images/monitor/es_2.png)

- Choosing the instance type as `t2.small.elasticsearch` (since this is for demo).

![normal](/static/images/monitor/es_3.png)

- Choosing the Network configuration as `Public access` so that it can be shared across different people easily. (With VPC also it can be shared but it requires some more configuration.)

![normal](/static/images/monitor/es_4.png)

- Get the ip of the machine [using this link](https://whatismyipaddress.com/) and then add that in the domain access policy.

![normal](/static/images/monitor/es_5.png)

- Since the instance chosen is `t2.small` it does not support `https` encryption. Deselect that option.

![normal](/static/images/monitor/es_6.png)

- After reviewing everything create the instance. This will take some time (5mins +). Once the cluster is created, the status will be shown as `Active`.

![normal](/static/images/monitor/es_7.png)

## Configuring Cloudwatch Logs with Elastic Search

### Creating a IAM role with necessary permissions

In order to stream logs to Elasticsearch cluster, Cloudwatch should have necessary permissions to write to ES Cluster. Let's a create a role with the permissions required.

- Go to the [IAM console](https://console.aws.amazon.com/iam/home) and `Roles` section. Click on `Create Role`

![normal](/static/images/monitor/iam_1.png)

- Select the `AWS Service` and the use case as `Lambda`

![normal](/static/images/monitor/iam_2.png)

- Search for `esfull` and select the `AmazonESFullAccess` policy.

![normal](/static/images/monitor/iam_3.png)

- Give the role a name `mlops-cluster-role` and save it.

![normal](/static/images/monitor/iam_4.png)

### Configuring Elasticsearch cluster to Cloudwatch logs

- Now that the role has created, go to the cloudwatch `Log Group` of the lambda. Under `Actions/Subscription Filters` there will be `Create Elasticsearch Subscription Filter`. Select it.

![normal](/static/images/monitor/cf_1.png)

- Select `mlops-cluster` under the Amazon ES cluster option. Select the `mlops-cluster-role` IAM Execution role.

![normal](/static/images/monitor/cf_2.png)

- Select the Log format as `JSON`, since we are printing the logs in Json format. Filter patterns will help in filtering the unnecessary logs and focus on the necessary ones. As a simple usecase, let's write the filter pattern as `prediction`. This will filter the logs which has prediction in it.

![normal](/static/images/monitor/cf_3.png)

- Test the filter pattern by select one of the latest log stream and then click `Test Pattern`. Now in the test results you can see only the prediction related logs.

![normal](/static/images/monitor/cf_4.png)

- Cross check once the configuration and then create it.

## Creating Index Patterns in Kibana

Now that cloudwatch is configured with Elasticsearch, let's go to Kibana dashboard. Kibana link can be accessed as below:

![normal](/static/images/monitor/es_8.png)

- Go to the `Discover` section.

![normal](/static/images/monitor/ip_1.png)

- You might see page like this. In that case fire some queries so that some logs will be created.

![normal](/static/images/monitor/ip_2.png)

- Once few logs are there, the page will look like this. Click on `Create Index Pattern`

![normal](/static/images/monitor/ip_3.png)

<!-- ![normal](/static/images/monitor/ip_4.png) -->

- Add the index pattern as `cwl-*` which indicates all the Cloudwatch Logs.

![normal](/static/images/monitor/ip_5.png)

- Include the `@timestamp` field also

![normal](/static/images/monitor/ip_6.png)

- Now we can see `prediction.label`, `prediction.label.keyword`, `prediction.score`, `text` in the extracted fields.

![normal](/static/images/monitor/ip_7.png)

- Once the pattern is created, the logs will be visible in `Discover`

![normal](/static/images/monitor/ip_8.png)

- Create a data table by selecting the relevant fields. I have selected the fields `prediction.label`, `prediction.label.keyword`, `prediction.score`, `text`..

![normal](/static/images/monitor/ip_12.png)

- Save the data table by giving it a name.

![normal](/static/images/monitor/ip_11.png)

- Adjust the refresh time to 5 secs so that the latest logs will be fetched. Click on `Start`.

![normal](/static/images/monitor/ip_10.png)

## Creating Kibana Visualisations

Let's create some visualisations.

- Go to `Visualize` section in the Kibana.

![normal](/static/images/monitor/vs_1.png)

- Click on `Create new Visualization`

![normal](/static/images/monitor/vs_2.png)

- Select `Vertical Bar` (bar chart) as the type of visualization.

![normal](/static/images/monitor/vs_3.png)

- Select the input for visualisation as `MLOps Datatable`

![normal](/static/images/monitor/vs_5.png)

- By default only Y-Axis is created with aggregation as Count. Let's create X-Axis.

![normal](/static/images/monitor/vs_6.png)

- Select the aggregation type as `Terms` and select the required field. Any custom label can also be provided.

![normal](/static/images/monitor/vs_7.png)

- Save the visualisation by giving it a title.

![normal](/static/images/monitor/vs_4.png)

## Creating Kibana Dashboard

Dashboards can be created with the necessary visualisations.

- Go to the `Dashboard` section in the Kibana

![normal](/static/images/monitor/db_1.png)

- Create new dashboard

![normal](/static/images/monitor/db_2.png)

- Click on `Add` button. This will open up a pane on the right side containing all the visualisations we have created till now.

![normal](/static/images/monitor/db_3.png)

- Select the necessary visualisations and save the dashboard by giving it a name.

![normal](/static/images/monitor/db_4.png)

- Now the dashboard contains the visualisations. These visualisations can be arranged according to the needs.

![normal](/static/images/monitor/db_5.png)

- In order to share this dashboard, click on `share` button on top right, and `permalink`. Get the `Short URL` and share it with the concerned people.

![normal](/static/images/monitor/db_6.png)

## 🔚

This concludes the post. We have seen how to monitor predictions using `Cloudwatch` & `Kibana`.

Complete code can also be found here: [Github](https://github.com/graviraja/MLOps-Basics)

## References

- [Youtube tutorial on Kibana dashboard with AWS Elasticsearch](https://www.youtube.com/watch?v=06nJRWOz5uc)

- [Jeremy Jordan Blog on Monitoring ML systems](https://www.jeremyjordan.me/ml-monitoring/#grafana)


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

### Monitoring - Kibana

Refer to the [Blog Post here](https://www.ravirajag.dev/blog/mlops-monitoring) for detailed instructions on configuring kibana using elasticsarch cluster and integrating with cloudwatch logs.


### Running notebooks

I am using [Jupyter lab](https://jupyter.org/install) to run the notebooks.

Since I am using a virtualenv, when I run the command `jupyter lab` it might or might not use the virtualenv.

To make sure to use the virutalenv, run the following commands before running `jupyter lab`

```
conda install ipykernel
python -m ipykernel install --user --name project-setup
pip install ipywidgets
```
