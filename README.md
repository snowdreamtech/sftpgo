# SFTPGo

[![dockeri.co](https://dockerico.blankenship.io/image/snowdreamtech/sftpgo)](https://hub.docker.com/r/snowdreamtech/sftpgo)

Docker Image packaging for [drakkan/sftpgo](https://github.com/drakkan/sftpgo). (amd64, arm32v5,  arm32v6, arm32v7, arm64v8, i386, mips64le, ppc64le,riscv64, s390x)

# Usage

To help you get started creating a container from this image you can either use docker-compose or the docker cli.

## Docker Cli

### Simple

```bash
docker run -d \
  --name=sftpgo \
  -e TZ=Asia/Shanghai \
  -p 8080:8080 \
  -p 2022:2022 \
  --restart unless-stopped \
  snowdreamtech/sftpgo:latest
```

### Advance

```bash
docker run -d \
  --name=sftpgo \
  -e TZ=Asia/Shanghai \
  -p 8080:8080 \
  -p 2022:2022 \
  -v /path/to/data:/srv/sftpgo \
  -v /path/to/config:/var/lib/sftpgo \
  --restart unless-stopped \
  snowdreamtech/sftpgo:latest
```

## Docker Compose

### Simple

```bash
services:
  sftpgo:
    image: snowdreamtech/sftpgo:latest
    container_name: sftpgo
    environment:
      - TZ=Asia/Shanghai
    ports:
      - "8080:8080"
      - "2022:2022"
    restart: unless-stopped
```

### Advance

```bash
services:
  sftpgo:
    image: snowdreamtech/sftpgo:latest
    container_name: sftpgo
    environment:
      - TZ=Asia/Shanghai
    ports:
      - "8080:8080"
      - "2022:2022"
    volumes:
      - /path/to/data:/srv/sftpgo
      - /path/to/config:/var/lib/sftpgo
    restart: unless-stopped
```

# Development

```bash
docker buildx create --use --name build --node build --driver-opt network=host
docker buildx build -t snowdreamtech/sftpgo --platform=linux/386,linux/amd64,linux/arm/v6,linux/arm/v7,linux/arm64,linux/ppc64le,linux/riscv64,linux/s390x . --push
```

## Reference

1. [使用 buildx 构建多平台 Docker 镜像](https://icloudnative.io/posts/multiarch-docker-with-buildx/)
1. [如何使用 docker buildx 构建跨平台 Go 镜像](https://waynerv.com/posts/building-multi-architecture-images-with-docker-buildx/#buildx-%E7%9A%84%E8%B7%A8%E5%B9%B3%E5%8F%B0%E6%9E%84%E5%BB%BA%E7%AD%96%E7%95%A5)
1. [Building Multi-Arch Images for Arm and x86 with Docker Desktop](https://www.docker.com/blog/multi-arch-images/)
1. [How to Rapidly Build Multi-Architecture Images with Buildx](https://www.docker.com/blog/how-to-rapidly-build-multi-architecture-images-with-buildx/)
1. [Faster Multi-Platform Builds: Dockerfile Cross-Compilation Guide](https://www.docker.com/blog/faster-multi-platform-builds-dockerfile-cross-compilation-guide/)
1. [docker/buildx](https://github.com/docker/buildx)

## Contact (备注：base)

* Email: <sn0wdr1am@qq.com>
* QQ: 3217680847
* QQ群: 949022145
* WeChat/微信群: sn0wdr1am

## License

AGPL-3.0 license
