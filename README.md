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

1. **变量配置**

   在 variables.yaml 文件中配置项目部署所需的各类参数，具体包括 Maven 和 Docker 注册表的相关信息、项目名称、分支等动态变量。以下是 variables.yaml 的配置示例：

   ```yaml
   .variables:
      variables:
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
         IMAGE_NAME: dolphin
         IMAGE_VERSION: $CI_PIPELINE_IID
         IMAGE_ADDRESS: $REGISTRY_ENDPOINT/$REGISTRY_NAMESPACE/$IMAGE_NAME:$IMAGE_VERSION
   
         # GitLab 配置
         GITLAB_URL: http://gitlab.sky.org
         TAG: gemini
         PROJECT_ID: 1
         PIPELINE_TRIGGER_URL: $GITLAB_URL/api/v4/projects/$PROJECT_ID/trigger/pipeline
         PIPELINE_TRIGGER_TOKEN: $PIPELINE_TRIGGER_TOKEN
   
         # 特定镜像配置
         NODE_IMAGE: $REGISTRY_ENDPOINT/library/node:18
         MAVEN_IMAGE: $REGISTRY_ENDPOINT/library/maven:3.9.9-jdk21
         DOCKER_IMAGE: $REGISTRY_ENDPOINT/library/docker:26.1.4
   ```

2. **CI 配置**

   在 ci.yaml 文件中，配置项目的构建阶段，具体包括以下流程：

   - 前后端打包：针对不同的项目和需求，配置前后端代码的构建过程。
   - 镜像构建：构建 Docker 镜像，推送到指定的 Docker 注册表。
   - 缓存优化：配置 Maven 和 Docker 构建过程中的缓存，提升构建效率。

3. **CD 配置**

   在 deploy 阶段，您可以通过配置 GitLab CI 使用 curl 命令调用 GitLab API 来触发其他的流水线，进行跨项目部署或者触发其他步骤。

4. **触发流水线**

   在实际项目中编写 `.gitlab-ci.yaml` 文件，内容如下：

   ```yaml
   include:
     - project: "Pocket/sunflower"
       ref: main
       file:
         - "/project/dolphin/ci.yaml"
         - "/project/dolphin/cd.yaml"
   
   stages:
     - ci-code-build
     - ci-image-build
     - cd
   
   ci-code-build:
     stage: ci-code-build
     extends: [ .code-build ]
     allow_failure: false
   
   ci-image-build:
     stage: ci-image-build
     extends: [ .image-build ]
     allow_failure: false
   
   cd:
     stage: cd
     extends: [ .deploy ]
   ```

   通过 GitLab 界面或 API 触发 CI/CD 流水线。

## 贡献

欢迎任何形式的贡献！请提交问题或拉取请求，您的参与将使项目更加完善。

## 许可证

本项目采用 MIT 许可证，详细信息请参见 LICENSE 文件。

