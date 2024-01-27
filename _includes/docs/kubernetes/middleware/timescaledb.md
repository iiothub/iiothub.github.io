* TOC
{:toc}



## 一、概述



### 1.Helm

**Bitnami**

```shell
https://bitnami.com/
https://github.com/bitnami
https://bitnami.com/stacks
https://github.com/bitnami/charts/tree/master/bitnami
```



```shell
# timescale/timescaledb-kubernetes
https://github.com/timescale/timescaledb-kubernetes


# 安装
# TimescaleDB Single
https://github.com/timescale/timescaledb-kubernetes/tree/master/charts/timescaledb-single


# 官方
# TimescaleDB Multinode
https://github.com/timescale/timescaledb-kubernetes/blob/master/charts/timescaledb-multinode
```

```shell
[root@k8s-master timescale]# helm repo add timescale 'https://charts.timescale.com'
"timescale" has been added to your repositories


[root@k8s-master timescale]# helm repo list
NAME         	URL                                                   
aliyun       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
bitnami      	https://charts.bitnami.com/bitnami                    
ingress-nginx	https://kubernetes.github.io/ingress-nginx            
gitlab       	https://charts.gitlab.io                              
elastic      	https://helm.elastic.co                               
harbor       	http://172.51.216.85:8888/chartrepo/charts            
chartmuseum  	http://172.51.216.85:9999                             
presslabs    	https://presslabs.github.io/charts                    
timescale    	https://charts.timescale.com


[root@k8s-master timescale]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "chartmuseum" chart repository
...Successfully got an update from the "harbor" chart repository
...Successfully got an update from the "elastic" chart repository
...Successfully got an update from the "ingress-nginx" chart repository
...Successfully got an update from the "aliyun" chart repository
...Successfully got an update from the "presslabs" chart repository
...Successfully got an update from the "timescale" chart repository
...Successfully got an update from the "gitlab" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈


[root@k8s-master timescale]# helm search repo timescaledb
NAME                           	CHART VERSION	APP VERSION	DESCRIPTION                      
timescale/timescaledb-multinode	0.8.0        	           	TimescaleDB Multinode Deployment.
timescale/timescaledb-single   	0.8.2        	           	TimescaleDB HA Deployment. 
```



| Recipe                                                       | TimescaleDB    | Description                                                  |
| ------------------------------------------------------------ | -------------- | ------------------------------------------------------------ |
| [TimescaleDB Single](https://github.com/timescale/timescaledb-kubernetes/blob/master/charts/timescaledb-single) | Based on 1.x   | TimescaleDB Single allows you to deploy a highly-available TimescaleDB database configuration. |
| [TimescaleDB Multinode](https://github.com/timescale/timescaledb-kubernetes/blob/master/charts/timescaledb-multinode) | In Development | TimescaleDB Multinode allows you to deploy a multi-node, distributed version of TimescaleDB. |





## 二、基础



### 1.Helm

```shell
[root@k8s-master timescale]# helm search repo timescaledb
NAME                           	CHART VERSION	APP VERSION	DESCRIPTION                      
timescale/timescaledb-multinode	0.8.0        	           	TimescaleDB Multinode Deployment.
timescale/timescaledb-single   	0.8.2        	           	TimescaleDB HA Deployment.  


# timescale/timescaledb-kubernetes
https://github.com/timescale/timescaledb-kubernetes


# TimescaleDB Single
https://github.com/timescale/timescaledb-kubernetes/tree/master/charts/timescaledb-single


# TimescaleDB Multinode
https://github.com/timescale/timescaledb-kubernetes/blob/master/charts/timescaledb-multinode


# 所有版本
[root@k8s-master ~]# helm search repo timescaledb --versions
NAME                           	CHART VERSION	APP VERSION	DESCRIPTION                      
timescale/timescaledb-multinode	0.8.0        	           	TimescaleDB Multinode Deployment.
timescale/timescaledb-multinode	0.6.3        	           	TimescaleDB Multinode Deployment.
timescale/timescaledb-multinode	0.3.0        	           	TimescaleDB Multinode Deployment.
timescale/timescaledb-single   	0.8.2        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.8.1        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.8.0        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.7.1        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.7.0        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.6.2        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.6.1        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.6.0        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.5.8        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.5.7        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.5.6        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.5.5        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.5.4        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.5.3        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.5.2        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.5.1        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.5.0        	           	TimescaleDB HA Deployment.       
timescale/timescaledb-single   	0.4.0        	           	TimescaleDB HA Deployment.


helm fetch timescale/timescaledb-single --version 0.8.1
helm fetch timescale/timescaledb-multinode --version 0.6.3
```





## 三、实践



### 1.部署高可用TimescaleDB

```shell
# 部署单实例

[root@k8s-master timescale]# helm search repo timescaledb
NAME                           	CHART VERSION	APP VERSION	DESCRIPTION                      
timescale/timescaledb-single   	0.8.2        	           	TimescaleDB HA Deployment. 


# TimescaleDB Single
https://github.com/timescale/timescaledb-kubernetes/tree/master/charts/timescaledb-single
```



#### 1.1.下载安装包

```shell
[root@k8s-master timescale]# helm fetch timescale/timescaledb-single

[root@k8s-master timescale]# ll
total 56
-rw-r--r-- 1 root root 55788 Dec  7 16:53 timescaledb-single-0.8.2.tgz
[root@k8s-master timescale]# tar -zxf timescaledb-single-0.8.2.tgz 
[root@k8s-master timescale]# ll
total 56
drwxr-xr-x 5 root root   257 Dec  7 16:53 timescaledb-single
-rw-r--r-- 1 root root 55788 Dec  7 16:53 timescaledb-single-0.8.2.tgz
[root@k8s-master timescale]# cd timescaledb-single
[root@k8s-master timescaledb-single]# ll
total 120
-rw-r--r-- 1 root root 44242 Jan 13  2021 admin-guide.md
-rw-r--r-- 1 root root   370 Jan 13  2021 Chart.yaml
-rw-r--r-- 1 root root  4432 Jan 13  2021 generate_kustomization.sh
drwxr-xr-x 4 root root    55 Dec  7 16:53 kustomize
-rw-r--r-- 1 root root  6875 Jan 13  2021 README.md
-rw-r--r-- 1 root root   188 Jan 13  2021 requirements.yaml
drwxr-xr-x 2 root root  4096 Dec  7 16:53 templates
-rw-r--r-- 1 root root  5693 Jan 13  2021 upgrade-guide.md
drwxr-xr-x 2 root root   141 Dec  7 16:53 values
-rw-r--r-- 1 root root 13355 Jan 13  2021 values.schema.yaml
-rw-r--r-- 1 root root 21571 Jan 13  2021 values.yaml
```



#### 1.2.修改配置

```shell
# values.yaml


persistentVolumes:
  # For sanity reasons, the actual PGDATA and wal directory will be subdirectories of the Volume mounts,
  # this allows Patroni/a human/an automated operator to move directories during bootstrap, which cannot
  # be done if we did not use subdirectories
  # https://www.postgresql.org/docs/current/creating-cluster.html#CREATING-CLUSTER-MOUNT-POINTS
  data:
    enabled: true
    size: 20Gi
    ## database data Persistent Volume Storage Class
    ## If defined, storageClassName: <storageClass>
    ## If set to "-", storageClassName: "", which disables dynamic provisioning
    ## If undefined (the default) or set to null, no storageClassName spec is
    ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
    ##   GKE, AWS & OpenStack)
    ##
    storageClass: "rook-ceph-block"
    subPath: ""
    mountPath: "/var/lib/postgresql"
    annotations: {}
    accessModes:
      - ReadWriteOnce
  # WAL will be a subdirectory of the data volume, which means enabling a separate
  # volume for the WAL files should just work for new pods.
  wal:
    enabled: true
    size: 1Gi
    subPath: ""
    storageClass: "rook-ceph-block"
    # When changing this mountPath ensure you also change the following key to reflect this:
    # patroni.postgresql.basebackup.[].waldir
    mountPath: "/var/lib/postgresql/wal"
    annotations: {}
    accessModes:
      - ReadWriteOnce
  # Any tablespace mentioned here requires a volume that will be associated with it.
  # tablespaces:
    # example1:
    #   size: 5Gi
    #   storageClass: gp2
    # example2:
    #   size: 5Gi
    #   storageClass: gp2


# 修改存储
# 修改内容
storageClass: "rook-ceph-block"
size: 20Gi
```



#### 1.3.安装

```shell
# 安装证书

[root@k8s-master timescaledb-single]# pwd
/k8s/middleware/timescale/timescaledb-single
[root@k8s-master timescaledb-single]# ll
total 120
-rw-r--r-- 1 root root 44051 Jan 13  2021 admin-guide.md
-rw-r--r-- 1 root root   370 Jan 13  2021 Chart.yaml

# 证书相关文件
-rw-r--r-- 1 root root  4432 Jan 13  2021 generate_kustomization.sh
drwxr-xr-x 4 root root    55 Dec  8 10:07 kustomize


-rw-r--r-- 1 root root  6875 Jan 13  2021 README.md
-rw-r--r-- 1 root root   188 Jan 13  2021 requirements.yaml
drwxr-xr-x 2 root root  4096 Dec  8 10:28 templates
-rw-r--r-- 1 root root  5693 Jan 13  2021 upgrade-guide.md
drwxr-xr-x 2 root root   141 Dec  8 10:07 values
-rw-r--r-- 1 root root 13287 Jan 13  2021 values.schema.yaml
-rw-r--r-- 1 root root 21428 Dec  8 11:03 values.yaml



[root@k8s-master timescaledb-single]# chmod +x generate_kustomization.sh 


./generate_kustomization.sh tsdb
[root@k8s-master timescaledb-single]# ./generate_kustomization.sh tsdb
Generating a 4096 bit RSA private key
..........................++
.......................................................................................................................++
writing new private key to './kustomize/tsdb/tls.key'
-----
Do you want to configure the backup of your database to S3 (compatible) storage? (y/n)
n
./generate_kustomization.sh: line 62: read: `-r': not a valid identifier

Generated a kustomization named tsdb in directory ./kustomize/tsdb.


WARNING: The generated certificate in this directory is self-signed and is only
         fit for development and demonstration purposes.
         The certificate should be replaced by a signed certificate, signed by
         a Certificate Authority (CA) that you trust.


You may now wish to (p)review the files that have been created and further edit
them before deployment.


To preview the deployment of the secrets:

    kubectl kustomize "./kustomize/tsdb"

Or you may want to install the secrets directly? (y/n)
y
Installing secrets...
secret/tsdb-certificate created
secret/tsdb-credentials created
secret/tsdb-pgbackrest created


# 查看
[root@k8s-master timescaledb-single]# cd kustomize/
[root@k8s-master kustomize]# ll
total 0
drwxr-xr-x 2 root root 109 Dec  8 11:00 example
drwxr-xr-x 2 root root 109 Dec  8 10:07 example2
drwxr-xr-x 2 root root 109 Dec  8 11:12 tsdb
[root@k8s-master kustomize]# cd tsdb/
[root@k8s-master tsdb]# ll
total 16
-rw-r--r-- 1 root root  178 Dec  8 11:12 credentials.conf
-rw-r--r-- 1 root root  564 Dec  8 11:12 kustomization.yaml
-rw-r--r-- 1 root root    0 Dec  8 11:12 pgbackrest.conf
-rw-r--r-- 1 root root 1773 Dec  8 11:12 tls.crt
-rw-r--r-- 1 root root 3272 Dec  8 11:12 tls.key


[root@k8s-master tsdb]# kubectl kustomize ./

apiVersion: v1
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUU4VENDQXRtZ0F3SUJBZ0lKQUlZakc3UGdKUnB4TUEwR0NTcUdTSWIzRFFFQkN3VUFNQTh4RFRBTEJnTlYKQkFNTUJIUnpaR0l3SGhjTk1qRXhNakE0TURNeE1qUTRXaGNOTWpFeE1qQTVNRE14TWpRNFdqQVBNUTB3Q3dZRApWUVFEREFSMGMyUmlNSUlDSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQWc4QU1JSUNDZ0tDQWdFQXZwd1JoZzM3CkNKMmJiemJxL0c0NVdiQ3FndkRPeHNDQVJEY2h4cWw4RmMxd1BQK0ZGcHQrK2k0eHdNMmlGUEdFMWttNXJZRXEKcnVGZHptZFlvSjBJLzJxaW5NaUVNdUhYUE1Cb21SSldrbHF3TXU2ZnZqU2F6cWhDRS9DeHpTZjlRQ3Q5RjVjYQozeW0wc1hva1RDNjBndDVCcUtxOGx3STJwWHM1MUUrM3R0NjY5bVUrc1EvTGYwMDM1ZytHRDhnRDZBVnlSaTdGCk5kZTdaSy9hWHFHb0oyL2ZyemVqUVZTVndaS0JNMHhaeG5malBNelMzU2M5dUM4V0puU3VJQkFaejZxcWpTdkUKdUhFWC9EK2IrM3JURzJ4cy8zVU1hanl0N0QwdzZmeGZYcU9Qc2VqZXVnbXhTWjZiV1drNDIzVW9lUStQNTR3VApSSFBsTDhVUGk0RmZianpOYWIwN2tWYnIvL2ljdERhSlROSWk2SGlwd0wvQjBNSnRYRUFsckZmWEF1azZLRGtJCkFXaXI3cUc4Sjk5S01pZHA2RE1ad091THhvNzIxRnJkS3l4NzRXNm1sS0dlUWxpRHNvNENLbFJ5UW9iNktTbDgKbmVQZVBiaEwyWDk4SHRRRjN5eTMyc0U3eTI3d1JMY25HeUJ2V2NnZ2p6cjY2eTZjVndpWHhIQXZMR2tvNUdwYQo3akhrZStkQ0JKRGprRURySWJKUkRhZmR4bkR3bks2RlpJY3VxT0c2Wm5Qd0wvMUx1MWZsZFJqdHNpUnI4NVRkCm1zTE9BeERqVGE5eS90Y1NMVE9qVWxJdHNzUVZuMmtWV0cwVGRBTTlLTHR3Skk1M2hWV3BsTGNtaTBySHVyRk8KZ2NnM2RVVTJ2NWs3NHlzUjBKMXd5dUxlaGRCeHNhOGdEQmNDQXdFQUFhTlFNRTR3SFFZRFZSME9CQllFRkplYQpWWmMvTWZ6NUY2WTdMQjAzNlpsNlRVaUxNQjhHQTFVZEl3UVlNQmFBRkplYVZaYy9NZno1RjZZN0xCMDM2Wmw2ClRVaUxNQXdHQTFVZEV3UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dJQkFJeGhpcitOVlFSZ0VrRDUKdGM2bDdoSW80VFhsTXdRSzNwbHgyMXB2SjZPQklURGphNUx5QzdkN2lIVEFBV0lmVjRyeS9kQld3RWY5RjZOTwpvRlFOSDlEOTVUQ3JUbUpKSVMxMDdhZWdKQVgrdjhIRmtWWUJNeUFXTG44WVFnakRydUwreHhGZmVYMUk1TDJzCnF6c05UbHp5Zjc2b3NZK2VqVVp2RXNuVU1QYjByMVNmcUdzbGc2MFlvemo1VHErMGZaaEZGMW9DWnYyYkRuN2gKbW8yVEdpRGEzM21KYjM5NGFsQllRdkZwbUFGNWswSS80UTlRMTNRbUw2R0tBejZ5MnVabEFCcFFiVzZVS1RmbwovUHZYYjBKbmlVTkx6enQ0K3g2MTFReDd4RG1Hcnl1MDRNc0o5c2kza3ZoS3Y1NE5xVGoxcWlMZ3pmUVJBakROClFUY0JGN1M5eFFyNUZ2TDFZcWdxZkhMZEx0MGlIQS85S0owYmtXanBxNVp2RDZZOHZtRFpZbVFUYmR6NWNEWW8KeHE3MWhEV3c5QXY5YUtVaFppL0Q0T0U1WG9ScFpibC8zV0xUWWVaR2FLdWdhdjVMU0MyM0dNMmk0a2UvWDVJWAp4clFlOUxZL1I4L0crWDA1NVhBWCtldWs3eW9vUnZwVzlFZm02WW8waVhIU1RyUnFZOHFnbUFESUNOdEw1YzYrCmY2cmhJYUlia2dvTVFrdUppeC8vNzFqcHdoYmhPYXNPelBEbDF5ZG1zeVFKSldvcmpTR2xHWlNxQWxtOHZGdkIKaklLbkw1Ni92UFpTWDIzdDhYM1V0WXhrRWZwTXBYSlFqZSs3L3NXa1hlQnlNNUhDMm14Y0VpTERid2ZEYU1jSgpBd2xxL2hBTklpditBS3BIaW5NcHVub2tveWdHCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUpSQUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQ1M0d2dna3FBZ0VBQW9JQ0FRQytuQkdHRGZzSW5adHYKTnVyOGJqbFpzS3FDOE03R3dJQkVOeUhHcVh3VnpYQTgvNFVXbTM3NkxqSEF6YUlVOFlUV1NibXRnU3F1NFYzTwpaMWlnblFqL2FxS2N5SVF5NGRjOHdHaVpFbGFTV3JBeTdwKytOSnJPcUVJVDhMSE5KLzFBSzMwWGx4cmZLYlN4CmVpUk1MclNDM2tHb3FyeVhBamFsZXpuVVQ3ZTIzcnIyWlQ2eEQ4dC9UVGZtRDRZUHlBUG9CWEpHTHNVMTE3dGsKcjlwZW9hZ25iOSt2TjZOQlZKWEJrb0V6VEZuR2QrTTh6TkxkSnoyNEx4WW1kSzRnRUJuUHFxcU5LOFM0Y1JmOApQNXY3ZXRNYmJHei9kUXhxUEszc1BURHAvRjllbzQreDZONjZDYkZKbnB0WmFUamJkU2g1RDQvbmpCTkVjK1V2CnhRK0xnVjl1UE0xcHZUdVJWdXYvK0p5ME5vbE0waUxvZUtuQXY4SFF3bTFjUUNXc1Y5Y0M2VG9vT1FnQmFLdnUKb2J3bjMwb3lKMm5vTXhuQTY0dkdqdmJVV3QwckxIdmhicWFVb1o1Q1dJT3lqZ0lxVkhKQ2h2b3BLWHlkNDk0OQp1RXZaZjN3ZTFBWGZMTGZhd1R2TGJ2QkV0eWNiSUc5WnlDQ1BPdnJyTHB4WENKZkVjQzhzYVNqa2FscnVNZVI3CjUwSUVrT09RUU9zaHNsRU5wOTNHY1BDY3JvVmtoeTZvNGJwbWMvQXYvVXU3VitWMUdPMnlKR3Z6bE4yYXdzNEQKRU9OTnIzTCsxeEl0TTZOU1VpMnl4QldmYVJWWWJSTjBBejBvdTNBa2puZUZWYW1VdHlhTFNzZTZzVTZCeURkMQpSVGEvbVR2akt4SFFuWERLNHQ2RjBIR3hyeUFNRndJREFRQUJBb0lDQVFDaEtXeFhvREtFMGwyOTV2MUFlaUhiCmg5aGo1aCt2Uk13dTRUNlpybXZRTTJlVzY2VW52RW5jVm5xU3Zrc3cwaFN5NnBzbjdIS2Vjc2JSNGVlNXhZejYKQ0x0OVBMMHFHSEhZV0FjWWhielUrZ0RJejZJWlBIazRDbVAwWUNxMWYvNU01M2haSGRZR29raTVWc0NoK1V0Kwo2MVV3dTB2QmtvbUoyV3JIN0s4MDI1WFJCMEcraTlCUHpvWlk2djg3RUs3YzJ0NElJVHQxanZaTzU3RUhHa0FICk9EdzA5aUgrOXZtNE5ac3dPSG9mcVBabFRmbHNLK1d2ZVlDTSsxTE9vVTV4bTZlZ3ZVVVRmZXY5eC9xbTR1N2oKM1FVNzZVZVorN3hDWm1xWkxGMm1zenluL0hGMWN3U0hicWVETGNoNUpkSVdxZVhPclUwTkw4QzBaellCaWwyWQphdWJGZC9pVWpiVllqejVzZG1kM2E5c1FzOG90aUFQaXE2a0JJVmQvdVV0a2srbUlXeDI0VlBhdTU0c0tnSlErCmEyUnEzZDdNa1FxamNPckJNNlBsaWpkSnI4alFEMmdiVC84cCtUcUxjUEJYUVNtQW93TVNEbFR3L1VVb3oraFMKb0hhYzErTGVLcm8wVlZLNnZraUJsdnZsaW9CS2Ixck9RUDFua01kYTNaMXJuOWhWd015RkVLTTdyWDJHMG1xdwphWXJsYVpKUXQ4cEVaUXc4TlBuV2xid1R1SVdBeVk3UERrd0FEQmpVZGtwQVFUYXMwaFMyZ3VsZG9lMG13cE9IClNxL2h5Q1hoRzhXandHV1ljelc3YnNJTHZONXFrNXU5SEZ5US9JTTZHTjJOTXFOUHI5NEFPZ2puMFVoakEzR2MKMGpmSDJrd3BkUU9md2ZoQk1pNUpvUUtDQVFFQSs4K2VzVTB2RVRlSm1JdVN0N2REQjF1YWVVSzFaZytIazJiMAprVGRURS9lMWoycTRmSGR4eS9yN3lhTEtCcTNuTnNPNXI4KzlKZDg5R2J6bmFObW9DTU8yUDdPNjJXejhEMFRhCk8xRGZNVlNXRkpja0sxN0NWdHJWY2lpTU9kQ3ZtMWZZNVVORXdmQlJEY2tJK2QvbHlyeWNFaHRhRm0vazVkdFQKRnQzRG5GSFdzUldtUXh4Z1o4bjhJcmVRSEZXN2ZJbFVNM1VDSVpZdFNNZmZQeUVZYjRJQms2dDd0VzRaYXJaZQpUVGl2Wmg3WmFncUhuR1MrNE1jejFrU25kbklOanBNc04rUkRkamJSbVRhRlhoSEk5c01NOWQ4UUNiNXpMZVVoClFYcDdQenNLbW5DVjUzNTltb0hXenJEaHJnQmVzdjRmR09lcWdJMDdBOE52UEdDNUZRS0NBUUVBd2NmUDVPUjUKN3l5SWljTU9KSzB6SGErSmdmeFJVclJjbHV0d2ZNRndIdlFzOVlMTXVVUUZPU252cDhQN0RoMTlReGU2dFVtaQpSaE13a29ablNlcTAvUGFxNjNMei9Ec09aeUpnaVJONUxERERKc2hhcVpLMytJaDdyNXhDdTllcHpIQXFzUEJ0CmdEQjd4SVh4R0ZBWkFmV25ZRnUwL01BY0JHK1hUMmZKMklua25mdGd4M1FMVEhDWVJMOXZWaTlRQnFkVUVCeVYKanYwRFhQaXJHUDFQeFp5dUtIdTRQa3dwVXczdTJMT2JjSGk4MkRZNXBaQ2FTQktGN0U3ak5RVWNQUVlrNDYvdgprTDAwMmxTY1RaWWNjU0lHZTJlUGh5NFRwdTliT05qSmVCczU4bzBPajV5dDNBWVNQenNGd3hLMmcxUU1nSjR0CkV2OVpjbFZJamNSamV3S0NBUUVBbC9OS1JKMVp5SjdsMWZwclY1Y3J5SFhiZWs3cDNhT0RZVXhnU256REVpcUoKRWZrSlNIcGZYZ0tmQzZiREdGSzZVazY3Yno4QldhZ2pXN09sUkowTEMvYmxzLzBGeEl4Q2NnaFBWRG5SNVJldwoxTTM3a254RTgxcHNNTTROQ0JwSXpZbXVKWEo1UERxQy9ybVFSQkI2dnVNZm5zR0lsRCtETmIwcW40TEV0a0NnCk9BM2pYVjN6UXM0YzZ1b3YrUmsyNE1pUjJkZENxUGZSYmJqR20zYWlJeStsT2ZIaDNiS0pmZDU4ZTBhNGVQd20Kb0Jtb2laUTFwcmd1TEo4VEdxTnFVTjI0Y2lXNUc4MnFuelRLTitDdGoxSldNTXdoQm5BNVdybUlYdFhGSjN1KwpRMEdyQTUxTnQvMmZuT3daMHdFQUpDeTZvVEd5cm80S3paT1NQVEgwblFLQ0FRRUF1WGRKWmRTN1U0djIvb3RlCnpRR3NTN3hIU3M2NDh3UkhIZmNuU1hCR3BJbWxRakczU1padHZXR0N5N29LWGw4aEZZYnZuelZqaDlnMEEvbDUKZ0VpUWd4Qi9WQ3hJa3QvZkVCelo4amhlSUVwbmJZWVRKL3VSOHVDR0tQVFE5a0lhZE4zaUxrbGZOSEt5OHN3VApqZWFUbU1tS01mSjBEZmk4bUE1SjdxanlpanFvUWdCbmgxNXN0Zk9KK1RxVUVCUG1id2ZWc0RuN2RzaDRZSkpzCkI5WGlkTXRaOE10QTh2Y29FaUxpdUN0bkdVV2wwUFpOUkVaYk02SHU1ZFkzSmZiSEtTenBQN2FTL3Avb3B4S1cKS3JnQ2J0RlhET040ZnJRK0FPZFVFdHVDTUY4Mm5nd2RwSndiMXR0RXVabi9FTTZuT3BqR0FvL2hxaTRWYVJWeApCS2F6WXdLQ0FRQThmVHpaZndxZjBUcE1Vb3pFS3BxVFBWc0xIb2FvQmJ2eGJqTk04eUViakpGWHM5cTJDVkRKCmJySzNQR3B4SjZnb2JndVB4VzQvaE95VXR0NFB6czBIWG03NVJmWkQzK3g2bE83YWpETXNXUUpZZFd0RzVOdnYKYlpoK09rM3lubHNXWDdJTEx3RzZ1TzRHYzBEbDlqZFlsWnI3czd2VFh5cjNmVzU0Q2JlSGFSVzJRMW92Q2RDNApIL3l0MDNkSW9EVHRvN01RZDFJK3JOQUYzTnYzcjM4MVRjTCtlWkd3Zld5SFR2ODVtbzdFVlU1QjFIeldiRGY5CnBDSTkrOUw2dUhPRG40dzc0YUZBV25nMU5ta3VDMCtKZHNYZ3ErS05oNmJJS1Qyb0NxUW9WWmwwN0pFT2I4V3YKTTRrc3M3Ujg4YUVxY1RuVkF6SHhLK0RPcFppQ1RheEcKLS0tLS1FTkQgUFJJVkFURSBLRVktLS0tLQo=
kind: Secret
metadata:
  labels:
    app: tsdb-timescaledb
    cluster-name: tsdb
  name: tsdb-certificate
type: kubernetes.io/tls
---
apiVersion: v1
data:
  PATRONI_REPLICATION_PASSWORD: Y0txZWFBWkxEYnlSV0ZKSUl5TDd5UXR2dWU3VERQdUY=
  PATRONI_SUPERUSER_PASSWORD: YjBPbFJjQmcxUm9PVVFaa3g4UG52elBjN2ZhN3YyQzg=
  PATRONI_admin_PASSWORD: aGpGTzJuN3pCSlB4TVNJUFVSenJmUElsbXJxRzBLeDk=
kind: Secret
metadata:
  labels:
    app: tsdb-timescaledb
    cluster-name: tsdb
  name: tsdb-credentials
type: Opaque
---
apiVersion: v1
kind: Secret
metadata:
  labels:
    app: tsdb-timescaledb
    cluster-name: tsdb
  name: tsdb-pgbackrest
type: Opaque


# 上述内容保存为文件：timescaledbMap.yaml

[root@k8s-master tsdb]# kubectl apply -f timescaledbMap.yaml
secret/tsdb-certificate unchanged
secret/tsdb-credentials unchanged
secret/tsdb-pgbackrest unchanged
```



```shell
[root@k8s-master timescale]# helm install tsdb  timescaledb-single
NAME: tsdb
LAST DEPLOYED: Wed Dec  8 11:24:45 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
TimescaleDB can be accessed via port 5432 on the following DNS name from within your cluster:
tsdb.default.svc.cluster.local

To get your password for superuser run:

    # superuser password
    PGPASSWORD_POSTGRES=$(kubectl get secret --namespace default tsdb-credentials -o jsonpath="{.data.PATRONI_SUPERUSER_PASSWORD}" | base64 --decode)

    # admin password
    PGPASSWORD_ADMIN=$(kubectl get secret --namespace default tsdb-credentials -o jsonpath="{.data.PATRONI_admin_PASSWORD}" | base64 --decode)

To connect to your database, chose one of these options:

1. Run a postgres pod and connect using the psql cli:
    # login as superuser
    kubectl run -i --tty --rm psql --image=postgres \
      --env "PGPASSWORD=$PGPASSWORD_POSTGRES" \
      --command -- psql -U postgres \
      -h tsdb.default.svc.cluster.local postgres

    # login as admin
    kubectl run -i --tty --rm psql --image=postgres \
      --env "PGPASSWORD=$PGPASSWORD_ADMIN" \
      --command -- psql -U admin \
      -h tsdb.default.svc.cluster.local postgres

2. Directly execute a psql session on the master node

   MASTERPOD="$(kubectl get pod -o name --namespace default -l release=tsdb,role=master)"
   kubectl exec -i --tty --namespace default ${MASTERPOD} -- psql -U postgres



[root@k8s-master timescale]# helm list
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART                   	APP VERSION
tsdb	default  	1       	2021-12-08 11:24:45.002796972 +0800 CST	deployed	timescaledb-single-0.8.1


[root@k8s-master timescale]# kubectl get all
NAME                     READY   STATUS    RESTARTS   AGE
pod/tsdb-timescaledb-0   1/1     Running   0          2m30s
pod/tsdb-timescaledb-1   1/1     Running   0          2m30s
pod/tsdb-timescaledb-2   1/1     Running   0          2m30s

NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/kubernetes     ClusterIP      10.96.0.1        <none>        443/TCP          112d
service/tsdb           LoadBalancer   10.99.100.99     <pending>     5432:30134/TCP   2m30s
service/tsdb-config    ClusterIP      None             <none>        8008/TCP         2m30s
service/tsdb-replica   ClusterIP      10.100.214.105   <none>        5432/TCP         2m30s

NAME                                READY   AGE
statefulset.apps/tsdb-timescaledb   3/3     2m30s


[root@k8s-master timescale]# kubectl get pvc
NAME                                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
storage-volume-tsdb-timescaledb-0   Bound    pvc-fd3d36f0-922f-462b-9d21-d40822c4aa75   20Gi       RWO            rook-ceph-block   2m37s
storage-volume-tsdb-timescaledb-1   Bound    pvc-ca97fa0b-c56e-4b90-84c4-8686ab1dd0f4   20Gi       RWO            rook-ceph-block   2m37s
storage-volume-tsdb-timescaledb-2   Bound    pvc-f8764025-5d07-4040-a14c-60a7fb621ce9   20Gi       RWO            rook-ceph-block   2m37s
wal-volume-tsdb-timescaledb-0       Bound    pvc-0277a264-13d7-43e9-93d5-2c0c3c062b0d   1Gi        RWO            rook-ceph-block   2m37s
wal-volume-tsdb-timescaledb-1       Bound    pvc-ee810352-cdd0-4e4d-bf6c-6eeccf4bcf6a   1Gi        RWO            rook-ceph-block   2m37s
wal-volume-tsdb-timescaledb-2       Bound    pvc-5296b42d-8fc1-417d-a7a6-427fbee632d7   1Gi        RWO            rook-ceph-block   2m37s
```

```shell
# 查看
[root@k8s-master helm]# kubectl get all
NAME                     READY   STATUS    RESTARTS   AGE
pod/my-pg-postgresql-0   1/1     Running   0          53s

NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP    111d
service/my-pg-postgresql            ClusterIP   10.110.139.14   <none>        5432/TCP   54s
service/my-pg-postgresql-headless   ClusterIP   None            <none>        5432/TCP   54s

NAME                                READY   AGE
statefulset.apps/my-pg-postgresql   1/1     54s


[root@k8s-master helm]# kubectl get pvc
NAME                      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-my-pg-postgresql-0   Bound    pvc-633f7d51-c04f-40aa-9070-fd5c6381e998   20Gi       RWO            rook-ceph-block   62s
```



#### 1.4.测试

```shell
# 测试数据库


# superuser password
export PGPASSWORD_POSTGRES=$(kubectl get secret --namespace default tsdb-credentials -o jsonpath="{.data.PATRONI_SUPERUSER_PASSWORD}" | base64 --decode)

    # admin password
export PGPASSWORD_ADMIN=$(kubectl get secret --namespace default tsdb-credentials -o jsonpath="{.data.PATRONI_admin_PASSWORD}" | base64 --decode)


# 1.获取密码（用户postgres）
[root@k8s-master ~]# export PGPASSWORD_POSTGRES=$(kubectl get secret --namespace default tsdb-credentials -o jsonpath="{.data.PATRONI_SUPERUSER_PASSWORD}" | base64 --decode)

[root@k8s-master ~]# echo $PGPASSWORD_POSTGRES
tea

# 获取密码（用户admin）
[root@k8s-master ~]# export PGPASSWORD_ADMIN=$(kubectl get secret --namespace default tsdb-credentials -o jsonpath="{.data.PATRONI_admin_PASSWORD}" | base64 --decode)

[root@k8s-master ~]# echo $PGPASSWORD_ADMIN
hjFO2n7zBJPxMSIPURzrfPIlmrqG0Kx9


# 2.数据库连接信息
tsdb.default.svc.cluster.local
5432:30134
postgres
b0OlRcBg1RoOUQZkx8PnvzPc7fa7v2C8


# 3.连接数据库
kubectl run -i --tty --rm psql --image=postgres \
--env "PGPASSWORD=$PGPASSWORD_POSTGRES" \
--command -- psql -U postgres \
-h tsdb.default.svc.cluster.local postgres


[root@k8s-master ~]# kubectl run -i --tty --rm psql --image=postgres \
> --env "PGPASSWORD=$PGPASSWORD_POSTGRES" \
> --command -- psql -U postgres \
> -h tsdb.default.svc.cluster.local postgres
Error from server (AlreadyExists): pods "psql" already exists


[root@k8s-master ~]# kubectl get pod
NAME                 READY   STATUS    RESTARTS   AGE
psql                 1/1     Running   0          19m
tsdb-timescaledb-0   1/1     Running   0          30m
tsdb-timescaledb-1   1/1     Running   0          30m
tsdb-timescaledb-2   1/1     Running   0          30m
```

```shell
# Pod
# 区分主从：role=master； role=replica；

[root@k8s-master ~]# kubectl get pod --show-labels
NAME                 READY   STATUS    RESTARTS   AGE   LABELS
# 从
tsdb-timescaledb-0   1/1     Running   0          40m   app=tsdb-timescaledb,cluster-name=tsdb,controller-revision-hash=tsdb-timescaledb-56fdb9b9cf,release=tsdb,role=replica,statefulset.kubernetes.io/pod-name=tsdb-timescaledb-0

# 从
tsdb-timescaledb-1   1/1     Running   0          40m   app=tsdb-timescaledb,cluster-name=tsdb,controller-revision-hash=tsdb-timescaledb-56fdb9b9cf,release=tsdb,role=replica,statefulset.kubernetes.io/pod-name=tsdb-timescaledb-1

# 主
tsdb-timescaledb-2   1/1     Running   0          40m   app=tsdb-timescaledb,cluster-name=tsdb,controller-revision-hash=tsdb-timescaledb-56fdb9b9cf,release=tsdb,role=master,statefulset.kubernetes.io/pod-name=tsdb-timescaledb-2
```

```shell
# 查看service

[root@k8s-master ~]# kubectl get pod -o wide
NAME                 READY   STATUS    RESTARTS   AGE   IP               NODE        NOMINATED NODE   READINESS GATES
psql                 1/1     Running   0          32m   10.244.36.121    k8s-node1   <none>           <none>
tsdb-timescaledb-0   1/1     Running   0          43m   10.244.36.80     k8s-node1   <none>           <none>
tsdb-timescaledb-1   1/1     Running   0          43m   10.244.169.133   k8s-node2   <none>           <none>
tsdb-timescaledb-2   1/1     Running   0          43m   10.244.107.208   k8s-node3   <none>           <none>


[root@k8s-master ~]# kubectl get svc
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
tsdb           LoadBalancer   10.99.100.99     <pending>     5432:30134/TCP   43m
tsdb-config    ClusterIP      None             <none>        8008/TCP         43m
tsdb-replica   ClusterIP      10.100.214.105   <none>        5432/TCP         43m


# 主
[root@k8s-master ~]# kubectl describe svc tsdb
Name:                     tsdb
Namespace:                default
Labels:                   app=tsdb-timescaledb
                          app.kubernetes.io/managed-by=Helm
                          chart=timescaledb-single-0.8.1
                          cluster-name=tsdb
                          heritage=Helm
                          release=tsdb
                          
                          
                          role=master
                          
                          
Annotations:              meta.helm.sh/release-name: tsdb
                          meta.helm.sh/release-namespace: default
                          service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: 4000
Selector:                 <none>
Type:                     LoadBalancer
IP Families:              <none>
IP:                       10.99.100.99
IPs:                      10.99.100.99
Port:                     postgresql  5432/TCP
TargetPort:               postgresql/TCP
NodePort:                 postgresql  30134/TCP
Endpoints:                10.244.107.208:5432
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>


# 从
[root@k8s-master ~]# kubectl describe svc tsdb-replica
Name:              tsdb-replica
Namespace:         default
Labels:            app=tsdb-timescaledb
                   app.kubernetes.io/managed-by=Helm
                   chart=timescaledb-single-0.8.1
                   cluster-name=tsdb
                   component=postgres
                   heritage=Helm
                   release=tsdb
                   
                   
                   role=replica
                   
                   
Annotations:       meta.helm.sh/release-name: tsdb
                   meta.helm.sh/release-namespace: default
                   service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: 4000
Selector:          app=tsdb-timescaledb,cluster-name=tsdb,role=replica
Type:              ClusterIP
IP Families:       <none>
IP:                10.100.214.105
IPs:               10.100.214.105
Port:              postgresql  5432/TCP
TargetPort:        postgresql/TCP
Endpoints:         10.244.169.133:5432,10.244.36.80:5432
Session Affinity:  None
Events:            <none>
```



#### 1.5.外部访问数据库

```shell
# 查看
[root@k8s-master ~]# kubectl get svc
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
tsdb           LoadBalancer   10.99.100.99     <pending>     5432:30134/TCP   48m
tsdb-config    ClusterIP      None             <none>        8008/TCP         48m
tsdb-replica   ClusterIP      10.100.214.105   <none>        5432/TCP         48m


# 修改从service类型
[root@k8s-master ~]# kubectl edit svc tsdb-replica
service/tsdb-replica edited

spec:
......
  type: NodePort


[root@k8s-master ~]# kubectl get svc
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
tsdb           LoadBalancer   10.99.100.99     <pending>     5432:30134/TCP   50m
tsdb-config    ClusterIP      None             <none>        8008/TCP         50m
tsdb-replica   NodePort       10.100.214.105   <none>        5432:32296/TCP   50m


# 外部访问信息
# 主
172.51.216.81
5432:30134
postgres
b0OlRcBg1RoOUQZkx8PnvzPc7fa7v2C8

# 从
172.51.216.81
5432:32296
postgres
b0OlRcBg1RoOUQZkx8PnvzPc7fa7v2C8
```

![](/images/kubernetes/middleware/timescale-1.png)

![](/images/kubernetes/middleware/timescale-2.png)



#### 1.6.删除TimescaleDB

```shell
# 查看

[root@k8s-master ~]# helm list
NAME	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART                   	APP VERSION
tsdb	default  	1       	2021-12-08 11:24:45.002796972 +0800 CST	deployed	timescaledb-single-0.8.1	           
[root@k8s-master ~]# kubectl get all
NAME                     READY   STATUS    RESTARTS   AGE
pod/psql                 1/1     Running   0          81m
pod/tsdb-timescaledb-0   1/1     Running   0          92m
pod/tsdb-timescaledb-1   1/1     Running   0          92m
pod/tsdb-timescaledb-2   1/1     Running   0          92m

NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/tsdb           LoadBalancer   10.99.100.99     <pending>     5432:30134/TCP   92m
service/tsdb-config    ClusterIP      None             <none>        8008/TCP         92m
service/tsdb-replica   NodePort       10.100.214.105   <none>        5432:32296/TCP   92m

NAME                                READY   AGE
statefulset.apps/tsdb-timescaledb   3/3     92m

[root@k8s-master ~]# kubectl get pvc
NAME                                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
storage-volume-tsdb-timescaledb-0   Bound    pvc-fd3d36f0-922f-462b-9d21-d40822c4aa75   20Gi       RWO            rook-ceph-block   92m
storage-volume-tsdb-timescaledb-1   Bound    pvc-ca97fa0b-c56e-4b90-84c4-8686ab1dd0f4   20Gi       RWO            rook-ceph-block   92m
storage-volume-tsdb-timescaledb-2   Bound    pvc-f8764025-5d07-4040-a14c-60a7fb621ce9   20Gi       RWO            rook-ceph-block   92m
wal-volume-tsdb-timescaledb-0       Bound    pvc-0277a264-13d7-43e9-93d5-2c0c3c062b0d   1Gi        RWO            rook-ceph-block   92m
wal-volume-tsdb-timescaledb-1       Bound    pvc-ee810352-cdd0-4e4d-bf6c-6eeccf4bcf6a   1Gi        RWO            rook-ceph-block   92m
wal-volume-tsdb-timescaledb-2       Bound    pvc-5296b42d-8fc1-417d-a7a6-427fbee632d7   1Gi        RWO            rook-ceph-block   92m
```

```shell
# 删除

[root@k8s-master ~]# helm delete tsdb
release "tsdb" uninstalled
           
[root@k8s-master ~]# kubectl delete pvc storage-volume-tsdb-timescaledb-0
persistentvolumeclaim "storage-volume-tsdb-timescaledb-0" deleted
[root@k8s-master ~]# kubectl delete pvc storage-volume-tsdb-timescaledb-1
persistentvolumeclaim "storage-volume-tsdb-timescaledb-1" deleted
[root@k8s-master ~]# kubectl delete pvc storage-volume-tsdb-timescaledb-2
persistentvolumeclaim "storage-volume-tsdb-timescaledb-2" deleted
[root@k8s-master ~]# kubectl delete pvc wal-volume-tsdb-timescaledb-0
persistentvolumeclaim "wal-volume-tsdb-timescaledb-0" deleted
[root@k8s-master ~]# kubectl delete pvc wal-volume-tsdb-timescaledb-1
persistentvolumeclaim "wal-volume-tsdb-timescaledb-1" deleted
[root@k8s-master ~]# kubectl delete pvc wal-volume-tsdb-timescaledb-2
persistentvolumeclaim "wal-volume-tsdb-timescaledb-2" deleted

[root@k8s-master ~]# kubectl delete pod psql
pod "psql" deleted

[root@k8s-master ~]# kubectl delete svc tsdb-config
service "tsdb-config" deleted


# 查看
[root@k8s-master ~]# helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
[root@k8s-master ~]# kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   113d
```



### 2.部署多节点TimescaleDB

```shell
# 部署多节点

[root@k8s-master ~]# helm search repo timescaledb
NAME                           	CHART VERSION	APP VERSION	DESCRIPTION                      
timescale/timescaledb-multinode	0.8.0        	           	TimescaleDB Multinode Deployment.


# TimescaleDB Multinode
https://github.com/timescale/timescaledb-kubernetes/blob/master/charts/timescaledb-multinode
```



#### 2.1.下载安装包

```shell
[root@k8s-master multinode]# helm fetch timescale/timescaledb-multinode
[root@k8s-master multinode]# ll
total 12
-rw-r--r-- 1 root root 10093 Dec  8 13:19 timescaledb-multinode-0.8.0.tgz
[root@k8s-master multinode]# tar -zxf timescaledb-multinode-0.8.0.tgz 
[root@k8s-master multinode]# ll
total 12
drwxr-xr-x 3 root root   143 Dec  8 13:19 timescaledb-multinode
-rw-r--r-- 1 root root 10093 Dec  8 13:19 timescaledb-multinode-0.8.0.tgz
[root@k8s-master multinode]# cd timescaledb-multinode
[root@k8s-master timescaledb-multinode]# ll
total 28
-rw-r--r-- 1 root root 7579 Jan 13  2021 admin-guide.md
-rw-r--r-- 1 root root  380 Jan 13  2021 Chart.yaml
-rw-r--r-- 1 root root 4989 Jan 13  2021 README.md
-rw-r--r-- 1 root root  188 Jan 13  2021 requirements.yaml
drwxr-xr-x 2 root root  335 Dec  8 13:19 templates
-rw-r--r-- 1 root root 3891 Jan 13  2021 values.yaml
```



#### 2.2.修改配置

```shell
# values.yaml


persistentVolume:
  enabled: true
  size: 20G
  ## database data Persistent Volume Storage Class
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  storageClass: "rook-ceph-block"
  subPath: ""
  mountPath: "/var/lib/postgresql"
  annotations: {}
  accessModes:
    - ReadWriteOnce


# 修改存储
# 修改内容
storageClass: "rook-ceph-block"
size: 20Gi
```



#### 2.3.安装

```shell
[root@k8s-master multinode]# helm install tsdb-mn timescaledb-multinode
NAME: tsdb-mn
LAST DEPLOYED: Wed Dec  8 14:57:46 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
# This file and its contents are licensed under the Apache License 2.0.
# Please see the included NOTICE for copyright information and LICENSE for a copy of the license.

TimescaleDB can be accessed via port 5432 on the following DNS name from within your cluster:
tsdb-mn-timescaledb.default.svc.cluster.local

To get your password for superuser run:

    # superuser password
    PGPASSWORD_SUPERUSER=$(kubectl get secret --namespace default tsdb-mn-timescaledb -o jsonpath="{.data.password-superuser}" | base64 --decode)

    # admin password
    PGPASSWORD_ADMIN=$(kubectl get secret --namespace default tsdb-mn-timescaledb -o jsonpath="{.data.password-admin}" | base64 --decode)

To connect to your database:

1. Run a postgres pod and connect using the psql cli:
    # login as superuser
    kubectl run -i --tty --rm psql --image=postgres \
      --env "PGPASSWORD=$PGPASSWORD_SUPERUSER" \
      --command -- psql -U postgres \
      -h tsdb-mn-timescaledb.default.svc.cluster.local postgres

    # login as admin
    kubectl run -i -tty --rm psql --image=postgres \
      --env "PGPASSWORD=$PGPASSWORD_ADMIN" \
      --command -- psql -U admin \
      -h tsdb-mn-timescaledb.default.svc.cluster.local postgres



# 查看
[root@k8s-master multinode]# helm list
NAME   	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART                      	APP VERSION
tsdb-mn	default  	1       	2021-12-08 13:57:07.141168114 +0800 CST	deployed	timescaledb-multinode-0.8.0	 

[root@k8s-master multinode]# kubectl get all
NAME                                   READY   STATUS                  RESTARTS   AGE
pod/attachdn-tsdb-mn-db0-data0-cpbhx   1/1     Running                 0          19m
pod/attachdn-tsdb-mn-db0-data1-hnvdn   1/1     Running                 0          19m
pod/attachdn-tsdb-mn-db0-data2-567l2   1/1     Running                 0          19m
pod/attachdn-tsdb-mn-db1-data0-zfvbf   1/1     Running                 0          19m
pod/attachdn-tsdb-mn-db1-data1-mbrwz   1/1     Running                 0          19m
pod/attachdn-tsdb-mn-db1-data2-hxpsl   1/1     Running                 0          19m
pod/createdb-tsdb-mn-db0-8s4q9         1/1     Running                 0          19m
pod/createdb-tsdb-mn-db1-6cf2g         1/1     Running                 0          19m
pod/tsdb-mn-timescaledb-access-0       0/1     Init:ImagePullBackOff   0          19m
pod/tsdb-mn-timescaledb-data-0         0/1     Init:ImagePullBackOff   0          19m
pod/tsdb-mn-timescaledb-data-1         0/1     Init:ImagePullBackOff   0          19m
pod/tsdb-mn-timescaledb-data-2         0/1     Init:ImagePullBackOff   0          19m

NAME                               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/kubernetes                 ClusterIP      10.96.0.1        <none>        443/TCP          113d
service/tsdb-mn-timescaledb        LoadBalancer   10.103.224.166   <pending>     5432:31823/TCP   19m
service/tsdb-mn-timescaledb-data   ClusterIP      None             <none>        5432/TCP         19m

NAME                                          READY   AGE
statefulset.apps/tsdb-mn-timescaledb-access   0/1     19m
statefulset.apps/tsdb-mn-timescaledb-data     0/3     19m

NAME                                   COMPLETIONS   DURATION   AGE
job.batch/attachdn-tsdb-mn-db0-data0   0/1           19m        19m
job.batch/attachdn-tsdb-mn-db0-data1   0/1           19m        19m
job.batch/attachdn-tsdb-mn-db0-data2   0/1           19m        19m
job.batch/attachdn-tsdb-mn-db1-data0   0/1           19m        19m
job.batch/attachdn-tsdb-mn-db1-data1   0/1           19m        19m
job.batch/attachdn-tsdb-mn-db1-data2   0/1           19m        19m
job.batch/createdb-tsdb-mn-db0         0/1           19m        19m
job.batch/createdb-tsdb-mn-db1         0/1           19m        19m

[root@k8s-master multinode]# kubectl get pvc
NAME                                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
storage-volume-tsdb-mn-timescaledb-access-0   Bound    pvc-58eb560c-4bf6-4a81-b61d-90bc1940e787   19Gi       RWO            rook-ceph-block   19m
storage-volume-tsdb-mn-timescaledb-data-0     Bound    pvc-4347c24f-6549-46ba-a891-410ccc86523f   19Gi       RWO            rook-ceph-block   19m
storage-volume-tsdb-mn-timescaledb-data-1     Bound    pvc-6127b102-4df8-47c3-8f75-6581e7af1421   19Gi       RWO            rook-ceph-block   19m
storage-volume-tsdb-mn-timescaledb-data-2     Bound    pvc-72392eca-dedd-4a19-bbac-44297aa4e3bb   19Gi       RWO            rook-ceph-block   19m
```

```shell
# docker pull  timescaledev/timescaledb-ha:pg12-ts2.0.0-p0
# 镜像无法下载


# 问题解决
# 暂时用 hub.docker.com解决
# https://hub.docker.com/r/timescale/timescaledb-ha/tags
# docker pull timescale/timescaledb-ha:pg12-ts2.0.0-rc4


# 下载镜像
[root@k8s-node1 ~]# docker pull timescale/timescaledb-ha:pg12-ts2.0.0-rc4
pg12-ts2.0.0-rc4: Pulling from timescale/timescaledb-ha
e9d0e04f5910: Pull complete 
7a4bf858cf94: Pull complete 
971a7ee38608: Pull complete 
9300ac60165d: Pull complete 
a7f8436fd92b: Pull complete 
e22397ba29ae: Pull complete 
3df9900d6305: Pull complete 
ce917a6cd990: Pull complete 
e93dd3167788: Pull complete 
1d7e9a6929a1: Pull complete 
4f4fb700ef54: Pull complete 
fc2f0530307a: Pull complete 
cc7920a00e4f: Pull complete 
a572713db581: Pull complete 
9ce06a860e69: Pull complete 
369621055520: Pull complete 
Digest: sha256:f7c65851e360ef8ec71e332ca3d7d03f00f63236118682a3229a6271577794b9
Status: Downloaded newer image for timescale/timescaledb-ha:pg12-ts2.0.0-rc4
docker.io/timescale/timescaledb-ha:pg12-ts2.0.0-rc4


# 打标签
docker tag timescale/timescaledb-ha:pg12-ts2.0.0-rc4 timescaledev/timescaledb-ha:pg12-ts2.0.0-p0


# 查看
[root@k8s-master multinode]# helm list
NAME   	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART                      	APP VERSION
tsdb-mn	default  	1       	2021-12-08 14:37:43.991102838 +0800 CST	deployed	timescaledb-multinode-0.8.0	           
[root@k8s-master multinode]# kubectl get all
NAME                                   READY   STATUS      RESTARTS   AGE
pod/attachdn-tsdb-mn-db0-data0-9hkxv   0/1     Completed   0          9m53s
pod/attachdn-tsdb-mn-db0-data1-nzsk2   0/1     Completed   0          9m53s
pod/attachdn-tsdb-mn-db0-data2-ghnjx   0/1     Completed   1          9m53s
pod/attachdn-tsdb-mn-db1-data0-6vkv7   0/1     Completed   0          9m53s
pod/attachdn-tsdb-mn-db1-data1-s2xns   0/1     Completed   0          9m53s
pod/attachdn-tsdb-mn-db1-data2-s8lcx   0/1     Completed   0          9m53s
pod/createdb-tsdb-mn-db0-x4d55         0/1     Completed   0          9m53s
pod/createdb-tsdb-mn-db1-ttrvn         0/1     Completed   0          9m53s
pod/tsdb-mn-timescaledb-access-0       1/1     Running     0          9m53s
pod/tsdb-mn-timescaledb-data-0         1/1     Running     0          9m53s
pod/tsdb-mn-timescaledb-data-1         1/1     Running     0          9m53s
pod/tsdb-mn-timescaledb-data-2         1/1     Running     0          9m53s

NAME                               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes                 ClusterIP      10.96.0.1       <none>        443/TCP          113d
service/tsdb-mn-timescaledb        LoadBalancer   10.102.211.33   <pending>     5432:31257/TCP   9m53s
service/tsdb-mn-timescaledb-data   ClusterIP      None            <none>        5432/TCP         9m53s

NAME                                          READY   AGE
statefulset.apps/tsdb-mn-timescaledb-access   1/1     9m53s
statefulset.apps/tsdb-mn-timescaledb-data     3/3     9m53s

NAME                                   COMPLETIONS   DURATION   AGE
job.batch/attachdn-tsdb-mn-db0-data0   1/1           34s        9m53s
job.batch/attachdn-tsdb-mn-db0-data1   1/1           33s        9m53s
job.batch/attachdn-tsdb-mn-db0-data2   1/1           36s        9m53s
job.batch/attachdn-tsdb-mn-db1-data0   1/1           35s        9m53s
job.batch/attachdn-tsdb-mn-db1-data1   1/1           34s        9m53s
job.batch/attachdn-tsdb-mn-db1-data2   1/1           36s        9m53s
job.batch/createdb-tsdb-mn-db0         1/1           15s        9m53s
job.batch/createdb-tsdb-mn-db1         1/1           16s        9m53s
[root@k8s-master multinode]# kubectl get pvc
NAME                                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
storage-volume-tsdb-mn-timescaledb-access-0   Bound    pvc-019e9b62-a946-445f-8b69-82a32d3565d6   19Gi       RWO            rook-ceph-block   10m
storage-volume-tsdb-mn-timescaledb-data-0     Bound    pvc-bc8eddf7-6d53-4c38-91c6-f99f987d3ed7   19Gi       RWO            rook-ceph-block   10m
storage-volume-tsdb-mn-timescaledb-data-1     Bound    pvc-ed395dae-edd7-4972-8c65-19d391dbc1d1   19Gi       RWO            rook-ceph-block   10m
storage-volume-tsdb-mn-timescaledb-data-2     Bound    pvc-66f3137a-5504-4c03-b09f-2bd6ca063187   19Gi       RWO            rook-ceph-block   10m
```



#### 2.4.测试

```shell
# superuser password
export PGPASSWORD_SUPERUSER=$(kubectl get secret --namespace default tsdb-mn-timescaledb -o jsonpath="{.data.password-superuser}" | base64 --decode)

# admin password
export PGPASSWORD_ADMIN=$(kubectl get secret --namespace default tsdb-mn-timescaledb -o jsonpath="{.data.password-admin}" | base64 --decode)


# 1.获取密码
[root@k8s-master ~]# export PGPASSWORD_SUPERUSER=$(kubectl get secret --namespace default tsdb-mn-timescaledb-access -o jsonpath="{.data.password-superuser}" | base64 --decode)

[root@k8s-master ~]# echo $PGPASSWORD_SUPERUSER
tea

[root@k8s-master ~]# export PGPASSWORD_ADMIN=$(kubectl get secret --namespace default tsdb-mn-timescaledb-access -o jsonpath="{.data.password-admin}" | base64 --decode)
[root@k8s-master ~]# echo $REPMGR_PASSWORD


# 2.数据库连接信息
tsdb-mn-timescaledb.default.svc.cluster.local
172.51.216.81
5432:31875
postgres
tea


# 链接数据库
kubectl run -i --tty --rm psql --image=postgres \
--env "PGPASSWORD=$PGPASSWORD_SUPERUSER" \
--command -- psql -U postgres \
-h tsdb-mn-timescaledb.default.svc.cluster.local postgres


[root@k8s-master ~]# kubectl run -i --tty --rm psql --image=postgres \
> --env "PGPASSWORD=$PGPASSWORD_SUPERUSER" \
> --command -- psql -U postgres \
> -h tsdb-mn-timescaledb.default.svc.cluster.local postgres
If you don't see a command prompt, try pressing enter.
postgres=# \d
Did not find any relations.
postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

postgres=# \l
                              List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   
-----------+----------+----------+---------+---------+-----------------------
 example   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
(4 rows)

# 修改密码
postgres=# ALTER USER postgres with encrypted password 'postgres';
ALTER ROLE
postgres=# 
postgres=# \q
Session ended, resume using 'kubectl attach psql -c psql -i -t' command when the pod is running
pod "psql" deleted
```



#### 2.5.外部访问数据库

```shell
# 查看
[root@k8s-master ~]# kubectl get all
NAME                                   READY   STATUS      RESTARTS   AGE
pod/attachdn-tsdb-mn-db0-data1-lqz8v   0/1     Completed   0          17m
pod/attachdn-tsdb-mn-db0-data2-n9j7d   0/1     Completed   1          17m
pod/attachdn-tsdb-mn-db1-data0-qjd9q   0/1     Completed   1          17m
pod/attachdn-tsdb-mn-db1-data1-vclzk   0/1     Completed   1          17m
pod/createdb-tsdb-mn-db0-rs24k         0/1     Completed   0          17m
pod/createdb-tsdb-mn-db1-lkff4         0/1     Completed   0          17m
pod/tsdb-mn-timescaledb-access-0       1/1     Running     0          17m
pod/tsdb-mn-timescaledb-data-0         1/1     Running     0          17m
pod/tsdb-mn-timescaledb-data-1         1/1     Running     0          17m
pod/tsdb-mn-timescaledb-data-2         1/1     Running     0          17m

NAME                               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/kubernetes                 ClusterIP      10.96.0.1      <none>        443/TCP          113d
service/tsdb-mn-timescaledb        LoadBalancer   10.106.29.78   <pending>     5432:31875/TCP   17m
service/tsdb-mn-timescaledb-data   ClusterIP      None           <none>        5432/TCP         17m

NAME                                          READY   AGE
statefulset.apps/tsdb-mn-timescaledb-access   1/1     17m
statefulset.apps/tsdb-mn-timescaledb-data     3/3     17m

NAME                                   COMPLETIONS   DURATION   AGE
job.batch/attachdn-tsdb-mn-db0-data0   0/1           17m        17m
job.batch/attachdn-tsdb-mn-db0-data1   1/1           35s        17m
job.batch/attachdn-tsdb-mn-db0-data2   1/1           36s        17m
job.batch/attachdn-tsdb-mn-db1-data0   1/1           36s        17m
job.batch/attachdn-tsdb-mn-db1-data1   1/1           37s        17m
job.batch/attachdn-tsdb-mn-db1-data2   0/1           17m        17m
job.batch/createdb-tsdb-mn-db0         1/1           20s        17m
job.batch/createdb-tsdb-mn-db1         1/1           19s        17m


# service
[root@k8s-master ~]# kubectl get svc
NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
tsdb-mn-timescaledb        LoadBalancer   10.106.29.78   <pending>     5432:31875/TCP   24m
tsdb-mn-timescaledb-data   ClusterIP      None           <none>        5432/TCP         24m


# 外部访问信息
172.51.216.81
5432:31875
postgres
tea
```

![](/images/kubernetes/middleware/timescale-3.png)

![](/images/kubernetes/middleware/timescale-4.png)

![](/images/kubernetes/middleware/timescale-5.png)



#### 2.6.删除TimescaleDB

```shell
# 查看

[root@k8s-master ~]# helm list
NAME   	NAMESPACE	REVISION	UPDATED                               	STATUS  	CHART                      	APP VERSION
tsdb-mn	default  	1       	2021-12-08 14:57:46.33942306 +0800 CST	deployed	timescaledb-multinode-0.8.0	           
[root@k8s-master ~]# kubectl get all
NAME                                   READY   STATUS      RESTARTS   AGE
pod/attachdn-tsdb-mn-db0-data1-lqz8v   0/1     Completed   0          26m
pod/attachdn-tsdb-mn-db0-data2-n9j7d   0/1     Completed   1          26m
pod/attachdn-tsdb-mn-db1-data0-qjd9q   0/1     Completed   1          26m
pod/attachdn-tsdb-mn-db1-data1-vclzk   0/1     Completed   1          26m
pod/createdb-tsdb-mn-db0-rs24k         0/1     Completed   0          26m
pod/createdb-tsdb-mn-db1-lkff4         0/1     Completed   0          26m
pod/tsdb-mn-timescaledb-access-0       1/1     Running     0          26m
pod/tsdb-mn-timescaledb-data-0         1/1     Running     0          26m
pod/tsdb-mn-timescaledb-data-1         1/1     Running     0          26m
pod/tsdb-mn-timescaledb-data-2         1/1     Running     0          26m

NAME                               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/kubernetes                 ClusterIP      10.96.0.1      <none>        443/TCP          113d
service/tsdb-mn-timescaledb        LoadBalancer   10.106.29.78   <pending>     5432:31875/TCP   26m
service/tsdb-mn-timescaledb-data   ClusterIP      None           <none>        5432/TCP         26m

NAME                                          READY   AGE
statefulset.apps/tsdb-mn-timescaledb-access   1/1     26m
statefulset.apps/tsdb-mn-timescaledb-data     3/3     26m

NAME                                   COMPLETIONS   DURATION   AGE
job.batch/attachdn-tsdb-mn-db0-data0   0/1           26m        26m
job.batch/attachdn-tsdb-mn-db0-data1   1/1           35s        26m
job.batch/attachdn-tsdb-mn-db0-data2   1/1           36s        26m
job.batch/attachdn-tsdb-mn-db1-data0   1/1           36s        26m
job.batch/attachdn-tsdb-mn-db1-data1   1/1           37s        26m
job.batch/attachdn-tsdb-mn-db1-data2   0/1           26m        26m
job.batch/createdb-tsdb-mn-db0         1/1           20s        26m
job.batch/createdb-tsdb-mn-db1         1/1           19s        26m
[root@k8s-master ~]# 
[root@k8s-master ~]# 
[root@k8s-master ~]# kubectl get pvc
NAME                                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
storage-volume-tsdb-mn-timescaledb-access-0   Bound    pvc-d6b69f20-6a53-45ea-95ee-8e91eb3d3739   19Gi       RWO            rook-ceph-block   26m
storage-volume-tsdb-mn-timescaledb-data-0     Bound    pvc-bc248305-6f43-41a9-8a5d-a7016097594c   19Gi       RWO            rook-ceph-block   26m
storage-volume-tsdb-mn-timescaledb-data-1     Bound    pvc-3087ec5e-44fc-490b-a936-91c3221e5693   19Gi       RWO            rook-ceph-block   26m
storage-volume-tsdb-mn-timescaledb-data-2     Bound    pvc-8ec27b90-53bd-48ba-bd74-e1d41266eb93   19Gi       RWO            rook-ceph-block   26m
```

```shell
# 删除
[root@k8s-master ~]# helm delete tsdb-mn
release "tsdb-mn" uninstalled


[root@k8s-master ~]# kubectl delete pvc storage-volume-tsdb-mn-timescaledb-data-0
persistentvolumeclaim "storage-volume-tsdb-mn-timescaledb-data-0" deleted
[root@k8s-master ~]# kubectl delete pvc storage-volume-tsdb-mn-timescaledb-data-1
persistentvolumeclaim "storage-volume-tsdb-mn-timescaledb-data-1" deleted
[root@k8s-master ~]# kubectl delete pvc storage-volume-tsdb-mn-timescaledb-data-2
persistentvolumeclaim "storage-volume-tsdb-mn-timescaledb-data-2" deleted
[root@k8s-master ~]# kubectl delete pvc storage-volume-tsdb-mn-timescaledb-access-0
persistentvolumeclaim "storage-volume-tsdb-mn-timescaledb-access-0" deleted


[root@k8s-master ~]# helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
[root@k8s-master ~]# kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   113d
```



