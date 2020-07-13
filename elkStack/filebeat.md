## Filebeat

### Installation
**deb**
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.8.0-amd64.deb
sudo dpkg -i filebeat-7.8.0-amd64.deb
```
----
**rpm**
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.8.0-x86_64.rpm
sudo rpm -vi filebeat-7.8.0-x86_64.rpm
```

----
----

### Configure filebeat
.
.
.
.

----
----

### Start / Stop / Enable service
```
sudo systemctl daemon-reload
sudo systemctl enable filebeat.service
sudo systemctl status filebeat.service
sudo systemctl start filebeat.service
sudo systemctl stop filebeat.service
```

* These commands provide no feedback as to whether Filebeat was started successfully or not. Log information can be accessed via:  `journalctl -u filebeat.service`
