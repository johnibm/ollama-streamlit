
## Configure GPU time-slicing in OpenShift

### Doc
https://docs.nvidia.com/datacenter/cloud-native/openshift/latest/time-slicing-gpus-in-openshift.html

### Create ConfigMap
oc create -f device-plugin-config.yaml

### Patch policy to use this 

oc patch clusterpolicy gpu-cluster-policy \
    -n nvidia-gpu-operator --type merge \
    -p '{"spec": {"devicePlugin": {"config": {"name": "device-plugin-config"}}}}'

### Label GPU nodes

oc label --overwrite node --selector=nvidia.com/gpu.product=NVIDIA-L40S nvidia.com/device-plugin.config=NVIDIA-L40S

### Validate
oc get node --selector=nvidia.com/gpu.product=NVIDIA-L40S -o json | jq '.items[0].status.capacity'
oc get node --selector=nvidia.com/gpu.product=NVIDIA-L40S-SHARED -o json | jq '.items[0].status.capacity'
oc get node --selector=nvidia.com/gpu.product=NVIDIA-L40S-SHARED -o json  | jq '.items[0].metadata.labels' | grep nvidia

### Update MachineSet 

oc get machineset fusion101-8jxlg-gpu-us-south-1 -n openshift-machine-api --show-labels

NAME                             DESIRED   CURRENT   READY   AVAILABLE   AGE   LABELS
fusion101-8jxlg-gpu-us-south-1   1         1         1       1           18h   machine.openshift.io/cluster-api-cluster=fusion101-8jxlg,machine.openshift.io/cluster-api-machine-role=worker,machine.openshift.io/cluster-api-machine-type=worker


oc patch machineset fusion101-8jxlg-gpu-us-south-1 -n openshift-machine-api --type merge \
    --patch '{"spec": {"template": {"spec": {"metadata": {"labels": {"nvidia.com/device-plugin.config": "NVIDIA-L40S"}}}}}}'

