---
title: Uso de cuentas de almacenamiento administradas por el cliente en Log Analytics de Azure Monitor
description: Use su propia cuenta de almacenamiento para los escenarios de Log Analytics
ms.subservice: logs
ms.topic: conceptual
author: noakup
ms.author: noakuper
ms.date: 09/03/2020
ms.openlocfilehash: 9d54e6eb84e3269eb95f8d314875474f78536652
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90526432"
---
# <a name="using-customer-managed-storage-accounts-in-azure-monitor-log-analytics"></a>Uso de cuentas de almacenamiento administradas por el cliente en Log Analytics de Azure Monitor

Log Analytics se basa en Azure Storage en diversos escenarios. Normalmente, este uso se administra automáticamente. Sin embargo, en algunos casos es necesario proporcionar y administrar la propia cuenta de almacenamiento, también denominada cuenta de almacenamiento administrada por el cliente. En este documento se detalla el uso del almacenamiento administrado por el cliente para la ingesta de registros de WAD/LAD, los escenarios específicos de Private Link y el cifrado de CMK. 

> [!NOTE]
> Se recomienda no depender del contenido que Log Analytics carga en el almacenamiento administrado por el cliente, dado que el formato y el contenido pueden cambiar.

## <a name="ingesting-azure-diagnostics-extension-logs-wadlad"></a>Ingesta de registros de extensión de Azure Diagnostics (WAD/LAD)
Los agentes de extensión de Azure Diagnostics (también denominados WAD y LAD para los agentes de Windows y Linux, respectivamente) recopilan varios registros del sistema operativo y los almacenan en una cuenta de almacenamiento administrada por el cliente. Después, puede ingerir estos registros en Log Analytics para revisarlos y analizarlos.
Procedimiento para la recopilación de registros de extensión de Azure Diagnostics de la cuenta de almacenamiento: conecte la cuenta de almacenamiento a su área de trabajo de Log Analytics como origen de datos de almacenamiento mediante [Azure Portal](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostics-extension-logs#collect-logs-from-azure-storage) o mediante una llamada a la [API de Storage Insights](https://docs.microsoft.com/rest/api/loganalytics/connectedsources/storage%20insights/createorupdate).

Tipos de datos admitidos:
* syslog
* Eventos de Windows
* Service Fabric
* Eventos de ETW
* Registros IIS

## <a name="using-private-links"></a>Uso de vínculos privados
Las cuentas de almacenamiento administradas por el cliente se requieren en algunos casos de uso cuando se usan vínculos privados para conectarse a recursos de Azure Monitor. Uno de estos casos es la ingesta de registros personalizados o de registros de IIS. Estos tipos de datos se cargan primero como blobs en una cuenta intermediaria de Azure Storage y solo entonces se ingieren en un área de trabajo. Del mismo modo, algunas soluciones de Azure Monitor pueden usar cuentas de almacenamiento para almacenar archivos grandes, como archivos de volcado Watson, que se usan en la solución Azure Security Center. 

##### <a name="private-link-scenarios-that-require-a-customer-managed-storage"></a>Escenarios de Private Link que requieren un almacenamiento administrado por el cliente
* Ingesta de registros personalizados y registros de IIS
* Permitir que la solución ASC recopile archivos de volcado Watson

### <a name="how-to-use-a-customer-managed-storage-account-over-a-private-link"></a>Procedimiento para el uso de una cuenta de almacenamiento administrada por el cliente a través de Private Link
##### <a name="workspace-requirements"></a>Requisitos del área de trabajo
Al conectarse a Azure Monitor a través de un vínculo privado, los agentes de Log Analytics solo pueden enviar registros a las áreas de trabajo vinculadas a la red a través de un vínculo privado. Esta regla requiere que configure correctamente un ámbito de Private Link de Azure Monitor (AMPLS), que lo conecte a las áreas de trabajo y luego que conecte AMPLS a la red a través de un vínculo privado. Para más información sobre el procedimiento para la configuración de AMPLS, consulte [Uso de Azure Private Link para conectar redes a Azure Monitor de forma segura](https://docs.microsoft.com/azure/azure-monitor/platform/private-link-security). 
##### <a name="storage-account-requirements"></a>Requisitos de la cuenta de almacenamiento
Para que la cuenta de almacenamiento se conecte correctamente a su vínculo privado, debe:
* Encontrarse en la red virtual o en una red emparejada y conectada a la red virtual a través de un vínculo privado. Esto permite que los agentes de la red virtual envíen registros a la cuenta de almacenamiento.
* Debe encontrarse en la misma región que el área de trabajo a la que está vinculada.
* Permitir que Azure Monitor tenga acceso a la cuenta de almacenamiento. Si eligió permitir solo el acceso de redes seleccionadas a la cuenta de almacenamiento, también debe permitir esta excepción: "permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento". Esto permite a Log Analytics leer los registros ingeridos en esta cuenta de almacenamiento.
* Si el área de trabajo también controla el tráfico de otras redes, debe configurar la cuenta de almacenamiento para permitir el tráfico entrante procedente de las redes pertinentes o Internet.

##### <a name="link-your-storage-account-to-a-log-analytics-workspace"></a>Vinculación de la cuenta de almacenamiento a un área de trabajo de Log Analytics
Puede vincular la cuenta de almacenamiento al área de trabajo a través de la [CLI de Azure](https://docs.microsoft.com/cli/azure/monitor/log-analytics/workspace/linked-storage) o la [API de REST](https://docs.microsoft.com/rest/api/loganalytics/linkedstorageaccounts). Valores de dataSourceType aplicables:
* CustomLogs: para usar el almacenamiento para los registros personalizados y los registros de IIS durante la ingesta.
* AzureWatson: para usar el almacenamiento para los archivos de volcado Watson cargados por la solución ASC (Azure Security Center). Para obtener más información sobre la administración de la retención, el reemplazo de una cuenta de almacenamiento vinculada y la supervisión de la actividad de la cuenta de almacenamiento, consulte [Administración de cuentas de almacenamiento vinculadas](#managing-linked-storage-accounts). 

## <a name="encrypting-data-with-cmk"></a>Cifrado de datos con CMK
Azure Storage cifra todos los datos en reposo de una cuenta de almacenamiento. De manera predeterminada, cifra los datos con claves administradas por Microsoft (MMK). Sin embargo, en su lugar, Azure Storage le permitirá usar una clave administrada por el cliente (CMK) de Azure Key Vault para cifrar los datos de almacenamiento. Puede importar sus propias claves a Azure Key Vault, o puede usar las API de Azure Key Vault para generarlas.
##### <a name="cmk-scenarios-that-require-a-customer-managed-storage-account"></a>Escenarios de CMK que requieren una cuenta de almacenamiento administrada por el cliente
* Cifrado de consultas de alertas de registro con CMK
* Cifrado de consultas guardadas con CMK

### <a name="how-to-apply-cmk-to-customer-managed-storage-accounts"></a>Cómo aplicar CMK a cuentas de almacenamiento administradas por el cliente
##### <a name="storage-account-requirements"></a>Requisitos de la cuenta de almacenamiento
La cuenta de almacenamiento y el almacén de claves deben estar en la misma región, pero pueden estar en distintas suscripciones. Para más información sobre cifrado de Azure Storage y la administración de claves, consulte [Cifrado de Azure Storage para datos en reposo](https://docs.microsoft.com/azure/storage/common/storage-service-encryption).

##### <a name="apply-cmk-to-your-storage-accounts"></a>Aplicación de CMK a cuentas de almacenamiento
Para configurar la cuenta de almacenamiento de Azure para usar claves administradas por el cliente con Azure Key Vault use [Azure Portal](https://docs.microsoft.com/azure/storage/common/storage-encryption-keys-portal?toc=/azure/storage/blobs/toc.json), [PowerShell](https://docs.microsoft.com/azure/storage/common/storage-encryption-keys-powershell?toc=/azure/storage/blobs/toc.json) o la [CLI](https://docs.microsoft.com/azure/storage/common/storage-encryption-keys-cli?toc=/azure/storage/blobs/toc.json). 

## <a name="managing-linked-storage-accounts"></a>Administración de cuentas de almacenamiento vinculadas

Para vincular o desvincular cuentas de almacenamiento del área de trabajo, use la [CLI de Azure](https://docs.microsoft.com/cli/azure/monitor/log-analytics/workspace/linked-storage) o la [API de REST](https://docs.microsoft.com/rest/api/loganalytics/linkedstorageaccounts).

##### <a name="create-or-modify-a-link"></a>Creación o modificación de un vínculo
Una vez que vincule una cuenta de almacenamiento a un área de trabajo, Log Analytics comenzará a usarla en lugar de la cuenta de almacenamiento propiedad del servicio. Puede 
* Registrar varias cuentas de almacenamiento para distribuir la carga de registros entre ellas
* Reutilizar la misma cuenta de almacenamiento para varias áreas de trabajo

##### <a name="unlink-a-storage-account"></a>Desvinculación de una cuenta de almacenamiento
Para dejar de usar una cuenta de almacenamiento, desvincule el almacenamiento del área de trabajo. Al desvincular todas las cuentas de almacenamiento de un área de trabajo, Log Analytics intentará basarse en las cuentas de almacenamiento administradas por el servicio. Si la red tiene acceso limitado a Internet, es posible que estos almacenamientos no estén disponibles y que se produzca un error en cualquier escenario que se base en el almacenamiento.

##### <a name="replace-a-storage-account"></a>Reemplazo de una cuenta de almacenamiento
Para reemplazar una cuenta de almacenamiento usada para la ingesta,
1.  **Cree un vínculo a una nueva cuenta de almacenamiento.** Los agentes de registro obtendrán la configuración actualizada y comenzarán a enviar datos también al almacenamiento nuevo. El proceso puede tardar unos minutos.
2.  **Después, desvincule la cuenta de almacenamiento anterior para que los agentes dejen de escribir en la cuenta quitada.** El proceso de ingesta sigue leyendo los datos de esta cuenta hasta que se realice la ingesta de todos. No elimine la cuenta de almacenamiento hasta que vea que se ingirieron todos los registros.

### <a name="maintaining-storage-accounts"></a>Mantenimiento de cuentas de almacenamiento
##### <a name="manage-log-retention"></a>Administración de la retención de registros
Cuando use su propia cuenta de almacenamiento, la retención depende de usted. Es decir, Log Analytics no elimina los registros almacenados en el almacenamiento privado. En su lugar, debe configurar una directiva para controlar la carga según sus preferencias.

##### <a name="consider-load"></a>Consideración de la carga
Las cuentas de almacenamiento pueden controlar cierta carga de solicitudes de lectura y escritura antes de empezar a limitar las solicitudes (consulte [Objetivos de escalabilidad y rendimiento de Blob Storage](https://docs.microsoft.com/azure/storage/common/scalability-targets-standard-account) para más detalles). La limitación afecta al tiempo que se tarda en ingerir registros. Si la cuenta de almacenamiento está sobrecargada, registre una cuenta de almacenamiento adicional para distribuir la carga entre ellas. Para supervisar la capacidad y el rendimiento de la cuenta de almacenamiento, revise su [Información en Azure Portal]( https://docs.microsoft.com/azure/azure-monitor/insights/storage-insights-overview).

### <a name="related-charges"></a>Cargos relacionados
Las cuentas de almacenamiento se cobran según el volumen de datos almacenados, el tipo de almacenamiento y el tipo de redundancia. Para detalles, consulte [Precios de los blobs en bloques](https://azure.microsoft.com/pricing/details/storage/blobs) y [Precios de Azure Table Storage](https://azure.microsoft.com/pricing/details/storage/tables).


## <a name="next-steps"></a>Pasos siguientes

- Información sobre el [uso de Azure Private Link para conectar redes a Azure Monitor de forma segura](private-link-security.md)
- Información sobre las [claves administradas por el cliente de Azure Monitor](customer-managed-keys.md)
