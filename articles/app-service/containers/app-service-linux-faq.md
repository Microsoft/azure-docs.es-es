---
title: 'Preguntas frecuentes sobre App Service en Linux: Azure | Microsoft Docs'
description: Peguntas más frecuentes sobre Azure App Service en Linux.
keywords: servicio de aplicación de azure, aplicación web, preguntas más frecuentes, linux, oss, aplicaciones web para contenedores, varios contenedores, multicontenedor
services: app-service
documentationCenter: ''
author: yili
manager: stefsch
editor: ''
ms.assetid: ''
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/30/2018
ms.author: yili
ms.custom: seodec18
ms.openlocfilehash: dbf63ff47b11c2e75966b4a4b91fb1b00b40d216
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65594278"
---
# <a name="azure-app-service-on-linux-faq"></a>Peguntas más frecuentes sobre Azure App Service en Linux

Con el lanzamiento de App Service en Linux, estamos trabajando en incorporar características y realizar mejoras en nuestra plataforma. Este artículo proporciona respuestas a preguntas que nuestros clientes nos han formulado recientemente.

Si tiene alguna pregunta, comente este artículo.

## <a name="built-in-images"></a>Imágenes integradas

**Quiero bifurcar los contenedores de Docker integrados que ofrece la plataforma. ¿Dónde puedo encontrar esos archivos?**

Puede encontrar todos los archivos de Docker en [GitHub](https://github.com/azure-app-service). Puede encontrar todos los contenedores de Docker en [Docker Hub](https://hub.docker.com/u/appsvc/).

<a id="#startup-file"></a>

**¿Cuáles son los valores previsibles para la sección del archivo de inicio cuando se configura la pila en tiempo de ejecución?**

| Pila           | Valor esperado                                                                         |
|-----------------|----------------------------------------------------------------------------------------|
| Java SE         | el comando para iniciar la aplicación del archivo JAR (por ejemplo, `java -jar my-app.jar --server.port=80`) |
| Tomcat, Wildfly | la ubicación de un script para realizar las configuraciones necesarias (por ejemplo, `/home/site/deployments/tools/startup_script.sh`)          |
| Node.js         | el archivo de configuración de PM2 o el archivo de script                                |
| .Net Core       | el nombre de archivo DLL compilado como `dotnet <myapp>.dll`                                 |
| Ruby            | el script de Ruby que se va a inicializar la aplicación con                     |

Estos comandos o scripts se ejecutan después de que se inicia el contenedor de Docker integrado, pero antes de la aplicación se inicia el código.

## <a name="management"></a>Administración

**¿Qué ocurre al hacer clic en botón de reinicio de Azure Portal?**

Esta acción es igual a un reinicio de Docker.

**¿Puedo usar Secure Shell (SSH) para conectarme a la máquina virtual del contenedor de la aplicación?**

Sí, puede hacerlo a través del sitio de administración de control de código fuente (SCM).

> [!NOTE]
> También puede conectarse al contenedor de la aplicación directamente desde la máquina de desarrollo local mediante SSH, SFTP o Visual Studio Code (para aplicaciones de Node.js de depuración en directo). Para más información, consulte [Depuración remota y SSH en App Service en Linux](https://aka.ms/linux-debug).
>

**¿Cómo se puede crear un plan de App Service en Linux a través de un SDK o una plantilla de Azure Resource Manager?**

Debe establecer el campo **reservado** del servicio de aplicación en *true*.

## <a name="continuous-integration-and-deployment"></a>Integración e implementación continuas

**Mi aplicación web sigue usando una imagen de contenedor de Docker antigua después de actualizar la imagen en Docker Hub. ¿Se admite la integración e implementación continuas de contenedores personalizados?**

Sí, para configurar la integración e implementación continua de Azure Container Registry o Docker Hub, consulte [Implementación continua con Web App for Containers](./app-service-linux-ci-cd.md). Para registros privados, puede actualizar el contenedor deteniendo y, luego, iniciando la aplicación web. También puede cambiar o agregar una configuración de aplicación ficticia para forzar una actualización del contenedor.

**¿Los entornos de ensayo son compatibles?**

Sí.

**¿Puedo usar *WebDeploy/MSDeploy* para implementar mi aplicación web?**

Sí, es necesario establecer un valor de la aplicación llamado `WEBSITE_WEBDEPLOY_USE_SCM` en *false*.

**Se produce un error en la implementación de GIT de la aplicación cuando se usa la aplicación web de Linux. ¿Cómo puedo solucionar el problema?**

Si se produce un error en la implementación de Git en la aplicación web de Linux, puede elegir una de las opciones siguientes para implementar el código de aplicación:

- Característica de entrega continua (versión preliminar): Puede almacenar el código fuente de la aplicación en un repositorio de DevOps Git de Azure o el repositorio de GitHub para utilizar la entrega continua de Azure. Para más información, consulte la publicación [How to configure Continuous Delivery for Linux web app](https://blogs.msdn.microsoft.com/devops/2017/05/10/use-azure-portal-to-setup-continuous-delivery-for-web-app-on-linux/) (Configuración de entrega continua para aplicaciones web de Linux).

- [ZIP deploy API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file): para usar esta API, [use SSH en la aplicación web](https://docs.microsoft.com/azure/app-service/containers/app-service-linux-ssh-support) y vaya a la carpeta en la que desea implementar el código. Ejecute el código siguiente:

   ```bash
   curl -X POST -u <user> --data-binary @<zipfile> https://{your-sitename}.scm.azurewebsites.net/api/zipdeploy
   ```

   Si recibe un error que indica que no se encuentra el comando `curl`, asegúrese de que instalar curl mediante `apt-get install curl` antes de ejecutar el comando `curl` anterior.

## <a name="language-support"></a>Compatibilidad con idiomas

**Quiero usar sockets web en mi aplicación Node.js. ¿Hay alguna opción o configuración especial que deba establecer?**

Sí, deshabilite `perMessageDeflate` en el código de Node.js en el servidor. Por ejemplo, si usa socket.io, use el código siguiente:

```nodejs
const io = require('socket.io')(server,{
  perMessageDeflate :false
});
```

**¿Se admiten aplicaciones de .NET Core sin compilar?**

Sí.

**¿Admite un compositor como un administrador de dependencias para aplicaciones PHP?**

Sí, durante una implementación de Git, Kudu debe detectar que va a implementar una aplicación PHP (gracias a la presencia de un archivo composer.lock) y Kudu desencadenará una instalación del compositor.

## <a name="custom-containers"></a>Contenedores personalizados

**Utilizo mi propio contenedor personalizado. Deseo que la plataforma monte un recurso compartido de SMB en el directorio `/home/`.**

Puede hacerlo estableciendo la opción `WEBSITES_ENABLE_APP_SERVICE_STORAGE` de la aplicación en *true*. Tenga en cuenta que de este modo se evitará que el contenedor se reinicie cuando el almacenamiento de la plataforma cambie.

>[!NOTE]
>Si el valor de `WEBSITES_ENABLE_APP_SERVICE_STORAGE` no se ha especificado o es *false*, el directorio `/home/` no se compartirá entre instancias de escala y los archivos que se escriben no se conservarán al reiniciar.

**Mi contenedor personalizado tarda mucho tiempo en iniciarse y la plataforma reinicia el contenedor antes de que finalice el inicio.**

Puede configurar el tiempo que va a esperar la plataforma antes de reiniciar el contenedor. Para ello, establezca el valor `WEBSITES_CONTAINER_START_TIME_LIMIT` de la aplicación en el valor que desee. El valor predeterminado es de 230 segundos y el valor máximo es de 1800 segundos.

**¿Cuál es el formato de la dirección URL del servidor de registro privado?**

Debe escribir la dirección URL completa del registro, incluidos `http://` o `https://`.

**¿Cuál es el formato del nombre de la imagen en la opción del registro privado?**

Agregue el nombre de la imagen completo, incluida la dirección URL de registro privado (por ejemplo, myacr.azurecr.io/dotnet:latest). Los nombres de imágenes que usan un puerto personalizado [no se pueden especificar a través del portal](https://feedback.azure.com/forums/169385-web-apps/suggestions/31304650). Para establecer `docker-custom-image-name`, use la [`az`herramienta de línea de comandos](https://docs.microsoft.com/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set).

**¿Se puede exponer más de un puerto en mi imagen de contenedor personalizado?**

No se admite exponer más de un puerto.

**¿Puedo traer mi propio almacenamiento?**

Sí, la prestación [traiga su propio almacenamiento](https://docs.microsoft.com/azure/app-service/containers/how-to-serve-content-from-azure-storage) está en versión preliminar.

**¿Por qué no puedo examinar el sistema de archivos de mi contenedor personalizado o ejecutar procesos desde el sitio de SCM?**

El sitio de SCM se ejecuta en un contenedor independiente. no puede comprobar el sistema de archivos o los procesos en ejecución del contenedor de la aplicación.

**Mi contenedor personalizado escucha en un puerto distinto al puerto 80. ¿Cómo puedo configurar mi aplicación para enrutar las solicitudes hacia ese puerto?**

Disponemos de detección automática de puerto. También puede especificar un valor de la aplicación llamado *WEBSITES_PORT* y asignarle el valor del número de puerto esperado. Anteriormente, la plataforma utilizaba el valor de aplicación *PORT*. Tenemos previsto para dejar de utilizar este valor de la aplicación y usar *WEBSITES_PORT* exclusivamente.

**¿Es necesario implementar HTTPS en mi contenedor personalizado?**

No, la plataforma controla la terminación HTTPS en los front-end compartidos.

## <a name="multi-container-with-docker-compose"></a>Varios contenedores con Docker Compose

**¿Cómo puedo configurar Azure Container Registry (ACR) para usar con varios contenedores?**

Para poder usar ACR con varios contenedores, **todas las imágenes de contenedor** deben estar hospedadas en el mismo servidor de registro ACR. Una vez que estén en el mismo servidor de registro, deberá crear la configuración de la aplicación y, a continuación, actualice el archivo de configuración de Docker Compose para incluir el nombre de imagen ACR.

Cree la siguiente configuración de la aplicación:

- DOCKER_REGISTRY_SERVER_USERNAME
- DOCKER_REGISTRY_SERVER_URL (total de dirección URL, p. ej.: `https://<server-name>.azurecr.io`)
- DOCKER_REGISTRY_SERVER_PASSWORD (habilitar el acceso de administrador en la configuración de ACR)

En el archivo de configuración, haga referencia a la imagen ACR similar al ejemplo siguiente:

```yaml
image: <server-name>.azurecr.io/<image-name>:<tag>
```

**¿Cómo puedo saber qué contenedor es accesible desde Internet?**

- Solo un contenedor puede estar abierto para el acceso
- Solo el puerto 80 y 8080 es accesible (puertos expuestos)

Estas son las reglas para determinar qué contenedor es accesible, en el orden de prioridad:

- Configuración de la aplicación `WEBSITES_WEB_CONTAINER_NAME` establecida en el nombre del contenedor
- El primer contenedor para definir el puerto 80 u 8080
- Si ninguna de las opciones anteriores es true, el primer contenedor definido en el archivo será accesible (expuesto)

## <a name="pricing-and-sla"></a>Precios y contrato de nivel de servicio

**¿Cuál es el precio ahora que el servicio está disponible con carácter general?**

Se le cobrará el número de horas que se ejecute la aplicación, con los precios normales de Azure App Service.

## <a name="other-questions"></a>Otras preguntas

**¿Cuáles son los caracteres admitidos en los nombres de configuración de la aplicación?**

Puede usar solo letras (A-Z, a-z), números (0-9) y el carácter de subrayado (_) para la configuración de la aplicación.

**¿Dónde puedo solicitar nuevas características?**

Puede enviar su idea en el [foro de comentarios de Web Apps](https://aka.ms/webapps-uservoice). Agregue [Linux] en el título de la idea.

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es Azure App Service en Linux?](app-service-linux-intro.md)
- [Configurar entornos de ensayo en Azure App Service](../../app-service/deploy-staging-slots.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)
- [Implementación continua con Web App for Containers](./app-service-linux-ci-cd.md)
