# Nvidia GPU Acceleration of Azure Custom Vision Models for Jetson Devices
One of the challenges in leveraging Azure's Custom Vision containers for production has been GPU acceleration for the exported model when deploying to IoT Edge. The purpose of this repository is to share my learnings from experimenting with Nvidia Jetson-based GPU devices, using IoT Edge with Azure Custom Vision containers.

If you're utilizing Jetpack 4.4, there is a pre-built ONNX runtime container available to implement, which saves a bit of pre-work.  For the purposes of this repository, you would simply replace the base container in the secondary docker file FROM mcr.microsoft.com/azureml/onnxruntime:v.1.4.0-jetpack4.4-l4t-base-r32.4.3 and build the container as noted. 

If, however, you want to utilize Jetpack 4.5.1 or 4.6 (which is the latest as of this publish date) then you would need to follow the ONNX-runtime base container build in the folder above, using the appropriate base container from Nvidia, i.e. nvcr.io/nvidia/l4t-base:r32.5.0 for Jetpack 4.5.1.  This is based off of the example container build available in the ONNX Runtime repo:  https://github.com/microsoft/onnxruntime/blob/master/dockerfiles/Dockerfile.jetson

Since hardware acceleration for Nvidia Jetson-based devices is achieved via a passthrough to the underlying CUDA/cuDNN/TensorRT instantiation on the device using the libnvidia-container library (https://github.com/NVIDIA/libnvidia-container/tree/jetson), the base container build for the ONNX runtime should be done on the Nvidia device itself.  Otherwise, the ONNX Runtime wheel file will not recognize it has access to GPU.  After building, you can then log into your Azure Container Repository, and do a docker push of the ONNX-base container to be used as an artifact for the secondary build.  

As a pre-requisite to the build, I've found that downloading the appropriate pre-built ONNX Runtime wheel file and including it via 'COPY' works much better than performing a 'RUN wget' in the dockerfile.  Here is a link to the pre-built wheels from Nvidia Jetson Zoo: https://elinux.org/Jetson_Zoo#ONNX_Runtime.  This also seems to work much better than calling the build.sh script to build the wheel from source files as noted in the ONNX Runtime repo documentation:  https://github.com/microsoft/onnxruntime/tree/master/dockerfiles#nvidia-jetson-tx1tx2nanoxavier.

 Testing raw .jpg images over HTTP with this build running the Tegra OS/Jetpack 4.5.1 using a Flask app has been yielding results in the ~80ms range when tested on a Xavier AGX device, once the ONNX Runtime engine is initiated.  Faster round-trip inference times could likely be achieved through utilizing gRCP endpoints in lieu of HTTP, which is a planned enhancement to the example.

To get started, go to https://azure.microsoft.com/en-us/services/cognitive-services/custom-vision-service/ and train your object detection model using Azure Custom Vision. Export the trained model as an ONNX model (FP16 or FP32), and replace the model.onnx placeholder file in the folder above, as well as the labels.txt file.

I hope that you find this useful for your Azure IoT Edge deployments!
