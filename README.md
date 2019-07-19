
### Instruction for manual setup CPI with OpenStack

#### In-tree cloud provider integration

On Master and Worker nodes:

1. Generate `/etc/kubernetes/cloud-config` file

2. Add to `/etc/sysconfig/kubelet`
```
KUBELET_EXTRA_ARGS="--cloud-provider=openstack --cloud-config=/etc/kubernetes/cloud-config"
```

On Master nodes:
1. Add `/etc/kubernetes/manifests/kube-apiserver.yaml` and `/etc/kubernetes/manifests/kube-controller-manager.yaml`
```
    - --cloud-provider=openstack
    - --cloud-config=/etc/kubernetes/cloud-config
```
```
    - mountPath: /etc/kubernetes/cloud-config
      name: cloud-config
      readOnly: true
```
```
  - hostPath:
      path: /etc/kubernetes/cloud-config
      type: FileOrCreate
    name: cloud-config
```

On Master and Worker nodes:
1. Restart kubelet service
```
systemctl restart kubelet
```

#### Out-tree cloud provider integration

On Master and Worker nodes:

1. Add to `/etc/sysconfig/kubelet`
```
KUBELET_EXTRA_ARGS="--cloud-provider=external"
```
2. Restart kubelet service
```
systemctl restart kubelet
```

On Master node:

1. Update and copy `cloud-config` to `/etc/kubernetes/cloud-config`

2. Add `"- --cloud-provider=external"` to configuration to:
```
/etc/kubernetes/manifests/kube-apiserver.yaml
/etc/kubernetes/manifests/kube-controller-manager.yaml
```

3. Restart kubelet service
```
systemctl restart kubelet
```

4. Copy manifests folder on Master node:

5. Create `cloud-config` secret
```
export CLOUD_CONFIG=/etc/kubernetes/cloud-config
kubectl create secret -n kube-system generic cloud-config --from-literal=cloud.conf="$(cat $CLOUD_CONFIG)" --dry-run -o yaml > manifests/cloud-config-secret.yaml
kubectl apply -f manifests/cloud-config-secret.yaml
```

6. Create cloud-controller-manager ServiceAccount
```
kubectl apply -f manifests/cpi-service-account.yaml
```

7. Create RBAC for cloud-controller-manager
```
kubectl apply -f manifests/rbac/cloud-controller-manager-roles.yaml
kubectl apply -f manifests/rbac/cloud-controller-manager-role-bindings.yaml

```

8. Deploy cloud-controller-manager pod
```
kubectl apply -f manifests/cloud-controller-manager.yaml
```

9. Check logs of cloud-controller-manager
```
kubectl logs -n kube-system cloud-controller-manager
```

8. Deploy demo examples
