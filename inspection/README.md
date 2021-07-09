# 部署

默认centos环境，如果系统不一致、请根据系统安装docker

## docker

### 1. 安装
```
yum install https://download.docker.com/linux/fedora/30/x86_64/stable/Packages/containerd.io-1.2.6-3.3.fc30.x86_64.rpm
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce
curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

### 2. 检查docker和docker-compose是否安装成功

```
docker version
docker-compose --version
```

### 3.文件授权

```
chmod +x /usr/local/bin/docker-compose
```

### 4.启动docker

```
systemctl start docker
```

## 打包编译文件

### 前端

```
npm run build:prod
```

### 后端

1. 因为使用docker部署，需要修改配置文件内的mysql和redis连接名称
   
    application.yml:

    ```
    # redis 配置
    redis:
        # 地址
        host: inspection-redis
    ```
    
    >把redis的host名称由localhost改为redis的容器名称inspection-redis

    application-druid.yml：

    ```
    druid:
    # 主库数据源
    master:
        url: jdbc:mysql://inspection-mysql:3306/inspection?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
        username: root
        password: inspection_123#
    ```

    >把mysql的url 由原本的ip改成mysql的容器名称inspection-mysql

    ```
    # 文件路径 示例（ Windows配置D:/monks/uploadPath，Linux配置 /home/monks/uploadPath）
    #profile: D:/monks/uploadPath
    profile: /home/inspection/uploadPath
    ```

    > 修改文件上传路径，不然需要修改dockerfile

2. 使用maven打成jar包

   > 确保jar包文件名为inspection-admin.jar。如果修改了，请改动dockerfile
   
## 组织部署文件

```
│  docker-compose.yml
│  inspection-dockerfile
│  mysql-dockerfile
│  nginx-dockerfile
│  README.md
│  redis-dockerfile
│
├─conf              #存放redis.conf和nginx.conf配置
│      nginx.conf
│      redis.conf
│
├─db                #存放数据库脚本，初始化数据库时执行
│      init.sql
│      inspection.sql
│
├─html
│  └─dist           #存放打包好的静态页面文件
└─jar               #存放打包好的jar应用文件
```

- 数据库mysql地址需要修改成inspection-mysql
- 缓存redis地址需要修改成inspection-redis
- 数据库脚本头部需要添加SET NAMES 'utf8';（防止乱码）

## 上传至服务器

上传到服务器指定目录。以后该目录作为日志和数据的存储目录

## 构建docker服务

确保当前目录为inspection文件夹下

```
docker-compose build
```

## 启动docker容器

```
docker-compose up -d
```

## 访问应用地址

打开浏览器，输入：( http://localhost:80 )。若能正确展示页面，则表明环境搭建成功。

> tips
>
> 启动服务的容器docker-compose up inspection-mysql inspection-server inspection-nginx inspection-redis
>
> 停止服务的容器docker-compose stop inspection-mysql inspection-server inspection-nginx inspection-redis

如果访问不通，请使用
```
docker ps -a
```
查看这四个容器哪一个没有正常启动。



### 初始执行时，因为数据库初始化较慢、需要耐心等待一会。如果容器状态都为正常，大概2分钟左右启动完毕（具体时间由机器配置影响）

