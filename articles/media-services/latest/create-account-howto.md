---
title: Creación de una cuenta de Azure Media Services
description: Este tutorial le guía por los pasos que de creación de una cuenta de Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: 93f5e4d659b94bd79345a5e687de14ab6a5e8ba6
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267979"
---
# <a name="create-a-media-services-account"></a>Creación de una cuenta de Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Para iniciar el cifrado, la codificación, el análisis, la administración y el streaming de contenido multimedia en Azure, debe crear una cuenta de Media Services. La cuenta de Media Services debe asociarse con una o varias cuentas de almacenamiento. En este artículo se describen los pasos para crear una cuenta de Azure Media Services.

> [!NOTE]
> La cuenta de Media Services y todas las cuentas de almacenamiento asociadas deben estar en la misma suscripción de Azure. Se recomienda encarecidamente usar cuentas de almacenamiento que se encuentren en la misma ubicación que la cuenta de Media Services para evitar costos adicionales debidos a la latencia y a la salida de datos.

 Puede usar Azure Portal o la CLI para crear una cuenta de Media Services. Elija la pestaña para el método que quiere usar.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="portal"></a>[Portal](#tab/portal/)

## <a name="use-the-azure-portal"></a>Uso de Azure Portal

<!-- Move this section. This section should be moved to conceptual.  It doesn't belong in task based -->
Azure Portal proporciona una forma de crear rápidamente una cuenta de Azure Media Services. Puede usar la cuenta para obtener acceso a Media Services que le permiten almacenar, cifrar, codificar, administrar y transmitir contenido multimedia en Azure.

Actualmente, se puede usar [Azure Portal](https://portal.azure.com/) para:

* administrar los [eventos en directo](live-events-outputs-concept.md) de la versión 3 de Media Services, 
* ver (no administrar) los [recursos](assets-concept.md) de la versión 3, 
* [obtener información sobre el acceso a las API](./access-api-howto.md). 

Para las restantes tareas de administración (por ejemplo, [Transformaciones y trabajos](transforms-jobs-concept.md) y [Protección de contenido](content-protection-overview.md)), use la [API REST](https://aka.ms/ams-v3-rest-ref), la [CLI](https://aka.ms/ams-v3-cli-ref), o uno de los[SDK](media-services-apis-overview.md#sdks) compatibles.
<!-- Move this section. This section should be moved to conceptual.  It doesn't belong in task based -->

### <a name="use-the-azure-portal-to-create-a-media-services-account"></a>Creación de una cuenta de Media Services desde Azure Portal

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

## <a name="cli"></a>[CLI](#tab/cli/)

## <a name="use-the-azure-cli"></a>Uso de la CLI de Azure

<!-- NOTE: The following are in the includes file and are reused in other How To articles. All task based content should be in the includes folder with the task- prefix prepended to the file name. -->

### <a name="set-the-azure-subscription"></a>Establecimiento de la suscripción de Azure

[!INCLUDE [Set the Azure subscription with CLI](./includes/task-set-azure-subscription-cli.md)]

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [Create a resource group with CLI](./includes/task-create-resource-group-cli.md)]

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

[!INCLUDE [Create a storage account with CLI](./includes/task-create-storage-account-cli.md)]

### <a name="create-a-media-services-account"></a>Creación de una cuenta de Media Services

[!INCLUDE [Create a Media Services account with CLI](./includes/task-create-media-services-account-cli.md)]

### <a name="see-also"></a>Consulte también

* [CLI de Azure](/cli/azure/ams?view=azure-cli-latest)
* [Asociar un almacenamiento secundario a una cuenta de Media Services](/cli/azure/ams/account/storage?view=azure-cli-latest#az-ams-account-storage-add)

---

## <a name="next-steps"></a>Pasos siguientes

[Streaming de un archivo](stream-files-dotnet-quickstart.md)
