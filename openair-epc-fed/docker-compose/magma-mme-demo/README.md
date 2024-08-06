<table style="border-collapse: collapse; border: none;">
  <tr style="border-collapse: collapse; border: none;">
    <td style="border-collapse: collapse; border: none;">
      <a href="http://www.openairinterface.org/">
         <img src="../../images/oai_final_logo.png" alt="" border=3 height=50 width=150>
         </img>
      </a>
    </td>
    <td style="border-collapse: collapse; border: none; vertical-align: center;">
      <b><font size = "5">OpenAirInterface 核心网部署 : 一个使用 Docker-Compose 部署的 MAGMA-MME 案例</font></b>
    </td>
  </tr>
</table>

**目录**

1.  [初始化Cassandra数据库](#1-初始化Cassandra数据库)
2.  [部署EPC的剩余部分](#2-部署EPC的剩余部分)
3.  [卸载EPC](#3-卸载EPC)
4.  [如何编辑docker compose文件](#4-如何编辑docker-compose文件)
    1.  [UE SIM卡和其他UE配置](#41-UE-SIM卡和其他UE配置)
    2.  [您的公共陆地移动网络](#42-您的公共陆地移动网络)
    3.  [您的网络配置](#43-您的网络配置)
    4.  [杂项](#44-杂项)
5.  [连接eNB](#5-连接enb)
6.  [使用OAI RAN的NSA网络连接5G手机](#6-使用OAI-RAN的NSA网络连接5G手机)

# 1. 初始化Cassandra数据库 #

启动和初始化数据库需要一点时间。

在 `docker-compose 3.x` 中，部署时不再支持基于健康检查的条件依赖。
（注："条件健康依赖" 指的是在 Docker Compose 配置中，服务的启动顺序依赖于其他服务的健康状态。例如，一个服务只有在另一个服务被认为是健康的（即通过健康检查）情况下才会启动。）

这一步现在是 **纯手动的**.

```bash
$ cd docker-compose/magma-mme-demo
$ docker-compose up -d db_init
Creating network "demo-oai-private-net" with the default driver
Creating network "demo-oai-public-net" with the default driver
Creating demo-cassandra ... done
Creating demo-db-init   ... done

$ docker logs demo-db-init --follow
Connection error: ('Unable to connect to any servers', {'192.168.68.130': error(111, "Tried connecting to [('192.168.68.130', 9042)]. Last error: Connection refused")})
Connection error: ('Unable to connect to any servers', {'192.168.68.130': error(111, "Tried connecting to [('192.168.68.130', 9042)]. Last error: Connection refused")})
Connection error: ('Unable to connect to any servers', {'192.168.68.130': error(111, "Tried connecting to [('192.168.68.130', 9042)]. Last error: Connection refused")})
Connection error: ('Unable to connect to any servers', {'192.168.68.130': error(111, "Tried connecting to [('192.168.68.130', 9042)]. Last error: Connection refused")})
OK

$ docker rm -f demo-db-init
demo-db-init
```

注意：我们正在删除`demo-db-init`容器，因为不再需要它了。

你也可以留着这个容器，但它已经挂了。

要进入下一步，您**必须**在 `demo-db-init` 容器日志中看到“OK”消息。

稍等片刻，确保在启动 Cassandra 镜像时出现以下日志。如果没有，HSS 镜像将在下一步中过早停止。

```bash
$ docker logs demo-cassandra
....
INFO  20:19:40 Initializing vhss.extid_imsi_xref
```


# 2. 部署EPC的剩余部分 #

```bash
$ docker-compose up -d oai_spgwu
demo-cassandra is up-to-date
Creating demo-redis   ... done
Creating demo-oai-hss ... done
Creating demo-magma-mme ... done
Creating demo-oai-spgwc ... done
Creating demo-oai-spgwu-tiny ... done

# wait a bit

$ docker ps -a
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS                    PORTS                                         NAMES
cf93fa4c5fdf        oai-spgwu-tiny:production   "/openair-spgwu-tiny…"   46 seconds ago      Up 44 seconds (healthy)   2152/udp, 8805/udp                            demo-oai-spgwu-tiny
f59ceac1dba5        oai-spgwc:production        "/openair-spgwc/bin/…"   49 seconds ago      Up 46 seconds (healthy)   2123/udp, 8805/udp                            demo-oai-spgwc
80d71373ef4d        magma-mme:nsa-support       "/bin/bash -c 'cd /m…"   51 seconds ago      Up 49 seconds             3870/tcp, 2123/udp, 5870/tcp                  demo-magma-mme
7b2f67eeeac0        oai-hss:production          "/openair-hss/bin/en…"   56 seconds ago      Up 51 seconds (healthy)   5868/tcp, 9042/tcp, 9080-9081/tcp             demo-oai-hss
b94d74330f92        redis:6.0.5                 "/bin/bash -c 'redis…"   56 seconds ago      Up 52 seconds             6379/tcp                                      demo-redis
e51afc6e107c        cassandra:2.1               "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes (healthy)    7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp   demo-cassandra

```

这样就算准备好连接RAN和UE了

# 3. 卸载EPC #

```
$ docker-compose down
Stopping demo-oai-spgwu-tiny ... done
Stopping demo-oai-spgwc      ... done
Stopping demo-magma-mme      ... done
Stopping demo-oai-hss        ... done
Stopping demo-redis          ... done
Stopping demo-cassandra      ... done
Removing demo-oai-spgwu-tiny ... done
Removing demo-oai-spgwc      ... done
Removing demo-magma-mme      ... done
Removing demo-oai-hss        ... done
Removing demo-redis          ... done
Removing demo-cassandra      ... done
Removing network demo-oai-private-net
Removing network demo-oai-public-net
```

# 4. 如何编辑docker compose文件

显然，您需要一个适合您的环境的 EPC。这意味着：

*  专用的 UE SIM 卡配置
*  不同的 PLMN
*  您的网络配置肯定跟我们不一样

**尽量少修改 `docker-compose` 文件中的环境变量值。**

我在接下来的几节中描述的变量应该是您应该关注的变量！

* 它们在此 Markdown 页面中的格式为 **<code>ENV_VARIABLE</code>**

如果您不了解某个值是干什么的， **最好别碰它！** 

## 4.1. UE SIM卡和其他UE配置 ##

部署 EPC 的最终目的是通过 eNB 将 4G UE（智能手机或流量上网卡）连接到互联网。

当烧录 SIM 卡时，您需要确定以下参数：

* **IMSI** : 国际移动用户识别码 International Mobile Subscriber Identity
  * IMSI的具体格式详见 [这里](https://en.wikipedia.org/wiki/International_mobile_subscriber_identity)
* **LTE_KEY** 和 **OPC_KEY**
* 还有更多，但这里我就不展开说了

完成此操作后，将 SIM 卡放入智能手机并启动。您需要添加新的 **APN**（接入点名称）。

在这里您可以发挥创意。这是一个字符串。我们唯一要求 --> 它**必须**至少有一个“.”（点）。在我们的示例中为“oai.ipv4”。

现在您需要将用户配置到 Cassandra 数据库中，因此需要一些 **HSS** 参数：

* **<code>LTE_K</code>** 必须和你烧录进SIM卡的 **LTE_KEY** 匹配 
* **<code>OP_KEY</code>** **不是** OPC_KEY，但可以通过 OPC_KEY 和 LTE_KEY 计算得到。
  * 一个生成器的例子 [Ki/OPc Generator](https://github.com/PodgroupConnectivity/kiopcgenerator)
* **<code>APN1</code>** **必须**与您在智能手机上创建的APN相匹配。
* **<code>FIRST_IMSI</code>** 应该与您选择的相匹配

The **SPGW-C**, 自引入 FDQN 支持以来，有 2 个变量：

* **<code>APN_NI_1</code>**
* **<code>DEFAULT_APN_NI_1</code>**

这也应与您创建的 APN 相匹配。

## 4.2. 您的公共陆地移动网络 ##

又称 PLMN.

我强烈建议只修改 **MCC** 类型和 **MNC** 类型的参数。

你可以触及 **TAC** 类型的参数，但对于一个简单的教程来说，这会更加困难。

在我们的示例中，我们使用了 **222** 和 **01**，以及 **TAC** 为 **1**、**2**、**3**（**1** 为主要 TAC）。

**注意**：对于 **MCC** 和 **MNC** 的值，我使用了 ``` -- 2 或 3 位数字 -- ``` 的表示法：

- 尤其是当你的 **MNC** 像是 **'02'** 时，2 位数字，第一个是 '0'
- 这个第一个 `0` 必须在所有 cNF 配置文件中存在！

请注意，**MNC3** 类型的参数是用 3 个字符编码的。这是强制性的。
如果需要，你必须用 `0` 填充。

对于 **MME**，在撰写本文时，配置文件被硬编码在与 docker-compose 文件（`mme.conf`）相同的文件夹中：


```bash
    # ------- MME served GUMMEIs
    GUMMEI_LIST = (
         { MCC="222" ; MNC="01"; MME_GID="32768" ; MME_CODE="3"; }
    );

    # ------- MME served TAIs
    TAI_LIST = (
         {MCC="222" ; MNC="01";  TAC = "1"; },
         {MCC="222" ; MNC="01";  TAC = "2"; },
         {MCC="222" ; MNC="01";  TAC = "3"; }
    );

    TAC_LIST = (
         {MCC="222" ; MNC="01";  TAC = "1"; }
    );

    CSFB :
    {
        NON_EPS_SERVICE_CONTROL = "OFF";
        CSFB_MCC = "222";
        CSFB_MNC = "01";
        LAC = "1";
    };
```

对于docker-compose文件的 **SPGWC-C** 和 **SPGW-U-TINY**部分：

* **<code>MCC</code>**
* **<code>MNC</code>**
* **<code>MNC03</code>**
* **<code>TAC</code>**

**最后一个重点**： 您的 MME PLMN **必须** 与 eNB 配置中的 PLMN 匹配！

例如，如果您使用 OAI eNB 配置文件：

```bash
    tracking_area_code  =  1;
    plmn_list = (
      { mcc = 222; mnc = 01; mnc_length = 2; }
    );
```

## 4.3. 您的网络配置 ##

### DNS设置 ###

你的EPC Docker主机有自己的网关，你需要将这个网关设置为"本地DNS服务器"。

**否则** 当你的用户设备（UE）连接到核心网络时，将无法访问互联网！

在你的EPC容器宿主机上输入:

```bash
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.18.129  0.0.0.0         UG    300    0        0 nm-bond
....
```

在docker-compose文件中的 **SPGW-C** 节

使用第二个字段值作为 **<code>DEFAULT_DNS_IPV4_ADDRESS</code>** 字段。

在我们的示例中，**<code>DEFAULT_DNS_SEC_IPV4_ADDRESS</code>** 我们使用 Google DNS作为备用DNS。您可以选择其他任何值。

您也可以询问您的 IT 团队！

您可以使用以下 2 个变量进行调整（但它们已经是最佳值）：

* **<code>PUSH_PROTOCOL_OPTION</code>**
* **<code>NETWORK_UE_NAT_OPTION</code>**

### UE IP地址分配池 ###

最后一点：UE IP池用于在UE连接时为其分配IP地址。

在我们的示例中, 我选择 "12.1.1.2 - 12.1.1.254" 作为IP的范围。

请注意，SPGW-U 的IP地址范围也用 **CIDR** 格式定义为 "12.1.1.0/24"。

如果你需要更改，请同时遵循这两种格式。

docker-compose文件的 **SPGW-C** 部分:

* **<code>UE_IP_ADDRESS_POOL_1</code>**

docker-compose文件的 **SPGW-U-TINY** 部分:

* **<code>NETWORK_UE_IP</code>**

## 4.4. 杂项 ##

在撰写本文时（2021/02/01），MAGMA-MME 镜像上的自动化尚未完成。

没有entry-point脚本，没有默认的 MME 配置文件。

因此该文件夹中有 2 个文件：

- `mme-cfg.sh` 用作 `entrypoint`
- `mme.conf` 已经完成的MME配置文件

两个文件中都预先填好了参数，比如领域（realm）、IP地址等。

修改时要小心。

注意，在 `mme.conf` 文件中，**S6A** 部分已经更改：MME 现在支持与不同领域的 HSS 实体连接。（注：S6A 是指MME和HSS之间的通信接口）

```bash
    S6A :
    {
        S6A_CONF                   = "/magma-mme/etc/mme_fd.conf"; # YOUR MME freeDiameter config file path
        HSS_HOSTNAME               = "hss.openairinterface.org";
        HSS_REALM                  = "openairinterface.org";
    };
```

### 时区设置 ###

您可以在每个容器的部署时指定时区。

只需修改每个容器部分中的 **<code>TZ</code>**！

请注意，在构建时我默认设置了 `Europe/Paris`.

您可以在 Linux 系统的`/usr/share/zoneinfo/`中找到正确的值。

# 5. 连接eNB #

eNB 服务器上的网络配置仍然有效。

参阅[这里](../../配置MAGMA网络.md#step-2-在您的-enbgnb-服务器上创建路由)了解要执行的命令。

还有[这里](../../配置MAGMA网络.md#验证网络配置)以获取要验证的命令。

# 6. 使用OAI RAN的NSA网络连接5G手机 #

本部分解释了 2021 年 2 月 3 日 MAGMA-dev 会议期间所做演示的 RAN 相关部分。
参阅 [这里](../../docs/支持NSA的RAN.md)
