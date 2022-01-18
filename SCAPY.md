SCAPY

```python
def simple_tcp_packet_ext_taglist(pktlen=100,
                                  eth_dst=None,
                                  eth_src=None,
                                  dl_taglist_enable=False,
                                  dl_vlan_pcp_list=[0],
                                  dl_vlan_cfi_list=[0],
                                  dl_tpid_list=[0x8100],
                                  dl_vlanid_list=[1],
                                  ip_src='192.168.0.1',
                                  ip_dst='192.168.0.2',
                                  ip_tos=0,
                                  ip_ecn=None,
                                  ip_dscp=None,
                                  ip_ttl=64,
                                  ip_id=0x0001,
                                  ip_frag=0,
                                  tcp_sport=1234,
                                  tcp_dport=80,
                                  tcp_flags="S",
                                  ip_ihl=None,
                                  ip_options=False,
                                  with_tcp_chksum=True
                                  ):
    """
    Return a simple dataplane TCP packet without tag or with one or multiple tags
    The size of the list of the argument "dl_vlan_pcp_list", "dl_vlan_cfi",
    "dl_tpid_list" and "dl_vlanid_list" must be the same when "dl_taglist_enable" is True.

    Supports a few parameters:
    @param pktlen Length of packet in bytes w/o CRC
    @param eth_dst Destinatino MAC
    @param eth_src Source MAC
    @param dl_taglist_enable False if no tag. True if the packet contains one or multiple tags.
    @param dl_vlan_pcp_list VLAN priority list. Not used when dl_taglist_enable is False.
    @param dl_vlan_cfi_list The VLAN CFI list. Not used when dl_taglist_enable is False.
    @param dl_tpid_list The TPID list. Not used when dl_taglist_enable is False.
    @param dl_vlanid_list The VLAN ID list. Not used when dl_taglist_enable is False.
    @param ip_src IP source
    @param ip_dst IP destination
    @param ip_tos IP ToS
    @param ip_ecn IP ToS ECN
    @param ip_dscp IP ToS DSCP
    @param ip_ttl IP TTL
    @param ip_id IP ID
    @param tcp_sport TCP source port
    @param tcp_dport TCP destination port
    @param tcp_flags TCP Control flags
    @param with_tcp_chksum Valid TCP checksum

    Generates a simple TCP request.  Users
    shouldn't assume anything about this packet other than that
    it is a valid ethernet/IP/TCP frame.
    """

    if MINSIZE > pktlen:
        pktlen = MINSIZE

    if with_tcp_chksum:
        tcp_hdr = scapy.TCP(sport=tcp_sport, dport=tcp_dport, flags=tcp_flags)
    else:
        tcp_hdr = scapy.TCP(sport=tcp_sport, dport=tcp_dport, flags=tcp_flags, chksum=0)

    ip_tos = ip_make_tos(ip_tos, ip_ecn, ip_dscp)

    # Note Dot1Q.id is really CFI
    if (dl_taglist_enable):
        pkt = scapy.Ether(dst=eth_dst, src=eth_src)

        for i in range(0, len(dl_vlanid_list)):
            pkt = pkt/scapy.Dot1Q(prio=dl_vlan_pcp_list[i], id=dl_vlan_cfi_list[i], vlan=dl_vlanid_list[i])

        pkt = pkt/scapy.IP(src=ip_src, dst=ip_dst, tos=ip_tos, ttl=ip_ttl, id=ip_id, ihl=ip_ihl)/ \
              tcp_hdr

        for i in range(1, len(dl_tpid_list)):
            pkt[Dot1Q:i].type=dl_tpid_list[i]
        pkt.type=dl_tpid_list[0]
    else:
        if not ip_options:
            pkt = scapy.Ether(dst=eth_dst, src=eth_src)/ \
                scapy.IP(src=ip_src, dst=ip_dst, tos=ip_tos, ttl=ip_ttl, id=ip_id, ihl=ip_ihl, frag=ip_frag)/ \
                tcp_hdr
        else:
            pkt = scapy.Ether(dst=eth_dst, src=eth_src)/ \
                scapy.IP(src=ip_src, dst=ip_dst, tos=ip_tos, ttl=ip_ttl, id=ip_id, ihl=ip_ihl, frag=ip_frag, options=ip_options)/ \
                tcp_hdr

    if (pktlen - len(pkt)) > 0:
        pkt = pkt/codecs.decode("".join(["%02x"%(x%256) for x in range(pktlen - len(pkt))]), "hex")
    return pkt

def simple_tcp_packet(pktlen=100,
                      eth_dst=None,
                      eth_src=None,
                      dl_vlan_enable=False,
                      vlan_vid=0,
                      vlan_pcp=0,
                      dl_vlan_cfi=0,
                      ip_src='192.168.0.1',
                      ip_dst='192.168.0.2',
                      ip_tos=0,
                      ip_ecn=None,
                      ip_dscp=None,
                      ip_ttl=64,
                      ip_id=0x0001,
                      ip_frag=0,
                      tcp_sport=1234,
                      tcp_dport=80,
                      tcp_flags="S",
                      ip_ihl=None,
                      ip_options=False,
                      with_tcp_chksum=True
                      ):
    """
    Return a simple dataplane TCP packet

    Supports a few parameters:
    @param pktlen Length of packet in bytes w/o CRC
    @param eth_dst Destinatino MAC
    @param eth_src Source MAC
    @param dl_vlan_enable True if the packet is with vlan, False otherwise
    @param vlan_vid VLAN ID
    @param vlan_pcp VLAN priority
    @param dl_vlan_cfi The VLAN CFI value
    @param ip_src IP source
    @param ip_dst IP destination
    @param ip_tos IP ToS
    @param ip_ecn IP ToS ECN
    @param ip_dscp IP ToS DSCP
    @param ip_ttl IP TTL
    @param ip_id IP ID
    @param tcp_dport TCP destination port
    @param tcp_sport TCP source port
    @param tcp_flags TCP Control flags
    @param with_tcp_chksum Valid TCP checksum

    Generates a simple TCP request.  Users
    shouldn't assume anything about this packet other than that
    it is a valid ethernet/IP/TCP frame.
    """

    pcp_list = []
    cfi_list = []
    tpid_list = []
    vlan_list = []

    if (dl_vlan_enable):
        pcp_list.append(vlan_pcp)
        cfi_list.append(dl_vlan_cfi)
        tpid_list.append(0x8100)
        vlan_list.append(vlan_vid)

    pkt = simple_tcp_packet_ext_taglist(pktlen=pktlen,
                                        eth_dst=eth_dst,
                                        eth_src=eth_src,
                                        dl_taglist_enable=dl_vlan_enable,
                                        dl_vlan_pcp_list=pcp_list,
                                        dl_vlan_cfi_list=cfi_list,
                                        dl_tpid_list=tpid_list,
                                        dl_vlanid_list=vlan_list,
                                        ip_src=ip_src,
                                        ip_dst=ip_dst,
                                        ip_tos=ip_tos,
                                        ip_ecn=ip_ecn,                             
                                        ip_ecn=ip_ecn,
                                        ip_dscp=ip_dscp,
                                        ip_ttl=ip_ttl,
                                        ip_id=ip_id,
                                        ip_frag=ip_frag,
                                        tcp_sport=tcp_sport,
                                        tcp_dport=tcp_dport,
                                        tcp_flags=tcp_flags,
                                        ip_ihl=ip_ihl,
                                        ip_options = ip_options,
                                        with_tcp_chksum = with_tcp_chksum)
    return pkt
```
