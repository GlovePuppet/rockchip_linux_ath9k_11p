# rockchip_linux_ath9k_11p

A patch to enable ITS-G5 channels on ath9k and the Rock Pi 4. 

[Inspired by a post on Harrison's Sandbox](https://harrisonsand.com/802-11p-v2x-hunting/)

[ath9k patch derived from content of this thread](https://mailarchive.ietf.org/arch/msg/its/dTP7e2qH4ST2fEtJMTosXFwHN6E)

Note: I had to reorder the channel list in net/wireless/db.txt to get the 5G-ITS frequencies to showup when the reg db is queried

## Hardware

Rock Pi 4
NGFF M.2 M Key to PCIE x4 adapter
PCIE to mini PCIE adapter
Ath9280 mini PC WIFI card

## Setup


### Fetch & build the Kernel

[Based on instructions on the radxa wiki](https://wiki.radxa.com/Rockpi4/dev/kernel-4.4)

Install the toolchain

`$ wget https://releases.linaro.org/components/toolchain/binaries/7.3-2018.05/aarch64-linux-gnu/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu.tar.xz`<br>
`$ sudo tar xvf gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu.tar.xz  -C /opt/`

Get the kernel source

`$ git clone -b release-4.4-rockpi4 https://github.com/radxa/kernel.git`<br>
`$ cd kernel`
   
Copy in and apply the patch 

`$ path -p1 < rockchip_linux_ath9k_11p.patch`

Configure and build the Kernel

`$ export ARCH=arm64`<br>
`$ export CROSS_COMPILE=/opt/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-`<br>
`$ make rockchip_linux_ath9k_11p_defconfig`<br>
`$ make -j8`

Build .deb packages

`$ export build_id="999"`<br>
`$ export lv="-$build_id-rockchip"`<br>
`$ export kv=$(make kernelversion)`<br>
`$ export debv="$kv$lv"`<br>
`$ make  bindeb-pkg -j8    LOCALVERSION=$lv    KDEB_PKGVERSION=$debv`

Copy linux-image-4.4.154-999-rockchip-*_all.deb and linux-firmware-image-4.4.154-999-rockchip-*_all.deb to your ROCK Pi 4 and install them

`# dpkg -i linux-image-4.4.154-999-rockchip-*_all.deb linux-firmware-image-4.4.154-999-rockchip-*_all.deb`

Reboot 

### Enable OCB mode

`# iw dev wlp1s0 set type ocb`<br>
`# ifconfig wlp1s0 up`<br>
`# iw dev wlp1s0 ocb join 5900 10MHz`

