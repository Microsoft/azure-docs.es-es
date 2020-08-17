---
title: Modelos de paquetes
titleSuffix: Azure Machine Learning
description: Empaquetar un modelo como Dockerfile
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 07/31/2020
ms.topic: conceptual
ms.openlocfilehash: d5fb2539d79c31de5a5e0196a7a4814c02a84602
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87544493"
---
# <a name="how-to-package-a-registered-model-with-docker"></a>Cómo empaquetar un modelo registrado con Docker

En este artículo se muestra cómo empaquetar un modelo de Azure Machine Learning registrado con Docker.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por supuesto que ya ha entrenado y registrado un modelo en el área de trabajo de aprendizaje automático. Para obtener información sobre cómo entrenar y registrar un modelo de scikit-learn, [siga este tutorial](how-to-train-scikit-learn.md).


## <a name="package-models"></a>Modelos de paquetes

En algunos casos, es posible que desee crear una imagen de Docker sin implementar el modelo (si, por ejemplo, pretende [implementar en Azure App Service](how-to-deploy-app-service.md)). O bien, puede que desee descargar la imagen y ejecutarla en una instalación local de Docker. Incluso, puede que desee descargar los archivos usados para compilar la imagen, inspeccionarlos, modificarlos y compilar la imagen manualmente.

El empaquetado de modelos permite hacer estas cosas. Empaqueta todos los recursos necesarios para hospedar un modelo como servicio web y le permite descargar una imagen de Docker totalmente compilada o los archivos necesarios para compilar una. Hay dos maneras de usar el empaquetado del modelos:

**Descarga de un modelo empaquetado:** descargue una imagen de Docker que contiene el modelo y otros archivos necesarios para hospedarla como servicio web.

**Generar un Dockerfile:** descargue el Dockerfile, el modelo, el script de entrada y otros recursos necesarios para compilar una imagen de Docker. Después, puede inspeccionar los archivos o realizar cambios antes de compilar la imagen localmente.

Ambos paquetes se pueden usar para obtener una imagen de Docker local.

> [!TIP]
> La creación de un paquete es similar a la implementación de un modelo. Se usa un modelo registrado y una configuración de inferencia.

> [!IMPORTANT]
> Para descargar una imagen completamente compilada o compilar una imagen localmente, debe tener [Docker](https://www.docker.com) instalado en el entorno de desarrollo.

### <a name="download-a-packaged-model"></a>Descarga del modelo empaquetado

En el ejemplo siguiente se muestra cómo compilar una imagen, que se registra en el registro de contenedor de Azure del área de trabajo:

```python
package = Model.package(ws, [model], inference_config)
package.wait_for_creation(show_output=True)
```

Después de crear un paquete, puede usar `package.pull()` para extraer la imagen en el entorno de Docker local. La salida de este comando mostrará el nombre de la imagen. Por ejemplo: 

`Status: Downloaded newer image for myworkspacef78fd10.azurecr.io/package:20190822181338`. 

Después de descargar el modelo, use el comando `docker images` para ver en una lista las imágenes locales:

```text
REPOSITORY                               TAG                 IMAGE ID            CREATED             SIZE
myworkspacef78fd10.azurecr.io/package    20190822181338      7ff48015d5bd        4 minutes ago       1.43 GB
```

Para iniciar un contenedor local basándose en esta imagen, use el siguiente comando para iniciar un contenedor con nombre desde el shell o desde la línea de comandos. Reemplace el valor `<imageid>` por el identificador de la imagen que el comando `docker images` devuelve.

```bash
docker run -p 6789:5001 --name mycontainer <imageid>
```

Este comando inicia la versión más reciente de la imagen con nombre `myimage`. Asigna el puerto local 6789 al puerto del contenedor en el que el servicio web está escuchando (5001). También asigna el nombre `mycontainer` al contenedor, lo que facilita la detención de este. Una vez iniciado el contenedor, puede enviar solicitudes a `http://localhost:6789/score`.

### <a name="generate-a-dockerfile-and-dependencies"></a>Generación de un Dockerfile y dependencias

En el ejemplo siguiente se muestra cómo descargar el Dockerfile, el modelo y otros recursos necesarios para compilar una imagen localmente. El parámetro `generate_dockerfile=True` indica que quiere los archivos, no una imagen completamente compilada.

```python
package = Model.package(ws, [model], inference_config, generate_dockerfile=True)
package.wait_for_creation(show_output=True)
# Download the package.
package.save("./imagefiles")
# Get the Azure container registry that the model/Dockerfile uses.
acr=package.get_container_registry()
print("Address:", acr.address)
print("Username:", acr.username)
print("Password:", acr.password)
```

Este código descarga los archivos necesarios para compilar la imagen en el directorio `imagefiles`. El Dockerfile incluido en los archivos guardados hace referencia a una imagen base almacenada en un registro de contenedor de Azure. Al compilar la imagen en la instalación de Docker local, necesita usar la dirección, el nombre de usuario y la contraseña para autenticarse en el registro. Siga estos pasos para compilar la imagen mediante una instalación local de Docker:

1. Desde un shell o una sesión de línea de comandos, use el comando siguiente para autenticar Docker con el registro de contenedor de Azure. Reemplace `<address>`, `<username>` y `<password>` por los valores recuperados con `package.get_container_registry()`.

    ```bash
    docker login <address> -u <username> -p <password>
    ```

2. Para compilar la imagen, utilice el siguiente comando. Reemplace `<imagefiles>` por la ruta de acceso del directorio donde `package.save()` guardó los archivos.

    ```bash
    docker build --tag myimage <imagefiles>
    ```

    Este comando establece el nombre de la imagen en `myimage`.

Para comprobar la imagen está compilada, use el comando `docker images`. Debería ver la imagen `myimage` en la lista:

```text
REPOSITORY      TAG                 IMAGE ID            CREATED             SIZE
<none>          <none>              2d5ee0bf3b3b        49 seconds ago      1.43 GB
myimage         latest              739f22498d64        3 minutes ago       1.43 GB
```

Para iniciar un nuevo contenedor basado en esta imagen, use el comando siguiente:

```bash
docker run -p 6789:5001 --name mycontainer myimage:latest
```

Este comando inicia la versión más reciente de la imagen con nombre `myimage`. Asigna el puerto local 6789 al puerto del contenedor en el que el servicio web está escuchando (5001). También asigna el nombre `mycontainer` al contenedor, lo que facilita la detención de este. Una vez iniciado el contenedor, puede enviar solicitudes a `http://localhost:6789/score`.

### <a name="example-client-to-test-the-local-container"></a>Cliente de ejemplo para probar el contenedor local

El código siguiente es un ejemplo de un cliente de Python que se puede usar con el contenedor:

```python
import requests
import json

# URL for the web service.
scoring_uri = 'http://localhost:6789/score'

# Two sets of data to score, so we get two results back.
data = {"data":
        [
            [ 1,2,3,4,5,6,7,8,9,10 ],
            [ 10,9,8,7,6,5,4,3,2,1 ]
        ]
        }
# Convert to JSON string.
input_data = json.dumps(data)

# Set the content type.
headers = {'Content-Type': 'application/json'}

# Make the request and display the response.
resp = requests.post(scoring_uri, input_data, headers=headers)
print(resp.text)
```

Para ver clientes de ejemplo en otros lenguajes de programación, vea [Consumir modelos implementados como servicios web](how-to-consume-web-service.md).

### <a name="stop-the-docker-container"></a>Detención del contenedor de Docker

Para detener el contenedor, use el siguiente comando desde una línea de comandos o un shell diferente:

```bash
docker kill mycontainer
```

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de implementaciones con errores](how-to-troubleshoot-deployment.md)
* [Implementación en Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)
* [Creación de aplicaciones cliente para consumir servicios web](how-to-consume-web-service.md)
* [Actualización de servicios web](how-to-deploy-update-web-service.md)
* [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md)
* [Uso de TLS para proteger un servicio web con Azure Machine Learning](how-to-secure-web-service.md)
* [Supervisión de los modelos de Azure Machine Learning con Application Insights](how-to-enable-app-insights.md)
* [Recopilar datos de modelos en producción](how-to-enable-data-collection.md)
* [Creación de flujos de trabajo de aprendizaje automático controlados por eventos (versión preliminar)](how-to-use-event-grid.md)
