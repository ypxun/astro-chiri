---
title: Overleaf与AI
pubDate: '2025-04-23'
---

## 前言
Overleaf是一款协作式的云端LaTeX编辑工具。以前上学的时候，软件工程课的老师要求我们使用该工具和小组组员协作写报告。刚接触时，一头雾水，觉得它比word麻烦。不过随着写报告过程中的使用与熟悉，发现了它的便利与强大。先让AI总结下它的优缺点：
| 优点    | 缺点 |
| -------- | ------- |
| 在线协作  | 依赖网络(国内最好代理连接)    |
| 跨平台与便捷性 | 编译速度与稳定性     |
| 模板丰富    | 学习曲线    |
| 集成化功能(PDF,Zotero,Git)    | 自定义性受限    |
| 免费基础功能够用    | 高级功能需付费    |
| 云存储与分享    | 隐私与数据安全    |

说下我自己的看法：
1. 命令确实繁多，对新手不友善。但实际使用中，最常用的命令只有图片、表格和引文这关键三项。附上[官方教程](https://www.overleaf.com/learn/latex/Tutorials)。

2. 从零构建项目费点事，但模板实在是太多了。大学或机构（特别是国外）格式要求严格的各种形式的文档、报告、论文模板等等几乎都有人上传，而且一般都带有使用说明。附上[模板市场](https://www.overleaf.com/latex/templates)

3. 云端编辑，实时保存，所以几乎不会丢失最新进度。

所以后面不少课程的报告，包括最后61页的毕业论文，我都是用Overleaf完成的。工作后有些文书工作，我还在继续使用Overleaf

自部署Overleaf的想法，始于官方端免费账户的限制（编译时间限制已经缩短到20s）以及国内网络连接的稳定性。20s编译时限这一刀砍得太厉害，对于小报告来说，没啥影响。但对于稍大些的项目，编译时长很容易超限，导致编译失败。另外，部署在国内VPS上，能保证网络连接的稳定性。
## 部署
官方推荐使用[Overleaf Toolkit](https://github.com/overleaf/toolkit)部署和管理，不过也支持Docker部署。个人更习惯于使用Docker搭建。
### 构建镜像包
Overleaf只提供了amd64架构的镜像包，而我的VPS是arm64架构，因此需要依照[官方所说](https://github.com/overleaf/overleaf?tab=readme-ov-file#overleaf-docker-image)，利用[`Dockerfile-base`](https://github.com/overleaf/overleaf/blob/main/server-ce/Dockerfile-base)和[`Dockerfile`](https://github.com/overleaf/overleaf/blob/main/server-ce/Dockerfile)分别构建arm64架构的`sharelatex/sharelatex-base`和`sharelatex/sharelatex`。

Fork源仓库后，编写自动化文档，完成自动追随上游代码更新和自动生成最新的基于arm64架构的镜像包发布到GitHub上,代码详见[ypxun/overleaf](https://github.com/ypxun/overleaf/blob/main/.github/workflows/build-arm64-overleaf.yml)<br>
耗时1h30min+后，镜像包构建完成，地址：[ypxun/overleaf_packages](https://github.com/ypxun?tab=packages&repo_name=overleaf)
### 搭建容器
官方的[compose文档](https://github.com/overleaf/overleaf/blob/main/docker-compose.yml)，按需修改，并注释Pro用户相关参数。
```yml
services:
  sharelatex:
    restart: always
    # Server Pro users:
    # image: quay.io/sharelatex/sharelatex-pro
    image: ghcr.io/ypxun/sharelatex:arm64-latest
    container_name: sharelatex
    depends_on:
      mongo:
        condition: service_healthy
      redis:
        condition: service_started
    ports:
        - 80:80
    stop_grace_period: 60s
    volumes:
            - ~/sharelatex_data:/var/lib/overleaf
      ########################################################################
      ####  Server Pro: Uncomment the following line to mount the docker  ####
      ####             socket, required for Sibling Containers to work    ####
      ########################################################################
      # - /var/run/docker.sock:/var/run/docker.sock
    environment:
      OVERLEAF_APP_NAME: Overleaf Community Edition
      OVERLEAF_MONGO_URL: mongodb://mongo/sharelatex
      # Same property, unfortunately with different names in
      # different locations
      OVERLEAF_REDIS_HOST: redis
      REDIS_HOST: redis
      ENABLED_LINKED_FILE_TYPES: project_file,project_output_file
      # Enables Thumbnail generation using ImageMagick
      ENABLE_CONVERSIONS: "true"
      # Disables email confirmation requirement
      EMAIL_CONFIRMATION_DISABLED: "true"
      ## Set for SSL via nginx-proxy
      #VIRTUAL_HOST: 103.112.212.22

      OVERLEAF_SECURE_COOKIE: true
      OVERLEAF_BEHIND_PROXY: true
      # OVERLEAF_SITE_URL: http://overleaf.example.com
      # OVERLEAF_NAV_TITLE: Overleaf Community Edition
      # OVERLEAF_HEADER_IMAGE_URL: http://example.com/mylogo.png
      # OVERLEAF_ADMIN_EMAIL: support@it.com

      # OVERLEAF_LEFT_FOOTER: '[{"text": "Another page I want to link to can be found <a href=\"here\">here</a>"} ]'
      # OVERLEAF_RIGHT_FOOTER: '[{"text": "Hello I am on the Right"} ]'

      # OVERLEAF_EMAIL_FROM_ADDRESS: "hello@example.com"

      # OVERLEAF_EMAIL_AWS_SES_ACCESS_KEY_ID:
      # OVERLEAF_EMAIL_AWS_SES_SECRET_KEY:

      # OVERLEAF_EMAIL_SMTP_HOST: smtp.example.com
      # OVERLEAF_EMAIL_SMTP_PORT: 587
      # OVERLEAF_EMAIL_SMTP_SECURE: false
      # OVERLEAF_EMAIL_SMTP_USER:
      # OVERLEAF_EMAIL_SMTP_PASS:
      # OVERLEAF_EMAIL_SMTP_TLS_REJECT_UNAUTH: true
      # OVERLEAF_EMAIL_SMTP_IGNORE_TLS: false
      # OVERLEAF_EMAIL_SMTP_NAME: '127.0.0.1'
      # OVERLEAF_EMAIL_SMTP_LOGGER: true
      # OVERLEAF_CUSTOM_EMAIL_FOOTER: "This system is run by department x"

      # ENABLE_CRON_RESOURCE_DELETION: true

      ################
      ## Server Pro ##
      ################

      ## Sandboxed Compiles: https://github.com/overleaf/overleaf/wiki/Server-Pro:-Sandboxed-Compiles
      # SANDBOXED_COMPILES: 'true'
      # SANDBOXED_COMPILES_SIBLING_CONTAINERS: 'true'
      ### Bind-mount source for /var/lib/overleaf/data/compiles inside the container.
      # SANDBOXED_COMPILES_HOST_DIR_COMPILES: '/home/user/sharelatex_data/data/compiles'
      ### Bind-mount source for /var/lib/overleaf/data/output inside the container.
      # SANDBOXED_COMPILES_HOST_DIR_OUTPUT: '/home/user/sharelatex_data/data/output'

      ## Works with test LDAP server shown at bottom of docker compose
      # OVERLEAF_LDAP_URL: 'ldap://ldap:389'
      # OVERLEAF_LDAP_SEARCH_BASE: 'ou=people,dc=planetexpress,dc=com'
      # OVERLEAF_LDAP_SEARCH_FILTER: '(uid={{username}})'
      # OVERLEAF_LDAP_BIND_DN: 'cn=admin,dc=planetexpress,dc=com'
      # OVERLEAF_LDAP_BIND_CREDENTIALS: 'GoodNewsEveryone'
      # OVERLEAF_LDAP_EMAIL_ATT: 'mail'
      # OVERLEAF_LDAP_NAME_ATT: 'cn'
      # OVERLEAF_LDAP_LAST_NAME_ATT: 'sn'
      # OVERLEAF_LDAP_UPDATE_USER_DETAILS_ON_LOGIN: 'true'

      # OVERLEAF_TEMPLATES_USER_ID: "578773160210479700917ee5"
      # OVERLEAF_NEW_PROJECT_TEMPLATE_LINKS: '[ {"name":"All Templates","url":"/templates/all"}]'


      # OVERLEAF_PROXY_LEARN: "true"

  mongo:
    restart: always
    image: mongo:6.0
    container_name: sharelatex-mongo
    command: --replSet overleaf
    volumes:
        - ~/mongo_data:/data/db
        - ./bin/shared/mongodb-init-replica-set.js:/docker-entrypoint-initdb.d/mongodb-init-replica-set.js
    environment:
      MONGO_INITDB_DATABASE: sharelatex
    extra_hosts:
      # Required when using the automatic database setup for initializing the replica set.
      # This override is not needed when running the setup after starting up mongo.
      - mongo:127.0.0.1
    healthcheck:
      test: echo 'db.stats().ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 10s
      retries: 5
  redis:
    restart: always
    image: redis:6.2
    container_name: sharelatex-redis
    volumes:
        - ~/redis_data:/data  
#   ldap:
  #    restart: always
  #    image: rroemhild/test-openldap
  #    container_name: ldap

  # See https://github.com/jwilder/nginx-proxy for documentation on how to configure the nginx-proxy container,
  # and https://github.com/overleaf/overleaf/wiki/HTTPS-reverse-proxy-using-Nginx for an example of some recommended
  # settings. We recommend using a properly managed nginx instance outside of the Overleaf Server Pro setup,
  # but the example here can be used if you'd prefer to run everything with docker-compose

  # nginx-proxy:
  #     image: jwilder/nginx-proxy
  #     container_name: nginx-proxy
  #     ports:
  #       - "80:80"
  #       - "443:443"
  #     volumes:
  #       - /var/run/docker.sock:/tmp/docker.sock:ro
  #       - /home/overleaf/tmp:/etc/nginx/certs

```
### Nginx反代
[官方反代文档](https://github.com/overleaf/overleaf/wiki/HTTPS-reverse-proxy-using-Nginx#example-nginx-ssl-config)，虽然标记为已过时，但实际可用。

至此，Overleaf搭建完成，网页前端设置Admin账户后，就可以开始使用了。
## 感想
标题说的AI，似乎并没有提及，其实就在最关键的构建镜像包那一步。平常我也有用ChatGPT、Claude和DeepSeek，只是没想到现在的AI强到这个地步。最惊讶到我的是我仅仅使用一段不长的普通的人类语句询问Claude 3.7 Sonnet，就一步成型自动化配置文档，文档中的代码一句不用改，完美运行。AI现在的实力真是恐怖如斯，也难怪现在的通用智能体如火如荼。