---
title: Configuración de un entorno de ensayo en Azure Spring Cloud | Microsoft Docs
description: Aprenda a usar la implementación blue-green con Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: brendm
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: d36c185272b7b67ffb8a21a77cc3a64f22d0df5a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90888471"
---
# <a name="set-up-a-staging-environment-in-azure-spring-cloud"></a>Configuración de un entorno de ensayo en Azure Spring Cloud

**Este artículo se aplica a:** ✔️ Java

En este artículo se aborda cómo configurar una implementación de ensayo con el patrón de implementación azul-verde en Azure Spring Cloud. La implementación azul/verde es un patrón de entrega continua de Azure DevOps que se basa en mantener una versión existente (azul) activa mientras se implementa una nueva (verde). En este artículo se muestra cómo llevar la implementación de ensayo a producción sin cambiar directamente la implementación de producción.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por hecho que ya ha implementado la aplicación PiggyMetrics, para lo que ha usado el [tutorial en el que se explica cómo iniciar una aplicación en Azure Spring Cloud](spring-cloud-quickstart-launch-app-portal.md). PiggyMetrics consta de tres aplicaciones: "gateway", "account-service" y "auth-service".  

Si desea usar otra aplicación en este ejemplo, deberá realizar un sencillo cambio en una parte pública de la aplicación.  Con este cambio se diferencia la implementación de ensayo de la producción.

>[!TIP]
> Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar las instrucciones de este artículo.  Incluye herramientas comunes de Azure preinstaladas, entre las que se incluyen las versiones más recientes de Git, JDK, Maven y la CLI de Azure. Si ha iniciado sesión en su suscripción a Azure, inicie [Azure Cloud Shell](https://shell.azure.com).  Para obtener más información, consulte [Información general de Azure Cloud Shell](../cloud-shell/overview.md).

Para configurar un entorno de ensayo en Azure Spring Cloud, siga las instrucciones de las secciones siguientes.

## <a name="install-the-azure-cli-extension"></a>Instalación de la extensión de la CLI de Azure

Para instalar la extensión de Azure Spring Cloud para la CLI de Azure, use el siguiente comando:

```azurecli
az extension add --name spring-cloud
```
    
## <a name="view-all-deployments"></a>Visualización de todas las implementaciones

Vaya a la instancia de servicio en Azure Portal y seleccione **Administración de implementación** para ver todas las implementaciones. Para ver más detalles, puede seleccionar cada implementación.

## <a name="create-a-staging-deployment"></a>Creación de una implementación de ensayo

1. En el entorno de desarrollo local, realice una pequeña modificación en la aplicación de puerta de enlace de PiggyMetrics. Por ejemplo, cambie el color en el archivo *gateway/src/main/resources/static/css/launch.css*. Hacerlo le permitirá diferenciar fácilmente las dos implementaciones. Para compilar el paquete jar, ejecute el siguiente comando: 

    ```console
    mvn clean package
    ```

1. En la CLI de Azure, cree una implementación y asígnele el nombre de la implementación de ensayo "green".

    ```azurecli
    az spring-cloud app deployment create -g <resource-group-name> -s <service-instance-name> --app gateway -n green --jar-path gateway/target/gateway.jar
    ```

1. Después de que la implementación se complete correctamente, acceda a la página de la puerta de enlace desde el **Panel de la aplicación** y vea todas las instancias en la pestaña **App Instances** (Instancias de la aplicación) de la izquierda.
  
> [!NOTE]
> El estado de la detección es *OUT_OF_SERVICE*, por lo que el tráfico no se enrutará a esta implementación antes de que se complete la verificación.

## <a name="verify-the-staging-deployment"></a>Comprobación de la implementación de ensayo

1. Vuelva a la página **Administración de implementación** y seleccione la nueva implementación. El estado de la implementación que debería aparecer es *En ejecución*. El botón **Asignar/Desasignar dominio** debe aparecer atenuado, ya que el entorno es un entorno de ensayo.

1. En el panel **Información general**, debería ver un **Punto de conexión de prueba**. Cópielo y péguelo en una nueva ventana del explorador, debería mostrarse la nueva página de PiggyMetrics.

>[!TIP]
> * Confirme que el punto de conexión de prueba termina con una barra diagonal (/), con el fin de asegurarse de que el archivo CSS se ha cargado correctamente.  
> * Si el explorador requiere que escriba las credenciales de inicio de sesión para ver la página, use [Descodificar como dirección URL ](https://www.urldecoder.org/) para descodificar el punto de conexión de prueba. La descodificación de URL devuelve una dirección URL con el formato "https://\<username>:\<password>@\<cluster-name>.test.azureapps.io/gateway/green".  Use este formulario para acceder al punto de conexión.

>[!NOTE]    
> Los valores del servidor de configuración se aplican tanto al entorno de ensayo como al de producción. Por ejemplo, si establece la ruta de acceso del contexto (`server.servlet.context-path`) para la puerta de enlace de aplicaciones en el servidor de configuración como *somepath*, la ruta de acceso a la implementación green cambia a "https://\<username>:\<password>@\<cluster-name>.test.azureapps.io/gateway/green/somepath/...".
 
 Si va a la puerta de enlace de aplicaciones pública en este momento, debería ver la página anterior sin el nuevo cambio.
    
## <a name="set-the-green-deployment-as-the-production-environment"></a>Establecimiento de la implementación green como entorno de producción

1. Una vez que haya verificado el cambio en el entorno de ensayo, puede enviarlo a producción. Vuelva a **Administración de implementación** y active la casilla de la aplicación de la **puerta de enlace**.

2. Seleccione **Set deployment** (Establecer implementación).
3. En la lista **Implementación de producción**, seleccione **Green** y, a continuación, **Aplicar**.
4. Vaya a la página **Información general** de la aplicación de la puerta de enlace. Si ya ha asignado un dominio a la aplicación de puerta de enlace, la dirección URL aparecerá en el panel **Información general**. Para ver la página de PiggyMetrics modificada, seleccione la dirección URL y vaya al sitio.

>[!NOTE]
> Después de haber establecido la implementación green como entorno de producción, la implementación anterior se convierte en la implementación de ensayo.

## <a name="modify-the-staging-deployment"></a>Modificación de la implementación de ensayo

Si no está satisfecho con el cambio, puede modificar el código de la aplicación, compilar un nuevo paquete jar y cargarlo en la implementación green mediante la CLI de Azure.

```azurecli
az spring-cloud app deploy  -g <resource-group-name> -s <service-instance-name> -n gateway -d green --jar-path gateway.jar
```

## <a name="delete-the-staging-deployment"></a>Eliminación de la implementación de ensayo

Para eliminar una implementación de ensayo desde Azure Portal, vaya a la página de la implementación de ensayo y seleccione el botón **Eliminar**.

Como alternativa, también puede eliminarla desde la CLI de Azure con el siguiente comando:

```azurecli
az spring-cloud app deployment delete -n <staging-deployment-name> -g <resource-group-name> -s <service-instance-name> --app gateway
```

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Implementación de la primera aplicación de Azure Spring Cloud](spring-cloud-quickstart.md)
