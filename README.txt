REFERENCES:
	Setup Raspberry pi for bluetooth
		https://www.raspberrypi.org/forums/viewtopic.php?t=68779
		https://www.raspberrypi.org/forums/viewtopic.php?p=635509#p635509
		https://www.raspberrypi.org/forums/viewtopic.php?t=156120

		Bootable Raspberry pi on Windows:
			https://howtoraspberrypi.com/create-raspbian-sd-card-raspberry-pi-windows/

	NGINX Server setup basics
		https://opensource.com/article/19/1/basic-live-video-streaming-server

	NGINX server to publish to other rtmp servers
		https://www.iamjack.co.uk/blog/post/setting-multistream-server-nginx

	Reference to docker creation
		https://github.com/alfg/docker-nginx-rtmp

Raspberry pi command to stream bluetooth audio to rtmp server
		My implementation:
        arecord -f cd -D default | ffmpeg -i - -ab 32k -ac 1 -f flv rtmp://3.225.119.164:1935/stream/audio

        Generic command:
        arecord -f cd -D default | ffmpeg -i - -ab 32k -ac 1 -f flv rtmp://<server_ip>:1935/stream/<stream_name>

        	<server_ip> is the ip of the server
        	<stream_name> can be selected by you

**
** In general, I would follow the sources linked in the REFERENCES to
** setup the raspberry pi.  If you want to check, I listed the commands
** that I ran in pi_audio.txt.
**


Make sure that the AWS security group allows RTMP through the network
	I allowed all connections from 0.0.0.0/0 to open everything
        In production, you would only allow from your pi ip address
        on the rtmp port 1935.

*** Docker install ***

-- Launch two basic EC2 Ubuntu server instance --

For the master EC2 instance (Where the rtmp stream is sent),
you will want to allocate an Elastic IP address from AWS.
	EC2 dashboard > Elastic IP
	Then assign it the master instance

-- SSH into both instances --

-- Clean any old versions of docker --
sudo apt-get remove docker docker-engine docker.io

-- Install Docker --
sudo apt install docker.io

-- Use super user --
sudo su

-- Download the Dockerfile --
git clone https://github.com/jkubath/nginx-docker.git

-- You must update the Listener nginx.config file to point to the master ip address --
	Search in the nginx.conf
		rtmp > server > stream
	Update the ip address in the ffmpeg command

-- Build the docker --
cd ...</nginx-docker/master> or ...</nginx-docker/listener>
docker build -t nginx-rtmp .

-- Launch the NGINX docker --
https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-docker/
https://hub.docker.com/_/nginx
Reference: https://github.com/alfg/docker-nginx-rtmp/Dockerfile

-- Run the docker container for NGINX with rtmp module --
docker run --name nginx-pi -p 1935:1935 -p 8080:80 --rm nginx-rtmp

-- stop the container by running --
	-- get the container id --
	sudo docker ps -aq

sudo docker stop <container id>

-- Connect to the master rtmp stream --
rtmp://<master ip>:1935/stream/<stream name>

-- listen in browser --
http://<listener ip>:8080/live/<stream name>.m3u8

-- view stream stats --
http://<listener ip>:8080/stat


-- Setting up the AWS Load Balancer --

Default http port 80
Add http port 8080
Choose two availability zones

Choose a security group
	For testing, I have a security group all open 
		(0.0.0.0/0 on inbound and outbound)

Setup new Target Group
	Target Type: Instance
	Protocol: HTTP
	Port : 8080 # The port to forward the traffic to the instances

-- Setup the AWS Auto Scaling Group --
I created an AMI from the nginx-listener ec2 instance I had running
I then used this as my launch configuration
	In your launch configuration, tell the Auto Scaler to start the docker listener on startup
		#!/bin/bash
		sudo docker run --name nginx-pi -p 1935:1935 -p 8080:80 --rm nginx-rtmp

Desired Cap - 2
Min - 2
Max - 3

Set the default region (I used us-east-1d)

Target Group - select your target group created in the Load balancer

	-- Listen in browser --
	http://<load balancer address>:8080/stream/audio.m3u8




