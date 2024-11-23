# **Desplegando tu Servicio en el Clúster de Kubernetes en Azure**

Esta guía proporciona todos los pasos necesarios para desplegar tu servicio en nuestro clúster compartido de Kubernetes en Azure e interactuar con otros servicios dentro del clúster.

---

## **Requisitos Previos**
Antes de comenzar, asegúrate de tener las siguientes herramientas instaladas localmente:
1. **kubectl**: Herramienta CLI para Kubernetes.
    - Guía de instalación: [kubectl Install Guide](https://kubernetes.io/docs/tasks/tools/#kubectl)
2. **Azure CLI**: Herramienta CLI para interactuar con Azure.
    - Guía de instalación: [Azure CLI Install Guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
3. **Docker**: Para construir imágenes de contenedores de tu servicio.
    - Guía de instalación: [Docker Install Guide](https://docs.docker.com/get-docker/)
4. **Acceso a nuestro Azure Kubernetes Service (AKS)**:
    - Asegúrate de haber sido añadido al proyecto de Azure y tener acceso al clúster de AKS.

---

## **Paso 1: Clona este Repositorio**
Clona este repositorio, que contiene todos los archivos necesarios para desplegar tu servicio:

```bash
git clone https://github.com/our-org/k8s-deployment-guide.git
cd k8s-deployment-guide
```

---

## **Paso 2: Prepara tu Servicio para el Despliegue**
Tu servicio debe cumplir con los siguientes requisitos:

### **1. Servicio Dockerizado**
- Tu servicio debe tener un archivo `Dockerfile` en el directorio raíz.
- Construye y sube tu imagen Docker a un registro de contenedores (por ejemplo, Azure Container Registry (ACR) o DockerHub).

#### **Pasos para Subir a un Registro:**
1. **Construir la Imagen Docker**:
   ```bash
   docker build -t <tu-registro>/<tu-nombre-del-servicio>:<versión> .
   ```

2. **Subir la Imagen Docker**:
   ```bash
   docker push <tu-registro>/<tu-nombre-del-servicio>:<versión>
   ```

3. **Ejemplo en Azure ACR**:
   Si usas Azure Container Registry:
   ```bash
   az acr login --name <nombre-del-registro>
   docker tag <nombre-del-servicio>:<versión> <acr-nombre>.azurecr.io/<nombre-del-servicio>:<versión>
   docker push <acr-nombre>.azurecr.io/<nombre-del-servicio>:<versión>
   ```

---

## **Paso 3: Configura el Despliegue en Kubernetes**

1. **Editar `deployment.yaml`**:
   Reemplaza los marcadores de posición en el archivo `deployment.yaml`:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
      name: <nombre-del-servicio>
   spec:
      replicas: 2
      selector:
         matchLabels:
            app: <nombre-del-servicio>
      template:
         metadata:
            labels:
               app: <nombre-del-servicio>
         spec:
            containers:
               - name: <nombre-del-servicio>
                 image: <tu-registro>/<tu-nombre-del-servicio>:<versión>
                 ports:
                    - containerPort: 8080
   ```

2. **Editar `service.yaml`**:
   Define un servicio de Kubernetes para exponer tu despliegue:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
      name: <nombre-del-servicio>
   spec:
      selector:
         app: <nombre-del-servicio>
      ports:
         - protocol: TCP
           port: 80
           targetPort: 8080
      type: ClusterIP
   ```

---

## **Paso 4: Manejar Variables de Entorno con Secrets y ConfigMaps**

### **1. ConfigMaps (Datos No Sensibles)**
Usa ConfigMaps para almacenar datos de configuración no sensibles, como endpoints de API o banderas de características.

#### **Crear un ConfigMap**
1. Define tu ConfigMap en `configmap.yaml`:

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
      name: mi-configmap
   data:
      FEATURE_FLAG: "true"
      API_ENDPOINT: "https://api.example.com"
   ```

2. Aplica el ConfigMap:
   ```bash
   kubectl apply -f configmap.yaml
   ```

3. Usa el ConfigMap en tu despliegue (`deployment.yaml`):
   ```yaml
   env:
      - name: FEATURE_FLAG
        valueFrom:
           configMapKeyRef:
              name: mi-configmap
              key: FEATURE_FLAG
      - name: API_ENDPOINT
        valueFrom:
           configMapKeyRef:
              name: mi-configmap
              key: API_ENDPOINT
   ```

---

### **2. Secrets (Datos Sensibles)**
Usa Secrets para almacenar datos sensibles como contraseñas, claves de API o credenciales de base de datos.

#### **Crear un Secret**
1. Define tu Secret en `secret.yaml`:
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
      name: mi-secret
   type: Opaque
   data:
      sensitive-key: dXNlcm5hbWU=  # Valor codificado en Base64
   ```

2. Aplica el Secret:
   ```bash
   kubectl apply -f secret.yaml
   ```

3. Usa el Secret en tu despliegue (`deployment.yaml`):
   ```yaml
   env:
      - name: SENSITIVE_ENV
        valueFrom:
           secretKeyRef:
              name: mi-secret
              key: sensitive-key
   ```

---

## **Paso 5: Despliega tu Servicio en el Clúster**

1. Conéctate al Clúster de AKS:
   ```bash
   az login
   az aks get-credentials --resource-group <grupo-de-recursos> --name <nombre-del-clúster>
   ```

2. Aplica los archivos de despliegue:
   ```bash
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   ```

3. Verifica el despliegue:
   ```bash
   kubectl get pods
   kubectl get services
   ```

---

## **Paso 6: Acceso a tu Servicio**

### **Pruebas Locales: Port-forwarding**
Puedes acceder a tu servicio localmente utilizando `kubectl port-forward`.

1. Encuentra el nombre de tu pod:
   ```bash
   kubectl get pods
   ```

2. Redirige un puerto local al pod:
   ```bash
   kubectl port-forward <nombre-del-pod> 8080:8080
   ```

3. Accede al servicio en `http://localhost:8080`.

---

## **Paso 7: Actualiza tu Servicio**
Cuando actualices tu servicio, repite los siguientes pasos:
1. Construye y sube una nueva versión de tu imagen Docker:
   ```bash
   docker build -t <tu-registro>/<tu-nombre-del-servicio>:<nueva-versión> .
   docker push <tu-registro>/<tu-nombre-del-servicio>:<nueva-versión>
   ```

2. Actualiza el despliegue para usar la nueva imagen:
   ```bash
   kubectl set image deployment/<nombre-del-servicio> <nombre-del-servicio>=<tu-registro>/<tu-nombre-del-servicio>:<nueva-versión>
   ```

3. Verifica la actualización:
   ```bash
   kubectl rollout status deployment/<nombre-del-servicio>
   ```
   
## **Workflows de GitHub**

Si deseas realizar un Despliegue Continuo (Continuous Deployment), revisa el archivo [k8-deployment.yml](https://github.com/TeachMe-CloudUS/teachme-student-service/blob/k8/.github/workflows/k8-deployment.yml) del repositorio `teachme-student-service`.
