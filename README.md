# RTMP2SRT

## Introduction

This is a fully featured protocol translator between RTMP and SRT. The single tenant app is a docker container that can be instantiated using mimimun compute resources. It can be hosted in AWS with a t2.micro EC2 instance without issues. The app features a configuration page for SRT that allows to bypass the input and generate a color-bars stream at 720p30 CBR 5Mbps to check that the SRT side is working before the RTMP input becomes available.

This application is meant to be used for testing or short none critical events.

This application was created using PHP, FFMPEG and srt-live-transmit (libraries 1.4.4). This is an stateful application that does not use DB, all relevant variables are files. The two backend scripts are written in Bash. One is used for monitoring and log creation, the other one is orchestrating the the specific FFMPEG or srt-live-stream commands and checks that they are up and running.

The application has a Graph tab that will show the graph of all data points (packet Loss, Packet Retransmit and Packet Drop) in the SRT file.

## What is new on version 1.4.6

1.- When the container is launched, it creates a self-signed certifiate for 365 days, HTTPS is enable and traffic is redirected to port 443 instead of 8090 or (HTTP)
2.- Basic authentication is now enabled, there is no session management yet implemented.<br>
User: I------------a<br>
Pass: I------------s

## Starting the application

There are two docker-compose files on this reposetory:

docker-compose.yml has specific ports opened for TCP and UDP traffic.

docker-compose-host.yml has the network_mode set to host, taking all networking from the host system.

here is an example of one of the files:

```
version: '3.3'

services:
  app:
    image: rubenc2/rtmp2srt:1.4.6
    hostname: app
    container_name: rtmp2srt
    restart: unless-stopped
    networks:
      - intelsat
    ports:
      - 443:443                  #HTTPS
      - 8090:80/tcp              #Map host's port 8090 to container port 80
      - 1935:1935/tcp            #RTMP port
      - 4200:4200/udp            #Single UDP port for SRT
      - 6000-6002:6000-6002/udp  #Range of UDP ports for SRT
    environment:
      - SERVER_IP=[YOUR SERVER IP]

networks:
  intelsat:
    driver: bridge
```

## Enabling the application

Once you start the container and go to port 8090 (in our case). You will have a message that says that the app is not enabled.

To enable the application, you will need to call the enable.php script with a sys=1 query parameter as follows:

```http://[IP:8090]/enable.php?sys=1```

To disable the application, or better said, to disable control of the application use the query parameter sys=0 as follows:

```http://[IP:8090]/enable.php?sys=0```

## Locking the application

In an event that the application is in used, and to stop other users from interacting with the application and potentially stop a stream, the lock.php script will allow the user to lock the application while in use.

To lock:

```http://[IP:8090]/lock.php?lock=1```

To unlock:

```http://[IP:8090]/lock.php?lock=0```

## Usage

Clicking on the SRT button will bring you to the SRT Configuration page. Hovering over the SRT button will show the current SRT configuration.

Clicking on the RTMP button will copy the RTMP parameters to the clipboard. Hovering over the RTMP button will show the current RTMP configuration.

The RTMP server is configured to listen on 0.0.0.0:1935 for connections. It is under the "live" application and uses a ramdomly generated eight number key that changes everytime a session is stopped. Make sure to contruct the URL properly when working with the RTMP applications that will be generating the stream. Please remember that RTMP uses TCP as the transport protocol and it is susceptible to latency.

When the application is active and working it will check for the connections of both the input and output. It will become RED if no connections are established, these conditions are also logged (under the Logs page). Once active and GREEN (all connections are good) it will show the bitrates at each side of the buttons. The Logs will have more detailed information, there will be a log entry every 10 seconds or so. The logs page will automatically paginate showing always the most recent log on the first page 😃

The application has also the ability to check and monitor the input to the SRT sender. Click on the Input nav menu item to navigate to the input monitoring page. The video.js pgae will have the player ready to be used.

## Test and Deploy

Use the docker-compose up -d command (in the same directory where the .yml file is) to create the container from the docker image. The image will be downloaded from the docker hub registry.

You can use the test input stream to just test SRT (under SRT Configuration page), or use OBS to stream an RTMP from your computer.

## Demo

This is a demo app on our sandbox

## Roadmap

When having the time, I will work on an SRTMP2SRT application based on this same framework. the app will support RTMP, SRT, and RIST as input or output

## License

This project is licensed under the MIT License.


