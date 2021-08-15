# Linux Basics
## Topics left
- Vagrant explore
- Get OS Details => `cat /etc/*release*`
## Package management
- To install package on linux distro we generally use package manager such as `apt`, `rpm`, `yum`, `pacman`
- **`RPM` Red Hat Package Manager**
	- It does not install any dependencies required by a package.
	- Install a Package => `rpm -i telnet.rpm`
	- Uninstall a Package => `rpm -e telnet.rpm`
	- Query the database to get deatils about the package => `rpm -q telnet.rpm`
	- List packages => `rpm -qa`
- **`YUM`** 
	- YUM is a high level package manager that uses rpm underneath.
	- It install all the dependencies required by a package.
	- List YUM repos => `yum repolist`
	- List packages => `yum list or yum list <package name>`
		- List duplicate packages from different repos => `yum --showduplicates list <package name>`
	- Remove package => `yum remove ansible`
	- Install specific version of a package => `yum install <package name>-version`
## Services
- `service` command uses `systemctl` utilities underneath
- Start a service => `systemctl start <service name>`
- Stop a service => `systemctl stop <service name>`
- Check Status of a service => `systemctl status <service name>`
- Enable a service => `systemctl enable <service name>`
- Disable a service => `systemctl disable <service name>`
- systemd unit files directory => `/etc/systemd/system/`
```
[Unit]
Description=Service Description

[Service]
ExecStart=<command to start the application>
ExecStartPre=<command or script to be executed before running the service>
ExecStartPost=<commands or script to be executed after running the service>
Restart=always

[Install]
#Configure service to run after a particular service that runs at boot up, we can configure this service to run after the multi-user run level is started
WantedBy=multi-user.target 
```
# Virtual Box & Networking Basics
- To list and modify interfaces on the host => `ip link`
- IP show => `ip addr show`
- Set IP => `ip addr add <ip address>/<subnet mask> <name> <adapter name>` 
- Show routing table  =>  `route`
- Adding gateway => `ip route add 192.168.2.0/24 via 192.168.1.1`
- Using router as default gateway => `ip route add deafult via 192.168.2.1`
	- Instead of `default` we can also use `0.0.0.0`+
- Enable IP forward => `cat /proc/sys/net/ipv4/ip_forward` 
	- If it shows 1 then ip forward in enbled else change it to 1 to enable it 
		- `echo 1 > /proc/sys/net/ipv4/ip_forward`
	- To make the changes permanent
		- Edit `/etc/sysctl.conf`, add `net.ipv4.ip_forward=1`