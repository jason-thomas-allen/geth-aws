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

Setting up API user and GitHub secrets

https://docs.github.com/en/actions/deployment/deploying-to-your-cloud-provider/deploying-to-google-kubernetes-engine

Create a new service account

```
$ export SA_NAME=user-id
$ gcloud iam service-accounts create $SA_NAME --display-name $SA_NAME
```

Retrieve email address of new account

```
$ gcloud iam service-accounts list
```

In the console create new role from Kubernetes Engine Developer and assign it to this service account.
How to do this from CLI?

Add custom role to SA

```
$ gcloud projects add-iam-policy-binding $GKE_PROJECT \
  --member=serviceAccount:$SA_EMAIL \
  --role=projects/civic-karma-326401/roles/CustomKubernetesEngineDevelope
```

Download JSON key file -- do not commit this to repo!!

```
$ gcloud iam service-accounts keys create key.json --iam-account=$SA_EMAIL
```

Store the service account key as a secret named GKE_SA_KEY

```
$ export GKE_SA_KEY=$(cat key.json | base64)
```
