# Runtime Service for EDK2

## Environment setup

1. Clone both `edk2` and `linux` repositories to your local environment.

2. Build them. Default config is fine.

3. Build a `initramfs.img` as the rootfs for your Linux, reference is [here](https://gist.github.com/chrisdone/02e165a0004be33734ac2334f215380e). (No need to read the whole guide, just segments about building the initramfs.)

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
    if [ "$1" = "--clear-flash" ]; then
        cp $BUILDIR/FV/OVMF.fd          hda-contents/flash.img
    fi
    
    rm $BUILDIR/X64/*.debug >/dev/null
    
    qemu-system-x86_64 \
        -m 2048 \
        -net none \
        -cpu host \
        -drive file=fat:rw:hda-contents,format=raw \
        -drive file=fat:rw:$BUILDIR/X64,id=fat32,format=raw \
        -drive file=hda-contents/flash.img,format=raw,if=pflash \
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
host> . ./RunQemu.sh --clear-flash
qemu> ehco "hello flash" > /sys/firmware/efi/flash_encrypted
qemu> head -n 1 /sys/firmware/efi/flash_encrypted
# ----- reboot -----
host> . ./RunQemu.sh
qemu> head -n 1 /sys/firmware/efi/flash_encrypted
qemu> echo "hello raw flash" > /sys/firmware/efi/flash
qemu> head -n 1 /sys/firmware/efi/flash
```
