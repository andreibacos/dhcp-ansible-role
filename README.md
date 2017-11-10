# Ansible role `dhcp`

Used as main starting point: ([bertvv/ansible-role-dhcp)](https://github.com/bertvv/ansible-role-dhcp)

Ansible role for setting up ISC DHCPD. The responsibilities of this role are to:

- Manage configuration ([dhcpd.conf(5)](http://linux.die.net/man/5/dhcpd.conf))

## A note on compatibility

Version 2.0.0 of this role uses features that are introduced in Ansible 2.0. Users of older versions of Ansible can use version 1.1.0 which has identical functionality. An overview of role compatibility:

| Role version | Ansible version | Supported platform                         |
| :---         | :---            | :---                                       |
| 1.0.0        | >= 2.0          | Ubuntu 14.04, 16.04                        |

Only the platforms on which the role was successfully tested are mentioned, but it should also work on other versions of these distributions. If you get it working on another platform, let me know and I will add it to the list.

## Requirements

No specific requirements

## Role Variables

This role is able to set (some) global options, and to specify subnet declarations.

See the test playbook (see below) for a working example of a DHCP server with two subnets. This section is mainly a reference of all supported options.

### Global options

The following variables, when set, will be added to the global section of the DHCP configuration file. There is no default value, so when they are not set, they will be left out.

See the [dhcp-options(5)](http://linux.die.net/man/5/dhcp-options) man page for more information about these options.

| Variable                          | Comments                                                                |
| :---                              | :---                                                                    |
| `dhcp_global_authoritative`       | Global authoritative statement (authoritative, not authoritative)       |
| `dhcp_global_log_facility`        | Global log facility                                                     |
| `dhcp_global_booting`             | Global booting (allow, deny, ignore)                                    |
| `dhcp_global_bootp`               | Global bootp (allow, deny, ignore)                                      |
| `dhcp_global_broadcast_address`   | Global broadcast address                                                |
| `dhcp_global_classes`             | Class definitions with a match statement(2)                             |
| `dhcp_global_default_lease_time`  | Default lease time in seconds                                           |
| `dhcp_global_domain_name_servers` | A list of IP addresses of DNS servers(1)                                |
| `dhcp_global_domain_name`         | The domain name the client should use when resolving host names         |
| `dhcp_global_domain_search`       | A list of domain names too be used by the client to locate non-FQDNs(1) |
| `dhcp_global_filename`            | Filename to request for boot                                            |
| `dhcp_global_includes`            | List of config files to be included (from `dhcp_config_dir`) (uses dhcp_subnet_names to include subnet definitions per servers rack)            |
| `dhcp_global_max_lease_time`      | Maximum lease time in seconds                                           |
| `dhcp_global_next_server`         | IP for boot server                                                      |
| `dhcp_global_ntp_servers`         | List of IP addresses of NTP servers                                     |
| `dhcp_global_routers`             | IP address of the router                                                |
| `dhcp_global_server_name`         | Server name sent to the client                                          |
| `dhcp_global_subnet_mask`         | Global subnet mask                                                      |
| `dhcp_global_omapi_port`          | OMAPI port                                                              |
| `dhcp_global_omapi_secret`        | OMAPI secret                                                            |
| `dhcp_global_other_options`       | Array of arbitrary additional global options                            |

(1) This option may be written either as a list (when you have more than one item), or as a string (when you have only one). The following snippet shows an example of both:

```Yaml
# A single DNS server
dhcp_global_domain_name_servers: 8.8.8.8

# A list of DNS servers
dhcp_global_domain_name_servers:
  - 8.8.8.8
  - 8.8.4.4
```

(2) This role supports the definition of classes with a match statement, e.g.:

```Yaml
# Class for VirtualBox VMs
dhcp_global_classes:
  - name: vbox
    match: 'match if binary-to-ascii(16,8,":",substring(hardware, 1, 3)) = "8:0:27"'
```

Class names can be used in the definition of address pools (see below).

### Subnet declarations

The role variable `dhcp_subnets` contains a list of dicts, specifying the subnet declarations to be added to the DHCP configuration file. Every subnet declaration should have an `ip` and `netmask`, other options are not mandatory. We start this section with an example, a compelete overview of supported options follows.

```Yaml
dhcp_subnet_names:
  - rack_no_1
  - rack_no_2
```

```Yaml
dhcp_subnets:
  rack_no_1:
    name: 'rack_no_1'
    ip: 192.168.50.0
    netmask: 255.255.255.0
    domain_name_servers:
      - 10.0.1.2
      - 10.0.1.3
    range_begin: 192.168.50.1
    range_end: 192.168.50.127
    routers: 192.168.50.252
    dhcp_hosts:
      - name: 'server-one'
        mac: '11:22:33:44:55:66'
      - name: 'server-two'
        mac: 'aa:bb:cc:dd:ee:ff'
  rack_no_2:
    name: 'rack_no_2'
    ip: 192.168.51.0
    default_lease_time: 3600
    max_lease_time: 7200
    netmask: 255.255.255.0
    routers: 192.168.51.252
    dhcp_hosts:
      - name: 'server_one'
        mac: '1a:2b:3c:4d:5e:6f'
      - name: 'server_two'
        mac: 'aa:11:bb:22:cc:33'
```

An alphabetical list of supported options in a subnet declaration:

| Option                | Required | Comment                                                               |
| :---                  | :---:    | :--                                                                   |
| `booting`             | no       | allow,deny,ignore                                                     |
| `bootp`               | no       | allow,deny,ignore                                                     |
| `default_lease_time`  | no       | Default lease time for this subnet (in seconds)                       |
| `domain_name_servers` | no       | List of domain name servers for this subnet(1)                        |
| `domain_search`       | no       | List of domain names for resolution of non-FQDNs(1)                   |
| `filename`            | no       | filename to retrieve from boot server                                 |
| `ip`                  | yes      | **Required.** IP address of the subnet                                |
| `max_lease_time`      | no       | Maximum lease time for this subnet (in seconds)                       |
| `netmask`             | yes      | **Required.** Network mask of the subnet (in dotted decimal notation) |
| `next_server`         | no       | IP address of the boot server                                         |
| `range_begin`         | no       | Lowest address in the range of dynamic IP addresses to be assigned    |
| `range_end`           | no       | Highest address in the range of dynamic IP addresses to be assigned   |
| `routers`             | no       | IP address of the gateway for this subnet                             |
| `server_name`         | no       | Server name sent to the client                                        |
| `subnet_mask`         | no       | Overrides the `netmask` of the subnet declaration                     |
| `dhcp_hosts`          | no       | List of hosts for static ip lease based on hw address                 |

You can specify address pools within a subnet by setting the `pools` options. This allows you to specify a pool of addresses that will be treated differently than another pool of addresses, even on the same network segment or subnet. It is a list of dicts with the following keys, all of which are optional:

| Option                | Comment                                             |
| :---                  | :---                                                |
| `allow`               | Specifies which hosts are allowed in this pool(1)   |
| `default_lease_time`  | The default lease time for this pool                |
| `deny`                | Specifies which hosts are not allowed in this pool  |
| `domain_name_servers` | The domain name servers to be used for this pool(1) |
| `max_lease_time`      | The maximum lease time for this pool                |
| `min_lease_time`      | The minimum lease time for this pool                |
| `range_begin`         | The lowest address in this pool                     |
| `range_end`           | The highest address in this pool                    |

(1) For the `allow` and `deny` fields, the options are enumerated in [dhcpd.conf(5)](http://linux.die.net/man/5/dhcpd.conf), but include:

- `booting`
- `bootp`
- `client-updates`
- `known-clients`
- `members of "CLASS"`
- `unknown-clients`

### Host declarations

You can specify hosts that should get a fixed IP address based on their MAC by setting the `dhcp_hosts` option. This is a list of dicts with the following three keys, all of which are mandatory:

| Option | Comment                                   |
| :---   | :---                                      |
| `name` | The name of the host                      |
| `mac`  | The MAC address of the host               |
| `ip`   | The IP address to be assigned to the host |

**Obs.**: On this role, IP field is generated automatically based on U**xx** ID.

**Obs.**: This behaviour can be overwritten by modifing fixed-address field in roles/dhcp-role/templates/subnet.conf.j2 file and using the following value {{ host.ip }}.

```Yaml
dhcp_hosts:
  - name: cl1
    mac: '00:11:22:33:44:55'
    ip: 192.168.222.150
  - name: cl2
    mac: '00:de:ad:be:ef:00'
    ip: 192.168.222.151
```

## Dependencies

No dependencies
