### Prueba de concepto de insertar un contenedor sobre un Pod existen
![Descripción de la imagen](https://raw.githubusercontent.com/0x04e1/K8s/refs/heads/main/images/PoC1.png)

# Creación del Deploy de MariaDB

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
          value: "rootpass"
        - name: MARIADB_DATABASE
          value: "exampledb"
        - name: MARIADB_USER
          value: "exampleuser"
        - name: MARIADB_PASSWORD
          value: "examplepass"

```
