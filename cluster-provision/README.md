# k8s clusters in QEMU in docker

* `centos7` adds a vagrant centos7 box to the image
* `cli` contains a tool for provisioning, running and managing the containerized clusters
* `k8s-1.10.11` k8s-1.10.11 cluster based on the centos7 image, provisioned with kubeadm
* `k8s-1.11.0` k8s-1.11.0 cluster based on the centos7 image, provisioned with kubeadm
* `k8s-1.13.3` k8s-1.13.3 cluster based on the centos7 image, provisioned with kubeadm

## Versions to use

* `kubevirtci/cli`: `sha256:1dd015dea4f12e6dcb8e31be3eeb677fed96f290ef4a4892a33c43d666053536`
* `kubevirtci/gocli`: `sha256:220f55f6b1bcb3975d535948d335bd0e6b6297149a3eba1a4c14cad9ac80f80d`
* `kubevirtci/centos:1905_01`: `sha256:4b292b646f382d986c75a2be8ec49119a03467fe26dccc3a0886eb9e6e38c911`
* `kubevirtci/centos:2001_01`: `sha256:6f2548dcc23489d0c945aef516781ae2ea678424c3760d1dafa0a83d29411713`
* `kubevirtci/k8s-1.14.6`: `sha256:ec29c07c94fce22f37a448cb85ca1fb9215d1854f52573316752d19a1c88bcb3`
* `kubevirtci/k8s-1.15.1`: `sha256:14d7b1806f24e527167d2913deafd910ea46e69b830bf0b094dde35ba961b159`
* `kubevirtci/k8s-1.16.2`: `sha256:5bae6a5f3b996952c5ceb4ba12ac635146425909801df89d34a592f3d3502b0c`

## Using gocli

`gocli` is a tiny go binary which helps managing the containerized clusters. It
can be used from a docker images, so no need to install it. You can for instance
use a bash alias:

```bash
alias gocli="docker run --net=host --privileged --rm -it -v /var/run/docker.sock:/var/run/docker.sock kubevirtci/gocli:latest"
gocli help
```

## Quickstart Kubernetes

### Start the cluster

Start a k8s cluster which contains of one master and two nodes:

```bash
gocli run --random-ports --nodes 3 --background kubevirtci/k8s-1.13.3
```

### Connect to the cluster

Find out the connection details of the cluster:

```bash
$ gocli ports k8s
33396
$ gocli scp /etc/kubernetes/admin.conf - > ./kubeconfig
$ kubectl --kubeconfig ./kubeconfig --insecure-skip-tls-verify --server https://localhost:33396 get pods -n kube-system
NAME                             READY     STATUS    RESTARTS   AGE
etcd-node01                      1/1       Running   0          14m
kube-apiserver-node01            1/1       Running   0          13m
kube-controller-manager-node01   1/1       Running   0          14m
kube-dns-6f4fd4bdf-mh6nb         3/3       Running   0          14m
kube-flannel-ds-4bk76            1/1       Running   0          14m
kube-flannel-ds-5zgmt            1/1       Running   1          14m
kube-flannel-ds-qbm2r            1/1       Running   1          14m
kube-proxy-gtvpb                 1/1       Running   0          14m
kube-proxy-knc6p                 1/1       Running   0          14m
kube-proxy-vx9t6                 1/1       Running   0          14m
kube-scheduler-node01            1/1       Running   0          13m
```

or to permamently edit kubeconfig:

```bash
$ gocli scp /etc/kubernetes/admin.conf - > ./kubeconfig
$ kubectl --kubeconfig=./kubeconfig config set-cluster kubernetes --server=https://127.0.0.1:$(gocli ports k8s|tr -d '\r\n')
$ kubectl --kubeconfig=./kubeconfig config set-cluster kubernetes --insecure-skip-tls-verify=true
$ kubectl --kubeconfig ./kubeconfig get pods -n kube-system
NAME                             READY     STATUS    RESTARTS   AGE
etcd-node01                      1/1       Running   0          14m
kube-apiserver-node01            1/1       Running   0          13m
kube-controller-manager-node01   1/1       Running   0          14m
kube-dns-6f4fd4bdf-mh6nb         3/3       Running   0          14m
kube-flannel-ds-4bk76            1/1       Running   0          14m
kube-flannel-ds-5zgmt            1/1       Running   1          14m
kube-flannel-ds-qbm2r            1/1       Running   1          14m
kube-proxy-gtvpb                 1/1       Running   0          14m
kube-proxy-knc6p                 1/1       Running   0          14m
kube-proxy-vx9t6                 1/1       Running   0          14m
kube-scheduler-node01            1/1       Running   0          13m
```

### Destroy the cluster

```bash
$ gocli rm
```

### Accessing the webconsole

Make sure that `node01` resolves to `127.0.0.1` and that you added `--ocp-port
8443` when creatin the cluster. If you did that, you can simply access the
webconsole at `https://127.0.0.1:8443`. The login credentials are
`admin:admin`.

The two preconditions are necessary to make the authentication redirects work.

### Destroy the cluster

```bash
$ gocli rm
```
