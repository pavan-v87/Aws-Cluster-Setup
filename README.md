# Aws-Cluster-Setup
## Cluster Create
This setup is intended for optimum Free-Tier usages. More details available in kops [instance_groups.md](https://github.com/kubernetes/kops/blob/master/docs/instance_groups.md).

Use the following command to initialize yaml containing the cluster description
```sh
kops create cluster --name=<cluster-name> --state=s3://<S3-bucket-name> --zones=ap-south-1a \
--node-count=2 --master-size="t2.micro" --node-size=t2.micro --dry-run -o yaml > mycluster.yaml
```
`mycluster.yaml` will contain cluster description of t2.micro instance types of 1 master and 2 nodes. We can use this file to create clusted with command.(More details [here](https://github.com/kubernetes/kops/issues/2603#issuecomment-354034970)).
```sh
kops create -f mycluster.yaml
```
Dont change much in this default file. (More Details [here](https://github.com/kubernetes/kops/blob/master/docs/manifests_and_customizing_via_api.md#closing-thoughts)).

## Edit Cluster configuration
Once the cluster is created we can optionally edit the cluster parameters:

### Edit the cluster 
```sh
kops edit cluster <cluster-name>
```

## Update Cluster
Finally update the cluster we have created with:
* Preview: `kops update cluster <clustername>`
* Apply: `kops update cluster <clustername> --yes`
Succesfull update will set the kubectl context. (We can see this message `kops has set your kubectl context to <cluster-name>`)

This might throw up error like:"SSH public key must be specified when running with AWS". If so, create the ssh key and update throgh kops
```sh
shh-keygen //to create ssh key
kops create secret --name <cluster-name> sshpublickey admin -i ~/.ssh/id_rsa.pub
kops update cluster <clustername> --yes
```

## Validate
We can then validate that cluster is up and running with:
```sh
kops validate cluster <clustername> 
```
## Edit Instance configurations
### Edit the master instance
```sh
kops edit ig --name=<cluster-name> master-ap-south-1a
```
For example, to set up a 10GB root volume(rootVolumeSize), your InstanceGroup spec might look like:
```
metadata:
  creationTimestamp: "2016-07-11T04:14:00Z"
  name: nodes
spec:
  machineType: t2.medium
  maxSize: 2
  minSize: 2
  role: Node
  rootVolumeSize: 10
```

### Edit the child nodes
```sh
kops edit ig --name=<cluster-name> nodes
```

### Rolling update
After editing `update` the cluster and perform `rolling-update`
* `kops update cluster <cluster-name> --yes`
* `kops rolling-update cluster <cluster-name> --yes`
Update will take some time to effect and then after its completed we can validate the cluster
