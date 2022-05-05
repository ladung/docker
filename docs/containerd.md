## Tìm hiểu về Containerd ##

## 1. Khái niệm ##

- Trái tim của hệ thống container là containerd. Nó là container runtime cho docker engine sử dụng để tạo và quản lý các container. Nó trừu tượng hóa các lời gọi tới chức năng cụ thể của system hoặc OS để chạy container trên windows, solaris và OS khác. Phạm vi của containerd bao gồm các thứ sau:

	* Create, start, stop, pause, resume, signal, delete một container
    * Chức năng cho overlay, aufs và copy-on-write file system khác cho các container
    * Build, push, pull images và quản lý các image
    * Persistent container logs
	
  ![alt](../../images/kientruc2.png)
  
- Containerd được giới hạn trên một host. Trong bài này, chúng ta sẽ thảo luận về hai tính năng chính: runtime và snapshotter.

## 2. Các thành phần:##

## 2.1. RunC

- runC là 1 container runtime, runC được viết bằng Go, và nó tuân theo đặc tả của OCI ở https://www.opencontainers.org. RunC sử dụng các công nghệ trong kernel Linux: `namespaces, cgroups, apparmor/selinux` để run container. 

## 2.2. Snapshotter

- Docker container sử dụng một system được biết như layers. các layer cho phép thực hiện sửa đổi một file system và lưu trữ chúng như một dạng thay đổi ở một base layer trên cùng. Trước đó Docker sử dụng graphdriver để tạo ra snapshots, tuy nhiên, hiện giờ containerd sử dụng snapshotter.

- Một snapshot là một trạng thái filesystem. Mỗi snapshot có một cha và khi không có cha sẽ là một chuỗi rỗng. Một layer chia tách giữa các snapshot. Khi một container được tạo, nó thêm vào một writeable layer (lớp có thể ghi) vào đỉnh của tất cả các layer. Tất cả các thay đổi được ghi vào writeable layer này. Writeable layer này là những gì khác biết của một container và một image.

- Tất cả các container sử dụng chung các layer cơ bản. Nếu một layer cần thay đổi, khi đó một layer mới được tạo ra như một bản copy của layer đó (và tất cả các layer trên đỉnh của nó) và thêm vào các thay đổi trong layer mới. Những layer mới này được hiển thị trong container yêu cầu thay đổi và những container khác vẫn sử dụng layer gốc. Nếu bạn kiểm tra hai image từ docker repository và nếu hai image có chung các base layer thì docker sẽ chỉ tải xuống các layer chung đó một lần.

  ![alt](../../images/snapshotter.png)

- Có nhiều loại driver khác nhau thực hiện chức năng này. Có nhiều loại driver khác nhau thực hiện chức năng này. Các storage driver được hỗ trợ là overlay2, aufs (sử dụng bởi các version cũ), devicemapper (CentOS và RHEL cũ), btrfs và zfs.

