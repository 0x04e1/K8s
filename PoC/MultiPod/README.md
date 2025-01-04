### Prueba de concepto de insertar un contenedor sobre un Pod existen
![Descripción de la imagen](https://raw.githubusercontent.com/0x04e1/K8s/refs/heads/main/images/PoC1.png)

### Creación del *Deploy* de MariaDB:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb
        image: mariadb:latest
        ports:
        - containerPort: 3306
        env:
        - name: MARIADB_ROOT_PASSWORD
          value: "CGs66L4@<56#!"
        - name: MARIADB_DATABASE
          value: "wordpress-db"
        - name: MARIADB_USER
          value: "user01-db"
        - name: MARIADB_PASSWORD
          value: "P@ssw0rd!"
```

### Creación del *Deploy* de WordPress:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest
        ports:
        - containerPort: 80
        env:
        - name: WORDPRESS_DB_HOST
          value: "mariadb-cluster-ip-service"
        - name: WORDPRESS_DB_USER
          value: "user01-db"
        - name: WORDPRESS_DB_PASSWORD
          value: "P@ssw0rd!"
        - name: WORDPRESS_DB_NAME
          value: "wordpress-db"
```
⚠️ El valor de la variable *WORDPRESS_DB_HOST* hará referencia al registro del *ClusterIP* para alcanzar al Pod de MariaDB.

### Creación del *ClusterIP* de MariaDB:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mariadb-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    app: mariadb
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```

### Creación del ClusterIP de WordPress:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    app: wordpress 
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
### Creación del *Ingress* contea el *ClusterIP* de WordPress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wordpress-ingress
spec:
  rules:
  - host: wordpress.local.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wordpress-cluster-ip-service
            port:
              number: 80
```
Al adicionar otro Pod, y al compartir el mismo *network namespace*, es posible que desde el nuevo Pod, podamos leer la información que pasa por el Pod original en este caso.

![Descripción de la imagen](https://raw.githubusercontent.com/0x04e1/K8s/refs/heads/main/images/PoC2.png)

Como posible resultado:

![Descripción de la imagen](https://raw.githubusercontent.com/0x04e1/K8s/refs/heads/main/images/PoC3.png)
