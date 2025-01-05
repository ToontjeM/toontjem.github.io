# Installation EDB Postgres AI Control Plane on kind

## Pre-reqs
- git (OS repos)
- make (OS repos)
- kind (https://kind.sigs.k8s.io/)
- helm (https://helm.sh/)

## Installation
> [!TIP]
> TL;DR Just run the `00-provision.sh` script which does all the below and go take a ☕.
>
> Script looks like this:
> ```
> #!/bin/bash
>
> make create-kind
> EDBTOKEN=(`cat $HOME/.edbtoken`)
> echo $EDBTOKEN | ./scripts/install-static-pull-secret.sh
> helm upgrade -n edbpgai-bootstrap \
> --install -f deploy/charts/edbpgai-bootstrap/values-kind.yaml \
> --set bootstrapImage=docker.enterprisedb.com/staging_pgai-platform/edbpgai-bootstrap/bootstrap-kind:v1.0.0-gm-appl \
> --set remoteContainerRegistryURL=docker.enterprisedb.com/staging_pgai-platform \
> --set internalContainerRegistryURL=docker.enterprisedb.com/staging_pgai-platform \
> bootstrap deploy/charts/edbpgai-bootstrap
> POD=$(kubectl get pods -n edbpgai-bootstrap -o custom-columns=":metadata.name" | grep -v "NAME")
> kubectl logs -n edbpgai-bootstrap $POD -f
> ```

`git clone https://github.com/EnterpriseDB/edbpgai-boot`

`cd edbpgai-boot`

``export EDBToken=(`cat $HOME/.edbtoken`)``

`make create-kind`
 
```
bash /Users/ton.machielsen/edbpgai-bootstrap/scripts/create-kind.sh /Users/ton.machielsen/edbpgai-bootstrap/bin/kind-v0.24.0 /Users/ton.machielsen/edbpgai-bootstrap/kind-config.yaml
Creating cluster "edbpgai" ...
 ✓ Ensuring node image (kindest/node:v1.29.8) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-edbpgai"
You can now use your cluster with:

kubectl cluster-info --context kind-edbpgai

Have a nice day! 👋
```

`echo $EDBTOKEN | ./scripts/install-static-pull-secret.sh`

```
Enter the password for staging_beaconator@docker.enterprisedb.com
Creating secret edb-cred
namespace/upm-replicator created
secret/edb-cred created
namespace/edbpgai-bootstrap created
secret/edb-cred created
secret/edb-cred annotated
Installation completed
```
Run the bootstrap using helm.
```
helm upgrade -n edbpgai-bootstrap \
  --install -f deploy/charts/edbpgai-bootstrap/values-kind.yaml \
  --set bootstrapImage=docker.enterprisedb.com/staging_pgai-platform/edbpgai-bootstrap/bootstrap-kind:v1.0.0-gm-appl \
  --set remoteContainerRegistryURL=docker.enterprisedb.com/staging_pgai-platform \
  --set internalContainerRegistryURL=docker.enterprisedb.com/staging_pgai-platform \
  bootstrap deploy/charts/edbpgai-bootstrap
```

> [!NOTE]
> This will take appx 15 minutes to complete. You can follow the progess using `kubectl logs -n edbpgai-bootstrap edbpgai-bootstrap-job-v1.0.0-<tab> -f`

```
...

12:32:20PM: create customresourcedefinition/filters.fluentbit.fluent.io (apiextensions.k8s.io/v1) cluster
12:32:20PM: create customresourcedefinition/clusterfluentdconfigs.fluentd.fluent.io (apiextensions.k8s.io/v1) cluster
12:32:20PM: create customresourcedefinition/clusterinputs.fluentbit.fluent.io (apiextensions.k8s.io/v1) cluster
12:32:22PM: create customresourcedefinition/clusterinputs.fluentd.fluent.io (apiextensions.k8s.io/v1) cluster
12:32:25PM: create secret/loki-http-pass (v1) namespace: logging
12:32:25PM: create customresourcedefinition/outputs.fluentd.fluent.io (apiextensions.k8s.io/v1) cluster
12:32:27PM: create customresourcedefinition/fluentds.fluentd.fluent.io (apiextensions.k8s.io/v1) cluster
12:32:27PM: ---- waiting on 30 changes [0/39 done] ----
-main- +2 -2 (base) √ ~/edbpgai-bootstrap
```

The UI is now available at http://localhost:8000

Currently the UI can only be accessed from the host where it is installed or using X-forwarding (which is terribly slow).

Login using `owner@mycompany.com` with password `password` and create a cluster.

Once a cluster is created you will see the following:
![image](https://github.com/user-attachments/assets/f9538215-0ab6-42d5-9a73-6aef32f59143)

Click on the cluster and in overview you will see something like this:
![image](https://github.com/user-attachments/assets/1841fdd8-2543-4203-ba07-bf6ee9329bc2)

Since the pgai.com domain doesn't exist and the port 30495 isn't accessible, do the following:
- Check out what is the name of the cluster using `kubectl get clusters -A`. The cluster you are looking for tis the one that starts with the `p-`.
 ```
NAMESPACE                NAME                       AGE   INSTANCES   READY   STATUS                     PRIMARY
edb-migration-portal     cluster-migration-portal   53m   1           1       Cluster in healthy state   cluster-migration-portal-1
*** p-bvhzn7zaie ***     p-bvhzn7zaie               30m   1           1       Cluster in healthy state   p-bvhzn7zaie-1
transporter-ui-service   transporter-db             40m   1           1       Cluster in healthy state   transporter-db-1
upm-beaco-ff-base        app-db                     46m   1           1       Cluster in healthy state   app-db-1
upm-beacon               beacondb                   46m   1           1       Cluster in healthy state   beacondb-1
```
- Forward port 5432 in that cluster to 5433 on your local machine using `kubectl port-forward -n p-bvhzn7zaie services/p-bvhzn7zaie-rw-external 5433:5432 --address 0.0.0.0`

Now that port 5433 is forwarded to your PC you can run `psql -h localhost -p 5433 -U edb_admin postgres`.
```
Password for user edb_admin: 
psql (17.0 (Homebrew petere/postgresql), server 17.2 (Debian 17.2-1EDB.bookworm))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

postgres=#
```
