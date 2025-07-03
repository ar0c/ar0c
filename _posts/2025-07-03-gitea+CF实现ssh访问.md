# 方案
1. 使用公网ip不使用cf代理 + 路由器端口转发

缺点

    1. 需要连接ssh时设置端口号
    2. 安全性差
2. 使用 cloudflare tunnel + 子域名转发

缺点

    1. 客户点需要安装cloudflared
    2. .ssh/config需要手动配置

# 方案3步骤
环境 alpine + k3s, ssh域名为ssh.example.com git域名为git.example.com

```bash
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o /usr/local/bin/cloudflared
chmod +x /usr/local/bin/cloudflared
cloudflared --version
```



```bash
cloudflared tunnel login

#生成配置文件
cloudflared tunnel create gitea-ssh

vim /root/.cloudflared/config.yml
```



```yaml
tunnel: gitea-ssh
# 前面生成的文件
credentials-file: /root/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: ssh.example.com
    service: ssh://localhost:30022
  - service: http_status:404

```



> 注意尽量不要走代理，使用DIRECT规则
>

```bash
# 启动tunnel
cloudflared tunnel run gitea-ssh
# 添加dns解析
cloudflared tunnel route dns gitea-ssh ssh.example.com
```

作为服务运行

```bash
cat <<EOF > /etc/init.d/cloudflared
#!/sbin/openrc-run

command=/usr/local/bin/cloudflared
command_args="--config /etc/cloudflared/config.yml tunnel run"
pidfile=/var/run/cloudflared.pid
name=cloudflared
description="Cloudflare Tunnel"
command_background=true
output_log="/var/log/cloudflared.log"
error_log="/var/log/cloudflared.err"

depend() {
  need net
}
EOF
```

```bash
chmod +x /etc/init.d/cloudflared
rc-update add cloudflared
rc-service cloudflared start
```

### 访问
需要先安装cloudflared

```bash
 ssh -o "ProxyCommand=cloudflared access ssh --hostname ssh.example.com" git@git.example.com
```

配置.ssh/config

```plain
Host git.example.com
ProxyCommand cloudflared access ssh --hostname git.example.com
```

