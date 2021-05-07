# docker

* 创建项目
* 创建基础镜像
  * 安装python环境
  * 或者将依赖全部打入基础镜像
* 创建dockerfile
  * 声明基础镜像
  * 拷贝目录到镜像
  * 启动命令
* 创建docker-compose，启动各组件 
* 启动测试，能否正常访问

docker-compose --compatibility up -d

# Docker-compose



# install gitlib

[安装文档](https://docs.gitlab.com/runner/install/osx.html)

#### 使用docker安装

1. 设置挂载卷位置：添加环境变量 GITLAB_HOME=$HOME/gitlab
2. 创建docker-compose.yml

```yml
version: '3'
services:
  web:
    image: 'gitlab/gitlab-ee:latest'
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://localhost:8929'
        gitlab_rails['gitlab_shell_ssh_port'] = 2224
    ports:
      - '8929:8929'
      - '2224:22'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
```

3. Docker-compose up -d

# install gitlib-runner

#### mac安装

1. 下载文件

> sudo curl --output /usr/local/bin/gitlab-runner "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-amd64"

2. 授予其执行权限

> sudo chmod +x /usr/local/bin/gitlab-runner

3. 安装

> cd ~ 
>
> gitlab-runner install 
>
> gitlab-runner start

4. 停止

> gitlab-runner stop

# CI

#### 登录gitlab

* 输入管理员密码注册，管理员账号为root

* 创建项目，testCI
* 点击设置-CICD，点击runner

#### 注册runner

1. 运行以下命令：

   ```
   gitlab-runner register
   ```

2. 输入您的GitLab实例URL（也称为`gitlab-ci coordinator URL`）。

3. 输入token

4. 输入跑步者的描述，随便输入

5. 输入与[Runner关联](https://docs.gitlab.com/ee/ci/runners/#using-tags)的[标签](https://docs.gitlab.com/ee/ci/runners/#using-tags)，以逗号分隔，随便输入

6. 提供[跑步执行者](https://docs.gitlab.com/runner/executors/README.html)。输入shell

7. 此时查看runner已被激活，变为绿色

#### 激活

gitlab-runner verify/run/start。测试哪条命令OK

# .gitlab.yml

关键字

* stages：定义pipe中流程节点
* stage：表示当前job在哪些流程中运行
* script：当前流程运行的脚本
* tags
* image/services这两个关键字可使用Docker的镜像和服务运行Job
* only/except这两个关键字后面跟的值是tag或者分支名的列表。
* When表示当前Job在何种状态下运行，它可设置为3个值
  * on_success: 仅当先前pipeline中的所有Job都成功（或因为已标记，被视为成功allow_failure）时才执行当前Job 。这是默认值。
  * on_failure: 仅当至少一个先前阶段的Job失败时才执行当前Job。
  * always: 执行当前Job，而不管先前pipeline的Job状态如何。



