# 1. Docker Engine
 Docker Engine là thành phần chính của Docker. Nó hoạt động như một ứng dụng client-server.
 
 Các thành phần của Docker Engine:
 
 - Server hay còn được gọi là docker daemon (dockerd): chịu trách nhiệm tạo, quản lý các Docker objects như images, containers, networks, volume.
 - REST API: docker daemon cung cấp các api cho Client sử dụng để thao tác với Docker.
 - Client là thành phần đầu cuối cung cấp một tập hợp các câu lệnh sử dụng api để người dùng thao tác với Docker. (Ví dụ docker images, docker ps, docker rmi image v.v..)
 
 <img src="https://i.imgur.com/4ebOxUy.png">
 
 # 2. Kiến trúc của Docker
  Docker sử dụng kiến trúc client-server. Docker server (hay còn gọi là daemon) sẽ chịu trách nhiệm build, run, distrubute Docker container. Docker client và Docker server có thể nằm trên cùng một server hoặc khác server. Chúng giao tiếp với nhau thông qua REST API dựa trên UNIX sockets hoặc network interface.
 ## Docker daemon
 Docker daemon (dockerd) là thành phần core, lắng nghe API request và quản lý các Docker object. Docker daemon host này cũng có thể giao tiếp được với Docker daemon ở host khác.