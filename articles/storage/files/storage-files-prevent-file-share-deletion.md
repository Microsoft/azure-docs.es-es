---
title: 'Evitar la eliminación accidental: recursos compartidos de archivos de Azure'
description: Obtenga información sobre la eliminación temporal para recursos compartidos de archivos de Azure y cómo se puede usar para recuperar datos y evitar la eliminación accidental.
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 05/28/2020
ms.author: rogarana
ms.subservice: files
services: storage
ms.openlocfilehash: 11940a43438b72eb8a2e9391d56806744c4c27fc
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86527819"
---
# <a name="prevent-accidental-deletion-of-azure-file-shares"></a>Evitar la eliminación accidental de recursos compartidos de archivos de Azure

Ahora Azure Storage ofrece eliminación temporal para recursos compartidos de archivos (versión preliminar). La eliminación temporal permite recuperar el recurso compartido de archivos cuando una aplicación u otro usuario de la cuenta de almacenamiento lo ha eliminado por error.

## <a name="how-soft-delete-preview-works"></a>Funcionamiento de la eliminación temporal (versión preliminar)

Cuando la eliminación temporal de recursos compartidos de archivos de Azure está habilitada, si se elimina uno de ellos, realiza la transición a un estado de eliminación temporal, en lugar de borrarse de forma permanente. Se puede configurar el tiempo durante el que los datos eliminados de forma temporal se pueden recuperar antes de que se eliminen permanentemente y durante este período de retención el recurso compartido se puede recuperar en cualquier momento. Después de recuperarlo, tanto el recurso compartido como todo su contenido, incluidas las instantáneas, se restaurarán al estado en que se encontraban antes de la eliminación. La eliminación temporal solo funciona a nivel de recurso compartido de archivos (los archivos individuales que se eliminen se borrarán de forma permanente).

La eliminación temporal se puede habilitar en recursos compartidos de archivos nuevos o existentes. La eliminación temporal también es compatible con versiones anteriores por lo que no es necesario realizar ningún cambio en las aplicaciones para aprovechar las ventajas de las protecciones que se obtienen con esta característica. 

Para eliminar de forma permanente un recurso compartido de archivos en un estado de eliminación temporal antes de su hora de expiración, debe recuperar el recurso compartido, deshabilitar la eliminación temporal y, después, volver a eliminar el recurso compartido. Luego, debe volver a habilitar la eliminación temporal, ya que los restantes recursos compartidos de archivos de esa cuenta de almacenamiento serán vulnerables a la eliminación accidental mientras la eliminación temporal esté desactivada.

En el caso de los recursos compartidos de archivos Premium eliminados temporalmente, la cuota de recursos compartidos de archivos (el tamaño aprovisionado de un recurso compartido de archivos) se usa en el cálculo de la cuota de cuenta de almacenamiento total hasta la fecha de expiración del recurso compartido con eliminación temporal, cuando este se elimina de forma completa.

## <a name="availability"></a>Disponibilidad

La eliminación temporal de los recursos compartidos de archivos de Azure (versión preliminar) está disponible en todas las capas de almacenamiento, todos los tipos de cuenta de almacenamiento y en todas las regiones en que Azure Files está disponible.

## <a name="configuration-settings"></a>Parámetros de configuración

### <a name="enabling-or-disabling-soft-delete"></a>Habilitación o deshabilitación de la eliminación temporal

La eliminación temporal de recursos compartidos de archivos está habilitada en el nivel de cuenta de almacenamiento, a causa de ello, la configuración de eliminación temporal se aplica a todos los recursos compartidos de archivos de una cuenta de almacenamiento. La eliminación temporal se puede habilitar o deshabilitar en cualquier momento. Cuando se crea una cuenta de almacenamiento, la eliminación temporal en los recursos compartidos de archivos está desactivada de forma predeterminada. La eliminación temporal también está deshabilitada de forma predeterminada en las cuentas de almacenamiento existentes. Si ha configurado la [copia de seguridad de recursos compartidos de archivos de Azure ](../../backup/azure-file-share-backup-overview.md) para un recurso compartido de archivos de Azure, la eliminación temporal de recursos compartidos de archivos de Azure se habilitará automáticamente en la cuenta de almacenamiento del recurso compartido.

Si habilita la eliminación temporal para recursos compartidos de archivos, elimina algunos y, después, deshabilita la eliminación temporal, si durante ese período se han guardado los recursos compartidos, todavía podrá acceder a ellos y recuperarlos. Al habilitar la eliminación temporal, también debe configurar el período de retención.

### <a name="retention-period"></a>Período de retención

El período de retención es la cantidad de tiempo durante el que los recursos compartidos de archivos eliminados temporalmente se almacenan y están disponibles para su recuperación. En el caso de los recursos compartidos de archivos que se eliminan explícitamente, el reloj del período de retención se pone en marcha cuando se eliminan los datos. Actualmente se puede especificar un periodo de retención que oscile entre 1 y 365 días. El período de retención de la eliminación temporal se puede cambiar en cualquier momento. Un período de retención actualizado solo se aplicará a los recursos compartidos eliminados una vez que se haya actualizado el período de retención. Los recursos compartidos eliminados antes del período de retención expirarán en función del período de retención que se ha configurado en el momento de eliminar los datos.

## <a name="pricing-and-billing"></a>Precios y facturación

Los recursos compartidos de archivos Estándar y Premium se facturan según la capacidad usada cuando se han eliminado temporalmente, en lugar de la capacidad aprovisionada. Además, los recursos compartidos de archivos Premium se facturan según la tasa de instantáneas mientras se encuentran en el estado de eliminación temporal. Los recursos compartidos de archivos Estándar se facturan según la tasa habitual mientras se encuentran en el estado de eliminación temporal. Los datos que se eliminen permanentemente después del período de retención configurado no se cobrarán.

Para obtener más información sobre los precios de Azure File Storage en general, vea la [página de precios de Azure File Storage](https://azure.microsoft.com/pricing/details/storage/files/).

Cuando se habilita inicialmente la eliminación temporal, se recomienda usar un período de retención pequeño para comprender mejor cómo afecta la característica a la facturación.

## <a name="next-steps"></a>Pasos siguientes

Para aprender a habilitar y usar la eliminación temporal, vaya a [Habilitación de la eliminación temporal](storage-files-enable-soft-delete.md).
