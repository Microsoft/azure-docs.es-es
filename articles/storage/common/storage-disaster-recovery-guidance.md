---
title: Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento
titleSuffix: Azure Storage
description: Azure Storage admite la conmutación por error de la cuenta en cuentas de almacenamiento con redundancia geográfica. Con la conmutación por error de la cuenta, puede iniciar el proceso de conmutación por error de la cuenta de almacenamiento si el punto de conexión principal deja de estar disponible.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 05/05/2020
ms.author: tamram
ms.reviewer: artek
ms.subservice: common
ms.openlocfilehash: e9bd2db8bcc427118a76f87e49ade422a74a11c1
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87276931"
---
# <a name="disaster-recovery-and-storage-account-failover"></a>Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento

Microsoft se esfuerza por garantizar que los servicios de Azure siempre estén disponibles. Sin embargo, es posible que se produzcan interrupciones en el servicio. Si la aplicación requiere resistencia, Microsoft recomienda usar el almacenamiento con redundancia geográfica para que los datos se copien en una segunda región. Además, los clientes deben tener implementado un plan de recuperación ante desastres para controlar una interrupción del servicio regional. Parte importante de un plan de recuperación ante desastres es preparar la conmutación por error en el punto de conexión secundario ante la eventualidad de que el punto de conexión principal deje de estar disponible.

Azure Storage admite la conmutación por error de la cuenta en cuentas de almacenamiento con redundancia geográfica. Con la conmutación por error de la cuenta, puede iniciar el proceso de conmutación por error de la cuenta de almacenamiento si el punto de conexión principal deja de estar disponible. La conmutación por error actualiza el punto de conexión secundario para convertirlo en el principal de la cuenta de almacenamiento. Una vez finalizada la conmutación por error, los clientes pueden empezar a escribir en el nuevo punto de conexión principal.

La conmutación por error de la cuenta está disponible para los tipos de cuenta de uso general v1 y v2 y de Blob Storage con implementaciones de Azure Resource Manager. La conmutación por error de la cuenta se admite para todas las regiones públicas, pero no está disponible en las nubes soberanas o nacionales en este momento.

En este artículo se describen los conceptos y el proceso que implica la conmutación por error de una cuenta y se analiza cómo preparar la cuenta de almacenamiento para la recuperación con el menor impacto en el cliente. Para aprender a iniciar la conmutación por error de una cuenta en Azure Portal o PowerShell, consulte el artículo sobre la [iniciación de la conmutación por error de una cuenta](storage-initiate-account-failover.md).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="choose-the-right-redundancy-option"></a>Elección de la opción de redundancia correcta

Azure Storage mantiene varias copias de la cuenta de almacenamiento para garantizar su durabilidad y alta disponibilidad. La opción de redundancia que elija para la cuenta depende del grado de resistencia que necesita. Para protegerse contra interrupciones regionales, configure la cuenta para el almacenamiento con redundancia geográfica, ya sea con o sin la opción de acceso de lectura desde la región secundaria:  

El **almacenamiento con redundancia geográfica (GRS) o almacenamiento con redundancia de zona geográfica (GZRS)** copia los datos de manera asincrónica en dos regiones geográficas separadas por al menos cientos de kilómetros. Si se produce una interrupción en la región primaria, la región secundaria sirve como origen redundante de los datos. Puede iniciar una conmutación por error para convertir el punto de conexión secundario en el principal.

El **almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) o almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS)** ofrece almacenamiento con redundancia geográfica con la ventaja adicional del acceso de lectura al punto de conexión secundario. Si se produce una interrupción en el punto de conexión principal, las aplicaciones configuradas para acceso de lectura al secundario y diseñadas para la alta disponibilidad pueden seguir leyendo desde el punto de conexión secundario. Microsoft recomienda RA-GZRS para obtener la máxima disponibilidad y durabilidad de sus aplicaciones.

Para más información sobre la redundancia, consulte [Redundancia de Azure Storage](storage-redundancy.md).

> [!WARNING]
> El almacenamiento con redundancia geográfica supone un riesgo de pérdida de datos. Los datos se copian en la región secundaria de manera asincrónica, lo que significa que hay un retraso entre el momento en que los datos escritos en la región primaria se escriben en la región secundaria. Si se produce una interrupción, se perderán las operaciones de escritura del punto de conexión principal que todavía no se hayan copiado en el punto de conexión secundario.

## <a name="design-for-high-availability"></a>Diseño para lograr alta disponibilidad

Es importante diseñar la aplicación para lograr alta disponibilidad desde el principio. Consulte estos recursos de Azure para instrucciones sobre cómo diseñar la aplicación y planificar la recuperación ante desastres:

- [Diseño de aplicaciones resistentes de Azure](/azure/architecture/framework/resiliency/app-design): información general de los conceptos clave para diseñar aplicaciones altamente disponibles en Azure.
- [Lista de comprobación de resistencia](/azure/architecture/checklist/resiliency-per-service): lista de comprobación para verificar que la aplicación implementa los mejores procedimientos recomendados para lograr alta disponibilidad.
- [Uso de redundancia geográfica para diseñar aplicaciones de alta disponibilidad](geo-redundant-design.md): instrucciones de diseño para compilar aplicaciones para aprovechar el almacenamiento con redundancia geográfica.
- [Tutorial: Creación de una aplicación de alta disponibilidad con Blob Storage](../blobs/storage-create-geo-redundant-storage.md): tutorial que muestra cómo compilar una aplicación altamente disponible que cambia entre puntos de conexión de manera automática a medida que se simulan errores y recuperaciones. 

Además, tenga en cuenta estos procedimientos recomendados para mantener la alta disponibilidad de los datos de Azure Storage:

- **Discos:** use [Azure Backup](https://azure.microsoft.com/services/backup/) para crear copias de seguridad de los discos de VM que usan las máquinas virtuales de Azure. Considere también la posibilidad de usar [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/) para proteger las máquinas virtuales en caso de un desastre regional.
- **Blobs en bloques:** active la [eliminación temporal](../blobs/storage-blob-soft-delete.md) para proteger contra eliminaciones o sobrescrituras de nivel de objeto o copie los blobs en bloques en otra cuenta de almacenamiento de otra región mediante[AzCopy](storage-use-azcopy.md), [Azure PowerShell](/powershell/module/az.storage/) o la [biblioteca de movimiento de datos de Azure](storage-use-data-movement-library.md).
- **Archivos:** use [AzCopy](storage-use-azcopy.md) o [Azure PowerShell](/powershell/module/az.storage/) para copiar los archivos en otra cuenta de almacenamiento en una región distinta.
- **Tablas:** use [AzCopy](storage-use-azcopy.md) para exportar datos de tabla a otra cuenta de almacenamiento en una región distinta.

## <a name="track-outages"></a>Seguimiento de las interrupciones

Los clientes se pueden suscribir al [panel de Azure Service Health](https://azure.microsoft.com/status/) para hacer el seguimiento del estado de Azure Storage y otros servicios de Azure.

Microsoft también recomienda diseñar la aplicación para prepararse para la posibilidad de errores de escritura. La aplicación debe exponer los errores de escritura de manera de avisarle sobre la posibilidad de que se produzca una interrupción en la región primaria.

## <a name="understand-the-account-failover-process"></a>Descripción del proceso de conmutación por error de una cuenta

La conmutación por error de una cuenta administrada por el cliente le permite conmutar por error toda una cuenta de almacenamiento en la región secundaria si la región principal deja de estar disponible por cualquier motivo. Cuando se fuerza una conmutación por error en la región secundaria, los clientes pueden empezar a escribir datos en el punto de conexión secundario una vez que se completa la conmutación por error. Por lo general, la conmutación por error tarda aproximadamente una hora.

### <a name="how-an-account-failover-works"></a>Funcionamiento de la conmutación por error de una cuenta

En circunstancias normales, un cliente escribe los datos en una cuenta de Azure Storage en la región primaria y esos datos se copian de forma asincrónica en la región secundaria. En la imagen siguiente se muestra el escenario donde está disponible la región primaria:

![Los clientes escriben los datos en la cuenta de almacenamiento de la región primaria](media/storage-disaster-recovery-guidance/primary-available.png)

Si el punto de conexión principal deja de estar disponible por cualquier motivo, el cliente ya no puede escribir en la cuenta de almacenamiento. En la imagen siguiente muestra el escenario en que la región primaria ha dejado de estar disponible, pero todavía no se ha realizado la recuperación:

![La región primaria no está disponible, por lo que los clientes no pueden escribir datos](media/storage-disaster-recovery-guidance/primary-unavailable-before-failover.png)

El cliente inicia la conmutación por error de la cuenta en el punto de conexión secundario. El proceso de conmutación por error actualiza la entrada DNS que proporciona Azure Storage, de modo tal que el punto de conexión secundario se convierte en el nuevo punto de conexión principal de la cuenta de almacenamiento, tal como se muestra en la imagen siguiente:

![El cliente inicia la conmutación por error de la cuenta en el punto de conexión secundario](media/storage-disaster-recovery-guidance/failover-to-secondary.png)

El acceso de escritura se restaura para las cuentas con redundancia geográfica una vez que la entrada DNS se actualiza y las solicitudes se dirigen al nuevo punto de conexión principal. Los puntos de conexión de servicio de almacenamiento existentes para blobs, tablas, colas y archivos no cambian después de la conmutación por error.

> [!IMPORTANT]
> Una vez que se completa la conmutación por error, la cuenta de almacenamiento se configura para que use la redundancia local en el nuevo punto de conexión principal. Para reanudar la replicación en el nuevo punto de conexión secundario, vuelva a configurar la cuenta para la redundancia geográfica.
>
> Tenga en cuenta que convertir una cuenta LRS en una cuenta con redundancia geográfica supone un costo. Este costo se aplica a la actualización de la cuenta de almacenamiento de la nueva región principal después de una conmutación por error.  

### <a name="anticipate-data-loss"></a>Previsión de la pérdida de datos

> [!CAUTION]
> Por lo general, la conmutación por error de una cuenta implica perder algunos datos. Es importante entender las implicaciones que tiene iniciar la conmutación por error de una cuenta.  

Como los datos se escriben de manera asincrónica desde la región primaria a la secundaria, siempre se produce un retraso antes de que una escritura en la región primaria se copie en la secundaria. Si la región primaria deja de estar disponible, puede que las escrituras más recientes todavía no se hayan copiado en la región secundaria.

Cuando se fuerza una conmutación por error, todos los datos de la región primaria se pierden cuando la región secundaria se convierte en la nueva región primaria y la cuenta de almacenamiento está configurada para usar la redundancia local. Todos los datos ya copiados en la región secundaria se conservan cuando se produce la conmutación por error. Sin embargo, los datos escritos en la región primaria que no se hayan copiado en la región secundaria se pierden de forma permanente.

La propiedad **Last Sync Time** indica la hora más reciente en que se garantiza que los datos de la región primaria se escribieron en la secundaria. Todos los datos escritos antes de la hora de la última sincronización están disponibles en la región secundaria, mientras que es posible que los datos escritos después de la hora de la última sincronización no se hayan escrito en la secundaria y pueden haberse perdido. Use esta propiedad si se produce una interrupción para calcular la cantidad de datos perdidos que puede haber al iniciar la conmutación por error de una cuenta.

Como procedimiento recomendado, diseñe la aplicación para que pueda usar la hora de la última sincronización para evaluar la pérdida de datos esperada. Por ejemplo, si registra todas las operaciones de escritura, puede comparar la hora de las últimas operaciones de escritura con la hora de la última sincronización para determinar las escrituras que no se sincronizaron en la región secundaria.

Para obtener más información sobre cómo comprobar la propiedad **Hora de la última actualización**, consulte [Comprobación de la propiedad Hora de la última sincronización de una cuenta de almacenamiento](last-sync-time-get.md).

### <a name="use-caution-when-failing-back-to-the-original-primary"></a>Precaución al conmutar por recuperación en la región primaria original

Después de conmutar por error desde la región primaria a la secundaria, la cuenta de almacenamiento se configura con redundancia local en la nueva región primaria. Después, puede volver a configurar la cuenta para la redundancia geográfica. Cuando la cuenta se vuelve a configurar para usar la redundancia geográfica después de una conmutación por error, la nueva región primaria empieza de inmediato a copiar los datos en la nueva región secundaria, que antes de la conmutación por error original era la primaria. Sin embargo, puede pasar un tiempo antes de que los datos existentes en la región primaria se copien totalmente en la nueva región secundaria.

Una vez que la cuenta de almacenamiento se vuelve a configurar para la redundancia geográfica, es posible iniciar otra conmutación por error desde la nueva región primaria de vuelta a la nueva secundaria. En este caso, la región primaria original antes de la conmutación por error vuelve a serlo y se configura para usar la redundancia local. Todos los datos de la región primaria posterior a la conmutación por error (la región secundaria original) se pierden. Si la mayoría de los datos de la cuenta de almacenamiento nunca se ha copiado en la nueva región secundaria antes de la conmutación por recuperación, podría ocurrir una pérdida de datos importante.

Para evitar que suceda esto, compruebe el valor de la propiedad **Hora de última sincronización** antes de la conmutación por recuperación. Compare la hora de última sincronización con las últimas horas en que los datos se escribieron en la nueva región primaria para evaluar la pérdida de datos esperada. 

## <a name="initiate-an-account-failover"></a>Iniciación de una conmutación por error de la cuenta

Puede iniciar la conmutación por error de una cuenta desde Azure Portal, PowerShell, la CLI de Azure o la API de proveedor de recursos de Azure Storage. Para más información sobre cómo iniciar una conmutación por error, consulte el artículo sobre la [iniciación de la conmutación por error de una cuenta](storage-initiate-account-failover.md).

## <a name="additional-considerations"></a>Consideraciones adicionales

Revise las consideraciones adicionales que se describen en esta sección para entender cómo las aplicaciones y los servicios pueden verse afectados cuando se fuerza una conmutación por error.

### <a name="storage-account-containing-archived-blobs"></a>Cuenta de almacenamiento que contiene blobs archivados

Las cuentas de almacenamiento que contienen blobs archivados admiten la conmutación por error de la cuenta. Una vez completada la conmutación por error, todos los blobs archivados deben rehidratarse en un nivel en línea para que la cuenta se pueda configurar para la redundancia geográfica.

### <a name="storage-resource-provider"></a>Proveedor de recursos de Storage

Una vez que se completa la conmutación por error, los clientes pueden volver a leer y escribir los datos de Azure Storage datos en la nueva región primaria. Sin embargo, el proveedor de recursos de Azure Storage no realiza la conmutación por error, por lo que las operaciones de administración de recursos deben seguir teniendo lugar en la región primaria. Si la región primaria no está disponible, no podrá realizar operaciones de administración en la cuenta de almacenamiento.

Dado que el proveedor de recursos de Azure Storage no realiza la conmutación por error, la propiedad [Location](/dotnet/api/microsoft.azure.management.storage.models.trackedresource.location) devolverá la ubicación principal original después de que se complete la conmutación por error.

### <a name="azure-virtual-machines"></a>Azure Virtual Machines

Las máquinas virtuales (VM) de Azure no conmutan por error como parte de la conmutación por error de una cuenta. Si la región primaria deja de estar disponible y se realiza la conmutación por error en la región secundaria, deberá volver a crear cualquier máquina virtual después de la operación. También hay una potencial pérdida de datos asociada a la conmutación por error de la cuenta. Microsoft recomienda la siguiente guía para [alta disponibilidad](../../virtual-machines/windows/manage-availability.md) y [recuperación ante desastres](../../virtual-machines/windows/backup-recovery.md), que es específica para máquinas virtuales de Azure.

### <a name="azure-unmanaged-disks"></a>Discos no administrados de Azure

Como procedimiento recomendado, Microsoft aconseja convertir los discos no administrados en discos administrados. Sin embargo, si necesita conmutar por error una cuenta que contiene discos no administrados conectados a máquinas virtuales de Azure, deberá apagar la máquina virtual antes de iniciar la conmutación por error.

Los discos no administrados se almacenan como blobs en páginas en Azure Storage. Cuando una máquina virtual se ejecuta en Azure, se conceden los discos no administrados que están conectados a la VM. La conmutación por error de una cuenta no puede continuar si hay una concesión sobre un blob. Para realizar la conmutación por error, siga estos pasos:

1. Antes de empezar, anote los nombres de los discos no administrados, sus números de unidad lógica (LUN) y la máquina virtual a la que están conectados. Si lo hace, será más fácil volver a conectar los discos después de la conmutación por error.
2. Apague la máquina virtual.
3. Elimine la máquina virtual, pero conserve los archivos VHD para los discos no administrados. Anote la hora a la que eliminó la máquina virtual.
4. Espere hasta que se actualice la **hora de la última sincronización** y sea posterior a la hora en la que eliminó la máquina virtual. Este paso es importante, porque si el punto de conexión secundario no se actualiza por completo con los archivos VHD cuando se produce la conmutación por error, es posible que la máquina virtual no funcione correctamente en la nueva región primaria.
5. Inicie la conmutación por error de la cuenta.
6. Espere hasta que se complete la conmutación por error de la cuenta y que la región secundaria se haya convertido en la nueva región primaria.
7. Cree una máquina virtual en la nueva región primaria y vuelva a conectar los discos duros virtuales.
8. Inicie la nueva máquina virtual.

Recuerde que los datos almacenados en un disco temporal se pierden cuando se apaga la máquina virtual.

## <a name="unsupported-features-and-services"></a>Características y servicios no admitidos

Las siguientes características y servicios no son compatibles con la conmutación por error de una cuenta:

- Azure File Sync no admite la conmutación por error de una cuenta de almacenamiento. No se debe realizar la conmutación por error de las cuentas de almacenamiento que contienen recursos compartidos de archivos de Azure y que se usan como puntos de conexión de nube en Azure File Sync. Si lo hace, la sincronización dejará de funcionar y también podría provocar una pérdida inesperada de datos en el caso de archivos recién organizados en capas.
- Actualmente, no se admiten cuentas de almacenamiento ADLS Gen2 (cuantas que tienen el espacio de nombres jerárquico habilitado).
- No se puede conmutar por error una cuenta de almacenamiento que contiene blobs en bloques Premium. Las cuentas de almacenamiento que admiten los blobs en bloques Premium actualmente no admiten la redundancia geográfica.
- No se puede conmutar por error una cuenta de almacenamiento que contenga contenedores habilitados por la [Directiva de inmutabilidad de gusanos](../blobs/storage-blob-immutable-storage.md). Las directivas de retención legal o retención basada en tiempo desbloqueada o bloqueada impiden la conmutación por error para mantener el cumplimiento.

## <a name="copying-data-as-an-alternative-to-failover"></a>Copia de datos como alternativa a la conmutación por error

Si la cuenta de almacenamiento está configurada para el acceso de lectura al secundario, puede diseñar la aplicación para que lea desde el punto de conexión secundario. Si prefiere no realizar la conmutación por error en caso de que se produzca una interrupción en la región primaria, puede usar herramientas como [AzCopy](storage-use-azcopy.md), [Azure PowerShell](/powershell/module/az.storage/) o la [biblioteca de movimiento de datos de Azure](../common/storage-use-data-movement-library.md) para copiar datos de la cuenta de almacenamiento que se encuentra en la región secundaria a otra cuenta de almacenamiento en una región no afectada. Luego puede apuntar sus aplicaciones a esa cuenta de almacenamiento para obtener disponibilidad de lectura y escritura.

> [!CAUTION]
> Una conmutación por error de cuenta no debe usarse como parte de la estrategia de migración de datos.

## <a name="microsoft-managed-failover"></a>Conmutación por error administrada por Microsoft

En casos extremos en los que se pierde una región debido a un desastre importante, Microsoft puede iniciar una conmutación por error regional. En este caso, no se requieren acciones por su parte. No tendrá acceso de escritura a la cuenta de almacenamiento hasta que se complete la conmutación por error administrada por Microsoft. Las aplicaciones pueden leer de la región secundaria si la cuenta de almacenamiento está configurada para RA-GRS o RA-GZRS.

## <a name="see-also"></a>Consulte también

- [Uso de redundancia geográfica para diseñar aplicaciones de alta disponibilidad](geo-redundant-design.md)
- [Iniciación de la conmutación por error de una cuenta](storage-initiate-account-failover.md)
- [Comprobación de la propiedad Hora de la última sincronización de una cuenta de almacenamiento](last-sync-time-get.md)
- [Tutorial: Creación de una aplicación de alta disponibilidad con Blob Storage](../blobs/storage-create-geo-redundant-storage.md)
