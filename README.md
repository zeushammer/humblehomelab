# humblehomelab

! ----------
! Interfaces 
! ----------

! EdgeRouter
! 1. eth0 WAN		203.0.113.1
! 2. eth1 LAN		192.168.1.1/24 
! 3. vti0    		169.254.x.x/30
! 3. vti1    		169.254.x.x/30
! 4. asn			65000
 
! AWS
! 1. vgw        	192.0.2.1
! 2. vgw 			198.51.100.1
! 3. vpc cidr		172.16.0.0/22
! 4. vpc subnet		172.16.1.0/24
! 5. asn			65515

! ---------------------------
! Show Configuration Commands 
! ---------------------------

set vpn ipsec auto-firewall-nat-exclude enable

set vpn ipsec esp-group FOO0 lifetime 3600
set vpn ipsec esp-group FOO0 pfs enable
set vpn ipsec esp-group FOO0 proposal 1 encryption aes128
set vpn ipsec esp-group FOO0 proposal 1 hash sha1

set vpn ipsec ike-group FOO0 key-exchange ikev1
set vpn ipsec ike-group FOO0 lifetime 28800
set vpn ipsec ike-group FOO0 proposal 1 dh-group 2
set vpn ipsec ike-group FOO0 proposal 1 encryption aes128
set vpn ipsec ike-group FOO0 proposal 1 hash sha1
set vpn ipsec ike-group FOO0 dead-peer-detection action restart
set vpn ipsec ike-group FOO0 dead-peer-detection interval 15
set vpn ipsec ike-group FOO0 dead-peer-detection timeout 30

set vpn ipsec site-to-site peer 192.0.2.1 authentication mode pre-shared-secret
set vpn ipsec site-to-site peer 192.0.2.1 authentication pre-shared-secret <secret>
set vpn ipsec site-to-site peer 192.0.2.1 connection-type initiate
set vpn ipsec site-to-site peer 192.0.2.1 description IPsecAWS
set vpn ipsec site-to-site peer 192.0.2.1 ike-group FOO0
set vpn ipsec site-to-site peer 192.0.2.1 local-address 203.0.113.1
set vpn ipsec site-to-site peer 192.0.2.1 vti bind vti0
set vpn ipsec site-to-site peer 192.0.2.1 vti esp-group FOO0

set vpn ipsec site-to-site peer 198.51.100.1 authentication mode pre-shared-secret
set vpn ipsec site-to-site peer 198.51.100.1 authentication pre-shared-secret <secret>
set vpn ipsec site-to-site peer 198.51.100.1 connection-type initiate
set vpn ipsec site-to-site peer 198.51.100.1 description IPsecAWS
set vpn ipsec site-to-site peer 198.51.100.1 ike-group FOO0
set vpn ipsec site-to-site peer 198.51.100.1 local-address 203.0.113.1
set vpn ipsec site-to-site peer 198.51.100.1 vti bind vti1
set vpn ipsec site-to-site peer 198.51.100.1 vti esp-group FOO0

set interfaces vti vti0 address 169.254.x.x/30
set interfaces vti vti1 address 169.254.x.x/30

set firewall options mss-clamp interface-type vti
set firewall options mss-clamp mss 1379

set protocols bgp 65000 timers holdtime 30
set protocols bgp 65000 timers keepalive 10
set protocols bgp 65000 network 192.168.1.0/24

set protocols bgp 65000 neighbor 169.254.x.x prefix-list export BGP
set protocols bgp 65000 neighbor 169.254.x.x prefix-list import BGP
set protocols bgp 65000 neighbor 169.254.x.x remote-as 65515
set protocols bgp 65000 neighbor 169.254.x.x soft-reconfiguration inbound

set protocols bgp 65000 neighbor 169.254.x.x prefix-list export BGP
set protocols bgp 65000 neighbor 169.254.x.x prefix-list import BGP
set protocols bgp 65000 neighbor 169.254.x.x remote-as 65515
set protocols bgp 65000 neighbor 169.254.x.x soft-reconfiguration inbound

set policy prefix-list BGP rule 10 action deny
set policy prefix-list BGP rule 10 description 'deny local wan'
set policy prefix-list BGP rule 10 prefix 203.0.113.1/32

set policy prefix-list BGP rule 20 action deny
set policy prefix-list BGP rule 20 description 'deny aws vgw1'
set policy prefix-list BGP rule 20 prefix 192.0.2.1/32

set policy prefix-list BGP rule 30 action deny
set policy prefix-list BGP rule 30 description 'deny aws vgw2'
set policy prefix-list BGP rule 30 prefix 198.51.100.1/32

set policy prefix-list BGP rule 100 action permit
set policy prefix-list BGP rule 100 description 'permit local lan'
set policy prefix-list BGP rule 100 prefix 192.168.1.0/24

set policy prefix-list BGP rule 110 action permit
set policy prefix-list BGP rule 110 description 'permit aws vpc'
set policy prefix-list BGP rule 110 prefix 172.16.0.0/22

set system offload ipsec enable 

! ------------------
! Show Configuration
! ------------------

...
firewall {
    options {
        mss-clamp {
            interface-type vti
            mss 1379
        }
    }
}
interfaces {
    vti vti0 {
        address 169.254.x.x/30
    }
    vti vti1 {
        address 169.254.x.x/30
    }
}
policy {
    prefix-list BGP {
        rule 10 {
            action deny
            description 'deny local wan'
            prefix 203.0.113.1/32
        }
        rule 20 {
            action deny
            description 'deny aws vgw1'
            prefix 192.0.2.1/32
        }
        rule 30 {
            action deny
            description 'deny aws vgw2'
            prefix 198.51.100.1/32
        }
        rule 100 {
            action permit
            description 'permit local lan'
            prefix 192.168.1.0/24
        }
        rule 110 {
            action permit
            description 'permit aws vpc'
            prefix 172.16.0.0/22
        }
    }
}
protocols {
    bgp 65000 {
        neighbor 169.254.x.x {
            prefix-list {
                export BGP
                import BGP
            }
            remote-as 65515
            soft-reconfiguration {
                inbound
            }
        }
        neighbor 169.254.x.x {
            prefix-list {
                export BGP
                import BGP
            }
            remote-as 65515
            soft-reconfiguration {
                inbound
            }
        }
        network 192.168.1.0/24 {
        }
        timers {
            holdtime 30
            keepalive 10
        }
    }
}
vpn {
    ipsec {
        esp-group FOO0 {
            lifetime 3600
            pfs enable
            proposal 1 {
                encryption aes128
                hash sha1
            }
        }
        ike-group FOO0 {
            dead-peer-detection {
                action restart
                interval 15
                timeout 30
            }
            key-exchange ikev1
            lifetime 28800
            proposal 1 {
                dh-group 2
                encryption aes128
                hash sha1
            }
        }
        site-to-site {
            peer 192.0.2.1 {
                authentication {
                    mode pre-shared-secret
                    pre-shared-secret <secret>
                }
                connection-type initiate
                description IPsecAWS
                ike-group FOO0
                local-address 203.0.113.1
                vti {
                    bind vti0
                    esp-group FOO0
                }
            }
            peer 198.51.100.1 {
                authentication {
                    mode pre-shared-secret
                    pre-shared-secret <secret>
                }
                connection-type initiate
                description IPsecAWS
                ike-group FOO0
                local-address 203.0.113.1
                vti {
                    bind vti1
                    esp-group FOO0
                }
            }
        }
    }
}

! -----------
! End of file
! -----------
