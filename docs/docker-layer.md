Tổng quan Union file system trong Docker
----------------------------------------

Để có thể hiểu được về mối quan hệ giữa images và containers, chúng ta cần phải hiểu về nền tảng công nghệ cơ bản giúp tạo nên Docker, đó chính là UFS (đôi khi gọi tắt là Union Mount). Union file system cho phép nhiều file system có thể nằm chồng lên nhau (hay chuyên ngành gọi là overlaid), trong khi nhìn từ phía user lại chỉ thấy một hệ thống file thống nhất.

Files thuộc nhiều hệ thống file khác nhau có thể cùng nằm trong một folder, tuy nhiên nếu 2 file có chung path, thì file được mounted sau cùng sẽ giấu đi những file được mount trước đó.

Docker hỗ trợ vài UFS implementations khác nhau, mà chúng ta có thể kể đến AUFS, Overlay, devicemapper, BTRFS, ZFS. Implementation nào được sử dụng tuỳ thuộc vào hệ thống đang chạy, mà bạn có thể check bằng docker info.

```
Client:
 Debug Mode: false

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 19.03.8
 Storage Driver: overlay2
  Backing Filesystem: <unknown>
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version:
 runc version:
 init version:
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 5.8.0-43-generic
 Operating System: Ubuntu 20.04.2 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 8
 Total Memory: 7.747GiB
 Name: ubuntu
 ID: IJFS:243G:N4NA:HUKX:X36A:TDE7:GZU3:TT72:TI53:KEJZ:LUYM:5SIP
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

```

Overlay2 driver và layer
------------------------

Cũng thử build 1 image từ Dockerfile sau:

```
FROM ubuntu:18.04
COPY . /app
CMD /app/test.sh

```

Đối với overlay2 driver các layer sẽ được lưu trữ lại /var/lib/docker/overlay2/ có dạng:

```
$ ls -l /var/lib/docker/overlay2
total 20
drwx------ 4 root root 4096 Feb 21 17:36 6a5673dc6361f49b840aab91920b1578d2613c4eb03e9014fbd6e68abd3b5222
drwx------ 4 root root 4096 Feb 21 14:32 8c88fe206eb369ee160d41d46d034e7fe68777759664fcfef8773a2cbdd8c255
drwx------ 4 root root 4096 Feb 21 17:36 a53bd2bdb994b7a8bcc6d8986d66f9f8c20172ee848b7a1997da626dbaf10cfd
drwx------ 3 root root 4096 Feb 21 14:32 f8de4636f1ca8d41e816c86f7ab1568076b0cfb38cd80982d5e5ca82e1d01b2f
drwx------ 2 root root 4096 Feb 21 17:36 l

```

Thư mục l chứa Symbolic Links là tên rút gọn ta thấy trong docker history link đến thư mục diff của các layer.

```
$ ls -l /var/lib/docker/overlay2/l
total 16
lrwxrwxrwx 1 root root 72 Feb 21 14:32 CI7RATO2LAA5GKHFQ64QYDI4PU -> ../8c88fe206eb369ee160d41d46d034e7fe68777759664fcfef8773a2cbdd8c255/diff
lrwxrwxrwx 1 root root 72 Feb 21 17:36 F4MFYDRSYIJY2RXPCRF5R7WYVG -> ../a53bd2bdb994b7a8bcc6d8986d66f9f8c20172ee848b7a1997da626dbaf10cfd/diff
lrwxrwxrwx 1 root root 72 Feb 21 14:32 KFEV7RAUKJDFB363AIPH2IAILO -> ../6a5673dc6361f49b840aab91920b1578d2613c4eb03e9014fbd6e68abd3b5222/diff
lrwxrwxrwx 1 root root 72 Feb 21 14:32 OHFY57GGE3GANKOVQO2Z57IGSP -> ../f8de4636f1ca8d41e816c86f7ab1568076b0cfb38cd80982d5e5ca82e1d01b2f/diff

```

Trong mỗi layer thường có dạng như sau:

```
$ ls /var/lib/docker/overlay2/a53bd2bdb994b7a8bcc6d8986d66f9f8c20172ee848b7a1997da626dbaf10cfd
diff  link  lower

```

Thư mục /diff chứa system file của layer:

```
$ ls /var/lib/docker/overlay2/a53bd2bdb994b7a8bcc6d8986d66f9f8c20172ee848b7a1997da626dbaf10cfd/diff
app

```

Thư mục /link chứa tên rút gọn của layer:

```
$ cat /var/lib/docker/overlay2/a53bd2bdb994b7a8bcc6d8986d66f9f8c20172ee848b7a1997da626dbaf10cfd/link
F4MFYDRSYIJY2RXPCRF5R7WYVG

```

Thư mục /lower chứa l PATH của những layer phía dưới :

```
$ cat /var/lib/docker/overlay2/a53bd2bdb994b7a8bcc6d8986d66f9f8c20172ee848b7a1997da626dbaf10cfd/lower
l/KFEV7RAUKJDFB363AIPH2IAILO:l/CI7RATO2LAA5GKHFQ64QYDI4PU:l/OHFY57GGE3GANKOVQO2Z57IGSP

```

Ngoài ra còn có thể có thêm một số layer khác như /merged, /work trong các layer tạo bới container. Bạn có thể tìm hiểu thêm [tại đây](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)

Container, Image và layer
-------------------------

Một Image được tạo thành từ một stack các read only layer

![](https://images.viblo.asia/5519fa20-abd6-42db-a88d-7430e108fed2.jpg)

Khi tạo một container từ image, một read/write layer sẽ được thêm vào trên cũng của stack. Khi có nhiều container được khởi tạo từ image, chúng sẽ sử dụng chung các layer phía dưới của image. Việc này giúp tiết kiệm tài nguyên cho hệ thống, khi container bị remove chỉ top layer bị loại bỏ, các layer của image sẽ không thay đổi.

![](https://images.viblo.asia/1c53c8eb-93a2-4eec-b774-249389c4db12.jpg)

Tài liệu tham khảo
------------------

<https://kipalog.com/posts/Docker-Images--Containers-va-Union-file-system> <https://docs.docker.com/storage/storagedriver/overlayfs-driver/> <https://docs.docker.com/storage/storagedriver/>
