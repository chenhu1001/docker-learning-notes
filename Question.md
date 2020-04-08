## 1、运行容器报错：Error response from daemon: Error running DeviceCreate (createSnapDevice) dm_task_run failed
```
metadata目录在 docker info查看
```
```
service docker stop
```
```
thin_check /var/lib/docker/devicemapper/devicemapper/metadata
```
```
thin_check --clear-needs-check-flag /var/lib/docker/devicemapper/devicemapper/metadata
```
```
service docker start
```
