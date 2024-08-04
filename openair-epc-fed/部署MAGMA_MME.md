<table style="border-collapse: collapse; border: none;">
  <tr style="border-collapse: collapse; border: none;">
    <td style="border-collapse: collapse; border: none;">
      <a href="http://www.openairinterface.org/">
         <img src="./images/oai_final_logo.png" alt="" border=3 height=50 width=150>
         </img>
      </a>
    </td>
    <td style="border-collapse: collapse; border: none; vertical-align: center;">
      <b><font size = "5">OpenAirInterface 核心网 : 基于 MAGMA MME 的部署</font></b>
    </td>
  </tr>
</table>

# 为什么应该部署这个? #

- 您想体验最新版：
  - `magma` 项目的 `master` 分支每天都会更新新功能、错误修复……
- 您希望整个 EPC 能够连续运行数小时和数天而无需重新启动

**2021/07/28 更新: 您现在可以选择拉取镜像或自行构建。**

**2022/02/25 更新：MAGMA-MME 现在可以与完整的 OAI 协议栈一起使用来执行 OAI RF 模拟。**

**目录**

1.  [前置条件](./部署MAGMA的前置条件.md)
2.  获取镜像
    1.  [获取容器镜像](./获取MAGMA官方镜像.md)
    2.  [构建容器镜像](./构建MAGMA_MME镜像.md)
3.  [用docker-compose部署MAGMA-MME](../docker-compose/magma-mme-demo/README.md)
4.  [网络环境验证/纠正](./配置MAGMA网络.md)
5.  [2021 年 MAGMA-dev 会议期间做的demo](./支持NSA的RAN.md)
