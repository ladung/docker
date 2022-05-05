### 1. Giới thiệu

- Mặc định tất cả các file được tạo trong container sẽ được lưu ở writable container layer. Điều này có nghĩa rằng:

	- Dữ liệu sẽ không còn tồn tại khi mà container không còn chạy và rất khó để lấy một dữ liệu từ bên trong container ra bên ngoài nếu có một tiến trình khác cần nó.
	
	- Dữ liệu của container được gắn kết và lưu trữ mật thiết với Docker host. Vì thế mà không thể dễ dàng di chuyển dữ liệu đến một nơi khác.

	- Việc ghi dữ liệu vào một layer của container yêu cầu storage driver để quản lý filesystem. Việc này làm giảm hiệu suất so với việc sử dụng data volumes ghi trực tiếp vào filesystem trên Docker host.
	
- Docker cung cấp 3 cách khác nhau để có thể chia sẻ dữ liệu (mount data) từ Docker host tới container đó là: 

    - volumes.
	
	- bind mounts.
	
	- tmpfs mounts (chạy Docker trên Linux).

- Không có vấn đề nào xảy ra khi ta lựa chọn cách chia sẻ dữ liệu để sử dụng, vì các dữ liệu đều giống nhau trong container. Sự khác biệt giữa volumes, bind mounts và tmpfs mounts là sự khác nhau về vị trí lưu trữ dữ liệu trên Docker host:

  ![alt](../../images/persistenvolumes.png)
  
  - volumes được lưu trữ như một phần của filesystem trên Docker host và được quản lý bởi Docker (lưu ở /var/lib/docker/volumes trên Linux). Các tiến trình không phải của Docker ( Non-Docker process ) sẽ không có quyền thay đổi trên phân vùng này. Đây được xem là cách tốt nhất để duy trì dữ liệu trong Docker

  - bind mounts cho phép lưu trữ bất cứ đâu trong host system. Các tiến trình không phải của Docker có thể sửa đổi chúng bất cứ lúc nào.

  - tmpfs mounts cho phép dữ liệu chỉ được lưu vào memory của Docker host, không bao giờ ghi vào filesystem của Docker host.

### 2. Chi tiết về các kiểu Persistent storage

#### volumes: 

- Tạo và quản lý bởi Docker. Bạn có thể tạo 1 volume sử dụng lệnh `docker volume create`, hoặc tạo volume trong khi tạo container.

- Khi tạo 1 volume, nó sẽ được lưu trữ trong một thư mục trên Docker host. Khi ta thực hiện mount volumes, thì thư mục này sẽ được mount vào container. Điều này tương tự như cách bind mounts hoạt động, ngoại trừ volumes được quản lý bởi Docker quản lý.

- volumes có thể được mount vào nhiểu containers cùng một lúc. Khi không có containers nào sử dụng volumes thì volumes vẫn ở trạng thái cho phép mount vào containers và không bị xóa một cách tự động. Bạn có thể xóa các volume bằng lệnh `docker volume prune`.
	
- Khi mount 1 volume, bạn có thể đặt tên cụ thể cho nó hoặc không. Khi bạn thực hiện mounted volume ẩn danh (anonymous volumes) tới container, Docker sẽ đặt một tên ngẫu nhiên cho các volume này để đảm bảo nó là duy nhất trên Docker host. 
	
- volumes hỗ trợ volume drivers, do đó ta có thể sử dụng để lưu trữ dữ liệu từ remote hosts hoặc cloud providers.
	
#### bind mounts:

- Bind mounts có chức năng hạn chế so với volumes. Khi ta sử dụng bind mounts thì một file hoặc một thư mục trên Docker host sẽ được mount tới containers với đường dẫn đầy đủ.

- Khi thực hiện bind mount 1 thư mục trên Docker host vào container, nó được tạo ra theo yêu cầu nếu nó chưa tồn tại.
	
- Bạn không thể sử dụng Docker CLI để quản lí bind mounts.
	
- Một đặc điểm khi sử dụng bind mounts là bạn có thể thay đổi các host filesystem thông qua các tiến trình chạy trong container, bao gồm tạo, sửa hoặc xóa các tệp tin hoặc thư mục quan trọng của hệ thống. Đây là một khả năng có tác động mạnh tới vấn đề về bảo mật, ảnh hưởng tới các tiến trình non-Docker trên host system.
	
#### tmpfs mounts:
	
- Cho phép lưu trữ tạm thời dữ liệu vào memory của Docker host, nó dùng để lưu trữ các trạng thái không bền bỉ hoặc các thông tin nhạy cả. Ví dụ: swarm service sử dụng `tmpfs mounts` để lưu secret key của các service’s containers.

### 3. Các trường hợp sử dụng các kiểu persistent storage tương ứng.

### 3.1. Trường hợp nên sử dụng volumes

- Chia sẻ dữ liệu với nhiều containers đang chạy. Dữ liệu yêu cầu phải tồn tại kể cả khi dừng hoặc loại bỏ containers.

- Khi Docker host có cấu trúc filesystem không thống nhất, thường xuyên thay đổi.

- Khi muốn lưu trữ dữ liệu containers trên remote hosts, cloud thay vì Docker host.

- Khi có nhu cầu sao lưu, backup hoặc migrate dữ liệu tới Docker host khác thì volumes là một sự lựa tốt. Ta cần phải dừng containers sử dụng volumes sau đó thực hiện backup tại đường dẫn /var/lib/docker/volumes/<volume-name>

### 3.2. Trường hợp nên sử dụng bind mounts

- `Bind mounts` thích hợp được sử dụng trong một số trường hợp sau:
	
- Chia sẻ các file cấu hình từ Docker host tới containers.

- Chia sẻ khi các file hoặc cấu trúc thư mục trên Docker host có cấu trúc cố định phù hợp với yêu cầu của containers.

- Kiểm soát được các thay đổi của containers đối với filesystem trên Docker host. Do khi sử dụng bind mounts, containers có thể trực tiếp thay đổi filesystem trên Docker host.

### 3.3. Trường hợp nên sử dụng tmpfs mounts

- `tmpfs mounts` được sử dụng trong các trường hợp ta không muốn dữ liệu tồn tại trên Docker host hay containers vì lý do bảo mật hoặc đảm bảo hiệu suất của containers khi ghi một lượng lớn dữ liệu một cách không liên tục.

### 4. Cách sử dụng các kiểu persistent storage

### 4.1. Cách sử dụng volumes:

### Giới thiệu

- `Volumes` là cơ chế ưa thích cho việc lưu trữ các dữ liệu bền bỉ, được tạo và sử dụng bởi `Docker containers` và được quản lý bởi `Docker`. Trong khi `bind mounts` phụ thuộc vào cấu trúc thư mục của Docker host.

- `Volumes` có một số lợi thế hơn so với `bind mounts`:
	
	- `Volumes` dễ dàng backup, migrate hơn so với `bind mounts`.
	
	- Bạn có thể sử dụng Docker CLI hoặc Docker API để quản lí `volumes`.
	
	- `Volumes` làm việc trên cả Linux và Windows containers.
	
	- `Volumes` an toàn hơn khi sử dụng chia sẻ giữa các containers. 
	
	- Volume drivers cho phép lưu trữ volumes trên remote host hoặc cloud.

### Choose the -v or --mount flag:

- Ban đầu, `-v` hoặc `--volume` sử dụng cho các containers độc lập và `--mount` được sử dụng cho swarm services. Tuy nhiên, bắt đầu từ phiên bản Docker 17.06 trở đi, bạn có thể sử dụng `--mount` cho cả các container độc lập. Sự khác biệt lớn nhất là option `-v` kết hợp tất cả các tùy chọn lại với nhau trong một trường, trong khi với option `--mount` thì phân tách chúng.

- `-v` or `--volume`: Gồm có 3 trường, mỗi trường được phân cách nhau bằng `:`. Các trường này phải theo đúng thứ tự.

	- Trường đầu tiên là tên của volume, và nó là duy nhất trên host. Nếu bạn không đặt tên cho volume, Docker sẽ đặt một tên ngẫu nhiên cho các volume này.
	
	- Trường thứ 2 sẽ chỉ ra volume này sẽ được mount vào file, hay thư mục nào ở bên trong container.
	
	- Trường thứ 3 là tùy ý, và là một danh sách các tùy chọn, ví dụ như `ro`.

- `--mount`: Bao gồm nhiều cặp key-value, được phân tách bằng dấu phẩy và mỗi cặp bao gồm một `<key> = <value>`. Cú pháp `--mount` dài hơn so với `-v` hoặc `--volume`, nhưng không đáng kể và các flag của `--mount` là dễ hiểu hơn.
	
	- `type` trong mount, có thể là `bind`, `volume`, hoặc `tmpfs`. Bài viết này thảo luận về kiểu `type` là `volume`.
	
	- `source`: tên của volume trên host, có thể chỉ rõ là `source` hoặc `src`.
	
	- `destination`: chỉ ra volume này sẽ được mount vào file, hay thư mục nào ở bên trong container, có thể chỉ định là `destination`, `dst` hoặc `target`.
	
	- `readonly`: Nếu sử dụng option này, thư mục trong container được mount sẽ chỉ được phép đọc.
	
	- `volume-opt`: Option này có thể được chỉ định nhiều lần, 1 cặp `key-value` bao gồm tên tùy chọn và giá trị của nó.

### Tạo và quản lí volumes

- Create volume:

  ```
  docker volume create my-vol
  ```
- List volumes: 

  ```
  docker volume ls

  local               my-vol
  ```

- Inspect a volume:

  ```
  docker volume inspect my-vol
  [
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
  ]
  ```

### Khởi tạo container với volume

- Start container với volume sử dụng option `-v` (`--volume`).

  ```
  docker run -d --name=nginxtest -v nginx-vol:/usr/share/nginx/html:ro nginx:latest
  ```
- Start container với volume sử dụng option `--mount`.

  ```
  docker run -d --name=nginxtest --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly nginx:latest
  ```
### Khởi tạo service với volume

- Khi bạn start service và chỉ rõ volume, mỗi service container sẽ sử dụng local volume của riêng nó, và không chia sẻ dữ liệu với nhau.

- Các container trong 1 service có thể chia sẻ dữ liệu với nhau nếu ta sử dụng volume drivers hỗ trợ việc share storage.

- Ví dụ start service nginx, với 2 replicas ( 2 container trên các Docker host trong 1 cụm swarm ), mỗi 1 container sẽ sử dụng 1 local volume có tên là `myvol2`:

  ```
  docker service create -d \
  --replicas=2 \
  --name devtest-service \
  --mount source=myvol2,target=/app \
  nginx:latest
  ```
  
### Start container với volume sử dụng NFS server làm storage.

- Mặc định trên mỗi Docker host, container sẽ sử dụng local volume của riêng nó, và không chia sẻ dữ liệu với nhau. Trong phần này, mình sẽ giới thiệu cách cấu hình volume nằm trên NFS server, dữ liệu trong các container nằm trên các Docker host sẽ được đồng bộ với nhau.

- Mô hình 3 con ( chạy ubuntu 18.04 )

	* NFS-server: 192.168.99.111 (ubuntu 18.04)
	
	* Docker-host (nfs-client): 192.168.99.108 
	
	* Docker-host (nfs-client): 192.168.99.109

### Ở trên nfs-server:

- Install nfs-server:

  ```
  apt install nfs-kernel-server
  ```
- Tạo thư mục share:
  
  ```
  mkdir -p /var/docker
  
  chown -R nobody:nogroup /var/docker
  ```
- Cấu hình NFS Exports:
  
  ```
  vim /etc/exports
  /var/docker 192.168.99.108(rw,sync,no_subtree_check)
  /var/docker 192.168.99.109(rw,sync,no_subtree_check)
  ```
  
- Restart nfs-server:

  ```
  systemctl restart nfs-kernel-server
  ```

- Một lưu ý nhỏ là phải kiểm tra firewall của máy chủ NFS. Kiểm tra trạng thái của firewall

  ```
  ufw status
  ```
  
- Nếu ở trạng thái Status: inactive thì có thể bỏ qua việc thêm rule, còn nếu đang Status: active thì bổ sung các rule sau:

  ```
  ufw allow from 192.168.99.108 to any port nfs
  ```
  
  ``` 
  ufw allow from 192.168.99.109 to any port nfs
  ```

- Rule sẽ mở port 2049 trên firewall.
  
### Ở trên 2 Docker host ( nfs-client ):

  ```
  apt install nfs-common
  ```

- Tiếp theo, ta phải cài đặt một plugin cho docker volume để sử dụng được driver bên ngoài. Ở đây, tôi sử dụng plugin netshare

- Install docker-volume-netshare:

  ```
  wget https://github.com/ContainX/docker-volume-netshare/releases/download/v0.35/docker-volume-netshare_0.35_amd64.deb
  
  sudo dpkg -i docker-volume-netshare_0.35_amd64.deb
  ```
- Start service: 
  
  ```
  service docker-volume-netshare start
  ```
- Tạo volume:

  ```
  docker volume create -d nfs --name volnfs1 -o share=192.168.99.115:/var/docker
  ```
- List volume:
  
  ```
  docker volume ls
  
  DRIVER              VOLUME NAME
  local               nfsvolume
  local               vol1
  nfs                 volnfs1
  ```
  
- Start container ở trên host 1:

  ```
  docker run -d --name container1 -v volnfs1:/usr/share/nginx/html -p 80:80 nginx
  ```
- Start container ở trên host 2:
  
  ```
  docker run -d --name container2 -v volnfs1:/usr/share/nginx/html -p 80:80 nginx
  ```
  
- Thực hiện test thay đổi ở trên Docker host 1:

  ```
  docker exec -it container1 /bin/bash
  touch /usr/share/nginx/html/test.txt 
  ```
- Trên Docker host 2 ta kiểm tra:

  ```
  docker exec -it container2 /bin/bash
  ls /usr/share/nginx/html
  test.txt
  ```
  
- Thư mục lưu trữ các volume sử dụng backend storage nfs là:
  
  ```
  /var/lib/docker-volumes/netshare/nfs
  ```
- -> Ngoài các storage driver sẵn có (native) của docker volume bên trong docker-engine, ta còn có thể sử dụng các storage driver khác thông qua plugin. Sử dụng một external storage sẽ linh hoạt trong quá trình scale container ở nhiều host. Các container này cùng chia sẻ volume nên dữ liệu được đồng nhất.

### 4.2. Cách sử dụng bind mount

### Giới thiệu:

- So với volume thì bind mount có ít chức năng hơn. Khi ta dùng bind mount, ta có thể mount 1 file hoặc một thư mục lên container. File hoặc thư mục này được truy cập theo đường dẫn tuyệt đối trên máy host. Bind mount có hiệu năng truy xuất rất cao, nhưng phụ thuộc vào file system của máy host.

- Chú ý: khi dùng bind mount, các process trong container có thể thay đổi filesystem của máy host (tạo file, thêm xóa sửa các dữ liệu hoặc thư mục quan trọng của hệ thống). Tính năng này tuy mạnh nhưng có thể tạo ra nhiều nguy cơ về bảo mật, gây ảnh hưởng tới các process khác trên máy host.

### Khởi tạo container với bind mount

- Start container sử dụng flag `-v` (`--volume`):

  ```
  docker run -d \
  -it \
  --name devtest \
  -v /data/web:/var/www/html \
  nginx:latest
  ```

- Start container sử dụng flag `--mount`:

  ```
  docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source=/data/web,target=/var/www/html \
  nginx:latest
  ``` 
### 4.3. Cách sử dụng tmpfs mount

- Khi sử dụng tmpfs mounts thì ta không thể chia sẻ dữ liệu giữa containers.

- `tmpfs mounts` chỉ làm việc đối với Linux containers.

- Tương tự như khi sử dụng volumes, ta chỉ cần thay đổi giá trị type trong --mount:

  ```
  docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app \
  nginx:latest
  ```

### 4.4. Một số chú ý khi dùng bind mount hoặc volumes:

- Nếu mount một volumes trống vào 1 thư mục trên container, và trong thư mục tương ứng trên container đã có sẵn dữ liệu thì các file trên đó sẽ được copy vào volume. Tương tự vậy, nếu ta khởi động một container và yêu cầu một volume (volume này chưa tồn tại), Docker engine sẽ tạo ra một volume trống cho ta.
    
- Nếu mount một bind mount hoặc một volumes đã có dữ liệu vào một thư mục trên container và thư mục này cũng đã có dữ liệu, dữ liệu trên container sẽ bị “tạm thời thay thế” bởi dữ liệu mới mount. Khi ta unmount thì dữ liệu này sẽ hiện ra như cũ.
    
- Khi dùng cờ -v hoặc --volume để bind-mount một file hay thư mục chưa tồn tại trên Docker host thì Docker sẽ tạo ra một thư mục mới.
    
- Nếu dùng cờ --mount để bind-mount một file hay thư mục chưa tồn tại trên Docker host thì Docker không tự động tạo thư mục mới mà sẽ thông báo lỗi.

### 5. Tham khảo

- https://docs.docker.com/storage
- https://github.com/hocchudong/ghichep-docker/blob/master/docs/docker-coban/docker-volume.md
- https://github.com/ContainX/docker-volume-netshare
- http://netshare.containx.io/

	
	
	
