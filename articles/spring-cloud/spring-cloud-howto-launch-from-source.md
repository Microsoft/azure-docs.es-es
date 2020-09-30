---
title: 'Procedimiento: Inicio de la aplicación Spring Cloud desde el código fuente'
description: En este inicio rápido, aprenderá a iniciar una aplicación Azure Spring Cloud directamente desde el código fuente
author: MikeDodaro
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 09/03/2020
ms.author: brendm
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: a95ec76c63a35c29b061c2fddff67a28e0f8f553
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90883656"
---
# <a name="how-to-launch-your-spring-cloud-application-from-source-code"></a>Procedimiento para iniciar la aplicación Spring Cloud desde el código fuente

**Este artículo se aplica a:** ✔️ Java

Azure Spring Cloud habilita las aplicaciones de microservicios basadas en Spring Cloud en Azure.

Puede iniciar aplicaciones directamente desde el código fuente de Java o desde un archivo JAR generado previamente. En este artículo se explican los procedimientos de implementación.

En este inicio rápido se explica cómo:

> [!div class="checklist"]
> * Aprovisionar una instancia de servicio
> * Establecer un servidor de configuración para una instancia
> * Compilar una aplicación de microservicios localmente
> * Implementar cada microservicio
> * Asignar un punto de conexión público para la aplicación

## <a name="prerequisites"></a>Prerrequisitos
Antes de empezar, asegúrese de que su suscripción de Azure tiene las dependencias necesarias:

1. [Instalación de Git](https://git-scm.com/)
2. [Instalación de JDK 8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
3. [Instalación de Maven 3.0, o cualquier versión superior](https://maven.apache.org/download.cgi)
4. [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)
5. [Registro para obtener una suscripción a Azure](https://azure.microsoft.com/free/)

> [!TIP]
> Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo.  Incluye herramientas comunes de Azure preinstaladas, entre las que se incluyen las versiones más recientes de Git, JDK, Maven y la CLI de Azure. Si ha iniciado sesión en su suscripción de Azure, inicie [Azure Cloud Shell](https://shell.azure.com) desde shell.azure.com.  Para más información sobre Azure Cloud Shell, [lea la documentación](../cloud-shell/overview.md).

## <a name="install-the-azure-cli-extension"></a>Instalación de la extensión de la CLI de Azure

Instale la extensión de Azure Spring Cloud para la CLI de Azure, para lo cual debe usar el siguiente comando

```azurecli
az extension add --name spring-cloud
```

## <a name="provision-a-service-instance-using-the-azure-cli"></a>Aprovisionamiento de una instancia de servicio con la CLI de Azure

Inicie sesión en la CLI de Azure y elija una suscripción activa. 

```azurecli
az login
az account list -o table
az account set --subscription
```

Cree un grupo de recursos que contenga el servicio Azure Spring Cloud. Más información sobre los [grupos de recursos de Azure](../azure-resource-manager/management/overview.md).

```azurecli
az group create --location eastus --name <resource group name>
```

Ejecute los siguientes comandos para aprovisionar una instancia de Azure Spring Cloud. Prepare un nombre para el servicio Azure Spring Cloud. El nombre debe tener entre 4 y 32 caracteres, y solo puede contener números, letras minúsculas y guiones. El primer carácter del nombre del servicio debe ser una letra y el último debe ser una letra o un número.

```azurecli
az spring-cloud create -n <resource name> -g <resource group name>
```

La instancia de servicio tardará aproximadamente cinco minutos en implementarse.

Establezca el nombre del grupo de recursos y el nombre de instancia de Azure Spring Cloud predeterminados con los siguientes comandos:

```azurecli
az configure --defaults group=<service group name>
az configure --defaults spring-cloud=<service instance name>
```

## <a name="create-the-azure-spring-cloud-application"></a>Creación de una aplicación Azure Spring Cloud

El siguiente comando crea una aplicación Azure Spring Cloud en su suscripción.  Esto crea un servicio vacío donde se puede cargar la aplicación.

```azurecli
az spring-cloud app create -n <app-name>
```

## <a name="deploy-your-spring-cloud-application"></a>Implementación de la aplicación Spring Cloud

Puede implementar la aplicación desde un archivo JAR pregenerado o desde un repositorio de Gradle o Maven.  A continuación, instrucciones para cada caso.

### <a name="deploy-a-built-jar"></a>Implementación de un archivo JAR compilado

Para realizar la implementación desde un archivo JAR compilado en la máquina local, asegúrese de que la compilación genera un archivo [fat-JAR](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-build.html#howto-create-an-executable-jar-with-maven).

Para implementar el archivo fat-JAR en una implementación activa

```azurecli
az spring-cloud app deploy -n <app-name> --jar-path <path-to-fat-JAR>
```

Para implementar el archivo fat-JAR en una implementación específica

```azurecli
az spring-cloud app deployment create --app <app-name> -n <deployment-name> --jar-path <path-to-built-jar>
```

### <a name="deploy-from-source-code"></a>Implementación desde el código fuente

Azure Spring Cloud usa [kpack](https://github.com/pivotal/kpack) para compilar el proyecto.  Puede usar la CLI de Azure para cargar el código fuente, compilar el proyecto con kpack e implementarlo en la aplicación de destino.

> [!WARNING]
> El proyecto solo debe generar un archivo JAR con una entrada `main-class` en `MANIFEST.MF` de `target` (para implementaciones de Maven) o `build/libs` (para implementaciones de Gradle).  Si hay varios archivos JAR con entradas `main-class`, se producirá un error en la implementación.

Para los proyectos de Maven/Gradle de un solo módulo:

```azurecli
cd <path-to-maven-or-gradle-source-root>
az spring-cloud app deploy -n <app-name>
```

En caso de proyectos de Maven/Gradle con varios módulos, repita el procedimiento para cada módulo:

```azurecli
cd <path-to-maven-or-gradle-source-root>
az spring-cloud app deploy -n <app-name> --target-module <relative-path-to-module>
```

### <a name="show-deployment-logs"></a>Visualización de los registros de implementación

Revise los registros de compilación de kpack con el siguiente comando:

```azurecli
az spring-cloud app show-deploy-log -n <app-name> [-d <deployment-name>]
```

> [!NOTE]
> Los registros de kpack solo mostrarán la implementación más reciente (si la implementación se compiló a partir del origen con kpack).

## <a name="assign-a-public-endpoint-to-gateway"></a>Asignación de un punto de conexión público a una puerta de enlace

1. Abra la página **Panel de la aplicación**.
2. Seleccione la aplicación `gateway` y se mostrará la página **Detalles de la aplicación**.
3. Seleccione **Assign endpoint** (Asignar punto de conexión) para asignar un punto de conexión público a la puerta de enlace. Esta operación puede tardar unos minutos. 
4. Escriba la dirección IP pública asignada en el explorador para ver la aplicación en ejecución.

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/javae2e?tutorial=asc-source-quickstart&step=public-endpoint)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a hacer lo siguiente:

> [!div class="checklist"]
> * Aprovisionar una instancia de servicio
> * Establecer un servidor de configuración para una instancia
> * Compilar una aplicación de microservicios localmente
> * Implementar cada microservicio
> * Editar variables de entorno para las aplicaciones
> * Asignar una dirección IP pública para la puerta de enlace de aplicaciones

> [!div class="nextstepaction"]
> [Registros, métricas y seguimiento de Spring Cloud](spring-cloud-quickstart-logs-metrics-tracing.md)

Hay más ejemplos disponibles en GitHub: [Ejemplos de Azure Spring Cloud](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/service-binding-cosmosdb-sql).
