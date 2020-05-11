# Cài đặt docker
*Tài liệu sẽ hướng dẫn cài đặt Docker trên linux (CentOS), tham khảo thêm cách cài đặt [Docker trên MAC](https://docs.docker.com/docker-for-mac/install/) và cài đặt [Docker trên Windows](https://docs.docker.com/docker-for-windows/install/)*
- Có hai phiên bản chính của Docker là Docker EE, và Docker CE.
 - Docker EE (Docker Enterprise Edition)
    Docker EE có 3 versions chính là Basic, Standard, Advanced. Bản Basic bao gồm Docker platform, hỗ trợ support và certification. Bản Standard và Advanced thêm các tính năng như container management (Docker Datacenter) và Docker Security Scanning.

    Docker EE được support bởi Alibaba, Canonical, HPE, IBM, Microsoft…

    Docker cũng cung cấp một certification để giúp các doanh nghiệp đảm bảo các sản phẩm của họ được hoạt động với Docker EE.
 - Docker CE (Docker Community Edition)
    Docker CE, đúng như tên gọi, nó là một phiên bản Docker do cộng đồng support và phát triển, hoàn toàn miễn phí.

    Có hai phiên bản của Docker CE là Edge và Stable. Bản Edge sẽ được release hàng tháng với các tính năng mới nhất, còn Stable sẽ release theo quý.
    *Phiên bản mà ta sử dụng để tìm hiểu là Docker CE do đó ta sẽ cài đặt Docker CE*
## Cài bản stable mới nhất
**Chuẩn bị** OS demmo: Centos 7 64bit

**Cài đặt gói cần thiết**

```sh
	yum install -y yum-utils
```

**Thêm repo docker**

```sh
yum-config-manager \
	--add-repo \
	https://download.docker.com/linux/centos/docker-ce.repo
```

**Cài đặt bản lastest của Docker CE**

```sh
yum install docker-ce docker-ce-cli containerd.io
```

**Khởi động docker**

```sh
systemctl start docker
```

**Kiểm tra trạng thái sau khi khởi động**

```sh
systemctl status docker
```

 - Kết quả hiện ra như sau là ok:
 
```sh
docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2020-05-11 14:45:41 +07; 5min ago
     Docs: https://docs.docker.com
 Main PID: 9473 (dockerd)
    Tasks: 10
   Memory: 43.9M
   CGroup: /system.slice/docker.service
           â””â”€9473 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/cont...

May 11 14:45:40 CentOS dockerd[9473]: time="2020-05-11T14:45:40.571270410+0...pc
May 11 14:45:40 CentOS dockerd[9473]: time="2020-05-11T14:45:40.571284519+0...pc
May 11 14:45:40 CentOS dockerd[9473]: time="2020-05-11T14:45:40.571291869+0...pc
May 11 14:45:40 CentOS dockerd[9473]: time="2020-05-11T14:45:40.761093609+0...."
May 11 14:45:41 CentOS dockerd[9473]: time="2020-05-11T14:45:41.151239409+0...s"
May 11 14:45:41 CentOS dockerd[9473]: time="2020-05-11T14:45:41.394334701+0...."
May 11 14:45:41 CentOS dockerd[9473]: time="2020-05-11T14:45:41.472753705+0....8
May 11 14:45:41 CentOS dockerd[9473]: time="2020-05-11T14:45:41.472870858+0...n"
May 11 14:45:41 CentOS dockerd[9473]: time="2020-05-11T14:45:41.683771233+0...k"
May 11 14:45:41 CentOS systemd[1]: Started Docker Application Container Engine.
Hint: Some lines were ellipsized, use -l to show in full.
```

**Kiểm tra hoạt động của docker**
```sh
docker pull hello-world
```
- Kết quả như sau:
	
```sh
		Using default tag: latest
		latest: Pulling from library/hello-world
		0e03bdcc26d7: Pull complete 
		Digest: sha256:8e3114318a995a1ee497790535e7b88365222a21771ae7e53687ad76563e8e76
		Status: Downloaded newer image for hello-world:latest
		docker.io/library/hello-world:latest
```

**Thử tạo container đầu tiên**

```sh
	docker run hello-world
```

- Kết quả như sau: 
```sh
		Hello from Docker!
		This message shows that your installation appears to be working correctly.

		To generate this message, Docker took the following steps:
		 1. The Docker client contacted the Docker daemon.
		 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
		 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
		 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

		To try something more ambitious, you can run an Ubuntu container with:
		$ docker run -it ubuntu bash

		Share images, automate workflows, and more with a free Docker ID:
		https://hub.docker.com/

		For more examples and ideas, visit:
		https://docs.docker.com/get-started/
```