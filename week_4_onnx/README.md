# 📦 Model Packaging

Why do we need model packaging? Models can be built using any machine learning framework available out there (sklearn, tensorflow, pytorch, etc.). We might want to deploy models in different environments like (mobile, web, raspberry pi) or want to run in a different framework (trained in pytorch, inference in tensorflow).
A common file format to enable AI developers to use models with a variety of frameworks, tools, runtimes, and compilers will help a lot.

This is achieved by a community project `ONNX`.

In this post, I will be going through:

- `What is ONNX?`

- `How to convert a trained model to ONNX format?`

- `What is ONNX Runtime?`

- `How to run ONNX converted model in ONNX Runtime?`

- `Comparisons`

## What is ONNX?

ONNX is an open format built to represent machine learning models.

ONNX defines a common set of operators - the building blocks of machine learning and deep learning models - and a common file format to enable AI developers to use models with a variety of frameworks, tools, runtimes, and compilers.

![onnx](/static/images/onnx/onnx_1.jpeg)

The ONNX format is the basis of an open ecosystem that makes AI more accessible and valuable to all:

- developers can choose the right framework for their task
- framework authors can focus on innovative enhancements
- hardware vendors can streamline optimizations for neural network computations.

![onnx](/static/images/onnx/onnx_2.png)

Thus, ONNX is an open file format to store (trained) machine learning models/pipelines containing sufficient detail (regarding data types etc.) to move from one platform to another.

Models in ONNX format can be easily deployed to various cloud platforms as well as to IoT devices.

## ⏳ How to convert a trained model to ONNX format?

<div>
  Since we are using <code>Pytorch Lightning</code> ⚡️ which is a wrapper around{' '}
  <code>Vanilla Pytorch</code> 🍦, there are two ways to convert the model into
  <div style={{ display: 'inline-block' }}>
    <img
      alt="ocean"
      src="/static/images/onnx/logo.png"
      style={{
        width: 18 + 'px',
        height: 18 + 'px',
        display: 'inline-block',
        margin: 0,
        marginLeft: 5 + 'px',
        marginRight: 5 + 'px',
      }}
    />
  </div>
  format.
</div>

- Using `onnx.export` method in 🍦

- Using `to_onnx` method in ⚡️

### Exporting using model using 🍦 Pytorch

In order to convert the model into ONNX format, we need to specify some things:

#### Trained model which needs to be converted

```python
model_path = f"{root_dir}/models/best-checkpoint.ckpt"
cola_model = ColaModel.load_from_checkpoint(model_path)
```

#### Sample input format (which the `forward` method takes with batch_size as 1)

```python
input_batch = next(iter(data_model.train_dataloader()))
input_sample = {
    "input_ids": input_batch["input_ids"][0].unsqueeze(0),
    "attention_mask": input_batch["attention_mask"][0].unsqueeze(0),
}
```

- Input names, Output names
- Dynamic axes (batch size dimension)

Complete code looks like:

```python
torch.onnx.export(
    cola_model,  # model being run
    (
        input_sample["input_ids"],
        input_sample["attention_mask"],
    ),  # model input (or a tuple for multiple inputs)
    f"{root_dir}/models/model.onnx",  # where to save the model
    export_params=True,
    opset_version=10,
    input_names=["input_ids", "attention_mask"],  # the model's input names
    output_names=["output"],  # the model's output names
    dynamic_axes={            # variable length axes
        "input_ids": {0: "batch_size"},
        "attention_mask": {0: "batch_size"},
        "output": {0: "batch_size"},
    },
)
```

### Exporting using model using ⚡️ Pytorch Lightning

Converting model to onnx with multi-input is not added yet. [See the issue here](https://github.com/PyTorchLightning/pytorch-lightning/pull/7458)

Let's see when the input is a single tensor.

⚡️ Module class comes with an in-built method `to_onnx`. Call that method with the necessary parameters:

- Name of the onnx model
- Input sample
- Input names
- Output names
- Dynamic axes

The code looks like:

```python
model.to_onnx(
  "model.onnx",             # where to save the model
  input_sample,             # input samples with atleast batch size as 1
  export_params=True,
  opset_version=10,
  input_names = ['input'],    # Input names
  output_names = ['output'],  # Output names
  dynamic_axes={              # variable length axes
    'input' : {0 : 'batch_size'},
    'output' : {0 : 'batch_size'},
  },
)
```

<div>
  Now that the model is converted into
  <div style={{ display: 'inline-block' }}>
    <img
      alt="ocean"
      src="/static/images/onnx/logo.png"
      style={{
        width: 18 + 'px',
        height: 18 + 'px',
        display: 'inline-block',
        margin: 0,
        marginLeft: 5 + 'px',
        marginRight: 5 + 'px',
      }}
    />
  </div>
  format, Let's load it to run the inference.
</div>

## 👟 What is ONNX Runtime?

ONNX Runtime is a performance-focused inference engine for ONNX models.

ONNX Runtime was designed with a focus on performance and scalability in order to support heavy workloads in high-scale production scenarios. It also has extensibility options for compatibility with emerging hardware developments.

![onnx](/static/images/onnx/onnx_runtime.png)

#### ⚙️ Installation

Install onnxruntime using the following command:

```shell
pip install onnxruntime
```

ONNX Runtime is supported on different Operating System (OS) and hardware (HW) platforms. The Execution Provider (EP) interface in ONNX Runtime enables easy integration with different HW accelerators.

Check all the providers of ONNXRuntime using the command

```python
from onnxruntime import  get_all_providers
print(get_all_providers())
```

Sample output looks like:

```shell
[
  'TensorrtExecutionProvider',
  'CUDAExecutionProvider',
  'MIGraphXExecutionProvider',
  'ROCMExecutionProvider',
  'OpenVINOExecutionProvider',
  'DnnlExecutionProvider',
  'NupharExecutionProvider',
  'VitisAIExecutionProvider',
  'NnapiExecutionProvider',
  'ArmNNExecutionProvider',
  'ACLExecutionProvider',
  'DmlExecutionProvider',
  'RknpuExecutionProvider',
  'CPUExecutionProvider'
]
```

## 🏃‍♂️ How to run ONNX converted model in ONNX Runtime?

<div>
  In order to load the
  <div style={{ display: 'inline-block' }}>
    <img
      alt="ocean"
      src="/static/images/onnx/logo.png"
      style={{
        width: 18 + 'px',
        height: 18 + 'px',
        display: 'inline-block',
        margin: 0,
        marginLeft: 5 + 'px',
        marginRight: 5 + 'px',
      }}
    />
  </div>
  model, certain things needs to be done:
</div>

#### Create Inference Session which will load the onnx model

```python
import onnxruntime as ort
ort_session = ort.InferenceSession(onnx_model_path)
```

#### Prepare the inputs for the session

The input names should match the names used while creating the onnx model.

```python
ort_inputs = {
    "input_ids": np.expand_dims(processed["input_ids"], axis=0),
    "attention_mask": np.expand_dims(processed["attention_mask"], axis=0),
}
```

#### Run the session

Run the inference session with the inputs

```python
ort_output = ort_session.run(None, ort_inputs)
```

None will return all the outputs. If the model return multiple outputs, specifying the output name here will return only that output

Complete code looks like:

```python
class ColaPredictor:
    def __init__(self, model_path):
        # creating the onnxruntime session
        self.ort_session = ort.InferenceSession(model_path)
        self.processor = DataModule()
        self.lables = ["unacceptable", "acceptable"]

    def predict(self, text):
        inference_sample = {"sentence": text}
        processed = self.processor.tokenize_data(inference_sample)
        # Preparing inputs
        ort_inputs = {
            "input_ids": np.expand_dims(processed["input_ids"], axis=0),
            "attention_mask": np.expand_dims(processed["attention_mask"], axis=0),
        }
        # Run the model (None = get all the outputs)
        ort_outs = self.ort_session.run(None, ort_inputs)

        # Normalising the outputs
        scores = softmax(ort_outs[0])[0]
        predictions = []
        for score, label in zip(scores, self.lables):
            predictions.append({"label": label, "score": score})
        return predictions
```

This is the `python` api example for onnxruntime. For other language support refer to the [documentation here](https://www.onnxruntime.ai/docs/tutorials/inferencing/api-basics.html)

## ⏲ Comparisons

Let's compare the response time for both methods (standard pytorch inference, onnxruntime inference)

`Experiment`: Running a sample of 10 sentences after a initial warmp-up(loading the model and running inference on 1 sentence)

#### Inference times of Pytorch Model

```shell
function:'predict' took: 0.00427 sec
function:'predict' took: 0.00420 sec
function:'predict' took: 0.00437 sec
function:'predict' took: 0.00587 sec
function:'predict' took: 0.00531 sec
function:'predict' took: 0.00504 sec
function:'predict' took: 0.00658 sec
function:'predict' took: 0.00491 sec
function:'predict' took: 0.00520 sec
function:'predict' took: 0.00476 sec
```

#### Inference times of ONNX format model

```shell
function:'predict' took: 0.00144 sec
function:'predict' took: 0.00128 sec
function:'predict' took: 0.00132 sec
function:'predict' took: 0.00136 sec
function:'predict' took: 0.00134 sec
function:'predict' took: 0.00132 sec
function:'predict' took: 0.00144 sec
function:'predict' took: 0.00132 sec
function:'predict' took: 0.00172 sec
function:'predict' took: 0.00187 sec
```

As it is visible from the logs there is an improvment of `2-3x` using ONNX + ONNXRuntime formant for inference compared to standard pytorch inference.

## 🔚

<div>
  This concludes the post. These are only a few capabilities of
  <div style={{ display: 'inline-block' }}>
    <img
      alt="ocean"
      src="/static/images/onnx/logo.png"
      style={{
        width: 18 + 'px',
        height: 18 + 'px',
        display: 'inline-block',
        margin: 0,
        marginLeft: 5 + 'px',
        marginRight: 5 + 'px',
      }}
    />
  </div>
  format.
</div>

I have used pytorch only. ONNX supports various frameworks. Look into the [documentation here](https://onnx.ai/supported-tools.html#buildModel) for more information

Complete code for this post can also be found here: [Github](https://github.com/graviraja/MLOps-Basics)

## References

- [Abhishek Thakur tutorial on onnx model conversion](https://www.youtube.com/watch?v=7nutT3Aacyw)
- [Pytorch Lightning documentation on onnx conversion](https://pytorch-lightning.readthedocs.io/en/stable/common/production_inference.html)
- [Huggingface Blog on ONNXRuntime](https://medium.com/microsoftazure/accelerate-your-nlp-pipelines-using-hugging-face-transformers-and-onnx-runtime-2443578f4333)
- [Piotr Blog on onnx conversion](https://tugot17.github.io/data-science-blog/onnx/tutorial/2020/09/21/Exporting-lightning-model-to-onnx.html)


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


### Running notebooks

I am using [Jupyter lab](https://jupyter.org/install) to run the notebooks.

Since I am using a virtualenv, when I run the command `jupyter lab` it might or might not use the virtualenv.

To make sure to use the virutalenv, run the following commands before running `jupyter lab`

```
conda install ipykernel
python -m ipykernel install --user --name project-setup
pip install ipywidgets
```
