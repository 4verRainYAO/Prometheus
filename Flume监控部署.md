# Flume监控部署

## 1.暴漏指标

启动flume时添加启动参数：

Dflume.monitoring.type=http -Dflume.monitoring.port=30000

其中端口号30000自己自定义

```shell
bin/flume-ng agent --conf conf --conf-file conf/flume.conf --name a1 -Dflume.monitoring.type=http -Dflume.monitoring.port=30000 -Dflume.root.logger=INFO,console
```



### 1.部署flume_exporter

通过git 下载https://github.com/woozhijun/flume_exporter项目，上传到linux上

#### 1.1修改flume_exporter项目下的config.yml

 ```yaml
agents:
- name: "flume-agents"
  enabled: true
# multiple urls can be separated by ,
  urls: ["http://localhost:30000/metrics"]
 ```

其中urls端口号要对应flume启动时暴漏的那个端口号

#### 1.2 修改metrics.yml文件，这里面主要是配置的监控指标，暂时没有修改，监控了所有的指标

#### 1.3编译flume_exporter项目出现可执行文件flume_exporter

#### 1.4启动flume_exporter

```shell
./flume_exporter --metric-file=./metrics.yml --config-file=./config.yml &
```

启动之后默认端口号9360，访问数据

```url
http://localhost:9360/metrics
```

## 2.整合到Prometheus

#### 2.1 修改**prometheus.yml**

```yaml
- job_name: flume-discovery
    file_sd_configs:
      - files:
        - '/usr/local/prometheus/prometheus-2.8.1.linux-amd64/config/flume-nodes.json'
    metric_relabel_configs:
      - source_labels: [host]
        regex: '10.180.249.60:30000'
        replacement: flume-192.168.112.46-34557
        target_label: logs
        action: replace
```

#### 2.2  **flume-nodes.json文件内容如下**

```json
[{
	"targets": ["localhost:9360"],
	"labels": {
		"alias": "flume-kafka",
		"job": "flume"
	}
}]
```

#### 2.3 **reload prometheus的配置文件**

 ```
curl -X POST http://localhost:9090/-/reload
 ```

在启动prometheus时加入如下参数，开启lifecycle

```
./prometheus --config.file=prometheus.yml --web.enable-lifecycle
```

#### **2.4 reload成功之后可在prometheus web ui查看到监控的指标**

## 3.Grafana图标配置

https://grafana.com/grafana/dashboards/10736/revisions下载Flume监控模板

将改模板导入Grafana