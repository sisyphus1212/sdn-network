1. 

# [VPC组播介绍](https://iwiki.woa.com/pages/viewpage.action?pageId=955701230)

[转至元数据结尾](https://iwiki.woa.com/pages/viewpage.action?isEmbeded=true&pageId=955701230#page-metadata-end)

- 由 [cooperggu(谷峰博)](https://iwiki.woa.com/display/~cooperggu)创建于[八月 17, 2021](https://iwiki.woa.com/pageadmin/955701230/history "查看变更")

[转至元数据起始](https://iwiki.woa.com/pages/viewpage.action?isEmbeded=true&pageId=955701230#page-metadata-start)

**VPC组播功能介绍**

组播是一对多的通信方式，通过单点到多点的高效数据传送，可以为企业节约网络带宽、降低网络负载。云平台支持按照VPC维度开启组播，通过VPC组播开关按钮即可轻松控制组播功能的开通和关闭。

组播较多应用于金融和游戏行业：

- 金融行业主要用于同步行情数据。例如，获取股票价格等实时数据时，券商可通过组播，对多台 client 实时发送股票数据，有效降低网络负载。
- 游戏行业主要用于多台服务器之间的心跳保持。如果使用单播技术，发送主机需要分别向 N 个主机发送，共发送 N 次。如果使用组播，主机向 N 个主机发送相同的数据时，只要发送1次，既节省服务器资源，也节省了网络主干的带宽资源。

**VPC组播整体设计方案**

     组播通过专用高性能组播网关集群实现组播流量的转发，并通过分布式控制器实现实时组播路由的学习和更新。VPC组播通信逻辑架构如下图所示。

![](https://iwiki.woa.com/download/attachments/955701230/image2021-8-17_16-14-7.png?version=1&modificationDate=1629188047000&api=v2)

VPC组播逻辑架构中主要包含如下组件：

- 组播控制器（VPC_Mcctl）：是分布式组播的控制器，负责组播路由的管理和维护。
- 组播Agent（VPC_mcagent）：宿主机上的组播控制面组件，负责监听宿主机上的组播组和组播成员信息，并同步相关信息给组播控制器。
- 组播网关（MCGW）：独立部署的物理网关集群，负责组播流量的转发。
- VPC控制器（VPC OSS）：VPC分布式控制器，负责VPC路由和配置的管理和维护。

**VPC组播的技术原理**

VPC组播的主要通信流程如下：VPC开启组播后，VPC控制器将给VPC下所有云服务器的宿主机下发组播路由，路由的下一跳指向组播网关MCGW的VIP地址。

- 宿主机上VPC_mcagent向本地所有vpc发送igmp查询报文，并收集加入组播组的组播组成员信息，然后再调用控制器更新组播控制器的路由信息；MCGW会通过VPC agent向控制器获取到所有组播组成员路由信息。
- 组播源CVM发送组播报文给所在宿主机，宿主机封装报文并根据组播路由信息将报文转发给组播网关。
- 组播网关MCGW接收到组播源的报文，会查询该组播组下所有组播客户端的路由信息并进行报文的封装和转发。
- 组播组成员所在宿主机接收到报文后进行报文解封装，并转发给对应的组播客户端。
- 组播组成员离开组播组，会依次触发控制器和组播网关MCGW的路由更新。

[赞](https://iwiki.woa.com/pages/viewpage.action?isEmbeded=true&pageId=955701230)成为第一个赞同者

- 无标签
- [编辑标签](https://iwiki.woa.com/pages/viewpage.action?isEmbeded=true&pageId=955701230# "编辑标签 (Type 'l')")

[![用户图标: 添加头像](https://iwiki.woa.com/s/-f0u680/8402/f3adc6954ad4949d305c2db2c18353c2544063a7/_/images/icons/profilepics/add_profile_pic.svg)](https://iwiki.woa.com/users/profile/editmyprofilepicture.action)


