---
title: 'Inicio rápido: Aprovisionamiento del servicio Azure Spring Cloud'
description: Describe la creación de una instancia del servicio Azure Spring Cloud para la implementación de aplicaciones.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 08/03/2020
ms.custom: devx-track-java
ms.openlocfilehash: c91237e3a14c60e477f58be0bf62f634b462960b
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88951677"
---
# <a name="quickstart-provision-azure-spring-cloud-service"></a>Inicio rápido: Aprovisionamiento del servicio Azure Spring Cloud

Puede crear instancias de Azure Spring Cloud mediante Azure Portal o la CLI de Azure.  Ambos métodos se explican en los siguientes procedimientos.
## <a name="prerequisites"></a>Requisitos previos

* [Instalación de JDK 8](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable)
* [Registro para obtener una suscripción a Azure](https://azure.microsoft.com/free/)
* (Opcional) [Instale la versión 2.0.67 de la CLI de Azure, o cualquier versión superior](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest), e instale la extensión de Azure Spring Cloud con el comando `az extension add --name spring-cloud`
* (Opcional) [Instale Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij/) e [inicie sesión](https://docs.microsoft.com/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in)

## <a name="provision-an-instance-of-azure-spring-cloud"></a>Aprovisionamiento de una instancia de Azure Spring Cloud

#### <a name="portal"></a>[Portal](#tab/Azure-portal)

En el procedimiento siguiente se crea una instancia de Azure Spring Cloud desde Azure Portal.

1. Abra [Azure Portal](https://ms.portal.azure.com/) en una pestaña ventana. 

2. En el cuadro de búsqueda superior, busque **Azure Spring Cloud**.

3. Seleccione **Azure Spring Cloud** en los resultados.

    ![Icono de inicio de ASC](media/spring-cloud-quickstart-launch-app-portal/find-spring-cloud-start.png)

4. En la página Azure Spring Cloud, haga clic en **+ Add** (+ Agregar).

    ![Icono de adición de ASC](media/spring-cloud-quickstart-launch-app-portal/spring-cloud-add.png)

5. Rellene el formulario en la página **Crear** de Azure Spring Cloud.  Tenga en cuenta las directrices siguientes:
    - **Suscripción**: seleccione la suscripción a la que desea que se facture este recurso.
    - **Grupo de recursos**: se recomienda crear grupos de recursos para los nuevos recursos. Tenga en cuenta que esto se usará en los pasos posteriores como **\<resource group name\>** .
    - **Detalles o nombre del servicio**: Especifique **\<service instance name\>** .  El nombre debe tener entre 4 y 32 caracteres, y solo puede contener números, letras minúsculas y guiones.  El primer carácter del nombre del servicio debe ser una letra y el último debe ser una letra o un número.
    - **Ubicación**: seleccione la ubicación de la instancia de servicio.

    ![Inicio del portal de ASC](media/spring-cloud-quickstart-launch-app-portal/portal-start.png)

6. Haga clic en **Revisar y crear**.

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/javae2e?tutorial=asc-cli-quickstart&step=public-endpoint)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

En el siguiente procedimiento se usa la extensión de la CLI de Azure para aprovisionar una instancia de Azure Spring Cloud.

1. Inicie sesión en la CLI de Azure y elija una suscripción activa. Asegúrese de elegir la suscripción activa que se encuentre en la lista de permitidos para Azure Spring Cloud

    ```azurecli
    az login
    az account list -o table
    az account set --subscription <Name or ID of subscription, skip if you only have 1 subscription>
    ```

1. Prepare un nombre para el servicio Azure Spring Cloud.  El nombre debe tener entre 4 y 32 caracteres, y solo puede contener números, letras minúsculas y guiones.  El primer carácter del nombre del servicio debe ser una letra y el último debe ser una letra o un número.

1. Cree un grupo de recursos que contenga el servicio Azure Spring Cloud.

    ```azurecli
    az group create --location eastus --name <resource group name>
    ```

    Más información sobre los [grupos de recursos de Azure](../azure-resource-manager/management/overview.md).

1. Abra una ventana de la CLI de Azure y ejecute los siguientes comandos para aprovisionar una instancia de Azure Spring Cloud.

    ```azurecli
    az spring-cloud create -n <service instance name> -g <resource group name>
    ```

    La instancia de servicio tardará aproximadamente cinco minutos en implementarse.
---

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Configuración de un servidor de configuración](spring-cloud-quickstart-setup-config-server.md)


