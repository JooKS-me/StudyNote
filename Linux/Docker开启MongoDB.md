获取镜像：

- `docker pull mongo`

运行MongoDB镜像：

- `docker run --name mongo -p 27017:27017 -v mongo:/data/db -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin -d mongo`

登录到MongoDB容器中：

- `docker exec -it mongo bash`

通过shell连接MongoDB：

- `mongo -u admin -p admin`