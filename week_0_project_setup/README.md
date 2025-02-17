

# 🎬 Start of the series

The goal of the series is to understand the basics of MLOps (model building, monitoring, configurations, testing, packaging, deployment, cicd). As a first step, Let's setup the project. I am particularly interested towards NLP (personal bias) but the process and tools stays the same irrespective of the project. I will be using a simple classification task.

In this post, I will be going through the following topics:

- `How to get the data?`
- `How to process the data?`
- `How to define dataloaders?`
- `How to declare the model?`
- `How to train the model?`
- `How to do the inference?`

_Note: Basic knowledge of Machine Learning is needed_

# 🛠 Deep Learning Library

There are many libraries available to develop deeplearning projects. The prominent ones are:

- [`Tensorflow`](https://www.tensorflow.org/)

- [`Pytorch`](https://pytorch.org/)

- [`Pytorch Lightning`](https://www.pytorchlightning.ai/) (Pytorch lightning is a wrapper around pytorch)

and many more...

I will be using `Pytorch Lightning` since it automates a lot of engineering code and comes with many cool features.

# 📚 Dataset

I will be using `CoLA`(Corpus of Linguistic Acceptability) dataset. The task is about given a sentence it has to be classified into one of the two classes.

- ❌ `Unacceptable`: Grammatically not correct
- ✅ `Acceptable`: Grammatically correct

I am using ([`Huggingface datasets`](https://huggingface.co/docs/datasets/quicktour.html)) to download and load the data. It supports `800+` datasets and also can be used with custom datasets.

Downloading the dataset is as easy as

```python
cola_dataset = load_dataset("glue", "cola")
print(cola_dataset)
```

```shell
DatasetDict({
    train: Dataset({
        features: ['sentence', 'label', 'idx'],
        num_rows: 8551
    })
    validation: Dataset({
        features: ['sentence', 'label', 'idx'],
        num_rows: 1043
    })
    test: Dataset({
        features: ['sentence', 'label', 'idx'],
        num_rows: 1063
    })
})
```

Let's see a sample datapoint

```python
train_dataset = cola_dataset['train']
print(train_dataset[0])
```

```shell
{
    'idx': 0,
    'label': 1,
    'sentence': "Our friends won't buy this analysis, let alone the next one we propose."
}
```

# 🛒 Loading data

Data pipelines can be created with:

- 🍦 Vanilla Pytorch [`DataLoaders`](https://pytorch.org/tutorials/beginner/basics/data_tutorial.html)
- ⚡ Pytorch Lightning [`DataModules`](https://pytorch-lightning.readthedocs.io/en/latest/extensions/datamodules.html)

`DataModules` are more structured definition, which allows for additional optimizations such as automated distribution of workload between CPU & GPU.
Using `DataModules` is recommended whenever possible!

A `DataModule` is defined by an interface:

- `prepare_data` (optional) which is called only once and on 1 GPU -- typically something like the data download step we have below
- `setup`, which is called on each GPU separately and accepts **stage** to define if we are at **fit** or **test** step
- `train_dataloader`, `val_dataloader` and `test_dataloader` to load each dataset

A `DataModule` encapsulates the five steps involved in data processing in PyTorch:

- Download / tokenize / process.
- Clean and (maybe) save to disk.
- Load inside Dataset.
- Apply transforms (rotate, tokenize, etc…).
- Wrap inside a DataLoader.

The DataModule code for the project looks like:

```python
class DataModule(pl.LightningDataModule):
    def __init__(self, model_name="google/bert_uncased_L-2_H-128_A-2", batch_size=32):
        super().__init__()

        self.batch_size = batch_size
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)

    def prepare_data(self):
        cola_dataset = load_dataset("glue", "cola")
        self.train_data = cola_dataset["train"]
        self.val_data = cola_dataset["validation"]

    def tokenize_data(self, example):
        # processing the data
        return self.tokenizer(
            example["sentence"],
            truncation=True,
            padding="max_length",
            max_length=256,
        )

    def setup(self, stage=None):
        if stage == "fit" or stage is None:
            self.train_data = self.train_data.map(self.tokenize_data, batched=True)
            self.train_data.set_format(
                type="torch", columns=["input_ids", "attention_mask", "label"]
            )

            self.val_data = self.val_data.map(self.tokenize_data, batched=True)
            self.val_data.set_format(
                type="torch", columns=["input_ids", "attention_mask", "label"]
            )

    def train_dataloader(self):
        return torch.utils.data.DataLoader(
            self.train_data, batch_size=self.batch_size, shuffle=True
        )

    def val_dataloader(self):
        return torch.utils.data.DataLoader(
            self.val_data, batch_size=self.batch_size, shuffle=False
        )
```

# 🏗️ Building a Model with Lightning

In PyTorch Lightning, models are built with [`LightningModule`](https://pytorch-lightning.readthedocs.io/en/latest/lightning_module.html), which has all the functionality of a vanilla `torch.nn.Module` (🍦) but with a few delicious cherries of added functionality on top (🍨).
These cherries are there to cut down on boilerplate and help separate out the ML engineering code from the actual machine learning.

For example, the mechanics of iterating over batches as part of an epoch are extracted away, so long as you define what happens on the `training_step`.

To make a working model out of a `LightningModule`, we need to define a new `class` and add a few methods on top.

A `LightningModule` is defined by an interface:

- `init` define the initialisations here
- `forward` what should for a given input (keep only the forward pass things here not the loss calculations / weight updates)
- `training_step` training step (loss calculation, any other metrics calculations.) No need to do weight updates
- `validation_step` validation step
- `test_step` (optional)
- `configure_optimizers` define what optimizer to use

There are a lot of other functions also which can be used. Check the [doucmentation](https://pytorch-lightning.readthedocs.io/en/latest/common/lightning_module.html) for all other methods.

The LightningModule code for the project looks like:

```python
class ColaModel(pl.LightningModule):
    def __init__(self, model_name="google/bert_uncased_L-2_H-128_A-2", lr=1e-2):
        super(ColaModel, self).__init__()
        self.save_hyperparameters()

        self.bert = AutoModel.from_pretrained(model_name)
        self.W = nn.Linear(self.bert.config.hidden_size, 2)
        self.num_classes = 2

    def forward(self, input_ids, attention_mask):
        outputs = self.bert(input_ids=input_ids, attention_mask=attention_mask)

        h_cls = outputs.last_hidden_state[:, 0]
        logits = self.W(h_cls)
        return logits

    def training_step(self, batch, batch_idx):
        logits = self.forward(batch["input_ids"], batch["attention_mask"])
        loss = F.cross_entropy(logits, batch["label"])
        self.log("train_loss", loss, prog_bar=True)
        return loss

    def validation_step(self, batch, batch_idx):
        logits = self.forward(batch["input_ids"], batch["attention_mask"])
        loss = F.cross_entropy(logits, batch["label"])
        _, preds = torch.max(logits, dim=1)
        val_acc = accuracy_score(preds.cpu(), batch["label"].cpu())
        val_acc = torch.tensor(val_acc)
        self.log("val_loss", loss, prog_bar=True)
        self.log("val_acc", val_acc, prog_bar=True)

    def configure_optimizers(self):
        return torch.optim.Adam(self.parameters(), lr=self.hparams["lr"])
```

# 👟 Training

The `DataLoader` and the `LightningModule` are brought together by a `Trainer`, which orchestrates data loading, gradient calculation, optimizer logic, and logging.

We setup `Trainer` and can customize several options, such as logging, gradient accumulation, half precision training, distributed computing, etc.

We'll stick to the basics for this example.

```python
cola_data = DataModule()
cola_model = ColaModel()

trainer = pl.Trainer(
    gpus=(1 if torch.cuda.is_available() else 0),
    max_epochs=1,
    fast_dev_run=False,
)
trainer.fit(cola_model, cola_data)

```

By enabling `fast_dev_run=True`, will run one batch of training step and one batch of validation step **(always good to do this)**. It can catch any mistakes happening the validation step right away rather than waiting for the whole training to be completed.

## 📝 Logging

`Logging` of the model training is as simple as

```python
cola_data = DataModule()
cola_model = ColaModel()

trainer = pl.Trainer(
    default_root_dir="logs",
    gpus=(1 if torch.cuda.is_available() else 0),
    max_epochs=1,
    fast_dev_run=False,
    logger=pl.loggers.TensorBoardLogger("logs/", name="cola", version=1),
)
trainer.fit(cola_model, cola_data)
```

It will create a directory called `logs/cola` if not present. You can visualise the tensorboard logs using the following command

```bash
tensorboard --logdir logs/cola
```

You can see the tensorboard at `http://localhost:6006/`

## 🔁 Callback

`Callback` is a self-contained program that can be reused across projects.

As an example, I will be implementing **ModelCheckpoint** callback. This will save the trained model. We can selectively choose which model to save by monitoring a metric.(`val_loss` in this case). The best model will be saved in the `dirpath`.

Refer to the [documentation](https://pytorch-lightning.readthedocs.io/en/latest/extensions/callbacks.html) to learn more about callbacks.

```python
cola_data = DataModule()
cola_model = ColaModel()

checkpoint_callback = ModelCheckpoint(
    dirpath="./models", monitor="val_loss", mode="min"
)

trainer = pl.Trainer(
    default_root_dir="logs",
    gpus=(1 if torch.cuda.is_available() else 0),
    max_epochs=1,
    fast_dev_run=False,
    logger=pl.loggers.TensorBoardLogger("logs/", name="cola", version=1),
    callbacks=[checkpoint_callback],
)
trainer.fit(cola_model, cola_data)
```

We can also chain multiple callbacks. `EarlyStopping` callback helps the model not to overfit by mointoring on a certain parameter (val_loss in this case).

```python

early_stopping_callback = EarlyStopping(
    monitor="val_loss", patience=3, verbose=True, mode="min"
)

trainer = pl.Trainer(
    default_root_dir="logs",
    gpus=(1 if torch.cuda.is_available() else 0),
    max_epochs=1,
    fast_dev_run=False,
    logger=pl.loggers.TensorBoardLogger("logs/", name="cola", version=1),
    callbacks=[checkpoint_callback, early_stopping_callback],
)
trainer.fit(cola_model, cola_data)
```

# 🔍 Inference

Once the model is trained, we can use the trained model to get predictions on the run time data. Typically `Inference` contains:

- Load the trained model
- Get the run time (inference) input
- Convert the input in the required format
- Get the predictions

```python
class ColaPredictor:
    def __init__(self, model_path):
        self.model_path = model_path
        # loading the trained model
        self.model = ColaModel.load_from_checkpoint(model_path)
        # keep the model in eval mode
        self.model.eval()
        self.model.freeze()
        self.processor = DataModule()
        self.softmax = torch.nn.Softmax(dim=0)
        self.lables = ["unacceptable", "acceptable"]

    def predict(self, text):
        # text => run time input
        inference_sample = {"sentence": text}
        # tokenizing the input
        processed = self.processor.tokenize_data(inference_sample)
        # predictions
        logits = self.model(
            torch.tensor([processed["input_ids"]]),
            torch.tensor([processed["attention_mask"]]),
        )
        scores = self.softmax(logits[0]).tolist()
        predictions = []
        for score, label in zip(scores, self.lables):
            predictions.append({"label": label, "score": score})
        return predictions
```

This conculdes the post. In the next post, I will be going through:

- `How to monitor model performance with Weights and Bias?`

Complete code for this post can also be found here: [Github](https://github.com/graviraja/MLOps-Basics)

---
---

**Note: The purpose of the project to explore the libraries and learn how to use them. Not to build a SOTA model.**

# Requirements:

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

# Running

### Training

After installing the requirements, in order to train the model simply run:

```
python train.py
```

### Inference

After training, update the model checkpoint path in the code and run

```
python inference.py
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
<br/>
<br/>
<br/>

# MLOps 베이직 [Week 0]: Project 설정
<br/>

## 🎬 강의 시작    
이 강의의 목표는 MLOps의 기본적인 요소들(eg. 모델 빌드, 모니터링, 설정, 테스트, 페키징, 배포, CI/CD)을 이해하는 것 입니다. 첫번째로 프로젝트를 설정해봅시다. 저자는 NLP에 관심이 많아서 NLP모델 위주로 설명이 진행되지만 기본적인 절차는 모두 비슷하게 진행됩니다. 여기서는 간단한 classification task를 가지고 진행합니다.

이 글에서는 아래와 같은 질문들을 다뤄봅니다.

- <b>`데이터를 얻는 방법은 무엇인가?`</b>
- <b>`데이터를 어떻게 처리할 것인가?`</b>
- <b>`dataloader를 정의하는 방법은 무엇인가?`</b>
- <b>`모델은 어떻게 선언하는 것인가?`</b>
- <b>`모델을 어떻게 학습하는가?`</b>
- <b>`모델 추론을 어떻게 하는 것인가?`</b>

<i>노트: 기본적인 머신러닝에 대한 이해가 필요합니다.</i>   
<br/>
<br/>

## 🛠 딥러닝 라이브러리   
딥러닝 프로젝트를 개발하기 위해서 아래와 같은 다양한 라이브러리를 활용할 수 있습니다.
- <b>[Tensorflow](https://www.tensorflow.org/)</b>
- <b>[Tensorflow Lite](https://www.tensorflow.org/lite)</b>
- <b>[Pytorch](https://pytorch.org/)</b>
- <b>[Pytorch Lightning](https://www.pytorchlightning.ai/)</b>
- etc..

저자는 여러 특성과 자동화된 코드를 활용하기 위해서 `Pytorch Lightning`을 이용합니다.
<br/>
<br/>

## 📚 데이터 셋   
여기서는 `CoLA`(Corpus of Linguistic Acceptability) 데이터 셋을 이용합니다. 주어지는 문장이 문법적으로 맞는지, 아닌지 2개의 class로 분류하는 작업을 진행해봅시다.

- ❌ `Unacceptable`: 문법적으로 맞지 않음
- ✅ `Acceptable`: 문법적으로 맞음

데이터를 다운로드 하고 로드하기 위해서 [Huggingface datasets](https://huggingface.co/docs/datasets/quicktour.html)을 사용합니다. 이 라이브러리는 800+의 데이터셋을 지원하고 커스텀 데이터도 이용할 수 있습니다.   

아래와 같이 쉽게 다운로드 할 수 있습니다.
~~~
cola_dataset = load_dataset("glue", "cola")
print(cola_dataset)
~~~
~~~
DatasetDict({
    train: Dataset({
        features: ['sentence', 'label', 'idx'],
        num_rows: 8551
    })
    validation: Dataset({
        features: ['sentence', 'label', 'idx'],
        num_rows: 1043
    })
    test: Dataset({
        features: ['sentence', 'label', 'idx'],
        num_rows: 1063
    })
})
~~~

데이터를 한번 살펴봅니다.
~~~
train_dataset = cola_dataset['train']
print(train_dataset[0])
~~~
~~~
{
    'idx': 0,
    'label': 1,
    'sentence': "Our friends won't buy this analysis, let alone the next one we propose."
}
~~~
<br/>
<br/>

## 🛒 데이터 불러오기   
데이터 파이프라인은 아래의 도구를 이용해서 생성할 수 있습니다.   

- 🍦 Vanilla Pytorch [DataLoaders](https://pytorch.org/tutorials/beginner/basics/data_tutorial.html)
- ⚡ Pytorch Lightning [DataModules](https://pytorch-lightning.readthedocs.io/en/latest/extensions/datamodules.html)   
  
`DataModules`가 CPU & GPU에 더 최적화 되어있고 구조화가 잘 되어 있습니다. 가능하다면 `DataModules`을 이용하는 것을 추천합니다.   

`DataModules`은 다음과 같은 interface에 의해 정의됩니다.   
- `prepare_data` (optional) 는 1개의 GPU에서 한번만 호출됩니다. -- 일반적으로 아래에서 설명할 데이터 다운로드하는 것과 같은 작업이 포함될 수 있습니다.   
- `setup`은 각각 GPU에서 호출되며 **fit**또는 **test**단계에 있는지 정의하기 위해서 **stage**를 이용합니다.
- 각 데이터셋을 불러오기 위해서 `train_dataloader`, `val_dataloader` and `test_dataloader`를 이용합니다.

`DataModule`은 PyTorch에서 데이터 처리에 관여하는 5가지 스텝으로 캡슐화 되어 있습니다.
- 전처리 단계(다운로드/ 토큰화/ process)
- 정리하거나 디스크에 저장하기
- Dataset으로 불러오기
- transforms 적용(회전, 토큰화, 기타…)
- DataLoader로 감싸기

프로젝트에서 사용하는 `DataModule`코드는 다음과 같습니다.
~~~
class DataModule(pl.LightningDataModule):
    def __init__(self, model_name="google/bert_uncased_L-2_H-128_A-2", batch_size=32):
        super().__init__()

        self.batch_size = batch_size
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)

    def prepare_data(self):
        cola_dataset = load_dataset("glue", "cola")
        self.train_data = cola_dataset["train"]
        self.val_data = cola_dataset["validation"]

    def tokenize_data(self, example):
        # processing the data
        return self.tokenizer(
            example["sentence"],
            truncation=True,
            padding="max_length",
            max_length=256,
        )

    def setup(self, stage=None):
        if stage == "fit" or stage is None:
            self.train_data = self.train_data.map(self.tokenize_data, batched=True)
            self.train_data.set_format(
                type="torch", columns=["input_ids", "attention_mask", "label"]
            )

            self.val_data = self.val_data.map(self.tokenize_data, batched=True)
            self.val_data.set_format(
                type="torch", columns=["input_ids", "attention_mask", "label"]
            )

    def train_dataloader(self):
        return torch.utils.data.DataLoader(
            self.train_data, batch_size=self.batch_size, shuffle=True
        )

    def val_dataloader(self):
        return torch.utils.data.DataLoader(
            self.val_data, batch_size=self.batch_size, shuffle=False
        )
~~~
<br/>
<br/>

## 🏗️ Lightning을 이용한 모델 구성    
PyTorch Lightning에서 모델은 [LightningModule](https://pytorch-lightning.readthedocs.io/en/latest/common/lightning_module.html)로 구성됩니다. 기본적으로 `torch.nn.Module`(🍦)위에 다양한 function들(🍨)이 올라가 있는 구조 입니다(바닐라 아이스크림 위에 체리!).
이 체리들은 반복되는 코드를 줄일 수 있고 엔지니어링 코드를 머신러닝 코드와 분리할 수 있도록 도와줍니다.

예를들면, 한 epoch에 반복되는 batch에서 어떠한 동작이 이루어지는지를 `training_step`을 이용해서 정의할 수 있습니다.   

모델을 `LightningModule`로 동작하게 하려면 새로운 `class`를 정의하고 몇가지 메서드를 추가해야 합니다.   

`LightningModule`은 다음과 같은 interface에 의해 정의됩니다.
- `init` 모듈의 초기 설정을 정의합니다.
- `forward` 주어지는 입력에 대해서 어떠한 동작을 할지 정의합니다(loss 계산, weight 업데이트는 제외합니다.).
- `training_step` training step으로 loss 계산이나 metric 계산을 수행합니다. weight업데이트는 필요하지 않습니다.
- `validation_step` validation step
- `test_step` test step (optional)
- `configure_optimizers` 어떤 optimizer를 이용할지 설정합니다.   

이외에도 다양한 기능들을 이용할 수 있습니다. 자세한 정보는 [여기](https://pytorch-lightning.readthedocs.io/en/latest/common/lightning_module.html)를 확인하세요.   

이 프로젝트에서는 다음과 같이 `LightningModule`을 이용합니다.
~~~
class ColaModel(pl.LightningModule):
    def __init__(self, model_name="google/bert_uncased_L-2_H-128_A-2", lr=1e-2):
        super(ColaModel, self).__init__()
        self.save_hyperparameters()

        self.bert = AutoModel.from_pretrained(model_name)
        self.W = nn.Linear(self.bert.config.hidden_size, 2)
        self.num_classes = 2

    def forward(self, input_ids, attention_mask):
        outputs = self.bert(input_ids=input_ids, attention_mask=attention_mask)

        h_cls = outputs.last_hidden_state[:, 0]
        logits = self.W(h_cls)
        return logits

    def training_step(self, batch, batch_idx):
        logits = self.forward(batch["input_ids"], batch["attention_mask"])
        loss = F.cross_entropy(logits, batch["label"])
        self.log("train_loss", loss, prog_bar=True)
        return loss

    def validation_step(self, batch, batch_idx):
        logits = self.forward(batch["input_ids"], batch["attention_mask"])
        loss = F.cross_entropy(logits, batch["label"])
        _, preds = torch.max(logits, dim=1)
        val_acc = accuracy_score(preds.cpu(), batch["label"].cpu())
        val_acc = torch.tensor(val_acc)
        self.log("val_loss", loss, prog_bar=True)
        self.log("val_acc", val_acc, prog_bar=True)

    def configure_optimizers(self):
        return torch.optim.Adam(self.parameters(), lr=self.hparams["lr"])
~~~   
<br/>
<br/>

## 👟 학습   
`DataLoader`, `LightningModule`는 `Trainer`의해 사용됩니다. `Trainer`는 데이터 로드, gradient 계산, optimizer 로직, 로깅등을 조율해서 처리합니다.   

`Trainer`는 여러가지 옵션들로 로깅, 그라디언트 축적, half precision training, 분산 컴퓨팅 등과 같이 커스텀 할 수 있습니다.   

여기서는 기본적인 예제를 이용하겠습니다.
~~~
cola_data = DataModule()
cola_model = ColaModel()

trainer = pl.Trainer(
    gpus=(1 if torch.cuda.is_available() else 0),
    max_epochs=1,
    fast_dev_run=False,
)
trainer.fit(cola_model, cola_data)
~~~

`fast_dev_run=True`로 설정하면 training 1스텝,validation 1step으로 진행합니다(True로 설정하는 것이 대부분 좋습니다. validation 스텝에서 잘못된 부분이 바로 발생하기 떄문에 학습이 완료될때까지 기다릴 필요가 없습니다.).   
<br/>
<br/>

## 📝 로깅   
모델 학습을 로깅하는 것은 다음과 같이 간단합니다.
~~~
cola_data = DataModule()
cola_model = ColaModel()

trainer = pl.Trainer(
    default_root_dir="logs",
    gpus=(1 if torch.cuda.is_available() else 0),
    max_epochs=1,
    fast_dev_run=False,
    logger=pl.loggers.TensorBoardLogger("logs/", name="cola", version=1),
)
trainer.fit(cola_model, cola_data)
~~~

`logs/cola`디렉터리가 생성되고 아래의 커맨드를 이용해서 tensorboard로 시각화할 수 있습니다.
~~~
tensorboard --logdir logs/cola
~~~
텐서보드는 `http://localhost:6006/`로 접근할 수 있습니다.
<br/>
<br/>

## 🔁 Callback   
`Callback`은 프로젝트에 전반적으로 재활용될 수 있는 내장된 프로그램입니다.

예를들어, **ModelCheckpoint** callback을구현한다고 합니다. 이 callback은 학습된 모델을 저장합니다.   
metric을 모니터링해서 어떤 모델을 저장할지 선택할 수 있습니다(여기서는 `val_loss`를 이용합니다.). 가장 좋은 모델은 `dirpath`에 저장됩니다.   

Callback에 대한 자세한 정보는 [여기](https://pytorch-lightning.readthedocs.io/en/latest/extensions/callbacks.html)를 참고해주세요.   
~~~
cola_data = DataModule()
cola_model = ColaModel()

checkpoint_callback = ModelCheckpoint(
    dirpath="./models", monitor="val_loss", mode="min"
)

trainer = pl.Trainer(
    default_root_dir="logs",
    gpus=(1 if torch.cuda.is_available() else 0),
    max_epochs=1,
    fast_dev_run=False,
    logger=pl.loggers.TensorBoardLogger("logs/", name="cola", version=1),
    callbacks=[checkpoint_callback],
)
trainer.fit(cola_model, cola_data)
~~~   

또한 callback을 여러개 엮을 수 있습니다. `EarlyStopping`callback은 특정 파라미터(여기서는 `val_loss`를 이용합니다.)를 모니터링하면서 모델의 overfit을 방지합니다.   
~~~
early_stopping_callback = EarlyStopping(
    monitor="val_loss", patience=3, verbose=True, mode="min"
)

trainer = pl.Trainer(
    default_root_dir="logs",
    gpus=(1 if torch.cuda.is_available() else 0),
    max_epochs=1,
    fast_dev_run=False,
    logger=pl.loggers.TensorBoardLogger("logs/", name="cola", version=1),
    callbacks=[checkpoint_callback, early_stopping_callback],
)
trainer.fit(cola_model, cola_data)
~~~   
<br/>
<br/>

## 🔍 추론   
모델이 학습되고나면 예측을위해서 학습된 모델을 이용할 수 있습니다.   
일반적으로 `Inference`는 다음과 같은 과정을 포함합니다.
- 학습된 모델 로드
- 런타임 입력 얻기
- 입력을 알맞은 포멧으로 변경하기
- 예측값 얻기
~~~
class ColaPredictor:
    def __init__(self, model_path):
        self.model_path = model_path
        # loading the trained model
        self.model = ColaModel.load_from_checkpoint(model_path)
        # keep the model in eval mode
        self.model.eval()
        self.model.freeze()
        self.processor = DataModule()
        self.softmax = torch.nn.Softmax(dim=0)
        self.lables = ["unacceptable", "acceptable"]

    def predict(self, text):
        # text => run time input
        inference_sample = {"sentence": text}
        # tokenizing the input
        processed = self.processor.tokenize_data(inference_sample)
        # predictions
        logits = self.model(
            torch.tensor([processed["input_ids"]]),
            torch.tensor([processed["attention_mask"]]),
        )
        scores = self.softmax(logits[0]).tolist()
        predictions = []
        for score, label in zip(scores, self.lables):
            predictions.append({"label": label, "score": score})
        return predictions
~~~   

Complete code for this post can also be found here: [Github](https://github.com/graviraja/MLOps-Basics)
