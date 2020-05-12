# Containerizing an app
 - Đưa ứng dụng vào container và ta sẽ tạo ra ứng dụng bằng build, ship, run. Quá trình như sau:
  
  - code của app
  - viết 1 dockerfile để miêu tả app, các thành phần phụ thuộc và cách run nó.
  - Tạo image từ dockerfile: docker image build 
 
 - 	Bây giờ ta có thể mang nó đi và  chạy nó như một container.
 
 <img src="https://i.imgur.com/aBcOXGr.png">
 
 ## Build apps
 
 clone code từ github: `git clone https://github.com/nigelpoulton/psweb.git`. Sau khi clone về ta sẽ được thư mục `psweb`, nó chứa tất cả những thứ như sourcecode và các thư mục con. Trong đó có file Dockerfile để chỉ dẫn cách thức đóng gói 1 image.

*Lưu ý: phải viết đúng Dockerfile, nếu viết dockerfile hay Docker file thì sẽ không hợp lệ*

 ```sh
 [root@manager psweb]# cat Dockerfile 
# Test web-app to use with Pluralsight courses and Docker Deep Dive book
# Linux x64
FROM alpine

LABEL maintainer="nigelpoulton@hotmail.com"

# Install Node and NPM
RUN apk add --update nodejs nodejs-npm

# Copy app to /src
COPY . /src

WORKDIR /src

# Install dependencies
RUN  npm install

EXPOSE 8080

ENTRYPOINT ["node", "./app.js"]
 ```
 
 - Tạo image từ Dockerfile trên: `docker image build -t web:latest .`. Dấu . thể hiện Dockerfile ở trong thư mục hiện tại.
 
 ```sh
 [root@manager psweb]# docker image build -t web:latest .
Sending build context to Docker daemon  95.74kB
Step 1/8 : FROM alpine
 ---> f70734b6a266
Step 2/8 : LABEL maintainer="nigelpoulton@hotmail.com"
 ---> Running in 361e9abe3451
Removing intermediate container 361e9abe3451
 ---> ee4da6c412dd
Step 3/8 : RUN apk add --update nodejs nodejs-npm
 ---> Running in 33ea279bd4ad
.......................................................
Step 7/8 : EXPOSE 8080
 ---> Running in c282fb3d6f31
Removing intermediate container c282fb3d6f31
 ---> 8c42789204d1
Step 8/8 : ENTRYPOINT ["node", "./app.js"]
 ---> Running in 8d85aed999c6
Removing intermediate container 8d85aed999c6
 ---> a6529223ef47
Successfully built a6529223ef47
Successfully tagged web:latest
 ```
 
 **push image**
 - Sau khi tạo được image, ta có thể lưu image trong một registry (Docker hub) để đảm bảo an toàn cũng như cho người khác sử dụng. 
 *login docker hub*
 ```sh
 [root@manager psweb]# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: ladung
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
 ```
 - Đổi tag:`docker image tag web:latest ladung/web:latest`. Do ta phải cần truyền tên của repository với các unoffical repository.
 - push image lên repository: `docker image push ladung/web:latest`
 **Run app**
 `docker container run -d --name web1 -p 80:8080 web:latest`
 