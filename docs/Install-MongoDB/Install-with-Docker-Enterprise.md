# 使用Docker安装MongoDB企业版

重要

将容器与MongoDB结合使用的推荐解决方案是：

- 为了进行开发和测试，请使用 [MongoDB社区Docker容器](https://hub.docker.com/_/mongo/)。
- 对于MongoDB企业版生产安装，请通过[MongoDB Ops Manager](https://docs.opsmanager.mongodb.com/current/tutorial/install-k8s-operator)使用Kubernetes 。



注意

此过程使用Docker的官方[mongo image](https://github.com/docker-library/mongo)，该[镜像](https://github.com/docker-library/mongo)由Docker社区*而非* MongoDB支持。

如果以上推荐的解决方案无法满足您的需求，请按照本教程中的步骤手动将Docker 安装到 [MongoDB企业版](https://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server)。





## 注意事项

[Docker](https://docs.docker.com/)的完整描述超出了本文档的范围。本页面假定您具有Docker的先验知识。

本文档仅描述了如何在Docker上安装MongoDB企业版，并且不会替换Docker上的其他资源。我们鼓励您在将Docker安装到MongoDB 企业版之前，彻底熟悉Docker及其相关主题。

重要

此过程使用Docker的官方[mongo image](https://github.com/docker-library/mongo)，该[镜像](https://github.com/docker-library/mongo)由Docker社区*而非* MongoDB支持。它仅支持在其[存储库](https://github.com/docker-library/mongo)中列出的主要版本，只有每个主要版本有特定的次版本。次要版本可以在每个主要版本的文件夹中的`Dockerfile`中找到。





## 使用企业版MongoDB创建Docker镜像

### 1. 下载用于企业版MongoDB的Docker构建文件。

安装 [Docker](https://docs.docker.com/install/)并设置 [Docker Hub](https://hub.docker.com/)帐户后， 使用以下命令从[Docker Hub mongo项目](https://github.com/docker-library/mongo)下载构建文件 。设置`MONGODB_VERSION`为您选择的主要版本。

DOCKER HUB MONGO项目

MongoDB *不*维护Docker Hub mongo项目。任何支持请求都应发送给[Docker](https://github.com/docker-library/mongo)。

复制

```
export MONGODB_VERSION=4.0
curl -O --remote-name-all https://raw.githubusercontent.com/docker-library/mongo/master/$MONGODB_VERSION/{Dockerfile,docker-entrypoint.sh}
```



### 2. 构建Docker容器。

使用下载的构建文件来创建围绕企业版MongoDB的Docker容器镜像。将您的Docker Hub用户名设置为`DOCKER_USERNAME`。

复制

```
export DOCKER_USERNAME=username
chmod 755 ./docker-entrypoint.sh
docker build --build-arg MONGO_PACKAGE=mongodb-enterprise --build-arg MONGO_REPO=repo.mongodb.com -t $DOCKER_USERNAME/mongo-enterprise:$MONGODB_VERSION .
```



### 3. 测试您的镜像。

在Docker容器中本地运行mongod并检查版本，使用以下命令：

复制

```
docker run --name mymongo -itd $DOCKER_USERNAME/mongo-enterprise:$MONGODB_VERSION
docker exec -it mymongo /usr/bin/mongo --eval "db.version()"
```

这应该输出MongoDB的shell和服务器版本。





## 将镜像推送到Docker Hub

（可选）您可以将Docker镜像推送到远程存储库（例如Docker Hub），以在其他主机上使用该镜像。如果将镜像推送到Docker Hub，则可以在要通过Docker安装企业版MongoDB的每台主机上运行`docker pull`。有关使用`docker pull`的完整指导，请在[此处](https://docs.docker.com/engine/reference/commandline/pull/#examples)参考其文档 。



### 1. 检查您的本地镜像。

以下命令显示您的本地Docker镜像：

复制

```
docker images
```

您应该在命令输出中看到您的企业版MongoDB镜像。如果不这样做，请尝试[使用企业版MongoDB创建Docker镜像](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-with-docker/#create-docker-image-enterprise)。



### 2. 推送至Docker Hub。

将您的本地企业版MongoDB镜像推送到您的远程Docker Hub帐户。

复制

```
docker login
docker push $DOCKER_USERNAME/mongo-enterprise:$MONGODB_VERSION
```

如果您登录[Docker Hub](https://hub.docker.com/)站点，则应该看到存储库下面列出的镜像。



原文链接：https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-with-docker/

译者：小芒果

