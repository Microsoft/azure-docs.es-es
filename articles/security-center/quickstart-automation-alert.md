---
title: Creación de una automatización de seguridad para alertas de seguridad específicas mediante una plantilla de Azure Resource Manager
description: Aprenda a crear una automatización de Azure Security Center para desencadenar una aplicación lógica, que se desencadenarán alertas de Security Center concretas mediante una plantilla de Azure Resource Manager
services: azure-resource-manager
author: memildin
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: memildin
ms.date: 08/20/2020
ms.openlocfilehash: 6c8a3d6c291435a379a637707a42f9ad7fe8dc00
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906351"
---
# <a name="quickstart-create-an-automatic-response-to-a-specific-security-alert-using-an-azure-resource-manager-template-arm-template"></a>Inicio rápido: Creación de una respuesta automática a una alerta de seguridad concreta mediante una plantilla de Azure Resource Manager

En este inicio rápido se describe cómo usar una plantilla de Azure Resource Manager para crear una automatización de flujos de trabajo que desencadene una aplicación lógica cuando Azure Security Center reciba determinadas alertas de seguridad.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-securitycenter-create-automation-for-alertnamecontains%2fazuredeploy.json)


## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Para ver una lista de los roles y permisos necesarios para trabajar con la característica de automatización de flujos de trabajo de Azure Security Center, consulte [Automatización de flujos de trabajo](workflow-automation.md).


## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-securitycenter-create-automation-for-alertnamecontains/).

:::code language="json" source="~/quickstart-templates/101-securitycenter-create-automation-for-alertnamecontains/azuredeploy.json":::

### <a name="relevant-resources"></a>Recursos relevantes

- [**Microsoft.Security/automations**](https://github.com/Azure/azure-quickstart-templates/tree/master/101-securitycenter-create-automation-for-alertnamecontains/#microsoftsecurity-resource-provider): La automatización que desencadenará la aplicación lógica al recibir una alerta de Azure Security Center que contenga una cadena específica.
- [**Microsoft.Logic/workflows**](https://github.com/Azure/azure-quickstart-templates/tree/master/101-securitycenter-create-automation-for-alertnamecontains/#microsoftlogic-resource-provider): Una aplicación lógica vacía que se puede desencadenar.

Para ver otras plantillas de inicio rápido de Security Center, consulte estas plantillas de [aportadas por la comunidad](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Security).

## <a name="deploy-the-template"></a>Implementación de la plantilla

  - **PowerShell**:
    ```azurepowershell-interactive
    New-AzResourceGroup -Name <resource-group-name> -Location <resource-group-location> #use this command when you need to create a new resource group for your deployment
    New-AzResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-securitycenter-create-automation-for-alertnamecontains/azuredeploy.json
    ```

  - **CLI**:
    ```azurecli-interactive
    az group create --name <resource-group-name> --location <resource-group-location> #use this command when you need to create a new resource group for your deployment
    az group deployment create --resource-group <my-resource-group> --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-securitycenter-create-automation-for-alertnamecontains/azuredeploy.json
    ```

  - **Portal**:

    [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-securitycenter-create-automation-for-alertnamecontains%2fazuredeploy.json)

    Para más información acerca de esta opción de implementación, consulte [Uso de un botón de implementación para implementar plantillas desde el repositorio de GitHub](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-to-azure-button).

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Use Azure Portal para comprobar que la automatización de flujos de trabajo se ha implementado. 

1. En [Azure Portal](https://portal.azure.com), abra **Security Center**.
1. En la barra de menús superior, seleccione el icono de filtro y, después, seleccione la suscripción específica en la que ha implementado la nueva automatización de flujos de trabajo.
1. En la barra lateral de Security Center, abra la **automatización de flujos de trabajo** y busque la nueva automatización.
    :::image type="content" source="./media/quickstart-automation-alert/validating-template-run.png" alt-text="Lista de automatizaciones configuradas" lightbox="./media/quickstart-automation-alert/validating-template-run.png":::
    >[!TIP]
    > Si tiene muchas automatizaciones de flujos de trabajo en la suscripción, use la opción de **filtrar por nombre**. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no la necesite, elimine la automatización de flujos de trabajo mediante Azure Portal.

1. En [Azure Portal](https://portal.azure.com), abra **Security Center**.
1. En la barra de menús superior, seleccione el icono de filtro y, después, seleccione la suscripción específica en la que ha implementado la nueva automatización de flujos de trabajo.
1. En la barra lateral de Security Center, abra la **automatización de flujos de trabajo** y busque la automatización que se va a eliminar.
    :::image type="content" source="./media/quickstart-automation-alert/deleting-workflow-automation.png" alt-text="Pasos para quitar una automatización de flujo de trabajo" lightbox="./media/quickstart-automation-alert/deleting-workflow-automation.png":::
1. Active la casilla correspondiente al elemento que se va a eliminar.
1. En la barra de herramientas, seleccione **Eliminar**.


## <a name="next-steps"></a>Pasos siguientes

Para obtener un tutorial paso a paso que le guíe en el proceso de creación de una plantilla, consulte:

> [!div class="nextstepaction"]
> [Tutorial: Creación e implementación de su primera plantilla de Resource Manager](/azure/azure-resource-manager/templates/template-tutorial-create-first-template)