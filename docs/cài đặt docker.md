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