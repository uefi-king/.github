# Runtime Service for EDK2

## Environment setup

1. Clone both `edk2` and `linux` repositories to your local environment.

2. Build them.

3. Build a `initramfs.img` as the rootfs for your Linux, reference is [here].(https://gist.github.com/chrisdone/02e165a0004be33734ac2334f215380e) (No need to read the whole guide, just segments about building the initramfs.)

## Run Linux with EDK2

1. Create & enter a directory like `run-ovmf` to run qemu.

2. Create directory `hda-contents`, and place a script `start-linux.nsh` inside:

```shell
linux.efi console=ttyS0 initrd=initramfs.img
```

3. Update your `RunQemu.sh`. For me, it's like:

```shell
export BUILDIR=/dir/to/edk2/Build/OvmfX64/DEBUG_GCC5
export LINUX_DIR=/dir/to/linux
export ROOTFS_DIR=/dir/to/rootfs

cp $LINUX_DIR/arch/x86/boot/bzImage hda-contents/linux.efi
cp $ROOTFS_DIR/initramfs.img        hda-contents/initramfs.img

qemu-system-x86_64 \
    -m 2048 \
    -net none \
    --bios $BUILDIR/FV/OVMF.fd \
    -cpu host \
    -drive file=fat:rw:hda-contents,format=raw \
    -drive file=fat:rw:$BUILDIR/X64,id=fat32,format=raw \
    -debugcon file:"ovmf-boot.log" \
    -global isa-debugcon.iobase=0x402 \
    -global ICH9-LPC.disable_s3=1 \
    -nographic \
    -enable-kvm
```

4. Run qemu and start linux.

```shell
$> . ./RunQemu.sh
$> fs0:start-linux.nsh
```

## Try runtime service

```shell
$> cat /sys/firmware/efi/uptime
```