## Elasticsearch

### Installation
https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html

**deb**
* Download and install the public signing key:
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
* You may need to install the `apt-transport-https` package on Debian before proceeding:
```
sudo apt-get install apt-transport-https
```
* Save the repository definition to `/etc/apt/sources.list.d/elastic-7.x.list`:
```
echo "deb https://artifacts.elastic.co/packages/oss-7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```
* **Install:** `sudo apt-get update && sudo apt-get install elasticsearch`


**rpm**
* **Import the Elastic PGP key**
```
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
* Create a file called `elasticsearch.repo` in the `/etc/yum.repos.d/` directory for RedHat based distributions containing:
```
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/oss-7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```
* **Install:** `sudo yum install --enablerepo=elasticsearch elasticsearch`

----
----

### Configure Elasticsearch
.
.
.
.
.

----
----

### Start / Stop / Enable service
```
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl status elasticsearch.service
sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service
```

* These commands provide no feedback as to whether Elasticsearch was started successfully or not. Log information can be accessed via: `journalctl --unit elasticsearch`
