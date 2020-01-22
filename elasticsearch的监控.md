# elasticsearch的监控

## 1.安装elasticsearch

参考

https://blog.csdn.net/ooobama/article/details/95889496

```shell
#启动
bin/elasticsearch 
#后台启动
bin/elasticsearch -d
```

访问数据：

```url
localhost:9200
```

## 2.安装elasticsearch-prometheus-exporter插件

wget指令下载插件

```shell
wget http://download.baiyongjie.com/linux/prometheus/prometheus-exporter-6.6.2.0.zip
```

启动插件

```ruby
/usr/local/elasticsearch/elasticsearch-6.6.2/bin/elasticsearch-plugin install file:///data/elasticsearch-6.6.2/prometheus-exporter-6.6.2.0.zip 
```

安装：

```ruby
Continue with installation? [y/N]y
-> Installed prometheus-exporter
```

## 3.配置Prometheus.yml

```yaml
- job_name: 'es'
    scrape_interval: 30s
    metrics_path: "/_prometheus/metrics"
    static_configs:
    - targets:
      - 10.180.249.60:9200
```

访问10.180.249.60:9200/_prometheus/metrics出现数据则成功

## 4.将数据显示到Grafana上

导入266模板