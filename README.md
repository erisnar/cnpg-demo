# CNPG

Opprett nytt kind cluster

```bash
kind create cluster --name pg
```

Installer CNPG operatoren

```bash
kubectl apply -f \
  https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/release-1.20/releases/cnpg-1.20.6.yaml
```

Deploy PostgreSQL

```bash
kubectl apply -f cluster-example.yaml
```

Ressurser generert

```bash
 > k get po
NAME                READY   STATUS    RESTARTS   AGE
cluster-example-1   1/1     Running   0          52m
cluster-example-2   1/1     Running   0          52m
cluster-example-3   1/1     Running   0          52m
 > k get svc
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
cluster-example-r    ClusterIP   10.96.71.205    <none>        5432/TCP   53m
cluster-example-ro   ClusterIP   10.96.107.230   <none>        5432/TCP   53m
cluster-example-rw   ClusterIP   10.96.96.229    <none>        5432/TCP   53m
kubernetes           ClusterIP   10.96.0.1       <none>        443/TCP    55m
 > k get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                       STORAGECLASS   REASON   AGE
pvc-1299252c-e001-4fae-8866-54e9b5c20aa0   1Gi        RWO            Delete           Bound    default/cluster-example-3   standard                56m
pvc-18a3d857-f8e3-4823-be39-f95bd4982071   1Gi        RWO            Delete           Bound    default/cluster-example-2   standard                56m
pvc-c658b309-4869-46bd-991c-d00baea6e215   1Gi        RWO            Delete           Bound    default/cluster-example-1   standard                57m
 > k get pvc
NAME                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
cluster-example-1   Bound    pvc-c658b309-4869-46bd-991c-d00baea6e215   1Gi        RWO            standard       57m
cluster-example-2   Bound    pvc-18a3d857-f8e3-4823-be39-f95bd4982071   1Gi        RWO            standard       56m
cluster-example-3   Bound    pvc-1299252c-e001-4fae-8866-54e9b5c20aa0   1Gi        RWO            standard       56m
 > k get secrets
NAME                          TYPE                       DATA   AGE
cluster-example-app           kubernetes.io/basic-auth   9      57m
cluster-example-ca            Opaque                     2      57m
cluster-example-replication   kubernetes.io/tls          2      57m
cluster-example-server        kubernetes.io/tls          2      57m
cluster-example-superuser     kubernetes.io/basic-auth   9      57m
```

Installer cnpg plugin

```bash
curl -sSfL \
  https://github.com/cloudnative-pg/cloudnative-pg/raw/main/hack/install-cnpg-plugin.sh | \
  sudo sh -s -- -b /usr/local/bin
```

PSQL

```bash
 > k cnpg psql cluster-example
psql (16.1 (Debian 16.1-1.pgdg110+1), server 16.2 (Debian 16.2-1.pgdg110+2))
Type "help" for help.

postgres=# \l
                                                  List of databases
   Name    |  Owner   | Encoding | Locale Provider | Collate | Ctype | ICU Locale | ICU Rules |   Access privileges
-----------+----------+----------+-----------------+---------+-------+------------+-----------+-----------------------
 app       | app      | UTF8     | libc            | C       | C     |            |           |
 postgres  | postgres | UTF8     | libc            | C       | C     |            |           |
 template0 | postgres | UTF8     | libc            | C       | C     |            |           | =c/postgres          +
           |          |          |                 |         |       |            |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | C       | C     |            |           | =c/postgres          +
           |          |          |                 |         |       |            |           | postgres=CTc/postgres
(4 rows)
```

Nyttige kommandoer for feilsøking

```bash
 > k cnpg status cluster-example
 Cluster Summary
Name:                cluster-example
Namespace:           default
System ID:           7423314479403827221
PostgreSQL Image:    ghcr.io/cloudnative-pg/postgresql:16.1
Primary instance:    cluster-example-1
Primary start time:  2024-10-08 08:11:33 +0000 UTC (uptime 29m21s)
Status:              Cluster in healthy state
.
.
.
```

Generere rapport og logger

```bash
 > k cnpg report cluster cluster-example
Successfully written report to "report_cluster_cluster-example_2024-10-08T08:42:05Z.zip" (format: "yaml")

 > unzip report_cluster_cluster-example_2024-10-08T08:42:05Z.zip
Archive:  report_cluster_cluster-example_2024-10-08T08:42:05Z.zip
   creating: report_cluster_cluster-example_2024-10-08T08:42:05Z/
   creating: report_cluster_cluster-example_2024-10-08T08:42:05Z/manifests/
  inflating: report_cluster_cluster-example_2024-10-08T08:42:05Z/manifests/cluster.yaml
  inflating: report_cluster_cluster-example_2024-10-08T08:42:05Z/manifests/cluster-pods.yaml
  inflating: report_cluster_cluster-example_2024-10-08T08:42:05Z/manifests/cluster-jobs.yaml
  inflating: report_cluster_cluster-example_2024-10-08T08:42:05Z/manifests/events.yaml
  inflating: report_cluster_cluster-example_2024-10-08T08:42:05Z/manifests/cluster-pvcs.yaml
```

## Ressurser

- https://cloudnative-pg.io/documentation/1.20/quickstart/
- https://github.com/cloudnative-pg/cloudnative-pg/blob/main/docs/src/samples/cluster-example.yaml
- https://cloudnative-pg.io/documentation/1.20/kubectl-plugin

## Backup eksempel

Apply eksempelet med backup

```bash
 > k apply -f cluster-example-with-backup.yaml
```

Vi bruker Minio som S3 provider. Apply configen

```bash
 > k apply -f minio.yaml
```

Et ScheduledBackup objekt ble opprettet med frekvens satt til `schedule: "0 0 * * 0"`. Dette betyr at en full backup blir kjørt hvert uke.

```bash
 > k get ScheduledBackup
NAME             AGE   CLUSTER     LAST BACKUP
backup-example   14h   pg-backup   5h37m
```

Kjør port-forward for å tilgjengeliggjøre portene til poden på localhost

```bash
 > kubectl port-forward service/minio 9001:9001 9000:9000
```

I en annen terminal sett opp og kjør Minio client

```bash
 > brew install minio/stable/mc
 > mc alias set cnpg http://localhost:9001 minio minio123
 Added `cnpg` successfully.
 > mc ls cnpg
[2024-10-08 16:41:59 CEST]     0B backups/
 > mc ls cnpg/backups/example-backup/
[2024-10-09 07:41:28 CEST]     0B wals/
```

`wals/` er write ahead logs. En full backup blir bare kjørt hver uke i dette eksempelet. For å trigge en backup kan vi kjøre Backup CRD'en

```bash
 > k apply -f on-demand-backup.yaml
 > mc ls cnpg/backups/example-backup/
[2024-10-09 07:41:28 CEST]     0B base/
[2024-10-09 07:41:28 CEST]     0B wals/
 > mc ls cnpg/backups/recoveredCluster/base/20241008T144435/
[2024-10-08 16:44:40 CEST] 1.3KiB STANDARD backup.info
[2024-10-08 16:44:40 CEST]  31MiB STANDARD data.tar
```

Hver backup har et identifiserbart mappenavn
