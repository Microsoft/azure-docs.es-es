---
title: Uso de Docker Compose para implementar varios contenedores
titleSuffix: Azure Cognitive Services
description: Aprenda a implementar varios contenedores de Cognitive Services. En este artículo se muestra cómo coordinar varias imágenes de contenedor de Docker mediante Docker Compose.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 740311226a662ea3d3f8bba3ee5156e14f74516b
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88244302"
---
# <a name="use-docker-compose-to-deploy-multiple-containers"></a>Uso de Docker Compose para implementar varios contenedores

En este artículo se muestra cómo implementar varios contenedores de Azure Cognitive Services. En concreto, aprenderá a usar Docker Compose para coordinar varias imágenes de contenedor de Docker.

> [Docker Compose](https://docs.docker.com/compose/) es una herramienta para definir y ejecutar aplicaciones de Docker de varios contenedores. En Compose, se usa un archivo YAML para configurar los servicios de la aplicación. Después, con un solo comando, se crean y se inician todos los servicios de la configuración.

Puede ser útil coordinar varias imágenes de contenedor en un solo equipo host. En este artículo, reuniremos los contenedores de lectura y de Form Recognizer.

## <a name="prerequisites"></a>Requisitos previos

Este procedimiento requiere varias herramientas que se deben instalar y ejecutar localmente:

* Suscripción a Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services) antes de empezar.
* [Motor de Docker](https://www.docker.com/products/docker-engine). Confirme que la CLI de Docker funciona en una ventana de consola.
* Un recurso de Azure con el plan de tarifa correcto. Solo los siguientes planes de tarifa funcionan con este contenedor:
  * El recurso **Computer Vision** solo con el plan de tarifa estándar o F0.
  * El recurso **Form Recognizer** solo con el plan de tarifa estándar o F0.
  * El recurso **Cognitive Services** con el plan de tarifas S0.

## <a name="request-access-to-the-container-registry"></a>Solicitud de acceso al registro de contenedor

Complete y envíe el [formulario de solicitud de contenedores de Voz de Cognitive Services](https://aka.ms/speechcontainerspreview/). 

[!INCLUDE [Request access to the container registry](../../../includes/cognitive-services-containers-request-access-only.md)]

[!INCLUDE [Authenticate to the container registry](../../../includes/cognitive-services-containers-access-registry.md)]

## <a name="docker-compose-file"></a>Archivo de Docker Compose

El archivo YAML define todos los servicios que se van a implementar. Estos servicios se basan en un archivo `DockerFile` o en una imagen de contenedor existente. En este caso, usaremos dos imágenes de vista previa. Copie y pegue el siguiente archivo YAML y guárdelo como *docker-compose.yaml*. Especifique los valores adecuados de **apikey**, **billing** y **EndpointUri** en el archivo.

```yaml
version: '3.7'
services:
  forms:
    image: "containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer"
    environment:
       eula: accept
       billing: # < Your form recognizer billing URL >
       apikey: # < Your form recognizer API key >
       FormRecognizer__ComputerVisionApiKey: # < Your form recognizer API key >
       FormRecognizer__ComputerVisionEndpointUri: # < Your form recognizer URI >
    volumes:
       - type: bind
         source: E:\publicpreview\output
         target: /output
       - type: bind
         source: E:\publicpreview\input
         target: /input
    ports:
      - "5010:5000"

  ocr:
    image: "containerpreview.azurecr.io/microsoft/cognitive-services-read"
    environment:
      eula: accept
      apikey: # < Your computer vision API key >
      billing: # < Your computer vision billing URL >
    ports:
      - "5021:5000"
```

> [!IMPORTANT]
> Cree los directorios en la máquina host que se especifican bajo el nodo **volumes**. Esto es necesario porque los directorios deben existir para intentar montar una imagen mediante enlaces de volumen.

## <a name="start-the-configured-docker-compose-services"></a>Inicio de los servicios configurados de Docker Compose

Un archivo de Docker Compose permite administrar todas las fases del ciclo de vida de un servicio definido: iniciar, detener y volver a generar los servicios, ver su estado y transmitir los registros. Abra una interfaz de línea de comandos desde el directorio del proyecto (donde se encuentra el archivo docker-compose.yaml).

> [!NOTE]
> Para evitar errores, asegúrese de que la máquina host comparte correctamente las unidades con el motor de Docker. Por ejemplo, si *E:\publicpreview* se usa como directorio en el archivo *docker-compose.yaml*, comparta la unidad **E** con Docker.

Desde la interfaz de la línea de comandos, ejecute el siguiente comando para iniciar (o reiniciar) todos los servicios definidos en el archivo *docker-compose.yaml*:

```console
docker-compose up
```

La primera vez que Docker ejecuta el comando **docker-compose up** mediante esta configuración, extrae las imágenes configuradas en el nodo **services** y, luego, las descarga y las monta:

```console
Pulling forms (containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer:)...
latest: Pulling from microsoft/cognitive-services-form-recognizer
743f2d6c1f65: Pull complete
72befba99561: Pull complete
2a40b9192d02: Pull complete
c7715c9d5c33: Pull complete
f0b33959f1c4: Pull complete
b8ab86c6ab26: Pull complete
41940c21ed3c: Pull complete
e3d37dd258d4: Pull complete
cdb5eb761109: Pull complete
fd93b5f95865: Pull complete
ef41dcbc5857: Pull complete
4d05c86a4178: Pull complete
34e811d37201: Pull complete
Pulling ocr (containerpreview.azurecr.io/microsoft/cognitive-services-read:)...
latest: Pulling from microsoft/cognitive-services-read
f476d66f5408: Already exists
8882c27f669e: Already exists
d9af21273955: Already exists
f5029279ec12: Already exists
1a578849dcd1: Pull complete
45064b1ab0bf: Download complete
4bb846705268: Downloading [=========================================>         ]  187.1MB/222.8MB
c56511552241: Waiting
e91d2aa0f1ad: Downloading [==============================================>    ]  162.2MB/176.1MB
```

Una vez que se han descargado las imágenes, se inician los servicios de imagen:

```console
Starting docker_ocr_1   ... done
Starting docker_forms_1 ... doneAttaching to docker_ocr_1, docker_forms_1forms_1  | forms_1  | forms_1  | Notice: This Preview is made available to you on the condition that you agree to the Supplemental Terms of Use for Microsoft Azure Previews [https://go.microsoft.com/fwlink/?linkid=2018815], which supplement your agreement [https://go.microsoft.com/fwlink/?linkid=2018657] governing your use of Azure. If you do not have an existing agreement governing your use of Azure, you agree that your agreement governing use of Azure is the Microsoft Online Subscription Agreement [https://go.microsoft.com/fwlink/?linkid=2018755] (which incorporates the Online Services Terms [https://go.microsoft.com/fwlink/?linkid=2018760]). By using the Preview you agree to these terms.
forms_1  | 
forms_1  | 
forms_1  | Using '/input' for reading models and other read-only data.
forms_1  | Using '/output/forms/812d811d1bcc' for writing logs and other output data.
forms_1  | Logging to console.
forms_1  | Submitting metering to 'https://westus2.api.cognitive.microsoft.com/'.
forms_1  | WARNING: No access control enabled!
forms_1  | warn: Microsoft.AspNetCore.Server.Kestrel[0]
forms_1  |       Overriding address(es) 'http://+:80'. Binding to endpoints defined in UseKestrel() instead.
forms_1  | Hosting environment: Production
forms_1  | Content root path: /app/forms
forms_1  | Now listening on: http://0.0.0.0:5000
forms_1  | Application started. Press Ctrl+C to shut down.
ocr_1    | 
ocr_1    | 
ocr_1    | Notice: This Preview is made available to you on the condition that you agree to the Supplemental Terms of Use for Microsoft Azure Previews [https://go.microsoft.com/fwlink/?linkid=2018815], which supplement your agreement [https://go.microsoft.com/fwlink/?linkid=2018657] governing your use of Azure. If you do not have an existing agreement governing your use of Azure, you agree that your agreement governing use of Azure is the Microsoft Online Subscription Agreement [https://go.microsoft.com/fwlink/?linkid=2018755] (which incorporates the Online Services Terms [https://go.microsoft.com/fwlink/?linkid=2018760]). By using the Preview you agree to these terms.
ocr_1    |
ocr_1    | 
ocr_1    | Logging to console.
ocr_1    | Submitting metering to 'https://westcentralus.api.cognitive.microsoft.com/'.
ocr_1    | WARNING: No access control enabled!
ocr_1    | Hosting environment: Production
ocr_1    | Content root path: /
ocr_1    | Now listening on: http://0.0.0.0:5000
ocr_1    | Application started. Press Ctrl+C to shut down.
```

## <a name="verify-the-service-availability"></a>Comprobación de la disponibilidad del servicio

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

A continuación se muestra una salida de ejemplo:

```
IMAGE ID            REPOSITORY                                                                 TAG
2ce533f88e80        containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer   latest
4be104c126c5        containerpreview.azurecr.io/microsoft/cognitive-services-read              latest
```

### <a name="test-containers"></a>Contenedores de prueba

Abra un explorador en la máquina host y vaya a **localhost**, para lo que usará el puerto especificado en el archivo *docker-compose.yaml*, como http://localhost:5021/swagger/index.html. Por ejemplo, puede usar la característica **Probarlo** de la API para probar el punto de conexión de Form Recognizer. Las páginas de Swagger de ambos contenedores deberían estar disponibles y deberían poder probarse.

![Contenedor de Form Recognizer](media/form-recognizer-swagger-page.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Contenedores de Cognitive Services](../cognitive-services-container-support.md)
