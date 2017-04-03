
# Chromium OS build guide


### Install depot_tools

depot_tools is a set of build scripts, install it and export to system path
```
cd ~
mkdir chromiusos_build
cd chromiusos_build
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH=`pwd`/depot_tools:"$PATH"
```

### download chromium source
chromiumos using a chroot system to build, you need set umask to avoid chroot errors 
```
echo "umask 022" >> ~/.bashrc
. ~/.bashrc
```

chromiumos manage code by repo, same as Android
download source code by these commands
```
mkdir chromiumos
cd chromiumos
repo init -u https://chromium.googlesource.com/chromiumos/manifest.git --repo-url https://chromium.googlesource.com/external/repo.git 
repo sync -j4
```

### build
before build, you need login a chroot system by this
```
cros_sdk
```

setup build environment
```
~/trunk/src/scripts$ export BOARD=x86-generic
~/trunk/src/scripts$ ./setup_board --board=${BOARD}
```

build package & image
```
~/trunk/src/scripts$ ./build_packages --board=${BOARD}
~/trunk/src/scripts$ ./build_image --board=${BOARD} --noenable_rootfs_verification test
```

convert image to qemu format
```
~/trunk/src/scripts$ ./image_to_vm.sh --board=${BOARD} --test_image
```

### run
then you can launch chromiumos in local console
* launch by chromiumos, then you need login through VNC
```
cd ~/chromiusos_build/chromiumos/src/scripts/bin
cros_start_vm --image_path=../../build/images/x86-generic/latest/chromiumos_qemu_image.bin
```
* launch directly
```
sudo kvm -m 1024 -vga cirrus -pidfile /tmp/kvm.pid -net nic,model=virtio -net user,hostfwd=tcp:127.0.0.1:9222-:22 -hda /mnt/host/source/src/build/images/x86-generic/latest/chromiumos_qemu_image.bin
```

if you need, you can login chromiumos by ssh
```
ssh root@127.0.0.1 -p 9222 -o StrictHostKeyChecking=no
```

### change boot animation
start a feature branch
```
repo start feature_boot_animation
```

find the assets package name
```
cros_workon info --board=${BOARD} --all | grep assets
```

replace images in `chromiumos/src/platform/chromiumos-assets` by custom animation resources
rebuild package & deploy to qemu instance
````
cros_workon --board=${BOARD} start chromiumos-assets
cros_workon_make --board=${BOARD} chromiumos-assets
cros deploy ssh://root@127.0.0.1:9222 chromiumos-assets
```


**Links**
http://dev.chromium.org/chromium-os/developer-guide
https://www.chromium.org/chromium-os/how-tos-and-troubleshooting/running-chromeos-image-under-virtual-machines
https://www.chromium.org/chromium-os/build/cros-deploy
