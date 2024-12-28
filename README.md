# K8s

### POD

## Crear un POD
```bash
kubectl run pod1 --image=busybox -- /bin/sh -c "busybox nc 192.168.1.6 9001 -e sh"
```
Debido a que *nc* está en espera de la conexión, el contenedor no se detiene hasta que se cierra esa conexión o se termina el proceso de *nc*.

## Obtener los PODS
```bash
kubectl get pods
```
```bash
kubectl get pods -o wide
```
```bash
kubectl describe pod pod1   
```
```bash
Name:         pod1
Namespace:    default
Priority:     0
Node:         minikube/192.168.59.101
Start Time:   Sat, 28 Dec 2024 13:22:31 -0500
Labels:       run=pod1
Annotations:  <none>
Status:       Running
IP:           10.244.0.21
IPs:
  IP:  10.244.0.21
[...]
```
## Obtener logs de un POD
```bash
kubectl logs pod1
```
