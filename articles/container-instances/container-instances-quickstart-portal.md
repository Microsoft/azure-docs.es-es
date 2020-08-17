---
title: 'Inicio rápido: implementación de un contenedor de Docker en una instancia de contenedor (portal)'
description: En este inicio rápido, usará Azure Portal para implementar rápidamente una aplicación web en contenedores que se ejecuta en una instancia de contenedor aislada de Azure
ms.topic: quickstart
ms.date: 03/09/2020
ms.custom: seodec18, mvc, devx-track-javascript
ms.openlocfilehash: 5f36c81abd462f6dfd59c42da71e096e07206cae
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87408074"
---
# <a name="quickstart-deploy-a-container-instance-in-azure-using-the-azure-portal"></a>Inicio rápido: Implementación de una instancia de contenedor en Azure mediante Azure Portal

Use Azure Container Instances para ejecutar contenedores de Docker sin servidor en Azure con sencillez y velocidad. Implemente una aplicación en una instancia de contenedor a petición cuando no necesite una plataforma de orquestación de contenedores completa, como Azure Kubernetes Service.

En esta guía de inicio rápido, va a usar Azure Portal para implementar un contenedor de Docker aislado y hacer que su aplicación esté disponible con un nombre de dominio completo (FQDN). Después de configurar algunos valores e implementar el contenedor, puede ir a la aplicación en ejecución:

![Aplicación implementada en Azure Container Instances vista en el explorador][aci-portal-07]

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita][azure-free-account] antes de empezar.

## <a name="create-a-container-instance"></a>Creación de instancia de contenedor

Seleccione **Crear un recurso** > **Contenedores** > **Container Instances**.

![Comenzar a crear una instancia de contenedor nueva en Azure Portal][aci-portal-01]

En la página **Aspectos básicos**, escriba los siguientes valores en los cuadros de texto **Grupo de recursos**, **Nombre de contenedor** e **Imagen de contenedor**. Deje los otros valores predeterminados y haga clic en **Aceptar**.

* Grupo de recursos: **Crear nuevo** > `myresourcegroup`
* Nombre de contenedor: `mycontainer`
* Origen de imagen: **Imágenes de inicio rápido**
* Imagen de contenedor: `mcr.microsoft.com/azuredocs/aci-helloworld` (Linux)

![Configuración básica de una instancia de contenedor nueva en Azure Portal][aci-portal-03]

En este inicio rápido, use la configuración predeterminada para implementar la imagen `aci-helloworld` pública de Microsoft. Esta imagen de Linux de ejemplo empaqueta una pequeña aplicación web escrita en Node.js que sirve una página HTML estática. También puede traer sus propias imágenes de contenedor almacenadas en Azure Container Registry, Docker Hub u otros registros.

En la página **Redes**, especifique una **etiqueta de nombre DNS** para el contenedor. El nombre debe ser único dentro de la región de Azure en la que crea la instancia de contenedor. El contenedor estará públicamente accesible en `<dns-name-label>.<region>.azurecontainer.io`. Si recibe un mensaje de error "DNS name label not available" (La etiqueta de nombre DNS no está disponible), pruebe otra etiqueta de nombre DNS diferente.

![Configuración de red para una instancia de contenedor nueva en Azure Portal][aci-portal-04]

Deje el resto de opciones con sus valores predeterminados y, luego, seleccione **Revisar y crear**.

Cuando se complete la validación, verá un resumen de la configuración del contenedor. Seleccione **Crear** para enviar la solicitud de implementación de contenedor.

![Resumen de la configuración de una instancia de contenedor nueva en Azure Portal][aci-portal-05]

Cuando se inicia una implementación, aparece una notificación que indica que la implementación está en curso. Aparecerá otra notificación cuando se haya implementado el grupo de contenedores.

Para abrir la información general del grupo de contenedores, vaya a **Grupos de recursos** > **myresourcegroup** > **mycontainer**. Tome nota del **FQDN** (el nombre de dominio completo) de la instancia de contenedor y también de su **estado**.

![Información general del grupo de contenedores en Azure Portal][aci-portal-06]

Cuando el **Estado** sea *En ejecución*, vaya al FQDN del contenedor en el explorador.

![Aplicación implementada mediante Azure Container Instances vista en el explorador][aci-portal-07]

Felicidades. Con tan solo implementar algunos valores, ha implementado una aplicación de acceso público en Azure Container Instances.

## <a name="view-container-logs"></a>Visualización de registros de contenedores

Ver los registros de una instancia de contenedor resulta de utilidad al solucionar problemas con el contenedor o la aplicación en la que se ejecuta.

Para ver los registros del contenedor, en **Configuración**, seleccione **Contenedores** y luego **Registros**. Debería ver la solicitud GET HTTP que se genera cuando se ve la aplicación en el explorador.

![Registros de contenedor en Azure Portal][aci-portal-11]

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con el contenedor, seleccione **Introducción** para el contenedor *mycontainer* y luego seleccione **Eliminar**.

![Eliminación de una instancia de contenedor en Azure Portal][aci-portal-09]

Seleccione **Sí** cuando aparezca el cuadro de diálogo de confirmación.

![Eliminar la confirmación de una instancia de contenedor en Azure Portal][aci-portal-10]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una instancia de contenedor de Azure a partir de una imagen de Microsoft pública. Si quiere compilar una imagen de contenedor e implementarla desde un registro de contenedor privado de Azure, vaya al tutorial de Azure Container Instances.

> [!div class="nextstepaction"]
> [Tutorial de Azure Container Instances](./container-instances-tutorial-prepare-app.md)

<!-- IMAGES -->
[aci-portal-01]: ./media/container-instances-quickstart-portal/qs-portal-01.png
[aci-portal-03]: ./media/container-instances-quickstart-portal/qs-portal-03.png
[aci-portal-04]: ./media/container-instances-quickstart-portal/qs-portal-04.png
[aci-portal-05]: ./media/container-instances-quickstart-portal/qs-portal-05.png
[aci-portal-06]: ./media/container-instances-quickstart-portal/qs-portal-06.png
[aci-portal-07]: ./media/container-instances-quickstart-portal/qs-portal-07.png
[aci-portal-08]: ./media/container-instances-quickstart-portal/qs-portal-08.png
[aci-portal-09]: ./media/container-instances-quickstart-portal/qs-portal-09.png
[aci-portal-10]: ./media/container-instances-quickstart-portal/qs-portal-10.png
[aci-portal-11]: ./media/container-instances-quickstart-portal/qs-portal-11.png

<!-- LINKS - External -->
[azure-free-account]: https://azure.microsoft.com/free/