---
title: Creación de una cuenta de Azure Media Services
description: Este tutorial le guía por los pasos que de creación de una cuenta de Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/15/2020
ms.author: juliako
ms.custom: devx-track-azurecli
ms.openlocfilehash: b48aa215b621ab617ef3ff99ce66d972059a4ffc
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498763"
---
# <a name="create-a-media-services-account"></a>Creación de una cuenta de Media Services

Para iniciar el cifrado, la codificación, el análisis, la administración y el streaming de contenido multimedia en Azure, debe crear una cuenta de Media Services. La cuenta de Media Services debe asociarse con una o varias cuentas de almacenamiento.

> [!NOTE]
> La cuenta de Media Services y todas las cuentas de almacenamiento asociadas deben estar en la misma suscripción de Azure. Se recomienda encarecidamente usar cuentas de almacenamiento que se encuentren en la misma ubicación que la cuenta de Media Services para evitar costos adicionales debidos a la latencia y a la salida de datos.

En este artículo se describen los pasos para crear una cuenta de Azure Media Services. Elija entre las pestañas siguientes.

## <a name="use-the-azure-portal"></a>Uso de Azure Portal

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

Azure Portal proporciona una forma de crear rápidamente una cuenta de Azure Media Services. Puede usar la cuenta para obtener acceso a Media Services que le permiten almacenar, cifrar, codificar, administrar y transmitir contenido multimedia en Azure.

Actualmente, se puede usar [Azure Portal](https://portal.azure.com/) para:

* administrar los [eventos en directo](live-events-outputs-concept.md) de la versión 3 de Media Services, 
* ver (no administrar) los [recursos](assets-concept.md) de la versión 3, 
* [obtener información sobre el acceso a las API](./access-api-howto.md). 

Para las restantes tareas de administración (por ejemplo, [Transformaciones y trabajos](transforms-jobs-concept.md) y [Protección de contenido](content-protection-overview.md)), use la [API REST](https://aka.ms/ams-v3-rest-ref), la [CLI](https://aka.ms/ams-v3-cli-ref), o uno de los[SDK](media-services-apis-overview.md#sdks) compatibles.

En este artículo se muestra cómo crear una cuenta de Media Services con Azure Portal.

### <a name="create-a-media-services-account"></a>Creación de una cuenta de Media Services

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Haga clic en **+ Crear un recurso** > **Multimedia** > **Media Services**.
1. En la sección **Creación de una cuenta de Media Services** escriba los valores necesarios.
    
    | Nombre | Descripción |
    | ---|---|
    |**Nombre de cuenta**|Escriba el nombre de la nueva cuenta de Media Services. El nombre de la cuenta de Media Services debe estar compuesto totalmente de minúsculas o de números, sin espacios, y con una longitud de entre 3 y 24 caracteres.|
    |**Suscripción**|Si tiene más de una suscripción, en la lista de suscripciones de Azure seleccione una de entre aquellas a las que tiene acceso.|
    |**Grupo de recursos**|Seleccione el recurso nuevo o existente. Un grupo de recursos es una colección de recursos que comparten ciclos de vida, permisos y directivas. Obtenga más información [aquí](../../azure-resource-manager/management/overview.md#resource-groups).|
    |**Ubicación**|Seleccione la región geográfica que se usará para almacenar los registros de elementos multimedia y de metadatos para la cuenta de Media Services. Esta región se utilizará para procesar y transmitir contenido multimedia. Solo las regiones de Media Services disponibles aparecen en la lista desplegable. |
    |**Storage Account**|Seleccione una cuenta de almacenamiento para proporcionar almacenamiento de blobs del contenido multimedia desde la cuenta de Media Services. Puede seleccionar una cuenta de almacenamiento existente en la misma región geográfica que la cuenta de Media Services o crear una nueva cuenta de almacenamiento. Se crea una nueva cuenta de almacenamiento en la misma región. Las reglas para los nombres de cuenta de almacenamiento son las mismas que para las cuentas de Media Services.<br/><br/>Debe tener una cuenta de almacenamiento **Principal** y puede tener cualquier número de cuentas de almacenamiento **Secundarias** asociadas a su cuenta de Media Services. Puede usar Azure Portal para agregar cuentas de almacenamiento secundarias. Para más información, consulte [Cuentas de Azure Storage con cuentas de Azure Media Services](storage-account-concept.md).<br/><br/>La cuenta de Media Services y todas las cuentas de almacenamiento asociadas deben estar en la misma suscripción de Azure. Se recomienda encarecidamente usar cuentas de almacenamiento que se encuentren en la misma ubicación que la cuenta de Media Services para evitar costos adicionales debidos a la latencia y a la salida de datos.|
    
1. Seleccione **Anclar al panel** para ver el progreso de la implementación de la cuenta.
1. Haga clic en **Crear** en la parte inferior del formulario.

    Cuando se crea la cuenta de Media Services, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Detenido**. Para iniciar la transmisión del contenido y aprovechar el [empaquetado dinámico](dynamic-packaging-overview.md) y el [cifrado dinámico](content-protection-overview.md), el punto de conexión de streaming desde el que va a transmitir el contenido tiene que estar en estado **Ejecutando**. 

## <a name="use-the-azure-cli"></a>Uso de la CLI de Azure

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

### <a name="set-the-azure-subscription"></a>Establecimiento de la suscripción de Azure

En el siguiente comando, proporcione el identificador de suscripción de Azure que quiere usar para la cuenta de Media Services. Para ver una lista de las suscripciones a las que tiene acceso, vaya a [Suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).

```azurecli
az account set --subscription mySubscriptionId
```

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos con el siguiente comando. Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran recursos como las cuentas de Azure Media Services y las cuentas de almacenamiento asociadas.

Puede sustituir `amsResourceGroup` por su valor.

```azurecli
az group create --name amsResourceGroup --location westus2
```

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Al crear una cuenta de Media Services, debe proporcionar el nombre de un recurso de cuenta de Azure Storage. La cuenta de almacenamiento especificada está asociada a su cuenta de Media Services. Para más información sobre cómo se utilizan las cuentas de almacenamiento en Media Services, consulte [Cuentas de almacenamiento](storage-account-concept.md).

Debe tener una cuenta de almacenamiento **Principal** y puede tener cualquier número de cuentas de almacenamiento **Secundarias** asociadas a su cuenta de Media Services. Media Services admite cuentas de **Uso general v2** (GPv2) o **Uso general v1** (GPv1). No se permiten cuentas de solo BLOB como **Principal**. Si quiere obtener más información sobre las cuentas de almacenamiento, consulte [Opciones de la cuenta de Azure Storage](../../storage/common/storage-account-overview.md). 

En este ejemplo, se va a crear una cuenta LRS estándar de uso general v2. Si quiere experimentar con las cuentas de almacenamiento, use `--sku Standard_LRS`. Sin embargo, al seleccionar una SKU de producción debe considerar `--sku Standard_RAGRS`, que proporciona replicación geográfica para la continuidad empresarial. Para más información, consulte los comandos [storage accounts](/cli/azure/storage/account?view=azure-cli-latest).
 
El siguiente comando crea una cuenta de almacenamiento que se asociará a la cuenta de Media Services. En el siguiente script, puede sustituir `storageaccountforams` por su valor. `amsResourceGroup` debe coincidir con el valor asignado al grupo de recursos del paso anterior. El nombre de la cuenta de almacenamiento debe tener una longitud inferior a 24.

```azurecli
az storage account create --name storageaccountforams \  
  --kind StorageV2 \
  --sku Standard_LRS \
  -l westus2 \
  -g amsResourceGroup
```

### <a name="create-a-media-services-account"></a>Creación de una cuenta de Media Services

El siguiente comando de la CLI de Azure crea una nueva cuenta de Media Services. Puede reemplazar los valores siguientes: `amsaccount`, `storageaccountforams` (deben coincidir con el valor que especificó para la cuenta de almacenamiento) y `amsResourceGroup` (debe coincidir con el valor que especificó para el grupo de recursos).

```azurecli
az ams account create --name amsaccount \
   -g amsResourceGroup --storage-account storageaccountforams \
   -l westus2 
```

### <a name="see-also"></a>Consulte también

* [CLI de Azure](/cli/azure/ams?view=azure-cli-latest)
* [Asociar un almacenamiento secundario a una cuenta de Media Services](/cli/azure/ams/account/storage?view=azure-cli-latest#az-ams-account-storage-add)

---

## <a name="next-steps"></a>Pasos siguientes

[Streaming de un archivo](stream-files-dotnet-quickstart.md)
