[toc]

# 正名

- Docker 是一個平台即服務（platform as a service, PaaS）產品，這個產品使用作業系統級別 (operating system-level) 的虛擬化 (virtualization) 技術來交付軟體，也稱容器化 (Containerization)
- Docker, Inc. 是一間推廣 Docker 軟體的公司

# 什麼是 Docker？

Docker 字面上的意思是「碼頭工人」，在碼頭上會有打包、運送…等服務，碼頭工人 (Docker) 可快速的用貨櫃 (Container) 將貨物 (Application) 裝上船

![](https://i.imgur.com/Mr9AHqV.jpg)

- Docker 是一種容器化技術 (Containerization)，可以把你的應用程式連同環境一起打包，部署的時候不用擔心環境的問題
- Docker 的哲學是 **Build, Ship and Run any Application Anywhere**
- 原始碼在 GitHub 上進行維護，專案名稱為 [moby](https://github.com/moby/moby)
- 使用 Go 語言開發

# Docker 能做什麼？

- 一致性的發佈環境
- 在同一台機器運行更多的工作
- 基礎設施即代碼 (Infrastructure as Code) 及軟體定義網路
- 可攜性發佈和按需式縮放 (scaling)
    ![](https://img.scoop.it/YRf2mZDL6fRHIohz12etd7nTzqrqzN7Y9aBZTaXoQ8Q=)

# Containers vs Virtual Machines

容器 (container) 與虛擬機 (virtual machine, vm) 都是一種部署應用程式的方法，功能都是在隔離環境及底層硬體

主要差異在隔離等級（容器是將作業系統層虛擬化 (Containerization)，虛擬機器則是虛擬化硬體 (Virtualization)）

將作業系統核心虛擬化，可以允許使用者空間軟體實體（instances）被分割成幾個獨立的單元，在核心中運行，而不是只有一個單一實體運行

![](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/bltb6200bc085503718/5e1f209a63d1b6503160c6d5/containers-vs-virtual-machines.jpg)

![](https://docs.microsoft.com/zh-tw/virtualization/windowscontainers/about/media/virtual-machine-diagram.svg)
![](https://docs.microsoft.com/zh-tw/virtualization/windowscontainers/about/media/container-diagram.svg)

隔離 Networks, Filesystems, Processes, 共享相同的 kernel

```bash
docker container run --detach --publish 80:80 --name hello-world docker/getting-started

curl -v 127.0.0.1:80
ls -al /etc/ | grep -i 'nginx'
ps -auxw | grep nginx
```

| --- | Virtualization | Containerization |
| --- | --- | --- |
| 啟動  | 慢 (minute) | 快 (second) |
| 容量  | 大 (GB) | 小 (MB) |
| 效能  | 慢   | 快   |
| 主機可支撐的數量 | 數個～數十個 | 數個～數百個 |

```bash
vagrant up

docker run --rm debian:9 bash
```

# 如何安裝 Docker

- online
    - [play-with-docker](https://labs.play-with-docker.com/)
- ubuntu 1804
    - install using the repository
    - install using the convenience script (it is not recommended for production environments)
    - install docker compose cli
    - install docker & docker-compose auto completion
    - use docker as non-root user, add user to docker group (no security way, but convenient way)
- [others](https://docs.docker.com/engine/install/)

## ubuntu 1804

### repository

```bash
sudo apt-get update \
  && sudo apt-get install -y --no-install-recommends \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
  && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
  && echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null \
  && sudo apt-get update \
  && sudo apt-get install -y --no-install-recommends \
    docker-ce \
    docker-ce-cli \
    containerd.io \
  && sudo docker version
```

### convenience script

```bash
URL="https://get.docker.com"
curl -fsSL ${URL} | sh
```

### docker compose cli

```bash
VERSION=$(curl -sL https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d'"' -f 4)
VERSION=1.27.4
URL="https://github.com/docker/compose/releases/download/${VERSION}/docker-compose-$(uname -s)-$(uname -m)"
OUTPUT="/usr/local/bin/docker-compose"
sudo curl -fsSL ${URL} -o ${OUTPUT}
sudo chmod +x ${OUTPUT}
```

### auto completion

```bash
VERSION=$(curl -sL https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d'"' -f 4)
VERSION=1.27.4
URL="https://raw.githubusercontent.com/docker/compose/${VERSION}/contrib/completion/bash/docker-compose"
OUTPUT="/etc/bash_completion.d/docker-compose"
sudo curl -fsSL ${URL} -o ${OUTPUT}
sudo apt-get install -y -qq --no-install-recommends bash-completion
```

### use docker as non-root user, add user to docker group (no security way, but convenient way)

```bash
sudo usermod -aG docker ${USER}
```

see more -> [click me!](https://docs.docker.com/engine/security/#docker-daemon-attack-surface)

# Docker 基本概念

- Images
- Containers
- Repository & Registry
- Volumes
- Networks

## Images

Image 包裝了一個執行特定環境所需要的資源，由 Dockerfile 建立而來的，相同的 Dockerfile 會建立出相同的 Image

每個 Image 都有獨一無二的 digest，這是從 Image 內容做 sha256 產生的，這個設計能讓 Image 無法隨意改變內容，維持資料一致性，僅提供讀內容

雖然 Image 裡有必要的資源，但它無法獨立執行，必須要靠 Container 間接執行

## Containers

基於 Image 可以建立出 Container (runtime instance)

一個 Image 可以建立出相同多個不同的 Containers

Container 的概念像是建立一個可讀寫內容的外層，架在 Image 上，實際存取 Container 會經過可讀寫層與 Image，因此看到的內容會是兩者合併後的結果

Container 特性跟 Image 不一樣，因為有可讀寫層，所以 Container 可以讀寫，也可以拿來執行

## Repository & Registry

Repository 是存放 Image 的集合

Registry 是一項存儲多個 Image 的服務，可以由第三方託管，也可以由私人託管

- [Docker Hub](https://hub.docker.com/)
- [Quay.io](https://quay.io/)
- [AWS Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/)
- [Google Container Registry (GCR)](https://cloud.google.com/container-registry)
- [Azure Container Registry](https://azure.microsoft.com/services/container-registry/)

## Volumes

持久化資料

container 內的持久化空間，等同於本機空間

see more -> [click me!](https://docs.docker.com/storage/)

## Network

drivers

- bridge
- host
- none
- ...

### bridge

如果沒有指定 driver，預設使用 bridge driver

通常在需要網路的 Container 中運行時，通常會使用 bridge network

### host

對於 Container 來說，會刪除與主機之間的網路隔離，然後直接使用主機的網路

see more -> [click me!](https://docs.docker.com/network/)

# 常見的 Dockerfile 指令

- ADD
- COPY
- ARG
- ENV
- CMD
- ENTRYPOINT
- FROM
    - 用來指定這個 image 使用的 base image 以讓後面接續使用
- LABEL
    - 用來替 image 添加 metadata
- RUN
    - 會在現有 image 之上加入新的一層，指令於該層被執行並提交結果
- USER
    - 用來指定後續 RUN, CMD 及 ENTRYPOINT 指令或容器中的 user (UID), group (GID)
- EXPOSE
    - 用來指定此 container 在執行時要 publish 的 port
- VOLUME
    - 用來建立一個掛載點，以掛載本機或其他容器的卷宗
- WORKDIR
    - 用來設定在 RUN, CMD, ENTRYPOINT, COPY, ADD 指令中的工作目錄
- others
    - .dockerignore
        - 用來避免加入不必要的大檔案或是機敏文件

see more -> [click me!](https://docs.docker.com/engine/reference/builder/)

## ADD vs COPY

- ADD 用來將 **遠端** 的檔案複製到 image 的 filesystem 中
- COPY 用來將 **本地端** 的檔案複製到 image 的 filesystem 中
- 如果是一般的複製，文件建議使用 COPY 指令

## ARG vs ENV

- ARG 由建立 image 的時候帶入
- ENV 是在 container 執行時作為環境變數使用

```bash
docker builder build --build-arg VERSION=10 .
```

```dockerfile
ARG VERSION=14

FROM node:${VERSION}
```

## CMD vs ENTRYPOINT

- 一個 Dockerfile 中只能有一個 CMD 指令，若出現多次則最後一次會產生作用
- ENTRYPOINT 的用途和 CMD 類似，都是用來在 container 中執行指令
- ENTRYPOINT 在 container 啟動時預設執行
- CMD 在 ENTRYPOINT 存在時作為參數使用

```dockerfile
FROM alpine:latest

ENTRYPOINT [ "ping" ]

CMD [ "localhost" ]
```

# Practice

## Hello World

```bash
docker container run --detach --publish 80:80 --name hello-world docker/getting-started
```

![](https://mileschou.github.io/images/ironman/12th/docker-newbie/components.png)

## Demo Application

- demo code
```bash
git clone https://github.com/newjett0617/docker101
```

- Dockerfile
```dockerfile
FROM node:12-alpine

WORKDIR /app

COPY . .

RUN yarn install --production

CMD [ "node", "/app/src/index.js" ]
```

- build dockerfile
```bash
docker builder build --tag demo-app:latest .
```

- run image to container
```bash
docker container run --detach --publish 3000:3000 --name demo-app demo-app:latest
```

- create volume to keep data
```bash
docker volume create todo-db
```

- run image to container with volume
```bash
docker container run --detach --publish 3000:3000 --volume todo-db:/etc/todos --name demo-app demo-app:latest
```

- stop running container
```bash
docker container stop demo-app
```

- delete container
```bash
docker container rm demo-app
```




# Reference

- [https://en.wikipedia.org/wiki/Docker_(software)](https://en.wikipedia.org/wiki/Docker_%28software%29 "https://en.wikipedia.org/wiki/Docker_(software)")
- https://en.wikipedia.org/wiki/OS-level_virtualization
- https://www.weave.works/blog/a-practical-guide-to-choosing-between-docker-containers-and-vms
- https://docs.docker.com/glossary/
- https://docs.docker.com/get-started/overview/
- https://docs.docker.com/engine/install/ubuntu/
- https://docs.microsoft.com/virtualization/windowscontainers/about/containers-vs-vm
- https://docs.microsoft.com/visualstudio/docker/tutorials/docker-tutorial
- https://tw.alphacamp.co/blog/docker-introduction
