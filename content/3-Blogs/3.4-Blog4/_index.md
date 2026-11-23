---
title: "Blog 4"
date: 2026-11-21
weight: 4
chapter: false
pre: " <b> 3.4. </b> "
---

# From SageMaker to the Device: Deploying ML Models to the Edge with AWS IoT Greengrass V2

by Nguyễn Tiến Long | November 21, 2026 | published in [AWS Study Group VN](https://www.facebook.com/groups/awsstudygroupfcj/) | [Original post on Facebook](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2344828116282147/)

*A post by team member Nguyễn Tiến Long. The post was written and published in Vietnamese; this page is an English translation that stays faithful to the published version.*

With AWS IoT Greengrass, a device can run machine learning inference locally on the data it generates, using a model trained in the cloud. This post introduces the "Perform machine learning inference" section of the AWS IoT Greengrass V2 Developer Guide. I have not tried it yet; the content is what I learned from the documentation.

### 1. Why run AI directly on the device

The documentation lists two benefits of local inference: low latency and cost savings. Training and complex processing are still handled in the cloud. The model can be trained anywhere, for example built and trained on Amazon SageMaker AI, stored in an S3 bucket, and then used as an artifact in a component that runs on the device.

For robots, the Guidance for Physical AI for Robotics on AWS page says that Greengrass plays two roles: running ML inference to control the robot in real time, and collecting sensor data to improve the model. In the AWS Physical AI Toolchain architecture, Greengrass is the last step, delivering the optimized model to NVIDIA Jetson devices.

### 2. Greengrass V2 and the basic concepts

Greengrass is an open-source IoT runtime that runs on devices. It lets you process data locally, run predictions with ML models, and still communicate securely with AWS IoT Core (Figure 1).

![Overview of AWS IoT Greengrass](/images/3-Blog/blog4_figure1.jpg)

*Figure 1: Overview of AWS IoT Greengrass: devices run the Greengrass client software and connect securely to the Greengrass service in the AWS Cloud. Source: AWS IoT Greengrass Developer Guide.*

There are three concepts to remember. A **core device** is a device with the Greengrass software installed. A **component** is a packaged unit of software installed on the device. A **deployment** pushes one or more components from the cloud to devices. The core software that runs on the device is called the **nucleus**.

What is different in V2 is that the model, the runtime, and the inference code are packaged as components, so installing the runtime, downloading the model, and running inference happen in a single step. Greengrass V1 reached end of support on October 7, 2026, so older tutorials online (often written for V1) may no longer work.

### 3. How ML inference works

AWS splits the ML part into three types of components:

- **Model:** contains the ML model as a Greengrass artifact.
- **Runtime:** contains the script that installs the ML framework and its dependencies on the device.
- **Inference:** contains the inference code and declares the two types above as dependencies.

The process has four steps: create the model component, create the runtime component, create the inference component, and then deploy. When you deploy the inference component, Greengrass automatically deploys the model and runtime it depends on as well (Figure 2). AWS provides sample components for image classification and object detection, so you can try it without writing anything yourself.

![The ML inference flow with Greengrass](/images/3-Blog/blog4_figure2.jpg)

*Figure 2: The ML inference flow with Greengrass. Diagram drawn by me based on the AWS documentation.*

### 4. Where the model comes from and what it takes to run

Greengrass uses your own trained models stored in Amazon S3. The bucket that holds the model must meet a few conditions: same AWS account and same Region as the components, no SSE-C encryption (only SSE-S3 or SSE-KMS are supported), and no dots in the bucket name. The device role must allow the `s3:GetObject` action so that Greengrass can read the model.

On the device side, you need a Greengrass core device and at least 500 MB of free storage to use the AWS sample components. The sample runtimes are Deep Learning Runtime (DLR) and TensorFlow Lite. The sample models are ResNet-50 for image classification and YOLOv3 for object detection.

### 5. Customizing for your needs

The sample components can be used as they are or replaced piece by piece. You can change the configuration of the inference component when you deploy it. You can replace the sample model component with your own model, as long as that model was trained with the same runtime as the sample model. To use your own inference code, write your own inference component; to use a different runtime, write your own runtime component. The sample components can also serve as templates for writing your own.

### 6. How to get started

The first thing you need is a core device. The *Getting started with AWS IoT Greengrass V2* tutorial walks you through installing Greengrass on a Linux device (for example a Raspberry Pi) or a Windows device, then creating and deploying a Hello World component, in about 20 to 30 minutes. This tutorial does not cover ML yet; the sample image classification inference is in the ML documentation. The documentation gives step-by-step commands, which I do not copy here because this post is only an introduction.

### 7. Things to watch out for

- SageMaker AI Edge Manager still has its own section in the Greengrass documentation, but the documentation itself says the service was discontinued on April 26, 2024. Anyone looking for guidance on it should use the component customization approach in section 5.
- Some AWS-provided components depend on specific minor versions of the Greengrass nucleus, so check them when upgrading the nucleus.
- The troubleshooting section of the documentation lists the common errors when running inference: missing libraries, no CUDA device found, memory errors, running out of disk space, and timeouts.
- The sample runtimes in the documentation are pinned to specific versions (DLR v1.6.0 and v1.3.0, TensorFlow Lite v2.5.0), so if you use them for real, check whether they still fit your environment.

### Conclusion and thoughts

The ML part of Greengrass is neat because everything is a component: the model, the runtime, and the inference code are separate and combined through dependency declarations, so changing the model or the runtime only means replacing that one part.

While reading, I noticed that the samples are general image classification and object detection; the Greengrass documentation does not talk about robots specifically. The link to robotics comes from the AWS Guidance page in section 1. The documentation also has a few outdated parts, such as the Edge Manager sections and the V1 tutorials online.

This topic suits people building IoT devices or robots that need local inference because of latency or unreliable networks. For someone learning AWS like me, it shows a fairly complete flow: train on SageMaker, store the model in S3, and push it down to the device.

A question for the group: has anyone deployed a model to a device with Greengrass, and which runtime did you choose?

### References

- Main document: [Perform machine learning inference (AWS IoT Greengrass V2 Developer Guide)](https://docs.aws.amazon.com/greengrass/v2/developerguide/perform-machine-learning-inference.html)
- [What is AWS IoT Greengrass? (AWS IoT Greengrass V2 Developer Guide)](https://docs.aws.amazon.com/greengrass/v2/developerguide/what-is-iot-greengrass.html)
- [Tutorial: Getting started with AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started.html)
- [Guidance for Physical AI for Robotics on AWS](https://aws.amazon.com/solutions/physical-ai-for-robotics-on-aws/)
- [Introducing AWS Physical AI Toolchain (AWS Physical AI Blog, October 7, 2026)](https://aws.amazon.com/blogs/physical-ai/introducing-aws-physical-ai-toolchain/)

#AWS #AWSIoTGreengrass #EdgeAI #MachineLearning #IoT #Robotics #AmazonSageMaker #AWSStudyGroup
