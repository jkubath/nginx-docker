# nginx-docker
Dockerfile for nginx-rtmp-master

This Dockerfile and nginx.conf can be used to setup a docker container
that will accept an rtmp stream.  This stream is then broadcasted and made
available through a browser.

In the nginx config file $stream_name = "audio"

To stream to the server:
  rtmp://$stream_name:1935/stream/$stream_name
  
  Sample command: ffmpeg -i - -ab 32k -ac 1 -f flv rtmp://$SERVER_IP:1935/stream/$stream_name
  
To listen to the stream:
  In a browser:
    http://$SERVER_IP:8080/live/$stream_name.m3u8
  
  In VLC or other rtmp applications:
    rtmp://$SERVER_IP:1935/stream/$stream_name

Reference: https://github.com/alfg/docker-nginx-rtmp
