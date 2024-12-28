# K8s

- [POD](#POD)
  - [Crear POD](#Crear-POD)
  - [Listar PODS](#Listar-PODS)
  - [Obtener detalles de un POD](#Obtener-detalles-de-un-POD)
  - [Ejecutar comandos en un POD](#Ejecutar-comandos-en-un-POD)
  - [Logs de un POD](#Logs-de-un-POD)
  - [Port-forwarding](#Port-forwarding)
 
### POD

## Crear POD
```bash
kubectl run pod1 --image=busybox -- /bin/sh -c "busybox nc 192.168.1.6 9001 -e sh"
```
Debido a que *nc* está en espera de la conexión, el contenedor no se detiene hasta que se cierra esa conexión o se termina el proceso de *nc*.

## Listar PODS
```bash
kubectl get pods
```
```bash
kubectl get pods -o wide
```
```bash
kubectl describe pod pod1   
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
### Logs de un POD
```bash
kubectl logs apache

# Para ver los logs en tiempo real, usar la opción -f
kubectl logs apache -f

# Para ver las últimas líneas
kubectl logs --tail=20 apache

# Para ver los logs de la última hora
kubectl logs --since=1h apache
```
### Acceso a través del Proxy
```bash
# Iniciarl el servicio de proxy
kubectl proxy

# Acceso a la API
curl -s http://127.0.0.1:8001/api/v1/namespaces/default/pods/apache/proxy/
```
### Crear un servicio
```bash
kubectl expose pod apache --port=80 --name=svc-apache --type=LoadBalancer

# Ver la IP de Minikube
minikube ip

# Ver el detalle del servicio creado
kubectl get svc

curl -s 192.168.59.101:32635
```
### *Port-forwarding*
```bash
kubectl port-forward apache 8080:80
```
### Crear POD con manifiest
```yml
apiVersion: v1
kind: Pod
metadata:
  name: apche2
  labels:
    so: debian
    account: pdn01
    version: v1
spec:
  containers:
   - name: httpd
     image: httpd
```
```bash
# Si el recurso no existe, se creará.
kubectl create -f httpd.yaml

# Si el recurso ya existe, se actualizará.
kubectl apply -f httpd.yaml
```
