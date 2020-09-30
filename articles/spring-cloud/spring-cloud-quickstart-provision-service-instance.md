---
title: 'Inicio rápido: Aprovisionamiento del servicio Azure Spring Cloud'
description: Describe la creación de una instancia del servicio Azure Spring Cloud para la implementación de aplicaciones.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 09/08/2020
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 16d40c334d51a66df4a4d2d56e2fa2379dda3726
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905399"
---
# <a name="quickstart-provision-azure-spring-cloud-service"></a>Inicio rápido: Aprovisionamiento del servicio Azure Spring Cloud

::: zone pivot="programming-language-csharp"
En esta guía de inicio rápido, usará la CLI de Azure para aprovisionar una instancia del servicio Azure Spring Cloud.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1). El servicio Azure Spring Cloud admite .NET Core 3.1 y versiones posteriores.
* [CLI de Azure  versión 2.0.67 o superior](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true).
* [Git](https://git-scm.com/).

## <a name="install-azure-cli-extension"></a>Instalación de la extensión de la CLI de Azure

Compruebe que la versión de la CLI de Azure sea 2.0.67 o posterior:

```azurecli
az --version
```

Instale la extensión de Azure Spring Cloud para la CLI de Azure con el siguiente comando:

```azurecli
az extension add --name spring-cloud
```

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

1. Inicie sesión en la CLI de Azure.

    ```azurecli
    az login
    ```

1. Si tiene más de una suscripción, elija la que quiere usar para esta guía de inicio rápido.

   ```azurecli
   az account list -o table
   ```

   ```azurecli
   az account set --subscription <Name or ID of a subscription from the last step>
   ```

## <a name="provision-an-instance-of-azure-spring-cloud"></a>Aprovisionamiento de una instancia de Azure Spring Cloud

1. Cree un [grupo de recursos](../azure-resource-manager/management/overview.md) que contenga el servicio Azure Spring Cloud. El nombre del grupo de recursos puede incluir caracteres alfanuméricos, carácter de subrayado, paréntesis, guión, punto (excepto al final) y caracteres Unicode.

   ```azurecli
   az group create --location eastus --name <resource group name>
   ```

1. Aprovisione una instancia del servicio Azure Spring Cloud. El nombre de la instancia de servicio debe ser único, tener entre 4 y 32 caracteres, y solo puede contener números, letras minúsculas y guiones. El primer carácter del nombre del servicio debe ser una letra y el último debe ser una letra o un número.

    ```azurecli
    az spring-cloud create -n <service instance name> -g <resource group name>
    ```

    Este comando puede tardar varios minutos en completarse.

1. Establezca el nombre del grupo de recursos y el nombre de la instancia de servicio predeterminados para no tener que especificar estos valores repetidamente en los comandos siguientes.

   ```azurecli
   az configure --defaults group=<resource group name>
   ```

   ```azurecli
   az configure --defaults spring-cloud=<service instance name>
   ```
::: zone-end

::: zone pivot="programming-language-java"
Puede crear instancias de Azure Spring Cloud mediante Azure Portal o la CLI de Azure.  Ambos métodos se explican en los siguientes procedimientos.
## <a name="prerequisites"></a>Requisitos previos

* [Instalación de JDK 8](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable&preserve-view=true)
* [Registro para obtener una suscripción a Azure](https://azure.microsoft.com/free/)
* (Opcional) [Instale la versión 2.0.67 de la CLI de Azure, o cualquier versión superior](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true), e instale la extensión de Azure Spring Cloud con el comando `az extension add --name spring-cloud`
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

1. Inicie sesión en la CLI de Azure y elija una suscripción activa.

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
::: zone-end

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea continuar con la siguiente guía de inicio rápido de esta serie, omita este paso.

En esta guía de inicio rápido, ha creado recursos de Azure que seguirán acumulando cargos si permanecen en la suscripción. Si no prevé continuar con la siguiente guía de inicio rápido y espera que no va a necesitar estos recursos en el futuro, elimine el grupo de recursos desde el portal o con el siguiente comando en Cloud Shell:

```azurecli
az group delete --name <your resource group name; for example: helloworld-1558400876966-rg> --yes
```

En esta guía de inicio rápido, también estableció el nombre del grupo de recursos predeterminado. Si no desea continuar con la siguiente guía de inicio rápido, borre ese nombre predeterminado con el siguiente comando de la CLI:

```azurecli
az configure --defaults group=
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de un servidor de configuración](spring-cloud-quickstart-setup-config-server.md)
