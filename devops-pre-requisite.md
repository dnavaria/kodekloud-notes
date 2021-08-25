# Linux Basics
## Topics left
- Vagrant explore
- Get OS Details => `cat /etc/*release*`
## Package management
- To install package on linux distro we generally use package manager such as `apt`, `rpm`, `yum`, `pacman`
- **`RPM` Red Hat Package Manager**
	- It does not install any dependencies required by a package.
	- Install a Package => `rpm -i telnet.rpm`
	- Uninstall a Package => `rpm -e telnet.r****pm`
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
- Record types in DNS server 
	- `A` records => ipv4 is mapped to hostname in this record
	- `AAAA` records => ipv6 is mapped to hostname in this record
	- `CNAME` records => multiple aliases for the same application / hostname, hostname to hostname mapping
- `dig/nslookup` dns server or nameserver ping command
# Building & Compiling
## Java
- Stages
	- Develop
	- Compile
	- Package
	- Document
- Build Tools for java
	- Maven
	- Gradle
	- ANT 
- Setting environment variable for java `export PATH=$PATH:/opt/jdk-13.0.2/bin` 
- Bulding a jar file => `jar cf <name of jar file> <Classes/dependencies> ... `
- Creating Documentation => `javadoc -d doc <.java file or class>`
![[Pasted image 20210821153122.png]]

### ANT
- Create Documentation => `ant docs`
- Compile and generate jar package using => `ant`
### Maven
- Compile and package => `cd /opt/maven/my-app/; sudo mvn package`
- Run `java -cp <jar file name>`
	
# WebServer
## Apache
- Install apache web server => `sudo yum install -y httpd`
- Check status => `service httpd status`
- Allowing http trafic through firewall => `firewall-cmd --permanent --add-service=http`
	- Or If using ufw or iptables search appropriate commands
- Config File => `/etc/httpd/conf/httpd.conf`
	- You can also create individual configuration file in the above directory and then import them in the httpd.conf
	- `Include conf/<custom config file name>.conf`
### Logs
- Access logs => `/var/log/httpd/access_log`
- Error logs => `/var/log/httpd/error_log`
## Apache Tomcat
- Used to deploy java based web application
- Config File => `conf/server.xml`
- Connecter Tag => Port forwarding, similar to proxy pass in nginx
- webapps directory => application code should be in this directory
- Creating web archive => `jar -cvf app.war *` or use maven or gradle
	- Move app.war to webapps directory and start the server
	- We can verify by tailing logs of catalina.out file present in the logs directory
## Deploying Python App
- Production grade servers for python application deployment
	- Gunicorn
	- uWSGI
	- Gevent
	- Twisted Web
- `sudo sed -i 's/8080/5000/g' app.py; python app.py`
## NodeJS
- PM2 Production process manager
- Install pm2 => `sudo npm install pm2@latest -g`
- Start Single Instance of app => `sudo pm2 start <app name>.js`
- Fork Process and start multiple instance => `sudo pm2 start <app name>.js -i 4`
- Stop Node Process => `pm2 delete <app name>.js or pm2 stop id shown in pm2 ls command`
# Databases
## MySQL
- `sudo yum install mariadb-server`
- `sudo yum install https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm`
	- `sudo yum install mysql-community-server`
- Install MySQL on centos => `yum install mysql-server`
- Start MySQL server => `sudo service mysqld start`
- Initially a temproary password is generated and is logged in the log file => `/var/log/mysqld.log`
	- `sudo grep 'temporary password' /var/log/mysqld.log`
- Login to mysql using that temproary password => `mysql -u root -p<temproary password>`
- We are going to change that  temproary password now => `ALTER USER 'root'@'localhost' IDENTIFIED BY '<new password>';`
- Creating a new user => `CREATE USER '<username>'@'<user ip>' IDENTIFIED BY '<new password>';`
- Creating a new user with access from any machine => `CREATE USER '<username>'@'%' IDENTIFIED BY '<new password>';`
- Giving Privileges => `GRANT <permission> ON <DB.TABLE> TO '<username>'@'<user ip>'`
- `GRANT ALL PRIVILEGES ON kk_db.* TO 'kk_user'@'localhost';`
- Giving Privileges to all tables in all databases  => `GRANT <permission> ON *.* TO '<username>'@'<user ip>'`
# SSL/TLS Certificates
- Generating certificates
	- Generating private key => `openssl genrsa -out <keyname>.key 1024`
	- Generating certificate using private key => `openssl rsa -in <keyname>.key -pubout > <certificate name>.pem`
	- Certificate Signing requests => `openssl req -new -key <keyname>.key -out <name>.csr -sbj "/C=US/ST=CA/O=MyOrg, Inc./CN=<domain name>"`
- Usually certificates with public key have `*.crt or *.pem` extension and the one with the private key have `*.key or *-key.pem`
- `sudo openssl req -new -newkey rsa:2048 -nodes -keyout app01.key -out app01.csr`
- Creating a self-signed certificate => `sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout app01.key -out app01.crt`
- To test if server is using correct certificate or not run this command and check if it returns your certificate:
	- `echo | openssl s_client -showcerts -servername app01.com -connect app01:443 2>/dev/null | openssl x509 -inform pem`
# JSON Path
- Json Path is a query language that when applied to a dataset, returns you subset of that dataset.
- In json query we can replace the root element with `$`
	- `$.car`
- All results of json path query a encapsulated in an array
- Querying a list => `$[0], $[3], or $[0,3] gets you 0th and 3rd element`	
	- fetching elements based on some condition => `$[check if each item in array is > 40]`
		- `check if` can be replaced by `?(<query>)` and `each item in the list` can be replaced by `@`  => `$[?( @ > 40)]`
		- Other operators such as `@ == 40, @!= 40, @ in [x,y,z], @ nin [x,y,z]`
- Query example
	- `$.car.wheels[?(@.location == "rear-right")].model`