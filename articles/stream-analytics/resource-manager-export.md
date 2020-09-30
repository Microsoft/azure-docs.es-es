---
title: Exportación de plantillas de Azure Resource Manager para trabajos de Azure Stream Analytics
description: En este artículo se describe cómo exportar una plantilla de Azure Resource Manager para un trabajo de Azure Stream Analytics.
services: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 03/10/2020
ms.openlocfilehash: c421c730321213a9adbf0a0043874a21ba26bb05
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906242"
---
# <a name="export-an-azure-stream-analytics-job-azure-resource-manager-template"></a>Exportación de plantillas de Azure Resource Manager para trabajos de Azure Stream Analytics

Las plantillas de [Azure Resource Manager](../azure-resource-manager/templates/overview.md) permiten implementar la infraestructura como código. La plantilla es un archivo de notación de objetos JavaScript (JSON) que define la infraestructura y configuración de sus recursos. El usuario es quien especifica los recursos que se van a implementar y las propiedades de esos recursos.

Puede implementar un trabajo de Azure Stream Analytics mediante la exportación de la plantilla de Azure Resource Manager.

## <a name="open-a-job-in-vs-code"></a>Apertura de un trabajo en VS Code

Para poder exportar una plantilla, antes es preciso abrir un trabajo de Stream Analytics existente en Visual Studio Code. 

Para exportar un trabajo a un proyecto local, busque el trabajo que desea exportar en el **explorador de Stream Analytics** de Azure Portal. En la página **Consulta**, seleccione **Abrir en Visual Studio**. Luego, seleccione **Visual Studio Code**.

![Abrir un trabajo de Stream Analytics en Visual Studio Code](./media/resource-manager-export/open-job-vs-code.png)

Para más información sobre el uso de Visual Studio Code para administrar trabajos de Stream Analytics, consulte el [inicio rápido de Visual Studio Code](quick-create-visual-studio-code.md).

## <a name="compile-the-script"></a>Compilación del script 

El siguiente paso consiste en compilar el script del trabajo en una plantilla de Azure Resource Manager. Antes de compilar el script, asegúrese de que el trabajo tiene al menos una entrada y una salida configuradas. Si no las tiene, lo primero que debe hacer es configurarlas.

1. En Visual Studio Code, vaya al archivo *Transformation.asaql* del trabajo.

   ![Archivo Transformation.asaql en Visual Studio Code](./media/resource-manager-export/transformation-asaql.png)

1. Haga clic con el botón derecho en el archivo *Transformation.asaql* y seleccione **ASA: Compile Script** (ASA: compilar script) en el menú.

1. Observe que aparece la carpeta **Deploy** en el área de trabajo del trabajo de Stream Analytics.

1. Explore el archivo *JobTemplate.json*, que es la plantilla de Azure Resource Management que se usa para la implementación.

## <a name="complete-the-parameters-file"></a>Completar el archivo de parámetros

A continuación, complete el archivo de parámetros de la plantilla de Azure Resource Management.

1. Abra el archivo *JobTemplate.parameters.json*, que se encuentra en la carpeta **Deploy** del área de trabajo del trabajo de Stream Analytics en Visual Studio Code.

1. Observe que las claves de entrada y salida tienen valores NULL. Reemplace estos valores por las clave de acceso reales de los recursos de entrada y salida.

1. Guarde el archivo de parámetros.

## <a name="deploy-using-templates"></a>Implementación mediante plantillas

Ya está preparado para implementar un trabajo de Azure Stream Analytics mediante las plantillas de Azure Resource Manager que generó en la sección anterior.

En una ventana de PowerShell, ejecute el siguiente comando. Asegúrese de reemplazar *ResourceGroupName*, *TemplateFile* y *TemplateParameterFile* por el nombre del grupo de recursos real y las rutas de acceso completas a los archivos *JobTemplate.json* y *JobTemplate.parameters.json* de la **carpeta Deploy** del área de trabajo.

Si no tiene Azure PowerShell configurado, siga los pasos que se indican en [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-Az-ps).

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName "<your resource group>" -TemplateFile "<path to JobTemplate.json>" -TemplateParameterFile "<path to JobTemplate.parameters.json>"
```

## <a name="next-steps"></a>Pasos siguientes

* [Prueba de los trabajos de Azure Stream Analytics localmente con una entrada activa mediante Visual Studio Code](visual-studio-code-local-run-live-input.md)

* [Exploración de trabajos de Azure Stream Analytics con Visual Studio Code (versión preliminar)](visual-studio-code-explore-jobs.md)