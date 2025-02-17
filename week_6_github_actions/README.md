
![normal](/static/images/cicd/cicd.png)

## What is CI/CD ?

CI/CD is a coding philosophy and set of practices with which you can continuously build, test, and deploy iterative code changes.

This iterative process helps reduce the chance that you develop new code based on buggy or failed previous versions. With this method, you strive to have less human intervention or even no intervention at all, from the development of new code until its deployment

A simple flow looks like:

![normal](/static/images/cicd/basic_flow.png)

And a complex flow looks like:

![normal](/static/images/cicd/complex_flow.png)

Let's stick to the `basic flow` for now.

There are many tools with which we can perform CI/CD. The prominent ones are:

- [Jenkins](https://www.jenkins.io/)

- [CircleCI](https://circleci.com/)

- [Travis CI](https://travis-ci.com/)

- [GitLab](https://about.gitlab.com/)

- [GitHub Actions](https://github.com/features/actions)

and many more...

I will be using `GitHub Actions`.

In this post, I will be going through the following topics:

- `Basics of GitHub Actions`
- `First GitHub Action`
- `Creating Google Service Account`
- `Giving access to service account`
- `Configuring DVC to use Google Service account`
- `Configuring Github Action`

_Note: MLOps includes model development also as a part of the cycle. This post covers only devops part._

<div style={{ display: 'inline-block' }}>
  <img
    alt="ocean"
    src="/static/images/cicd/ga.svg"
    style={{
      width: 25 + 'px',
      height: 25 + 'px',
      margin: 0,
      marginBottom: -5 + 'px',
      marginRight: 10 + 'px',
    }}
  />
</div>
<div style={{ display: 'inline-block' }}>
  <h2 style={{ margin: 0, padding: 0 }}>Basics of GitHub Actions</h2>
</div>

<br />
<br />

<div style={{ display: 'inline-block' }}>
  Since we are using
  <img
    alt="ocean"
    src="/static/images/cicd/github.png"
    style={{
      width: 24 + 'px',
      height: 24 + 'px',
      display: 'inline-block',
      margin: 0,
      marginLeft: 5 + 'px',
      marginRight: 5 + 'px',
    }}
  />
  as the version control system, we can use GitHub Actions right off the bat without having the need
  to setup another tool. (Might not be the same if you are using different version control system.)
</div>

GitHub Actions are just a set instructions declared using `yaml` files.

These files needs to be in a specific folder: `.github/workflows` and this has to be in the root directory (where `.git` folder is present).

There are 5 main concepts in GitHub Actions:

- `Events`: An event is a trigger for workflow.

- `Jobs`: Jobs defines the steps to run when a workflow is triggered. A workflow can contain multiple jobs.

- `Runners`: Defines where to run the code. By default, github will run the code in it's own servers.

- `Steps`: Steps contains actions to run. Each job can contains multiple steps to run.

- `Actions`: Actions contains actual commands to run like installing dependencies, testing code, etc.

## 🥇 First GitHub Action

Let's create the folder using the command:

```
mkdir .github/workflows
```

Now let's create a basic workflow file.

```yaml
name: GitHub Actions Basic Flow
on: [push]
jobs:
  Basic-workflow:
    runs-on: ubuntu-latest
    steps:
      - name: Basic Information
        run: |
          echo "🎬 The job was automatically triggered by a ${{ github.event_name }} event."
          echo "💻 This job is now running on a ${{ runner.os }} server hosted by GitHub!"
          echo "🎋 Workflow is running on the branch ${{ github.ref }}"
      - name: Checking out the repository
        uses: actions/checkout@v2
      - name: Information after checking out
        run: |
          echo "💡 The ${{ github.repository }} repository has been cloned to the runner."
          echo "🖥️ The workflow is now ready to test your code on the runner."
      - name: List files in the repository
        run: |
          ls ${{ github.workspace }}
      - run: echo "🍏 This job's status is ${{ job.status }}."
```

Let's understand what's happening here:

- Created a CICD workflow with name `GitHub Actions Basic Flow`

- `on` is called `Event` which triggers the workflow. Here it is `push` event. Whenever a push is happened on the repository, workflow will be triggered. There are 30+ ways of triggering the workflow. [Refer to the documentation for more information](https://docs.github.com/en/actions/reference/events-that-trigger-workflows)

- Workflow contains a single job called `Basic-workflow` running on `ubuntu-latest`

- `Basic-workflow` job contains multiple steps (Basic Information, Checking out the repository, Information after checking out, List files in the repository)

- `Basic Information` step contains the actions to do some echoing.

- `Checking out the repository` step contains the action to checkout the repository. Here we are using `actions/checkoutv2` which is a open source action. [Check for other available actions here](https://github.com/marketplace?type=actions)

- `Information after checking out` step contains the action to echo some information about repository and runner.

- `List files in the repository` step contains the action to list the contents of the repository.

Commit the file. Let's see how does it look in github.

- On GitHub, navigate to the main page of the repository.

- Under your repository name, click Actions.

![normal](/static/images/cicd/actions.png)

- In the left sidebar, you can see the workflows.

![normal](/static/images/cicd/workflow.png)

- You can see a workflow is trigged.

![normal](/static/images/cicd/start.png)

- Select the latest run of the required workflow.

![normal](/static/images/cicd/latest.png)

- Select the job

![normal](/static/images/cicd/job.png)

- Job contains all the steps it ran

![normal](/static/images/cicd/steps.png)

- The logs shows the execution of steps present in the job.

![normal](/static/images/cicd/logs.png)

- The icon indicates whether the workflow ran sucessfully or not.

![normal](/static/images/cicd/success.png)

Now that we understand how to configure a basic workflow, let's configure workflow for the project.

In the previous posts, we have seen how to push the model to dvc, pull the model from dvc, create docker container and test it locally. Let's see how to do it using GitHub Actions.

## ⚙️ Creating Google Service Account

I have used Google Drive as the remote server for storing the trained model. In order to pull the model from Google Drive we need authentication. Ideally a link will be prompted (for the first time) when we attempt to push / pull the model from remote server and copy&paste the password prompted.

But it will be difficult to do this automatically. Inorder to be able to download the model and test it automatically in CICD, `service account` can be used.

### What are service accounts?

A service account is a Google account associated with your GCP project, and not a specific user. They are intended for scenarios where your code needs to access data on its own, e.g. running inside a Compute Engine, automatic CI/CD, etc. No interactive user OAuth authentication is needed.

- Go to [GCP consle](https://console.cloud.google.com/) and create a project.

- To [create a service account](https://cloud.google.com/docs/authentication/getting-started#creating_a_service_account), navigate to `IAM & Admin` in the left sidebar, and select `Service Accounts`. Click `+ CREATE SERVICE ACCOUNT`, on the next screen, enter Service account name e.g. "MLOps", and click Create.

![normal](/static/images/cicd/service_account.png)

- Provide a name to service account like `model` and click `Done`

![normal](/static/images/cicd/name.png)

- Go the `keys` tab and create a new key. When prompted choose the key type as json.

![normal](/static/images/cicd/keys.png)

_A json file will be downloaded_

⚠️ Be careful about sharing the key file with others.

- Enable Google Drive API for the project. Search for Google Drive API in the search bar and enable it for the project

![normal](/static/images/cicd/drive_api.png)

![normal](/static/images/cicd/drive_api2.png)

## 🤝 Giving access to service account

Now that service account is created, add this to the remote server (google drive).

Go to the google drive and navigate to the remote storage folder (MLOps) and this service account email in the sharing permissions.

![normal](/static/images/cicd/access.png)

## ⚙️ Configuring DVC to use Google Service account

Now let's modify the dvc to use service account instead of acutal google account.

This can be done via

```shell
dvc remote add -d storage gdrive://19JK5AFbqOBlrFVwDHjTrf9uvQFtS0954
dvc remote modify storage gdrive_use_service_account true
dvc remote modify storage gdrive_service_account_json_file_path creds.json
```

Let's understand the commands here:

- We are creating a default remote storage with name `storage` and link `gdrive://19JK5AFbqOBlrFVwDHjTrf9uvQFtS0954`

- Configuring the remote storage to use service account

- Providing the credentials (json file which is created)

We can test it by trying to pull the model using the command:

```
cd dvcfiles
dvc pull trained_model.dvc
```

<br />

<div style={{ display: 'inline-block' }}>
  <img
    alt="ocean"
    src="/static/images/cicd/ga.svg"
    style={{
      width: 25 + 'px',
      height: 25 + 'px',
      margin: 0,
      marginBottom: -5 + 'px',
      marginRight: 10 + 'px',
    }}
  />
</div>
<div style={{ display: 'inline-block' }}>
  <h2 style={{ margin: 0, padding: 0 }}>Configuring GitHub Action</h2>
</div>

<br />

Before creating new a workflow, let's modify the dockerfile to accomodate all the changes.

```
FROM huggingface/transformers-pytorch-cpu:latest

COPY ./ /app
WORKDIR /app

# install requirements
RUN pip install "dvc[gdrive]"
RUN pip install -r requirements_inference.txt

# initialise dvc
RUN dvc init --no-scm
# configuring remote server in dvc
RUN dvc remote add -d storage gdrive://19JK5AFbqOBlrFVwDHjTrf9uvQFtS0954
RUN dvc remote modify storage gdrive_use_service_account true
RUN dvc remote modify storage gdrive_service_account_json_file_path creds.json

# pulling the trained model
RUN dvc pull dvcfiles/trained_model.dvc

ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

# running the application
EXPOSE 8000
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

**NOTE: Do not share the credentials json file publicly.**

Now let's create a new github action file in `.github/worflows` folder as `build_docker_image.yaml`

```yaml
name: Create Docker Container

on: [push]

jobs:
  mlops-container:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./week_6_github_actions
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          ref: ${{ github.ref }}
      - name: Build container
        run: |
          docker network create data
          docker build --tag inference:latest .
          docker run -d -p 8000:8000 --network data --name inference_container inference:latest
```

The file pretty self explanatory.

In Github, this will look this:

![normal](/static/images/cicd/dc_success.png)

## 🔚

This concludes the post. We have seen how to automatically create a docker image using GitHub Actions. But the problem here is it is not accessible. Also `json` way of configuring credentials is not a good practice. In the next post, I will explore into `AWS S3` as the remote storage, `AWS ECR` for storing the docker images.

Complete code for this post can also be found here: [Github](https://github.com/graviraja/MLOps-Basics)

## References

- [Configuring service account](https://dvc.org/doc/user-guide/setup-google-drive-remote)

- [Github actions](https://docs.github.com/en/actions/quickstart)


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

### Google Service account

Create service account using the steps mentioned here: [Create service account](https://www.ravirajag.dev/blog/mlops-github-actions)

### Configuring dvc

```
dvc init
dvc remote add -d storage gdrive://19JK5AFbqOBlrFVwDHjTrf9uvQFtS0954
dvc remote modify storage gdrive_use_service_account true
dvc remote modify storage gdrive_service_account_json_file_path creds.json
```

`creds.json` is the file created during service account creation


### Docker

Install the docker using the [instructions here](https://docs.docker.com/engine/install/)

Build the image using the command

```shell
docker build -t inference:latest .
```

Then run the container using the command

```shell
docker run -p 8000:8000 --name inference_container inference:latest
```

(or)

Build and run the container using the command

```shell
docker-compose up
```


### Running notebooks

I am using [Jupyter lab](https://jupyter.org/install) to run the notebooks.

Since I am using a virtualenv, when I run the command `jupyter lab` it might or might not use the virtualenv.

To make sure to use the virutalenv, run the following commands before running `jupyter lab`

```
conda install ipykernel
python -m ipykernel install --user --name project-setup
pip install ipywidgets
```
