---
title: 从零开始部署kubernetes v1.10
date: 2018-03-27 21:01:20
tags: [docker,kubernetes]
categories: kubernetes
---
使用ssh登录到远程服务器，切换到root账号。

# 关闭swap
```
swapoff -a
```
修改 `/etc/fstab`
```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/mapper/cst--01--vg-root /               ext4    errors=remount-ro 0       1
# /boot was on /dev/cciss/c0d0p1 during installation
UUID=331095c3-4a02-450c-ad2b-176f2013b4d2 /boot           ext2    defaults        0       2
#/dev/mapper/cst--01--vg-swap_1 none            swap    sw              0       0
#/dev/mapper/cryptswap1 none swap sw 0 0
```
文件最后一行注释掉。

# 下载二进制文件
[最新release版本](https://github.com/kubernetes/kubernetes/releases/latest)
![image.png](https://upload-images.jianshu.io/upload_images/8053527-0fdb2437a5390bdf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!-- more -->

最下面一行是的链接是二进制文件
[1.10版本](https://dl.k8s.io/v1.10.0/kubernetes.tar.gz)
下载的时候使用了学校的dns服务器发现 storage.googleapis.com 指向了一个内网ip，也就是dns污染。
解决办法：放弃使用学校dns服务器，使用8.8.8.8
修改 `/etc/network/interfaces` 的 `dns-nameservers` 一项(ps：使用的是静态ip所以手动配置过这个文件，如果使用DHCP本方法不适用)

```
tar -xzvf kubernetes.tar.gz # 解压文件
kubernetes/cluster/get-kube-binaries.sh
tar -xzvf kubernetes/server/kubernetes-server-linux-amd64.tar.gz 
```

去到 `kubernetes/server/kubernetes/server/bin` 目录把 `kubectl` `kubelet` `kube-proxy` 拷贝到 `/usr/local/bin`

# 制作证书
## 下载CFSSL

```
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
chmod +x cfssl_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl

wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x cfssljson_linux-amd64
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson

wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl-certinfo_linux-amd64
mv cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo
```
## 制作CA证书
创建工作目录 `/root/ssl`

`ca-config.json` ca配置

```
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF
```

`ca-csr.json` 请求签名文件

```
cat > ca-csr.json << EOF
 {
    "CN": "kubernetes",
    "key": {
      "algo": "rsa",
      "size": 2048
    },
    "names": [
      {
        "C": "CN",
        "ST": "GhuangZhou",
        "L": "GhuangZhou",
        "O": "k8s",
        "OU": "System"
      }
    ]
  }
EOF
```

```
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

`ca-key.pem` ca 的 私钥
`ca.pem` ca 的证书

## 创建 api-server 使用的证书
```
cat > kubernetes-csr.json << EOF
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "202.192.18.63",
      "10.254.0.1",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "GhuangZhou",
            "L": "GhuangZhou",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF
```

```
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
 -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
```

## 创建管理员使用的证书

```
cat > admin-csr.json << EOF
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "GhuangZhou",
      "L": "GhuangZhou",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF
```

```
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
 -profile=kubernetes admin-csr.json | cfssljson -bare admin
```

## 创建 kube-proxy 使用的证书

```
cat > kube-proxy-csr.json << EOF
 {
    "CN": "system:kube-proxy",
    "hosts": [],
    "key": {
      "algo": "rsa",
      "size": 2048
    },
    "names": [
      {
        "C": "CN",
        "ST": "GhuangZhou",
        "L": "GhuangZhou",
        "O": "k8s",
        "OU": "System"
      }
    ]
  }
EOF
```

```
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json \
 -profile=kubernetes  kube-proxy-csr.json | cfssljson -bare kube-proxy
```
## 把证书放到指定目录
```
mkdir /srv/kubernetes
cp ca.pem /srv/kubernetes/ca.crt
cp kubernetes.pem /srv/kubernetes/server.crt
cp kubernetes-key.pem /srv/kubernetes/server.key
cp admin.pem /srv/kubernetes/admin.crt
cp admin-key.pem /srv/kubernetes/admin.key
cp kube-proxy.pem /srv/kubernetes/kube-proxy.crt
cp kube-proxy-key.pem /srv/kubernetes/kube-proxy.key
cp ca-key.pem /srv/kubernetes/ca.key
```

# 配置环境变量

```
export NET_INTERFACE=enp2s2f1
export CA_CERT=/srv/kubernetes/ca.crt
export MASTER_CERT=/srv/kubernetes/server.crt
export MASTER_KEY=/srv/kubernetes/server.key
export MASTER_IP=`ifconfig | grep -A 1 $NET_INTERFACE | tail -n 1 | awk -F':' '{print $2}' | awk '{print $1}'`:6443
export CLUSTER_NAME=kubernetes
export USER=admin
export CLI_CERT=/srv/kubernetes/admin.crt
export CLI_KEY=/srv/kubernetes/admin.key
export CONTEXT_NAME=kubernetes
TOKEN=$(dd if=/dev/urandom bs=128 count=1 2>/dev/null | base64 | tr -d "=+/" | dd bs=32 count=1 2>/dev/null)
mkdir -p /var/lib/kube-apiserver
cat > /var/lib/kube-apiserver/known_tokens.csv << EOF
${TOKEN},kubelet-bootstrap,10001,"system:kubelet-bootstrap"
EOF
```
note：`NET_INTERFACE` 需要修改成在使用的网卡

# 利用 kubeconfig 分发证书
在k8s中分发证书到客户端方便的做法是把证书放到 `kubeconfig` 文件。

```
kubectl config set-cluster $CLUSTER_NAME --certificate-authority=$CA_CERT --embed-certs=true --server=https://$MASTER_IP
kubectl config set-credentials $USER --client-certificate=$CLI_CERT --client-key=$CLI_KEY --embed-certs=true --token=$TOKEN
```

# 生成 kube-proxy 使用的 kubeconfig
```
export MASTER_IP=`ifconfig | grep -A 1 $NET_INTERFACE | tail -n 1 | awk -F':' '{print $2}' | awk '{print $1}'`:6443
kubectl config set-cluster kubernetes \
--certificate-authority=/srv/kubernetes/ca.crt \
--embed-certs=true \
--server=https://${MASTER_IP} \
--kubeconfig=kubeconfig

kubectl config set-credentials kube-proxy \
--client-certificate=/srv/kubernetes/kube-proxy.crt \
--client-key=/srv/kubernetes/kube-proxy.key \
--embed-certs=true \
--kubeconfig=kubeconfig

kubectl config set-context kubernetes \
--cluster=kubernetes \
--user=kube-proxy \
--kubeconfig=kubeconfig

kubectl config use-context kubernetes --kubeconfig=kubeconfig
```
生成的 `kubeconfig` 放到 `/var/lib/kube-proxy/` 目录下（先创建）

# 生成 kubelet 使用的 bootstrap-kubeconfig

```
kubectl config set-cluster kubernetes \
  --certificate-authority=/srv/kubernetes/ca.crt \
  --embed-certs=true \
  --server=https://${MASTER_IP} \
  --kubeconfig=bootstrap.kubeconfig

kubectl config set-credentials kubelet-bootstrap \
  --token=${TOKEN} \
  --kubeconfig=bootstrap.kubeconfig

kubectl config set-context kubernetes \
  --cluster=kubernetes \
  --user=kubelet-bootstrap \
  --kubeconfig=bootstrap.kubeconfig

kubectl config use-context kubernetes --kubeconfig=bootstrap.kubeconfig
```
生成的 `bootstrap.kubeconfig` 放到 `/etc/kubernetes/ssl/` 目录下（先创建）

# 安装 docker

```
apt update
apt install docker.io -y
```

# 配置 kubelet

```
mkdir /etc/systemd/system/kubelet.service.d/

cat > /etc/systemd/system/kubelet.service.d/kubelet.conf << EOF
[Service]
ExecStart=
ExecStart=/usr/local/bin/kubelet \
--kubeconfig=/var/lib/kubelet/kubeconfig \
--pod-manifest-path=/etc/kubernetes/manifests \
--network-plugin=cni \
--cni-conf-dir=/etc/cni/net.d \
--cni-bin-dir=/opt/cni/bin \
--cluster-dns=10.254.0.2 \
--cluster-domain=cluster.local \
--allow-privileged=true \
--feature-gates=RotateKubeletClientCertificate=true,RotateKubeletServerCertificate=true \
--bootstrap-kubeconfig="/etc/kubernetes/ssl/bootstrap.kubeconfig" \
--cert-dir=/etc/kubernetes/ssl \
--rotate-certificates
EOF

cat > /lib/systemd/system/kubelet.service << EOF
[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=http://kubernetes.io/docs/

[Service]
ExecStart=/usr/local/bin/kubelet
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable kubelet.service
systemctl start kubelet.service
```
# 配置 kube-proxy

```
mkdir /etc/systemd/system/kube-proxy.service.d
cat > /etc/systemd/system/kube-proxy.service.d/kube-proxy.conf << EOF
[Service]
ExecStart=
ExecStart=/usr/local/bin/kube-proxy \
--master=https://202.192.18.63:6443 \
--kubeconfig=/var/lib/kube-proxy/kubeconfig
EOF

cat > /lib/systemd/system/kube-proxy.service << EOF
[Unit]
Description=kube-proxy: The Kubernetes Node Agent
Documentation=http://kubernetes.io/docs/

[Service]
ExecStart=/usr/bin/kube-proxy
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable kube-proxy.service
systemctl start kube-proxy.service
```

# 配置文件说明
以下配置文件出现的ip需要和 master 的ip一致
docker 的 images 版本要与k8s版本一致

# 配置 etcd
```
cat > /etc/kubernetes/manifests/etcd.json << EOF
{
"apiVersion": "v1",
"kind": "Pod",
"metadata": {
  "name":"etcd-server",
  "namespace": "kube-system",
  "annotations": {
    "scheduler.alpha.kubernetes.io/critical-pod": ""
  }
},
"spec":{
"hostNetwork": true,
"containers":[
    {
    "name": "etcd-container",
    "image": "gcr.io/google-containers/etcd:3.2.16",
    "command": [
              "/bin/sh",
              "-c",
              "if [ -e /usr/local/bin/migrate-if-needed.sh ]; then /usr/local/bin/migrate-if-needed.sh 1>>/var/log/etcd.log 2>&1; fi; exec /usr/local/bin/etcd --name node1 --listen-peer-urls https://0.0.0.0:2380 --initial-advertise-peer-urls https://202.192.18.63:2380 --advertise-client-urls http://202.192.18.63:2379 --listen-client-urls http://127.0.0.1:2379  --data-dir /var/etcd/data --initial-cluster-state new --initial-cluster node1=https://202.192.18.63:2380 1>>/var/log/etcd.log 2>&1"
            ],
    "env": [
      { "name": "TARGET_STORAGE",
        "value": "etcd3"
      },
      { "name": "TARGET_VERSION",
        "value": "3.2.16"
      },
      { "name": "DATA_DIRECTORY",
        "value": "/var/etcd/data"
      },
      { "name": "INITIAL_CLUSTER",
        "value": "node1=https://202.192.18.63:2380"
      },
      { "name": "ETCD_CERT_FILE",
        "value": "/srv/kubernetes/server.crt"
      },
      { "name": "ETCD_CA_FILE",
        "value": "/srv/kubernetes/ca.crt"
      },
      { "name": "ETCD_KEY_FILE",
        "value": "/srv/kubernetes/server.key"
      },
      { "name": "ETCD_CLIENT_CERT_AUTH",
        "value": "/srv/kubernetes/server.crt"
      },
      { "name": "ETCD_TRUSTED_CA_FILE",
        "value": "/srv/kubernetes/ca.crt"
      },
      { "name": "ETCD_PEER_CERT_FILE",
        "value": "/srv/kubernetes/server.crt"
      },
      { "name": "ETCD_PEER_KEY_FILE",
        "value": "/srv/kubernetes/server.key"
      },
      { "name": "ETCD_PEER_TRUSTED_CA_FILE",
        "value": "/srv/kubernetes/ca.crt"
      }
        ],
    "livenessProbe": {
      "httpGet": {
        "host": "127.0.0.1",
        "port": 2379,
        "path": "/health"
      },
      "initialDelaySeconds": 15,
      "timeoutSeconds": 15
    },
    "ports": [
      { "name": "serverport",
        "containerPort": 2380,
        "hostPort": 2380 
      },
      { "name": "clientport",
        "containerPort": 2379,
        "hostPort": 2379
      }
        ],
    "volumeMounts": [
      { "name": "varetcd",
        "mountPath": "/var/etcd",
        "readOnly": false
      },
      { "name": "varlogetcd",
        "mountPath": "/var/log/etcd.log",
        "readOnly": false
      },
      { "name": "etc",
        "mountPath": "/var/lib/etcd",
        "readOnly": false
      },
      { "name": "crt",
        "mountPath": "/srv/kubernetes",
        "readOnly": true
      }
    ]
    }
],
"volumes":[
  { "name": "varetcd",
    "hostPath": {
        "path": "/mnt/master-pd/var/etcd"}
  },
  { "name": "varlogetcd",
    "hostPath": {
        "path": "/var/log/etcd.log",
        "type": "FileOrCreate"}
  },
  { "name": "etc",
    "hostPath": {
        "path": "/var/lib/etcd"}
  },
  { "name": "crt",
    "hostPath": {
        "path": "/srv/kubernetes"}
  }
]
}}
EOF
```

# 配置 kube-apiserver

```
cat > /etc/kubernetes/manifests/kube-apiserver.json << EOF
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "kube-apiserver"
  },
  "spec": {
    "hostNetwork": true,
    "containers": [
      {
        "name": "kube-apiserver",
        "image": "gcr.io/google-containers/hyperkube:v1.10.0",
        "command": [
          "/hyperkube",
          "apiserver",
          "--etcd-servers=http://127.0.0.1:2379",
          "--tls-cert-file=/srv/kubernetes/server.crt",
          "--tls-private-key-file=/srv/kubernetes/server.key",
          "--service-account-key-file=/srv/kubernetes/server.key",
          "--allow-privileged=true",
          "--client-ca-file=/srv/kubernetes/ca.crt",
          "--admission-control=Initializers,NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota",
          "--token-auth-file=/var/lib/kube-apiserver/known_tokens.csv",
          "--service-cluster-ip-range=10.254.0.0/16"
        ],
        "ports": [
          {
            "name": "https",
            "hostPort": 6443,
            "containerPort": 6443
          },
          {
            "name": "local",
            "hostPort": 8080,
            "containerPort": 8080
          }
        ],
        "volumeMounts": [
          {
            "name": "srvkube",
            "mountPath": "/srv/kubernetes",
            "readOnly": true
          },
          {
            "name": "etcssl",
            "mountPath": "/etc/ssl",
            "readOnly": true
          },
          { 
            "name": "apilog",
            "mountPath": "/var/log/apiserver.log",
            "readOnly": false
          },
          {
            "name": "varlibkube",
            "mountPath": "/var/lib/kube-apiserver",
            "readOnly": true
          }
        ],
        "livenessProbe": {
          "httpGet": {
            "scheme": "HTTP",
            "host": "127.0.0.1",
            "port": 8080,
            "path": "/healthz"
          },
          "initialDelaySeconds": 15,
          "timeoutSeconds": 15
        }
      }
    ],
    "volumes": [
      {
        "name": "srvkube",
        "hostPath": {
          "path": "/srv/kubernetes"
        }
      },
      {
        "name": "etcssl",
        "hostPath": {
          "path": "/etc/ssl"
        }
      },
      {
        "name": "apilog",
        "hostPath": {
          "path": "/var/log/apiserver.log",
          "type": "FileOrCreate"
        }
      },
      {
        "name": "varlibkube",
        "hostPath": {
          "path": "/var/lib/kube-apiserver"
        }
      }
    ]
  }
}
EOF
```

# 配置 kube-controller-manager

```
cat > /etc/kubernetes/manifests/kube-controller-manager.json << EOF
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "kube-controller-manager"
  },
  "spec": {
    "hostNetwork": true,
    "containers": [
      {
        "name": "kube-controller-manager",
        "image": "gcr.io/google-containers/hyperkube:v1.10.0",
        "command": [
          "/hyperkube",
          "controller-manager",
          "--cluster-cidr=10.244.0.0/16",
          "--allocate-node-cidrs=true",
          "--service-cluster-ip-range=10.254.0.0/16",
          "--service-account-private-key-file=/srv/kubernetes/server.key",
          "--cluster-signing-cert-file=/srv/kubernetes/ca.crt",
          "--cluster-signing-key-file=/srv/kubernetes/ca.key",
          "--root-ca-file=/srv/kubernetes/ca.crt",
          "--feature-gates=RotateKubeletServerCertificate=true",
          "--master=127.0.0.1:8080"
        ],
        "volumeMounts": [
          {
            "name": "srvkube",
            "mountPath": "/srv/kubernetes",
            "readOnly": true
          },
          {
            "name": "etcssl",
            "mountPath": "/etc/ssl",
            "readOnly": true
          }
        ],
        "livenessProbe": {
          "httpGet": {
            "scheme": "HTTP",
            "host": "127.0.0.1",
            "port": 10252,
            "path": "/healthz"
          },
          "initialDelaySeconds": 15,
          "timeoutSeconds": 15
        }
      }
    ],
    "volumes": [
      {
        "name": "srvkube",
        "hostPath": {
          "path": "/srv/kubernetes"
        }
      },
      {
        "name": "etcssl",
        "hostPath": {
          "path": "/etc/ssl"
        }
      }
    ]
  }
}
EOF
```

# 配置 kube-scheduler

```
cat > /etc/kubernetes/manifests/kube-scheduler.json << EOF
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "kube-scheduler"
  },
  "spec": {
    "hostNetwork": true,
    "containers": [
      {
        "name": "kube-scheduler",
        "image": "gcr.io/google-containers/hyperkube:v1.10.0",
        "command": [
          "/hyperkube",
          "scheduler",
          "--master=127.0.0.1:8080"
        ],
        "volumeMounts": [
          {
            "name": "srvkube",
            "mountPath": "/srv/kubernetes",
            "readOnly": true
          }
        ],
        "livenessProbe": {
          "httpGet": {
            "scheme": "HTTP",
            "host": "127.0.0.1",
            "port": 10251,
            "path": "/healthz"
          },
          "initialDelaySeconds": 15,
          "timeoutSeconds": 15
        }
      }
    ],
    "volumes": [
      {
        "name": "srvkube",
        "hostPath": {
          "path": "/srv/kubernetes"
        }
      }
    ]
  }
}
EOF
```

# master kubelet 的 kubeconfig
由于集群还没初始化，所以kubelet 的 kubeconfig 使用 kube-proxy 的（node 不需要）。

# 安装 cni

```
# 创建 CNI 目录
mkdir -p /opt/cni/bin
# 下载 CNI 插件
wget https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz
tar -zxvf cni-plugins-amd64-v0.6.0.tgz
# 移动 CNI 插件
mv bridge flannel host-local loopback /opt/cni/bin
```

# 安装 Flannel
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
```

# 配置 TLS bootstrapping

```
# 自动批准 system:bootstrappers 组用户 TLS bootstrapping 首次申请证书的 CSR 请求
kubectl create clusterrolebinding node-client-auto-approve-csr --clusterrole=approve-node-client-csr --group=system:bootstrappers

# 自动批准 system:nodes 组用户更新 kubelet 自身与 apiserver 通讯证书的 CSR 请求
kubectl create clusterrolebinding node-client-auto-renew-crt --clusterrole=approve-node-client-renewal-csr --group=system:nodes

# 自动批准 system:nodes 组用户更新 kubelet 10250 api 端口证书的 CSR 请求
kubectl create clusterrolebinding node-server-auto-renew-crt --clusterrole=approve-node-server-renewal-csr --group=system:nodes
```

测试是否自动生成`kubelet`的`kubeconfig`：删掉`kubelet`的`kubeconfig`然后重启`kubelet`。

```
kubectl get csr
```
可以看到请求自动通过。

# kube-dns

可以使用[这个](https://github.com/kubernetes/kubernetes/blob/master/cluster/addons/dns/kube-dns.yaml.base)

把 `__PILLAR__DNS__SERVER__` 修改为 10.254.0.2
把 `__PILLAR__DNS__DOMAIN__` 修改为 kubernetes.default.svc.cluster.local
