# openwrt
fork frome openwrt tag：openwrt-21.02.3
<br>

# Build system usage

## Update the feeds
```
./scripts/feeds update -a
./scripts/feeds install -a
```

## Configure the firmware image and the kernel
```
make menuconfig V=s
make -j $(nproc) kernel_menuconfig V=s
```

## Build the firmware image
```
make -j $(nproc) defconfig download clean world V=s
```
<br>

# Cleaning up
```
make clean
```
Deletes contents of the directories /bin and /build_dir. This doesn't remove the toolchain, and it also avoids cleaning architectures/targets other than the one you have selected in your .config. It is a good practice to do make clean before a build to ensure that no outdated artefacts have been left from the previous builds. That may not be necessary always, but as a general rule it helps to ensure quality builds.

```
make dirclean
```
This is your basic “full clean” operation. Deletes contents of the directories /bin and /build_dir and additionally /staging_dir, /toolchain (=the cross-compile tools), /tmp (e.g data about packages) and /logs.

```
make distclean
```
Nukes everything you have compiled or configured and also deletes all downloaded feeds contents and package sources. :!: In addition to all else, this will erase your build configuration <buildroot>/.config. Use only if you need a “factory reset” of the build system!

## Selective cleanup
In more time, you may not want to clean so many objects, then you can use some of the commands below to do it.
```
# Clean linux objects
make target/linux/clean

# Clean package base-files objects
make package/base-files/clean

# Clean luci objects
make package/luci/clean
```

<br>

# Add Hello World
```
sed -i "/helloworld/d" "feeds.conf.default"
echo "src-git helloworld https://github.com/fw876/helloworld.git" >> "feeds.conf.default"

mkdir -p package/helloworld
for i in "dns2socks" "microsocks" "ipt2socks" "pdnsd-alt" "redsocks2"; do \
  svn checkout "https://github.com/immortalwrt/packages/trunk/net/$i" "package/helloworld/$i"; \
done

./scripts/feeds update helloworld
./scripts/feeds install -a -f -p helloworld
```

<br>

# VM setup

- Click on File → Preferences → Network  
  On macOS, this setting may be found through File > Host Network Manager…

- Click on Host-only Networks tab and then if you don't see a vboxnet0 entry click on the + icon on the right of the window to add a new one.  
  Now select the vboxnet0 entry, and click on the screwdriver icon on the right to open its settings.

- IPv4 Address should be 192.168.56.1, IPv4 Network Mask should be 255.255.255.0, IPv6 Address should be empty and IPv6 Network Mask should be 0

- (optional) you can also set the DHCP server as shown in the screenshot if you want to have dynamic addresses to the VM, but for this tutorial it is not required as we set a static address in the VM itself

## Network Settings
- configure Adapter 1  
    + with Host-only Adapter  
    + select vboxnet0 as (adapter) Name  
    + click on Advanced and in Adapter Type select Intel PRO/1000 MT Desktop  
    + Promiscuous mode should be set to Deny unless you have good reasons to enable it.  
<br>

- Configure Adapter 2
    + with NAT  
<br>

- (optional) Configure Adapter 3  
    + with Bridged Adapter  
    + in the Name field select the name of the network card (ethernet or wifi) of your PC that connected to a local network. On Windows it has a full device name, on Linux it will have codenames like eth0, eth1 for ethernet or wlp2s0 for wifi.  
    + Click on Advanced and do the same you did for Adapter 1's advanced options  

## Virtual Machine Settings
- Display the current network configuration
```
root@openwrt:~# uci show network
```

- Edit the network configuration to allow SSH access by writing these commands and pressing enter:
```
uci set network.lan.ipaddr='192.168.56.2'
uci commit
reboot
```

