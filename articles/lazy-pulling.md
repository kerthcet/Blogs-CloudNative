### What is eStargz?

Standard-Compatible Extension to Container Image Layers for Lazy Pulling. It is a *backward-compatible extension* which means that images can be pushed to the extension-agnostic registry and can run on extension-agnostic runtimes.This extension is based on stargz (stands for *seekable tar.gz*) proposed by [Google CRFS](https://github.com/google/crfs) project (initially [discussed in Go community](https://github.com/golang/go/issues/30829)). eStargz extends stargz for chunk-level verification and runtime performance optimization.

![img](https://miro.medium.com/max/1400/1*aba_56ZY6N-3Y-JuIwllpg.png)

![img](https://miro.medium.com/max/1400/1*jAQCEWTi3jzZX2Rn0XEUAg.png)

### What is Lazy Pulling?

Lazy pulling is a technique of pulling container images aiming at the faster cold start. This allows a container to startup without waiting for the entire image layer contents to be locally available. Instead, necessary files (or chunks for large files) in the layer are fetched *on-demand* during running the container.

### Why Lazy Pulling?

- FAST ON START

![Benchmarking result from the project repository.](https://devopstales.github.io/img/include/lazypull2.png)

- FAST ON BUILD

![img](https://miro.medium.com/max/1400/1*bVC7qorx-sDDXdYu6IfxzQ.png)

### How to use Lazy Pulling

![eStargz in container workflow](https://devopstales.github.io/img/include/lazypull3.png)

### Demo

```bash
$ kind create cluster --name stargz-demo --image ghcr.io/stargz-containers/estargz-kind-node:0.7.0
$ docker exec -it xxx bash
```

1. 镜像格式优化 optimize

```bash
$ ctr-remote --debug i optimize --oci docker.io/jonyhy/esgz:ov1 docker.io/jonyhy/esgz:v1
```

2. 拉取效果

```bash
$ ctr-remote i rpull docker.io/jonyhy/esgz:v1
$ journalctl -u stargz-snapshotter --no-pager --follow
```

3. 启动效果

```bash
$ ctr-remote run --rm -t --snapshotter=stargz docker.io/jonyhy/esgz:v1 test /bin/bash
$ ls /.stargz-snapshotter/*
$ cat /.stargz-snapshotter/*
```

### 产品结合需要满足

版本要求: https://github.com/containerd/stargz-snapshotter/issues/258

1. CRI containerd, podman or cri-o
2. fuse
3. run stargz-snapshotter or stargz-store

### Reference

1. https://medium.com/nttlabs/buildkit-lazypull-66c37690963f
2. https://github.com/containerd/stargz-snapshotter/blob/main/docs/estargz.md
3. https://medium.com/nttlabs/startup-containers-in-lightning-speed-with-lazy-image-distribution-on-containerd-243d94522361
4. https://devopstales.github.io/kubernetes/lazyimage/
5. https://github.com/containerd/stargz-snapshotter/blob/main/Dockerfile
6. https://medium.com/nttlabs/buildkit-lazypull-66c37690963f