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
 
 Image là một template read-only sử dụng để chạy container.

 Một image có thể base trên một image khác. *Ví dụ bạn muốn tạo một image nginx, tất nhiên nginx phải chạy trên linux CentOS chẳng hạn. Khi đó image nginx trước hết sẽ phải base trên CentOS trước đã.*

 Có 3 cách để có được image:
 
 - Tải các image có sẵn của người khác trên Docker Registry
 - Tạo một container, cài đặt các ứng dụng cần thiết trên container và sử dụng lệnh docker commit để tạo ra image mới.
 - Viết Dockerfile và build nó để tạo ra image.
 
 Các image là dạng read only file. Khi tạo một container mới, trong mỗi container sẽ tạo thêm một lớp có-thể-ghi được gọi là container-layer. Các thay đổi trên container như thêm, sửa, xóa file... sẽ được ghi trên lớp này. Do vậy, từ một image ban đầu, ta có thể tạo ra nhiều máy con mà chỉ tốn rất ít dung lượng ổ đĩa.
 
 Quy tắt đặt tên images: [REPOSITORY:TAG]
  *Trong đó, TAG là phiên bản của images. Mặc định, khi không khai báo tag thì docker sẽ hiểu tag là latest*
  
  ### Một số lệnh làm việc với images
  - Liệt kê các images: `docker images`