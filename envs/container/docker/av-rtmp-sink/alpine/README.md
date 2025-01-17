# av-rtmp-sink alpine

## Overview
This av-service av-rtmp-sink for alpine is a container running nginx RTMP server which convert an incoming RTMP stream into RTMP stream, HLS stream. Moreover, the server hosts a basic HTML page to play the HLS content.

## Using av-rtmp-sink alpine
It's recommended to use and manage the av-rtmp-sink alpine service with the avtool.sh command line tool.

### Installing the pre-requisites on the host machine
As avtool.sh is a Linux bash file, you could run this tool from a machine or virtual machine running Ubuntu 20.04 LTS.

1. Ensure git is installed running the following command

```bash
    sudo apt-get install git
```

2. Clone the av-services repository on your machine

```bash
    mkdir $HOME/git
    cd $HOME/git
    git clone https://github.com/HarryWEB/av-services.git
    cd av-services/envs/container/docker/av-rtmp-sink/alpine 
```
3. Run avtool.sh -a install to install docker 

```bash
    ./avtool.sh -a install
```

### Deploying/Undeploying av-rtmp-sink alpine service
Once the pre-requisites are installed, you can build the av-rtmp-sink alpine container.


1. Run the following command to build and run the container

```bash
    ./avtool.sh -a deploy
```

When you run avtool.sh for the first time, it creates a file called .avtoolconfig to store the av-rtmp-sink configuration. By default, the file contains these parameters:

```bash
    AV_IMAGE_NAME=av-rtmp-sink-alpine
    AV_IMAGE_FOLDER=av-services
    AV_CONTAINER_NAME=av-rtmp-sink-alpine-container
    AV_COMPANYNAME=contoso
    AV_HOSTNAME=localhost
    AV_PORT_HLS=8080
    AV_PORT_HTTP=80
    AV_PORT_SSL=443
    AV_PORT_RTMP=1935
```

Below further information about the parameters in the file .avtoolconfig:

| Variables | Description |
| ---------------------|:-------------|
| AV_IMAGE_NAME | The suffix of the image name   |
| AV_IMAGE_FOLDER | The image folder, the image name will be ${AV_IMAGE_FOLDER}/${AV_IMAGE_NAME}  |
| AV_CONTAINER_NAME | The name of the container  |
| AV_COMPANYNAME | The company name used to create the certificate. Default value: contoso |
| AV_HOSTNAME | The host name of the container. Default value: localhost  |
| AV_PORT_HLS | The HLS TCP port. Default value: 8080 |
| AV_PORT_HTTP | The HTTP port. Default value: 80 |
| AV_PORT_SSL | The SSL port. Default value: 443  |
| AV_PORT_RTMP | The RTMP port. Default value: 1935  |
| AV_TEMPDIR | The directory on the host machine used to store MKV and MP4 files for the tests |

When the service is running and fed with a RTMP stream, the following urls could be used for the tests:

- RTMP input url: rtmp://'$HOSTNAME:$PORT_RTMP'/live/stream
- HTTP url: http://'$HOSTNAME:$PORT_HTTP'/player.html
- HTTPS url: https://'$HOSTNAME:$PORT_SSL'/player.html
- HLS url: http://'$HOSTNAME:$PORT_HLS'/live/stream.m3u8
- RTMP output url: rtmp://'$HOSTNAME:$PORT_RTMP'/live/stream


### Starting/Stopping av-rtmp-sink alpine service
Once the image is built you can start and stop the container .


1. Run the following command to start the container

```bash
    ./avtool.sh -a start
```
By default the container will run the following command to encod the MKV file:


```bash
    ffmpeg -y -nostats -loglevel 0  -i ./camera-300s.mkv -codec copy /${AV_VOLUME}/camera-300s.mp4
```


2. If the container is still running, you can run the following command to stop the container

```bash
    ./avtool.sh -a stop
```

3. If the container is still running, you can run the following command to get the status of the container

```bash
    ./avtool.sh -a status
```

### Testing av-rtmp-sink alpine service
Once the image is built you can test if the container is fully functionning.

1. Run the following command to test the container

```bash
    ./avtool.sh -a test
```

For this container, the test feature will check if the output MP4 files have been created in the temporary folder from output HLS url and output RTMP url.

By default for the tests, we use an incoming Live RTMP stream created from a MKV file using the following ffmpeg command:

```bash
        ffmpeg -hide_banner -loglevel error  -re -stream_loop -1 -i "${AV_TEMPDIR}"/camera-300s.mkv -codec copy -bsf:v h264_mp4toannexb   -f flv rtmp://${AV_HOSTNAME}:${AV_PORT_RTMP}/live/stream
```


If on the host machine a Webcam is installed, you can use this webcam to generate the incoming RTMP stream using the following ffmpeg command:

```bash
            ffmpeg.exe -v verbose -f dshow -i video="Integrated Webcam":audio="Microphone (Realtek(R) Audio)"  -video_size 1280x720 -strict -2 -c:a aac -b:a 192k -ar 44100 -r 30 -g 60 -keyint_min 60 -b:v 2000000 -c:v libx264 -preset veryfast  -profile main -level 3.0 -pix_fmt yuv420p -bufsize 1800k -maxrate 400k    -f flv rtmp://localhost:1935/live/stream
```


