---
title: DHCP Snooping
author: Cumulus Networks
weight: 355
toc: 3
---
DHCP snooping enables Cumulus Linux to act as a middle layer between the DHCP infrastructure and DHCP clients by scanning DHCP control packets to build an IP-MAC database, accept DHCP offers from only trusted interfaces, and rate limit packets.

## Configure DHCP Snooping

To configure DHCP snooping, you need to:

- Enable DHCP snooping on a VLAN with the `net add bridge <bridge-id> dhcp-snoop|dhcp-snoop6 vlan <vlan>` command.
- Add a trusted interface with the `net add bridge <bridge-id> dhcp-snoop|dhcp-snoop6 vlan <vlan> trust <interface>` command. Cumulus Linux allows DHCP offers from only trusted interfaces to prevent malicious DHCP servers from assigning IP addresses inside the network. The interface must be a member of the bridge specified.
- Set the rate limit for DHCP packets with the `net add bridge <bridge-id> dhcp-snoop|dhcp-snoop6 vlan <vlan> rate-limit <0-1000>` command. The default value is 10 packets per second.

The following example commands show you how to configure DHCP snooping for IPv4 and IPv6.

{{< tabs "TabID01 ">}}

{{< tab "IPv4 ">}}

```
cumulus@leaf01:~$ net add bridge br0 dhcp-snoop vlan 100
cumulus@leaf01:~$ net add bridge br0 dhcp-snoop vlan 100 trust swp6
cumulus@leaf01:~$ net add bridge br0 dhcp-snoop vlan 100 rate-limit 100
cumulus@leaf01:~$ net pending
cumulus@leaf01:~$ net commit
```

{{< /tab >}}

{{< tab "IPv6 ">}}

```
cumulus@leaf01:~$ net add bridge br0 dhcp-snoop6 vlan 100
cumulus@leaf01:~$ net add bridge br0 dhcp-snoop6 vlan 100 trust swp6
cumulus@leaf01:~$ net add bridge br0 dhcp-snoop6 vlan 100 rate-limit 100
cumulus@leaf01:~$ net pending
cumulus@leaf01:~$ net commit
```

{{< /tab >}}

{{< /tabs >}}

The NCLU commands save the configuration in the `/etc/dhcpsnoop/dhcp_snoop.json` file. For example:

```
{
  "bridge": [
    {
      "bridge_id": "br0",  <<< Bridge name
      "vlan": [
        {
          "vlan_id": 100,
          "snooping": 1,
          "rate_limit": 100,
          "ip_version": 6,
          "trusted_interface": [
            "swp6"
          ],
        }
      ]
    }
  ]
}
```

When DHCP snooping detects a violation, the packet is dropped and a message is logged to the `/var/log/dhcpsnoop.log` file.

## Show the DHCP Binding Table

To show the DHCP binding table, run the `net show dhcp-snoop table` command:

```
cumulus@leaf01:~$ net show dhcp-snoop table
Port VLAN IP        MAC               Lease State Bridge
---- ---- --------- ----------------- ----- ----- ------

swp5 1002 10.0.0.3  00:02:00:00:00:04 7200  ACK   br0

swp5 1000 10.0.1.3  00:02:00:00:00:04 7200  ACK   br0
```
