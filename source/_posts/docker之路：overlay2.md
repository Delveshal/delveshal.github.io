---
title: docker之路：overlay2
date: 2018-06-11 16:27:06
tags: [docker,overlay2]
categories: docker
---

# docker之路：overlay2
> If your Linux kernel is version 4.0 or higher, and you use Docker CE, consider using the newer [overlay2](https://docs.docker.com/storage/storagedriver/overlayfs-driver/), which has potential performance advantages over the `aufs` storage driver.

docker 官方建议如果可以的话建议使用`overlay2`.

# overlay 架构图
![overlay.png](https://upload-images.jianshu.io/upload_images/8053527-45b4fcfb548f6040.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

overlay 有三层，`merge`，`upperdir`，`lowerdir`。`upperdir` 是可读写层，对应docker的容器层。`lowerdir` 是只读层，对应docker的镜像层。`merged` 是前面两层的统一视图，`upperdir` 的文件可以屏蔽`lowerdir`相同的文件。

<!-- more -->
# docker中的overlay2

如果使用 `overlay2` 文件系统那么在路径 `/var/lib/docker/overlay2` 下就存在你所下载的镜像文件

```
root@delveshal-GE62-2QD:/var/lib/docker/overlay2# ls
059aba775253e332d667164e4605d0d7279cc43fbb3666c2a22c900e705474d1  4c9b3636a5266dadfe9e29ca9e8a11fdcf96b836bf0b474bf4bbcfaf39e480b3  bc2741f691505a9b45734ca6676d032e4e3dac6e743e7bf5cc9618fce037a714
07855f6934fe9ef44dd14a929963bab9204c77d1204f2778433465f163e39823  591c5660ce0cf2b3999be54c78ce6a80de4053de6d23283b8d075a405b0d13f5  c262bf426fe60b5efa7ff670f76f01c99286f913c210980214e8777386b324e7
097215ce8005734e5590387af6fe7c7d721d00624dcc3ef42a73132e5789a48b  5beb5c50d707a856159e1fc3243fd809679dc5550899afb43dc1faa23db49fac  c292ac32bed77e01f621f47454b10a3e3e530bcdcfebf13df3405d4edb33f55d
153cbbe11c1c33b9bf5bb4d5cce83098cb647d3e0f65252e2cbb84c410524c52  69e929809908351db926cb4d7fca23500c1d72725b1686ee62b2520fe267617a  d21d411d2600bf6140355be5722ebe28df15d5f8b1ca46e30c53eb3e6776918b
15c189d06970c8e714ee651707c0cf9fa86b8037cc10eedcb6069dc093ae9056  6c0da9e8389be575e4f74c181f780b09645cc14789965b075b3b07c91c487731  d83ca11207b71d0c822176f1e3f2ba893a36ba8ad937985b4a38472360e4637e
189934b1a1f2ea7eb36ea48b9d4b235ce05f906d7a4bc2df5bbecd32096a4a2e  7b5409c090a293da9132bb7c58c555aa32488ddfd38f0e27b69f6b022b62adaf  e3e533e8b97b4a2772c113a6ea84f6c0dc0f8a61f9a5ed8c574ad587b5324a26
1de1629850d0e6fa956f7fd7bb3de03fe21c655e78b8c0da7fa177378e6a5b28  89d6622ec69013c8594adfa5923ecf159016c68994e7c6d002becf8fefa29295  ec97bbe80bd49d400f9fff3090e7f43c6be39df2fecb21270f4f26075505010e
2045d6d4cf40dafdbffdebc05ca4f3b99032b0418a2530970b050d147c75c600  8ab8e18aecf4ffc2a6601127db7ec70f7bcc8c6bfbc6784049b4f7ba25a37d0a  fa66e2d7ef4d3389a6faaf83d9308b3ae833c9c969bb48a0486e274813377731
3be605058803f661e71cf7c87ca331503bac791929cd27707ab0248430afb8d9  a6c02694661957bb8a3e5115e3ac82ed7021328d752e06c35d163a0e42f1c02d  ff0426547f42d69bad641f4e249d5fac8031d7a5d6638ec24b8170974f0510df
4635595a864efdf1b8ec4f3200e34fed61bb22b65a865714220b40290f7eddef  a7414fcdb9fede6a0f9ed9bef0f7200626bc98ceee070faa8786788623d295cd  l
4c9a0154462f58e36e6256b9604d952915274391f0a126bed8aabdd6bc011fb3  a88d15780625a6237074f1f75dd5446c076cd783d178339d3d0b3e998e7b6d78
```
最后有一个命名为`l`的文件夹，里面保存着链接文件

```
root@delveshal-GE62-2QD:/var/lib/docker/overlay2/l# ls
2KJUYMKFQLXKCADGHDA3NQZNYI  ASSA23YL7RM4LDLAE3FF3HATPB  CKT7NSMC5BAZCL7MVCUWZLA7S5  HRCS2NHTTAHO5GRKWXKJUOGD52  P2GFYTQOHTYUMYKMREXWO3AVAC  SXOD2BDPK5KANFBKQM4U3HLXVO  Z2IY74RSEKYZT3QCPZPN2KXGIW
2QREFX3OBTX4HT3GUSBECF376W  BCOKP4SLSMIUGCD3AH7AXXJWUO  COXIVWC2VHIRKW6O2WIZLHON7V  HUCJ5HHL6UBDCTVEIOH3LNP637  PQXCEOL5T3OILQVRGCV5HS672O  TLK7SKPVTSKXT5TODQHLNSDB2R
3HMHAYUV7JGUQ373OPRP5ZOC7U  BMFCZJ372FZN66DQXCK25J3CZ4  CQM7NXLNY2TQKCVILMOGGR6SGQ  JZKXTABJC47XZNNBDLCNDNUUYS  RKLQ37QVAG6AJHBNGO6B7QJNHM  VREXSZFB6MCTR577WH6XJXOHZZ
6WKIFVQ6BBLU4BMKD75GBLM7CL  CBCN6G625V7FCOZESRITIZACBL  EFVHQXB3PDMJV5FNOT22VHLB4C  LEKXA6AG2URC67WVT4NM327DKU  RTMEYBWMZHUVYO45RAZ7CGTQNR  WIZ7EUDS7H5HRZA7W4OF7LKHPK
7VDX36V6JQVOPY3M2RPUPZHFEB  CFXJIZNBWQTOCW46RG2BIW5XPA  FQLIEZJ3YTRGXEYTFAWIOEMWC4  N5YKZF47OI5F32R54HGZ3S4FIF  S33PRYV4VFALWZ6EC7O5PPQJAC  XJIASH5JLRDKGLSUUB5NZEJDVK
root@delveshal-GE62-2QD:/var/lib/docker/overlay2/l# readlink 2KJUYMKFQLXKCADGHDA3NQZNYI
../591c5660ce0cf2b3999be54c78ce6a80de4053de6d23283b8d075a405b0d13f5/diff
```
现在启动一个busybox容器
```
delveshal@delveshal-GE62-2QD:/var/lib/docker$ docker run -it busybox
```

`/var/lib/docker/overlay2`文件下就会多了两个文件夹

```
8710aaa81131d37e527a5d11cb61316e0446d3b25b87a5f0b398561cbd7f5b7e
8710aaa81131d37e527a5d11cb61316e0446d3b25b87a5f0b398561cbd7f5b7e-init
```

`8710aaa...-init` 文件夹是docker自动生成的最上层只读层，主要放置了容器运行所必要的文件系统
```
root@delveshal-GE62-2QD:/var/lib/docker/overlay2/8710aaa81131d37e527a5d11cb61316e0446d3b25b87a5f0b398561cbd7f5b7e-init/diff# ls
dev  etc  proc  sys
```

例如 `proc` 进程信息，`etc` hostname，resolve.conf等

`8710aaa...` 文件夹是 docker 容器的读写层

```
root@delveshal-GE62-2QD:/var/lib/docker/overlay2/8710aaa81131d37e527a5d11cb61316e0446d3b25b87a5f0b398561cbd7f5b7e# tree -L 1
.
├── diff
├── link
├── lower
├── merged
└── work

3 directories, 2 files
```

`diff` 是 overlay2 的`upperdir` 是一个文件夹。
`lower` 是 overlay2 的 `lowerdir`，但是是一个文本文件，里面存储的是链接路径，其指向镜像文件夹。
`merge` 是容器的根文件系统，并且只会在容器运行的时候才会出现。
```
root@delveshal-GE62-2QD:/var/lib/docker/overlay2/8710aaa81131d37e527a5d11cb61316e0446d3b25b87a5f0b398561cbd7f5b7e# cat lower
l/FV4T7O5OSNV77P2YSY4RHIC5Y2:l/QSZYTEU3AVKLJEQM7DGER2KL7F
```

在容器里面往`/root/` 写入一个文件
```
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # cd root/
~ # ls
~ # cat>test.txt<<EOF
> test
> EOF
```

那么文件就会出现在
```
root@delveshal-GE62-2QD:/var/lib/docker/overlay2/8710aaa81131d37e527a5d11cb61316e0446d3b25b87a5f0b398561cbd7f5b7e/diff/root# ls
test.txt
```
