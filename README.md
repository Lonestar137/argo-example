

# Helm

## Vendoring
Download the dependencies into charts/ for a helm project using:

```bash
pushd eks/jupyterhub
  helm dependency update
  helm install my-jupyterhub ./ --namespace jupyterhub --create-namespace
  # NOTE: To uninstall: helm uninstall my-jupyterhub --namespace jupyterhub
  helm status my-jupyterhub -n jupyterhub

  # If there is a failure
  kubectl describe pod <pod-name> -n jupyterhub


  # Redeploy the app with an updated values file
  helm upgrade <release-name> jupyterhub/jupyterhub -n jupyterhub -f values.yaml
popd
```

This what you use to package your application.


### Debugging PVC issues

Check for failed PVC events
```bash
kubectl describe pod <pod-name> -n jupyterhub
```

Check your clusters available storage classes
```bash
kubectl get storageclass
```

Compare to configured storage classes in values.yaml or implicit defaults:
```yaml
juptyerhub:
  hub:
    db:
      pvc:
        storageClassName: "standard"
        #create: true
        #storage: 1Gi 
```

The StorageClass should have a provisioner, **ebs.csi.aws.com** for AWS.


You can also create a manual partition, **pv.yaml**:
```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jupyterhub-hub-db-pv
spec:
  capacity:
    storage: 1Gi  # Match the PVC’s requested size
  accessModes:
    - ReadWriteOnce  # Match the PVC’s access mode
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ""  # Or match the PVC’s StorageClass
  hostPath:
    path: /data/jupyterhub  # For testing; use real storage in production
```

`kubectl apply -f pv.yaml`

Check quotas:
```bash
kubectl get quota -n jupyterhub
```


