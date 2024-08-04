<table style="border-collapse: collapse; border: none;">
  <tr style="border-collapse: collapse; border: none;">
    <td style="border-collapse: collapse; border: none;">
      <a href="http://www.openairinterface.org/">
         <img src="./images/oai_final_logo.png" alt="" border=3 height=50 width=150>
         </img>
      </a>
    </td>
    <td style="border-collapse: collapse; border: none; vertical-align: center;">
      <b><font size = "5">OpenAirInterface 核心网 Docker 部署：MAGMA-MME 与 OAI CN 的先决条件</font></b>
    </td>
  </tr>
</table>

# 1. 安装正确版本的 Docker #

撰写时间 (2021 / 02 / 01):

```bash
$ dpkg --list | grep docker
ii  docker-ce                              5:20.10.2~3-0~ubuntu-bionic                     amd64        Docker: the open-source application container engine
ii  docker-ce-cli                          5:20.10.2~3-0~ubuntu-bionic                     amd64        Docker CLI: the open-source application container engine
ii  docker-ce-rootless-extras              5:20.10.2~3-0~ubuntu-bionic                     amd64        Rootless support for Docker.
```

**注意：不要忘记将您的用户名添加到`docker`组**

要不每次都得输入`sudo`才能用docker

```bash
$ sudo usermod -a -G docker myusername
```

**注意：在撰写本文时（2021 / 02 / 01），我们仅支持 Ubuntu18.04 部署。**

请参考官方 [docker安装指南](https://docs.docker.com/engine/install/).

官方页面比这里更详细。

## 1.1. 安装最新版本的`docker-compose` ##

官方[安装指南](https://docs.docker.com/compose/install/).

我们推荐 `1.27` 版本以上的docker-compose.

# 2. 创建一个Docker Hub账号 #

前往 [https://hub.docker.com/](https://hub.docker.com/) 并创建个账号。

# 3. 拉取基础镜像 #

* Ubuntu 版：我们需要 3 个基础镜像： `ubuntu:bionic`, `cassandra:2.1` 还有 `redis:6.0.5`

首先使用你的 Docker Hub 凭证登录。如果你的组织在以 `anonymous` 进行拉取时已达到限额，这是必要的步骤。

```bash
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: 
Password: 
```

然后拉取基础镜像，

在Ubuntu18.04上:

```bash
$ docker pull ubuntu:bionic
$ docker pull cassandra:2.1
$ docker pull redis:6.0.5
```

如果 `redis` 标签不可用，选择最新的 `6.0.x` 标签 [Docker Hub Redis Tags](https://hub.docker.com/_/redis?tab=tags).

最后您可以注销-->您的令牌以纯文本形式存储。

```bash
$ docker logout
```

# 4. 网络配置 #

**注意：第一步必须执行**

基于此条 [建议](https://docs.docker.com/network/bridge/#enable-forwarding-from-docker-containers-to-the-outside-world):

```bash
$ sudo sysctl net.ipv4.conf.all.forwarding=1
$ sudo iptables -P FORWARD ACCEPT
```

**注意：在您的环境中可能不需要执行此第二步。**

* 默认的 docker 网络（即“bridge”）位于“172.17.0.0/16”范围内。
* 在我们的 Eurecom 专用网络中，此 IP 地址范围已被使用。
  - 我们必须将其更改为我们私有网络配置中可用的另一个 IP 范围。
  - 我们通过添加 `/etc/docker/daemon.json` 文件选择了 **new/IDLE** IP 范围：

```json
{
	"bip": "192.168.17.1/24"
}
```

重启docker守护进程:

```bash
$ sudo service docker restart
$ docker info
```

检查新的网络配置:

```bash
$ docker network inspect bridge
[
    {
        "Name": "bridge",
....
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "192.168.17.1/24",
                    "Gateway": "192.168.17.1"
                }
            ]
        },
....
```

最后，有两个选择:

*  从 Docker Hub 中拉取 [官方镜像](./获取MAGMA官方镜像.md).
*  自己 [构建MAGMA-MME镜像](./构建MAGMA_MME镜像.md).

