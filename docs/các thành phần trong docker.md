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
 ## Docker client
 Docker client (docker) là phương thức chính để người dùng thao tác với Docker. Khi người dùng gõ lệnh docker run imageABC tức là người dùng sử dụng CLI và gửi request đến dockerd thông qua api, và sau đó Docker daemon sẽ xử lý tiếp.

 Docker client có thể giao tiếp và gửi request đến nhiều Docker daemon.

 ## Docker registry
 Docker registry là một kho chứa các Image. Nổi tiếng nhất chính là Docker Hub, ngoài ra bạn có thể tự xây dựng một Docker registry cho riêng mình.

 ### Docker object
 Các object này chính là các đối tượng mà ta thường xuyên gặp phải khi sử dụng Docker gồm có:
 
 **image**
 
 Image là một template read-only sử dụng để chạy container. Image có ý nghĩa tương tự như VM template trên VMWare.

 Một image có thể base trên một image khác. *Ví dụ bạn muốn tạo một image nginx, tất nhiên nginx phải chạy trên linux CentOS chẳng hạn. Khi đó image nginx trước hết sẽ phải base trên CentOS trước đã.*

 Có 3 cách để có được image:
 
 - Tải các image có sẵn của người khác trên Docker Registry
 - Tạo một container, cài đặt các ứng dụng cần thiết trên container và sử dụng lệnh docker commit để tạo ra image mới.
 - Viết Dockerfile và build nó để tạo ra image.
 
 Các image là dạng read only file. Khi tạo một container mới, trong mỗi container sẽ tạo thêm một lớp có-thể-ghi được gọi là container-layer. Các thay đổi trên container như thêm, sửa, xóa file... sẽ được ghi trên lớp này. Do vậy, từ một image ban đầu, ta có thể tạo ra nhiều máy con mà chỉ tốn rất ít dung lượng ổ đĩa.
 
 Quy tắt đặt tên images: [REPOSITORY:TAG]
  *Trong đó, TAG là phiên bản của images. Mặc định, khi không khai báo tag thì docker sẽ hiểu tag là latest*
  
  ### Một số lệnh làm việc với images
  - Liệt kê các images tồn tại trên host: `docker images`
  
  Kết quả:
  
  ```sh
 [root@CentOS ~]# docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    hello-world         latest              bf756fb1ae65        4 months ago        13.3kB
  ```
  - Tìm kiếm image từ Docker Hub: `docker search centos`. Kết quả sẽ trả về các image có tên centos 
  
  ```sh
  [root@CentOS ~]# docker search centos
NAME                               DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
centos                             The official build of CentOS.                   5991                [OK]                
ansible/centos7-ansible            Ansible on Centos7                              128                                     [OK]
consol/centos-xfce-vnc             Centos container with "headless" VNC sessionâ€¦   114                                     [OK]
jdeathe/centos-ssh                 OpenSSH / Supervisor / EPEL/IUS/SCL Repos - â€¦   114                                     [OK]
centos/mysql-57-centos7            MySQL 5.7 SQL database server                   75                                      
imagine10255/centos6-lnmp-php56    centos6-lnmp-php56                              58                                      [OK]
tutum/centos                       Simple CentOS docker image with SSH access      46                                      
centos/postgresql-96-centos7       PostgreSQL is an advanced Object-Relational â€¦   44                                      
centos/httpd-24-centos7            Platform for running Apache httpd 2.4 or buiâ€¦   31                                      
kinogmt/centos-ssh                 CentOS with SSH                                 29                                      [OK]
pivotaldata/centos-gpdb-dev        CentOS image for GPDB development. Tag namesâ€¦   11                                      
guyton/centos6                     From official centos6 container with full upâ€¦   10                                      [OK]
drecom/centos-ruby                 centos ruby                                     6                                       [OK]
centos/tools                       Docker image that has systems administrationâ€¦   6                                       [OK]
pivotaldata/centos                 Base centos, freshened up a little with a Doâ€¦   4                                       
mamohr/centos-java                 Oracle Java 8 Docker image based on Centos 7    3                                       [OK]
darksheer/centos                   Base Centos Image -- Updated hourly             3                                       [OK]
pivotaldata/centos-gcc-toolchain   CentOS with a toolchain, but unaffiliated wiâ€¦   3                                       
pivotaldata/centos-mingw           Using the mingw toolchain to cross-compile tâ€¦   3                                       
miko2u/centos6                     CentOS6 æ—¥æœ¬èªžç’°å¢ƒ                                   2                                       [OK]
blacklabelops/centos               CentOS Base Image! Built and Updates Daily!     1                                       [OK]
indigo/centos-maven                Vanilla CentOS 7 with Oracle Java Developmenâ€¦   1                                       [OK]
pivotaldata/centos6.8-dev          CentosOS 6.8 image for GPDB development         0                                       
pivotaldata/centos7-dev            CentosOS 7 image for GPDB development           0                                       
smartentry/centos                  centos with smartentry                          0                                       [OK]
  ```
  
Trong đó:

 Cột NAME : tên của images

 Cột DESCRIPTION: Mô tả ngắn gọn của images

 Cột OFFICIAL: Là images chính thức do công ty Docker cung cấp. Trạng thái là OK.
 
 *Sau khi xác định được images muốn sử dụng, bạn sử dụng tiếp lệnh docker pull để kéo images từ internet về host cài docker của bạn.*
 
 - Xóa một image: `docker rmi [tên image/ID image]`
 
 **Container**: là một instance của một image. Bạn có thể create, start, stop, move or delete container dựa trên Docker CLI. 
 
 Ta có thể kết nối 1 hoặc nhiều network, lưu trữ nó, hoặc thậm chí tạo ra 1 image mới dựa trên trạng thái của nó.
 
 Container có ý nghĩa tương tự như là 1 VM trong ảo hóa VMware. 