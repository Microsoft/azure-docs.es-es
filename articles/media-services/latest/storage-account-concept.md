---
title: Cuentas de almacenamiento de Azure
titleSuffix: Azure Media Services
description: Obtenga información sobre cómo crear una cuenta de almacenamiento de Azure para usarla con Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 07/01/2019
ms.author: juliako
ms.openlocfilehash: 34b1061519f91c93be2f5eb43f813b83db8305f8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87072007"
---
# <a name="azure-storage-accounts"></a>Cuentas de Azure Storage

Para iniciar la administración, el cifrado, la codificación, el análisis y el streaming de contenido multimedia en Azure, debe crear una cuenta de Media Services. Al crear una cuenta de Media Services, debe proporcionar el nombre de un recurso de cuenta de Azure Storage. La cuenta de almacenamiento especificada está asociada a su cuenta de Media Services.

La cuenta de Media Services y todas las cuentas de almacenamiento asociadas deben estar en la misma suscripción de Azure. Se recomienda encarecidamente usar cuentas de almacenamiento que se encuentren en la misma ubicación que la cuenta de Media Services para evitar costos adicionales debidos a la latencia y a la salida de datos.

Debe tener una cuenta de almacenamiento **Principal** y puede tener cualquier número de cuentas de almacenamiento **Secundarias** asociadas a su cuenta de Media Services. Media Services admite cuentas de **Uso general v2** (GPv2) o **Uso general v1** (GPv1). No se permiten cuentas de solo BLOB como **Principal**.

Se recomienda usar GPv2 para que pueda sacar provecho de las últimas características y rendimiento. Para más información sobre las cuentas de almacenamiento, consulte la [Información general acerca de la cuenta de Azure Storage](../../storage/common/storage-account-overview.md).

> [!NOTE]
> Solo el nivel de acceso frecuente se admite para su uso con Azure Media Services, aunque los demás niveles de acceso se pueden usar para reducir los costos de almacenamiento del contenido que no se usa activamente.

Hay diferentes SKU que puede elegir para la cuenta de almacenamiento. Para más información, consulte los comandos [storage accounts](/cli/azure/storage/account?view=azure-cli-latest). Si quiere experimentar con las cuentas de almacenamiento, use `--sku Standard_LRS`. Sin embargo, al seleccionar una SKU de producción debe considerar `--sku Standard_RAGRS`, que proporciona replicación geográfica para la continuidad empresarial.

## <a name="assets-in-a-storage-account"></a>Recursos de una cuenta de almacenamiento

En Media Services v3, se usan las API de almacenamiento para cargar archivos en recursos. Para obtener más información, consulte [Recursos de la versión 3 de Azure Media Services](assets-concept.md).

> [!Note]
> No intente cambiar el contenido de los contenedores de blobs que generó el SDK de Media Services sin usar las API de Media Services.

## <a name="storage-side-encryption"></a>Cifrado del lado de almacenamiento

Para proteger los recursos en reposo, estos se deben cifrar mediante el cifrado del lado de almacenamiento. En la tabla siguiente se muestra cómo funciona el cifrado del lado de almacenamiento en Media Services v3:

|Opción de cifrado|Descripción|Media Services v3|
|---|---|---|
|Cifrado de almacenamiento en Media Services| Cifrado AES-256, clave administrada por Media Services |No compatible.<sup>(1)</sup>|
|[Storage Service Encryption para datos en reposo](../../storage/common/storage-service-encryption.md)|Cifrado en el lado servidor que ofrece Azure Storage, clave administrada por Azure o por el cliente.|Compatible.|
|[Cifrado en el lado de cliente de almacenamiento](../../storage/common/storage-client-side-encryption.md)|Cifrado en el lado cliente que ofrece Azure Storage, clave administrada por el cliente en Key Vault.|No compatible.|

<sup>1</sup> En Media Services v3, el cifrado de almacenamiento (cifrado con AES-256) solo es compatible con versiones anteriores si los recursos se crearon con Media Services v2, lo que significa que la versión 3 funciona con los recursos cifrados de almacenamiento existentes, pero no permite la creación de nuevos.

## <a name="storage-account-errors"></a>Errores de cuentas de almacenamiento

El estado "Desconectado" de una cuenta de Media Services indica que la cuenta ya no tiene acceso a una o varias de las cuentas de almacenamiento asociadas debido a un cambio en las claves de acceso de almacenamiento. Media Services requiere claves de acceso de almacenamiento actualizadas para realizar muchas tareas en la cuenta.

Los siguientes son los principales escenarios que darían lugar a una cuenta de Media Services que no tiene acceso a las cuentas de almacenamiento asociadas.

|Problema|Solución|
|---|---|
|La cuenta de Media Services o las cuentas de almacenamiento asociadas se han migrado para separar las suscripciones. |Migre las cuentas de almacenamiento o la cuenta de Media Services para que todas estén en la misma suscripción. |
|La cuenta de Media Services usa una cuenta de almacenamiento asociada en una suscripción diferente ya que se trata de una cuenta en la que anteriormente esto se admitía. Todas las cuentas antiguas de Media Services se convirtieron en cuentas basadas en Azure Resource Manager modernas y tendrán el estado Desconectado. |Migre la cuenta de almacenamiento o la cuenta de Media Services para que todas estén en la misma suscripción.|

## <a name="azure-storage-firewall"></a>Firewall de Azure Storage

Azure Media Services no admite cuentas de almacenamiento con el firewall de Azure Storage o [puntos de conexión privados](../../storage/common/storage-network-security.md) habilitados.

## <a name="next-steps"></a>Pasos siguientes

Para información sobre cómo adjuntar una cuenta de almacenamiento a la cuenta de Media Services, consulte [Creación de una cuenta](./create-account-howto.md).
