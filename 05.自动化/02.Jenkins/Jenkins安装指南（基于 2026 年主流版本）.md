# Jenkins安装指南（基于 2026 年主流版本）

Jenkins 是目前全球最流行的开源自动化服务器，主要用于实现 **CI/CD（持续集成/持续部署）**。它基于 Java 开发，拥有庞大的插件生态系统，可以连接几乎所有的开发工具（如 Git, Docker, Python, Maven, Kubernetes 等）。

以下是 Jenkins 从**安装、基础配置、插件管理**到**创建 Pipeline 流水线任务**的全流程详细指南（基于 2026 年主流版本）。

------

### 一、Jenkins 的安装与部署

Jenkins 支持多种安装方式，**Docker 部署**是目前最推荐的方式，因为环境隔离好、迁移方便且避免污染宿主机。当然，也可以直接安装在 Linux 或 Windows 上。

**1. 拉取镜像**

```bash
# 拉取最新长期支持版 (LTS)
docker pull jenkins/jenkins:lts
```

**2. 创建数据卷并启动容器**
需要映射端口（默认 8080）和数据目录（`/var/jenkins_home`），以便重启后数据不丢失。

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \  # 允许 Jenkins 内部调用 Docker (可选，用于构建镜像)
  -u root \  # 以 root 运行以便安装额外工具 (生产环境建议配置 sudo)
  jenkins/jenkins:lts
```

**3. 获取初始管理员密码**
容器启动后，查看日志获取初始密码：

```bash
docker logs jenkins
# 或者
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

------

### 二、初始化配置与插件管理

首次访问 `http://localhost:8080` (或服务器 IP) 会进入初始化向导。

#### 1. 解锁 Jenkins

输入之前获取的 **初始管理员密码**。

#### 2. 选择插件安装

- 推荐选择：

  "Install suggested plugins" (安装推荐插件)。

  - 这会安装 Git, Pipeline, Docker, SSH, Credentials 等核心插件。
  - *注意*：如果网络较慢（国内环境），可能会超时。如果失败，可以选择 "Skip" (跳过)，稍后在插件管理中手动安装。

#### 3. 创建第一个管理员用户

设置用户名、密码、邮箱等信息。建议使用 `admin` 或其他易记名称。

#### 4. 系统配置优化

进入 **Manage Jenkins (系统管理)** -> **System (系统配置)** 或 **Global Tool Configuration (全局工具配置)**：

- 配置 JDK/Maven/NodeJS/Python：
  - 在 **Global Tool Configuration** 中，找到对应工具（如 Python），勾选 "Install automatically" (自动安装) 并指定版本，或者填写本地已安装的路径（如 `/usr/bin/python3`）。
- 配置时区：
  - 在 **System** -> **Global properties**，勾选 "Timezone"，选择 `Asia/Shanghai`，确保日志时间正确。
- 配置 Git：
  - 确保 Git 路径正确（通常自动识别）。

#### 5. 安装常用插件 (Manage Jenkins -> Plugins)

搜索并安装以下关键插件（如果初始化未安装）：

- **Pipeline** (核心，必装)
- **Pipeline: Stage View** (可视化视图)
- **Git Plugin** / **GitHub Integration**
- **Docker Plugin** / **Docker Pipeline**
- **SSH Agent** (用于远程部署)
- **Allure Report** (如果你用 Allure 做测试报告)
- **HTML Publisher** (用于发布 pytest-html 报告)
- **Chinese (Simplified) Language** (如果界面是英文，可安装此插件汉化)

------

### 三、核心概念：凭证 (Credentials) 与 节点 (Nodes)

#### 1. 配置凭证 (Credentials)

Jenkins 需要凭证来拉取私有代码库或登录远程服务器。

- 路径：**Manage Jenkins** -> **Credentials** -> **(global)** -> **Add Credentials**。
- Kind (类型):
  - **Username with password**: 用于 Git 账号密码、Docker  registry 登录。
  - **SSH Username with private key**: 用于免密登录远程服务器部署。
  - **Secret text**: 用于 API Token (如 GitLab Token, AWS Key)。
- **ID**: 设置一个唯一的 ID（如 `git-cred`, `deploy-ssh`），在 Pipeline 脚本中会通过这个 ID 引用。

#### 2. 配置节点 (Nodes/Agents)

如果构建任务很重，可以在其他机器上安装 **Jenkins Agent**，让主节点只负责调度，子节点负责执行。

- 对于简单的 Python+Selenium 自动化，通常直接在 **Master** 节点（即 `agent any`）运行即可，前提是 Master 上安装了 Chrome 和 Python。

------

### 四、创建 Pipeline 流水线任务 (实战)

这是 Jenkins 最强大的功能。我们将创建一个任务来运行之前的 Python+Selenium 脚本。

#### 步骤 1: 新建任务

1. 点击首页 **"New Item" (新建任务)**。
2. 输入任务名称（如 `AutoTest-Pipeline`）。
3. 选择 **"Pipeline" (流水线)**，点击 **OK**。

#### 步骤 2: 配置流水线

在配置页面中：

1. **General (通用)**:

   - 勾选 **"Discard old builds" (丢弃旧的构建)**，设置保留策略（如保留最近 10 次），防止磁盘爆满。
   - 勾选 **"This project is parameterized" (参数化构建过程)** (可选)，可以添加字符串参数如 `BROWSER=chrome`。

2. **Pipeline (流水线)**:

   - Definition (定义): 选择 "Pipeline script from SCM" (Pipeline 脚本来自 SCM)。
     - 这意味着你的 `Jenkinsfile` 存放在 Git 仓库中，修改代码即修改流水线，符合 "Infrastructure as Code" 理念。
   - **SCM**: 选择 **Git**。
   - **Repository URL**: 填入你的代码仓库地址 (如 `https://github.com/yourname/mytest.git`)。
   - **Credentials**: 选择之前配置的 Git 凭证。
   - **Branch Specifier**: `*/main` 或 `*/master`。
   - **Script Path**: 默认为 `Jenkinsfile` (确保你根目录有这个文件)。

   *(如果是学习测试，也可以选择 "Pipeline script"，直接在文本框里写 Groovy 代码)*

3. 点击 **Save (保存)**。

#### 步骤 3: 运行任务

- 点击左侧菜单的 **"Build Now" (立即构建)**。
- 点击 **#1** 构建编号，选择 **"Console Output" (控制台输出)** 查看实时日志。
- 如果配置了 `archiveArtifacts`，构建成功后会在页面看到下载报告链接。

------

### 五、Jenkinsfile 语法详解 (Declarative Pipeline)

Jenkins Pipeline 使用 Groovy 语言，分为 **声明式 (Declarative)** 和 **脚本式 (Scripted)**。**强烈推荐使用声明式**，结构清晰，易于维护。

一个标准的 `Jenkinsfile` 结构如下：

```groovy
pipeline {
    // 1. Agent: 指定在哪运行
    agent any 
    // 或者指定标签: agent { label 'python-agent' }
    // 或者使用 Docker: 
    // agent {
    //     docker { image 'python:3.9-slim' }
    // }

    // 2. Tools: 引用全局配置的工具 (可选)
    tools {
        // jdk 'JDK-17' 
        // maven 'Maven-3.9'
    }

    // 3. Environment: 定义环境变量
    environment {
        DISPLAY = ':99' // 如果需要虚拟显示
        MY_VAR = 'Hello'
        // 从凭证加载秘密信息
        GIT_CRED = credentials('git-cred-id') 
    }

    // 4. Stages: 流水线的阶段
    stages {
        stage('Checkout') {
            steps {
                echo '正在拉取代码...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    source venv/bin/activate
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                // 执行测试
                sh '''
                    source venv/bin/activate
                    pytest tests/ --html=report.html --self-contained-html
                '''
            }
            post {
                // 测试结束后的操作
                always {
                    // 归档报告
                    archiveArtifacts artifacts: 'report.html', fingerprint: true
                }
                failure {
                    echo '测试失败了！'
                    // 可以发送邮件通知
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main' // 仅在 main 分支执行部署
            }
            steps {
                echo '部署到生产环境...'
                // sh './deploy.sh'
            }
        }
    }

    // 5. Post: 整个流水线结束后的操作
    post {
        always {
            cleanWs() // 清理工作空间
        }
        failure {
            mail to: 'team@example.com',
                 subject: "Failed: ${currentBuild.fullDisplayName}",
                 body: "Check console output at ${BUILD_URL}"
        }
    }
}
```

#### 关键语法点：

- **`agent`**: 决定构建运行的环境。`any` 表示任意可用节点；`docker` 表示临时启动一个容器运行，用完即毁（非常适合 Selenium，避免浏览器版本冲突）。
- **`stage`**: 逻辑阶段（如：拉取、安装、测试、部署）。
- **`steps`**: 阶段内具体执行的步骤。
  - `sh 'command'`: 在 Linux/Mac 执行 Shell 命令。
  - `bat 'command'`: 在 Windows 执行批处理命令。
  - `checkout scm`: 拉取代码。
- **`post`**: 后置动作，支持 `always`, `success`, `failure`, `unstable` 等条件。
- **`when`**: 条件执行（如只在特定分支运行）。

------

### 六、进阶技巧与最佳实践

1. **Webhook 自动触发**：

   - 不要依赖轮询（Poll SCM）。
   - 在 GitLab/GitHub 仓库设置 Webhook，指向 `http://jenkins-url/github-webhook/`。
   - 在 Jenkins 任务配置中勾选 **"GitHub hook trigger for GITScm polling"**。
   - 效果：代码 `git push` 后，Jenkins 秒级自动触发构建。

2. **并行执行 (Parallel)**：
   如果测试用例很多，可以在 `stage` 中使用 `parallel` 块并行运行，大幅缩短时间。

   ```groovy
   stage('Parallel Tests') {
       parallel {
           stage('Chrome') {
               steps { sh 'pytest -k chrome' }
           }
           stage('Firefox') {
               steps { sh 'pytest -k firefox' }
           }
       }
   }
   ```

3. **可视化报告**：

   - 安装 **HTML Publisher Plugin**。
   - 在 `post` -> `always` 中添加 `publishHTML` 步骤，可以直接在 Jenkins 界面内嵌展示 HTML 测试报告，无需下载。

4. **安全性**：

   - 定期更新 Jenkins 和插件。
   - 配置 **Matrix Authorization Strategy**，限制普通用户只能看不能改配置。
   - 不要在 `Jenkinsfile` 中硬编码密码，务必使用 **Credentials** 绑定。

通过以上步骤，你已经掌握了一个企业级 Jenkins 环境的搭建和使用核心。对于 Python+Selenium 项目，最关键的是**在 Jenkins 节点上正确安装 Chrome/Driver**，并在 `Jenkinsfile` 中配置好**无头模式 (Headless)** 和**虚拟环境**。
