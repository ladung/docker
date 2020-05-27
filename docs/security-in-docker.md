# Namespaces
- Docker sử dụng công nghệ namespace để cung cấp không gian làm việc cô lập cho container. Khi run một container, docker tạo 1 namespace cho container.
- Docker sử dụng các namespace sau trên Linux:
  - pid namespace: Docker sử dụng pid namespace cho việc cô lập process trên mỗi container. Mỗi container có cây process cho riêng mình. 1 container không thể thấy hoặc truy cập cây process của container khác hay trên host mà nó đang chạy.
  - net namespace (Network namespace):Docker sử dụng network namespace để cung cấp mỗi container có network stack riêng gồm: interface, IP address, port, route table.
  - ipc namespace (Inter-process Communication namespace): Docker sử dụng ipc namespace cho việc chia sẻ bộ nhớ truy cập bên trong container.
  - mnt namespace: Mỗi container có một hệ thống filesystem cô lập riêng. Process trong một container không thể truy cập mount namespace riêng của các các container khác.
  - user namespace: Tách biệt user trong container và host. VD: quyền của user root trong container có thể map với user thường trên host.
  - uts namespace: cung cấp mỗi container có một hostname của riêng nó.

# Control group
- Docker cũng sử dụng kernel control group để phân bổ và cô lập resource.Một cgroup giới hạn một ứng dụng cho một tập resource nhất định. Control groups cho phép Docker Engine chia sẻ những tài nguyên phần cứng cho các containers và cũng có thể đảm bảo giới hạn những tài nguyên này hay thiết lập các constraints.
- Namespace là sự cô lập thì control group (cgroups) cung cấp cơ chế giới hạn, giám sát tài nguyên hệ thống.
- Container cô lập lẫn nhau nhưng chia sẻ chung tài nguyên OS: RAM, CPU,.. Cgroups sẽ giới hạn tài nguyên cho mỗi container.

Ví dụ:
- Giới hạn tốc độ đọc 1mb/s cho container: `docker run -it --device-read-bps /dev/sda:1mb centos`
- Kiểm tra: 

```sh
[root@7-0-0-66 c57b145e32c5cee9cc95a8d9c775fbee6070d0e998eb75f1a6401bbaa527c521]# cat blkio.throttle.read_bps_device 
8:0 1048576
```
- Docker Engine sử dụng những cgroups sau:
	- Memory cgroup cho việc quản lý bộ nhớ
	- HugeTBL cgroup để quản lý việc sử dụng hugepages bởi process group.
	- CPU group để quản lý user/system CPU time và usage
	- CPUSet cgroup để đo đạc và giới hạn lượng blckIO bởi group
	- net_cls và net_prio cgroup để gắn thẻ traffic control
	- Devices cgroup để đọc và ghi các access devices
	- Freezer cgroup để đóng băng 1 group
# Capabilities