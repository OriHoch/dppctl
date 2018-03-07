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
Waiting for available worker and DB pods
Assigned Worker Pod: dppctl-worker-rokfi8489824
Assigned DB Pod: dppctl-db-koejijdif48joi
DB Web UI is available at https://dppctl.uumpa.com/dppctl-db-koejijdif48joi/
System: PostgreSQL, Server: localhost, Username: postgres, Password: idjnmfkjnfur29hr
./dump-to-db: SUCCESS, processed 90088 rows
INFO    :RESULTS:
INFO    :SUCCESS: ./dump-to-db {'bytes': 15566365, 'count_of_rows': 90088, 'dataset_name': '_', 'hash': 'a3d87beee69e1915b884540960271752', 'num rows': 90088}
```
