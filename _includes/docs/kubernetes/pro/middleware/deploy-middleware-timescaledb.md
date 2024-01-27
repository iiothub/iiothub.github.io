* TOC
{:toc}


### 1.安装方式

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



```shell
# 部署单实例

[root@k8s-master timescale]# helm search repo timescaledb
NAME                           	CHART VERSION	APP VERSION	DESCRIPTION                      
timescale/timescaledb-single   	0.8.2        	           	TimescaleDB HA Deployment. 



# TimescaleDB Single
https://github.com/timescale/timescaledb-kubernetes/tree/master/charts/timescaledb-single
```



### 2.下载安装包

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



### 3.修改配置

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



### 4.安装

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
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUU4VENDQXRtZ0F3SUJBZ0lKQU5KOUs5TUZ1aUlHTUEwR0NTcUdTSWIzRFFFQkN3VUFNQTh4RFRBTEJnTlYKQkFNTUJIUnpaR0l3SGhjTk1qRXhNakl5TURjeU5qUTFXaGNOTWpFeE1qSXpNRGN5TmpRMVdqQVBNUTB3Q3dZRApWUVFEREFSMGMyUmlNSUlDSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQWc4QU1JSUNDZ0tDQWdFQXlHSE1ncDFXCjNPMUNsMlJzeWRsa0haT2ltd1VVdXJZNkVHQUxrTnhpZDIwNEJNTDExNU94NHhOOTVEbnVOU0tnWE1RTXh6T2wKSSthSktsNjFuWHpaMythMDdGc0pQc1gzY3RhMllUNWpNME81VDFZSjJXT083eUJFSjY3YXhOS0QrRVZFV3hpUApIQTl5VVJja2VTc09abjJXbktBVjdBdFB5NkptZkVMdmNqcGdRaW53b0ZlbmM0SHgra0t6b1NBRUVLRktzRzFPCkxBMVlPdGQ1eC9rbXllOE9rWU5iNXFBeW9WV1N1c0Fnb3pnclpMcVdnT0VzOXY3dkdoL2xOWDh6ZjJSVXA0bUcKVzNZdkx3T3UzYWV6WElJSjBTR2ZmMEVUWVJQREg0a1h4UWRVTmhxanZtdUFtVks4cVNpZWI1bmlvYnNXNCsxawpWemF1bHg5RjZCQU9UZC80MjZ6ejQzSWkwTkZDU1ozYmZmNXg5Rk5qYW5wREw1bW5RNHJrNkpBNmJ3WnZHajZQCkdKK0dVQjJLb2F5cEo0OTBkQ3B4UzNVS3poYlE0ZlJMRE9MSS9ZZzM1Mm42K3BOcDZuTHZYay9pNVM0K0tRa0kKeHE3dGF5NDR6ZVVpMHQveml5dmlBQUJSMW52b0NpZzVScnlhOEJwYzFaOXFIMkZuTTAzcW4zSDNNdDVSZU81awpncUFKWHEzUTlHeTJnY2prUmVoVEFlRXZTcmE4dUt6cWQ3QnlVUHEzZlh4WkNSU0VRRDhvNGp2QVk5Q1BnZHRQClRIK0YwL2NhaUpDejVLcGRxV0JjTU1zTm1wQyt6UzRSUnVWUk43a2FRUHZyNElNelgxZHVOL2FkUkhudjBkdlUKQktpYUxuRjJBdGx6RGxBK25rWG54VXNOTzh2b1dwTmt1MU1DQXdFQUFhTlFNRTR3SFFZRFZSME9CQllFRk82SAo1dEFrbGtmNUE5eEwvOHEvZjdIclRJNllNQjhHQTFVZEl3UVlNQmFBRk82SDV0QWtsa2Y1QTl4TC84cS9mN0hyClRJNllNQXdHQTFVZEV3UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dJQkFLVGJUSG5wUXkzZ2lJL1UKbXh2Kzg5c0FuVkNFMC92cHozWDhaNzZBZjQyT2FKYU5YNy9ZSUNtVGlCaWI0dTg1ZisyUU9vbGVjNTJhNkEwQgozcGh3cU1MeGVyYmRwUUxYK045WFNGUDRKZ3N6UGlMM3I2Q2tNQ3M5ZWdzM0FlaFY3alBydlJ1K0R6ZWhzK1hyCk03WmczYTU0MnkzdkdNSGNIZERLM2JHQTIxZUZJeVpiRmtGNnoreFVKNk85UWcwNTJ1OEozQkRkUXJjSHdWeTkKNE11aFVZQS9PTHhSK0Jqbm5BM2dQTXRhbGtMQ2xMSEN5cjBQdDYwc0xJUGs1S2pOQVZndzFWUzJoYVduVmhMLwpsNnFTQThLNGVnWjF6RFhiUTFRTThha0dXNEVOVnhTRkxOelQ2T3FLMVFXRzZmYU9sdE9ZMnlrOUZFRWx3YzhWClBCY2g3QURQUVd6UExKNUhzMVNKbVZ4SVViNGJNK0lWUWdOWVR6Z0dOSE1mVHZ4MmxsdFJBaG1zUGJkTjcvdkUKMkx2Q3FZWEVrd1FCd0tZQXc1YXRLSk9USGlETkZYMWd2bWErNERhVUwxSzZ2YlAyaDgwYy8vRWJNSkY1Si9hWAp5K01FaXgya210VFhGVkdxUjhhSWlxMUUyRmNwVXFrYmg0bEcvMlJLclJJUms5SFpZWHFncStXUnQ4d1ZxTFRyCnNZKzAyUENPR0ROZDh6ZTB2VjN4RlF5YXhnY2w0Y0JPZzhuckVOeHhNS3ZBYnFiK1dnQWNId2MwUG41MkdGZHMKZ1FEdWw4ZCtLeGZOSUhzVlFER3V5UzVRNWhaVzF5ZDd4RkVoN3B2VUZ1b0N5MU5VdmRyQVRYRGwrYmZvOFVXVAp5THR2cEdJb3NGcE9uRVpGeXl2Zm9KazVzNWhNCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUpRZ0lCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQ1N3d2dna29BZ0VBQW9JQ0FRRElZY3lDblZiYzdVS1gKWkd6SjJXUWRrNktiQlJTNnRqb1FZQXVRM0dKM2JUZ0V3dlhYazdIakUzM2tPZTQxSXFCY3hBekhNNlVqNW9rcQpYcldkZk5uZjVyVHNXd2sreGZkeTFyWmhQbU16UTdsUFZnblpZNDd2SUVRbnJ0ckUwb1A0UlVSYkdJOGNEM0pSCkZ5UjVLdzVtZlphY29CWHNDMC9Mb21aOFF1OXlPbUJDS2ZDZ1Y2ZHpnZkg2UXJPaElBUVFvVXF3YlU0c0RWZzYKMTNuSCtTYko3dzZSZzF2bW9ES2hWWks2d0NDak9DdGt1cGFBNFN6Mi91OGFIK1UxZnpOL1pGU25pWVpiZGk4dgpBNjdkcDdOY2dnblJJWjkvUVJOaEU4TWZpUmZGQjFRMkdxTythNENaVXJ5cEtKNXZtZUtodXhiajdXUlhOcTZYCkgwWG9FQTVOMy9qYnJQUGpjaUxRMFVKSm5kdDkvbkgwVTJOcWVrTXZtYWREaXVUb2tEcHZCbThhUG84WW40WlEKSFlxaHJLa25qM1IwS25GTGRRck9GdERoOUVzTTRzajlpRGZuYWZyNmsybnFjdTllVCtMbExqNHBDUWpHcnUxcgpMampONVNMUzMvT0xLK0lBQUZIV2UrZ0tLRGxHdkpyd0dselZuMm9mWVdjelRlcWZjZmN5M2xGNDdtU0NvQWxlCnJkRDBiTGFCeU9SRjZGTUI0UzlLdHJ5NHJPcDNzSEpRK3JkOWZGa0pGSVJBUHlqaU84QmowSStCMjA5TWY0WFQKOXhxSWtMUGtxbDJwWUZ3d3l3MmFrTDdOTGhGRzVWRTN1UnBBKyt2Z2d6TmZWMjQzOXAxRWVlL1IyOVFFcUpvdQpjWFlDMlhNT1VENmVSZWZGU3cwN3kraGFrMlM3VXdJREFRQUJBb0lDQUEzU3VWSDFXcTJvN0dRWE9HNEFRaWpNCkszWjROa0xmR1VoUjU5cFphYTJGYWt6aHlpWFIrWDZKdExDTzBvRDEzNHdtdGg3ejBCdVc1clYyalI3Tkl4YVEKQ3NFWFVwN3k5eXdENWRiMWY5QmtocDhUZDJCNHZyNStRbFZlZVpjRVVyaEl4dnRseVZHTk96eWUxUlJLeFJhSwo2VjNxcVRoOFcwZlg3eXY1VGgxYUs1UEU0dVdjeGw5d2dtbmFPaHdPWWxsblZ3aXVzYXJXVE5UYVVudGFFN1B3CmV0Zk04UVVLM2hORkhQY25FOWxPb2FlME8zZXVrUFNGQjZlTXRib29DVHhyaG05OFREbDVBSzVFbWNhT3NBL2MKcEtLNXFCQVdSQ2o1UFFlcTVHbmlKSXdLOEdyTmJiU21BWC9GM3BBaVZJRUZzQUdQT2RIT1l1TG45R0dhNGZHYwpGdjloMmpqWE45Z3M5QjRlZFRDUDNybFJpYWNwQjRVV1pxU1BaOXZCUUs0VytMVUpmeW5ZQWtXcVBwbm4yVjJxClVhYkorM25BRGFMd3VodTc1YmdvMzVNdmpMRnYydGhGZXh6TGQ5MnNhdjlEcUM0L3hzdEpPb1JReGNpTFZIVXIKTW5oTzNOTmVlSXFJWEw4Nmw1ZUdlL0t5a1lGTXd0dGZUeTBHU3dDR2tkcmJpSFF5SU0wUDZtSFBHaTQ4eU40bgpJNU9SRmlRUm96N2ZMeWhhWkVUNktGYTd1Z016ZlRkazkzWW1zYXd5RXNjcWpxQVlVNkFVMWNKN3Z0TjR2RVBRCjlRZWRMWTRRSVQ1Z0plb29vaGdYTjdsM2ZZblZ0UW5ZbGVPUjZjdHk4MVoyZUFpL3FJeC91T0NXYjloWC9KVTYKcGRiZnMrTlRIZDhPeThJS1JIZVpBb0lCQVFEamNzWk40TnU5aGJHYkRkMzlQREZNMjFtRGlCc3c1RkpCdTVmMwp4dG9ud1cvRUVFdHFzeTMzcnd6NVdRNzNyQ3N2Qkg1cUJtZW1iUHk5WHM1dlFYQms5c1IvU3h3UXFIdDYyWG0vCjdtMEJGZUM0cVJaMm9JQmkrWVpSK2FTWU43RzlpazZnMTU2TUZ6Q1duK1ljakNGMDlDK1FGNm1wWjdIbGxqcnUKRm00eHF6WUMyWGxEVjNxZ2IybkpzMTd3V0M1TjNnMVJ0UHVIaEdQYjdjY1RxRzNEMjB1NktrcW1TNStLdGF4SgpqbFdzdENoRlcrWlNJL3Blb1NLWlVIUmxlSkZUME5CQk5qd1lkYzZJMlF3eHpybDNReFdoMzNTNndOdi9WNGZDCkExZ1BlYlVKWXBsanJaSGxiZ3JpSjREWEJCS0FIUVMwNnI4UGV5clZMNlFLSUU5M0FvSUJBUURoaVRwTWxKcksKbWhrNGtQOWltYzVOMnlxZnE3U2JKNWVrbXduM0Y4TUl6eHFTY3ZjKyswUnoybGFseXRqejRnRGVMc2RsazU0OQorYW5PQ0hINElUaDFkS3p3UGxXbTRjNW9uQmJjRTJUclZIdEhBdng5bjZaakxoTEVPM1o1TXVhM296RUhqUEdwCkFpTHRwWHM2ZFYwYmc1UHlnVE9JbWcrWE1Rc1hMWTVzUmNxYitaSThoUkNRbDRtUFdWdzlnbGZJbllucm9KN28KMm42SEhaQ2dUUURJandycE1iSi9EYVUxMXp3Z01nYTJaMFY5bjV4R1Q3VVZFVm9GeHZPL2J1K2NDWTlUK0RDQwoxMDBjNDRjeWZmZWoyK3pMZHpLc3RKV05WSUFjRzFwbGV3V2dVSkF5eEtZTWlXSFVRRUE0b0tHWXpoc0pSY2tUCjdCYnRuWXJnamNJRkFvSUJBRFJxVzliUXJmTWtIMFRyVWpBc3NmUFRUUEtwNkJKQlc4OTRLdEpZQ2loRlJMdDcKUWRZS0N0cmNoWEhsR3pUcWdWMHBmUFIwRzJqWUR2cVpJWnUwQ2ZIS2lJZ0pTQ055b0ZvMFNnRjRNYmloVVJOZApMQ2NVWCtIdlBRd2hLdFJGYVhtVHFRRWFENWliTTRCU3d4WHJHVDY1azBoeW00L0ZyTktLNTNPOHlaSTZzWXpBCmoxaDhqVzd4bmdCMGpMbDRxTnNiQkJqRFMzLzBlNHJRWmlOYW1ra2JmWDBlaCt1QTIvaDhXNExzQVVSMmxCMC8KeTNrOGYxTlZjUUxCN3NEL293WWN4aEZ4TFRJNTIrbmZreGJiWEJSbTZsSk9pN2tKL3VqK1EvUHJEMTBwb0JYVQptaUxGZWl6VVNqL0orTUFVV1NzYkJOMm9oM1ZLM2hrWkRJV2s0b3NDZ2dFQkFMWlZVZnVOZkdMbEdCVENMS1dUClFOVnlwVS8yNmdreGhnZytpMXpuS2ZjYU1CcEx0WldHWC8zbGUzMkhzOFBmWitJNElWMytiTVVmN1dhekx5aHgKK3dvQ0xMb0JPdyt5cUVPc1JWTGdud3NkL3BnWFV2ZGd0WXlqTityTFErbVIvREprVFlRVUwxNzZhakNFUTA2cwppWHh2OEpEeVlTNURsdTBkYWlEdjVKK21BTG4rbDNvei9ZTlg3NDhqcUUzVjdaQXp4TWZveisvaWpMNUJhYVllCitzNHB6cUZlV3pjYVdnRmdJNnpIcE9PY00vTHVzZEdxS1BTQ1Zhd3IvdTA2QzU2em45czc0RVEzT1pGc1pPV3UKTHlHYThDSkNHSWJGYTg2WmpRU3NISFhFY25UOERNZnVjV3ZiT1dyMkVyVjFMNCt3dU96VExVL2M0MkJ3cUZFSQphZDBDZ2dFQU9kL093YTBFczdYcWdMK2hvSEE4SG5kR1Y5c2JibS9LQzFNdWIyVnNtRlJCTFV5T0VZY1BtbVN4Ck03VUdKR1NSVlNxZlBLWS9oS25TbXJrM2o2a21kM1pUZ0JWRkVHTU9laTZ0L0pSbXhnaFpXQW42Y3NnNDlsd2oKOGt1b3g0alNCdCt0TXZMbzB3NDdGRjFnaXNMZWV3QTVTMmlpUzlwUEJaTmI0UGNTV09IOUlqanA2Vm10TUdVUgpOekg0QUdYb3lQUjN2L0FEeng1TzJzWVdEVmJvdHBrY3E2bHFFZVNhdUxnVmxjQmU5dElBaTlmODd6VmFCYmk2Cjlwa1RaU1FlcTlGeStIZDkwSW41dkp3TkxkWHRIRHJ2a3hqeEwxd2xEei9tVjdoL1NEMXZ1dk9CWWwwdGI0c3AKZldyZUZud1BaN09aSGh2VWM4NUpIblZtaDI0Tmd3PT0KLS0tLS1FTkQgUFJJVkFURSBLRVktLS0tLQo=
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
  PATRONI_REPLICATION_PASSWORD: RjhoZFhFQ1pNWG5WSktwVkRZUHFzWnJab3NjOUx5MGc=
  PATRONI_SUPERUSER_PASSWORD: TE1CVHhZVG9IeEkyZThZNVFkaFlaaDM2cXpTSUMyelg=
  PATRONI_admin_PASSWORD: QzBEbzE5NUFnU1BIYVNrOUVoMTRpWHlZaG5JOG5wQmo=
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
[root@k8s-master1 timescale]# helm install tsdb  timescaledb-single
NAME: tsdb
LAST DEPLOYED: Wed Dec 22 15:32:56 2021
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
tsdb	default  	1       	2021-12-08 11:24:45.002796972 +0800 CST	deployed	timescaledb-single-0.8.2


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


[root@k8s-master1 timescale]# kubectl get all | grep tsdb
pod/tsdb-timescaledb-0                        1/1     Running   0          10m
pod/tsdb-timescaledb-1                        1/1     Running   0          6m5s
pod/tsdb-timescaledb-2                        1/1     Running   0          73s
service/tsdb                                LoadBalancer   10.1.226.117   <pending>     5432:30508/TCP                   10m
service/tsdb-config                         ClusterIP      None           <none>        8008/TCP                         10m
service/tsdb-replica                        ClusterIP      10.1.102.69    <none>        5432/TCP                         10m
statefulset.apps/tsdb-timescaledb                        3/3     10m
```



### 5.测试

```shell
# 测试数据库


# superuser password
export PGPASSWORD_POSTGRES=$(kubectl get secret --namespace default tsdb-credentials -o jsonpath="{.data.PATRONI_SUPERUSER_PASSWORD}" | base64 --decode)

    # admin password
export PGPASSWORD_ADMIN=$(kubectl get secret --namespace default tsdb-credentials -o jsonpath="{.data.PATRONI_admin_PASSWORD}" | base64 --decode)



# 1.获取密码（用户postgres）
[root@k8s-master ~]# export PGPASSWORD_POSTGRES=$(kubectl get secret --namespace default tsdb-credentials -o jsonpath="{.data.PATRONI_SUPERUSER_PASSWORD}" | base64 --decode)

[root@k8s-master ~]#  echo $PGPASSWORD_POSTGRES
LMBTxYToHxI2e8Y5QdhYZh36qzSIC2zX

# 获取密码（用户admin）
[root@k8s-master ~]# export PGPASSWORD_ADMIN=$(kubectl get secret --namespace default tsdb-credentials -o jsonpath="{.data.PATRONI_admin_PASSWORD}" | base64 --decode)

[root@k8s-master ~]# echo $PGPASSWORD_ADMIN
C0Do195AgSPHaSk9Eh14iXyYhnI8npBj



# 2.数据库连接信息
tsdb.default.svc.cluster.local
5432:30508
postgres
LMBTxYToHxI2e8Y5QdhYZh36qzSIC2zX



# 3.连接数据库
kubectl run -i --tty --rm psql --image=postgres \
--env "PGPASSWORD=$PGPASSWORD_POSTGRES" \
--command -- psql -U postgres \
-h tsdb.default.svc.cluster.local postgres



[root@k8s-master pro]# kubectl run -i --tty --rm psql --image=postgres \
> --env "PGPASSWORD=$PGPASSWORD_POSTGRES" \
> --command -- psql -U postgres \
> -h tsdb.default.svc.cluster.local postgres
If you don't see a command prompt, try pressing enter.
postgres=# 



[root@k8s-master pro]# kubectl run -i --tty --rm psql --image=postgres \
> --env "PGPASSWORD=$PGPASSWORD_POSTGRES" \
> --command -- psql -U postgres \
> -h tsdb.default.svc.cluster.local postgres
If you don't see a command prompt, try pressing enter.
postgres=# \d
Did not find any relations.
postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 admin     | Create role, Create DB                                     | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 standby   | Replication                                                | {}

postgres=# \l
                              List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   
-----------+----------+----------+---------+---------+-----------------------
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
(3 rows)

# 修改密码
postgres=# ALTER USER postgres with encrypted password 'postgres';
ALTER ROLE
postgres=# 
postgres=# 
postgres=# \q
Session ended, resume using 'kubectl attach psql -c psql -i -t' command when the pod is running
pod "psql" deleted



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



### 6.外部访问数据库

```shell
# 查看
[root@k8s-master1 rabbitmq]# kubectl get svc
NAME                                TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                          AGE
tsdb                                LoadBalancer   10.1.226.117   <pending>     5432:30508/TCP                   21m
tsdb-config                         ClusterIP      None           <none>        8008/TCP                         21m
tsdb-replica                        ClusterIP      10.1.102.69    <none>        5432/TCP                         21m



# 修改从service类型
[root@k8s-master ~]# kubectl edit svc tsdb-replica
service/tsdb-replica edited

spec:
......
  type: NodePort


[root@k8s-master1 rabbitmq]# kubectl get svc
NAME                                TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                          AGE
tsdb                                LoadBalancer   10.111.84.227    <pending>     5432:31353/TCP                   34m
tsdb-config                         ClusterIP      None             <none>        8008/TCP                         34m
tsdb-replica                        NodePort       10.102.158.86    <none>        5432:30634/TCP                   34m



# 外部访问信息
# 主
172.51.216.81
tsdb.default.svc.cluster.local
5432:31353
postgres
postgres


# 从
172.51.216.81
tsdb-replica.default.svc.cluster.local
5432:30634
postgres
postgres
```



![](/images/kubernetes/pro/middleware/timescale-1.png)

![](/images/kubernetes/pro/middleware/timescale-2.png)



