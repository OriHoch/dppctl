# dppctl

Serverless Data Pipelines Framework

## Differentiating factors from other serverless frameworks

* Focus on data, relying on the [frictionless data](https://frictionlessdata.io/) methodolgy, specs and tools
* Minimal external 3rd party service dependencies, including minimal dependency on the `dppctl` framework itself
* Open to extend and customize using existing skills and knowledge
* Highly constrained to allow to focus on specific use-cases (does not intend to replace existing serverless frameworks)

## Core Technologies

* Pipelines processing using [datapackage-pipelines framework](https://github.com/frictionlessdata/datapackage-pipelines/blob/master/README.md) - Handles the data pipelines processing which is at the core of the framework. Pipelines are defined using a simple yaml format with a rich standard library of processors.
* Infrastructure management using [Kubernetes](kubernetes.io) - providing the dynamic and scalable infrastructure for running the pipelines, and allowing to dynamically add related infrastructure as needed
* Python 3.6 - Using latest Python is great for data processing, you can embed Python code inside the pipelines yaml, as files alongside the pipelines yaml or as separate Python packages.


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

### Using a local cluster with minikube, publish to public dppctl cluster

Run locally using [minikube](https://github.com/kubernetes/minikube), publish to the public cluster

```
$ dppctl init minikube-public
Initializing a local minikube cluster which publishes data to the public dppctl cluster
Track progress by re-running dppctl init
$ dppctl init
Waiting for local minikube cluster to be created
Cluster is ready with 1 worker pod
$ dppctl run
Using local minikube cluster with public publishing
do not use for private data!
Run dppctl init --help for cluster initialization and configuration options
Assigned Worker Pod: dppctl-worker-dkofij848dhiu
./render_index: SUCCESS, processed 90088 rows
./render_list: SUCCESS, processed 90088 rows
Syncing Static Website to Public Storage
data/index.html > https://dppctl.uumpa.com/dppctl-worker-dkofij848dhiu/
```

### Use a custom docker image + continuous deployment

```
$ ls
pipeline-spec.yaml Dockerfile .dppctl.yaml
$ cat Dockerfile
from frictionlessdata/datapackage-pipelines
$ cat .dppctl.yaml
clusterProvider: gcloud
gcloudProjectId: dppctl
gcloudZone: us-central1-a
gcloudClusterName: dppctl
gcloudClusterNamespace: dppctl
continuousDeployment:
- appRepoPath: OriHoch/dppctl/examples/customDockerImageAndContinuousDeployment
  opsRepoPath: OriHoch/examples/opsRepo
  actions:
  - branch: master
    environment: staging
    pipeline: all
  - tags: true
    environment: production
    pipeline: all
$ dppctl init
gcloud cluster is ready
to setup continuous deployment you need to create a GitHub machine user and provide the details
GitHub Auth Token:
GitHub Continuous Deployment user name:
GitHub Continuous Deployment user email:
setting up continuous deployment of 'all' pipelines on commits to master branch of OriHoch/dppctl/examples/customDockerImageAndContinuousDeployment to staging environment of ops repo OriHoch/examples/opsRepo
setting up continuous deployment of 'all' pipelines' on published tags of OriHoch/dppctl/examples/customDockerImageAndContinuousDeployment to ops repo OriHoch/examples/opsRepo
```


## Ideal Systems Architecture

### Dppctl Operator

A Helm chart that installs a Kubernetes custom resource definition using operator pattern to start / stop pipeline workers and related infra.

The dppctl operator will act on pipeline resource changes - when a request to run a pipeline is received the operator will deploy relevant pods / deployments using the dppctl pipelines Helm Chart

see [1](https://github.com/giantswarm/operator-example-python?files=1)

### Dppctl Pipelines Infra.

A Helm chart that starts pipelines and related infrastructure, see [1](https://github.com/OriHoch/datapackage-pipelines-playground/tree/master/charts/pipeline) [2](https://github.com/OriHoch/knesset-data-k8s/tree/master/charts-external/pipelines-jobs)

Each pipeline is a deployment with a specific pipeline docker image, when the pod starts it waits for the workload. Once it received the workload, it either - runs and exits (causing another pod to start), or - keeps running (e.g. to support cronjobs, or other design patterns)

The pipeline chart includes both the pipeline and the related infrastructure, all as a single chart, configurable by custom yaml configuration (the helm chart values).

### Dppctl CLI

The main user interface, interacts with the operator via the operator proxy.

Also, provides scripts to initialize a new dppctl environment and install the required components.

### Dppctl Operator Proxy

An API proxy that forwards requests to the Dppctl operator, validates with the auth and and business logic servers

### Auth Server

Provides authentication based on 3rd party OAuth providers (GitHub / Google)

### Business Logic Server

Provides the business logic, initiates actions based on changes, for example to enforce usage limits

Gets notified by the Operator proxy about every action and keeps full usage log and related metrics.
