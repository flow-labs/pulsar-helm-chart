# installation
## Create cluster
**warning - make sure you use a cluster that is exclusive to this pulsar install to avoid accidentally deleting or modifying some other important stuff**

Do this in the eks UI

You will need to install the Amazon EBS CSI Driver for persistent storage. This can be added by clicking the add-ons tab in the eks UI.

You will need to add tags to the aws subnets in order for loadbalancers to work in your new cluster. The tags need to be:
```
    Key: kubernetes.io/cluster/cluster-name Value: shared
```
where cluster-name is the name of the cluster. The subnets that you need to add this to are the two specified in node_group.yaml. Make sure these are also the same subnets that your cluster is using. The subnets are currently:
```
subnet-bf44bae2
subnet-4a671661
```

https://aws.amazon.com/premiumsupport/knowledge-center/eks-vpc-subnet-discovery/

## Setup the nodegroup
Create the nodegroup from the config using the following command:
```bash
  eksctl create nodegroup --config-file flow_labs_charts/dev/node_group.yaml
```

This will automatically set up a few required things such as ebs csi driver permissions.
    


## Install official pulsar helm chart

```bash
./scripts/pulsar/prepare_helm_release.sh \
    -n pulsar-dev \
    -k pulsar-dev \
    -c            \
    --symmetric
```

This will output commands that you can use to see the auth private key for use in clients.

```bash
    helm install pulsar-dev -f flow_labs_charts/dev/dev.yaml apache/pulsar   
```

Warning. On the first install, initialization can take up to 10 minutes. 



# Updating or changing deployment config
Before running this, you might have to manually delete some things depending on what changes you made. See the delete section below. If you upgrade it and things aren't working properly, it is likely that it didn't deploy properly because of existing stuff from the previous deployment.
```bash
    helm upgrade pulsar-dev -f flow_labs_charts/dev/dev.yaml apache/pulsar   
```


# Usage

You will need to provide the jwt secret token in the header of all requests when running in symmetric mode which this is set to. The jwt tokens are stored in secrets on the cluster. 

There are helper scripts in the scripts/pulsar directory that can help you view and generate jwt tokens that can be used for clients to connect. For example, the following command will show you the admin jwt token:
```bash
  scripts/pulsar/get_token.sh -n pulsar-dev -k pulsar-dev -r admin
```

You can copy this token and then add the following to the header that your client uses to connect to the server:
``` 
{"Authorization": "Bearer <your token here>"}
```

## Prometheus and Grafana
Loadbalancers will be automatically deployed for prometheus and grafana. You can find their external ip using:
```bash
    kubectl get services
```

Route 53 has been configured to forward pulsar-dev-grafana.flowlabs.info and pulsar-dev-prometheus.flowlabs.info to the external ip. However if the ip changes then these will need to be reset.
The default username and password for grafana is in the yaml file. It is currently admin:abcd1234. Be sure to log in and change the password after deploying.

# Deleting
```bash
    helm delete pulsar-dev
```

if you modify any persistent volume claim stuff you will need to delete the old ones using something like
```bash
    kubectl delete pvc --all -n pulsar-dev
```

if you modify anything related to tls or authentication, make sure you manually delete the secrets before reinstalling. Use a command like:
```bash
    kubectl delete secrets --all -n pulsar-dev      
```

You might also want to manually delete the pods
