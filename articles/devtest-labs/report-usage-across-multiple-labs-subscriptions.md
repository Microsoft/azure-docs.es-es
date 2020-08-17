---
title: Uso de Azure DevTest Labs en varios laboratorios y suscripciones
description: Aprenda a notificar el uso de Azure DevTest Labs en varios laboratorios y suscripciones.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 1d2663113e929145308f5a5712b968f3551668c2
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287282"
---
# <a name="report-azure-devtest-labs-usage-across-multiple-labs-and-subscriptions"></a>Notificación del uso de Azure DevTest Labs en varios laboratorios y suscripciones

La mayoría de las grandes organizaciones quiere realizar un seguimiento del uso de los recursos para hacerlos más eficaces, visualizando tendencias y valores atípicos en el uso. En función del uso de los recursos, los propietarios o administradores de laboratorios pueden personalizarlos para [mejorar el uso de los recursos y los costos](../cost-management-billing/manage/getting-started.md). En Azure DevTest Labs, puede descargar el uso de recursos por laboratorio, lo cual permite un análisis más profundo de los patrones de uso. Estos patrones de uso pueden ayudar a identificar cambios para mejorar la eficacia. A la mayoría de las empresas les gustaría conocer tanto el uso de laboratorios individuales como el uso total en [varios laboratorios y suscripciones](/azure/architecture/cloud-adoption/decision-guides/subscriptions/). 

En este artículo se describe cómo administrar la información sobre el uso de los recursos en varios laboratorios y suscripciones.

![Uso de informes](./media/report-usage-across-multiple-labs-subscriptions/report-usage.png)

## <a name="individual-lab-usage"></a>Uso de un laboratorio individual

En esta sección se describe cómo exportar el uso de recursos para un único laboratorio.

Antes de poder exportar el uso de recursos de DevTest Labs, tiene que configurar una cuenta de Azure Storage para permitir que se almacenen los distintos archivos que contienen los datos de uso. Hay dos formas habituales de ejecutar la exportación de datos:

* [API REST de DevTest Labs](/rest/api/dtl/labs/exportresourceusage) 
* El módulo [Invoke-AzResourceAction](/powershell/module/az.resources/invoke-azresourceaction?view=azps-2.5.0&viewFallbackFrom=azps-2.3.2) de Az.Resource de PowerShell con la acción `exportResourceUsage`, el identificador del recurso de laboratorio y los parámetros necesarios. 

    El artículo [Exportación o eliminación de datos personales](personal-data-delete-export.md) incluye un script de PowerShell de ejemplo con información detallada sobre los datos que se exportan. 

    > [!NOTE]
    > El parámetro date no incluye una marca de tiempo por lo que los datos incluyen toda la información a partir de medianoche según la zona horaria en la que se encuentre el laboratorio.

Una vez completada la exportación, habrá varios archivos CSV en el almacenamiento de blobs con diferente información de recursos.
  
En estos momentos hay dos archivos CSV:

* *virtualmachines.csv*: contiene información sobre las máquinas virtuales del laboratorio.
* *disks.csv*: contiene información sobre los diferentes discos del laboratorio 

Estos archivos se almacenan en el contenedor de blobs *labresourceusage* con el nombre del laboratorio, el identificador único de este, la fecha de ejecución y la fecha completa o la fecha de inicio basada en la solicitud de exportación. Un ejemplo de estructura de blobs sería:

* `labresourceusage/labname/1111aaaa-bbbb-cccc-dddd-2222eeee/<End>DD26-MM6-2019YYYY/full/virtualmachines.csv`
* `labresourceusage/labname/1111aaaa-bbbb-cccc-dddd-2222eeee/<End>DD-MM-YYYY/26-6-2019/20-6-2019<Start>DD-MM-YYYY/virtualmachines.csv`

## <a name="exporting-usage-for-all-labs"></a>Exportación del uso de todos los laboratorios

Para exportar la información de uso de varios laboratorios, considere la posibilidad de usar 

* [Azure Functions](../azure-functions/index.yml), disponible en muchos lenguajes, incluido PowerShell, o 
* [Runbook de Azure Automation](../automation/index.yml), use PowerShell, Python o un diseñador gráfico personalizado para escribir el código de exportación.

Con estas tecnologías, puede ejecutar las exportaciones de laboratorio individuales de todos los laboratorios en una fecha y hora específicas. 

La función de Azure debe insertar los datos en el almacenamiento a largo plazo. Cuando se exportan datos de varios laboratorios, la exportación puede tardar algún tiempo. Para ayudar a mejorar el rendimiento y reducir la posibilidad de duplicar la información, se recomienda ejecutar cada laboratorio en paralelo. Para lograr el paralelismo, ejecute Azure Functions de forma asincrónica. Aproveche también el desencadenador de temporizador que ofrece Azure Functions.

## <a name="using-a-long-term-storage"></a>Uso de un almacenamiento a largo plazo

Un almacenamiento a largo plazo consolida la información de exportación de los diferentes laboratorios en un único origen de datos. Otra ventaja de usar el almacenamiento a largo plazo es poder quitar los archivos de la cuenta de almacenamiento para reducir la duplicación y el costo. 

El almacenamiento a largo plazo se puede usar para realizar cualquier manipulación en el texto, por ejemplo: 

* agregar nombres descriptivos
* crear agrupaciones complejas
* agregar los datos.

Algunas soluciones de almacenamiento habituales son: [SQL Server](https://azure.microsoft.com/services/sql-database/), [Azure Data Lake](https://azure.microsoft.com/services/storage/data-lake-storage/) y [Cosmos DB](https://azure.microsoft.com/services/cosmos-db/). La elección de la solución de almacenamiento a largo plazo que elija dependerá de las preferencias. Puede considerar la posibilidad de elegir la herramienta en función de lo que ofrece en cuanto a la disponibilidad de interacción al visualizar los datos.

## <a name="visualizing-data-and-gathering-insights"></a>Visualización de datos y recopilación de información

Use una herramienta de visualización de datos de su elección para conectarse al almacenamiento a largo plazo para mostrar los datos de uso y recopilar información para comprobar el uso eficiente. Por ejemplo, [Power BI](/power-bi/power-bi-overview) se puede usar para organizar y mostrar los datos de uso. 

Puede usar [Azure Data Factory](https://azure.microsoft.com/services/data-factory/) para crear, vincular y administrar los recursos dentro de una única interfaz de ubicación. Si se necesita un mayor control, se puede crear el recurso individual dentro de un único grupo de recursos y administrarse independientemente del servicio Data Factory.  

## <a name="next-steps"></a>Pasos siguientes

Una vez configurado el sistema y trasladado los datos al almacenamiento a largo plazo, el siguiente paso es plantearse las preguntas que los datos deben responder. Por ejemplo: 

-   ¿Cuál es el uso del tamaño de la máquina virtual?

    ¿Eligen los usuarios tamaños de máquina virtual de alto rendimiento (más caros)?
-   ¿Qué imágenes de Marketplace se están usando?

    ¿Son las imágenes personalizadas la base más habitual de las máquinas virtuales? ¿Se debe compilar un almacén de imágenes común como [Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md) o como [imagen de fábrica](image-factory-create.md)?
-   ¿Qué imágenes personalizadas se usan y cuáles no?
