# docker-cicd-AWS

Aplicación Flask de demostración, containerizada con Docker y preparada para despliegue continuo en AWS mediante GitHub Actions, Amazon ECR y Amazon ECS.

## Visión general

Este proyecto muestra un flujo DevOps básico y reproducible para:

- construir una imagen Docker multi-stage,
- escanear la base del proyecto con Trivy,
- publicar la imagen en Amazon ECR,
- desplegar la nueva versión en un servicio de Amazon ECS.

## Stack tecnológico

| Componente | Tecnología |
|------------|------------|
| Backend | Python 3.14, Flask, Gunicorn |
| Contenedores | Docker multi-stage |
| Integración continua | GitHub Actions |
| Infraestructura cloud | Amazon ECR + Amazon ECS |

## Estructura del proyecto

```text
.
├── app/
│   ├── __init__.py
│   ├── routes.py
│   ├── demo_data.py
│   ├── static/
│   └── templates/
├── .github/workflows/
│   └── ci-cd.yaml
├── Dockerfile
├── requirements.txt
├── run.py
├── wsgi.py
└── README.md
```

## Rutas principales

| Ruta | Descripción |
|------|-------------|
| `/` | Dashboard principal |
| `/productos` | Listado de productos |
| `/clientes` | Listado de clientes |
| `/pedidos` | Listado de pedidos |
| `/health` | Endpoint de salud para comprobaciones del contenedor |
| `/runtime` | Información del entorno de ejecución |

## Requisitos previos

Antes de usar este proyecto en AWS, asegúrate de contar con:

- Python 3.11+ o 3.14 (según el Dockerfile)
- Docker instalado y en ejecución
- Una cuenta de AWS con permisos para ECR y ECS
- Un repositorio GitHub conectado a Actions
- Un repositorio ECR creado
- Un cluster y servicio ECS configurados
- Un archivo de definición de tarea de ECS compatible con esta aplicación

## Cómo copiar este proyecto

Si quieres trabajar con el código localmente, sigue estos pasos:

```bash
git clone <url-del-repositorio>
cd docker-cicd-azure
```

Si prefieres copiarlo sin usar Git, descarga el proyecto como ZIP y descomprímelo en la carpeta deseada.

## Qué configurar antes de correrlo

### 1. Configuración local

Asegúrate de tener instalado:

- Python
- pip
- Docker

### 2. Configuración de AWS

Antes de desplegar en AWS, configura lo siguiente:

- crea un repositorio en Amazon ECR,
- crea un rol de IAM para GitHub Actions,
- habilita el acceso a ECR y ECS desde ese rol,
- define el cluster y el servicio ECS que usarás,
- ajusta los valores del workflow con tus propios nombres y regiones.

### 3. Configuración de GitHub Actions

Para la autenticación con AWS, este proyecto está pensado para usar OIDC con GitHub Actions. En tu repositorio de GitHub debes configurar:

- un rol de IAM en AWS con confianza para GitHub Actions,
- la política mínima necesaria para publicar en ECR y actualizar ECS,
- la variable `AWS_REGION` con la región de despliegue,
- la variable `AWS_ROLE_ARN` con el ARN del rol creado.

Además, debes ajustar el archivo [.github/workflows/ci-cd.yaml](.github/workflows/ci-cd.yaml) con tus valores reales de ECR, cluster ECS, servicio ECS y región.

## Ejecución local

### 1 Crear entorno virtual

```bash
python -m venv .venv
source .venv/bin/activate

```

### 2 Instalar dependencias

```bash
pip install -r requirements.txt
```

### 3 Ejecutar la aplicación

```bash
python run.py
```

La aplicación queda disponible en `http://localhost:5000` por defecto. El puerto puede sobrescribirse mediante la variable de entorno `PORT`.

## Ejecución con Docker local

```bash
docker build -t docker-cicd-aws .
docker run -p 8000:8000 docker-cicd-aws
```

En el contenedor, la aplicación se expone por el puerto `8000` y utiliza Gunicorn como servidor WSGI.

## Cómo correrlo en AWS

Una vez que tengas la configuración de AWS lista, el flujo es este:

1. Haz push a la rama `main` del repositorio.
2. GitHub Actions construye la imagen Docker.
3. La imagen se publica en Amazon ECR.
4. El workflow actualiza el servicio ECS con la nueva imagen.
5. La aplicación queda disponible a través del Load Balancer o el endpoint público del servicio ECS.

Si quieres probarlo manualmente, puedes construir y ejecutar la imagen localmente primero, y luego repetir el mismo proceso en el entorno AWS con tus recursos reales.

## Pipeline CI/CD en AWS

El flujo definido en [.github/workflows/ci-cd.yaml](.github/workflows/ci-cd.yaml) ejecuta los siguientes pasos:

1. Checkout del código fuente.
2. Configuración del entorno Docker.
3. Escaneo de seguridad con Trivy.
4. Autenticación con AWS mediante OIDC.
5. Inicio de sesión en Amazon ECR.
6. Construcción y publicación de la imagen Docker.
7. Actualización del servicio ECS con la nueva imagen.

### Consideraciones importantes

El workflow está preparado para un entorno de ejemplo. Antes de usarlo en producción, debes ajustar los siguientes valores:

- el ARN del rol de GitHub Actions en AWS,
- la región de despliegue,
- el nombre del repositorio ECR,
- el cluster ECS,
- el servicio ECS,
- el nombre del contenedor definido en la tarea.

## Variables y secretos de GitHub

Configura los siguientes elementos en GitHub:

- Settings → Secrets and variables → Actions
- Define los secretos o variables necesarios para la autenticación de AWS, según el método elegido.

Recomendación general:

- usa OIDC para autenticar GitHub Actions con AWS,
- evita almacenar credenciales estáticas en el repositorio,
- limita los permisos al mínimo necesario.

## Requisitos de despliegue en ECS

Para que el despliegue funcione correctamente, tu tarea de ECS debe:

- exponer el puerto `8000`,
- usar la imagen publicada en ECR,
- definir la variable de entorno `PORT=8000`,
- permitir el endpoint `/health` como comprobación de salud.

## Seguridad

- No compartas credenciales ni secretos en el código fuente.
- Usa IAM con permisos mínimos.
- Mantén actualizadas las imágenes base y las dependencias.

## Sugerencias de mantenimiento

- Mantén el workflow alineado con la arquitectura real de tu cuenta AWS.
- Revisa periódicamente los logs de ECS para validar el estado del servicio.
- Si cambias el nombre del contenedor o la tarea, actualiza también el workflow.

## Nota sobre YAML en GitHub Actions

GitHub Actions requiere indentación con espacios, no tabs. Si el archivo de workflow presenta errores de sintaxis, verifica que no haya mezcla de tabulaciones y espacios en la indentación.
prueba
