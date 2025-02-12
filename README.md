# AMAZON WEB SERVICE

# Amazon Kinesis Video Streams, Realtek, and Plumerai Combine Vision AI at the Edge with Efficient Video Streaming

- Zihang Huang
- Marco Jacobs
- Siva Somasundaram
- Emily Chou
 
## 1 - INTRODUCTION
This blog highlights the integration solution of Kinesis Video Streams, Realtek Ameba Pro2 SoC and the Plumerai vision AI models. It shows how these components are combined to provide a low-power, efficient solution that for home security and IoT applications. This solution addresses the growing camera market demand for intelligent video systems that deliver actionable AI-powered insights. This blog post focuses on the event-based video uploading scenario with People Detection model on the edge. More video streaming scenarios combined with Familiar Face Identification will be implemented in a follow-up blog.

## 2 - TARGET SCENARIOS
### 2.1 - SMART HOME CAMERAS
Smart home cameras and video doorbells initially provided basic features such as recording and remote viewing, allowing users to monitor their homes from anywhere. These cameras have now evolved to include advanced features that analyze the images they capture, and automatically notifying users of interesting events, such as motion, familiar faces, or package deliveries, and starting and stopping video recordings.
### 2.2 - ENTERPRISE SURVEILLANCE CAMERAS
Enterprise surveillance cameras typically feature higher resolution and more capable compute capabilities compared to consumer-grade smart home cameras. This enhanced hardware allows running larger AI models, increasing accuracy and detection distances, further improving security and monitoring effectiveness.
### 2.3 - USE CASES
The solution is designed to enhance smart home security and elevate the user experience through innovative AI features, superior detection accuracy, cost-effective deployment, and excellent engineering support:
- Advanced Motion Detection
- People Detection
- Familiar Face Identification
- Stranger Detection
- Vehicle Detection
- Animal Detection
- Package Detection
- Person Re-identification
- AI Video Search

## 3 - SOLUTION ARCHITECTURE
A typical architecture of video streaming systems is shown below.
![image](https://github.com/user-attachments/assets/99daa7e1-6c1e-4b7d-b57c-5d98a8ee7e5d)

## 4 - INTEGRATION HIGHLIGHTS
### 4.1 - AMAZON KINESIS VIDEO STREAMS (KVS)
KVS provides the following advantages for building a video streaming system:
- Fully Managed Serverless Service: Kinesis Video Streams is a fully managed service, eliminating the need to provision, scale, and maintain video streaming infrastructure.
- Automatic Scaling: It automatically scales to handle any amount of video data, making it suitable for applications with highly variable video traffic.
- Durability and Availability: Video data is automatically replicated across multiple Availability Zones for high durability and availability.
- Low Latency: It offers low latency for real-time video streaming applications, such as live video broadcasting or surveillance systems.
- Secure Video Ingestion: It supports secure video ingestion from devices or applications using HTTPS.
- Video Analytics Integration: It integrates with AWS services like Amazon Rekognition Video for performing real-time video analytics, such as object detection and recognition.
- Cost-Effective: It offers a pay-per-use pricing model, making it cost-effective for applications with varying video traffic.
These advantages make Amazon KVS a compelling choice for building scalable, secure, and real-time video streaming solutions for smart home, robotics, and surveillance use cases.
### 4.2 - PLUMERAI
Plumerai’s AI solution is available and optimized for the Realtek Ameba Pro2 platform using the following techniques: 
- Arm Cortex-M host CPU:
    - Assembly-level optimizations
    - Leveraging DSP instructions
- 0.4 TOPS NPU acceleration:
    - Used Neural Architecture Search (NAS) to select optimal AI models
    - Optimized models for Realtek NPU and memory architecture
- Hardware acceleration: Uses on-chip accelerators (scalers, format converters) to reduce computational load
- RTOS: Integrates seamlessly with the SoC's real-time operating system
- Media framework: Integrates with Realtek's media streaming framework
- Fast boot: Supports rapid boot times, improving battery life and maintaining detection performance
### 4.3 - REALTEK AMEBA PRO2
![image](https://github.com/user-attachments/assets/09f189af-e0b7-441c-a006-ee2cfa369987)
Realtek Ameba Pro2 SoC consists of Integrated Video Encoder, ISP (Image Signal Processor), and:
- Video Offload Engine:
    - Manages multiple video channels
    - Concurrent streaming for video clip and motion detection
- Parallel Processing Unit (PPU):
    - Crops ROIs from high-resolution images
    - Resizes for NPU inference input
    - Retrieves final output from high-resolution channel
- Neural Network Processing Unit (NPU):
    - Performs inference on images or image regions
Key benefits for video analytics application:
- Optimized low CPU power consumption.
- Fast boot enables fast first frame capture upon PIR-detected motion.
- Early video frame capture and pre-processing during boot.
- Video output to SD card and securely to AWS cloud via WiFi/Ethernet.
- Optimized On-Chip AI supported by NPU.
- Multimedia framework SDK for PlumerAI models and Amazon KVS.

## 5 - WALKTHROUGH
This section outlines the steps to run the Plumerai AI on the Realtek Ameba Pro2 Mini board and streaming the video fragments into Amazon Kinesis Video Streams.
### 5.1 - PREREQUISITES
- AWS account with permission for
    - Amazon Kinesis Video Stream (KVS) for video ingesting, storage, and playback
    - Amazon Elastic Compute Cloud (EC2), to build KVS & Realtek SDK and binaries
- Pre-created KVS stream named “kvs-plumerai-realtek-stream” in [KVS Console](https://us-east-1.console.aws.amazon.com/kinesisvideo/home?region=us-east-1#/streams)
- Realtek [Ameba Pro2 Mini](https://www.amebaiot.com/en/video/)
- Basic knowledge about embedded system and Linux.
- Internet connection for downloading the SDK from GitHub and uploading video to AWS.
- Library and machine learning model files from Plumerai. Please submit your request on the [Plumerai Website](https://plumerai.com/partners/amazon-realtek.html).
### 5.2 - SET UP BUILDING ENVIRONMENT
This blog uses an [Amazon Elastic Compute Cloud](https://aws.amazon.com/ec2/) (Amazon EC2) instance that runs Ubuntu LTS 14.22 as our building environment. 
#### 5.2.1 - EC2 INSTANCE
Sign in into AWS management console and open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/), and launch an instance with configuration:
- Instance name: KVS_AmebaPlumerAI_poc
- Application and OS Images: Ubuntu Server 22.04 LTS (HVM)
- Instance type: t3.large
- Create a new key pair for login: kvs-plumerai-realtek-keypair
- Configure storage: 100GiB
#### 5.2.2 - DEPENDENCIES INSTALLATION
Login to the EC2 with the key pair from your local command line tool.
```console
ssh -o ServerAliveInterval=60 -i kvs-plumerai-realtek-keypair.pem ubuntu@54.64.xxx.xxx
mkdir KVS_Ameba_Plumerai
cd /home/ubuntu/KVS_Ameba_Plumerai
# install dependencies
sudo apt-get update
sudo apt-get install bzip2 
sudo apt-get install cmake
sudo apt-get install build-essential
```
#### 5.2.3 - OBTAIN PLUMERAI LIBRARY
After submitting your request, Plumerai will share the demo package, which will be placed in a directory named “plumerai”.
#### 5.2.4 - OBTAIN AMEBA SDK.
The Github for Ameba Pro2 SDK can be found here. 
Create a project directory named “KVS_Ameba_Plumerai”, and copy the directory from Plumerai. Go to the project directory, and download Ameba Pro2 SDK from Github and set up the building toolchain.
```console
# download SDK
git clone --recursive https://github.com/ambiot/ambpro2_sdk.git
chmod -R 777 ./ambpro2_sdk
```
#### 5.2.5 - REALTEK SDK TOOLCHAIN
```console
# prepare toolchain
cd ./ambpro2_sdk/tools
cat asdk-10.3.0-linux-newlib-build-3633-x86_64.tar.bz2.* | tar jxvf -
export PATH=$PATH:/home/ubuntu/environment/ambpro2_sdk/tools/asdk-10.3.0/linux/newlib/bin
```
The folder structure should look like:
```
└── KVS_Ameba_Plumerai
    ├── ambpro2_sdk
    │   └── ...
├── plumerai
    └── example_app
    └── include
    └── lib
    └── models
    └── patches
    └── src
```
#### 5.2.6 - APPLY PLUMERAI PATCH
Choose the patch from plumerai/patches/
```console
git -C ambpro2_sdk apply ./plumerai/patches/ambpro2_sdk.patch	
```
### 5.2.7 - COPY MODEL FILE
```console
cp plumerai/models/plumerai_* ambpro2_sdk/project/realtek_amebapro2_v0_example/src/test_model/model_nb/
```
## 5.3 - CONFIGURE KVS SDK
### 5.3.1 - DOWNLOAD KVS PRODUCER SDK
```console
cd ./ambpro2_sdk/project/realtek_amebapro2_v0_example/src/amazon_kvs/lib_amazon
git clone -b v1.0.1 --recursive https://github.com/aws-samples/amazon-kinesis-video-streams-producer-embedded-c.git producer
```
#### 5.3.2 - CONFIGURE KVS SDK SAMPLE
Configure AWS key, stream name and AWS region in file “component/example/kvs_producer_mmf/sample_config.h”
```c
# KVS general configuration
#define AWS_ACCESS_KEY                  "xxxxx"
#define AWS_SECRET_KEY                  "xxxxx"
# KVS stream configuration 
#define KVS_STREAM_NAME                 "kvs-plumerai-realtek-stream"
#define AWS_KVS_REGION                  "us-east-1"
#define AWS_KVS_SERVICE                 "kinesisvideo"
#define AWS_KVS_HOST                    AWS_KVS_SERVICE "." AWS_KVS_REGION ".amazonaws.com"
```
Mark out example_kvs_producer_with_object_detection() in file "/home/ubuntu/KVS_Ameba_Plumerai/ambpro2_sdk/component/example/kvs_producer_mmf/app_example.c"
```c
//example_kvs_producer_mmf();
//example_kvs_producer_with_object_detection();
```
Check sample files are available:
- "/home/ubuntu/KVS_Ameba_Plumerai/plumerai/example_app/mmf2_video_plumerai_ffid_rtsp.c"
- "/home/ubuntu/KVS_Ameba_Plumerai/plumerai/example_app/plumerai_kvs.cmake"
### 5.4 - COMPILE AND BUILD
```console
# Build project
cd /home/ubuntu/KVS_Ameba_Plumerai
cmake -DVIDEO_EXAMPLE=ON -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=../ambpro2_sdk/project/realtek_amebapro2_v0_example/GCC-RELEASE/toolchain.cmake -Sambpro2_sdk/project/realtek_amebapro2_v0_example/GCC-RELEASE -Bbuild
cmake --build build --target flash_nn
```
### 5.5 - RUN SAMPLE IN AMEBA PRO2
Use image tool to download the binary image to Ameba Pro2 Mini board, and reboot it.
For example, on a Windows computer:
```console
C:\Users\Administrator\Desktop\Pro2_PG_tool_v1.3.0>.\uartfwburn.exe -p COM3 -f flash_ntz.bin -b 2000000 -U
```
Use a serial tool to connect the board’s serial port with Baud rate = 115200, Data bits = 8, Parity=None, stop bits =1, XON_OFF.
### 5.6 -VERIFY VIDEO IN KVS CONSOLE
Open [Kinesis Video Streams console](https://ap-northeast-1.console.aws.amazon.com/kinesisvideo/home?region=ap-northeast-1#/streams/streamName/amebapro2), and choose the video stream you created. You should see the video with the bounding boxes showing the detection results. 
![image](https://github.com/user-attachments/assets/a20b138d-1941-4059-8f3e-14c8d201649c)

## 6 - CLEANING UP
Open [Amazon EC2 console](https://console.aws.amazon.com/ec2/), terminate the instance “KVS_AmebaPlumerAI_poc”.
Open [Amazon KVS Console](https://us-east-1.console.aws.amazon.com/kinesisvideo/home?region=us-east-1#/streams), delete the stream.

## 7 - PERFORMANCE SUMMARY
### 7.1 - DETECTION ACCURACY
The AI solution is designed for a variety of scenarios, built on an extensive training set, validated in the field, and compatible with a wide range of camera lenses and imagers. It ensures optimal performance across indoor, outdoor, and night conditions. Key capabilities:
- Trained on 30 million labeled images/videos for robust performance.
- Highly accurate, efficient, and with a tiny memory footprint.
- Supports lenses with up to 180° field of view.
People Detection:
- Detects individuals at distances greater than 20m (65ft).
- Tracks up to 20 people simultaneously per frame.
- Assigns unique IDs to track individuals over time.
- Works in daylight and in the dark with night vision support.
This demonstration highlights only People Detection model of Plumerai, but a complete AI solution that includes advanced motion detection, people detection, familiar face and stranger identification, vehicle, animal and package detection is readily available as well.
### 7.2 - FRAME RATE AND MODEL SIZE
People Detection model requires just 1.5MB of ROM space. Optimized for the Ameba Pro2's NPU, the software achieves approximately 10 frames per second processing speed while consuming only 20% of the CPU, leaving capacity for additional applications.
![image](https://github.com/user-attachments/assets/d036c554-f3af-48c4-ab62-c3346334c794)

## 8 - CONCLUSION
The collaboration between Amazon Kinesis Video Streams (KVS), Realtek, and Plumerai has resulted in a cutting-edge home security solution that leverages Edge Vision AI and advanced video streaming capabilities. This integrated system addresses the growing demand for sophisticated AI/ML video solutions in both smart home and enterprise surveillance markets.
This collaborative solution demonstrates the potential for AI-driven enhancements in home and enterprise security, offering improved monitoring capabilities, efficient processing, and seamless cloud integration. As the demand for intelligent video solutions continues to grow, this technology sets a new standard for what's possible in the realm of smart surveillance and video monitoring systems.

## 9 - ABOUT THE AUTHORS
<!---
.. ![image](https://github.com/user-attachments/assets/0c9283b0-9876-4d9a-901d-6db8fc842f58)
-->
<img src="https://github.com/user-attachments/assets/0c9283b0-9876-4d9a-901d-6db8fc842f58" align="left" alt="Alt Text" width="100">
Zihang Huang is a solution architect at AWS. He is dedicated for technical advocacy and solution design in IoT domains including connected vehicles, smart home, and industrial interconnection. He has technical experience at Bosch and Alibaba Cloud for multiple years. Currently, he’s focusing on researching interdisciplinary solutions integrating IoT, edge computing, big data, artificial intelligence, and machine learning.<br /><br /><br />

<!---
.. ![Marco portrait](https://github.com/user-attachments/assets/37df11e5-0cc0-4942-becd-9c7c26218b48)
-->
<img src="https://github.com/user-attachments/assets/37df11e5-0cc0-4942-becd-9c7c26218b48" align="left" alt="Alt Text" width="100">
Marco is the Head of Product Marketing at Plumerai, where he drives the adoption of tiny, highly accurate AI solutions for smart home cameras and IoT devices. With 25 years of experience in camera and imaging applications, he seamlessly connects executives and engineers to drive innovation. Holding seven issued patents, Marco is passionate about transforming cutting-edge AI technology into business opportunities that deliver real-world impact.<br /><br /><br />
