---
title: Creación de una instancia de Azure Data Factory mediante una plantilla de Azure Resource Manager (plantilla de Resource Manager)
description: Cree una canalización de Azure Data Factory de ejemplo mediante una plantilla de Azure Resource Manager (plantilla de Resource Manager).
services: data-factory
ms.service: data-factory
tags: azure-resource-manager
ms.workload: data-services
author: djpmsft
ms.author: daperlov
ms.reviewer: maghan, jingwang
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 07/16/2020
ms.openlocfilehash: eb423ebd354adeb8273755d34323b283d53eb8b5
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87283901"
---
# <a name="quickstart-create-an-azure-data-factory-using-arm-template"></a>Inicio rápido: Creación de una instancia de Azure Data Factory mediante una plantilla de Resource Manager

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-build-your-first-pipeline-using-arm.md)
> * [Versión actual](quickstart-create-data-factory-resource-manager-template.md)

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En esta guía de inicio rápido se describe cómo usar una plantilla de Azure Resource Manager (plantilla de Resource Manager) para crear una instancia de Azure Data Factory. La canalización que ha creado en esta factoría de datos **copia** los datos de una carpeta a otra en Azure Blob Storage. Para ver un tutorial acerca de cómo **transformar** datos mediante Azure Data Factory, consulte [Tutorial: Transformación de datos con Spark](transform-data-using-spark.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

> [!NOTE]
> En este artículo no se ofrece una introducción detallada al servicio Data Factory. Para ver una introducción al servicio Azure Data Factory, consulte [Introducción al servicio Azure Data Factory](introduction.md).

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-data-factory-v2-blob-to-blob-copy%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

### <a name="azure-subscription"></a>Suscripción de Azure

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

### <a name="create-a-file"></a>Creación de un archivo

Abra un editor de texto, como el **Bloc de notas**, y cree un archivo denominado **emp.txt** con el siguiente contenido:

```emp.txt
John, Doe
Jane, Doe
```

Guarde el archivo en la carpeta **C:\ADFv2QuickStartPSH**. (Si la carpeta no existe, créela).

## <a name="review-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-data-factory-v2-blob-to-blob-copy/).

:::code language="json" source="~/quickstart-templates/101-data-factory-v2-blob-to-blob-copy/azuredeploy.json":::

Hay recursos de Azure definidos en la plantilla:

- [Microsoft.Storage/storageAccounts](/azure/templates/Microsoft.Storage/storageAccounts): Define una cuenta de almacenamiento.
- [Microsoft.DataFactory/factories](/azure/templates/microsoft.datafactory/factories): Crea una instancia de Azure Data Factory.
- [Microsoft.DataFactory/factories/linkedServices](/azure/templates/microsoft.datafactory/factories/linkedservices): Crea un servicio vinculado de Azure Data Factory.
- [Microsoft.DataFactory/factories/datasets](/azure/templates/microsoft.datafactory/factories/datasets): Crea un conjunto de datos de Azure Data Factory.
- [Microsoft.DataFactory/factories/pipelines](/azure/templates/microsoft.datafactory/factories/pipelines): Crea una canalización de Azure Data Factory.

Encontrará más ejemplos de plantillas de Azure Data Factory en la [galería de plantillas de inicio rápido](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Datafactory&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla. La plantilla crea una cuenta de Azure Data Factory, una cuenta de almacenamiento y un contenedor de blobs.

    [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-data-factory-v2-blob-to-blob-copy%2Fazuredeploy.json)

2. Seleccione o escriba los siguientes valores.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-deploy-arm-template.png" alt-text="Implementación de la plantilla de Resource Manager para ADF":::

    A menos que se especifique, utilice los valores predeterminados para crear los recursos de Azure Data Factory:

    - **Suscripción**: Seleccione una suscripción de Azure.
    - **Grupo de recursos**: Seleccione **Crear nuevo**, escriba un nombre único para el grupo de recursos y seleccione **Aceptar**.
    - **Región**: Seleccione una ubicación.  Por ejemplo *Este de EE. UU.*
    - **Nombre de factoría de datos**: Use el valor predeterminado.
    - **Ubicación**: Use el valor predeterminado.
    - **Nombre de cuenta de almacenamiento**: Use el valor predeterminado.
    - **Contenedor de blobs**: Use el valor predeterminado.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

1. Seleccione **Ir al grupo de recursos**.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-go-to-resource-group.png" alt-text="Grupo de recursos":::

2.  Verifique que la instancia de Azure Data Factory se haya creado.
    1. El nombre de la instancia de Azure Data Factory tiene el formato: datafactory\<uniqueid\>.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-sample.png" alt-text="Data Factory de ejemplo":::

2. Verifique que se haya creado la cuenta de almacenamiento.
    1. El nombre de la cuenta de almacenamiento tiene el formato: storage\<uniqueid\>.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-storage-account.png" alt-text="Storage Account":::

3. Seleccione la cuenta de almacenamiento creada y, a continuación, seleccione **Contenedores**.
    1. En la página **Contenedores**, seleccione el contenedor de blob que ha creado.
        1. El nombre del contenedor de blobs tiene el formato: blob\<uniqueid\>.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-blob-container.png" alt-text="Contenedor de blobs":::

### <a name="upload-a-file"></a>Cargar un archivo

1. En la página **Contenedores**, seleccione **Cargar**.

2. En el panel derecho, seleccione el cuadro **Archivos**, a continuación, busque y seleccione el archivo **emp.txt** que creó anteriormente.

3. Expanda el título **Avanzado**.

4. En el cuadro **Cargar en carpeta**, escriba *input*.

5. Seleccione el botón **Cargar**. Debería ver el archivo **emp.txt** y el estado de la carga en la lista.

6. Seleccione el icono **Cerrar** (**X**) para cerrar la página **Cargar blob**.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-upload-blob-file.png" alt-text="Carga del archivo en la carpeta de entrada":::

Mantenga abierta la página Contenedor, ya que puede usarla para verificar el resultado al final de esta guía de inicio rápido.

### <a name="start-trigger"></a>Inicio del desencadenador

1. Vaya a la página **Factorías de datos**, y seleccione la factoría de datos que ha creado. 

2. Seleccione el icono **Crear y supervisar**. 

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-author-monitor-tile.png" alt-text="Crear y supervisar":::

2. Seleccione la pestaña **Autor** :::image type="icon" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-author.png" border="false":::.

3. Seleccione la canalización creada: ArmtemplateSampleCopyPipeline.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-pipelines.png" alt-text="Canalización de la plantilla de Resource Manager":::

4. Seleccione **Agregar desencadenador** > **Trigger Now** (Desencadenar ahora).

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-trigger-now.png" alt-text="Desencadenador":::

5. En el panel derecho, en **Ejecución de la canalización**, seleccione **Aceptar**.

### <a name="monitor-the-pipeline"></a>Supervisar la canalización

1. Seleccione la pestaña **Supervisar** :::image type="icon" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-monitor.png" border="false":::.

2. Vea las ejecuciones de actividad asociadas con la ejecución de la canalización. En este inicio rápido, la canalización tiene solo una actividad del tipo: Copy. Por lo tanto, se ve una ejecución de dicha actividad.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-successful-run.png" alt-text="Ejecución correcta":::

### <a name="verify-the-output-file"></a>Verificación del archivo de salida

La canalización crea automáticamente una carpeta de salida en el contenedor de blobs. A continuación, copia el archivo emp.txt de la carpeta de entrada a la carpeta de salida. 

1. En Azure Portal, en la página **Contenedores**, seleccione **Actualizar** para ver la carpeta de salida. 

2. En la lista de carpetas, seleccione **output**.

3. Confirme que **emp.txt** se ha copiado en la carpeta de salida. 

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-output.png" alt-text="Salida":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Hay dos forma de eliminar los recursos que ha creado en la guía de inicio rápido. Puede [eliminar el grupo de recursos de Azure](../azure-resource-manager/management/delete-resource-group.md), lo que incluye todos los recursos del grupo de recursos. Si quiere mantener intacto el resto de recursos, elimine solo la factoría de datos que creó en este tutorial.

Al eliminar un grupo de recursos, se eliminan todos los recursos incluidas las factorías de datos. Ejecute el comando siguiente para eliminar el grupo de recursos completo: 

```azurepowershell-interactive
Remove-AzResourceGroup -ResourceGroupName $resourcegroupname
```

Si desea eliminar solo la factoría de datos, no el grupo de recursos completo, ejecute el siguiente comando: 

```azurepowershell-interactive
Remove-AzDataFactoryV2 -Name $dataFactoryName -ResourceGroupName $resourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado una instancia de Azure Data Factory mediante una plantilla de Resource Manager y ha validado la implementación. Para obtener más información sobre Azure Data Factory y Azure Resource Manager, continúe con los artículos siguientes.

- [documentación sobre Azure Data Factory](index.yml)
- Obtenga más información sobre [Azure Resource Manager](../azure-resource-manager/management/overview.md).
- Obtención de otras [plantilla de Resource Manager para Azure Data Factory](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Datafactory&pageNumber=1&sort=Popular)