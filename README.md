# Nagios en Kubernetes - Guía de Despliegue

Esta guía te llevará paso a paso por el proceso de desplegar Nagios en un cluster de Kubernetes, utilizando archivos YAML para definir el Deployment, el Service y los Persistent Volume Claims (PVCs) necesarios. Nagios es una potente herramienta de monitorización que te permitirá supervisar tu infraestructura.

## Archivos de Configuración

1. [deployment-ng.yaml](#1-deployment-yaml---deployment-ngyaml): Define el despliegue de Nagios.
2. [nagios-config-pvc.yaml](#2-persistent-volume-claim-yaml-para-configuración---nagios-config-pvcyaml): Define el PVC para la configuración de Nagios.
3. [nagios-data-pvc.yaml](#3-persistent-volume-claim-yaml-para-datos---nagios-data-pvcyaml): Define el PVC para los datos de Nagios.
4. [service-ng.yaml](#4-service-yaml---service-ngyaml): Define el servicio para exponer Nagios.

**En caso de ya tener los archivos de configuracion, esto son los pasos de despliegue** [PASOS](#pasos-para-desplegar-nagios)

### 1. Deployment YAML - `deployment-ng.yaml`

Este archivo define el despliegue de Nagios, especificando el contenedor, sus variables de entorno y los volúmenes necesarios.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nagios
  labels:
    app: nagios
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nagios
  template:
    metadata:
      labels:
        app: nagios
    spec:
      containers:
      - name: nagios
        image: jasonrivers/nagios:latest
        ports:
        - containerPort: 80
        env:
        - name: NAGIOSADMIN_USER
          value: "admin"
        - name: NAGIOSADMIN_PASS
          value: "password"
        volumeMounts:
        - name: nagios-config
          mountPath: /opt/nagios/etc
        - name: nagios-data
          mountPath: /opt/nagios/var
      volumes:
      - name: nagios-config
        persistentVolumeClaim:
          claimName: nagios-config-pvc
      - name: nagios-data
        persistentVolumeClaim:
          claimName: nagios-data-pvc
```

### 2. Persistent Volume Claim YAML para Configuración - `nagios-config-pvc.yaml`

Este archivo define un PVC para almacenar de manera persistente la configuración de Nagios.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nagios-config-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### 3. Persistent Volume Claim YAML para Datos - `nagios-data-pvc.yaml`

Este archivo define un PVC para almacenar de manera persistente los datos de Nagios.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nagios-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### 4. Service YAML - `service-ng.yaml`

Este archivo define un servicio de Kubernetes para exponer Nagios, permitiendo el acceso a través de un LoadBalancer.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nagios-service
spec:
  selector:
    app: nagios
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

## Pasos para Desplegar Nagios

### Paso 1: Crear los Persistent Volume Claims

Aplica los archivos YAML de los Persistent Volume Claims para crear el almacenamiento necesario:

```sh
kubectl apply -f nagios-config-pvc.yaml
kubectl apply -f nagios-data-pvc.yaml
```

### Paso 2: Crear el Deployment de Nagios

Aplica el archivo YAML del Deployment para desplegar Nagios:

```sh
kubectl apply -f deployment-ng.yaml
```

### Paso 3: Crear el Service para Exponer Nagios

Aplica el archivo YAML del Service para crear un LoadBalancer que exponga Nagios:

```sh
kubectl apply -f service-ng.yaml
```

### Paso 4: Acceder a Nagios

Una vez desplegado, puedes acceder a Nagios a través de la IP externa proporcionada por el LoadBalancer. Usa las credenciales `admin` y `password` para iniciar sesión.

Para obtener la IP externa, puedes usar el siguiente comando:

```sh
kubectl get services nagios-service
```

Busca la columna `EXTERNAL-IP` en la salida del comando. 

## Conclusión

Esta guía proporciona una configuración básica para desplegar Nagios en Kubernetes. Puedes ajustar las configuraciones según tus necesidades específicas, como aumentar el almacenamiento, cambiar las credenciales, o ajustar los recursos del contenedor. ¡Feliz monitorización!
