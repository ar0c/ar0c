---
share: true
title: 部署gitea action用于k8s部署
date modified: 2026-01-14
sharetitle: 2026-01-14-部署gitea action用于k8s部署
date created: 2026-01-14
---

## Action Runner Image

基于官方镜像修改问题  
1. 无nodejs 导致无法checkou

添加用于helm部署的工具

```
from gitea/act_runner:nightly

RUN apk add nodejs
RUN apk add docker curl openssl
RUN curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

## docker-compose

```
services:
  runner:
    container_name: gitea_runner
    image: xxx/act_runner:0.2.11
    #image: docker.io/gitea/act_runner:latest
    environment:
      CONFIG_FILE: /config.yaml
      GITEA_INSTANCE_URL: https://git.xxx.com
      # 从管理后台获取token，用于全局runner
      GITEA_RUNNER_REGISTRATION_TOKEN: xxxxxx
      GITEA_RUNNER_NAME: runner
    ports:
      - 9989:9989
    volumes:
      - ./config.yaml:/config.yaml
      - ./data:/data
      - /var/run/docker.sock:/var/run/docker.sock
```
