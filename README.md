b-real@ubnt:~$ show configuration 
firewall {
    all-ping enable
    broadcast-ping disable
    ipv6-name WANv6_IN {
        default-action drop
        description "WAN inbound traffic forwarded to LAN"
        enable-default-log
        rule 10 {
            action accept
            description "Allow established/related sessions"
            state {
                established enable
                related enable
            }
        }
        rule 20 {
            action drop
            description "Drop invalid state"
            state {
                invalid enable
            }
        }
    }
    ipv6-name WANv6_LOCAL {
        default-action drop
        description "WAN inbound traffic to the router"
        enable-default-log
        rule 10 {
            action accept
            description "Allow established/related sessions"
            state {
                established enable
                related enable
            }
        }
        rule 20 {
            action drop
            description "Drop invalid state"
            state {
                invalid enable
            }
        }
        rule 30 {
            action accept
            description "Allow IPv6 icmp"
            protocol ipv6-icmp
        }
        rule 40 {
            action accept
            description "allow dhcpv6"
            destination {
                port 546
            }
            protocol udp
            source {
                port 547
            }
        }
    }
    ipv6-receive-redirects disable
    ipv6-src-route disable
    ip-src-route disable
    log-martians enable
    name WAN_IN {
        default-action drop
        description "WAN to internal"
        rule 10 {
            action accept
            description "Allow established/related"
            state {
                established enable
                related enable
            }
        }
        rule 20 {
            action drop
            description "Drop invalid state"
            state {
                invalid enable
            }
        }
    }
    name WAN_LOCAL {
        default-action drop
        description "WAN to router"
        rule 10 {
            action accept
            description "Allow established/related"
            state {
                established enable
                related enable
            }
        }
        rule 20 {
            action drop
            description "Drop invalid state"
            state {
                invalid enable
            }
        }
    }
    receive-redirects disable
    send-redirects enable
    source-validation disable
    syn-cookies enable
}
interfaces {
    ethernet eth0 {
        description Local
        duplex auto
        poe {
            output off
        }
        speed auto
    }
    ethernet eth1 {
        description Local
        duplex auto
        poe {
            output off
        }
        speed auto
    }
    ethernet eth2 {
        description Local
        duplex auto
        poe {
            output off
        }
        speed auto
    }
    ethernet eth3 {
        description Local
        duplex auto
        poe {
            output off
        }
        speed auto
    }
    ethernet eth4 {
        description Local
        duplex auto
        poe {
            output off
        }
        speed auto
    }
    ethernet eth5 {
        duplex auto
        speed auto
        vif 102 {
            address dhcp
            description Internet
            dhcpv6-pd {
                pd 0 {
                    interface switch0 {
                        host-address ::1
                        prefix-id :1
                        service slaac
                    }
                    prefix-length /48
                }
                rapid-commit enable
            }
            firewall {
                in {
                    ipv6-name WANv6_IN
                    name WAN_IN
                }
                local {
                    ipv6-name WANv6_LOCAL
                    name WAN_LOCAL
                }
            }
        }
    }
    loopback lo {
    }
    switch switch0 {
        address 192.168.1.1/24
        description Local
        mtu 1500
        switch-port {
            interface eth0 {
            }
            interface eth1 {
            }
            interface eth2 {
            }
            interface eth3 {
            }
            interface eth4 {
            }
            vlan-aware disable
        }
    }
}
port-forward {
    auto-firewall enable
    hairpin-nat disable
    lan-interface switch0
    rule 1 {
        description plex
        forward-to {
            address 192.168.1.165
            port 32400
        }
        original-port 32400
        protocol tcp_udp
    }
    rule 2 {
        description rtorrent
        forward-to {
            address 192.168.1.165
            port 53199
        }
        original-port 53199
        protocol tcp_udp
    }
    wan-interface eth5.102
}
service {
    dhcp-server {
        disabled false
        hostfile-update disable
        shared-network-name LAN {
            authoritative enable
            subnet 192.168.1.0/24 {
                default-router 192.168.1.1
                dns-server 192.168.1.59
                lease 86400
                start 192.168.1.38 {
                    stop 192.168.1.243
                }
                static-mapping Chromecast-Ultra {
                    ip-address 192.168.1.39
                    mac-address 44:09:b8:2e:24:2b
                }
                static-mapping UBNT {
                    ip-address 192.168.1.43
                    mac-address b4:fb:e4:c0:a2:c8
                }
                static-mapping UniFi-CloudKey {
                    ip-address 192.168.1.207
                    mac-address b4:fb:e4:8f:d3:8f
                }
                static-mapping blap {
                    ip-address 192.168.1.242
                    mac-address 9c:b6:d0:e6:6b:1d
                }
                static-mapping nas {
                    ip-address 192.168.1.165
                    mac-address 04:d9:f5:f5:96:b8
                }
                static-mapping pihole {
                    ip-address 192.168.1.59
                    mac-address b8:27:eb:06:07:a1
                }
                unifi-controller 192.168.1.207
            }
        }
        static-arp disable
        use-dnsmasq enable
    }
    dns {
        forwarding {
            cache-size 150
            listen-on switch0
        }
    }
    gui {
        http-port 80
        https-port 443
        older-ciphers enable
    }
    nat {
        rule 5010 {
            description "masquerade for WAN"
            outbound-interface eth5.102
            type masquerade
        }
    }
    ssh {
        port 22
        protocol-version v2
    }
    unms {
        connection wss://enaas........
    }
}
system {
    host-name ubnt
    login {
        user b-real {
            authentication {
                encrypted-password ****************
                plaintext-password ****************
            }
            full-name "Gisle En<C3><A5>sen"
            level admin
        }
        user ubnt {
            authentication {
                encrypted-password ****************
            }
            level admin
        }
    }
    ntp {
        server 0.ubnt.pool.ntp.org {
        }
        server 1.ubnt.pool.ntp.org {
        }
        server 2.ubnt.pool.ntp.org {
        }
        server 3.ubnt.pool.ntp.org {
        }
    }
    syslog {
        global {
            facility all {
                level notice
            }
            facility protocols {
                level debug
            }
        }
    }
    time-zone UTC
    traffic-analysis {
        dpi enable
        export enable
    }
}
traffic-control {
    optimized-queue {
        policy queues
    }
    smart-queue SQM {
        download {
            ecn enable
            flows 1024
            fq-quantum 1514
            limit 10240
            rate 80mbit
        }
        upload {
            ecn enable
            flows 1024
            fq-quantum 1514
            limit 10240
            rate 80mbit
        }
        wan-interface eth5.102
    }
}
b-real@ubnt:~$ 
