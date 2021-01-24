## 外网远程登录家中没公网IP的主机

通过一台公网vps当作跳板机，用内网穿透的方法来远程登录没有公网IP的主机，并保证不会因为公网vps的地址变更而无法登录内网主机的问题。

svr：有公网IP的vps

cnt：家里没有公网IP的内网主机

### 步骤
1. cnt已配置好ssh，只允许授信的公钥登录，保证连接Alive，端口为p1。
2. 在svr上安装frps，cnt上安装frpc，并分别配置好ini配置文件。svr和cnt通过frp建立连接，是通过cnt发起的连接，因为svr有公网IP。cnt将p1端口映射给svr的p2端口，这样我们就可以通过svr的p2端口来访问cnt的p1 (via frp)。相当于我们的ssh请求先到svr，svr再通过frp转发给cnt，即实现了内网穿透。
3. frps和frpc都需要添加到systemd中，并且随开机启动。svr和cnt都需要互相添加ssh授信公钥，并且若平时用笔记本远程登录，将笔记本的ssh公钥也要分别添加两台机器内。
4. 因为不能保证svr跳板机的IP永远不变，比如svr的地址在国内被墙了，应为frp的连接是通过cnt发起的，这样就会导致cnt无法再连接到新的svr上，应让cnt有主动更新svr的地址能力，这样即使svr地址变化，cnt也能自己主动重新连接到新的svr跳板机上。在cnt上创建crontab任务，通过脚本定时去拉svr的最新IP地址（比如把地址放到github的文件中），然后和本地已配置的svr地址比较。若svr地址变更，则更新cnt上frp的配置文件，并重启cnt上的frp客户端。


### 参考
1. 安装frp：https://www.mls-tech.info/linux/ubuntu-18-frp/ 文中systemd应该是在etc下的；fprc也应用systemd启动。
2. frp配置：https://zhuanlan.zhihu.com/p/57477087
3. frp的配置文件在google drive中也备份了一份。
