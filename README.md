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

## Ressurser

- https://cloudnative-pg.io/documentation/1.20/quickstart/
- https://github.com/cloudnative-pg/cloudnative-pg/blob/main/docs/src/samples/cluster-example.yaml
