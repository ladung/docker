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
 
 - Tạo image từ Dockerfile trên: 