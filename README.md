# Devops-Kubernetes

Este proyecto demuestra como se puede desplegar una app como servicio y escalarla por pods con GCP cn una Kubernetes Engine (GKE) que es el servicio de kubernetes de GCP.

Requisitos:
-----------
1. Crear una cuenta de google cloud, si no la tienes puedes crearla en https://cloud.google.com/
2. Instalación de Google Cloud SDK: Instalar y configurar el SDK de Google Cloud. Puedes seguir esta guía para instalarla: https://cloud.google.com/sdk/docs/install y kubectl https://kubernetes.io/docs/tasks/tools/
3. Configuración de acceso a Google Cloud: Configura tu CLI de Google Cloud ejecutando:

   ```gcloud init```

Creación del cluster de Kubernetes en Google Cloud
------------
Paso 1: Crear un cluster Kubernetes usando Google Kubernetes Engine (GKE)
1. Accede a la consola de Google Cloud:
Ve a la consola de Google Cloud y y busca **Kubernetes Engine**
2. Crear un nuevo cluster GKE:
   - En la consola de GKE, haz clic en **Crear Cluster**.
   - Completa los detalles:
     - **Cluster name**: mi-cluster-gke
     - **Location**: Selecciona la zona o región más cercana. (por ejemplo us-central1)
     - **Machine type**: Puedes usar el tipo predeterminado `e2-medium` para los nodos.
     - **Node count**: Comienza con 2 nodos.
3. Crear el cluster:
   - Haz clic en **Crear** y espera unos minutos hasta que el cluster se cree.

Paso 2: Configurar kubectl para acceder al cluster GKE
-------------
1. Obtener credenciales de GKE:

Pedira tener el plugin de auth de GCP: ```gcloud components install gke-gcloud-auth-plugin```
   
   Ejecuta el siguiente comando para obtener las credenciales de acceso del cluster GKE:
   ```bash
   gcloud container clusters get-credentials mi-cluster-gke --zone <zona> --project <tu-proyecto>
   ```

3. Verificar la conexión al cluster:
   Ejecuta el siguiente comando para verificar que kubectl está conectado correctamente al cluster:
   ```bash
   kubectl get nodes
   ```
Te dira No resources found debido a que aún no se han desplegado servicios.

Hacer deploy de una aplicación
------------
Paso 1: Crear un Deployment
------------
1. Crear un archivo YAML para el Deployment:
   Crea un archivo llamado `deployment.yaml` con el siguiente contenido:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mi-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: mi-app
     template:
       metadata:
         labels:
           app: mi-app
       spec:
         containers:
         - name: mi-app
           image: yamitan/mi-proyecto:latest
           ports:
           - containerPort: 3000
   ```
Configura tu imagen y port de acuerdo a tu aplicación.
   
Si tienes alguna variable de entorno puedes agregarlo asi:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mi-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: mi-app
     template:
       metadata:
         labels:
           app: mi-app
       spec:
         containers:
         - name: mi-app
           image: yamitan/mi-proyecto:latest
           ports:
           - containerPort: 3000
           env:
           - name: secret1
             value: "value"
           - name: secret2
             value: "value"
   ```
2. Desplegar el Deployment:
   Aplica el archivo YAML:
   ```bash
   kubectl apply -f deployment.yaml
   ```

Paso 2: Exponer el Deployment como un Servicio
--------------
1. Crear un archivo YAML para el Servicio:
   Crea un archivo llamado `service.yaml` con el siguiente contenido:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: mi-app-service
   spec:
     selector:
       app: mi-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 3000
     type: LoadBalancer
   ```

2. Aplicar el Servicio:
   Aplica el archivo YAML:
   ```bash
   kubectl apply -f service.yaml
   ```

Escalar la aplicación
-------------
Paso 1: Escalar el Deployment
-------------
1. Para escalar el número de Pods en tu aplicación, ejecuta:
   ```bash
   kubectl scale deployment mi-app --replicas=3
   ```

2. Verifica el nuevo número de Pods con:
   ```bash
   kubectl get pods
   ```

Probar tu aplicación:
--------------
Ejecuta:
  ```bash
  kubectl get svc mi-app-service
  ```
Ahora puedes hacer curl con el External-IP por ejemplo ```curl http://30.23...```

Limpiar recursos
--------------
1. Para eliminar el Deployment y el Servicio, ejecuta:
   ```bash
   kubectl delete -f deployment.yaml
   kubectl delete -f service.yaml
   ```

2. Eliminar el cluster:
   Si ya no necesitas el cluster, puedes eliminarlo desde la consola de Google Cloud o usando la CLI de Google Cloud:
   ```bash
   gcloud container clusters delete mi-cluster-gke --zone <zona> --project <tu-proyecto>
   ```
¿Que se podría mejorar?
-------------
| Clave | Descripción |
| --- | --- |
| Auto escalamiento | Actualmente el la app como servicio está solo se escala de forma manual, pero un escalamiento automático dependiendo de la demanda sería lo ideal. |
| Monitoreo | Actualmente solo está el monitoreo básico por default de GCP y no uno personalizado a medida de la aplicación. |
| Seguridad | Actualmente solo está la barrera de que solo se puede ingresar a la instancia por la cuenta de GCP pero lo ideal sería tener métodos adicionales como verificación en 2 pasos y una password en la propia VM. |

Que se aprendió
---------------
- Escalamiento de aplicaciones como servicio en GCP de forma horizontal.
- Despliegue de aplicaciones con kubernetes en GPC.
- Configuración inicial para usar kubernetes.
- Gestión de recursos en GCP.
