# nginx-docker
Dockerfile for nginx-rtmp-master

This Dockerfile and nginx.conf can be used to setup a docker container
that will accept an rtmp stream.  This stream is then broadcasted and made
available through a browser.

In the nginx config file <stream name> = "audio"

To stream to the server:
  rtmp://<server ip>:1935/stream/<stream name>
  
  Sample command: ffmpeg -i - -ab 32k -ac 1 -f flv rtmp://3.92.27.179:1935/stream/<stream name>
  
To listen to the stream:
  In a browser:
    http://<server ip>:8080/live/<stream name>.m3u8
  
  In VLC or other rtmp applications:
    rtmp://<server ip>:1935/stream/<stream name>


