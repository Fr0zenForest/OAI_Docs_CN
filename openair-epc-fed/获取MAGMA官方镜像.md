<table style="border-collapse: collapse; border: none;">
  <tr style="border-collapse: collapse; border: none;">
    <td style="border-collapse: collapse; border: none;">
      <a href="http://www.openairinterface.org/">
         <img src="./images/oai_final_logo.png" alt="" border=3 height=50 width=150>
         </img>
      </a>
    </td>
    <td style="border-collapse: collapse; border: none; vertical-align: center;">
      <b><font size = "5">OpenAirInterface 4G 核心网部署 : 在 MAGMA MME 环境中拉取容器镜像</font></b>
    </td>
  </tr>
</table>

本文仅对`Ubuntu18`有效。 

如果你正在使用其他发行版，请参阅 [构建自己的镜像](./构建MAGMA_MME镜像.md).

如果您想了解最新的新功能，请参阅 [构建自己的镜像](./构建MAGMA_MME镜像.md).

# 从 Docker Hub 拉取镜像 #

目前这些镜像托管在`oaisoftwarealliance`组织下。

将来可能会更换用户名。（注：以前镜像是放在rdefosseoai这个用户名下的，后来改成了oaisoftwarealliance，可参照[这里](https://github.com/OPENAIRINTERFACE/openair-epc-fed/pull/40)）

如果您的组织已达到`anonymous`的拉取限制，您可能需要再次登录 [docker-hub](https://hub.docker.com/) 

```bash
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username:
Password:
```

现在拉取镜像：

```bash
$ docker pull oaisoftwarealliance/oai-hss:latest
$ docker pull oaisoftwarealliance/oai-spgwc:latest
$ docker pull oaisoftwarealliance/oai-spgwu-tiny:latest
$ docker pull oaisoftwarealliance/magma-mme:latest
```

并**re-tag**它们以使教程的 docker-compose 文件能够正常工作。

```bash
$ docker image tag oaisoftwarealliance/oai-hss:latest oai-hss:production
$ docker image tag oaisoftwarealliance/oai-spgwc:latest oai-spgwc:production
$ docker image tag oaisoftwarealliance/oai-spgwu-tiny:latest oai-spgwu-tiny:production
$ docker image tag oaisoftwarealliance/magma-mme:latest magma-mme:master
```

**注意：`MAGMA-MME` 镜像的更新频率不如其他镜像高，并且不存在任何后门。**

**我们仍然建议您按照描述自行[构建 MAGMA-MME 映像](./构建MAGMA_MME镜像.md).**

# 同步教程 #

**注意：请仔细阅读本节！**

该存储库仅包含教程和持续集成脚本。

**截至（2022/02/25），最新发布版为 `v1.2.0`.**

| CNF 名称    | 分支名称 | Tag        | Ubuntu 18.04 | RHEL8 (UBI8)    |
| ----------- | ----------- | ---------- | ------------ | ----------------|
| FED REPO    | N/A         | `v1.2.0`   |              |                 |
| HSS         | `master`    | `v1.2.0`   | X            | X               |
| SPWG-C      | `master`    | `v1.2.0`   | X            | X               |
| SPGW-U-TINY | `master`    | `v1.2.0`   | X            | X               |

```bash
# 直接克隆最新版代码
$ git clone --branch v1.2.0 https://github.com/OPENAIRINTERFACE/openair-epc-fed.git
$ cd openair-epc-fed
# 如果忘了克隆最新版
#（可以这么补救，-f会直接覆盖原有的，拉来最新版）
$ git checkout -f v1.2.0

# 同步所有子模块
$ ./scripts/syncComponents.sh
---------------------------------------------------------
OAI-HSS    component branch : master
OAI-SPGW-C component branch : master
OAI-SPGW-U component branch : master
---------------------------------------------------------
git submodule deinit --all --force
git submodule init
git submodule update
```

`master`分支的后续版本可能无法与拉取的镜像兼容。

部署:

* 参阅 [部署MAGMA-MME](../docker-compose/magma-mme-demo/README.md).


