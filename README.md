# primitive-Streaming

- multipart/x-mixed-replace , img 태그를 이용하여 세련되지 못한 스트리밍이 가능

<img src="https://user-images.githubusercontent.com/57505385/227877630-62f10e98-b834-44b7-90f4-91bcc47ac4f0.png" width="70%" height="70%">


# nginx-rtmp + hls 
- 스트리밍 송출자로부터 rtmp를 받아서 hls로 변환해서 제공하는 방법  
nginx-rtmp 모듈을 합쳐서 컴파일해야하나 직접 해보니 안되었습니다 -> https://github.com/arut/nginx-rtmp-module  
make에서 컴파일이 막히는 문제가 생겼고, 구글링을 해보니 버전이 달라서 재빌드해야된다..뭐라뭐라 하는데 못고쳤습니다 -> https://github.com/arut/nginx-rtmp-module/issues/1569 정확히 이 에러는 아니나 이러한 느낌으로 에러가 납니다  
- 그 에러글에서 나온 소개방법이.. nginx를 그냥 설치 한 후에 아래와 같은 명령어로 설치해주면 사용할수 있다는 글이 있었습니다
```
sudo apt install libnginx-mod-rtmp
```

- 이후 nginx config을 수정합니다.
nginx config
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http { 
    default_type application/octet-stream;
 
    server { 
        listen 80; 
        location /live { 
            alias /usr/local/hls/; 
        } 
    }
 
    types {
        application/vnd.apple.mpegurl m3u8;
        video/mp2t ts;
        text/html html;
    } 
}

rtmp {
        server {
                listen 1935;
                chunk_size 4096;
                allow publish all;
                deny publish all;

                application live {
                        live on;
                        record off;
                        hls on;
                        hls_path /usr/local/hls;
                        hls_fragment 3;
                        hls_playlist_length 60;

                }
        }
}
```


# test
- tmp.tmp4 를 rtmp://localhost:live/test 로 송출한뒤 
```
ffmpeg -re -i tmp.mp4 -c:v libx264 -preset veryfast -maxrate 3000k -bufsize 6000k -pix_fmt yuv420p -g 50 -c:a aac -b:a 160k -ac 2 -ar 44100 -f flv rtmp://localhost/live/test
```

- 크롬에서 HLSplayer extention을 설치후 http://localhost/live/test.m3u8 로 접속하면 아래 그림과 같이 스트리밍이 보임  
<img src="https://user-images.githubusercontent.com/57505385/228704126-d83b620a-659c-42cf-8ea1-84cdbfc10326.png" width="70%" height="70%">

