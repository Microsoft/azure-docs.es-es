---
title: Mejora de la administración de SOC con las métricas de incidentes de Azure Sentinel | Microsoft Docs
description: Use la información de la pantalla de métricas de incidentes de Azure Sentinel y el libro como ayuda para administrar el Centro de operaciones de seguridad (SOC).
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/29/2020
ms.author: yelevin
ms.openlocfilehash: f14b0050aefc598d26dec7a7781a3378ccaa7570
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293844"
---
# <a name="manage-your-soc-better-with-incident-metrics"></a>Mejora en la administración de SOC con métricas de incidentes

> [!IMPORTANT]
> Las características de las métricas de incidentes se encuentran actualmente en versión preliminar pública.
> Estas características se ofrecen sin Acuerdo de Nivel de Servicio y no es recomendable usarlas para cargas de trabajo de producción.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Los administradores del Centro de operaciones de seguridad (SOC) deben tener medidas y métricas de eficiencia generales a su alcance para calibrar el rendimiento de su equipo. Querrá ver las operaciones de los incidentes a lo largo del tiempo clasificadas por muchos criterios diferentes, como la gravedad, las tácticas de MITRE, el tiempo medio para la evaluación de prioridades, el tiempo medio de resolución, etc. Ahora, Azure Sentinel pone todos estos datos a su disposición con las nuevas tablas y esquemas **SecurityIncident** de Log Analytics y el libro **Security operations efficiency** (Eficiencia de las operaciones de seguridad) que se incluye. Podrá visualizar el rendimiento del equipo a lo largo del tiempo y usar esta información para mejorar su eficacia. También puede escribir y usar sus propias consultas de KQL en la tabla de incidentes para crear libros personalizados que se ajusten a sus necesidades específicas de auditoría y a los indicadores clave de rendimiento.

## <a name="use-the-security-incidents-table"></a>Uso de la tabla de incidentes de seguridad

La tabla **SecurityIncident** está integrada en Azure Sentinel. La encontrará con las demás tablas de la colección **SecurityInsights** en **Registros**. Puede consultarla como cualquier otra tabla de Log Analytics.

:::image type="content" source="./media/manage-soc-with-incident-metrics/security-incident-table.png" alt-text="Tabla de incidentes de seguridad":::

Cada vez que se crea o se actualiza un incidente, se agregará una nueva entrada del registro a la tabla. Esto le permite realizar un seguimiento de los cambios realizados en incidentes y permite usar métricas de SOC aún más eficaces, pero debe ser consciente de ello al construir consultas para esta tabla, ya que puede que tenga que quitar entradas duplicadas de un incidente (en función de la consulta exacta que esté ejecutando). 

Por ejemplo, si deseara devolver una lista de todos los incidentes ordenados por número de incidente, pero solo deseara devolver el registro más reciente por incidente, podría hacerlo mediante el [operador summarize](https://docs.microsoft.com/azure/data-explorer/kusto/query/summarizeoperator) de KQL con la `arg_max()` [función aggregation](https://docs.microsoft.com/azure/data-explorer/kusto/query/arg-max-aggfunction):

`SecurityIncident` <br>
`| summarize arg_max(LastModifiedTime, *) by IncidentNumber`

## <a name="security-operations-efficiency-workbook"></a>Libro Security operations efficiency (Eficiencia de las operaciones de seguridad)

Para complementar la tabla **SecurityIncidents**, se ha proporcionado una plantilla del libro **security operations efficiency** (eficiencia de las operaciones de seguridad) que se puede usar para supervisar las operaciones de SOC. El libro contiene las siguientes métricas: 
- Incidentes creados a lo largo del tiempo 
- Incidentes creados por clasificación de cierre, gravedad, propietario y estado 
- Tiempo medio para la evaluación de prioridades 
- Tiempo medio para la clausura 
- Incidentes creados por gravedad, propietario, estado, producto y tácticas a lo largo del tiempo 
- Percentiles de tiempo para la evaluación de prioridades 
- Percentiles de tiempo para la clausura 
- Tiempo medio para la evaluación de prioridades por propietario 
- Actividades recientes 
- Clasificaciones de cierre recientes  

Para encontrar esta plantilla de libro nueva, elija **Libros** en el menú de navegación de Azure Sentinel y seleccione la pestaña **Plantillas**. Elija **Security operations efficiency** (Eficiencia de operaciones de seguridad) en la galería y haga clic en uno de estos botones: **Ver libro guardado** y **Ver plantilla**.

:::image type="content" source="./media/manage-soc-with-incident-metrics/security-incidents-workbooks-gallery.png" alt-text="Galería de Security incidents workbook"::: (Libro de incidentes de seguridad)

:::image type="content" source="./media/manage-soc-with-incident-metrics/security-operations-workbook-1.png" alt-text="Security incidents workbook (Libro de incidentes de seguridad) completado":::

Puede usar la plantilla para crear sus propios libros personalizados adaptados a sus necesidades específicas.

## <a name="securityincidents-schema"></a>Esquema de SecurityIncidents

[!INCLUDE [SecurityIncidents schema](../../includes/sentinel-schema-security-incident.md)]

## <a name="next-steps"></a>Pasos siguientes

- Para empezar a trabajar con Azure Sentinel, necesita una suscripción a Microsoft Azure. Si no tiene una suscripción, puede registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/free/).
- Aprenda a [incorporar los datos en Azure Sentinel](quickstart-onboard.md), [obtenga visibilidad sobre ellos y aprenda a defenderse de posibles amenazas](quickstart-get-visibility.md).
