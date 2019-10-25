# 安装和配置指南

Harbor可以通过以下两种方式安装:

- **在线安装程序:** 安装程序从Docker hub下载Harbor的图像。因此，安装程序非常小。

- **离线安装程序:** 使用此安装程序时，主机没有互联网连接。安装程序包含预构建的映像，因此其大小较大。

所有安装程序都可以从 **[官方发布](https://github.com/goharbor/harbor/releases)** 页面下载。

本指南描述了使用在线或离线安装程序安装和配置Harbor的步骤。安装过程几乎是一样的。

如果您运行的是Harbor以前的版本，您可能需要更新 ```harbor.yml``` 。并迁移数据以适应新的数据库schema。详情请参阅  **[Harbor 迁移指南](migration_guide.md)**.

此外，社区还创建了Kubernetes上的部署说明。详情请参考[Harbor on Kubernetes using Helm](https://github.com/goharbor/harbor-helm)。

## Harbor 组件

|组件|版本号|
|---|---|
|Postgresql|9.6.10-1.ph2|
|Redis|4.0.10-1.ph2|
|Clair|2.0.8|
|Beego|1.9.0|
|Chartmuseum|0.9.0|
|Docker/distribution|2.7.1|
|Docker/notary|0.6.1|
|Helm|2.9.1|
|Swagger-ui|3.22.1|

## 目标主机的先决条件

Harbor被部署为几个Docker容器，因此可以部署在任何支持Docker的Linux发行版上。目标主机需要安装Docker和Docker Compose。

### 硬件

|资源|容量|描述|
|---|---|---|
|CPU|minimal 2 CPU|4 CPU is preferred|
|Mem|minimal 4GB|8GB is preferred|
|Disk|minimal 40GB|160GB is preferred|

### 软件

|软件|版本号|描述|
|---|---|---|
|Docker engine|version 17.06.0-ce+ or higher|For installation instructions, please refer to: [docker engine doc](https://docs.docker.com/engine/installation/)|
|Docker Compose|version 1.18.0 or higher|For installation instructions, please refer to: [docker compose doc](https://docs.docker.com/compose/install/)|
|Openssl|latest is preferred|Generate certificate and keys for Harbor|

### 网络端口

|端口|协议|描述|
|---|---|---|
|443|HTTPS|Harbor portal and core API will accept requests on this port for https protocol, this port can change in config file|
|4443|HTTPS|Connections to the Docker Content Trust service for Harbor, only needed when Notary is enabled, This port can change in config file|
|80|HTTP|Harbor 门户和核心API将在此端口上接受HTTP协议的请求

## 安装步骤

安装步骤可归结为以下几个步骤

1. 下载安装程序;
2. 配置 **harbor.yml**;
3. 执行 **install.sh** 来安装和启动 Harbor;

#### 下载安装程序:

安装程序的二进制文件可以从 [release](https://github.com/goharbor/harbor/releases) 页面下载。选择在线或离线安装程序。使用 *tar* 命令提取包。

在线安装程序:

```bash
    $ tar xvf harbor-online-installer-<version>.tgz
```

离线安装程序:

```bash
    $ tar xvf harbor-offline-installer-<version>.tgz
```

#### 配置 Harbor

配置参数位于文件**harbor.yml**中。

参数有两类，**必选参数**和**可选参数**。

- **系统级参数**: 需要在配置文件中设置这些参数。在 ```harbor.yml``` 中更新它们，然后执行 ```install.sh``` 脚本重新安装 Harbor，才能生效。

- **用户级参数**: 这些参数可以在Web Portal第一次启动harbor后进行更新。特别是，您必须在注册或在Harbor中创建任何新用户之前设置
所需的**auth_mode**。当系统中有用户时(除了默认的admin用户之外)，不能更改**auth_mode**。

下面描述了参数——注意，至少需要更改**hostname**属性。

##### 必需的参数

- **hostname**: The target host's hostname, which is used to access the Portal and the registry service. It should be the IP address or the fully qualified domain name (FQDN) of your target machine, e.g., `192.168.1.10` or `reg.yourdomain.com`. _Do NOT use `localhost` or `127.0.0.1` or `0.0.0.0` for the hostname - the registry service needs to be accessible by external clients!_

- **data_volume**: The location to store harbor's data.

- **harbor_admin_password**: The administrator's initial password. This password only takes effect for the first time Harbor launches. After that, this setting is ignored and the administrator's password should be set in the Portal. _Note that the default username/password are **admin/Harbor12345** ._

- **database**: the configs related to local database
  - **password**: The root password for the PostgreSQL database. Change this password for any production use.
  - **max_idle_conns**: The maximum number of connections in the idle connection pool. If <=0 no idle connections are retained. The default value is 50 and if it is not configured the value is 2.
  - **max_open_conns**: The maximum number of open connections to the database. If <= 0 there is no limit on the number of open connections. The default value is 100 for the max connections to the Harbor database. If it is not configured the value is 0.

- **jobservice**: jobservice related service
  - **max_job_workers**: The maximum number of replication workers in job service. For each image replication job, a worker synchronizes all tags of a repository to the remote destination. Increasing this number allows more concurrent replication jobs in the system. However, since each worker consumes a certain amount of network/CPU/IO resources, please carefully pick the value of this attribute based on the hardware resource of the host.
- **log**: log related url
  - **level**: log level, options are debug, info, warning, error, fatal
  - **local**: The default is to retain logs locally.
      - **rotate_count**: Log files are rotated **rotate_count** times before being removed. If count is 0, old versions are removed rather than rotated.
      - **rotate_size**: Log files are rotated only if they grow bigger than **rotate_size** bytes. If size is followed by k, the size is assumed to be in kilobytes. If the M is used, the size is in megabytes, and if G is used, the size is in gigabytes. So size 100, size 100k, size 100M and size 100G are all valid.
      - **location**: the directory to store logs
  - **external_endpoint**: Enable this option to forward logs to a syslog server.
       - **protocol**: Transport protocol for the syslog server. Default is TCP.
       - **host**: The URL of the syslog server.
       - **port**: The port on which the syslog server listens.
     
##### 可选参数

- **http**:
  - **port** : the port number of you http

- **https**: The protocol used to access the Portal and the token/notification service.  If Notary is enabled, has to set to _https_.
refer to **[Configuring Harbor with HTTPS Access](configure_https.md)**.
  - **port**: port number for https
  - **certificate**: The path of SSL certificate, it's applied only when the protocol is set to https.
  - **private_key**: The path of SSL key, it's applied only when the protocol is set to https.

- **external_url**: Enable it if use external proxy, and when it enabled the hostname will no longer used

- **clair**: Clair related configs
  - **updaters_interval**: The interval of clair updaters, the unit is hour, set to 0 to disable the updaters
  - **http_proxy**: Config http proxy for Clair, e.g. `http://my.proxy.com:3128`.
  - **https_proxy**: Config https proxy for Clair, e.g. `http://my.proxy.com:3128`.
  - **no_proxy**: Config no proxy for Clair, e.g. `127.0.0.1,localhost,core,registry`.

- **chart**: chart related configs
  - **absolute_url**: if set to enabled chart will use absolute url, otherwise set it to disabled, chart will use relative url.

- **external_database**: external database configs, Currently only support POSTGRES.
  - **harbor**: harbor's core database configs
    - **host**: hostname for harbor core database
    - **port**: port of harbor's core database
    - **db_name**: database name of harbor core database
    - **username**: username to connect harbor core database
    - **password**: password to harbor core database
    - **ssl_mode**: is enable ssl mode
    - **max_idle_conns**: The maximum number of connections in the idle connection pool. If <=0 no idle connections are retained. The default value  is 2.
    - **max_open_conns**: The maximum number of open connections to the database. If <= 0 there is no limit on the number of open connections. The default value is 0.
  - **clair**: clair's database configs
    - **host**: hostname for clair database
    - **port**: port of clair database
    - **db_name**: database name of clair database
    - **username**: username to connect clair database
    - **password**: password to clair database
    - **ssl_mode**: is enable ssl mode
  - **notary_signer**: notary's signer database configs
    - **host**: hostname for notary signer database
    - **port**: port of notary signer database
    - **db_name**: database name of notary signer database
    - **username**: username to connect notary signer database
    - **password**: password to notary signer database
    - **ssl_mode**: is enable ssl mode
  - **notary_server**:
    - **host**: hostname for notary server database
    - **port**: port of notary server database
    - **db_name**: database name of notary server database
    - **username**: username to connect notary server database
    - **password**: password to notary server database
    - **ssl_mode**: is enable ssl mode

- **external_redis**: configs for use the external redis
  - **host**: host for external redis
  - **port**: port for external redis
  - **password**: password to connect external host
  - **registry_db_index**: db index for registry use
  - **jobservice_db_index**: db index for jobservice
  - **chartmuseum_db_index**: db index for chartmuseum

#### 配置存储后端(可选)

- **storage_service**: 默认情况下，Harbor将镜像和chart存储在本地文件系统中。
在生产环境中，您可以考虑使用其他存储后端而不是本地文件系统，如S3、OpenStack Swift、Ceph等。这些参数是注册表的配置。
  - **ca_bundle**:  The path to the custom root ca certificate, which will be injected into the trust store of registry's and chart repository's containers.  This is usually needed when the user hosts a internal storage with self signed certificate.
  - **provider_name**: Storage configs for registry, default is filesystem. for more info about this configuration please refer https://docs.docker.com/registry/configuration/
  - **redirect**:
    - **disable**: set disable to true when you want to disable registry redirect

例如，如果你使用Openstack Swift作为你的存储后端，参数可能是这样的:

``` yaml
storage_service:
  ca_bundle:
  swift:
    username: admin
    password: ADMIN_PASS
    authurl: http://keystone_addr:35357/v3/auth
    tenant: admin
    domain: default
    region: regionOne
    container: docker_images"
  redirect:
    disable: false
```

_注意: 有关注册表存储后端的详细信息，请参考[注册表配置参考](https://docs.docker.com/registry/configuration/) ._

#### 完成安装和启动 Harbor

一旦 **harbor.yml** 和存储后端 (可选) 被配置, 使用 `install.sh` 脚本安装并启动Harbor
请注意，在线安装程序从Docker hub下载Harbor镜像可能需要一些时间。

##### 默认安装 (没有 Notary/Clair)

Harbor 集成了 Notary 和 Clair (漏洞扫描)。但是，默认安装不包括Notary或Clair服务。

``` sh
    $ sudo ./install.sh
```

如果一切正常，您应该能够打开浏览器访问管理页面 `http://reg.yourdomain.com` (将`reg.yourdomain.com`更改为在`port.yml`中配置的主机名 `harbor.yml`)。注意，默认的管理员用户名/密码是 admin/Harbor12345。

登录到管理门户并创建一个新项目。例如：`myproject`。然后你可以使用docker命令来登录和推送镜像(默认情况下，注册服务器监听端口80):

```sh
$ docker login reg.yourdomain.com
$ docker push reg.yourdomain.com/myproject/myrepo:mytag
```

**重要:** Harbor的默认安装使用 _HTTP_ - 这样，您将需要添加选项 `--insecure-registry` 到您的客户端Docker守护进程中，并重新启动Docker服务。

##### 安装带有公证服务的Harbor
要安装带有公证服务的Harbor，请在运行`install.sh`时添加一个参数:

```sh
    $ sudo ./install.sh --with-notary
```

**注意**: 安装带有公证服务的Harbor， 参数 **ui_url_protocol** 必须被设置为 "https"。要配置HTTPS，请参考以下部分。

关于公证和Docker内容信任的更多信息，请参考 [Docker的文档](https://docs.docker.com/engine/security/trust/content_trust/).

##### Installation with Clair

要安装带Clair服务的Harbor，请在运行`install.sh`时添加一个参数:

```sh
    $ sudo ./install.sh --with-clair
```

关于Clair的更多信息，请参阅Clair的文档:
`https://coreos.com/clair/docs/2.0.1/`

##### Installation with chart repository service

To install Harbor with chart repository service, add a parameter when you run ```install.sh```:
要安装带有 chart 存储库服务的Harbor，请在运行 ```install.sh```时添加一个参数:

```sh
    $ sudo ./install.sh --with-chartmuseum
```

**注意**: 如果您想安装 Notary, Clair 和 chart repository 服务, 您必须在同一个命令中指定所有参数:

```sh
    $ sudo ./install.sh --with-notary --with-clair --with-chartmuseum
```

如何使用Harbor，请参考 **[User Guide of Harbor](user_guide.md)** .

#### 配置Harbor使用HTTPS访问

Harbor不附带任何证书，默认情况下使用HTTP来处理请求。虽然这使得设置和运行相对简单—特别是对于开发或测试环境—但是不推荐用于生产环境。
要启用HTTPS，请参考 **[使用HTTPS访问配置端口](configure_https.md)**。

### 管理Harbor的生命周期

您可以使用docker-compose来管理Harbor的生命周期。下面列出了一些有用的命令(必须运行在与*docker-compose.yml*相同的目录中)。


停止 Harbor:

``` sh
$ sudo docker-compose stop
Stopping nginx              ... done
Stopping harbor-portal      ... done
Stopping harbor-jobservice  ... done
Stopping harbor-core        ... done
Stopping registry           ... done
Stopping redis              ... done
Stopping registryctl        ... done
Stopping harbor-db          ... done
Stopping harbor-log         ... done
```

停止后启动 Harbor:

``` sh
$ sudo docker-compose start
Starting log         ... done
Starting registry    ... done
Starting registryctl ... done
Starting postgresql  ... done
Starting core        ... done
Starting portal      ... done
Starting redis       ... done
Starting jobservice  ... done
Starting proxy       ... done
```

要更改Harbor的配置，首先停止现有的Harbor实例并更新 `harbor.yml`。然后运行 `prepare` 脚本来填充配置。
最后重新创建并启动Harbor的实例:


``` sh
$ sudo docker-compose down -v
$ vim harbor.yml
$ sudo prepare
$ sudo docker-compose up -d
```

删除Harbor的容器，同时在文件系统中保留镜像数据和Harbor的数据库文件:

``` sh
$ sudo docker-compose down -v
```

删除Harbor的数据库和镜像数据(为了干净的重新安装):

``` sh
$ rm -r /data/database
$ rm -r /data/registry
```

#### *管理 Harbor 的生命周期，当它安装了Notary, Clair 和 chart repository 服务*

如果你想一起安装 Notary, Clair 和 chart repository 服务,你应该包括所有的组件在准备命令:

``` sh
$ sudo docker-compose down -v
$ vim harbor.yml
$ sudo prepare --with-notary --with-clair --with-chartmuseum
$ sudo docker-compose up -d
```

请查看[Docker撰写命令行参考](https://docs.docker.com/compose/reference/) 以获得关于docker-compose的更多信息。

### 持久数据和日志文件

默认情况下，注册表数据保存在主机的`/data/`目录中。即使在删除或者重新创建Harbor的容器时，这些数据仍然保持不变，您可以在`harbor.yml`中编辑 `data_volume`。文件来更改此目录。



另外，Harbor使用*rsyslog*来收集每个容器的日志。默认情况下，这些日志文件存储在目标主机的 `/var/log/harbor/`目录中，以便进行故障排除，
您还可以在 `harbor.yml`中更改日志目录。


## 配置端口监听自定义端口

默认情况下，对于管理门户和docker命令，Harbor监听端口80(HTTP)和443(如果配置为HTTPS)，这些默认端口可以在`harbor.yml`中配置。

## 使用外部数据库配置Harbor

目前，只有PostgreSQL数据库被Harbor支持。

要使用外部数据库，只需取消 `harbor.yml` 中的 `external_database` 部分的注释。并填写必要的信息。对于Harbor core、Clair、公证服务
和公证签名者，用户需要先创建四个数据库。当Harbor启动时，这些表会自动生成。

## 管理用户设置

发布1.8.0之后，用户设置与系统设置分离，所有用户设置都应该在web控制台或通过HTTP请求进行配置。

请参考[配置用户设置](configure_user_settings.md)来配置用户设置。

## 性能调优


默认情况下，Harbor将Clair container的CPU使用量限制在150000以内，避免了它耗尽所有的CPU资源。
这是在docker-compose.clair.yml中定义的。您可以根据您的硬件配置来修改它。

## 故障排除

1. 当Harbor不能正常工作时，运行下面的命令，看看是不是所有的Harbor的container都处于 **UP** 状态:
```
    $ sudo docker-compose ps
        Name                     Command               State                    Ports
  -----------------------------------------------------------------------------------------------------------------------------
  harbor-core         /harbor/start.sh                 Up
  harbor-db           /entrypoint.sh postgres          Up      5432/tcp
  harbor-jobservice   /harbor/start.sh                 Up
  harbor-log          /bin/sh -c /usr/local/bin/ ...   Up      127.0.0.1:1514->10514/tcp
  harbor-portal       nginx -g daemon off;             Up      80/tcp
  nginx               nginx -g daemon off;             Up      0.0.0.0:443->443/tcp, 0.0.0.0:4443->4443/tcp, 0.0.0.0:80->80/tcp
  redis               docker-entrypoint.sh redis ...   Up      6379/tcp
  registry            /entrypoint.sh /etc/regist ...   Up      5000/tcp
  registryctl         /harbor/start.sh                 Up
```

如果容器没有处于**UP**状态，请在目录 `/var/log/harbor` 中检查该容器的日志文件。
例如，如果容器 `harbor-core` 没有运行，您应该查看日志文件 `core.log`。


2.当在nginx代理或负载均衡器后设置Harbor时，请查看文件`common/config/nginx/nginx.conf`。
如果代理已经有类似的设置:`location /`, `location /v2/` and `location /service/`，则将其从部分中删除。
``` sh
proxy_set_header X-Forwarded-Proto $scheme;
```


重新部署Harbor请参考前面的章节“管理Harbor的生命周期”。
