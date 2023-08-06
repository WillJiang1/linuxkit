## How to build a TDX compatiable image

```sh
git clone https://github.com/yuguorui/linuxkit.git
cd linuxkit
make 

INITRD_LARGE_THAN_4GiB=1

if [ $INITRD_LARGE_THAN_4GiB -eq 1 ]; then
    (cd tools/grub && docker build -f Dockerfile.rhel -t linuxkit-hack/grub .)

    if [ -z $(docker ps -f name='registry' -q) ]; then
      docker run -d -p 5000:5000 --restart=always --name registry registry:2
    fi

    (
      remote_registry="localhost:5000/"
      tag="v0.1"
      cd tools/mkimage-raw-efi-ext4/ && 
      docker build . -t ${remote_registry}mkimage-raw-efi-ext4:$tag && 
      docker push ${remote_registry}mkimage-raw-efi-ext4:$tag
    )
    image_format="raw-efi-ext4"
else
    image_format="raw-efi"
fi

# build linux kernel
(
  cd contrib/foreign-kernels && 
  docker build -f Dockerfile.rpm.anolis.5.10 . -t linuxkit/kernel:5.10-tdx
)

# build a raw-efi image
bin/linuxkit build --docker examples/dm-crypt-tdx.yml -f $image_format

# push image to alibaba cloud
bin/linuxkit push alibabacloud dm-crypt-tdx-efi.img [...other params]
```
