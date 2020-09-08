---
title: Visualización del estado de los trabajos de Azure Import/Export | Microsoft Docs
description: Aprenda a visualizar el estado de los trabajos de Azure Import/Export y las unidades utilizadas. Conozca los factores que afectan al tiempo que tardan en procesarse los trabajos.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 05/17/2018
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: 8110b98c055a211203fb937990e860fc8dea74f4
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88520469"
---
# <a name="view-the-status-of-azure-importexport-jobs"></a>Visualización del estado de los trabajos de Azure Import/Export

Este artículo proporciona información sobre cómo visualizar el estado de la unidad y los trabajos para los trabajos de Azure Import/Export. El servicio Azure Import/Export se usa para transferir de forma segura grandes cantidades de datos a los blobs de Azure y a Azure Files. El servicio también se usa para exportar datos desde Azure Blob Storage.  

## <a name="view-job-and-drive-status"></a>Visualización del estado de trabajos y unidades de disco
Puede realizar el seguimiento del estado de sus trabajos de importación y exportación desde Azure Portal seleccionando la pestaña **Import/Export**. Aparece una lista de los trabajos en la página.

![Visualización del estado del trabajo](./media/storage-import-export-service/jobstate.png)

## <a name="view-job-status"></a>Ver estado del trabajo

Verá uno de los siguientes estados de trabajo en función de la fase del proceso en que se encuentre la unidad.

| Estado del trabajo | Descripción |
|:--- |:--- |
| Creating | Cuando se crea un trabajo, su estado se establece en **Creating**. Mientras el trabajo se encuentra en el estado **Creating**, el servicio Import/Export asume que las unidades no se han enviado al centro de datos. Un trabajo puede permanecer en este estado hasta un máximo de dos semanas, pasado este tiempo el servicio lo elimina automáticamente. |
| Envío | Después de enviar el paquete, debe actualizar la información de seguimiento en Azure Portal.  Esto cambia el estado del trabajo a **Shipping**. El trabajo permanecerá en el estado **Shipping** durante un máximo de dos semanas. 
| Received | Una vez recibidas todas las unidades en el centro de datos, el estado del trabajo se establecerá en **Received**. |
| Transferring | Cuando se haya iniciado el procesamiento de al menos una unidad, el estado del trabajo se establecerá en **Transferring**. Para más información, vaya a [Estados de la unidad](#view-drive-status). |
| Packaging | Cuando todas las unidades hayan terminado el procesamiento, el trabajo pasará al estado **Packaging** hasta que se le devuelvan las unidades. |
| Completed | Cuando todas las unidades se le hayan devuelto, si el trabajo se ha completado sin errores, se establecerá en el estado **Completed**. El trabajo se elimina automáticamente después de 90 días en el estado **Completed**. |
| Closed | Después de que todas las unidades se le envíen de vuelta, si se produjeron errores durante el procesamiento del trabajo, el trabajo se establece en **Closed**. El trabajo se elimina automáticamente después de 90 días en el estado **Closed**. |

## <a name="view-drive-status"></a>Visualización de estado de la unidad

En la tabla siguiente se describe el ciclo de vida de una unidad individual durante su transición a través de un trabajo de importación o exportación. El estado actual de cada unidad de un trabajo se puede ver ahora en Azure Portal.

En la tabla siguiente se describe cada estado por el que puede pasar una unidad de un trabajo.

| Estado de la unidad | Descripción |
|:--- |:--- |
| Specified | Para un trabajo de importación, cuando se crea el trabajo desde Azure Portal, el estado inicial de una unidad es **Specified**. Para un trabajo de exportación, ya que cuando se crea el trabajo no se especifica ninguna unidad, el estado inicial de la unidad es **Received**. |
| Received | La unidad cambia al estado **Received** cuando el servicio Import/Export ha procesado las unidades recibidas de la empresa remitente para un trabajo de importación. Para un trabajo de exportación, el estado inicial de la unidad es **Received**. |
| NeverReceived | La unidad pasa al estado **NeverReceived** si llega un paquete para un trabajo pero el paquete no contiene la unidad. Una unidad también puede cambiar a este estado si el centro de datos aún no ha recibido el paquete y el servicio recibió la información de envío hace al menos dos semanas. |
| Transferring | Una unidad pasará al estado **Transferring** cuando el servicio empiece a transferir datos de la unidad a Azure Storage. |
| Completed | Una unidad pasa al estado **Completed** cuando el servicio haya transferido correctamente todos los datos sin errores.
| CompletedMoreInfo | Una unidad de disco pasa al estado **CompletedMoreInfo** cuando el servicio experimente problemas mientras copia datos en la unidad o desde ella. La información puede incluir errores, advertencias o mensajes informativos por sobrescribir blobs.
| ShippedBack | La unidad de disco pasa al estado **ShippedBack** cuando se haya enviado desde el centro de datos a la dirección de devolución. |

Esta imagen de Azure Portal muestra el estado de la unidad de un trabajo de ejemplo:

![Visualización de estado de la unidad](./media/storage-import-export-service/drivestate.png)

En la tabla siguiente se describen los estados de error de la unidad y las medidas tomadas para cada estado.

| Estado de la unidad | Evento | Resolución y paso siguiente |
|:--- |:--- |:--- |
| NeverReceived | Una unidad que está marcada como **NeverReceived** (porque no se recibió como parte del envío del trabajo) llega en otro envío. | El equipo de operaciones mueve a la unidad a **Received**. |
| N/D | Una unidad que no forma parte de ningún trabajo llega al centro de datos como parte de otro trabajo. | La unidad está marcada como unidad extra. Se le devuelve cuando se complete el trabajo asociado al paquete original. |

## <a name="time-to-process-job"></a>Tiempo para procesar el trabajo
La cantidad de tiempo que tarda en procesar un trabajo de importación o exportación varía en función de una serie de factores como:

-  Tiempo de envío
-  Carga en el centro de datos
-  Tipo de trabajo y tamaño de los datos copiados
-  Número de discos en un trabajo. 

El servicio Import/Export no tiene un Acuerdo de Nivel de Servicio, pero se esfuerza por completar la copia en un período de tiempo de entre siete y diez días después de recibirse los discos. Además de para el estado que se publica en Azure Portal, puede usar las API REST para realizar un seguimiento del progreso del trabajo. Use el parámetro de porcentaje completo de la llamada API de la operación de [enumeración de trabajos](/previous-versions/azure/dn529083(v=azure.100)) para ver el porcentaje del progreso de copia.


## <a name="next-steps"></a>Pasos siguientes

* [Transferencia de datos con la utilidad de línea de comandos AzCopy](storage-use-azcopy.md)
* [Ejemplo de API de REST de Azure Import/Export](https://github.com/Azure-Samples/storage-dotnet-import-export-job-management/)
