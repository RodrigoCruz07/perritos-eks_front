# Tienda de Perritos - Capa Frontend

Este repositorio contiene el código fuente y las configuraciones de despliegue para la interfaz de usuario de la plataforma "Tienda de Perritos". La aplicación está diseñada como un microservicio desacoplado que interactúa con la capa de backend a través de peticiones HTTP/REST.

## Contenido del Repositorio
* `/src`: Código fuente de la aplicación web.
* `Dockerfile`: Archivo de configuración para la construcción de la imagen OCI (Docker).
* `/.github/workflows/deploy.yml`: Pipeline de integración y despliegue continuos (CI/CD).
* `/k8s`: Manifiestos de Kubernetes (`deployment.yaml`, `service.yaml`, `hpa.yaml`).

## Funcionamiento Arquitectónico
La capa Frontend se despliega de manera contenerizada dentro de un clúster de **Amazon EKS**, distribuida de forma redundante en subredes privadas a través de dos Zonas de Disponibilidad (Multi-AZ) para garantizar alta disponibilidad.

1. **Punto de Entrada:** El tráfico externo de los usuarios es recibido por un **Application Load Balancer (ALB)** en la subred pública, el cual actúa como proxy inverso.
2. **Distribución:** El ALB redirige las peticiones hacia los Pods de la capa Frontend en las subredes privadas.
3. **Consumo de API:** El Frontend procesa la interfaz y realiza peticiones asíncronas hacia la capa de Backend expuesta internamente en el puerto `8080`.

## Autoescala (Elasticidad)
La disponibilidad de este componente está regulada por un **Horizontal Pod Autoscaler (HPA)**. El HPA monitorea el consumo promedio de CPU; al superar el umbral del 70%, incrementa dinámicamente el número de réplicas de Pods (mínimo 2, máximo 10) para absorber la demanda de red.

## Automatización CI/CD (GitHub Actions)
El flujo automatizado se ejecuta ante cada evento `push` en la rama `main`:
1. **Build:** Compila el código fuente y genera la imagen Docker optimizada.
2. **Push:** Autentica con AWS CLI y envía la imagen etiquetada de forma única a **Amazon ECR**.
3. **Deploy:** Actualiza el deployment en el clúster EKS mediante la estrategia de actualización progresiva (`kubectl set image`), garantizando un tiempo de inactividad cero (Zero-Downtime).
