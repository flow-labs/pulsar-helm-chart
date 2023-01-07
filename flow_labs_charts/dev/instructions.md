# installation
## Create cluster
**warning - make sure you use a cluster that is exclusive to this pulsar install to avoid accidentally deleting or modifying some other important stuff**

Do this in the eks UI

You will need to install the Amazon EBS CSI Driver for persistent storage. This can be added by clicking the add-ons tab in the eks UI.


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
# Updating or changing deployment config
```bash
    helm upgrade pulsar-dev -f flow_labs_charts/dev/dev.yaml apache/pulsar   
```


# Deleting
```bash
    helm delete pulsar-dev
```

if you modify any persistent volume claim stuff you will need to delete the old ones using something like
```bash
    kubectl delete pvc --all -n pulsar-dev
```

You might also want to manually delete the pods
