---
title: Actualización a una cuenta de almacenamiento de uso general v2
titleSuffix: Azure Storage
description: Actualice a cuentas de almacenamiento de uso general V2 mediante Azure Portal, PowerShell o la CLI de Azure. Especifique un nivel de acceso para los datos de blobs.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 02/25/2019
ms.author: tamram
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: c80269c949ec97a09584d736371ccd11879fd82b
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89069721"
---
# <a name="upgrade-to-a-general-purpose-v2-storage-account"></a>Actualización a una cuenta de almacenamiento de uso general v2

Las cuentas de almacenamiento de uso general v2 son compatibles con las últimas características de Azure Storage, e incorporan todas las funcionalidades de las cuentas de Blob Storage y de uso general v1. Las cuentas de uso general v2 se recomiendan para la mayoría de los escenarios de almacenamiento. Las cuentas de uso general v2 ofrecen los precios de capacidad por gigabyte más bajos para Azure Storage, así como los precios de transacción más competitivos del sector. Las cuentas de uso general V2 admiten niveles de acceso de cuenta predeterminados de nivel de acceso frecuente o esporádico y de niveles de blobs entre acceso frecuente, esporádico o de archivo.

El proceso de actualizar a una cuenta de almacenamiento de uso general v2 desde la cuenta de uso general v1 o desde una cuenta de Blob Storage es sencillo. Puede usar Azure Portal, PowerShell, o la CLI de Azure para realizar la actualización. No se produce ningún tiempo de inactividad ni riesgo de pérdida de datos asociados con la actualización a una cuenta de almacenamiento de uso general v2. La actualización de la cuenta se realiza a través de una operación de Azure Resource Manager sencilla que cambia el tipo de cuenta.

> [!IMPORTANT]
> La actualización de una cuenta de uso general v1 o de Blob Storage a una de uso general v2 es permanente y no se puede deshacer.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Vaya a la cuenta de almacenamiento.
3. En la sección **Configuración**, haga clic en **Configuración**.
4. En **Tipo de cuenta**, haga clic en **Actualizar**.
5. En **Confirmar actualización**, escriba el nombre de la cuenta.
6. Haga clic en **Actualizar** en la parte inferior de la hoja.

    ![Actualización del tipo de cuenta](../blobs/media/storage-blob-account-upgrade/upgrade-to-gpv2-account.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Para actualizar una cuenta de uso general v1 a una cuenta de uso general v2 mediante PowerShell, primero hay que actualizar PowerShell con el fin de usar la versión más reciente del módulo **Az.Storage**. Para obtener información acerca de cómo instalar PowerShell, consulte [Instalación y configuración de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-Az-ps).

A continuación, llame al siguiente comando para actualizar la cuenta, pero sustituya el nombre del grupo de recursos, nombre de la cuenta de almacenamiento y nivel de acceso deseado de la cuenta.

```powershell
Set-AzStorageAccount -ResourceGroupName <resource-group> -AccountName <storage-account> -UpgradeToStorageV2 -AccessTier <Hot/Cool>
```
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para actualizar una cuenta de uso general v1 a una cuenta de uso general v2 mediante la CLI de Azure, en primer lugar instale la versión más reciente de la CLI de Azure. Para obtener información acerca de cómo instalar la CLI, consulte [Install Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) (Instalación de la CLI de Azure 2.0).

A continuación, llame al siguiente comando para actualizar la cuenta, pero sustituya el nombre del grupo de recursos, nombre de la cuenta de almacenamiento y nivel de acceso deseado de la cuenta.

```azurecli
az storage account update -g <resource-group> -n <storage-account> --set kind=StorageV2 --access-tier=<Hot/Cool>
```

---

## <a name="specify-an-access-tier-for-blob-data"></a>Especificación de un nivel de acceso para los datos de blob

Las cuentas de uso general v2 son compatibles con todos los servicios y objetos de datos de Azure Storage, pero los niveles de acceso están disponibles solo para blobs en bloques en Blob Storage. Cuando se actualiza a una cuenta de almacenamiento de uso general v2, puede especificar un nivel de acceso de cuenta predeterminado de acceso frecuente o esporádico, que indica el nivel predeterminado en el que se cargarán los datos de blobs como si no se especifica el parámetro del nivel de acceso del blob individual.

Los niveles de acceso de blobs le permiten elegir el almacenamiento más rentable en función de los patrones de uso previstos. Los blobs en bloques se pueden almacenar en niveles con acceso frecuente, esporádico o de archivo. Para más información sobre los niveles de acceso, consulte [Azure Blob Storage: Niveles de almacenamiento de acceso frecuente, de acceso esporádico y de acceso de archivo](../blobs/storage-blob-storage-tiers.md).

De forma predeterminada, se crea una nueva cuenta de almacenamiento en el nivel de almacenamiento de acceso frecuente y se puede actualizar una cuenta de almacenamiento de uso general v1 al nivel de acceso frecuente o esporádico. Si no se especifica un nivel de acceso de cuenta en la actualización, se actualizará a frecuente de forma predeterminada. Si está explorando qué nivel de acceso debe usar para la actualización, considere el escenario de uso de datos actual. Hay dos escenarios típicos del usuario para migrar a una cuenta de uso general v2:

* Tiene una cuenta de almacenamiento de uso general v1 y desea evaluar una actualización a una cuenta de almacenamiento de uso general v2 con el nivel de almacenamiento de acceso correcto para datos de blob.
* Ha decidido usar una cuenta de almacenamiento de uso general v2 o ya tiene una y quiere evaluar si debe usar el nivel de almacenamiento de acceso frecuente o esporádico para los datos de blob.

En ambos casos, la principal prioridad es estimar el costo de almacenamiento, acceso y uso de los datos almacenados en una cuenta de almacenamiento de uso general v2 y compararlo con los costos actuales.

## <a name="pricing-and-billing"></a>Precios y facturación

El proceso de actualización de la cuenta de almacenamiento v1 a una cuenta de uso general v2 es gratuito. Puede especificar el nivel de cuenta deseado durante el proceso de actualización. Si no se especifica un nivel de cuenta en la actualización, el nivel de cuenta predeterminado de la cuenta actualizada será `Hot`. Sin embargo, si se cambia el nivel de acceso de almacenamiento después de la actualización, es posible que se produzcan cambios en la factura, por lo que se recomienda especificar el nuevo nivel de cuenta durante la actualización.

Todas las cuentas de Blob Storage usan un modelo de precios para el almacenamiento de blobs basado en el nivel de cada blob. Al usar una cuenta de almacenamiento, se aplican las siguientes consideraciones de facturación:

* **Costos de almacenamiento**: además de la cantidad de datos almacenados, el costo varía en función del nivel de almacenamiento de acceso. El costo por gigabyte disminuye a medida que el nivel es más esporádico.

* **Costos de acceso a datos**: los gastos de acceso a los datos aumentan a medida que el nivel es más esporádico. En el nivel de almacenamiento de acceso esporádico y de archivo se cobra un cargo de acceso a datos por gigabyte por las operaciones de lectura.

* **Costos de transacciones**: hay un cargo por transacción para todos los niveles, que aumenta a medida que el nivel es más esporádico.

* **Costos de transferencia de datos de replicación geográfica**: este cargo solo se aplica a las cuentas con replicación geográfica configurada, incluidas GRS y RA-GRS. La transferencia de datos de replicación geográfica incurre en un cargo por gigabyte.

* **Costos de transferencia de datos salientes**: las transferencias de datos salientes (los datos que se transfieren fuera de una región de Azure) conllevan un cargo por el uso del ancho de banda por gigabyte, lo que es coherente con las cuentas de almacenamiento de uso general.

* **Cambio del nivel de almacenamiento de acceso**: el cambio del nivel de almacenamiento de acceso de la cuenta que pasa de esporádico a frecuente conlleva un cargo igual a la lectura de todos los datos existentes en la cuenta de almacenamiento. Sin embargo, cambiar el nivel de almacenamiento de acceso de la cuenta de frecuente a esporádico genera un cargo igual que escribir todos los datos en el nivel de acceso esporádico (solo cuentas de uso general v2).

> [!NOTE]
> Para más información acerca del modelo de precios de las cuentas de almacenamiento, consulte la página [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/). Para más información acercas los cargos por la transferencia de datos salientes, consulte la página [Detalles de precios de ancho de banda](https://azure.microsoft.com/pricing/details/data-transfers/).

### <a name="estimate-costs-for-your-current-usage-patterns"></a>Calculo de los costos de los patrones de uso actual

Para calcular el costo de almacenamiento y acceso a los datos de blob en una cuenta de almacenamiento de uso general v2 en un nivel específico, es preciso evaluar el patrón de uso existente o realizar una aproximación al patrón de uso esperado. En general, será preciso conocer:

* El consumo de almacenamiento de blobs, en gigabytes, incluido:
  * ¿Cuántos datos se almacenan en la cuenta de almacenamiento?
  * ¿Cómo cambia el volumen de datos mensualmente? Los datos nuevos, ¿reemplazan constantemente a los antiguos?

* El patrón de acceso principal para los datos de almacenamiento de blobs, incluido:
  * ¿Cuántos datos se leen y se escriben en la cuenta de almacenamiento?
  * ¿Cuántas operaciones de lectura frente a operaciones de escritura se producen en los datos de la cuenta de almacenamiento?

Para decidir sobre el mejor nivel de acceso para sus necesidades, puede resultar útil determinar la capacidad de datos de blob y cómo se usan esos datos. La mejor manera de hacerlo es observar las métricas de supervisión de su cuenta.

### <a name="monitoring-existing-storage-accounts"></a>Supervisión de las cuentas de almacenamiento existentes

Para supervisar las cuentas de almacenamiento existentes y recopilar estos datos, se puede usar Azure Storage Analytics, que realiza el registro y proporciona los datos de métricas de una cuenta de almacenamiento. Storage Analytics puede almacenar métricas que incluyen estadísticas de las transacciones agregadas y datos de capacidad de las solicitudes realizadas al servicio de almacenamiento de los tipos de cuenta de uso general v1, uso general v2 y almacenamiento de blobs. Estos datos se almacenan en tablas conocidas de la misma cuenta de almacenamiento.

Para más información, consulte [About Storage Analytics Metrics](https://msdn.microsoft.com/library/azure/hh343258.aspx) (Acerca de las métricas de Azure Storage Analytics) y [Storage Analytics Metrics Table Schema](https://msdn.microsoft.com/library/azure/hh343264.aspx) (Esquema de tabla de métricas de Storage Analytics)

> [!NOTE]
> Las cuentas de almacenamiento de blobs exponen el punto de conexión de Table service solo para el almacenamiento y el acceso a los datos de métricas de dicha cuenta.

Para supervisar el consumo de almacenamiento de la cuenta de almacenamiento de blobs, es preciso habilitar las métricas de capacidad.
Si está habilitado, los datos de capacidad se registran a diario en el servicio de blobs de una cuenta de almacenamiento y se registran como una entrada de tabla que se escribe en la tabla *$MetricsCapacityBlob*, en la misma cuenta de almacenamiento.

Para supervisar los patrones de acceso a los datos de la cuenta de almacenamiento de blobs, es preciso habilitar la métrica de transacción horaria desde la API. Con la métrica de transacción horaria habilitada, las transacciones por API se agregan cada hora y se registran como una entrada de tabla que se escribe en la tabla *$MetricsHourPrimaryTransactionsBlob* dentro de la misma cuenta de almacenamiento. La tabla *$MetricsHourSecondaryTransactionsBlob* registra las transacciones en el punto de conexión secundario cuando se usan cuentas de almacenamiento de RA-GRS.

> [!NOTE]
> Si tiene una cuenta de almacenamiento de uso general en la que ha almacenado blobs en páginas y discos de máquina virtual, o colas, archivos o tablas, junto con datos de blobs en bloques y en anexos, este proceso de estimación no se puede aplicar. Los datos de capacidad no diferencian los blobs en bloques de otros tipos, por lo que no proporcionan los datos de capacidad de los restantes tipos de datos. Si usa estos tipos, una metodología alternativa consiste en examinar las cantidades en la factura más reciente.

Para obtener una buena aproximación a su patrón de acceso y consumo de datos, se recomienda elegir un período de retención de métricas que sea representativo de su uso habitual y extrapolarlo. Una opción es conservar los datos de las métricas siete días y recopilar los datos todas las semanas, con el fin de realizar el análisis al final del mes. Otra opción es conservar los datos de las métricas de los últimos 30 días y recopilar y analizar los datos al final del período de 30 días.

Para obtener más información sobre cómo habilitar, recopilar y visualizar datos de métricas, consulte [Métricas de análisis de Storage](../common/storage-analytics-metrics.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

> [!NOTE]
> El almacenamiento, acceso y descarga de datos de análisis también se cobra como los datos de usuario normales.

### <a name="utilizing-usage-metrics-to-estimate-costs"></a>Utilización de métricas de uso para estimar costos

#### <a name="capacity-costs"></a>Costos de capacidad

La entrada más reciente de la tabla de métricas de capacidad *$MetricsCapacityBlob* con la clave de fila *'data'* muestra la capacidad de almacenamiento que han consumido los datos del usuario. La entrada más reciente de la tabla de métricas de capacidad *$MetricsCapacityBlob* con la clave de fila *'analytics'* muestra la capacidad de almacenamiento que han consumido los registros de análisis.

Posteriormente, esta capacidad total consumida por los datos del usuario y los registros de análisis (si está habilitado) se puede utilizar para calcular el costo del almacenamiento de datos en la cuenta de almacenamiento. También se puede usar el mismo método para calcular los costos de almacenamiento de las cuentas de almacenamiento de GPv1.

#### <a name="transaction-costs"></a>Costos de transacciones

La suma de *'TotalBillableRequests'* , en todas las entradas de una API en la tabla de métricas de transacciones indica el número total de transacciones de dicha API determinada. *Por ejemplo,* , el número total de transacciones *'GetBlob'* en un período dado se puede calcular mediante la suma del número total de solicitudes facturables para todas las entradas con la clave de fila *'user;GetBlob'* .

Para calcular los costos de transacción de las cuentas de Blob Storage, es preciso que desglose las transacciones en tres grupos, ya que tienen precios diferentes.

* Transacciones de escritura como *'PutBlob'* , *'PutBlock'* , *'PutBlockList'* , *'AppendBlock'* , *'ListBlobs'* , *'ListContainers'* , *'CreateContainer'* , *'SnapshotBlob'* y *'CopyBlob'* .
* Transacciones de eliminación como *'DeleteBlob'* y *'DeleteContainer'* .
* Las restantes transacciones.

Para calcular los costos de transacción de las cuentas de almacenamiento de GPv1, es preciso agregar todas las transacciones, independientemente de la operación o API.

#### <a name="data-access-and-geo-replication-data-transfer-costs"></a>Costos de transferencia de datos de acceso y de replicación geográfica

Aunque el análisis del almacenamiento no proporciona la cantidad de datos leídos de una cuenta de almacenamiento y escritos en ella, se puede realizar un cálculo aproximado mediante el examen de la tabla de métricas de transacciones. La suma de *'TotalIngress'* en todas las entradas de una API de la tabla de métricas de transacciones indica la cantidad total de datos de entrada, en bytes, de dicha API concreta. De igual modo, la suma de *'TotalEgress'* indica la cantidad total de datos de salida, en bytes.

Para calcular los costos de acceso a los datos en las cuentas de Blob Storage, es preciso que desglose las transacciones en dos grupos.

* La cantidad de datos que se recuperan de la cuenta de almacenamiento se puede calcular examinando la suma de *'TotalEgress'* principalmente en las operaciones *'GetBlob'* y *'CopyBlob'* .

* La cantidad de datos que se escriben en la cuenta de almacenamiento se puede calcular examinando la suma de *'TotalIngress'* principalmente en las operaciones *'PutBlob'* , *'PutBlock'* , *'CopyBlob'* y *'AppendBlock'* .

El costo de transferencia de datos de replicación geográfica para cuentas de Blob Storage también puede calcularse mediante la valoración de la cantidad de datos escritos cuando se usa una cuenta de almacenamiento GRS o RA-GRS.

> [!NOTE]
> Para ver un ejemplo más detallado sobre cómo calcular los costos del uso del nivel de almacenamiento de acceso frecuente o esporádico, eche un vistazo a la pregunta *'¿Qué son los niveles de acceso frecuente y esporádico y cómo se puede determinar el que debe usarse?'* de la página [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/).

## <a name="next-steps"></a>Pasos siguientes

* [Cree una cuenta de almacenamiento](storage-account-create.md)
