# Train CNN with PyTorch + Fastai and deploy on AWS Lambda

The idea of this repo is to take the binary output from [this Lambda function](<https://github.com/nicolasmetallo/serverless-ssd300-vgg>), where a person is cropped out of the background, and then classify that as belonging to either class A or B. I'm going to first create a dataset using images downloaded from Google and the Open Images Dataset, and then train a CNN (convolutional neural network) with Fastai and PyTorch. Once that's done I'm going to convert my model `Torch Script`, upload those weights to S3, and deploy the whole thing as a Lambda function. The input will be binary data (image) and the output will be a JSON.

## Pre-requisites

You should have:

- Docker CE
- PyTorch 1.0 (<https://pytorch.org/>)
- Fastai (<https://docs.fast.ai/>)
- Amazon AWS account
- AWS CLI + AWS SAM CLI

## Steps

The first step is to clone this repo

```
git clone https://github.com/nicolasmetallo/serverless-cnn-pytorch/
```

Now open `train_deploy_pytorch_serverless.ipynb` in Google Colab and follow the steps to create a training dataset that consist of classes "A" and "B" in order to create a simple binary image classification model. We are going to use the [**google-images-download**](https://github.com/hardikvasa/google-images-download) tool to download images from the class we want to recognise (i.e. "A") and the [**Open Images Dataset**](https://storage.googleapis.com/openimages/web/index.html) to get the other class. Running the notebook won't take more than 30 minutes and you will end up with a `model.tar.gz` file at the end that will contain the weights (`.pth`) and classes (`.txt`).

Google provides a free [NVIDIA T4](https://www.nvidia.com/en-gb/data-center/tesla-t4/) when you run Colab for up to 12 hours (it's not completely regular) so remember to have the Notebook set-up to use it. Go to **Edit** > **Notebook settings** > **Hardware accelerator** > **GPU**.

Once you have downloaded `model.tar.gz` to your local filesystem you need to upload it to **AWS S3**. Follow the steps in my other repo: <https://github.com/nicolasmetallo/serverless-ssd300-vgg> to upload files, package, build, and deploy your function. You don't actually need to build before you deploy as we are not using any dependency outside of the Lambda ARN we specified in `template.yaml`.

Once you have finished deploying your function, you should see something like this:
```
---------------------------------------------------------------------------------------------
|                                      DescribeStacks                                       |
+-------------+-----------------------------------------------------------------------------+
|  Description|  API Gateway endpoint URL for Prod stage for PyTorch function               |
|  OutputKey  |  PyTorchApi                                                                 |
|  OutputValue|  https://9xxft46ddb.execute-api.us-east-1.amazonaws.com/Prod/invocations/   |
+-------------+-----------------------------------------------------------------------------+
```

At the end, use the `test.ipynb` notebook to run queries against your new Lambda function.
