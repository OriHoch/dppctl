# dppctl

**work in progress**

## Usage Examples

### Dump datapackage to DB

```
$ ls
pipeline-spec.yaml
$ cat pipeline-spec.yaml
dump-to-db:
  pipeline:
  - run: load_resource
    parameters:
      url: https://storage.googleapis.com/knesset-data-pipelines/data/committees/all/datapackage.json
      resource: .*
  - run: dump.to_sql
    parameters:
      engine: env://DPP_DB_ENGINE
      tables:
        gcs_document_committee_session_files:
          resource-name: gcs_document_committee_session_files
        gcs_list_files
          resource-name: gcs_list_files
        joined_meetings
          resource-name: joined-meetings
        kns_cmtsessionitem
          resource-name: kns_cmtsessionitem
        kns_cmtsitecode
          resource-name: kns_cmtsitecode
        kns_committee
          resource-name: kns_committee
        kns_committeesession
          resource-name: kns_committeesession
        kns_documentcommitteesession
          resource-name: kns_documentcommitteesession
        kns_jointcommittee
          resource-name: kns_jointcommittee
$ dppctl run ./dump-to-db
Using the public dppctl cluster
do not use for private data!
Run dppctl init --help for cluster initialization and configuration options
Waiting for available worker and DB pods
Assigned Worker Pod: dppctl-worker-rokfi8489824
Assigned DB Pod: dppctl-db-koejijdif48joi
DB Web UI is available at https://dppctl.uumpa.com/dppctl-db-koejijdif48joi/
System: PostgreSQL, Server: localhost, Username: postgres, Password: idjnmfkjnfur29hr
./dump-to-db: SUCCESS, processed 90088 rows
INFO    :RESULTS:
INFO    :SUCCESS: ./dump-to-db {'bytes': 15566365, 'count_of_rows': 90088, 'dataset_name': '_', 'hash': 'a3d87beee69e1915b884540960271752', 'num rows': 90088}
Unassigned Worker Pod
Keeping the DB and Web UI running for 1 hour
```

### Private Cluster Initialization using Kamatera and usage of persistent pods

Public cloud uses non persitent pods which are limited to 1 hour

Starting a private cluster allows to use persistent pods which keep running forever

```
$ dppctl init kamatera
Free Sign Up for The Kamatera Kubernetes service at ???
Kamatera Account ID: 
Kamatera Secret: 
Initializing Cluster, re-run dppctl init to track initialization progress
$ dppctl init
Waiting for Kamatera cluster initialization
Cluster is ready, starting 1 persistent DB pod and 2 persistent worker pods
DB pod: dppctl-db-koejijdif48joi
Worker pod: dppctl-worker-dkofij848dhiu
Worker pod: dppctl-worker-0934j0fjoff
dppctl initialization completed succesfully
$ dppctl run ./dump-to-db
Using kamatera private cluster
Assigned Worker Pod: dppctl-worker-dkofij848dhiu
Assigned DB Pod: dppctl-db-koejijdif48joi
DB Web UI is available at https://dppctl.uumpa.com/dppctl-db-koejijdif48joi/
System: PostgreSQL, Server: localhost, Username: postgres, Password: lkdfoij08j43fjforewrfij
./dump-to-db: SUCCESS, processed 90088 rows
INFO    :RESULTS:
INFO    :SUCCESS: ./dump-to-db {'bytes': 15566365, 'count_of_rows': 90088, 'dataset_name': '_', 'hash': 'a3d87beee69e1915b884540960271752', 'num rows': 90088}
$ dppctl list pods
[{"name": "dppctl-worker-dkofij848dhiu", "status": "assigned"},
 {"name": "dppctl-worker-0934j0fjoffdf", "status": "available"},
 {"name": "dppctl-db-koejijdif4sss8joi", "status": "assigned"}]
$ dppctl ssh dppctl-worker-dkofij848dhiu
dppctl-worker-dkofij848dhiu:~$ dpp
Available Pipelines:
./dump-to-db
dppctl-worker-dkofij848dhiu:~$ dpp run --verbose ./dump-to-db
[./dump-to-db:T_0] >>> INFO    :c39d7233 RUNNING ./dump-to-db
[./dump-to-db:T_0] >>> INFO    :c39d7233 Collecting dependencies
[./dump-to-db:T_0] >>> INFO    :c39d7233 Running async task
[./dump-to-db:T_0] >>> INFO    :c39d7233 Waiting for completion
[./dump-to-db:T_0] >>> INFO    :c39d7233 Async task starting
[./dump-to-db:T_0] >>> INFO    :c39d7233 Searching for existing caches
[./dump-to-db:T_0] >>> INFO    :c39d7233 Building process chain:
[./dump-to-db:T_0] >>> INFO    :- load_resource
[./dump-to-db:T_0] >>> INFO    :- dump.to_sql
./dump-to-db: WAITING FOR OUTPUT
[./dump-to-db:T_0] >>> INFO    :- (sink)
./dump-to-db: RUNNING, processed 89800 rows
./dump-to-db: RUNNING, processed 89900 rows
./dump-to-db: RUNNING, processed 90000 rows
./dump-to-db: RUNNING, processed 90088 rows
[./dump-to-db:T_0] >>> INFO    :dump.to_sql: INFO    :Processed 90088 rows
[./dump-to-db:T_0] >>> INFO    :c39d7233 DONE /home/ori/virtualenvs/knesset-data-pipelines-fhQl9XOq/lib/python3.6/site-packages/datapackage_pipelines/manager/../lib/internal/sink.py
[./dump-to-db:T_0] >>> INFO    :c39d7233 DONE /home/ori/virtualenvs/knesset-data-pipelines-fhQl9XOq/lib/python3.6/site-packages/datapackage_pipelines/specs/../lib/dump/to_sql.py
[./dump-to-db:T_0] >>> INFO    :c39d7233 DONE V ./dump-to-db {'.dpp': {'out-datapackage-url': 'mayaslim/datapackage.json'}, 'bytes': 15566365, 'count_of_rows': 90088, 'dataset_name': '_', 'hash': 'cf1ad0f666601d174757b62d52643371', 'num./dump-to-db: SUCCESS, processed 90088 rows
INFO    :RESULTS:
INFO    :SUCCESS: ./dump-to-db {'bytes': 15566365, 'count_of_rows': 90088, 'dataset_name': '_', 'hash': 'cf1ad0f666601d174757b62d52643371', 'num rows': 90088}
```

### Persistent Storage of Public Data Packages

```
$ dppctl init public
Switching to the public dppctl cluster
$ ls
pipeline-spec.yaml
$ cat pipeline-spec.yaml
dump-to-path:
  pipeline:
  - run: load_resource
    parameters:
      url: https://storage.googleapis.com/knesset-data-pipelines/data/committees/all/datapackage.json
      resource: .*
  - run: dump.to_path
    parameters:
      out-path: data/committees
$ dppctl run ./dump-to-path
Using the public dppctl cluster
do not use for private data!
Run dppctl init --help for cluster initialization and configuration options
Waiting for available worker and DB pods
Assigned Worker Pod: dppctl-worker-dkofij848dhiu
./dump-to-path: SUCCESS, processed 90088 rows
INFO    :RESULTS:
INFO    :SUCCESS: ./dump-to-path {'bytes': 15566365, 'count_of_rows': 90088, 'dataset_name': '_', 'hash': 'a3d87beee69e1915b884540960271752', 'num rows': 90088}
Syncing Data Packages to Public Storage
data/committees > https://console.cloud.google.com/storage/browser/dppctl/dppctl-worker-dkofij848dhiu/committees/
```

### Static HTML Website Hosting

```
$ ls -R
.:
pipeline-spec.yaml template_functions.py render_list.py

./public:
404.html  bower_components  content  index.html  index_en.html  index_he.html  ribbons.css

./templates:
list.html list_item.html index.html
$ cat pipeline-spec.yaml
render_index:
  pipeline:
  - run: render_index
    code: |
        from datapackage_pipelines.wrapper import ingest, spew
        from template_functions import render_template
        parameters, datapackage, resources, stats = ingest() + ({},)
        render_template('index.html', {"title": "Hello World"}, 'data/index.html')
        spew(datapackage, [], stats)

render_list:
  pipline:
  - run: load_resource
    parameters:
      url: https://console.cloud.google.com/storage/browser/dppctl/dppctl-worker-dkofij848dhiu/committees/
      resource: .*
  - run: render_list
$ dppctl run all
Using the public dppctl cluster
do not use for private data!
Run dppctl init --help for cluster initialization and configuration options
Waiting for available worker and DB pods
Assigned Worker Pod: dppctl-worker-dkofij848dhiu
./render_index: SUCCESS, processed 90088 rows
./render_list: SUCCESS, processed 90088 rows
Syncing Static Website to Public Storage
data/index.html > https://dppctl.uumpa.com/dppctl-worker-dkofij848dhiu/
```
