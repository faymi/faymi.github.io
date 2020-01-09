---
layout:     post
title:      Docker 搭建 Gitlab 服务
subtitle:   Docker/Gitlab/运维
date:       2020-01-09
author:     Faymi
header-img: img/gitlab.jpeg
catalog: true
tags:
    - Gitlab
    - Docker
    - 容器
---


## Docker 搭建 Gitlab 服务

### 前置条件
- docker 已安装
- docker compose （用到的话需要安装）
- linux 等主机环境


### 拉取 Gitlab 镜像

- [GitLab CE Docker 镜像 （社区版）](https://hub.docker.com/r/gitlab/gitlab-ce/)
- [GitLab EE Docker 镜像 （企业版）](https://hub.docker.com/r/gitlab/gitlab-ee/)

Gitlab Docker 镜像是在单个容器上运行所有必需服务的GitLab的整体镜像。如果你想用最新的RC镜像，可以分别用`gitlab/gitlab-ce:rc` or `gitlab/gitlab-ee:rc`。
下面实例中我们主要用社区版镜像。

docker 命令拉取 gitlab 镜像

```
docker pull gitlab/gitlab-ce
```

### 运行 Gitlab 镜像

```
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

如果您使用的是SELinux，请改为运行以下命令：

```
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab:Z \
  --volume /srv/gitlab/logs:/var/log/gitlab:Z \
  --volume /srv/gitlab/data:/var/opt/gitlab:Z \
  gitlab/gitlab-ce:latest
```

启动一个GitLab CE容器，并发布访问SSH，HTTP和HTTPS所需的端口。所有的GitLab数据都将存储为`/srv/gitlab/`的子目录。系统重启后，容器将自动重启。

命令解释：

`--detach` -> `-d` 此时容器会在后台运行并不会把输出的结果 (STDOUT) 打印到宿主机上面（可以通过 `docker container logs [container ID or NAMES]` 获取容器的输出信息）

`--hostname` -> `-h` 配置容器主机名

`--publish` -> `-p` 映射容器端口到宿主机端口 `宿主机Port:容器Port`

`--name` -> 容器名称

`--restart` -> 重启配置，`always`：系统重启时，容器也会自动重启。`no`：默认策略，容器退出时不重启；`no-failue:n`：在容器非正常退出时重启容器，最大重启n次；`unless-stopped`：在容器退出时总是重启容器，但不考虑在Docker守护进程启动时就已经停止了的容器。

`--volume` -> `-v` 数据卷映射 `宿主机目录:容器目录`

`gitlab/gitlab-ce:latest` -> gitlab 镜像

### Gitlab 数据存储位置

本地目录              | 容器目录          |  作用  
-|-|-
`/srv/gitlab/data`   | `/var/opt/gitlab` | 存储应用数据 
`/srv/gitlab/logs`   | `/var/log/gitlab` | 存储日志数据 
`/srv/gitlab/config` | `/etc/gitlab`     | 存储 gitlab 配置文件 

### Gitlab 配置

该容器使用官方的Omnibus GitLab软件包，因此所有配置都在唯一的配置文件 `/etc/gitlab/gitlab.rb` 中完成。

- 可以进入容器进行修改配置文件

    ```
    // 1.进入容器
    sudo docker exec -it gitlab /bin/bash

    // 2.修改配置文件
    sudo vi /etc/gitlab/gitlab.rb

    // 或者一步操作
    sudo docker exec -it gitlab editor /etc/gitlab/gitlab.rb
    ```

    修改配置完成后，需要重启容器以重新配置 gitlab

    ```
    sudo docker restart gitlab
    ```

    *注：每当容器启动时，GitLab都会重新配置自身。*

了解配置的更多信息，请查看 [Omnibus GitLab documentation](https://docs.gitlab.com/omnibus/settings/configuration.html)。

### 访问 Gitlab 服务

- 初始化进程可能需要一段时间，可以通过以下命令查看进程状态：

    ```
    // 通过查看容器日志
    sudo docker logs -f gitlab

    // 或者通过查看容器状态（当 STATUS 显示为 healthy 时即正常启动完成）
    sudo docker container ls --all
    ```

- 初始化完成后，访问服务 `http://localhost` ，重置 root 密码后再登录。

### 升级 Gitlab 到新版本

升级新版本需要重新构建容器，所以要先停止、删除需要升级的 gitlab 容器服务

1. 停止 gitlab 容器服务

    ```
    sudo docker stop gitlab
    ```

2. 移除 gitlab 容器

    ```
    sudo docker rm gitlab
    ```

3. 拉取新镜像

    ```
    sudo docker pull gitlab/gitlab-ce:latest
    ```

4. 使用先前指定的选项再次创建容器

    ```
    sudo docker run --detach \
    --hostname gitlab.example.com \
    --publish 443:443 --publish 80:80 --publish 22:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
    ```

### 在公共IP地址上运行Gitlab CE

通过修改 `--publish` 标志使 Docker 使用您的IP地址并将所有流量转发到 GitLab CE 容器。

例如将 gitlab 映射到某个IP如 198.51.100.1：

```
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 198.51.100.1:443:443 \
  --publish 198.51.100.1:80:80 \
  --publish 198.51.100.1:22:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

容器构建完成后，可以通过 http://198.51.100.1 和 https://198.51.100.1 访问服务了。

### 在不同的端口上暴露 GitLab

如果要使用与80（HTTP）或443（HTTPS）不同的主机端口，则需要向 `docker run` 命令添加单独的 `--publish` 指令。

例如要暴露 web 界面服务端口在`8829`，暴露SSH 服务端口到`2289`：

1. 首先运行 `docker run` 命令 

    ```
    sudo docker run --detach \
    --hostname gitlab.example.com \
    --publish 8929:8929 --publish 2289:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
    ```

2. 配置 `gitlab.rb`

    - 设置 `external_url`：

        ```
        # For HTTP
        external_url "http://gitlab.example.com:8929"

        or

        # For HTTPS (notice the https)
        external_url "https://gitlab.example.com:8929"
        ```

        此URL中指定的端口必须与Docker发布到主机的端口匹配。此外，请注意，除非在 `nginx ['listen_port']` 中显式设置了NGINX侦听端口，否则将从该URL中提取该端口。有关更多信息，请参考[NGINX文档](https://docs.gitlab.com/omnibus/settings/nginx.html)。

    - 设置 `gitlab_shell_ssh_port` （git仓库地址显示的端口，实际还是工作于22端口）：

        ```
        gitlab_rails['gitlab_shell_ssh_port'] = 2289
        ```
    
按照上面的示例，可以通过Web浏览器在`<hostIP>:8929`下访问GitLab，并在端口`2289`下使用SSH进行访问。

### 通过 docker-pompose 安装 Gitlab

1. [安装](https://docs.docker.com/compose/install/) Docker Compose 

2. 新建一个 `docker-compose.yml` 文件

    ```
    web:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'gitlab.example.com'
    environment:
        GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab.example.com'
        # Add any other gitlab.rb configuration here, each on its own line
    ports:
        - '80:80'
        - '443:443'
        - '22:22'
    volumes:
        - '/srv/gitlab/config:/etc/gitlab'
        - '/srv/gitlab/logs:/var/log/gitlab'
        - '/srv/gitlab/data:/var/opt/gitlab'
    ```

3. 确保在 `docker-compose.yml` 文件目录下运行 `docker-compose up -d` 去构建 gitlab 容器。

- 运行在自定义的 `HTTP` 和 `SSH` 端口:

    ```
    web:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'gitlab.example.com'
    environment:
        GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.example.com:8929'
        gitlab_rails['gitlab_shell_ssh_port'] = 2224
    ports:
        - '8929:8929'
        - '2224:22'
    volumes:
        - '/srv/gitlab/config:/etc/gitlab'
        - '/srv/gitlab/logs:/var/log/gitlab'
        - '/srv/gitlab/data:/var/opt/gitlab'
    ```

    和 `--publish 8929:8929 --publish 2224:22` 效果一样。

### 通过 docker-compose 升级 Gitlab

若是通过 `docker-compose` 安装Gitlab，只需运行 `docker-compose pull` 和 `docker-compose up -d` 即可下载新版本并升级您的GitLab实例。

### 问题处理

#### 500 Internal Error

更新Docker镜像时，可能会遇到所有路径都显示臭名昭著的`500`错误页。如果发生这种情况，请尝试运行`sudo docker restart gitlab`来重新启动容器并解决问题。

#### Permission 问题
从较早的GitLab Docker镜像进行更新时，可能会遇到权限问题。发生这种情况的原因是以前的镜像中的用户没有正确保留。该脚本可修复所有文件的权限。

要修复您的容器，只需执行更新权限，然后重启容器：

```
sudo docker exec gitlab update-permissions
sudo docker restart gitlab
```

#### Windows/Mac: Error executing action run on resource ruby_block[directory resource: /data/GitLab]

在Windows或Mac上将Docker Toolbox与VirtualBox一起使用并利用Docker卷时，会发生此错误。/ c / Users卷作为VirtualBox共享文件夹安装，并且不支持所有POSIX文件系统功能。不重新安装就无法更改目录所有权和权限，并且会导致GitLab失败。建议是切换到使用适用于您的平台的本地Docker安装，而不是使用Docker Toolbox。如果您不能使用本地Docker安装（Windows 10 Home Edition或Windows < 10），那么另一种解决方案是为Docker Toolbox的boot2docker设置NFS挂载而不是VirtualBox共享。

#### Linux ACL问题

如果您在Docker主机上使用文件ACL，则docker组需要对卷具有完全访问权限才能使GitLab正常工作。

```
$ getfacl /srv/gitlab
# file: /srv/gitlab
# owner: XXXX
# group: XXXX
user::rwx
group::rwx
group:docker:rwx
mask::rwx
default:user::rwx
default:group::rwx
default:group:docker:rwx
default:mask::rwx
default:other::r-x
```

如果这些都不正确，请使用以下命令进行设置：

```
sudo setfacl -mR default:group:docker:rwx /srv/gitlab
```