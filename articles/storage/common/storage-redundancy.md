---
title: Redundancia de datos en Azure Storage | Microsoft Docs
description: Los datos de su cuenta de Microsoft Azure Storage se replican para garantizar la durabilidad y la alta disponibilidad. Entre las opciones de redundancia se incluyen el almacenamiento con redundancia local (LRS), el almacenamiento con redundancia de zona (ZRS), el almacenamiento con redundancia geográfica (GRS) y el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS).
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 01/18/2019
ms.author: tamram
ms.reviewer: artek
ms.subservice: common
ms.openlocfilehash: 7006e19616be51d79dc3e1319064d19024400bcc
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65789995"
---
# <a name="azure-storage-redundancy"></a>Redundancia de Azure Storage

Los datos de su cuenta de almacenamiento de Microsoft Azure se replican siempre para garantizar la durabilidad y la alta disponibilidad. Azure Storage copia los datos para protegerlos de eventos previstos e imprevistos, como errores transitorios del hardware, interrupciones del suministro eléctrico o de la red y desastres naturales masivos. Puede optar por replicar los datos en el mismo centro de datos, en centros de datos zonales que estén en la misma región o en regiones geográficamente separadas.

La replicación garantiza que la cuenta de almacenamiento cumpla el [contrato de nivel de servicio (SLA) para Storage](https://azure.microsoft.com/support/legal/sla/storage/), incluso en caso de errores. Consulte en el SLA información acerca de las garantías de durabilidad y disponibilidad de Azure Storage.

## <a name="choosing-a-redundancy-option"></a>Selección de una opción de redundancia

Cuando crea una cuenta de almacenamiento, puede seleccionar una de las siguientes opciones de redundancia:

* [Almacenamiento con redundancia local (LRS)](storage-redundancy-lrs.md)
* [Almacenamiento con redundancia de zona (ZRS)](storage-redundancy-zrs.md)
* [Almacenamiento con redundancia geográfica (GRS)](storage-redundancy-grs.md)
* [Almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS).](storage-redundancy-grs.md#read-access-geo-redundant-storage)

En la tabla siguiente se ofrece una rápida información general del ámbito de durabilidad y disponibilidad que cada estrategia de replicación le proporcionará para un tipo determinado de evento (o evento de impacto similar).

| Escenario                                                                                                 | LRS                             | ZRS                              | GRS                                  | RA-GRS                               |
| :------------------------------------------------------------------------------------------------------- | :------------------------------ | :------------------------------- | :----------------------------------- | :----------------------------------- |
| Falta de disponibilidad del nodo en un centro de datos                                                                 | Sí                             | Sí                              | Sí                                  | Sí                                  |
| Un centro de datos completo (de zona o no de zona) deja de estar disponible                                           | No                              | Sí                              | Sí                                  | Sí                                  |
| Una interrupción en toda la región                                                                                     | No                              | No                               | Sí                                  | Sí                                  |
| Acceso de lectura a los datos (en una región remota y con replicación geográfica) en caso de no disponibilidad en toda la región. | No                              | No                               | No                                   | Sí                                  |
| Diseñado para proporcionar una \_\_ durabilidad de objetos a lo largo de un año determinado.                                          | Como mínimo 99.999999999 % (once nueves) | Como mínimo 99.9999999999 % (doce nueves) | Como mínimo 99.99999999999999 % (dieciséis nueves) | Como mínimo 99.99999999999999 % (dieciséis nueves) |
| Tipos de cuenta de almacenamiento admitidos                                                                   | GPv2, GPv1, Blob                | GPv2                             | GPv2, GPv1, Blob                     | GPv2, GPv1, Blob                     |
| SLA de disponibilidad para las solicitudes de lectura | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) | Al menos un 99,99 % (99,9 % para el nivel de acceso esporádico) |
| SLA de disponibilidad para las solicitudes de escritura | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) |

Para obtener información sobre los precios de las diferentes opciones de redundancia, vea [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/). 

Para obtener información acerca de las garantías de durabilidad y disponibilidad de Azure Storage, vea [Acuerdo de Nivel de Servicio de Azure](https://azure.microsoft.com/support/legal/sla/storage/).

> [!NOTE]
> Premium Storage solo admite almacenamiento con redundancia local (LRS).

## <a name="changing-replication-strategy"></a>Cambio de estrategia de replicación
Puede cambiar la estrategia de replicación de la cuenta de almacenamiento mediante el [portal Azure](https://portal.azure.com/), [Azure Powershell](storage-powershell-guide-full.md), [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest), o uno de los [cliente de Azure bibliotecas](https://docs.microsoft.com/azure/index#pivot=sdkstools). Si cambia el tipo de replicación de la cuenta de almacenamiento, no se producirá ningún tiempo de inactividad.

   > [!NOTE]
   > Actualmente, no se puede usar el Portal o la API para convertir la cuenta a ZRS. Si desea convertir la replicación de la cuenta en ZRS, consulte [Almacenamiento con redundancia de zona (ZRS)](storage-redundancy-zrs.md) para más información.
    
### <a name="are-there-any-costs-to-changing-my-accounts-replication-strategy"></a>¿Hay algún costo al cambiar la estrategia de replicación de mi cuenta?
Depende de la ruta de acceso de la conversión. Si se solicita la oferta de redundancia de más económica a la más cara, tenemos LRS, ZRS, GRS, y RA-GRS. Por ejemplo, si se pasa *de* LRS a cualquier otra opción, se incurrirá en cargos adicionales porque se pasará a un nivel de redundancia más sofisticado. Si se pasa *a* GRS o RA-GRS se incurrirá en un cargo por ancho de banda de salida porque los datos (en la región primaria) se están replicando en la región secundaria remota. Se trata de un cargo único en la instalación inicial. Después de copiar los datos, ya no hay ningún cargo adicional de conversión. Solo se le cobrará por replicar cualquier dato nuevo o actualizado en los datos existentes. Para obtener más información sobre los cargos de ancho de banda, consulte la [página de precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/blobs/).

Si convierte la cuenta de almacenamiento de GRS a LRS, no hay ningún costo adicional, pero los datos replicados se eliminan de la ubicación secundaria.

Si convierte la cuenta de almacenamiento de RA-GRS a GRS o LRS, esa cuenta se factura como RA-GRS durante 30 días adicionales más allá de la fecha en que se ha convertido.

## <a name="see-also"></a>Vea también

- [Almacenamiento con redundancia local (LRS): redundancia de datos de bajo costo para Azure Storage](storage-redundancy-lrs.md)
- [Almacenamiento con redundancia de zona (ZRS): aplicaciones de Azure Storage de alta disponibilidad](storage-redundancy-zrs.md)
- [Almacenamiento con redundancia geográfica (GRS): replicación entre regiones para Azure Storage](storage-redundancy-grs.md)
- [Objetivos de escalabilidad y rendimiento de Azure Storage](storage-scalability-targets.md)
- [Diseño de aplicaciones de alta disponibilidad mediante RA-GRS](../storage-designing-ha-apps-with-ragrs.md)
- [Microsoft Azure Storage redundancia acceso de lectura y las opciones de almacenamiento con redundancia geográfica](https://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/11/introducing-read-access-geo-replicated-storage-ra-grs-for-windows-azure-storage.aspx)
- [SOSP Paper - Azure Storage: A Highly Available Cloud Storage Service with Strong Consistency](https://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx) (Documentación de SOSP: Azure Storage: Un servicio de almacenamiento en nube altamente disponible de gran coherencia).
