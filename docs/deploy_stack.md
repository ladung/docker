# Deploy một stack lên docker swarm
## Khái niệm về stack
- Một application thường được cấu thành từ nhiều service khác nhau. Ví dụ một web application thường bao gồm các service sau:
    - Web server
    - Database
    - Redis
- Các service cần thiết cho 1 application sẽ được mô ta trong  và được gọi là một stack.
- Để deploy một stack lên swarm, run comman docker stack deploy với input là một compose file.

## Tiến hành deploy stack
- Định nghĩa cấu trúc application
- Trong bài viết này mình sẽ tạo một web app đếm số lần user access và website, được cấu thành từ 2 service:
    - Python app
    - Redis
- Sau khi xác định được các service cần có của stack, việc tiếp theo đó là tiến hành build stack.

- Hãy đảm bảo các file sau được đặt trong cùng một folder trên node Manager.
**app.py**
```
from flask import Flask
from redis import Redis
app = Flask(__name__)
redis = Redis(host='redis-host', port=6379)
@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello World! I have been seen {} times.\n'.format(count)
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000, debug=True)
```
Đoạn code python xử lý công việc sau:

- Tạo một webpage in ra dòng chữ Hello World! I have been seen x times., - với x là số lần user access webpage
- Sử dụng redis để lưu số lần user access webpage. Chú ý address của redis host là redis-host.
- Publish webpage thông qua port 8000.
**requirements.txt**
File list các dependencies của python app
```
flask
redis
```
**Dockerfile**
```
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```
**docker-compose.yml**
```
version: '3'
services:
  web:
    image: dungla/stack-demo
    build: .
    ports:
      - "80:8000"
  redis-host:
    image: redis:alpine
```
Ở đây tên của redis service là redis-host, cũng chính là redis address được dùng ở file app.py. Với web service, port 8000 của service sẽ được map với port 80 của docker host.

**Build và publish image**
Để build image, tại folder /var/docker_demo, run command sau:
```
$ docker-compose build
...
Successfully built 51ed4ab952e7
Successfully tagged dungla/stack-demo:latest
```

**Deploy stack lên swarm**
```
$  docker stack deploy -c docker-compose.yml stack-demo
Ignoring unsupported options: build
Creating network stack-demo_default
Creating service stack-demo_web
Creating service stack-demo_redis-host
```
**List danh sách service của stack:**
```
docker stack services stack-demo
ID                  NAME                    MODE                REPLICAS            IMAGE                         PORTS
qkca6dy8skdd        stack-demo_redis-host   replicated          1/1                 redis:alpine
vcoaci43t89b        stack-demo_web          replicated          1/1                 dungla/stack-demo:latest   *:80->8000/tcp
```
- Như vậy là việc deploy stack đã hoàn tất, bây giờ ta có thể test app bằng curl
```
$ curl localhost
Hello World! I have been seen 1 times.
$ curl 172.31.30.109
Hello World! I have been seen 2 times.
$ curl 172.31.20.119
Hello World! I have been seen 3 times.
```

- Nhờ có routing mesh mà ta có thể access app thông qua bất cứ node nào trong swarm.
- Tiếp đến ta sẽ check xem các service này được deploy lên những node nào
```
$ docker service ps stack-demo_redis-host
ID                  NAME                      IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
jf0ezjjypbyn        stack-demo_redis-host.1   redis:alpine        ip-172-31-30-109    Running             Running 22 minutes ago
$ docker service ps stack-demo_web
ID                  NAME                IMAGE                         NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
e766xko4yhyo        stack-demo_web.1    dungla/stack-demo:latest   ip-172-31-27-110    Running             Running 22 minutes ago
```

## Chỉ định node cho service khi deploy
Mặc định, khi deploy một stack lên swarm, manager sẽ assign task của các service cho các node một cách random.

Có thể ở lần đầu tiên deploy, service redis-host được deploy lên node Worker1. Nhưng ở lần deploy tiếp theo có thể sẽ deploy lên node Worker2 hoặc Manager.

Trong thực tế, mỗi service sẽ yêu cầu cấu hình host khác nhau. VD: mysql yêu cầu tốc độ đọc/ghi ổ cứng cao, redis lại yêu cầu về RAM nhiều hơn.

Để giải quyết vấn đề này, docker cho phép ta chỉ định node để deploy service thông qua config placement.
**update docker-compose.yml**
```
version: '3'
services:
  web:
    image: dungla/stack-demo
    build: .
    ports:
      - "80:8000"
    deploy:
      placement:
        constraints:
         - "node.role==worker"
         - "node.labels.type.==webs-erver"
  redis-host:
    image: redis:alpine
    deploy:
      placement:
        constraints:
         - "node.role==worker"
         - "node.labels.type==redis-server"

```
- Ở đây mình mình config placement với rule như sau:

    - web: chỉ deploy lên node với role là worker và có gắn lable type=web-server
    - redis-host: chỉ deploy lên node với role là worker và có gắn lable name=redis-server

- Tiếp đến mình sẽ gán lable cho 2 node worker:

    - Gán lable type cho node Woker1 với giá trị là web-server
    - Gán lable type cho node Woker2 với giá trị là redis-server

```
$ docker node update --label-add type=web-server ip-172-31-30-109
ip-172-31-30-109
$ docker node update --label-add type=redis-server ip-172-31-20-119
ip-172-31-20-119
```

Cuối cùng deploy stack mới lên
```
$ docker stack deploy -c docker-compose.yml stack-demo-2
Ignoring unsupported options: build
Creating network stack-demo-2_default
Creating service stack-demo-2_redis-host
Creating service stack-demo-2_web
```

```
$ docker service ps stack-demo-2_redis-host
ID                  NAME                        IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
rj3jnejr702w        stack-demo-2_redis-host.1   redis:alpine        ip-172-31-20-119    Running             Running 53 seconds ago
$ docker service ps stack-demo-2_web
ID                  NAME                 IMAGE                         NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
pq9x35beany1        stack-demo-2_web.1   dungla/stack-demo:latest   ip-172-31-30-109    Running             Running about a minute ago
```