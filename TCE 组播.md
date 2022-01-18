## **设计思路**

证交所的组播流量先接入到用户自有IDC机房，然后再对接到TCE专线接入交换机。因为组播流量经过用户IDC机房，因此专线接入交换机与客户IDC机房PIM网络对接。

## **overlay网络是否要支持PIM？**

underlay流量封装vxlan是通过路由来实现的，即专线接入交换机作为三层网关，实现underlay与overlay的流量互通。在underlay侧必须与用户IDC进行PIM对接，在overlay侧有两种方式：1. 通过PIM对接 2. 通过IGMP对接  
overlay实现PIM的话，可以借助于PIM实现组播源的自动查找，最优转发路径选择，租户仅需在控制台开启通道组播即可，但是有两大局限：

1. 需要厂商设备支持：个厂商对overlay和underlay的PIM互联支持情况不一，且可选的型号较少
2. overlay实现PIM方案复杂，开发周期长

使用IGMP对接的话，首要的问题是在多通道的情况下，如何确定某个组需要向哪条通道，即哪个交换机点播。PIM网络中RP有网络中所有的组播组列表，当通过IGMP在overlay对接时，需要用户在控制台指定组播组配置在哪个通道上。  
TCE专线组播最终选择通过overlay igmp对接用户IDC PIM网络实现组播点播功能。

## **专线接入交换机是否参与RP的选举？**

在转发模型中，专线接入交换机一直处于PIM网络的边缘，因此专线接入交换机不参与RP的选举，只需要学习网络中的RP地址即可。

## **基于VPC的IGMP SNOOPING**

VPC内每一台cvm都直接通过IGMP向专线接入交换机加组和离组肯定是不现实的，TCE在组播网关上实现基于VPC的IGMP SNOOPING功能，基于VPC粒度向专线接入交换机发送IGMP report，leave报文。

## **组播认证**

组播认证客户端需要使用证交所分配的IP地址，vpc只能使用10，172，192三个网段。TCE放开弹性网卡上可配置的IP地址范围，cvm可以使用10，172，192之外的任意网段地址与证交所互联。

## **架构设计**

![](D:\笔记\knowledge-base\.images_base\2022-01-14-00-39-38-image.png)  
VPC首次点播时，VPC向DCPL发送IGMPv3 report，点播者地址固定为169.254.0.200。DCPL在overlay收到report报文后，PIM建立组播“流量树”。  
VPC最后一个点播者离组时，云向DCPL发送IGMPv3 leave报文，VPN内运行的PIM实例向PIM网络发送离开事件。  
DCPL每个VPN独立发送IGMPv3 Query报文，查询者地址固定为169.254.0.201。
