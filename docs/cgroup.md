## Cgroups trong Docker ###

### I. Giới thiệu về Cgroups

- Từ phiên bản 2.6.24, kernel linux có thêm tính năng mới gọi là “control groups” thường được gọi là “cgroups”. Cgroups cho phép bạn có thể giới hạn việc sử dụng tài nguyên của hệ thống – như giới hạn số CPU, memory, băng thông mạng.

- Sử dụng cgroups, tài nguyên phần cứng sẽ được chia sẽ hiệu quả giữa các ứng dụng và tăng khả năng ổn định của hệ thống.

- Trong cgroups, các tài nguyên hệ thống được gọi bằng thuật ngữ “subsystem” hay “resource controller” và các tiến trình trên hệ thống được gọi là “task”.

- Các subsystem phổ biến trên cgroups:
	
	* `cpu`: sử dụng OS scheduler để cấp phát CPU cho các “task”.

	* `cpuacct`: báo cáo về trình trạng sử dụng CPU của các “task”.
	
	* `cpuset`: giới hạn việc sử dụng số lượng CPU trên hệ thống nhiều CPU.
	
	* `memory`: giới hạn việc sử dụng bộ nhớ hệ thống.
	
	* `blkio`: giới hạn việc truy cập nhập/xuất(I/O) đến các thiết bị như ổ đĩa cứng.
	
	* `net_prio`: giới hạn băng thông mạng theo độ ưu tiên.
	
	* `pids`: giới hạn số lượng process được tạo.

### II. Docker sử dụng cgroups để giới hạn memory, cpu của container.

**1. Memory.**

- Mặc định, thì container sẽ không bị giới hạn tài nguyên của host. Docker sử dụng công nghệ cgroups của kernel linux để có thể kiểm soát memory, cpu mà 1 container sử dụng.
- Dùng lệnh `docker run với các options` để giới hạn mem, cpu khi khởi chạy 1 container.
  
  ```
  docker run --help
  ```
- Các tính năng để thực hiện việc limit tài nguyên của 1 container yêu cầu linux kernel phải hỗ trợ `capabilities`. Check xem `capability` có đang được enable trong kernel:
  
  ```
  docker info
  ```
- Nếu output của lệnh trên trả về: 

  ```
  WARNING: No swap limit support
  ```
- Ta thực hiện enable `capability` trong kernel, thực hiện edit file `/etc/default/grub`:

  ```
  GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
  ```
  
- Lưu lại file trên và update GRUB:

  ```
  sudo update-grub
  ```

**1.1. Understand the risks of running out of memory.**

- Ở trong Linux hosts, nếu kernel phát hiện không đủ mem để thực hiện các functions quan trọng của hệ thống, nó sẽ thực hiện kill các tiến trình để giải phóng bộ nhớ của hệ thống. Các tiến trình bị kill ở đây, có thể bao gồm Docker và cả các ứng dụng quan trọng khác. 

-> Đây gọi là hiện tượng out of memory (OOM).

- Docker cố gắng giảm thiểu những rủi ro này bằng cách điều chỉnh mức ưu tiên OOM trên daemon Docker để nó ít bị kill hơn các tiến trình khác trên hệ thống. OOM priority trên container không được điều chỉnh. Điều này làm cho nhiều khả năng một vài container riêng lẻ bị kill, hơn là cho daemon Docker hoặc các quá trình hệ thống khác bị kill. 

- Bạn có thể giảm thiểu những rủi no này bằng cách:
	
	* Thực hiện tests để biết được số memory cần cấp cho 1 ứng dụng là bao nhiêu, trước khi đưa vào chạy production.
	
	* Đảm bảo ứng dụng của bạn chỉ chạy trên các máy chủ đáp ứng đủ tài nguyên.
	
	* Giới hạn số memory cho ứng dụng của bạn, cụ thể ở đây là số mem cấp cho container, phần này mình sẽ mô tả kĩ hơn ở bên dưới.
	
	* Config swap trên hosts. Swap thì hiệu năng của nó sẽ chậm hơn memory ( vì nó lấy 1 phần lưu trữ trên disk ), nhưng nó có thể giúp tránh được hiện tượng out of memory.
	
	* Consider converting your container to a service, and using service-level constraints and node labels to ensure that the application runs only on hosts with enough memory.
	
**1.2 Limit a container’s access to memory**

- Docker có thể thực hiện giới hạn memory, soft limits của host để cấp cho 1 container. ( Hard limits là số open file tối đa cho phép mỗi user mở trong phiên đăng nhập. Nó được thiết lập bởi superuser/root. Giá trị này được thiết lập trong file /etc/security/limits.conf. Soft limits cũng giống như hard limits, nhưng mỗi user có thể tự thiết lập được giá trị này để phù hợp với yêu cầu riêng của từng user, giá trị thiết lập cho soft limit thì không được cao hơn hard limit.)

- Các options thiết lập để limit access to memory:


  | Options              | Mô tả                                                                                                                                                                                                                                                                                                                                                                           |
  | -------------------- |:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------     |
  | -m or --memory=      | Số memory tối đa sẽ cấp cho 1 container. Ví dụ ta cấp tối đa 1 container được sử dụng 300mb:docker run -it --memory=”300m” ubuntu /bin/bash                                                                                                                                                                                                                                     |
  | --memory-swap        | Dung lượng bộ nhớ swap cấp cho 1 container.                                                                                                                                                                                                                                                                                                                                     |
  | --memory-reservation | Cho phép bạn chỉ định soft limit nhỏ hơn –memory, nó được kích hoạt khi Docker phát hiện sự tranh chấp hoặc bộ nhớ thấp trên máy chủ. Nếu bạn sử dụng --memory-reserved, giá trị này phải được set thấp hơn --memory để nó được ưu tiên. Bởi vì nó là 1 soft limit, nó không đảm bảo rằng container không vượt quá giới hạn.                                                     |
  | --kernel-memory      | Dung lượng tối đa của kernel memory mà 1 container có thể sử dụng. Giá trị min cho phép: 4m. Do kernel memory không thể sử dụng bộ nhớ swap, nên nếu kernel memory của 1 container không đủ, thì nó sẽ block các tài nguyên ở trên host, và sẽ ảnh hưởng tới host và các containers khác                                                                                        |
  | --oom-kill-disable   | Mặc định, nếu xảy ra lỗi out of mem, kernel sẽ thực hiện kill các tiến trình của 1 container. Để thay đổi điều này, ta có thể sử dụng options –oom-kill-disable. Ta chỉ disable  oom ở trên container nào có set -m/--memory option. Nếu flag -m không được set, thì host có thể hết bộ nhớ và kernel có thể sẽ phải kill các tiến trình hệ thống của host để giải phóng bộ nhớ.  |

- Sau đây mình sẽ giải thích rõ hơn một số options ở trên

  ```
  --memory-swap
  ```
  
  - --memory-swap chỉ được sử dụng khi option --memory được set. Options này cho phép sử dụng bộ nhớ swap trên disk của host khi mà 1 container đã sử dụng hết số lượng RAM mà container đó được cấp.

  - Ví dụ nếu --memory="300m" và --memory-swap="1g", thì container đó sẽ sử dụng 300m của memory và 700m ( 1g - 300m ) của swap. Vì thế ta cần lưu ý, nếu ta set 2 giá trị --memory và --memory-swap này bằng nhau thì container sẽ không sử dụng bộ nhớ swap. 	

  - --memory-swap="0", thiết lập này được hiểu là ta sẽ không sử dụng bộ nhớ swap.

  - --memory-swap unset và --memory is set thì container sẽ sử dụng bộ nhớ swap gấp 2 lần số memory được thiết lập trong option --memory ở trên. Ví dụ: nếu --memory="300m", và ta không set --memory-swap, thì container sẽ sử dụng 300m của memory và 600m của swap.

  - --memory-swap set "-1", thì container sẽ được cho phép sử dụng unlimited swap của host system.

  - Ở trong mỗi container, bạn có thể sử dụng lệnh "free" để check.

  ```
  --kernel-memory details
  ```

  - Kernel memory limits được thể hiện dưới dạng bộ nhớ tổng thể được phân bổ cho một container. Nó được mô tả theo các kịch bản sau đây:
  
  		* Unlimited memory, unlimited kernel memory: Đây là hành vi mặc định.
		* Unlimited memory, limited kernel memory: Điều này phù hợp khi dung lượng memory cần thiết cho tất cả các cgroups lớn hơn dung lượng memory thực sự tồn tại trên host. Bạn có thể định cấu hình kernel memory để không bao giờ vượt quá memory trên host và các containers cần thêm memory cần phải chờ.
		* Limited memory, unlimited kernel memory: Tổng thể mem bị giới hạn, nhưng kernel memory thì không.
		* Limited memory, limited kernel memory: Hữu ích để debug các vấn đề liên quan đến bộ nhớ. Nếu 1 container sử dụng mem không như ý muốn, nó sẽ bị out of memory nhưng sẽ không ảnh hưởng tới các container khác hoặc host. Trong cài đặt này, nếu giới hạn của kernel memory thấp hơn user memory limit, việc hết kernel memory sẽ khiến container gặp lỗi OOM. Nếu giới hạn kernel memory cao hơn user memory limit, việc giới hạn kernel sẽ không khiến container gặp phải lỗi OOM.
		
**2. CPU**

- Mặc định, thì mỗi container sẽ không bị giới hạn việc sử dụng cpu của host. Bạn có thể đặt các ràng buộc khác nhau để giới hạn quyền truy cập vào các chu kỳ CPU của host.

- Hầu hết user sử dụng và cấu hình default CFS scheduler. Từ phiên bản Docker 1.13 trở lên, bạn có thể cấu hình the realtime scheduler để giới hạn việc sử dụng cpu.

**2.1. Configure the default CFS scheduler**

- CFS là bộ lập lịch Linux kernel CPU cho các tiến trình Linux thông thường. Một số runtime flag cho phép bạn chỉ định việc cấu hình lượng truy cập vào tài nguyên CPU mà container của bạn có. Khi bạn sử dụng các cài đặt này, Docker sẽ sửa đổi các cài đặt container's cgroup trên máy chủ ( Trên host tạo ra Docker cgroup, mình sẽ chỉ ra ở phần demo cuối bài ).

- Một số Options:
  
  ```
  --cpus=<value>
  ```
  
- Chỉ định tài nguyên CPU mà 1 container được sử dụng. Nếu host machine có 4 CPUs và bạn set --cpus="2", thì container sẽ chỉ sử dụng tối đa 2 cpus của host. Options ""--cpus" này có sẵn từ phiên bản Docker 1.13 trở lên, nó tương đương với việc ta thiết lập đồng thời 2 giá trị --cpu-period="100000" và --cpu-quota="200000". | 
 
  ```
  --cpu-period=<value>
  ```
  
- Chỉ định CPU CFS lập lịch khoảng thời gian xử lý 1 tiến trình của CPU, nó được sử dụng kèm theo option --cpu-quota. Default nó được set giá trị cpu-period=100000µs (= 100 micro-seconds). Chú ý, option này ta nên để default. Nếu bạn sử dụng Docker phiên bản từ 1.13 trở lên, thì sẽ được thay thế bởi --cpus.

  ```
  --cpu-quota=<value>
  ```
  
- Chỉ định quota sử dụng CPU của 1 container, nó được sử dụng đi kèm với option --cpu-period. Nếu bạn sử dụng Docker phiên bản từ 1.13 trở lên, thì sẽ được thay thế bởi --cpus.

  ```
  --cpuset-cpus
  ```
  
- Chỉ định rõ container sẽ được sử dụng CPU thứ bao nhiêu. Ví dụ: Host có 4 cpu với số thứ tụ từng CPU là: 0, 1, 2, 3. Bạn có thể dùng option này để chỉ rõ container sẽ dùng cpu số 2 chẳng hạn.

**2.2 Configure the realtime scheduler**

- Ngoài ra, từ phiên bản Docker 1.13 trở lên, bạn có thể cấu hình realtime scheduler, để tùy chỉnh việc sử dụng CPU, độ ưu tiên sử dụng CPU cho 1 container nào đó trong Docker. 

- *Note: Tuy nhiên, CPU scheduling và prioritization là những tính năng nâng cao của kernel, các bạn nên để default. Nếu việc thiết lập sai các giá trị này, có thể là nguyên nhân khiến cho host system của bạn chạy không ổn định hoặc thậm chí là có thể không chạy được.*

**3.Demo**

- Mô tả: Thực hiện test tạo thử container giới hạn mem, cpu cho container đó.

- Mục tiêu: Kiểm tra xem Docker sử dụng công nghệ cgroups của Linux kernel cho từng container như nào.

- Thực hiện cài tool `lscgroup` để check
  
  ```
  apt install cgroup-bin cgroup-lite libcgroup1
  ```
- Khi thực hiện cài đặt docker, thì trên host sẽ tạo ra 1 `cgroup Docker`. Ta sử dụng `lscgroup` để check:
  
  ```
  root@vagrant-ubuntu-trusty-64:/sys/fs/cgroup# lscgroup | grep docker
  cpuset:/docker
  cpu:/docker
  cpuacct:/docker
  memory:/docker
  devices:/docker
  freezer:/docker
  blkio:/docker
  perf_event:/docker
  hugetlb:/docker
  ```
-> Ta thấy được `resource controller` phổ biến trên `cgroups` mà mình đã nói ở trên, các resource controller này được lưu trong: **/sys/fs/cgroup**, ở đây mình nói về việc limit memory và cpu của 1 container trong Docker:

  ```
  /sys/fs/cgroup/memory/docker
  /sys/fs/cgroup/cpu/docker
  ```
   
- Trước khi khởi tạo 1 container, các bạn có thể list 2 thư mục trên để thấy được sự khác nhau trước và sau khi khởi tạo 1 container. 

- Tạo 1 container và giới hạn 200mb memory, giới hạn số cpus sử dụng là: 0,5 ( Ở đây mình dùng docker version 1.12. )
  
  ```
  docker run --name test_mem_cpu -m 200m --cpu-period=100000 --cpu-quota=50000 -d nginx_image
  ```
- List container vừa khởi tạo:
  
  ```
  docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
  dd8a4ee940ca        nginx_image         "nginx -g 'daemon off"   8 minutes ago       Up 8 minutes        80/tcp              test_mem_cpu
  a94ad6aba609        nginx_image         "nginx -g 'daemon off"   6 days ago          Up 6 minutes        80/tcp              zen_albattani
  ```
- Check limit mem của container đó:

  ```
  docker stats 
  CONTAINER           CPU %               MEM USAGE / LIMIT       MEM %               NET I/O             BLOCK I/O             PIDS
  dd8a4ee940ca        0.07%               4.066 MiB / 200 MiB     2.03%               1.296 kB / 648 B    45.06 kB / 4.096 kB   0
  a94ad6aba609        0.05%               2.762 MiB / 993.8 MiB   0.28%               648 B / 648 B       0 B / 0 B   
  ```
  
- Check cgroup vừa được khởi tạo bởi Docker được áp dụng để limit mem, cpu đối với container vừa khởi tạo ở trên:

	* Check cpu: 
  	  
	  ```
	  cat /sys/fs/cgroup/cpu/docker/dd8a4ee940ca8135fc2522286e95dc1ccd877b2de2c463494c5b3f194d7e015f/cpu.cfs_period_us 
	  100000
	  cat /sys/fs/cgroup/cpu/docker/dd8a4ee940ca8135fc2522286e95dc1ccd877b2de2c463494c5b3f194d7e015f/cpu.cfs_quota_us 
	  50000
  	  ```
	  
    * Check memory:
	  
	  ```
	  cat /sys/fs/cgroup/memory/docker/dd8a4ee940ca8135fc2522286e95dc1ccd877b2de2c463494c5b3f194d7e015f/memory.limit_in_bytes 
      209715200 ( 209715200bytes = 200mb )
	  ```

### Tổng kết

- Trên đây là những quá trình tìm hiểu của mình về công nghệ cgroups được sử dụng trong Docker như nào. Các bạn có thể tham khảo chi tiết thêm ở link sau:

- *https://mrhien.info/blog/gioi-thieu-ve-linux-control-groups-cgroups/*
 
- *https://docs.docker.com/config/containers/resource_constraints*
