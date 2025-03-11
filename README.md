# 用Docker部署Web应用

## 一、为什么要用Docker？

想象你要搬家：传统方式是把家具拆散再组装，费时费力。Docker就像一套标准化的集装箱系统，把你的应用和所需环境打包成标准货箱，无论搬到哪个"港口"（服务器）都能即插即用。

### 核心概念三分钟速通
- **镜像（Image）**：如同安装光盘，包含完整的应用运行环境
- **容器（Container）**：镜像运行时的实体，就像运行中的电脑程序
- **仓库（Registry）**：存放镜像的"应用商店"，Docker Hub是最著名的公共仓库

与传统虚拟机的对比：
| 特性         | Docker容器       | 传统虚拟机  |
|--------------|-----------------|------------|
| 启动速度     | 秒级            | 分钟级      |
| 资源占用     | 低（MB级）      | 高（GB级）  |
| 性能损耗     | 接近原生        | 较高        |

## 二、编写你的第一个Dockerfile

以Node.js应用为例，我们创建`Dockerfile`（无文件后缀）：

```dockerfile
# 基础镜像选择
FROM node:18-alpine

# 设置工作目录
WORKDIR /app

# 先拷贝依赖文件（利用Docker层缓存优化）
COPY package*.json ./

# 安装依赖
RUN npm install

# 拷贝全部源代码
COPY . .

# 暴露容器端口
EXPOSE 3000

# 启动命令
CMD ["npm", "start"]
```

**关键技巧**：
1. 使用`.dockerignore`文件忽略node_modules等无关文件
2. 分层构建：频繁变动的步骤放在Dockerfile后面
3. 选择轻量级基础镜像（如alpine版本）

## 三、构建与运行全流程

### 3.1 构建镜像
```bash
docker build -t my-web-app:v1 .
```
- `-t`：给镜像起名+标签
- 末尾的`.`表示Dockerfile所在路径

### 3.2 运行容器
```bash
docker run -d -p 8080:3000 --name web-server my-web-app:v1
```
- `-d`：后台运行
- `-p`：端口映射（主机端口:容器端口）
- `--name`：给容器命名

### 3.3 常用管理命令
```bash
# 查看运行中的容器
docker ps

# 查看日志
docker logs web-server

# 进入容器终端
docker exec -it web-server sh

# 停止/删除容器
docker stop web-server
docker rm web-server
```

## 四、部署到云平台（以阿里云为例）

### 4.1 镜像推送
1. 登录容器镜像服务控制台
2. 创建命名空间和镜像仓库
3. 本地登录并推送镜像：
```bash
docker login registry.cn-hangzhou.aliyuncs.com
docker tag my-web-app:v1 registry.cn-hangzhou.aliyuncs.com/your-namespace/your-repo:v1
docker push registry.cn-hangzhou.aliyuncs.com/your-namespace/your-repo:v1
```

### 4.2 云端部署
1. 进入容器服务控制台
2. 创建Kubernetes集群（或使用Serverless版）
3. 通过镜像创建部署（Deployment）
4. 配置服务（Service）和入口（Ingress）
5. 设置自动扩缩容策略（可选）

![部署架构图](https://example.com/deploy-arch.png)

## 五、避坑指南

1. **时区问题**：基础镜像默认UTC时间，可通过`ENV TZ=Asia/Shanghai`设置
2. **文件权限**：Linux容器默认root用户，添加`USER node`切换非特权用户
3. **存储持久化**：重要数据使用Volume挂载
4. **健康检查**：添加HEALTHCHECK指令保证服务可用性

## 六、下一步学习方向

- Docker Compose多容器编排
- Kubernetes容器编排实战
- CI/CD自动化部署流水线
- 服务网格（Service Mesh）应用

> 实践建议：从简单的静态网站开始，逐步尝试数据库容器化，最终完成全栈应用的容器化部署。

**附录：常用资源**
- [Docker官方文档](https://docs.docker.com/)
- [Dockerfile最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Play with Docker](https://labs.play-with-docker.com/) 在线练习平台

---

通过这篇指南，你已经掌握了使用Docker部署Web应用的核心技能。现在就去把你的应用装进"集装箱"，开启云原生之旅吧！遇到问题记得：善用`docker logs`查日志，多用`docker exec`进容器调试，祝你好运！
