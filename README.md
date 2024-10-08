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
