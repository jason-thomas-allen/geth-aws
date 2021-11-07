# geth

Geth running in Google Kubernetes Engine

Thx Daz Wilkin
https://medium.com/google-cloud/ethereum-on-google-cloud-platform-8f10c82493ca

## Switch kubectl context

```
$ kubectl config get-contexts

$ kubectl config current-context

$ kubectl config use-context CONTEXT_NAME
```

```
$ gcloud config list
```

Create the cluster
Note num-nodes is nodes per **zone**

```
$ gcloud container clusters create geth --zone asia-southeast1-a --node-locations asia-southeast1-a,asia-southeast1-b --num-nodes 1

```

Each node is a virtual machine provided by the Google Compute Engine (GCE) infrastructure-as-a-service platform. Each VM incurs costs.

```
$ gcloud compute instances list
```

Convenient way to save costs without deleting cluster objects is to scale to zero.

```
$ gcloud container clusters resize geth --size 0
```

Create persistent volume

```
$ gcloud compute disks create blockchain --size=500GiB --type=pd-standard --region=asia-southeast1 --replica-zones=asia-southeast1-a,asia-southeast1-b
$ kubectl apply -f persistent-volume.yaml
$ kubectl get pv
```

Deploy geth

```
$ kubectl apply -f deployment.yaml
```

Check deployment

```
$ kubectl get po
$ kubectl get svc
```

Test geth

Get external id of load balancer service

curl --header "Content-Type: application/json" \
 --request POST \
 --data '{"jsonrpc": "2.0","method": "web3_clientVersion","params": [],"id": 1}' \
http://EXTERNAL-IP:8545

curl --header "Content-Type: application/json" \
 --request POST \
 --data '{"jsonrpc": "2.0","method": "eth_gasPrice","params": [],"id": 1}' \
http://EXTERNAL-IP:8545

Delete the deployment

```
$ k delete all --all
```

Tear down cluster

```
$ gcloud container clusters delete geth
```

Tear down persistent disk

```
$ gcloud compute disks list
$ gcloud compute disks delete DISK-NAME
```

gcloud projects add-iam-policy-binding $GKE_PROJECT \
  --member=serviceAccount:$SA_EMAIL \
 --role=roles/container.admin \
 --role=roles/storage.admin \
 --role=roles/container.clusterViewer \
 --role=roles/container.persistentVolumeClaims.get \
 --role=roles/container.deployments.get \
 --role=roles/container.services.get
