## Logstash

### Installation
https://www.elastic.co/guide/en/logstash/current/installing-logstash.html

**deb**
* **Download and install the Public Signing Key:**
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

* You may need to install the apt-transport-https package on Debian before proceeding:
```
sudo apt-get install apt-transport-https
```

* Save the repository definition to /etc/apt/sources.list.d/elastic-7.x.list:
```
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```

* **Install:** `sudo apt-get update && sudo apt-get install logstash`

----

**rpm**
* **Import the Elastic PGP key**
```
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
* Create a file called `logstash.repo` in the `/etc/yum.repos.d/` directory for RedHat based distributions containing:
```
[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
* **Install:** `sudo yum install logstash`

----
----

### Configure logstash
.
.
.
.

----
----

### Start / Stop / Enable service
```
sudo systemctl daemon-reload
sudo systemctl enable logstash.service
sudo systemctl status logstash.service
sudo systemctl start logstash.service
sudo systemctl stop logstash.service
```

* These commands provide no feedback as to whether Logstash was started successfully or not. Log information can be accessed via: `journalctl -u logstash.service`
