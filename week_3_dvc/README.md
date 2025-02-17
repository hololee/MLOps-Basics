
# Data Version Control

Machine learning and data science come with a set of problems that are different from what you’ll find in traditional software engineering. Version control systems help developers manage changes to source code. But data version control, managing changes to models and datasets, isn’t so well established.

It’s not easy to keep track of all the data you use for experiments and the models you produce. Accurately reproducing experiments that you or others have done is a challenge.

There are many libraries which supports versioning of models and data. The prominent ones are:

- [DVC](https://dvc.org/)

- [DAGsHub](https://dagshub.com/)

- [Hub](https://www.activeloop.ai/)

- [Modelstore](https://modelstore.readthedocs.io/en/latest/)

- [ModelDB](https://github.com/VertaAI/modeldb/)

and many more...

I will be using `DVC`.

In this post, I will be going through the following topics:

- `Basics of DVC`
- `Initialising DVC`
- `Configuring Remote Storage`
- `Saving Model to the Remote Storage`
- `Versioning the models`

_Note: Basic Knowledge of GIT is needed_

<div style={{ display: 'inline-block' }}>
  <h1 style={{ margin: 0, padding: 0 }}>Basics of</h1>
</div>
<div style={{ display: 'inline-block' }}>
  <img
    alt="ocean"
    src="/static/images/dvc/dvc_svg.png"
    style={{
      width: 48 + 'px',
      height: 48 + 'px',
      margin: 0,
      marginBottom: -15 + 'px',
      marginLeft: 10 + 'px',
    }}
  />
</div>

<br />
<br />
<div>
  <div style={{ display: 'inline-block' }}>
    <img
      alt="ocean"
      src="/static/images/dvc/dvc_svg.png"
      style={{
        width: 28 + 'px',
        height: 28 + 'px',
        display: 'inline-block',
        margin: 0,
        marginLeft: 5 + 'px',
        marginRight: 5 + 'px',
      }}
    />
    (Data Version Control) is a new type of data versioning, workflow, and experiment management software,
    that builds upon Git.
  </div>
</div>

Data science experiment sharing and collaboration(processing, training code, configurations, etc.) can be done through a regular `Git flow` (commits, branching, pull requests, etc.), the same way it works for software engineers.

Data versioning is enabled by replacing large files, dataset directories, machine learning models, etc. with `small metafiles` (easy to handle with Git). These placeholders point to the original data, which is decoupled from source code management.

All the large files, datasets, models, etc. can be stored in `remote storage servers` (S3, Google Drive, etc). DVC supports easy-to-use commands to configure, push, pull datasets to remote storage.

Git tracks the metadata file, while DVC handles the remote repository.

Using `Git` and `DVC`, data science and machine learning teams can:

- version experiments
- manage large datasets
- make projects reproducible.

![normal](/static/images/dvc/dvc_1.png)

<br />
<div style={{ display: 'inline-block' }}>
  <h1 style={{ margin: 0, padding: 0 }}>🎬 Initialising</h1>
</div>
<div style={{ display: 'inline-block' }}>
  <img
    alt="ocean"
    src="/static/images/dvc/dvc_svg.png"
    style={{
      width: 48 + 'px',
      height: 48 + 'px',
      margin: 0,
      marginBottom: -15 + 'px',
      marginLeft: 10 + 'px',
    }}
  />
</div>

Let's first install DVC using the following command:

```shell
pip install dvc
```

See other ways of installation [here](https://dvc.org/doc/install)

Many commands are similar to `GIT`.

<div style={{ display: 'inline-block' }}>
  Let's initialise the
  <img
    alt="ocean"
    src="/static/images/dvc/dvc_svg.png"
    style={{
      width: 24 + 'px',
      height: 24 + 'px',
      display: 'inline-block',
      margin: 0,
      marginLeft: 5 + 'px',
      marginRight: 5 + 'px',
    }}
  />
  using the following command:
</div>

```shell
dvc init
```

_Make sure you run the command in the top level folder. Ideally where the .git folder is present_

Upon initialisation you will see output like:

![normal](/static/images/dvc/dvc_init.png)

This command will create `.dvc` folder and `.dvcignore` file. (Similar to git)

<br />

# 💽 Configuring Remote Storage

Now let's configure some remote storage to store our trained models (or datasets).

<div style={{ display: 'inline-block' }}>
  <img
    alt="ocean"
    src="/static/images/dvc/dvc_svg.png"
    style={{
      width: 24 + 'px',
      height: 24 + 'px',
      display: 'inline-block',
      margin: 0,
      marginLeft: 5 + 'px',
      marginRight: 5 + 'px',
    }}
  />
  offers support integration with wide range of remote storages.
</div>

For simplicity, I will be configuring `Google Drive` as the remote storage.

I have created a folder called `MLOps-Basics` in my Google Drive.

![normal](/static/images/dvc/gdrive.png)

Now let's configure this model as remote storage.

Run the following command:

```shell
dvc remote add -d storage gdrive://19JK5AFbqOBlrFVwDHjTrf9uvQFtS0954
```

Make sure the `ID` after _gdrive://_ matches the same in the google drive folder.

Once the command is ran, check the contents of the file `.dvc/config` whether the remote storage is configured correctly or not.

It will something like:

```shell
[core]
    remote = storage
['remote "storage"']
    url = gdrive://19JK5AFbqOBlrFVwDHjTrf9uvQFtS0954
```

<br />

# 🔁 Saving Model to the Remote Storage

Now let's add the trained model to the remote storage.

First run the code

```shell
python train.py
```

Now the trained model is available in the `models` folder as `best-checkpoint.ckpt`

Ideally, people do

```shell
dvc add models/best-checkpoint.ckpt

```

and this will create the file `models/best-checkpoint.ckpt.dvc`. I want to follow a slightly different way for making the management of `.dvc` files a bit easier.

Let's create a folder called `dvcfiles`.

The folder structure looks like:

```shell
.
├── README.md
├── configs
│   ├── config.yaml
│   ├── model
│   │   └── default.yaml
│   ├── processing
│   │   └── default.yaml
│   └── training
│       └── default.yaml
├── data.py
├── dvcfiles
├── experimental_notebooks
│   └── data_exploration.ipynb
├── inference.py
├── model.py
├── models
│   └── best-checkpoint.ckpt
├── outputs
├── requirements.txt
├── train.py
```

Now let's navigate to the `dvcfiles` folder and do the following.

```shell
dvc add ../models/best-checkpoint.ckpt --file trained_model.dvc
```

What we are doing here is:

- Adding the trained model
- Instead of deafult `.dvc` file name we are telling to create the `.dvc` file with `trained_model.dvc` name.

By doing this way, you can always know where the dvc files are. You don't need to remember the paths where the data is stored.

<div style={{ display: 'inline-block' }}>
  <img
    alt="ocean"
    src="/static/images/dvc/dvc_svg.png"
    style={{
      width: 24 + 'px',
      height: 24 + 'px',
      display: 'inline-block',
      margin: 0,
      marginLeft: 5 + 'px',
      marginRight: 5 + 'px',
    }}
  />
  creates 2 files when you run the <code>add</code> command. <code>.dvc</code> file and <code>
    .gitignore
  </code> file. So DVC takes care of not pushing the model to git.
</div>

Now let's push the model to `remote storage` by running the following command:

```shell
dvc push trained_model.dvc
```

This will ask for authenication

![normal](/static/images/dvc/auth1.png)

Copy paste the code in the link prompted

![normal](/static/images/dvc/auth2.png)

Once authenicated, the data will be pushed.

```
1 file pushed
```

Check the google drive, a folder will be created with some name.

![normal](/static/images/dvc/gdrive_folder.png)

Now the final step is to commit the dvc files to git. Run the following commands:

```shell
git add dvcfiles/trained_model.dvc ../models/.gitignore

git commit -m "Added trained model to google drive using dvc"

git push
```

Let's delete the model from `models/best-checkpoint.ckpt` and pull from remote storage using dvc.

```shell
rm models/best-checkpoint.ckpt
```

Then navigate to the `dvcfiles` folder and then run the command:

```shell
dvc pull trained_model.dvc
```

You will see output as:

```shell
A       ../models/best-checkpoint.ckpt
1 file added
```

<div style={{ display: 'inline-block' }}>
  As you can see,
  <img
    alt="ocean"
    src="/static/images/dvc/dvc_svg.png"
    style={{
      width: 24 + 'px',
      height: 24 + 'px',
      display: 'inline-block',
      margin: 0,
      marginLeft: 5 + 'px',
      marginRight: 5 + 'px',
    }}
  />
  follows
  <img
    alt="ocean"
    src="/static/images/dvc/git.png"
    style={{
      width: 18 + 'px',
      height: 18 + 'px',
      display: 'inline-block',
      margin: 0,
      marginLeft: 5 + 'px',
      marginRight: 5 + 'px',
    }}
  />
  pattern to <code>commit</code>, <code>push</code> and <code>pull</code> data to remote storage.
</div>

<br />
<br />

# 🏷 Versioning the models

Versioning is same as tagging in git. By tagging the commit, we are telling that particular dvc files belong to that version.

Let's create a tag called `v0.0` as the version for the trained model.

```shell
git tag -a "v0.0" -m "Version 0.0"
```

Then push the tags to git.

```shell
git push origin v0.0
```

Now you can see the tag in git under `tags`

![normal](/static/images/dvc/tag.png)

Let's update the model (as an example trained with more epochs).

```shell
python train.py training.max_epochs=3
```

<div style={{ display: 'inline-block' }}>
  Now the model is updated. Let's add it to
  <img
    alt="ocean"
    src="/static/images/dvc/dvc_svg.png"
    style={{
      width: 24 + 'px',
      height: 24 + 'px',
      display: 'inline-block',
      margin: 0,
      marginLeft: 5 + 'px',
    }}
  />
</div>

```shell
cd dvcfiles
dvc add ../models/best-checkpoint.ckpt --file trained_model.dvc
dvc push trained_model.dvc
```

Now let's create a new version for this model.

```shell
git tag -a "v1.0" -m "Version 1.0"
```

Let's push all this to git.

```shell
git commit -m "updated model version"
git push
# push the tag also
git push origin v1.0
```

Now in the git you can see

![normal](/static/images/dvc/tag_2.png)

Switching the versions is as simple as navigating to the required tag and pulling the corresponding files.

According to the data present `.dvc` file the model will be updated.

Make sure to run the command to get the corresponding data:

```
cd dvcfiles
dvc pull trained_model.dvc
```

## 🔚

<div style={{ display: 'inline-block' }}>
  This concludes the post. These are only a few capabilities of
  <img
    alt="ocean"
    src="/static/images/dvc/dvc_svg.png"
    style={{
      width: 24 + 'px',
      height: 24 + 'px',
      display: 'inline-block',
      margin: 0,
      marginLeft: 5 + 'px',
    }}
  />. There are many other functionalities like:
</div>

- [Fetching data from a different repository](https://dvc.org/doc/start/data-and-model-access#download)

- [Creating Data Pipelines for reproducing exact result](https://dvc.org/doc/start/data-pipelines#get-started-data-pipelines)

- [Plotting of metrics](https://dvc.org/doc/start/metrics-parameters-plots)

- [Running experiments with different configurations](https://dvc.org/doc/start/experiments)

and much more... Refer to the [original documentation](https://dvc.org/doc) for more information.

Complete code for this post can also be found here: [Github](https://github.com/graviraja/MLOps-Basics)

## References

- [DVC Documentation](https://dvc.org/doc)
- [DVC Tutorial on Versioning data](https://www.youtube.com/watch?v=kLKBcPonMYw)


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

### Inference

After training, update the model checkpoint path in the code and run

```
python inference.py
```

### Versioning data

Refer to the blog: [DVC Configuration](https://www.ravirajag.dev/blog/mlops-dvc)

### Running notebooks

I am using [Jupyter lab](https://jupyter.org/install) to run the notebooks.

Since I am using a virtualenv, when I run the command `jupyter lab` it might or might not use the virtualenv.

To make sure to use the virutalenv, run the following commands before running `jupyter lab`

```
conda install ipykernel
python -m ipykernel install --user --name project-setup
pip install ipywidgets
```
