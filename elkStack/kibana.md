## Kibana

### Installation
https://www.elastic.co/guide/en/kibana/current/install.html

**deb**
* **Import the Elastic PGP key:**
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
* Install from the APT repository: You may need to install the `apt-transport-https` package on Debian before proceeding:
```
sudo apt-get install apt-transport-https
```
* Save the repository definition to /etc/apt/sources.list.d/elastic-7.x.list:
```
echo "deb https://artifacts.elastic.co/packages/oss-7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```
* **Install:** `sudo apt-get update && sudo apt-get install kibana`

----

**rpm**
* **Import the Elastic PGP key:**
```
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
* Create a file called `kibana.repo` in the `/etc/yum.repos.d/` directory for RedHat based distributions containing:
```
[kibana-7.x]
name=Kibana repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/oss-7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
* **Install:** `sudo yum install kibana`

----
----

### Configure kibana
.
.
.
.

----
----

### Start / Stop / Enable service
```
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
sudo systemctl status kibana.service
sudo systemctl start kibana.service
sudo systemctl stop kibana.service
```

* These commands provide no feedback as to whether Kibana was started successfully or not. Log information can be accessed via: `journalctl -u kibana.service`
