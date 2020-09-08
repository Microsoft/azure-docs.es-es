---
title: 'Niveles de acceso frecuente, esporádico y de archivo para blobs: Azure Storage'
description: Lea información sobre los niveles de acceso frecuente, esporádico y de archivo para Azure Blob Storage. Revise las cuentas de almacenamiento que admiten niveles. Compare opciones de almacenamiento de blob en bloques.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 03/23/2019
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.reviewer: clausjor
ms.openlocfilehash: a46597087a3eee03f7c5b8d1c9746f968ea1980d
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87849733"
---
# <a name="azure-blob-storage-hot-cool-and-archive-access-tiers"></a>Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo

Azure Storage ofrece distintos niveles de acceso que le permiten almacenar datos de objeto de blob de la manera más rentable. Entre los niveles de acceso disponibles se incluyen:

- **Frecuente**: Optimizado para almacenar datos que se consultan con frecuencia.
- **Esporádico**: Optimizado para almacenar datos a los que se accede con poca frecuencia y al menos durante 30 días.
- **De archivo**: Optimizado para almacenar datos a los que se accede muy pocas veces. Estos datos se almacenan durante al menos 180 días con requisitos de latencia flexible (del orden de horas).

Las siguientes consideraciones se aplican a los distintos niveles de acceso:

- Solo los niveles de acceso frecuente y esporádico se pueden establecer en el nivel de cuenta. El nivel de acceso de archivo no está disponible en el nivel de cuenta.
- Los niveles frecuente, esporádico y de archivo se pueden establecer en el nivel de blob durante la carga o después de esta.
- Los datos del nivel de acceso esporádico pueden tolerar una disponibilidad ligeramente inferior, pero aun así requieren una gran durabilidad, una latencia de recuperación y unas características de rendimiento similares a las de los datos de acceso frecuente. En el caso de los datos de acceso esporádico, un contrato de nivel de servicio (SLA) con una disponibilidad ligeramente inferior y unos costos de acceso mayores, en comparación con los datos de acceso frecuente, es aceptable a cambio de unos costos de almacenamiento menores.
- El almacenamiento de archivo almacena datos sin conexión y ofrece los menores costos de almacenamiento, pero los mayores costos de acceso y rehidratación de datos.

Los datos almacenados en la nube están creciendo a un ritmo exponencial. Para administrar los costos de las crecientes necesidades de almacenamiento, resulta útil organizar los datos en función de atributos como frecuencia de acceso y el período de retención planeado para optimizar costos. Los datos almacenados en la nube pueden ser diferentes según la forma en que se generan, se procesan y se accede a ellos a lo largo de su vigencia. A algunos datos se accede y se modifican activamente a lo largo de su duración. A algunos datos se accede con frecuencia al principio de su duración, mientras que el acceso cae drásticamente a medida que envejecen los datos. Algunos datos permanecen inactivos en la nube y, después de que se almacenan, no se accede a ellos prácticamente nunca.

Cada uno de los escenarios de acceso a los datos se beneficia de un distinto nivel de acceso que está optimizado para un patrón de acceso concreto. Con los niveles de acceso frecuente, esporádico y de archivo, Azure Blob Storage satisface esta necesidad de que haya niveles de acceso diferenciados con modelos de precios independientes.

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="storage-accounts-that-support-tiering"></a>Cuentas de almacenamiento que admiten niveles

La disposición de niveles de datos entre almacenamiento de objetos en acceso frecuente, esporádico y de archivo solo se admite en cuentas de Blob Storage y de uso general v2 (GPv2). Las cuentas de uso general v1 (GPv1) no admiten niveles. Los clientes pueden convertir fácilmente sus cuentas de GPv1 o de Blob Storage existentes a cuentas de GPv2 mediante Azure Portal. GPv2 proporciona nuevos precios y características para blobs, archivos y colas. Algunos cortes de características y precios solo se ofrecen en cuentas de GPv2. Evalúe el uso de cuentas de GPv2 después de revisar concienzudamente los precios. Algunas cargas de trabajo pueden ser más caras en GPv2 que en GPv1. Para más información, vea [Introducción a las cuentas de Azure Storage](../common/storage-account-overview.md).

Las cuentas de Blob Storage y GPv2 exponen el atributo **access tier** en el nivel de cuenta. Este atributo le permite especificar el nivel de acceso predeterminado de cualquier blob para el que no se haya establecido este explícitamente en el nivel de objeto. En el caso de objetos con el nivel establecido en el nivel de objeto, el nivel de cuenta no se aplica. El nivel de archivo solo puede aplicarse en el nivel de objeto. Puede cambiar entre estos niveles de acceso en cualquier momento.

## <a name="hot-access-tier"></a>Nivel de acceso frecuente

El nivel de acceso frecuente tiene mayores costos de almacenamiento que los niveles esporádico y de archivo, pero menores costos de acceso. Entre los ejemplos de escenarios de uso del nivel de acceso frecuente se incluyen:

- Los datos que están en uso o a los que se espera que se acceda (se lean y se escriban) con frecuencia.
- Los datos que se almacenan provisionalmente para el procesamiento y, posteriormente, para la migración al nivel de acceso esporádico.

## <a name="cool-access-tier"></a>Nivel de acceso esporádico

El nivel de acceso esporádico tiene menores costos de almacenamiento y mayores costos de acceso, en comparación con el almacenamiento frecuente. Este nivel está diseñado para datos que van a permanecer en el nivel de acceso esporádico durante al menos 30 días. Entre los ejemplos de escenarios de uso del nivel de acceso esporádico se incluyen:

- Conjuntos de datos de copia de seguridad y recuperación ante desastres a corto plazo.
- Contenido multimedia antiguo que no se ve con frecuencia, pero que se espera que esté disponible de inmediato cuando se acceda a él.
- Conjuntos de datos grandes que se deben almacenar de forma rentable mientras se recopilan más datos para el procesamiento futuro. (*Por ejemplo,* , almacenamiento a largo plazo de datos científicos, datos de telemetría sin procesar de una instalación de fabricación)

## <a name="archive-access-tier"></a>Nivel de acceso de archivo

El nivel de acceso de archivo tiene el menor costo de almacenamiento. Pero tiene mayores costos de recuperación de datos en comparación con los niveles de acceso frecuente y esporádico. Los datos deben permanecer en el nivel de archivo durante al menos 180 días o estar sujetos a un cargo por eliminación temprana. Los datos en el nivel de archivo pueden tardar varias horas en recuperarse en función de la prioridad de la rehidratación. En el caso de objetos pequeños, una rehidratación de alta prioridad puede recuperar el objeto del archivo en menos de 1 hora. Consulte [Rehidratación de los datos de blob desde el nivel de archivo](storage-blob-rehydration.md) para obtener más información.

Mientras un blob está en almacenamiento de archivo, los datos del blob están sin conexión y no se pueden leer, sobrescribir ni modificar. Para leer o descargar un blob en un archivo, primero debe rehidratarlo en un nivel en línea. No puede tomar instantáneas de un blob en almacenamiento de archivo. Sin embargo, los metadatos de blob permanecen en línea y se mantienen disponibles, lo que permite enumerar el blob, sus propiedades, los metadatos y las etiquetas de índice de blob. No se permite establecer ni modificar los metadatos de blobs mientras se está en el archivo; sin embargo, puede establecer y modificar las etiquetas de índice de blob. En el caso de los blobs en el nivel de archivo, las únicas operaciones válidas son GetBlobProperties, GetBlobMetadata, SetBlobTags, GetBlobTags, FindBlobsByTags, ListBlobs, SetBlobTier, CopyBlob y DeleteBlob.

Entre los ejemplos de escenarios de uso del nivel de acceso de archivo se incluyen:

- Copia de seguridad a largo plazo, copia de seguridad secundaria y conjuntos de datos de archivado
- Datos originales (sin procesar) que deben conservarse, incluso después de que se han procesado en un formato útil final.
- Datos de cumplimiento y archivado que se deben almacenar durante un largo período de tiempo y a los que casi nunca se accede.

## <a name="account-level-tiering"></a>Almacenamiento por niveles de cuenta

Dentro de la misma cuenta pueden coexistir blobs de los tres niveles de acceso. Los blobs que no tienen un nivel asignado explícitamente infieren el nivel de la configuración del nivel de acceso de la cuenta. Si el nivel de acceso procede de la cuenta, verá que la propiedad de blob **Access Tier Inferred** se establece en "true", y que la propiedad de blob **Access Tier** coincide con el nivel de cuenta. En Azure Portal, la propiedad _access tier inferred_ se muestra con el nivel de acceso de blob como **Frecuente (inferido)** o **Esporádico (inferido)** .

El cambio del nivel de acceso de cuenta se aplica a todos los objetos _access tier inferred_ almacenados en la cuenta que no tengan un nivel explícito establecido. Si cambia el nivel de acceso de la cuenta de frecuente a esporádico, solo se le cobrarán las operaciones de escritura (por 10 000) de todos los blobs sin un nivel establecido en las cuentas de GPv2. En las cuentas de Blob Storage no se realiza ningún cargo por este cambio. Si cambia el nivel de acceso de su cuenta de Blob Storage o de GPv2 de esporádico a frecuente, se le cobran tanto las operaciones de lectura (por 10 000) como las de recuperación de datos (por GB).

## <a name="blob-level-tiering"></a>Almacenamiento por niveles de blob

El almacenamiento por niveles de blob permite cargar datos en el nivel de acceso de su elección mediante las operaciones [Put Blob](/rest/api/storageservices/put-blob) o [Put Block List](/rest/api/storageservices/put-block-list), y cambiar el nivel de los datos en el nivel de objeto mediante la operación [Set Blob Tier](/rest/api/storageservices/set-blob-tier) o la característica de [administración del ciclo de vida](#blob-lifecycle-management). Puede cargar datos a su nivel de acceso necesario y, a continuación, cambiar el nivel de acceso de un blob entre los niveles de acceso frecuente, esporádico y de archivo, a medida que cambien los patrones de uso, sin tener que mover los datos entre las cuentas. Todas las solicitudes de cambio de nivel se producen inmediatamente y los cambios entre el nivel de acceso frecuente y esporádico son instantáneos. Sin embargo, al rehidratar un blob de un nivel de acceso de archivo puede tardar varias horas.

La hora del último cambio de nivel de blob se expone a través de la propiedad de blob **access tier change time**. Al sobrescribir un blob en el nivel de acceso frecuente o esporádico, el blob recién creado hereda el nivel del blob que se ha sobrescrito a menos que el nuevo nivel de acceso del blob se establezca explícitamente en la creación. Si un blob está en el nivel de archivo, no se puede sobrescribir y, por tanto, la carga del mismo blob no se permite en este escenario. 

> [!NOTE]
> El almacenamiento de archivo y el almacenamiento por niveles de blob solo admiten blobs en bloques.

### <a name="blob-lifecycle-management"></a>Administración del ciclo de vida de blobs

La administración del ciclo de vida de Blob Storage ofrece una directiva enriquecida basada en reglas que se puede usar para la transición de los datos al mejor nivel de acceso y para hacer que los datos expiren cuando finalice su ciclo de vida. Para más información, consulte [Administración del ciclo de vida de Azure Blob Storage](storage-lifecycle-management-concepts.md).  

> [!NOTE]
> Los datos almacenados en una cuenta de almacenamiento de blobs en bloques (rendimiento Premium) no se pueden organizar en niveles de acceso frecuente, esporádico o de archivo con [Set Blob Tier](/rest/api/storageservices/set-blob-tier) ni mediante la administración del ciclo de vida de Azure Blob Storage.
> Para mover datos, debe copiar los blobs de forma sincrónica desde la cuenta de almacenamiento de blobs en bloques en el nivel de acceso frecuente de otra cuenta mediante la [API Put Block From URL](/rest/api/storageservices/put-block-from-url) o una versión de AzCopy que admita esta API.
> *Put Block From URL* API copia datos de forma asíncrónica en el servidor, lo que significa que la llamada solo se completa una vez que todos los datos se han movido de la ubicación original del servidor a la ubicación de destino.

### <a name="blob-level-tiering-billing"></a>Facturación del almacenamiento por niveles de blob

Cuando un blob se carga o se mueve al nivel de acceso frecuente, esporádico o de archivo, se cobra a la tarifa correspondiente inmediatamente después del cambio de nivel.

Cuando un blob se mueve a un nivel de almacenamiento de acceso más esporádico (frecuente -> esporádico, frecuente -> archivo o esporádico -> archivo), la operación se factura como una operación de escritura del nivel de destino, donde se aplican los cargos de la operación de escritura (por 10 000) y de la escritura de datos (por GB) del nivel de destino.

Cuando un blob se mueve a un nivel más frecuente (archivo->esporádico, archivo->frecuente, o esporádico->frecuente), la operación se factura como una lectura desde el nivel de origen, donde se aplican los cargos de la operación de lectura (por 10 000) y de la recuperación de datos (por GB) del nivel de origen. También podrían aplicarse cargos por la eliminación temprana de algún blob que se haya trasladado desde el nivel de acceso esporádico o de archivo. [La rehidratación de datos desde el archivo](storage-blob-rehydration.md) lleva tiempo, y los datos se cobran a los precios de archivo hasta que los datos se restauren en línea y el nivel de blob cambie a acceso frecuente o esporádico. En la tabla siguiente se resume cómo se facturan los cambios de nivel:

| | **Cargos de escritura (operación + acceso)** | **Cargos de lectura (operación + acceso)**
| ---- | ----- | ----- |
| **Dirección de SetBlobTier** | frecuente -> esporádico,<br> frecuente -> archivo,<br> esporádico -> archivo | archivo -> esporádico,<br> archivo -> frecuente,<br> esporádico -> frecuente

### <a name="cool-and-archive-early-deletion"></a>Eliminación temprana en los niveles de acceso esporádico y de archivo

Cualquier blob que se mueva al nivel de acceso esporádico (solo cuentas de GPv2) está sujeto a un período de eliminación temprana del acceso esporádico de 30 días. Cualquier blob que se mueva al nivel de acceso esporádico (solo cuentas de GPv2) está sujeto a un período de eliminación temprana del acceso esporádico de 180 días. Este cargo se prorratea. Por ejemplo, si un blob se mueve al nivel de acceso de archivo y, a continuación, se elimina o se mueve al nivel de acceso frecuente al cabo de 45 días, se le cobrará una cuota de eliminación temprana equivalente a 135 (180 menos 45) días a partir del almacenamiento de ese blob en el nivel de acceso de archivo.

Para calcular la eliminación temprana, use la propiedad de blob, **Last-Modified**, si no ha habido cambios en el nivel de acceso. También, cuando el nivel de acceso se modificó por última vez a esporádico o archivo, puede consultar la propiedad de blob: **access-tier-change-time**. Para más información sobre las propiedades de blob, consulte el artículo sobre cómo [obtener las propiedades de blob](https://docs.microsoft.com/rest/api/storageservices/get-blob-properties).

## <a name="comparing-block-blob-storage-options"></a>Comparar opciones de almacenamiento de blobs en bloques

En la siguiente tabla se muestra una comparación del almacenamiento de blobs en bloques de rendimiento Premium, así como los niveles de acceso frecuente, esporádico y de archivo.

|                                           | **Rendimiento Premium**   | **Nivel frecuente** | **Nivel esporádico**       | **Nivel de archivo**  |
| ----------------------------------------- | ------------------------- | ------------ | ------------------- | ----------------- |
| **Disponibilidad**                          | 99,9 %                     | 99,9 %        | 99%                 | Sin conexión           |
| **Disponibilidad** <br> **(lecturas de RA-GRS)**  | N/D                       | 99,99%       | 99,9 %               | Sin conexión           |
| **Cargos de uso**                         | Costos de almacenamiento más elevados y costos de acceso y transacción más bajos | Mayores costos de almacenamiento, menores costos de acceso y transacciones | Menores costos de almacenamiento, mayores costos de acceso y transacciones | Menores costos de almacenamiento, mayores costos de acceso y transacciones |
| **Tamaño mínimo de objeto**                   | N/D                       | N/D          | N/D                 | N/D               |
| **Duración mínima del almacenamiento**              | N/D                       | N/D          | 30 días<sup>1</sup> | 180 días
| **Latency** <br> **(Tiempo hasta el primer byte)** | Milisegundos de un solo dígito | milisegundos | milisegundos        | horas<sup>2</sup> |

<sup>1</sup> Los objetos del nivel de acceso esporádico en las cuentas de GPv2 tienen una duración mínima de la retención de 30 días. Las cuentas de Blob Storage no tienen ninguna duración mínima de la retención para el nivel esporádico.

<sup>2</sup> Archive Storage admite actualmente dos prioridades de rehidratación, alta y estándar, que ofrecen diferentes latencias de recuperación. Para más información, consulte [Rehidratación de los datos de blob desde el nivel de archivo](storage-blob-rehydration.md).

> [!NOTE]
> Las cuentas de Blob Storage admiten los mismos objetivos de rendimiento y escalabilidad que las cuentas de almacenamiento de uso general v2 (GPv2). Para más información, consulte [Objetivos de escalabilidad y rendimiento de Blob Storage](scalability-targets.md).

## <a name="quickstart-scenarios"></a>Escenarios de inicio rápido

En esta sección se muestran los siguientes escenarios mediante Azure Portal y PowerShell:

- Cambio del nivel de acceso predeterminado en una cuenta de GPv2 o de Blob Storage.
- Cambio del nivel de acceso de un blob en una cuenta de GPv2 o de Blob Storage.

### <a name="change-the-default-account-access-tier-of-a-gpv2-or-blob-storage-account"></a>Cambio del nivel de acceso predeterminado en una cuenta de GPv2 o de Blob Storage

# <a name="portal"></a>[Portal](#tab/azure-portal)
1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En Azure Portal, busque y seleccione **Todos los recursos**.

1. Seleccione la cuenta de almacenamiento.

1. En **Configuración**, seleccione **Configuración** para ver y cambiar la configuración de la cuenta.

1. Seleccione el nivel de acceso correcto para sus necesidades: Establezca **Nivel de acceso** en **Esporádico** o **Frecuente**.

1. Haga clic en **Guardar** en la parte superior.

![Cambio del nivel de cuenta de almacenamiento](media/storage-tiers/account-tier.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
El siguiente script de PowerShell puede utilizarse para cambiar el nivel de cuenta. La variable `$rgName` se debe inicializar con el nombre del grupo de recursos. La variable `$accountName` se debe inicializar con el nombre de la cuenta de almacenamiento. 
```powershell
#Initialize the following with your resource group and storage account names
$rgName = ""
$accountName = ""

#Change the storage account tier to hot
Set-AzStorageAccount -ResourceGroupName $rgName -Name $accountName -AccessTier Hot
```
---

### <a name="change-the-tier-of-a-blob-in-a-gpv2-or-blob-storage-account"></a>Cambiar el nivel de un blob en una cuenta de GPv2 o de Blob Storage
# <a name="portal"></a>[Portal](#tab/azure-portal)
1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En Azure Portal, busque y seleccione **Todos los recursos**.

1. Seleccione la cuenta de almacenamiento.

1. Seleccione el contenedor y, luego, seleccione el blob.

1. En las **propiedades del blob**, seleccione **Cambiar nivel**.

1. Seleccione el nivel de acceso **Frecuente**, **Esporádico** o **Archivo**. Si actualmente el blob usa el nivel de acceso de archivo y quiere rehidratarlo como un nivel en línea, también puede seleccionar una prioridad de rehidratación **Estándar** o **Alta**.

1. En la parte inferior, seleccione **Guardar**.

![Cambio del nivel de cuenta de almacenamiento](media/storage-tiers/blob-access-tier.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
El siguiente script de PowerShell puede utilizarse para cambiar el nivel de blob. La variable `$rgName` se debe inicializar con el nombre del grupo de recursos. La variable `$accountName` se debe inicializar con el nombre de la cuenta de almacenamiento. La variable `$containerName` se debe inicializar con el nombre del contenedor. La variable `$blobName` se debe inicializar con el nombre del blob. 
```powershell
#Initialize the following with your resource group, storage account, container, and blob names
$rgName = ""
$accountName = ""
$containerName = ""
$blobName == ""

#Select the storage account and get the context
$storageAccount =Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

#Select the blob from a container
$blob = Get-AzStorageBlob -Container $containerName -Blob $blobName -Context $ctx

#Change the blob’s access tier to archive
$blob.ICloudBlob.SetStandardBlobTier("Archive")
```
---

## <a name="pricing-and-billing"></a>Precios y facturación

Todas las cuentas de almacenamiento usan un modelo de precios para el almacenamiento de blobs en bloques basado en el nivel de cada blob. Tenga en cuenta las siguientes consideraciones de facturación:

- **Costos de almacenamiento**: Además de la cantidad de datos almacenados, el costo varía en función del nivel de acceso. El costo por gigabyte disminuye a medida que el nivel es más esporádico.
- **Costos de acceso a datos**: los gastos de acceso a los datos aumentan a medida que el nivel es más esporádico. En el nivel de acceso esporádico y de archivo se cobra un cargo de acceso a datos por gigabyte por las operaciones de lectura.
- **Costos de transacciones**: hay un cargo por transacción para todos los niveles, que aumenta a medida que el nivel es más esporádico.
- **Costos de transferencia de datos de replicación geográfica**: este cargo solo se aplica a las cuentas con replicación geográfica configurada, incluidas GRS y RA-GRS. La transferencia de datos de replicación geográfica incurre en un cargo por gigabyte.
- **Costos de transferencia de datos salientes**: las transferencias de datos salientes (los datos que se transfieren fuera de una región de Azure) conllevan un cargo por el uso del ancho de banda por gigabyte, lo que es coherente con las cuentas de almacenamiento de uso general.
- **Cambio del nivel de acceso**: El cambio del nivel de acceso de cuenta comportará cargos por cambio de nivel para los blobs _access tier inferred_ almacenados en la cuenta que no tengan un nivel explícito establecido. Para más información sobre cómo cambiar el nivel de acceso de un solo blob, consulte [Facturación del almacenamiento por niveles de blob](#blob-level-tiering-billing).

> [!NOTE]
> Para más información sobre los precios de los blobs en bloque, consulte la página [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/blobs/). Para más información acercas los cargos por la transferencia de datos salientes, consulte la página [Detalles de precios de ancho de banda](https://azure.microsoft.com/pricing/details/data-transfers/).

## <a name="faq"></a>Preguntas más frecuentes

**¿Debo usar cuentas de Blob Storage o de GPv2 si quiero almacenar mis datos por niveles?**

Se recomienda usar cuentas de GPv2 en lugar de las de Blob Storage para el almacenamiento por niveles. GPv2 admite todas las características de las cuentas de Blob Storage y muchas más. Los precios entre Blob Storage y GPv2 son casi idénticos, pero algunas características nuevas y reducciones de precios solo están disponibles en cuentas de GPv2. Las cuentas de GPv1 no admiten niveles.

La estructura de precios entre las cuentas de GPv1 y GPv2 es diferente y los clientes deben evaluar con cuidado ambas antes de decidirse a usar las cuentas de GPv2. Puede convertir fácilmente una cuenta existente de Blob Storage o de GPv1 en GPv2 con un simple clic. Para más información, vea [Introducción a las cuentas de Azure Storage](../common/storage-account-overview.md).

**¿Puedo almacenar objetos en los tres niveles de acceso (frecuente, esporádico y archivo) en la misma cuenta?**

Sí. El atributo **access tier** establecido en el nivel de cuenta es el nivel de cuenta predeterminado que se aplica a todos los objetos de esa cuenta sin un nivel establecido explícito. El almacenamiento por niveles de blob permite establecer el nivel de acceso en el nivel de objeto, independientemente de cuál sea el valor del nivel de acceso en la cuenta. Los blobs de cualquiera de los tres niveles de acceso (frecuente, esporádico o de archivo) pueden existir en la misma cuenta.

**¿Puedo cambiar el nivel de acceso predeterminado de la cuenta de Blob Storage o de GPv2?**

Sí, puede cambiar el nivel de cuenta predeterminado mediante la definición del atributo **access tier** en la cuenta de almacenamiento. El cambio del nivel de la cuenta se aplica a todos los objetos almacenados en la cuenta que no tengan establecido un nivel explícito (por ejemplo, **Frecuente (inferido)** o **Esporádico (inferido)** ). Al cambiar el nivel de cuenta de frecuente a esporádico, se generan cargos de operaciones de escritura (por 10 000) en todos los blobs sin un nivel establecido en cuentas de GPv2 únicamente y, al cambiar de esporádico a frecuente, se generan cargos de operaciones de lectura (por 10 000) y de recuperación de datos (por GB) en todos los blobs en cuentas de Blob Storage y de GPv2.

**¿Puedo establecer el nivel de acceso de cuenta predeterminado en archivo?**

No. Los niveles de acceso frecuente y esporádico se pueden establecer como el nivel de acceso de cuenta predeterminado. El nivel de acceso de archivo solo puede establecerse en el nivel de objeto. En la carga de blobs, se especifica el nivel de acceso de su elección para que sea frecuente, esporádico o de archivo, independientemente del nivel de cuenta predeterminado. Esta funcionalidad permite escribir datos directamente en el nivel de archivo para ahorrar en costos desde el momento en que se crean los datos en el almacenamiento de blobs.

**¿En qué regiones están disponibles los niveles de acceso frecuente, esporádico y de archivo?**

Los niveles de acceso frecuente y esporádico, junto con el almacenamiento por niveles de blob, están disponibles en todas las regiones. El almacenamiento de archivo solo estará disponible inicialmente en regiones seleccionadas. Para obtener una lista completa, consulte [Productos disponibles por región](https://azure.microsoft.com/regions/services/).

**¿Se comportan los blobs en el nivel de acceso esporádico de forma diferente a los del nivel de acceso frecuente?**

Los blobs del nivel de acceso frecuente tienen la misma latencia que los blobs de las cuentas de GPv1, GPv2 y Blob Storage. Los blobs del nivel de acceso esporádico tiene una latencia similar (en milisegundos) a la de los blobs de las cuentas de GPv1, GPv2 y Blob Storage. Los blobs del nivel de acceso de archivo presentan varias horas de latencia en cuentas de GPv1, GPv2 y Blob Storage.

Los blobs del nivel de acceso esporádico tienen un nivel de servicio (SLA) de disponibilidad ligeramente inferior a los blobs almacenados en el nivel de acceso frecuente. Para más información, consulte [Contrato de nivel de servicio para Storage](https://azure.microsoft.com/support/legal/sla/storage/v1_5/).

**¿Las operaciones entre los niveles de acceso frecuente, esporádico y de archivo son las mismas?**

Todas las operaciones entre los niveles de acceso frecuente y esporádico son coherentes al 100 %. Todas las operaciones de archivo válidas, como GetBlobProperties, GetBlobMetadata, SetBlobTags, GetBlobTags, FindBlobsByTags, ListBlobs, SetBlobTier y DeleteBlob, son 100 % coherentes con la capacidad de acceso frecuente y esporádico. Los datos de blob no se pueden leer ni modificar en el nivel de archivo hasta que se rehidratan; solo se admiten operaciones de lectura de metadatos de blobs en el archivo. Sin embargo, las etiquetas de índice de blob se pueden leer, establecer o modificar mientras están archivadas.

**Al rehidratar un blob desde el nivel de acceso de archivo al frecuente o esporádico, ¿cómo se sabe cuándo la rehidratación está completa?**

Durante la rehidratación, se puede usar la operación de obtención de propiedades de blob para sondear el atributo **Archive Status** y confirmar cuándo ha finalizado el cambio de nivel. El estado indica "rehydrate-pending-to-hot" (rehidratación pendiente para acceso frecuente) o "rehydrate-pending-to-cool" (rehidratación pendiente para acceso esporádico), según el nivel de destino. Al finalizar, se quita la propiedad de blob archive status y la propiedad de blob **Access Tier** refleja el nuevo nivel de acceso frecuente o esporádico. Consulte [Rehidratación de los datos de blob desde el nivel de archivo](storage-blob-rehydration.md) para obtener más información.

**Después de establecer el nivel de acceso de un blob, ¿cuándo comienzan a facturarme con la tarifa adecuada?**

Cada blob siempre se factura según el nivel indicado por la propiedad **access tier** del blob. Al establecer un nuevo nivel en línea para un blob, la propiedad **Access Tier** refleja inmediatamente el nuevo nivel para todas las transiciones. Sin embargo, al rehidratar un blob desde el nivel de almacenamiento de archivo sin conexión a un nivel de almacenamiento de acceso frecuente o esporádico puede tardar varias horas. En este caso, se le factura según las tarifas de archivo hasta que la rehidratación finalice, momento en el cual la propiedad **Access Tier** refleja el nuevo nivel. Una vez rehidratado en el nivel en línea, se le cobrará dicho blob a la tarifa del nivel de almacenamiento de acceso frecuente o esporádico.

**¿Cómo determino si me aplicarán un cargo por eliminación temprana al eliminar o trasladar un blob del nivel de acceso esporádico o de archivo?**

Los blobs que se eliminan o se trasladan del nivel de acceso esporádico (solo cuentas de GPv2) o de archivo antes de 30 días y 180 días, respectivamente, generarán un cargo por eliminación temprana prorrateado. Para determinar cuánto tiempo un blob ha estado en el nivel de acceso esporádico o de archivo, compruebe la propiedad del blob **access tier change time** que proporciona una marca del último cambio del nivel de acceso. Si el nivel del blob nunca ha cambiado, puede comprobar la propiedad de blob **Last Modified**. Para más información, consulte [Eliminación temprana en los niveles de acceso esporádico y de archivo](#cool-and-archive-early-deletion).

**¿Qué herramientas y SDK de Azure admiten almacenamiento por niveles de blob y almacenamiento de archivo?**

Las herramientas de Azure Portal, PowerShell y la CLI, y las bibliotecas de cliente de .NET, Java, Python y Node.js admiten todas almacenamiento por niveles de blob y almacenamiento de archivo.  

**¿Cuántos datos se pueden almacenar en los niveles de acceso frecuente, esporádico y de archivo?**

El almacenamiento de datos, junto con otros límites, se establece en el nivel de cuenta y no por nivel de acceso. Puede elegir usar todo el límite en un nivel o entre los tres niveles. Para más información, consulte [Objetivos de escalabilidad y rendimiento para cuentas de almacenamiento estándar](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

## <a name="next-steps"></a>Pasos siguientes

Evaluación de los niveles de acceso frecuente, esporádico y de archivo en cuentas de GPv2 y Blob Storage

- [Comprobación de la disponibilidad de los niveles de acceso frecuente, esporádico o de archivo por región](https://azure.microsoft.com/regions/#services)
- [Administración del ciclo de vida de Azure Blob Storage](storage-lifecycle-management-concepts.md)
- [Obtenga información sobre la rehidratación de datos de blob desde el nivel de archivo](storage-blob-rehydration.md)
- [Determinación de si el rendimiento Premium beneficiaría a la aplicación](storage-blob-performance-tiers.md)
- [Evaluación del uso de las cuentas de almacenamiento actuales mediante la habilitación de las métricas de Azure Storage](../common/storage-enable-and-view-metrics.md)
- [Comprobación de los precios de los niveles de acceso frecuente, esporádico y de archivo en cuentas de Blob Storage y de GPv2 por región](https://azure.microsoft.com/pricing/details/storage/)
- [Detalles de precios de Transferencias de datos](https://azure.microsoft.com/pricing/details/data-transfers/)
