---
title: 'Inicio rápido: configuración del servidor de configuración de Azure Spring Cloud'
description: Describe la configuración de Azure Spring Cloud Config Server para la implementación de aplicaciones.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 08/03/2020
ms.custom: devx-track-java
ms.openlocfilehash: 47f74b551919177b13f5a72d6eedeae3c77b9f14
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88951666"
---
# <a name="quickstart-set-up-azure-spring-cloud-configuration-server"></a>Inicio rápido: configuración del servidor de configuración de Azure Spring Cloud

Azure Spring Cloud Config Server es un servicio de configuración centralizado para sistemas distribuidos. Usa una capa de repositorio conectable que actualmente admite el almacenamiento local, Git y Subversion.  Configure el servidor de configuración para implementar aplicaciones de microservicios en Azure Spring Cloud.

## <a name="prerequisites"></a>Requisitos previos

* [Instalación de JDK 8](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable)
* [Registro para obtener una suscripción a Azure](https://azure.microsoft.com/free/)
* (Opcional) [Instale la versión 2.0.67 de la CLI de Azure, o cualquier versión superior](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest), e instale la extensión de Azure Spring Cloud con el comando `az extension add --name spring-cloud`
* (Opcional) [Instale Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij/) e [inicie sesión](https://docs.microsoft.com/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in)

## <a name="azure-spring-cloud-config-server-procedures"></a>Procedimientos de Azure Spring Cloud Config Server

#### <a name="portal"></a>[Portal](#tab/Azure-portal)

En el procedimiento siguiente se configura el servidor de configuración mediante Azure Portal para implementar el [ejemplo de Piggymetrics](spring-cloud-quickstart-sample-app-introduction.md).

1. Vaya a la página **Información general** del servicio y seleccione **Config Server**.

2. En la sección **Default repository** (Repositorio predeterminado), en **URI** seleccione "https://github.com/Azure-Samples/piggymetrics-config".

3. Seleccione **Aplicar** para guardar los cambios.

    ![Captura de pantalla del portal de ASC](media/spring-cloud-quickstart-launch-app-portal/portal-config.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

En el procedimiento siguiente se usa la CLI de Azure para configurar el servidor de configuración para implementar el [ejemplo de Piggymetrics](spring-cloud-quickstart-sample-app-introduction.md).

Configure config-server con la ubicación del repositorio de Git del proyecto:

```azurecli
az spring-cloud config-server git set -n <service instance name> --uri https://github.com/Azure-Samples/piggymetrics-config
```

---

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Compilación e implementación de aplicaciones](spring-cloud-quickstart-deploy-apps.md)
