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
## Obtener logs de un POD
```bash
kubectl logs pod1
```
