---
title: Carga y almacenamiento en la nube con Azure Media Services | Microsoft Docs
description: Este artículo trata los conceptos de carga y almacenamiento en la nube.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 05/11/2019
ms.author: juliako
ms.openlocfilehash: 9cbb995eb3310a2263185d6fd6dba20efce37f38
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65550145"
---
# <a name="cloud-upload-and-storage"></a>Carga y almacenamiento en la nube

Para iniciar la administración, el cifrado, la codificación, el análisis y el streaming de contenido multimedia en Azure, debe crear una cuenta de Media Services. Al crear una cuenta de Media Services, debe proporcionar el nombre de un recurso de cuenta de Azure Storage. La cuenta de almacenamiento especificada está asociada a su cuenta de Media Services. 

La cuenta de Media Services y todas las cuentas de almacenamiento asociadas deben estar en la misma suscripción de Azure. Se recomienda encarecidamente usar cuentas de almacenamiento que se encuentren en la misma ubicación que la cuenta de Media Services para evitar costos adicionales debidos a la latencia y a la salida de datos.

Debe tener una cuenta de almacenamiento **Principal** y puede tener cualquier número de cuentas de almacenamiento **Secundarias** asociadas a su cuenta de Media Services. Media Services admite cuentas de **Uso general v2** (GPv2) o **Uso general v1** (GPv1). 

>[!NOTE]
> No se permiten cuentas de solo BLOB como **Principal**. 

Se recomienda usar GPv2 para que pueda tener la ventaja de elegir entre niveles de acceso frecuente y niveles de acceso esporádico. Para más información sobre las cuentas de almacenamiento, consulte la [Información general acerca de la cuenta de Azure Storage](../../storage/common/storage-account-overview.md). 

Hay diferentes SKU, puede elegir la cuenta de almacenamiento. Para más información, consulte los comandos [storage accounts](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest). Si quiere experimentar con las cuentas de almacenamiento, use `--sku Standard_LRS`. Sin embargo, al seleccionar una SKU de producción debe considerar `--sku Standard_RAGRS`, que proporciona replicación geográfica para la continuidad empresarial. 

## <a name="assets-in-a-storage-account"></a>Recursos de una cuenta de almacenamiento

En Media Services v3, se usan las API de almacenamiento para cargar archivos en recursos. Para obtener más información, consulte [concepto activos](assets-concept.md).

> [!Note]
> No debe intentar cambiar el contenido de los contenedores de blobs que generó el SDK de Media Services sin usar las API de Media Services.
 
## <a name="storage-side-encryption"></a>Cifrado del lado de almacenamiento

Para proteger los recursos en reposo, estos se deben cifrar mediante el cifrado del lado de almacenamiento. En la tabla siguiente se muestra cómo funciona el cifrado del lado de almacenamiento en Media Services v3:

|Opción de cifrado|DESCRIPCIÓN|Media Services v3|
|---|---|---|
|Cifrado de almacenamiento en Media Services| Cifrado de AES-256, clave administrada por Media Services|Incompatible<sup>(1)</sup>|
|[Storage Service Encryption para datos en reposo](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)|Cifrado en el lado del servidor que ofrece Azure Storage, clave administrada por Azure o por el cliente|Compatible|
|[Cifrado en el lado de cliente de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-client-side-encryption)|Cifrado en el lado de cliente que ofrece Azure Storage, clave administrada por el cliente en Key Vault|No compatible|

<sup>1</sup> En Media Services v3, el cifrado de almacenamiento (cifrado con AES-256) solo es compatible con versiones anteriores si los recursos se crearon con Media Services v2. Esto significa que la versión v3 funciona con los recursos cifrados de almacenamiento ya existentes pero no permitirá la creación de otros nuevos.

## <a name="storage-account-errors"></a>Errores de la cuenta de almacenamiento

El estado "Desconectado" de una cuenta de Media Services indica que la cuenta ya no tiene acceso a una o varias de las cuentas de almacenamiento conectado debido a un cambio en las claves de acceso de almacenamiento. Media Services requiere claves de acceso de almacenamiento actualizada para realizar muchas tareas en la cuenta.

Las siguientes son los principales escenarios que daría lugar a una cuenta de Media Services si no tiene acceso a las cuentas de almacenamiento conectado. 

|Problema|Solución|
|---|---|
|La cuenta de Media Services o cuentas de almacenamiento se han migrado para separar las suscripciones. |Migre las cuentas de almacenamiento o la cuenta de Media Services para que todos se encuentran en la misma suscripción. |
|La cuenta de Media Services usa una cuenta de almacenamiento conectado en una suscripción diferente que tenía una cuenta de Media Services temprana donde esto se admite. Todas las cuentas de Media Services anticipadas se convirtieron en cuentas de administrador de recursos de Azure (ARM) en función moderna y tendrán un estado desconectado. |Migre la cuenta de almacenamiento o la cuenta de Media Services para que todos se encuentran en la misma suscripción.|

## <a name="next-steps"></a>Pasos siguientes

Para información sobre cómo adjuntar una cuenta de almacenamiento a la cuenta de Media Services, consulte [Creación de una cuenta](create-account-cli-quickstart.md).
