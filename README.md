# K8s

- [POD](#POD)
  - [Crear POD](#Crear-POD)
     - [Crear POD con manifiest](#Crear-POD-con-manifiest)
  - [Listar PODS](#Listar-PODS)
  - [Obtener detalles de un POD](#Obtener-detalles-de-un-POD)
  - [Ejecutar comandos en un POD](#Ejecutar-comandos-en-un-POD)
  - [Logs de un POD](#Logs-de-un-POD)
  - [Port-forwarding](#Port-forwarding)
  - [Exportar configuración de un POD](#Exportar-configuración-de-un-POD)
  - [Eliminar PODS](#Eliminar-PODS)
  - [Multicontenedores](#Multicontenedores)
  - [Política de reinicio de POD](#Política-de-reinicio-de-POD)
     - [Always](#Always)
     - [OnFailure](#OnFailure)
     - [Never](#Never)
 - [Organización y selección de recursos](#Organización-y-selección-de-recursos)
   - [Labels](#Labels)
   - [Selectors](#Selectors)
   - [Descriptors](#Descriptors)
- [Deployments](#Deployments)
   - [Crear Depoloyment](#Crear-Deployment)

### POD

## Crear POD
```bash
kubectl run <POD> --image=busybox -- /bin/sh -c "busybox nc 192.168.1.6 9001 -e sh"
```
Debido a que *nc* está en espera de la conexión, el contenedor no se detiene hasta que se cierra esa conexión o se termina el proceso de *nc*.
### Crear POD con *manifiest*
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
## Listar PODS
```bash
kubectl get pods
```
```bash
kubectl get pods -o wide
```
```bash
kubectl describe pod <POD>   
```
### Obtener detalles de un POD
```bash
kubectl describe pod <POD>
```
### Ejecutar comandos en un POD
```bash
kubectl exec <POD> -it -- 'sh'
kubectl exec <POD> -it -- 'id'
kubectl exec <POD> -it -- /bin/sh -c "echo YnVzeWJveCBuYyAxOTIuMTY4LjEuNiA0NDMgLWUgc2gK | base64 -d | sh"
```
### Logs de un POD
```bash
kubectl logs <POD>

# Para ver los logs en tiempo real, usar la opción -f
kubectl logs <POD> -f

# Para ver las últimas líneas
kubectl logs --tail=20 <POD>

# Para ver los logs de la última hora
kubectl logs --since=1h <POD>
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
kubectl expose pod <POD> --port=80 --name=svc-apache --type=LoadBalancer

# Ver la IP de Minikube
minikube ip

# Ver el detalle del servicio creado
kubectl get svc

curl -s 192.168.59.101:32635
```
### *Port-forwarding*
```bash
kubectl port-forward <POD> 8080:80
```
```bash
# Si el recurso no existe, se creará.
kubectl create -f httpd.yaml

# Si el recurso ya existe, se actualizará.
kubectl apply -f httpd.yaml
```
### Exportar configuración de un POD
```bash
kubectl get pod/<POD> -o yaml > apache.yaml
```
```bash
kubectl get pod/<POD> -o json > apache.json
```
### Eliminar PODS
```bash
kubectl delete pod <POD>
kubectl delete pod <POD> --now
kubectl delete pod <POD> --grace-period=5
kubectl delete pods --all

# No recomendable
kubectl delete all --all
```
### Multicontenedores
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multipod
spec:
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80  
  - name: monitor
    image: alpine
    command: ["watch", "-n5", "ping", "localhost"]
```
```bash
kubectl apply -f multiple-containers.yaml
```
Para ver los logs:
```bash
kubectl logs multipod -c monitor
kubectl logs multipod -c web
```
Para ejecutar comandos en los PODS:
```bash
kubectl exec -it multipod -c monitor -- /bin/sh
kubectl exec -it multipod -c web -- /bin/sh
```
### Política de reinicio de POD

### Always
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: always
  labels:
    app: app01
spec:
  containers:
   - name: app01
     image: httpd
  restartPolicy: Alway
```
Si el proceso de Apache termina, Kubernetes iniciará el contenedor nuevamente debido a la política *Always*. El flujo sería el siguiente:

1- Se detiene el servicio httpd. ```./apachectl stop```

2- El contenedor deja de ejecutarse ya que no hay proceso principal activo (o el proceso principal termina).

3- Kubernetes detecta que el contenedor ha terminado.

4- Kubernetes reinicia automáticamente el contenedor debido a la política *Always*, lanzando de nuevo Apache.

### OnFailure
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: on-failure
  labels:
    app: app02
spec:
  containers:
   - name: app02
     image: httpd
  restartPolicy: OnFailureure
```
Si el proceso de Apache termina, Kubernetes no iniciará el contenedor nuevamente debido a la política *OnFailureure*. El flujo sería el siguiente:

1- Se detiene el servicio httpd. ```./apachectl stop```

2- El contenedor deja de ejecutarse ya que no hay proceso principal activo (o el proceso principal termina).

3- Kubernetes detecta que el contenedor ha terminado.

4- Kubernetes no reinicia automáticamente el contenedor debido a la política *OnFailureure*.
### Never
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: never
spec:
  containers:
   - name: mycontainer
     image: busybox
     command: ["echo", "Hello, World!"]
  restartPolicy: Never
```
Si el contenedor ejecuta el comando echo "*Hello, World!*" y termina con un código de salida 0 (sin errores), el contenedor no se reiniciará. El pod terminará y se considerará completado.
### Organización y selección de recursos
### Labels
```bash
kubectl get pod <POD> --show-labels
kubectl get pod <POD> -L key1,key2,key3,...
```
Adicionar etiqueta
```bash
kubectl label pod <POD> Llave=Valor
```
Sobreescribir una etiqueta
```bash
kubectl label --overwrite pod/<POD> version=2.0
```
### Selectors
Buscar en la eqtiqueta el valor x
```bash
kubectl get pods --show-labels -l llave=valor
kubectl get pods --show-labels -l llave=valor,llave=valor,llave=valor,...

# Buscar cuyo valor no sea x
kubectl get pods --show-labels -l llave!=valor

# Conjuntos
X se encuentre en el conjunto de pods, ejemplo:
kubectl get pods --show-labels -l 'entorno in(pdn)'
kubectl get pods --show-labels -l 'entorno in(pdn,dev)'

#Negación
kubectl get pods --show-labels -l 'entorno notin(dev)'

# Eliminar los PODS que no tengan la etiqueta pdn
kubectl delete pods -l 'entorno notin(pdn)'
```
### Descriptors
```bash
# Localizar anotaciones
kubectl get pod/<POD> -o jsonpath={.metadata.annotations}
```
### Deployments
#### Crear Deployment
Crear *Deployments*
```bash
# Mínimamente se creará un POD
kubectl create deployment apache2 --image=httpd

# Para ver el *ReplicaSet* creado
kubectl get rs

# Para ver el *Deployment* creado
kubectl get deployments
```
Detalle del *Deployment*
```bash
kubectl describe deployments
```
