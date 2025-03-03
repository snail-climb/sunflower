# 🌻Sunflower

## 项目简介

> Sunflower 是一款专为团队项目部署设计的 GitLab CI/CD 流水线管理工具，旨在通过统一管理与高效整合，优化团队项目的构建、测试和部署流程。

## 目录结构

```
project/
├── eagle/
│   ├── ci.yaml          # 前端项目 CI 配置
│   ├── cd.yaml          # 前端项目 CD 配置
│   └── variables.yaml   # 前端项目 Eagle 变量配置
└── dolphin/
    ├── ci.yaml          # 后端项目 CI 配置
    ├── cd.yaml          # 后端项目 CD 配置
    └── variables.yaml   # 后端项目 Dolphin 变量配置
```

## 使用说明

### 1. **变量配置**

   在 variables.yaml 文件中，配置了项目部署所需的各类参数，包括 Maven 和 Docker 的认证信息、镜像设置、K8S 部署配置，以及 GitLab CI/CD 流水线的触发参数。以下是 variables.yaml 的示例配置：

   ```yaml
   .variables:
     variables:
       # 项目名称
       PROJECT_NAME: dolphin

       # Maven 配置
       MAVEN_URL: http://nexus.sky.org
       MAVEN_USERNAME: nexus
       MAVEN_PASSWORD: admin0527

       # Docker 配置
       REGISTRY_ENDPOINT: harbor.sky.org
       REGISTRY_NAMESPACE: $CI_COMMIT_BRANCH
       DOCKER_USERNAME: harbor
       DOCKER_PASSWORD: admin0527

       # 镜像配置
       IMAGE_NAME: $REGISTRY_ENDPOINT/$REGISTRY_NAMESPACE/$PROJECT_NAME
       IMAGE_VERSION: $CI_PIPELINE_IID
       IMAGE_ADDRESS: $IMAGE_NAME:$IMAGE_VERSION
       IMAGE_PORT: 3000:3000

       # K8S 配置 (非 K8S 环境部署, 无需配置)
       K8S_URL: http://k8s.sky.org
       K8S_CLUSTER_NAME: sky-k8s-cluster
       K8S_NAMESPACE: $CI_COMMIT_BRANCH
       K8S_DEPLOYMENT_NAME: $PROJECT_NAME-$CI_COMMIT_BRANCH
       K8S_USERNAME: admin
       K8S_ACCESS_KEY: xxxxxxxx.xxxxxxxxx
       K8S_UPDATE_IMAGE_TAG_URL: $K8S_URL/kuboard-api/cluster/$K8S_CLUSTER_NAME/kind/CICDApi/admin/resource/updateImageTag
       K8S_RESTART_WORK_LOAD_URL: $K8S_URL/kuboard-api/cluster/$K8S_CLUSTER_NAME/kind/CICDApi/admin/resource/restartWorkload

       # GitLab 配置 (非 GitLab 触发器形式部署, 无需配置)
       GITLAB_URL: http://gitlab.sky.org
       CI_TAG: gemini_ci
       CD_TAG: gemini_cd
       GITLAB_PROJECT_ID: 1
       PIPELINE_TRIGGER_URL: $GITLAB_URL/api/v4/projects/$GITLAB_PROJECT_ID/trigger/pipeline
       PIPELINE_TRIGGER_TOKEN: $PIPELINE_TRIGGER_TOKEN

       # 特定镜像配置
       NODE_IMAGE: $REGISTRY_ENDPOINT/library/node:18
       MAVEN_IMAGE: $REGISTRY_ENDPOINT/library/maven:3.9.9-jdk21
       DOCKER_IMAGE: $REGISTRY_ENDPOINT/library/docker:26.1.4
   ```

### 2. **CI 配置**

   在 ci.yaml 文件中，配置项目的构建阶段，具体包括以下流程：

   - 项目打包：
      - 针对后端项目，使用 Maven 完成构建与打包。
      - 针对前端项目，使用 pnpm 进行依赖安装并生成构建产物。
   - 镜像构建与推送：
      - 构建 Docker 镜像并推送到指定的 Docker 镜像仓库。根据不同的 Git 分支 (main, master, develop, gray) 设置相应的环境配置。
   - 缓存优化：
      - 使用 Maven 和 Docker 构建中的缓存来提升构建效率，减少不必要的重复操作。

### 3. **CD 配置**

   在 deploy 阶段，您可以选择以下两种部署方式：

   - Docker 部署：
      - 使用 docker pull 命令拉取最新构建的 Docker 镜像。
      - 使用 docker run 命令启动镜像，并在 Docker 容器中运行应用，设置自动重启和端口映射。
   - K8S 部署：
      - 通过 Kubernetes API 更新 Deployment 中的镜像标签，将新构建的镜像应用到 Kubernetes 集群。
      - 通过 Kubernetes API 重启指定的 Deployment，使新的镜像生效。
   - 第三方部署：
      - 通过 GitLab CI 使用 curl 命令调用 GitLab API，触发其他项目的流水线或者执行跨项目部署。

### 4. **项目配置**

#### 4.1 **前端**

   在实际项目中编写 `nginx.conf`、`Dockerfile`、`.gitlab-ci.yaml` 文件，示例如下：

   ```Dockerfile
   # Dockerfile
   FROM nginx:latest
   COPY dist /usr/share/nginx/html
   COPY nginx.conf /etc/nginx/conf.d/default.conf
   RUN chown -R nginx:nginx /usr/share/nginx/html && chmod -R 755 /usr/share/nginx/html
   ```

   ```yaml
   # .gitlab-ci.yaml
   include:
     - project: "Pocket/sunflower"
       ref: main
       file:
         - "/project/eagle/ci.yaml"
         - "/project/eagle/cd.yaml"

   stages:
     - code-build
     - image-build
     - cd

   code-build:
     stage: code-build
     extends: [ .code-build ]
     allow_failure: false

   image-build:
     stage: image-build
     extends: [ .image-build ]
     allow_failure: false

   cd:
     stage: cd
     extends: [ .docker-deploy ]
   ```

#### 4.2 **后端**

   在实际项目中编写 `Dockerfile`、`.gitlab-ci.yaml` 文件，示例如下：

   ```Dockerfile
   # Dockerfile
   FROM openjdk:21-oracle
   WORKDIR /app
   COPY target/*.jar app.jar
   ENTRYPOINT ["java", "-jar", "app.jar"]
   ```

   ```yaml
   # .gitlab-ci.yaml
   include:
     - project: "Pocket/sunflower"
       ref: main
       file:
         - "/project/dolphin/ci.yaml"
         - "/project/dolphin/cd.yaml"

   stages:
     - code-build
     - image-build
     - cd

   code-build:
     stage: code-build
     extends: [ .code-build ]
     allow_failure: false

   image-build:
     stage: image-build
     extends: [ .image-build ]
     allow_failure: false

   cd:
     stage: cd
     extends: [ .docker-deploy ]
   ```

## 贡献

欢迎任何形式的贡献！请提交问题或拉取请求，您的参与将使项目更加完善。

## 许可证

本项目采用 MIT 许可证，详细信息请参见 LICENSE 文件。

