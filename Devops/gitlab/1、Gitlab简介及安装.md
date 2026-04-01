#### **一、GitLab 简介** 📘

GitLab 是一个 **开源的 DevOps 全生命周期管理平台**，覆盖从代码开发到部署运维的全流程。支持 **自托管部署**，提供社区版（CE）和企业版（EE），适用于个人开发者、中小团队及大型企业。

---

##### **核心功能模块** 🛠️

|           |                                            |
| --------- | ------------------------------------------ |
| **模块**    | **功能**                                     |
| **代码托管**  | Git 仓库管理、分支保护、代码审查（MR）、Wiki 文档             |
| **CI/CD** | 自动化构建、测试、部署（`.gitlab-ci.yml`<br><br>配置流水线） |
| **项目管理**  | Issue 跟踪、看板（Kanban）、里程碑（Milestone）、甘特图     |
| **安全与合规** | 依赖扫描、SAST/DAST 安全测试、License 合规检查           |
| **监控与运维** | 集成 Prometheus、日志管理、性能监控（APM）               |
| **协作工具**  | 代码片段共享、内联评论、Slack/Mattermost 集成            |
|           |                                            |

---

##### **关键概念补充** 📚

1. **DevOps 全生命周期**：

- **Plan**（需求规划）→ **Code**（编码）→ **Build**（构建）→ **Test**（测试）→ **Deploy**（部署）→ **Monitor**（监控）→ **Secure**（安全）。

2. **GitLab Runner**：

- 执行 CI/CD 任务的代理程序，支持 Shell、Docker、Kubernetes 等多种执行器。

3. **Merge Request（MR）**：

- 类似 GitHub 的 Pull Request，用于代码审查和分支合并。

---

##### **优点** ✅

1. **一体化平台**：无需集成多个工具，降低运维复杂度。
2. **开源免费**：社区版功能齐全，适合中小团队私有化部署（MIT 协议）。
3. **强大的 CI/CD**：灵活配置流水线，支持多环境部署（Dev/Stage/Prod）。
4. **安全合规**：内置安全扫描工具，满足企业级合规需求。
5. **活跃社区**：丰富的插件生态和文档支持。

---

##### **缺点** ❌

1. **资源占用高**：自托管需较高服务器配置（建议 4核8GB 内存起步）。
2. **学习曲线陡峭**：CI/CD 配置和高级功能需时间熟悉。
3. **企业版成本高**：高级功能（如史诗级项目管理）需付费订阅。
4. **性能瓶颈**：大规模团队使用时，可能需优化数据库和缓存。

---

##### **适用场景** 🎯

- **中小团队**：免费社区版满足代码托管 + 基础 DevOps 需求。
- **企业级私有化**：自建 GitLab 实例，保障代码安全和合规性。
- **云原生开发**：与 Kubernetes、Docker 深度集成，简化容器化部署。

---

##### **快速对比 GitLab vs GitHub** ⚖️

|   |   |   |
|---|---|---|
|**特性**|**GitLab**|**GitHub**|
|**CI/CD**|原生集成（GitLab CI/CD）|依赖 GitHub Actions|
|**部署模式**|支持自托管 + SaaS|仅 SaaS（GitHub Enterprise 可自托管）|
|**开源协议**|社区版完全开源|代码不开放|
|**安全扫描**|内置（社区版基础功能）|依赖第三方插件（如 Dependabot）|

---

##### **总结** 🚀

GitLab 是 **一站式 DevOps 解决方案**，适合追求 **代码安全** 和 **自动化流程** 的团队。

- **推荐场景**：私有化部署、全链路 DevOps、企业级协作。
- **慎用场景**：轻量级代码托管、资源有限的小型项目。

用一句话概括：**“从代码到生产环境，GitLab 承包你的每一步！”** 💻🔧🌐

---

#### 二、安装最新版Gitlab

**使用Docker-compose一键安装最新版gitlab**

##### 🔥 **生产环境硬性要求**

|   |   |   |
|---|---|---|
|**组件**|**最低配置**|**推荐配置**|
|CPU|2核|4核|
|内存|4GB|8GB|
|存储|50GB|200GB+|
|操作系统|CentOS 7.6+|Ubuntu 22.04 LTS|

|   |   |
|---|---|
|**软件**|**版本**|
|docker-ce|docker-ce-26.1.4-1.el7.x86_64|
|docker-compose|Docker Compose version v2.34.0|
|gitlab|gitlab/gitlab-ce:17.10.1-ce.0|

##### 📂 安装步骤

**1.调整limits.conf**

```
vim /etc/security/limits.conf
* soft nproc 65536
* hard nproc 65536
* soft nofile 65536
* hard nofile 65536
```

**2.创建专用目录**

mkdir -p /data/gitlab/{config,logs,data}

**3.编写docker-compose.yaml**

```
cd /data/gitlab/
vim docker-compose.yaml
```

```
version: '3.8'

services:
  gitlab:
    image: gitlab/gitlab-ce:17.10.1-ce.0
    container_name: gitlab
    restart: unless-stopped  # 异常停止后自动恢复
    hostname: 'gitlab.yourdomain.com'  #   修改为你的域名/IP
    cpus: 2
    mem_limit: 4g
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.yourdomain.com'  #   访问地址
        gitlab_rails['time_zone'] = 'Asia/Shanghai' #   时区设置
        gitlab_rails['gitlab_shell_ssh_port'] = 2222 #   SSH端口映射
        prometheus['listen_address'] = '0.0.0.0:9090' #   开启监控 
        alertmanager['enable'] = false  #   内网关闭告警
        nginx['worker_processes'] = 4
        puma['worker_processes'] = 4
        sidekiq['concurrency'] = 10
        postgresql['shared_buffers'] = "256MB"
        redis['max_memory'] = "1GB"
        ## 邮件配置（按需启用）
        # gitlab_rails['smtp_enable'] = true
        # gitlab_rails['smtp_address'] = "smtp.example.com"
        # gitlab_rails['smtp_port'] = 587
        # gitlab_rails['smtp_user_name'] = "user@example.com"
        # gitlab_rails['smtp_password'] = "password"
        # gitlab_rails['smtp_domain'] = "example.com"
        # gitlab_rails['smtp_authentication'] = "login"
        # gitlab_rails['smtp_enable_starttls_auto'] = true
        ## https按需启用
        # external_url 'https://gitlab.yourdomain.com'
        # nginx['redirect_http_to_https'] = true
        # nginx['ssl_certificate'] = "/etc/gitlab/ssl/fullchain.pem"
        # nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/privkey.pem"
    ports:
      - "80:80"
      # - "443:443" 
      - "2222:22" 
      - "9090:9090" 
    volumes:
      - /data/gitlab/config:/etc/gitlab
      - /data/gitlab/logs:/var/log/gitlab
      - /data/gitlab/data:/var/opt/gitlab
      # - /data/gitlab/ssl:/etc/gitlab/ssl  # SSL 证书目录
    shm_size: '256m'  #   必须设置共享内存
    ulimits:
      nproc: 65535
      nofile:
        soft: 65535
        hard: 65535
    #   健康检查
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost/-/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5
    networks:
      - gitlab-net

networks:
  gitlab-net:
    driver: bridge
```

**4.启动服务**

```
# 检查yaml文件是否准确
docker compose config
# 测试启动
docker compose up --dry-run
# 实际部署
docker compose up -d && docker compose logs -f
```

**⏳** **初始化需要3-5分钟，可通过以下命令查看进度：**

docker logs -f gitlab | grep "gitlab Reconfigured"

![](https://cdn.nlark.com/yuque/0/2025/png/782222/1751955191727-542958d0-f034-48eb-acfc-c3aa0dca49fa.png)

**🔑** **获取初始root密码**

docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password

![](https://cdn.nlark.com/yuque/0/2025/png/782222/1751955191440-b67ab057-c280-4af5-b0c4-b80cd184cba8.png)

🌍 访问控制台浏览器打开 `http://your-server-ip` 即可登录（用户：root）

![](https://cdn.nlark.com/yuque/0/2025/png/782222/1751955191407-6660005f-15d0-4923-b34c-f5113cc81a45.png)

![[0509362c-5abc-4ba8-b790-2e7b027df4d4.png]]

##### 🔍 生产环境建议

1. **高可用基础架构** GitLab作为核心DevOps平台，需保障**服务连续性**。Docker部署需配合宿主机HA机制（如Keepalived）实现故障转移。
2. **资源隔离与限制** 通过CGroup限制CPU/内存使用，避免容器资源争抢导致宿主机雪崩。建议预留20%资源缓冲。
3. **数据持久化策略** 采用**卷挂载三件套**：

- `/etc/gitlab`：核心配置（含密钥）
- `/var/log/gitlab`：日志审计
- `/var/opt/gitlab`：应用数据

4. **安全纵深防御** 内网环境可不启用HTTPS，但需配合**IP白名单+防火墙规则**。SSH端口建议与宿主机隔离。

##### 🛡️ **生产运维指南**

**1. 监控三板斧**

```
# 实时资源
watch -n 2 "docker stats --no-stream prod-gitlab"

# 日志追踪
journalctl -u docker.service -f | grep gitlab

# 健康状态
docker inspect --format='{{json .State.Health}}' prod-gitlab
```

**2. 备份策略**

```
# 全量备份（每日2:00执行）
docker exec -t prod-gitlab gitlab-backup create CRON=1

# 配置文件备份（与数据分离存储）
rsync -avz /mnt/nas/gitlab/config/ backup-server:/gitlab-config/
```

**3. 灾备恢复**

```
# 停止服务
docker compose stop

# 恢复数据
docker exec -it prod-gitlab gitlab-backup restore BACKUP=xxxxxxx

# 重启验证
docker compose up -d
```

---

##### 🚦 **部署验证清单**

1. [ ] 宿主机swap分区≥4GB
2. [ ] 数据卷使用xfs文件系统
3. [ ] 防火墙放行80/2222端口
4. [ ] 配置NTP时间同步
5. [ ] 测试备份恢复流程

来自: [《从0到100：DevOps全栈实战手册》第二章：GitLab全栈教学——2.1GitLab简介与社区版安装](https://mp.weixin.qq.com/s/VXkDayXi00-vqdA9L3ObgQ)