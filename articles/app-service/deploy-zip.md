---
title: Implementación de código con un archivo ZIP o WAR
description: Aprenda a implementar una aplicación en Azure App Service con un archivo ZIP (o un archivo WAR en el caso de desarrolladores de Java).
ms.topic: article
ms.date: 08/12/2019
ms.reviewer: sisirap
ms.custom: seodec18
ms.openlocfilehash: f547f60bf6c67b757f2e12f06f1e39100c3e76cb
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88077159"
---
# <a name="deploy-your-app-to-azure-app-service-with-a-zip-or-war-file"></a>Implementación de una aplicación en Azure App Service con un archivo ZIP o WAR

En este artículo se muestra cómo usar un archivo ZIP o WAR para implementar una aplicación web en [Azure App Service](overview.md). 

Esta implementación del archivo ZIP usa el mismo servicio de Kudu que permite realizar implementaciones basadas en integraciones continuas. Kudu admite la siguientes funcionalidades para implementar el archivo ZIP: 

- La eliminación de archivos restantes de una implementación anterior.
- La opción de activar el proceso de compilación predeterminado, en el que se incluye la restauración del paquete.
- La personalización de la implementación, incluyendo la ejecución de scripts de implementación.  
- Registros de implementación. 
- Un límite de tamaño de archivo de 2048 MB.

Para obtener más información, consulte la [documentación de Kudu](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file).

La implementación del archivo WAR implementa su archivo [WAR](https://wikipedia.org/wiki/WAR_(file_format)) en Azure App Service para ejecutar la aplicación web de Java. Consulte [Implementación de un archivo WAR](#deploy-war-file).

> [!NOTE]
> Al usar `ZipDeploy`, solo se copiarán los archivos si sus marcas de tiempo no coinciden con lo que ya se ha implementado. La generación de un archivo ZIP mediante un proceso de compilación que almacena en memoria caché los resultados puede dar lugar a implementaciones más rápidas. Consulte el tema sobre la [implementación desde un archivo ZIP o una dirección URL](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url) para obtener más información.

## <a name="prerequisites"></a>Requisitos previos

Para completar los pasos en este artículo, [cree una aplicación de App Service](/azure/app-service/) o use alguna aplicación que haya creado para otro tutorial.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [Create a project ZIP file](../../includes/app-service-web-deploy-zip-prepare.md)]

[!INCLUDE [Deploy ZIP file](../../includes/app-service-web-deploy-zip.md)]
El punto de conexión anterior no funciona en este momento con App Services de Linux. Considere la posibilidad de usar en su lugar FTP o la [API de implementación de ZIP](faq-app-service-linux.md#continuous-integration-and-deployment).

## <a name="deploy-zip-file-with-azure-cli"></a>Implementación del archivo ZIP con la CLI de Azure

Implemente el archivo ZIP cargado en la aplicación web mediante el comando [az webapp deployment source config-zip](/cli/azure/webapp/deployment/source?view=azure-cli-latest#az-webapp-deployment-source-config-zip).  

En el ejemplo siguiente se implementa el archivo ZIP que cargó. Cuando use una instalación local de la CLI de Azure, debe especificar la ruta de acceso al archivo ZIP local de `--src`.

```azurecli-interactive
az webapp deployment source config-zip --resource-group <group-name> --name <app-name> --src clouddrive/<filename>.zip
```

Este comando implementa los archivos y directorios del archivo ZIP en la carpeta predeterminada de la aplicación App Service (`\home\site\wwwroot`) y reinicia la aplicación.

De forma predeterminada, el motor de implementación da por supuesto que un archivo ZIP está listo para ejecutarse tal cual y no ejecuta ninguna automatización de la compilación. Para habilitar la misma automatización de la compilación que en una [implementación de Git](deploy-local-git.md), establezca la configuración de la aplicación `SCM_DO_BUILD_DURING_DEPLOYMENT` ejecutando el siguiente comando en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings SCM_DO_BUILD_DURING_DEPLOYMENT=true
```

Para obtener más información, consulte la [documentación de Kudu](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url).

[!INCLUDE [app-service-deploy-zip-push-rest](../../includes/app-service-deploy-zip-push-rest.md)]  

## <a name="deploy-war-file"></a>Implementación de un archivo WAR

Para implementar un archivo WAR en App Service, envíe una solicitud POST a `https://<app-name>.scm.azurewebsites.net/api/wardeploy`. La solicitud POST debe contener el archivo .war en el cuerpo del mensaje. Las credenciales de implementación de la aplicación se proporcionan en la solicitud mediante la autenticación de HTTP BASIC.

Use siempre `/api/wardeploy` al implementar archivos WAR. Esta API expandirá el archivo WAR y lo colocará en la unidad de archivos compartidos. el uso de otras API de implementación puede dar lugar a un comportamiento incoherente. 

Para la autenticación básica HTTP, necesita las credenciales de implementación de App Service. Para ver cómo establecer sus credenciales de implementación, consulte [Establecimiento y restablecimiento de credenciales de nivel de usuario](deploy-configure-credentials.md#userscope).

### <a name="with-curl"></a>Con cURL

En el ejemplo siguiente se usa la herramienta cURL para implementar un archivo .war. Reemplace los marcadores de posición `<username>`, `<war-file-path>` y `<app-name>`. Cuando cURL se lo solicite, escriba la contraseña.

```bash
curl -X POST -u <username> --data-binary @"<war-file-path>" https://<app-name>.scm.azurewebsites.net/api/wardeploy
```

### <a name="with-powershell"></a>Con PowerShell

En el ejemplo siguiente se usa [Publish-AzWebapp](/powershell/module/az.websites/publish-azwebapp) para cargar el archivo .war. Reemplace los marcadores de posición `<group-name>`, `<app-name>` y `<war-file-path>`.

```powershell
Publish-AzWebapp -ResourceGroupName <group-name> -Name <app-name> -ArchivePath <war-file-path>
```

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="next-steps"></a>Pasos siguientes

Para ver escenarios de implementación más avanzados, pruebe [Implementación en Azure con Git](deploy-local-git.md). La implementación basada en Git en Azure permite el control de versiones, la restauración de paquetes, MSBuild y mucho más.

## <a name="more-resources"></a>Más recursos

* [Kudu: realizar una implementación desde un archivo .zip](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file)
* [Credenciales de implementación de Azure App Service](deploy-ftp.md)
