---
title: Creación de un área de trabajo de Azure Synapse mediante una plantilla de Azure Resource Manager
description: Aprenda a crear un área de trabajo de Synapse mediante una plantilla de Azure Resource Manager.
services: azure-resource-manager
author: julieMSFT
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: jrasnick
ms.date: 08/07/2020
ms.openlocfilehash: c64f3d835eeede79f937bbaadb0a54992176438d
ms.sourcegitcommit: 206629373b7c2246e909297d69f4fe3728446af5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/06/2020
ms.locfileid: "89500678"
---
# <a name="quickstart-create-an-azure-synapse-workspace-using-a-deployment-template"></a>Inicio rápido: Creación de un área de trabajo de Azure Synapse mediante una plantilla de implementación

Esta plantilla de Azure Resource Manager creará un área de trabajo de Azure Synapse con una instancia de Data Lake Storage subyacente. El área de trabajo de Azure Synapse es un límite de colaboración protegible para los procesos de análisis en Azure Synapse Analytics.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure 1](../media/template-deployments/deploy-to-azure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FSynapse%2Fmaster%2FManage%2FDeployWorkspace%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

Para examinar la plantilla, seleccione el vínculo **Visualizar** siguiente:

[![Visualizar](../media/template-deployments/template-visualize-button.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FSynapse%2Fmaster%2FManage%2FDeployWorkspace%2Fazuredeploy.json)

La plantilla define dos recursos:

- Cuenta de almacenamiento
- Área de trabajo

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir la plantilla. Esta plantilla crea un área de trabajo de Synapse.
   
   [![Implementación en Azure 2](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FSynapse%2Fmaster%2FManage%2FDeployWorkspace%2Fazuredeploy.json)

1. Escriba o actualice los siguientes valores:

   * **Suscripción**: Seleccione una suscripción de Azure.
   * **Grupo de recursos**: seleccione **Crear nuevo**, escriba un nombre único para el grupo de recursos y seleccione **Aceptar**. Un grupo de recursos nuevo facilitará la limpieza de los recursos.
   * **Región**: Seleccione una región.  Por ejemplo, **Centro de EE. UU**.
   * **Name**: Escriba un nombre del área de trabajo.
   * **Inicio de sesión de administrador de SQL**: escriba el nombre de usuario del administrador del servidor de SQL Server.
   * **Contraseña de administrador de SQL**: escriba la contraseña del administrador del servidor de SQL Server.
   * **Valores de etiqueta**: Acepte el valor predeterminado. 
   * **Revisar y crear**: Seleccionar.
   * **Crear**: Seleccionar.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Synapse Analytics y Azure Resource Manager, consulte los artículos siguientes.

- Lea la [introducción a Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) 
- Obtenga más información sobre [Azure Resource Manager](../azure-resource-manager/management/overview.md).
- [Creación e implementación de la primera plantilla de Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md)
