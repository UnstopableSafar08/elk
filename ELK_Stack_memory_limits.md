# Elastic Stack Memory Limit Setup

This guide provides commands and configuration for limiting memory usage for the full Elastic Stack:
Elasticsearch, Kibana, Logstash, Metricbeat, Filebeat, Heartbeat.

---

## 1. Elasticsearch (JVM-based)

```bash
mkdir -p /etc/systemd/system/elasticsearch.service.d
cat << EOF > /etc/systemd/system/elasticsearch.service.d/override.conf
[Service]
Environment="ES_JAVA_OPTS=-Xms1g -Xmx1g"
MemoryMax=2G
EOF
systemctl daemon-reexec
systemctl restart elasticsearch
systemctl show elasticsearch --property=MemoryMax
```
### OR
```bash
 vi /etc/elasticsearch/jvm.options

# Configure accordingly avaliable ram.
-Xms8g
-Xmx8g
```


## 2. Kibana (Node.js)

```bash
mkdir -p /etc/systemd/system/kibana.service.d
cat << EOF > /etc/systemd/system/kibana.service.d/override.conf
[Service]
Environment="NODE_OPTIONS=--max-old-space-size=1024"
MemoryMax=2G
EOF
systemctl daemon-reexec
systemctl restart kibana
```

## 3. Logstash (JVM-based)

```bash
mkdir -p /etc/systemd/system/logstash.service.d
cat << EOF > /etc/systemd/system/logstash.service.d/override.conf
[Service]
Environment="LS_JAVA_OPTS=-Xms512m -Xmx512m"
MemoryMax=1G
EOF
systemctl daemon-reexec
systemctl restart logstash
```

## 4. Beats (Metricbeat, Filebeat, Heartbeat - Go binaries)

```bash
for beat in filebeat metricbeat heartbeat; do
  mkdir -p /etc/systemd/system/${beat}.service.d
  cat << EOF > /etc/systemd/system/${beat}.service.d/override.conf
[Service]
MemoryMax=512M
EOF
  systemctl daemon-reexec
  systemctl restart $beat
 done
```

## 5. Verify Memory Limits

```bash
systemctl show elasticsearch kibana logstash filebeat metricbeat heartbeat --property=MemoryMax
ps -o pid,rss,vsz,cmd -p $(pgrep -f elasticsearch)
ps -o pid,rss,vsz,cmd -p $(pgrep -f kibana)
ps -o pid,rss,vsz,cmd -p $(pgrep -f logstash)
ps -o pid,rss,vsz,cmd -p $(pgrep -f filebeat)
ps -o pid,rss,vsz,cmd -p $(pgrep -f metricbeat)
ps -o pid,rss,vsz,cmd -p $(pgrep -f heartbeat)
```

## Notes

* JVM apps: Limit memory using `-Xms/-Xmx` + optional `MemoryMax`.
* Go-based Beats: Limit memory via `MemoryMax` in systemd.
* Use drop-in override files to avoid editing main unit files.
* Reload systemd with `systemctl daemon-reexec` after changes.
