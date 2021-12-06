# MLOps-기본

 > There is nothing magic about magic. The magician merely understands something simple which doesn’t appear to be simple or natural to the untrained audience. Once you learn how to hold a card while making your hand look empty, you only need practice before you, too, can “do magic.” – Jeffrey Friedl, 서적 Mastering Regular Expressions

**Note: 제안, 수정 또는 피드백이 있는 경우 Issue를 올려주세요.**

MLOps-Basics 시리즈의 목표는 모델의 `구축(building)`, `모니터링(monitoring)`, `구성(configurations)`, `테스트(testing)`, `패키징(packaging)`, `배포(deployment)`, `CI/CD`와 같은 MLOps의 기본을 이해하는 것입니다.

![pl](images/summary.png)

## 0주차: Project 준비

<img src="https://img.shields.io/static/v1.svg?style=for-the-badge&label=difficulty&message=easy&color=green"/>

자세한 내용은 [블로그 포스트](https://www.ravirajag.dev/blog/mlops-project-setup-part1)를 참고해주세요.

이 프로젝트에서는 간단한 classification 문제를 다루고 있습니다. 이번 주차에서는 아래의 질문에 답할 수 있는 범위를 다루게 됩니다:

- `데이터는 어떻게 구할까?`
- `데이터를 어떻게 처리해야 할끼?`
- `데이터 로더(dataloader)를 어떻게 정의 해야 할까?`
- `모델은 어떻게 정의할까?`
- `모델을 어떻게 학습할까?`
- `추론은 어떻게 해야하나?`

![pl](images/pl.jpeg)

이 프로젝트를 위해서 아래의 내용을(tech stack)숙지하고 있어야 합니다:

- [Huggingface Datasets](https://github.com/huggingface/datasets)
- [Huggingface Transformers](https://github.com/huggingface/transformers)
- [Pytorch Lightning](https://pytorch-lightning.readthedocs.io/)


## 1주차: 모델 모니터링 - 가중치(Weights)와 바이어스(Biases)

<img src="https://img.shields.io/static/v1.svg?style=for-the-badge&label=difficulty&message=easy&color=green"/>

자세한 내용은 [블로그 포스트](https://www.ravirajag.dev/blog/mlops-wandb-integration)를 참고해주세요.   

하이퍼 파라미터(hyper-parameters)를 수정하고 성능 테스트를 위해서 다른 모델을 사용하는 것 그리고 모델과 입력 데이터의 관계를 살펴보는 것과 같이 모든 상황을 추적하는 것은 더 나은 모델을 설계할 수 있도록 합니다.

이번 주차에서는 아래의 질문에 답할 수 있는 범위를 다루게 됩니다:

- `가중치(W)와 바이어스(B)로 기본적인 로깅(logging)을 어떻게 구성할까?`
- `어떻게 매트릭스를 연산하고 W와 B로서 기록할 수 있을까?`
- `W와 B를 어떻게 그래프로 나타낼 수 있을까?`
- `어떻게 데이터를 W와 B에 녹여낼 수 있을까?`

![wannb](images/wandb.png)

이 프로젝트를 위해서 아래의 내용을(tech stack)숙지하고 있어야 합니다:

- [Weights and Biases](https://wandb.ai/site)
- [torchmetrics](https://torchmetrics.readthedocs.io/)

References:

- [Tutorial on Pytorch Lightning + Weights & Bias](https://www.youtube.com/watch?v=hUXQm46TAKc)
- [WandB Documentation](https://docs.wandb.ai/)

## 2주차: 구성(Configurations) - Hydra

<img src="https://img.shields.io/static/v1.svg?style=for-the-badge&label=difficulty&message=easy&color=green"/>

자세한 내용은 [블로그 포스트](https://www.ravirajag.dev/blog/mlops-hydra-config)를 참고해주세요.   

구성 관리(Configuration management)는 복잡한 소프트웨어 시스템을 관리하는 데 필요합니다. Configuration management가 부족하면 안정성, 가동 시간, 시스템 확장 기능에 심각한 문제가 발생할 수 있습니다.

이번 주차에서는 아래와 같은 범위를 다루게 됩니다:

- `Hydra의 기본`
- `구성의 재정의(Overridding configurations)`
- `다양한 파일에 configuration을 분할하는 방법`
- `변수 인터폴레이션(Variable Interpolation)`
- `다른 파라미터 조합으로 어떻게 모델을 학슬할까?`

![hydra](images/hydra.png)

이 프로젝트를 위해서 아래의 내용을(tech stack)숙지하고 있어야 합니다:

- [Hydra](https://hydra.cc/)

References

- [Hydra Documentation](https://hydra.cc/docs/intro)
- [Simone Tutorial on Hydra](https://www.sscardapane.it/tutorials/hydra-tutorial/#executing-multiple-runs)


## Week 3: Data Version Control - DVC

<img src="https://img.shields.io/static/v1.svg?style=for-the-badge&label=difficulty&message=easy&color=green"/>

자세한 내용은 [블로그 포스트](https://www.ravirajag.dev/blog/mlops-dvc)를 참고해주세요.

전형적인 버전 컨트롤 시스템은 큰 파일들을 다룰 수 있도록 설계되어 있지 않습니다. 따라서 이러한 시스템은 기록을 복제하고 저장하는 것을 실용적이지 못하게 만듭니다. 머신러닝에서는 이러한 일이 다반사 입니다.

이번 주차에서는 아래와 같은 범위를 다루게 됩니다:

- `DVC의 기본`
- `DVC 초기화`
- `리모트 저장소를 구성하는 방법`
- `리모트 저장소에 모델을 저장하는 방법`
- `모델의 버전 관리`

![dvc](images/dvc.png)

이 프로젝트를 위해서 아래의 내용을(tech stack)숙지하고 있어야 합니다:

- [DVC](https://dvc.org/)

References

- [DVC Documentation](https://dvc.org/doc)
- [DVC Tutorial on Versioning data](https://www.youtube.com/watch?v=kLKBcPonMYw)

## 4주차: 모델 패킹(packing) - ONNX

<img src="https://img.shields.io/static/v1.svg?style=for-the-badge&label=difficulty&message=medium&color=orange"/>

자세한 내용은 [블로그 포스트](https://www.ravirajag.dev/blog/mlops-onnx)를 참고해주세요.

왜 모델 패킹이 필요할까요? 모델은 다양한 머신러닝 프레임워크(sklearn, tensorflow, pytorch, 기타 등등)를 통해서 만들어 질 수 있습니다. 이러한 모델들을 모바일, 웹, 라즈베리파이와 같은 다양한 환경에 배포하고 싶고 파이토치로 학습하고 텐서플로우로 추론하는 것과 같이 다양한 프레임워크를 이용하고 싶을 수도 있습니다.   
이처럼 AI 개발자가 다양한 프레임워크, 도구, 런타임 및 컴파일러와 함께 모델을 사용할 수 있도록 하는 공통 파일 포멧은 많은 도움이 될 수 있습니다.

커뮤니티 프로젝트인 `ONNX`를 이용하면 앞서 언급한 목적들을 달성할 수 있습니다.

이번 주차에서는 아래와 같은 범위를 다루게 됩니다:

- `ONNX란?`

- `학습된 모델을 ONNX 포멧으로 어떻게 변환할까?`

- `ONNX Runtime이란?`

- `변환된 ONNX 모델을 ONNX Runtime에서 구동하는 방법은?`

- `비교`

![ONNX](images/onnx.jpeg)

이 프로젝트를 위해서 아래의 내용을(tech stack)숙지하고 있어야 합니다:

- [ONNX](https://onnx.ai/)
- [ONNXRuntime](https://www.onnxruntime.ai/)

References

- [Abhishek Thakur tutorial on onnx model conversion](https://www.youtube.com/watch?v=7nutT3Aacyw)
- [Pytorch Lightning documentation on onnx conversion](https://pytorch-lightning.readthedocs.io/en/stable/common/production_inference.html)
- [Huggingface Blog on ONNXRuntime](https://medium.com/microsoftazure/accelerate-your-nlp-pipelines-using-hugging-face-transformers-and-onnx-runtime-2443578f4333)
- [Piotr Blog on onnx conversion](https://tugot17.github.io/data-science-blog/onnx/tutorial/2020/09/21/Exporting-lightning-model-to-onnx.html)


## 5주차: 모델 패킹(packaging) - 도커(docker)

<img src="https://img.shields.io/static/v1.svg?style=for-the-badge&label=difficulty&message=easy&color=green"/>

자세한 내용은 [블로그 포스트](https://www.ravirajag.dev/blog/mlops-docker)를 참고해주세요.

모델 패킹이 왜 필요할까요? 어플리케이션을 다른 누군가에게 공유해줘야 할 수도 있고, 이러한 경우 많은 상황에서 어플리케이션은 의존성 문제나 OS관련 문제로 돌아가지 않습니다. 그래서 많은 경우 다음과 같은 말을 남겨둡니다. `이 프로젝트는 내 OO랩탑, OO시스템에서 테스트 되었습니다.`

따라서 어플리케이션을 실행하기 위해서는 실제 동작했던 환경과 동일한 환경을 구성해야 합니다. 결국 동일한 환경을 구성하기 위해서는 수동으로 많은 것들을 설정 해야하고 많은 컴포넌트를 설치해야 합니다. (가끔은 이러한 환경 문제가 더 안풀리기도 하죠ㅠ)

이러한 한계를 극복할 수 있는 방법을 컨테이너(Containers)기술 이라고 합니다.

어플리케이션을 컨테이너화/패키징 함으로써 어떠한 클라우드 플랫폼에서도 어플리케이션을 실행할 수 있고 관리형 서비스(managed services), 오토스케일링(autoscaling), 안정성과 같은 다양한 이점을 얻을 수 있습니다.
이러한 작업을 위해서 가장 많이 찾는 툴이 바로 Docker🛳 입니다. 

이번 주차에서는 아래와 같은 범위를 다루게 됩니다:

- `FastAPI wrapper`
- `Docker 기본`
- `Docker Container 빌드하기`
- `Docker Compose`

![Docker](images/docker_flow.png)

References

- [Analytics vidhya blog](https://www.analyticsvidhya.com/blog/2021/06/a-hands-on-guide-to-containerized-your-machine-learning-workflow-with-docker/)


## Week 6: CI/CD - GitHub Actions

<img src="https://img.shields.io/static/v1.svg?style=for-the-badge&label=difficulty&message=medium&color=orange"/>

Refer to the [Blog Post here](https://www.ravirajag.dev/blog/mlops-github-actions)

CI/CD is a coding philosophy and set of practices with which you can continuously build, test, and deploy iterative code changes.

This iterative process helps reduce the chance that you develop new code based on a buggy or failed previous versions. With this method, you strive to have less human intervention or even no intervention at all, from the development of new code until its deployment.

In this post, I will be going through the following topics:

- Basics of GitHub Actions
- First GitHub Action
- Creating Google Service Account
- Giving access to Service account
- Configuring DVC to use Google Service account
- Configuring Github Action

![Docker](images/basic_flow.png)

References

- [Configuring service account](https://dvc.org/doc/user-guide/setup-google-drive-remote)

- [Github actions](https://docs.github.com/en/actions/quickstart)


## Week 7: Container Registry - AWS ECR

<img src="https://img.shields.io/static/v1.svg?style=for-the-badge&label=difficulty&message=medium&color=orange"/>

Refer to the [Blog Post here](https://www.ravirajag.dev/blog/mlops-container-registry)

A container registry is a place to store container images. A container image is a file comprised of multiple layers which can execute applications in a single instance. Hosting all the images in one stored location allows users to commit, identify and pull images when needed.

Amazon Simple Storage Service (S3) is a storage for the internet. It is designed for large-capacity, low-cost storage provision across multiple geographical regions.

In this week, I will be going through the following topics:

- `Basics of S3`

- `Programmatic access to S3`

- `Configuring AWS S3 as remote storage in DVC`

- `Basics of ECR`

- `Configuring GitHub Actions to use S3, ECR`

![Docker](images/ecr_flow.png)


## Week 8: Serverless Deployment - AWS Lambda

<img src="https://img.shields.io/static/v1.svg?style=for-the-badge&label=difficulty&message=medium&color=orange"/>

Refer to the [Blog Post here](https://www.ravirajag.dev/blog/mlops-serverless)

A serverless architecture is a way to build and run applications and services without having to manage infrastructure. The application still runs on servers, but all the server management is done by third party service (AWS). We no longer have to provision, scale, and maintain servers to run the applications. By using a serverless architecture, developers can focus on their core product instead of worrying about managing and operating servers or runtimes, either in the cloud or on-premises.

In this week, I will be going through the following topics:

- `Basics of Serverless`

- `Basics of AWS Lambda`

- `Triggering Lambda with API Gateway`

- `Deploying Container using Lambda`

- `Automating deployment to Lambda using Github Actions`

![Docker](images/lambda_flow.png)


## Week 9: Prediction Monitoring - Kibana

<img src="https://img.shields.io/static/v1.svg?style=for-the-badge&label=difficulty&message=medium&color=orange"/>

Refer to the [Blog Post here](https://www.ravirajag.dev/blog/mlops-monitoring)


Monitoring systems can help give us confidence that our systems are running smoothly and, in the event of a system failure, can quickly provide appropriate context when diagnosing the root cause.

Things we want to monitor during and training and inference are different. During training we are concered about whether the loss is decreasing or not, whether the model is overfitting, etc.

But, during inference, We like to have confidence that our model is making correct predictions.

There are many reasons why a model can fail to make useful predictions:

- The underlying data distribution has shifted over time and the model has gone stale. i.e inference data characteristics is different from the data characteristics used to train the model.

- The inference data stream contains edge cases (not seen during model training). In this scenarios model might perform poorly or can lead to errors.

- The model was misconfigured in its production deployment. (Configuration issues are common)

In all of these scenarios, the model could still make a `successful` prediction from a service perspective, but the predictions will likely not be useful. Monitoring machine learning models can help us detect such scenarios and intervene (e.g. trigger a model retraining/deployment pipeline).

In this week, I will be going through the following topics:

- `Basics of Cloudwatch Logs`

- `Creating Elastic Search Cluster`

- `Configuring Cloudwatch Logs with Elastic Search`

- `Creating Index Patterns in Kibana`

- `Creating Kibana Visualisations`

- `Creating Kibana Dashboard`

![Docker](images/kibana_flow.png)
