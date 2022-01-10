---
title: "K8s with Mac"
date: 2022-01-02T14:10:19+08:00
tags: ["k3s", "k8s", "k9s","multipass"]
categories: ["devops"]
series: [""]
summary: "Run k8s on Mac with k3s and multipass"
draft: false
editPost:
  URL: "https://github.com/Zhoutao822/zhoutao822.github.io/tree/main/content/"
  Text: "Suggest Changes"
  appendFilePath: true 
---

## 1. Multipass && K3s && k9s

### 1.1 Multipass说明

Multipass是一个轻量级的可以用于开启Ubuntu虚拟机的命令行工具，你可以把它当做一个无图形界面的`Virtual Box`或者`Parallels Desktop`。在mac上可以通过brew安装，命令为

```shell
brew install --cask multipass
```

安装完成后执行`multipass version`，输出如下结果说明安装成功

```
~ ❯ multipass version                                                                                20:45:29
multipass   1.8.1+mac
multipassd  1.8.1+mac
```

Multipass常见指令如下：

Launch an instance (by default you get the current Ubuntu LTS)

```shell
multipass launch --name foo
```

Run commands in that instance, try running bash (logout or ctrl-d to quit)

```shell
multipass exec foo -- lsb_release -a
```

Pass a cloud-init metadata file to an instance on launch. See [using cloud-init with multipass](https://blog.ubuntu.com/2018/04/02/using-cloud-init-with-multipass) for more details

```shell
multipass launch -n bar --cloud-init cloud-config.yaml
```

See your instances

```shell
multipass list
```

Stop and start instances

```shell
multipass stop foo bar
multipass start foo
```

Clean up what you don’t need

```shell
multipass delete bar
multipass purge
```

Find alternate images to launch with multipass

```shell
multipass find
```

Get help

```shell
multipass help
multipass help <command>
```

### 1.2 K3s

k3s用于快速搭建k8s集群，注意这里k3s不是安装到mac上的而是安装到multipass创建的Ubuntu实例中，常用k3s安装脚本如下

```shell
curl -sfL https://get.k3s.io | sh -
```

一般来说国内访问可能失败，导致无法拉取需要资源，所以可以使用国内镜像源

```shell
curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -
```

k3s常见环境变量如下

| Environment Variable            | Description                                                  |
| ------------------------------- | ------------------------------------------------------------ |
| `INSTALL_K3S_SKIP_DOWNLOAD`     | 如果设置为 "true "将不会下载 K3s 的哈希值或二进制。          |
| `INSTALL_K3S_SYMLINK`           | 默认情况下，如果路径中不存在命令，将为 kubectl、crictl 和 ctr 二进制文件创建符号链接。如果设置为'skip'将不会创建符号链接，而'force'将覆盖。 |
| `INSTALL_K3S_SKIP_ENABLE`       | 如果设置为 "true"，将不启用或启动 K3s 服务。                 |
| `INSTALL_K3S_SKIP_START`        | 如果设置为 "true "将不会启动 K3s 服务。                      |
| `INSTALL_K3S_VERSION`           | 从 Github 下载 K3s 的版本。如果没有指定，将尝试从"stable"频道下载。 |
| `INSTALL_K3S_BIN_DIR`           | 安装 K3s 二进制文件、链接和卸载脚本的目录，或者使用`/usr/local/bin`作为默认目录。 |
| `INSTALL_K3S_BIN_DIR_READ_ONLY` | 如果设置为 true 将不会把文件写入`INSTALL_K3S_BIN_DIR`，强制设置`INSTALL_K3S_SKIP_DOWNLOAD=true`。 |
| `INSTALL_K3S_SYSTEMD_DIR`       | 安装 systemd 服务和环境文件的目录，或者使用`/etc/systemd/system`作为默认目录。 |
| `INSTALL_K3S_EXEC`              | 带有标志的命令，用于在服务中启动 K3s。如果未指定命令，并且设置了`K3S_URL`，它将默认为“agent”。如果未设置`K3S_URL`，它将默认为“server”。要获得帮助，请参考[此示例。](https://docs.rancher.cn/docs/k3s/installation/install-options/how-to-flags/_index#示例-b-install_k3s_exec) |
| `INSTALL_K3S_NAME`              | 要创建的 systemd 服务名称，如果以服务器方式运行 k3s，则默认为'k3s'；如果以 agent 方式运行 k3s，则默认为'k3s-agent'。如果指定了服务名，则服务名将以'k3s-'为前缀。 |
| `INSTALL_K3S_TYPE`              | 要创建的 systemd 服务类型，如果没有指定，将默认使用 K3s exec 命令。 |
| `INSTALL_K3S_SELINUX_WARN`      | 如果设置为 true，则在没有找到 k3s-selinux 策略的情况下将继续。 |
| `INSTALL_K3S_SKIP_SELINUX_RPM`  | 如果设置为 "true "将跳过 k3s RPM 的自动安装。                |
| `INSTALL_K3S_CHANNEL_URL`       | 用于获取 K3s 下载网址的频道 URL。默认为 https://update.k3s.io/v1-release/channels 。 |
| `INSTALL_K3S_CHANNEL`           | 用于获取 K3s 下载 URL 的通道。默认值为 "stable"。选项包括：`stable`, `latest`, `testing`。 |
| `K3S_CONFIG_FILE`               | 指定配置文件的位置。默认目录为`/etc/rancher/k3s/config.yaml`。 |
| `K3S_TOKEN`                     | 用于将 server 或 agent 加入集群的共享 secret。               |
| `K3S_TOKEN_FILE`                | 指定 `cluster-secret`,`token` 的文件目录。                   |

### 1.3 multipass-k3s脚本

[k3s cluster on multipass instances](https://github.com/superseb/multipass-k3s)给出了脚本，可以直接利用multipass和k3s创建k8s集群，我做了一些修改，比如替换了国内用的镜像源、使用2个slave节点、修改内存之类

```shell
#!/usr/bin/env bash

# Configure your settings
# Name for the cluster/configuration files
NAME="demo-cluster"
# Ubuntu image to use (xenial/bionic)
IMAGE="focal"
# How many additional server instances to create
SERVER_COUNT_MACHINE="0"
# How many agent instances to create
AGENT_COUNT_MACHINE="2"
# How many CPUs to allocate to each machine
SERVER_CPU_MACHINE="2"
AGENT_CPU_MACHINE="1"
# How much disk space to allocate to each machine
SERVER_DISK_MACHINE="5G"
AGENT_DISK_MACHINE="5G"
# How much memory to allocate to each machine
SERVER_MEMORY_MACHINE="2G"
AGENT_MEMORY_MACHINE="1G"
# Install channel to use (embedded etcd is fully supported starting with v1.19.5+k3s1)
CHANNEL=stable
# Preconfigured secret to join the cluster (or autogenerated if empty)
SERVER_TOKEN=""
# Preconfigured secret to join the cluster (or autogenerated if empty)
AGENT_TOKEN=""


## Nothing to change after this line
if [ -x "$(command -v multipass.exe)" > /dev/null 2>&1 ]; then
    # Windows
    MULTIPASSCMD="multipass.exe"
elif [ -x "$(command -v multipass)" > /dev/null 2>&1 ]; then
    # Linux/MacOS
    MULTIPASSCMD="multipass"
else
    echo "The multipass binary (multipass or multipass.exe) is not available or not in your \$PATH"
    exit 1
fi

if [ -z $SERVER_TOKEN ]; then
    SERVER_TOKEN=$(cat /dev/urandom | base64 | tr -dc 'a-zA-Z0-9' | fold -w 20 | head -n 1 | tr '[:upper:]' '[:lower:]')
    echo "No server token given, generated server token: ${SERVER_TOKEN}"
fi

if [ -z $AGENT_TOKEN ]; then
    AGENT_TOKEN=$(cat /dev/urandom | base64 | tr -dc 'a-zA-Z0-9' | fold -w 20 | head -n 1 | tr '[:upper:]' '[:lower:]')
    echo "No agent token given, generated agent token: ${AGENT_TOKEN}"
fi

# Check if name is given or create random string
if [ -z $NAME ]; then
    NAME=$(cat /dev/urandom | base64 | tr -dc 'a-zA-Z0-9' | fold -w 6 | head -n 1 | tr '[:upper:]' '[:lower:]')
    echo "No name given, generated name: ${NAME}"
fi

echo "Creating cluster ${NAME} with $(( $SERVER_COUNT_MACHINE + 1 )) server(s) and ${AGENT_COUNT_MACHINE} agent(s)"

# Prepare cloud-init
# Cloud init template
read -r -d '' SERVER_INIT_CLOUDINIT_TEMPLATE << EOM
#cloud-config

runcmd:
 - '\curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn INSTALL_K3S_CHANNEL=$CHANNEL K3S_TOKEN=$SERVER_TOKEN K3S_AGENT_TOKEN=$AGENT_TOKEN INSTALL_K3S_EXEC="server --cluster-init" K3S_KUBECONFIG_MODE=644 sh -'
EOM

echo "$SERVER_INIT_CLOUDINIT_TEMPLATE" > "${NAME}-init-cloud-init.yaml"
echo "Cloud-init is created at ${NAME}-init-cloud-init.yaml"

echo "Creating initial server instance: k3s-server-${NAME}"

echo "Running $MULTIPASSCMD launch --cpus $SERVER_CPU_MACHINE --disk $SERVER_DISK_MACHINE --mem $SERVER_MEMORY_MACHINE $IMAGE --name k3s-server-$NAME --cloud-init ${NAME}-init-cloud-init.yaml"
$MULTIPASSCMD launch --cpus $SERVER_CPU_MACHINE --disk $SERVER_DISK_MACHINE --mem $SERVER_MEMORY_MACHINE $IMAGE --name k3s-server-$NAME --cloud-init "${NAME}-init-cloud-init.yaml"
if [ $? -ne 0 ]; then
    echo "There was an error launching the instance"
    exit 1
fi

echo "Checking for Node being Ready on k3s-server-${NAME}"
$MULTIPASSCMD exec k3s-server-$NAME -- /bin/bash -c 'while [[ $(sudo k3s kubectl get nodes --no-headers 2>/dev/null | grep -c -v "NotReady") -eq 0 ]]; do sleep 2; done'
echo "Node is Ready on k3s-server-${NAME}"

# Retrieve info to join agent to cluster
SERVER_IP=$($MULTIPASSCMD info k3s-server-$NAME | grep IPv4 | awk '{ print $2 }')
URL="https://$(echo $SERVER_IP | sed -e 's/[[:space:]]//g'):6443"

# Create additional servers
if [ "${SERVER_COUNT_MACHINE}" -gt 0 ]; then
    read -r -d '' SERVER_CLOUDINIT_TEMPLATE << EOM
#cloud-config

runcmd:
 - '\curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn INSTALL_K3S_CHANNEL=$CHANNEL K3S_TOKEN=$SERVER_TOKEN K3S_AGENT_TOKEN=$AGENT_TOKEN INSTALL_K3S_EXEC="server --server $URL" K3S_KUBECONFIG_MODE=644 sh -'
EOM

    echo "$SERVER_CLOUDINIT_TEMPLATE" > "${NAME}-cloud-init.yaml"

    echo "Creating ${SERVER_COUNT_MACHINE} additional server instances"
    for i in $(eval echo "{1..$SERVER_COUNT_MACHINE}"); do
        echo "Running $MULTIPASSCMD launch --cpus $SERVER_CPU_MACHINE --disk $SERVER_DISK_MACHINE --mem $SERVER_MEMORY_MACHINE $IMAGE --name k3s-server-$NAME-$i --cloud-init ${NAME}-cloud-init.yaml"
        $MULTIPASSCMD launch --cpus $SERVER_CPU_MACHINE --disk $SERVER_DISK_MACHINE --mem $SERVER_MEMORY_MACHINE $IMAGE --name k3s-server-$NAME-$i --cloud-init "${NAME}-cloud-init.yaml"
        if [ $? -ne 0 ]; then
            echo "There was an error launching the instance"
            exit 1
        fi

        echo "Checking for Node being Ready on k3s-server-${NAME}"
        $MULTIPASSCMD exec k3s-server-$NAME-$i -- /bin/bash -c 'while [[ $(sudo k3s kubectl get nodes --no-headers 2>/dev/null | grep -c -v "NotReady") -eq 0 ]]; do sleep 2; done'
        echo "Node is Ready on k3s-server-${NAME}-${i}"
    done
fi

if [ "${AGENT_COUNT_MACHINE}" -gt 0 ]; then
    # Prepare agent cloud-init
    # Cloud init template
    read -r -d '' AGENT_CLOUDINIT_TEMPLATE << EOM
#cloud-config

runcmd:
 - '\curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn INSTALL_K3S_CHANNEL=$CHANNEL K3S_TOKEN=$AGENT_TOKEN K3S_URL=$URL sh -'
EOM

    echo "$AGENT_CLOUDINIT_TEMPLATE" > "${NAME}-agent-cloud-init.yaml"
    echo "Cloud-init is created at ${NAME}-agent-cloud-init.yaml"

    for i in $(eval echo "{1..$AGENT_COUNT_MACHINE}"); do
        echo "Running $MULTIPASSCMD launch --cpus $AGENT_CPU_MACHINE --disk $AGENT_DISK_MACHINE --mem $AGENT_MEMORY_MACHINE $IMAGE --name k3s-agent-$NAME-$i --cloud-init ${NAME}-agent-cloud-init.yaml"
        $MULTIPASSCMD launch --cpus $AGENT_CPU_MACHINE --disk $AGENT_DISK_MACHINE --mem $AGENT_MEMORY_MACHINE $IMAGE --name k3s-agent-$NAME-$i --cloud-init "${NAME}-agent-cloud-init.yaml"
        if [ $? -ne 0 ]; then
            echo "There was an error launching the instance"
            exit 1
       fi
        echo "Checking for Node k3s-agent-$NAME-$i being registered"
        $MULTIPASSCMD exec k3s-server-$NAME -- bash -c "until sudo k3s kubectl get nodes --no-headers | grep -c k3s-agent-$NAME-$i >/dev/null; do sleep 2; done" 
        echo "Checking for Node k3s-agent-$NAME-$i being Ready"
        $MULTIPASSCMD exec k3s-server-$NAME -- bash -c "until sudo k3s kubectl get nodes --no-headers | grep k3s-agent-$NAME-$i | grep -c -v NotReady >/dev/null; do sleep 2; done"
        echo "Node k3s-agent-$NAME-$i is Ready on k3s-server-${NAME}"
    done
fi

$MULTIPASSCMD copy-files k3s-server-$NAME:/etc/rancher/k3s/k3s.yaml $NAME-kubeconfig-orig.yaml
sed "/^[[:space:]]*server:/ s_:.*_: \"https://$(echo $SERVER_IP | sed -e 's/[[:space:]]//g'):6443\"_" $NAME-kubeconfig-orig.yaml > $NAME-kubeconfig.yaml

echo "k3s setup finished"
$MULTIPASSCMD exec k3s-server-$NAME -- sudo k3s kubectl get nodes
echo "You can now use the following command to connect to your cluster"
echo "$MULTIPASSCMD exec k3s-server-${NAME} -- sudo k3s kubectl get nodes"
echo "Or use kubectl directly"
echo "kubectl --kubeconfig ${NAME}-kubeconfig.yaml get nodes"
```

### 1.4 k9s

K9s 提供了一个与 K8s 集群交互的终端 UI，用于简化导航、观察以及管理应用程序。K9s 会持续监控 K8s 的变化，并提供后续命令与所观察到的资源进行交互。可以利用如下GUI管理k8s

![Screen Shot 2022-01-06 at 19.43.21](https://gitee.com/tao2333/hugo-pic/raw/master/pictures/202201062048276.png)

### 1.5 一些问题

1. brew无法安装cask，超时之类，可以使用[中科大的brew源](https://mirrors.ustc.edu.cn/help/homebrew-cask.git.html)；
2. 在启用某些VPN软件时，multipass无法拉取镜像，导致无法创建Ubuntu实例或者无法安装k3s，只能关闭VPN软件；
3. k3s启动失败，一般需要使用国内镜像源。

## 2. 安装与测试

步骤如下：

1. Brew安装multipass、k9s和kubectl-cli；
2. 创建`k3s-launch.sh`，并复制粘贴上面的脚本内容，需要自行修改以适配自己的环境；
3. 运行`bash k3s-launch.sh`，等待集群部署，脚本执行成功后可以看到multipass多了3个实例，不带数字的是master节点，其他是slave节点，而且目录下多了几个文件；

![Screen Shot 2022-01-02 at 21.09.22](https://gitee.com/tao2333/hugo-pic/raw/master/pictures/202201022109129.png)

```
~/Projects/k3s ❯ ll                                                                                  21:13:40
total 56
-rw-r--r--  1 tao  staff   216B Jan  2 14:05 demo-cluster-agent-cloud-init.yaml
-rw-r--r--  1 tao  staff   283B Jan  2 14:04 demo-cluster-init-cloud-init.yaml
-rw-r--r--  1 tao  staff   2.9K Jan  2 14:07 demo-cluster-kubeconfig-orig.yaml
-rw-r--r--  1 tao  staff   2.9K Jan  2 14:07 demo-cluster-kubeconfig.yaml
-rw-r--r--@ 1 tao  staff   7.0K Jan  2 14:04 k3s-launch.sh
```

4. 创建一个deploy-nginx.yaml，内容如下：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:1.17.1
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
```

5. 使用kubectl控制集群，运行如下命令

```shell
# get nodes查看节点是否存活
~/Projects/k3s ❯ kubectl --kubeconfig demo-cluster-kubeconfig.yaml get nodes        ⎈ docker-desktop 21:13:53
NAME                       STATUS   ROLES                       AGE     VERSION
k3s-agent-demo-cluster-1   Ready    <none>                      7h10m   v1.22.5+k3s1
k3s-agent-demo-cluster-2   Ready    <none>                      7h9m    v1.22.5+k3s1
k3s-server-demo-cluster    Ready    control-plane,etcd,master   7h11m   v1.22.5+k3s1
```

   ```shell
   # create namespace dev创建dev命名空间
   ~/Projects/k3s ❯ kubectl --kubeconfig demo-cluster-kubeconfig.yaml create namespace dev
   namespace/dev created
   ```

```shell
# 创建测试pod
~/Projects/k3s ❯ kubectl --kubeconfig demo-cluster-kubeconfig.yaml create -f deploy-nginx.yaml                       ⎈ docker-desktop 21:48:22
deployment.apps/nginx created
# 查看pod状态
~/Projects/k3s ❯ kubectl --kubeconfig demo-cluster-kubeconfig.yaml get pods -n dev                                   ⎈ docker-desktop 21:48:32
NAME                     READY   STATUS    RESTARTS   AGE
nginx-66ffc897cf-55b6d   1/1     Running   0          53s
nginx-66ffc897cf-d5r29   1/1     Running   0          53s
nginx-66ffc897cf-vfpkg   1/1     Running   0          53s
# 创建暴露给外部的Service
~/Projects/k3s ❯ kubectl --kubeconfig demo-cluster-kubeconfig.yaml expose deploy nginx --name=svc-nginx --type=NodePort --port=80 --target-port=80 -n dev
service/svc-nginx exposed
# 查看Service状态和端口号映射
~/Projects/k3s ❯ kubectl --kubeconfig demo-cluster-kubeconfig.yaml get svc svc-nginx -n dev -o wide                  ⎈ docker-desktop 21:50:14
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE   SELECTOR
svc-nginx   NodePort   10.43.227.135   <none>        80:30355/TCP   25s   run=nginx
# 查看master节点ip
~/Projects/k3s ❯ multipass list                                                                                                       21:50:39
Name                      State             IPv4             Image
k3s-agent-demo-cluster-1  Running           192.168.64.16    Ubuntu 20.04 LTS
                                            10.42.1.0
                                            10.42.1.1
k3s-agent-demo-cluster-2  Running           192.168.64.17    Ubuntu 20.04 LTS
                                            10.42.2.0
                                            10.42.2.1
k3s-server-demo-cluster   Running           192.168.64.15    Ubuntu 20.04 LTS
                                            10.42.0.0
                                            10.42.0.1
# 访问 192.168.64.15:30355，能够输出nginx信息
~/Projects/k3s ❯ curl 192.168.64.15:30355                                                                                             21:51:45
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

6. 利用k9s查看k8s集群状态，详细玩法可以去看[k9s官网](https://k9scli.io)

```shell
k9s --kubeconfig demo-cluster-kubeconfig.yaml -n dev
```

pod状态

![Screen Shot 2022-01-06 at 20.51.53](https://gitee.com/tao2333/hugo-pic/raw/master/pictures/202201062054107.png)

service状态

![Screen Shot 2022-01-06 at 20.52.04](https://gitee.com/tao2333/hugo-pic/raw/master/pictures/202201062054265.png)

deploy状态

![Screen Shot 2022-01-06 at 20.53.11](https://gitee.com/tao2333/hugo-pic/raw/master/pictures/202201062054797.png)

# 参考

1. [k3s cluster on multipass instances](https://github.com/superseb/multipass-k3s)
2. [K3s: Lightweight Kubernetes](https://k3s.io/)
2. [Ubuntu VMs on demand for any workstation](https://multipass.run/)
2. [K3s 安装选项介绍](https://docs.rancher.cn/docs/k3s/installation/install-options/_index)
2. [k9s Kubernetes CLI To Manage Your Clusters In Style!](https://k9scli.io)