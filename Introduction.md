# VPP

[ドキュメント](https://docs.google.com/document/d/1zqYN7qMavgbdkPWIJIrsPXlxNOZ_GhEveHQxpYr3qrg/edit)
[リポジトリ追加](https://packagecloud.io/fdio/master/install)
https://www.slideshare.net/npsg/vpp-72218327
[frr + VPP](https://github.com/FRRouting/frr/wiki/Alternate-forwarding-planes:-VPP)
[frr install](https://deb.frrouting.org/)
[frr documet](http://docs.frrouting.org/en/latest/)


## Install 

リポジトリ追加
```
curl -s https://packagecloud.io/install/repositories/fdio/master/script.deb.sh | sudo bash
```
パッケージインストール
```
apt update
apt-get install vpp
apt-get install vpp-plugin-dpdk
```

## Start

```
ubuntu@ubuntu:~$ cat /etc/vpp/startup.conf

unix {
  nodaemon
  log /var/log/vpp/vpp.log
  full-coredump
  cli-listen /run/vpp/cli.sock
  gid vpp
}

api-trace {
## This stanza controls binary API tracing. Unless there is a very strong reason,
## please leave this feature enabled.
  on
## Additional parameters:
##
## To set the number of binary API trace records in the circular buffer, configure nitems
##
## nitems <nnn>
##
## To save the api message table decode tables, configure a filename. Results in /tmp/<filename>
## Very handy for understanding api message changes between versions, identifying missing
## plugins, and so forth.
##
## save-api-table <filename>
}

api-segment {
  gid vpp
}

socksvr {
  default
}

cpu {
        ## In the VPP there is one main thread and optionally the user can create worker(s)
        ## The main thread and worker thread(s) can be pinned to CPU core(s) manually or automatically

        ## Manual pinning of thread(s) to CPU core(s)

        ## Set logical CPU core where main thread runs, if main core is not set
        ## VPP will use core 1 if available
        # main-core 1

        ## Set logical CPU core(s) where worker threads are running
        # corelist-workers 2-3,18-19

        ## Automatic pinning of thread(s) to CPU core(s)

        ## Sets number of CPU core(s) to be skipped (1 ... N-1)
        ## Skipped CPU core(s) are not used for pinning main thread and working thread(s).
        ## The main thread is automatically pinned to the first available CPU core and worker(s)
        ## are pinned to next free CPU core(s) after core assigned to main thread
        # skip-cores 4

        ## Specify a number of workers to be created
        ## Workers are pinned to N consecutive CPU cores while skipping "skip-cores" CPU core(s)
        ## and main thread's CPU core
        # workers 2

        ## Set scheduling policy and priority of main and worker threads

        ## Scheduling policy options are: other (SCHED_OTHER), batch (SCHED_BATCH)
        ## idle (SCHED_IDLE), fifo (SCHED_FIFO), rr (SCHED_RR)
        # scheduler-policy fifo

        ## Scheduling priority is used only for "real-time policies (fifo and rr),
        ## and has to be in the range of priorities supported for a particular policy
        # scheduler-priority 50
}


dpdk {
   socket-mem 1024
   dev 0000:0b:00.0
}

# dpdk {
        ## Change default settings for all interfaces
        # dev default {
                ## Number of receive queues, enables RSS
                ## Default is 1
                # num-rx-queues 3

                ## Number of transmit queues, Default is equal
                ## to number of worker threads or 1 if no workers treads
                # num-tx-queues 3

                ## Number of descriptors in transmit and receive rings
                ## increasing or reducing number can impact performance
                ## Default is 1024 for both rx and tx
                # num-rx-desc 512
                # num-tx-desc 512

                ## VLAN strip offload mode for interface
                ## Default is off
                # vlan-strip-offload on
        # }

        ## Whitelist specific interface by specifying PCI address
        # dev 0000:02:00.0

        ## Blacklist specific device type by specifying PCI vendor:device
        ## Whitelist entries take precedence
        # blacklist 8086:10fb

        ## Set interface name
        # dev 0000:02:00.1 {
        #       name eth0
        # }

        ## Whitelist specific interface by specifying PCI address and in
        ## addition specify custom parameters for this interface
        # dev 0000:02:00.1 {
        #       num-rx-queues 2
        # }

        ## Specify bonded interface and its slaves via PCI addresses
        ##
        ## Bonded interface in XOR load balance mode (mode 2) with L3 and L4 headers
        # vdev eth_bond0,mode=2,slave=0000:02:00.0,slave=0000:03:00.0,xmit_policy=l34
        # vdev eth_bond1,mode=2,slave=0000:02:00.1,slave=0000:03:00.1,xmit_policy=l34
        ##
        ## Bonded interface in Active-Back up mode (mode 1)
        # vdev eth_bond0,mode=1,slave=0000:02:00.0,slave=0000:03:00.0
        # vdev eth_bond1,mode=1,slave=0000:02:00.1,slave=0000:03:00.1

        ## Change UIO driver used by VPP, Options are: igb_uio, vfio-pci,
        ## uio_pci_generic or auto (default)
        # uio-driver vfio-pci

        ## Disable multi-segment buffers, improves performance but
        ## disables Jumbo MTU support
        # no-multi-seg

        ## Increase number of buffers allocated, needed only in scenarios with
        ## large number of interfaces and worker threads. Value is per CPU socket.
        ## Default is 16384
        # num-mbufs 128000

        ## Change hugepages allocation per-socket, needed only if there is need for
        ## larger number of mbufs. Default is 256M on each detected CPU socket
        # socket-mem 2048,2048

        ## Disables UDP / TCP TX checksum offload. Typically needed for use
        ## faster vector PMDs (together with no-multi-seg)
        # no-tx-checksum-offload
# }


# plugins {
        ## Adjusting the plugin path depending on where the VPP plugins are
        #       path /ws/vpp/build-root/install-vpp-native/vpp/lib/vpp_plugins

        ## Disable all plugins by default and then selectively enable specific plugins
        # plugin default { disable }
        # plugin dpdk_plugin.so { enable }
        # plugin acl_plugin.so { enable }

        ## Enable all plugins by default and then selectively disable specific plugins
        # plugin dpdk_plugin.so { disable }
        # plugin acl_plugin.so { disable }
# }

```

```
service vpp start

```
## add interface
PCIデバイス確認
```
ubuntu@ubuntu:~$ sudo lshw -class network -businfo
Bus info          Device      Class      Description
====================================================
pci@0000:02:00.0  ens32       network    82545EM Gigabit Ethernet Controller (Copper)
pci@0000:0b:00.0              network    VMXNET3 Ethernet Controller

```

`/etc/vpp/startup.conf`に追記
```
dpdk{
  dev 0000:0b:00.0
}
```
vpp再起動
```
systemctl restart vpp
```


## shell
consoleに入るで〜
```
sudo vppctl
```
interface up
```
set interface state GigabitEthernetb/0/0 up
```

L3設定

```
vpp# set interface ip address GigabitEthernetb/0/0 10.0.0.2/24
vpp# # set interface ip address del GigabitEthernetb/0/0 10.0.0.2/24
vpp# show interface address
GigabitEthernetb/0/0 (up):
  L3 10.0.0.2/24
local0 (dn):
vpp# 
```

```
show interface
show hardware-interface
```

# TAP

https://wiki.fd.io/view/VPP/Configure_VPP_TAP_Interfaces_For_Container_Routing#Adding_a_new_tap_interface

```
tap connect GigabitEthernetb/0/0 tap0
tap connect GigabitEthernet13/0/0 tap1

# tapとgigabitethernetを繋ぐ
set interface l2 bridge GigabitEthernetb/0/0 0
set interface l2 bridge tapcli-0 0

set interface l2 bridge GigabitEthernet13/0/0 1
set interface l2 bridge tapcli-1 1
```

# FRR

install

```
curl -s https://deb.frrouting.org/frr/keys.asc | sudo apt-key add -
FRRVER="frr-stable"
echo deb https://deb.frrouting.org/frr $(lsb_release -s -c) $FRRVER | sudo tee -a /etc/apt/sources.list.d/frr.list
sudo apt update && sudo apt install frr frr-pythontools
```

```
sudo service frr start
sudo vtysh
configure terminal

int tap0
  ip address 10.0.2.4/24

int tap1
  ip address 10.0.3.4/24

```
