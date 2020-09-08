---
title: Configuración de la recopilación de datos para el agente de Azure Monitor (versión preliminar)
description: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/19/2020
ms.openlocfilehash: 97ba976c841e192646ebb484fe4d004556e34c54
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88640844"
---
# <a name="configure-data-collection-for-the-azure-monitor-agent-preview"></a>Configuración de la recopilación de datos para el agente de Azure Monitor (versión preliminar)
Las reglas de recopilación de datos (DCR) definen los datos que entran en Azure Monitor y especifican dónde se deben enviar. En este artículo se describe cómo crear una regla de recopilación de datos para recopilar datos de máquinas virtuales mediante el agente de Azure Monitor.

Para obtener una descripción completa de las reglas de recopilación de datos, consulte [Reglas de recopilación de datos en Azure Monitor (versión preliminar)](data-collection-rule-overview.md).

> [!NOTE]
> En este artículo se describe cómo configurar los datos de las máquinas virtuales con el agente de Azure Monitor que se encuentra actualmente en versión preliminar. Consulte [Información general sobre los agentes de Azure Monitor](agents-overview.md) para obtener una descripción de los agentes que están disponibles con carácter general y cómo usarlos para recopilar datos.


## <a name="dcr-associations"></a>Asociaciones de reglas de recopilación de datos
Para aplicar una reglas de recopilación de datos a una máquina virtual, cree una asociación para la máquina virtual. Una máquina virtual puede tener una asociación a varias reglas de recopilación de datos y una de estas reglas puede tener varias máquinas virtuales asociadas. Esto le permite definir un conjunto de reglas de recopilación de datos, cada una de las cuales coincide con un requisito determinado y aplicarlas solo a las máquinas virtuales en las que se aplican. 

Por ejemplo, imagine un entorno con un conjunto de máquinas virtuales que ejecutan una aplicación de línea de negocio y otras que se ejecutan SQL Server. Es posible que tenga una regla de recopilación de datos predeterminada que se aplique a todas las máquinas virtuales y a reglas de recopilación de datos independientes que recopilen datos específicamente para la aplicación de línea de negocio y SQL Server. Las asociaciones de las máquinas virtuales con las reglas de recopilación de datos tendrían un aspecto similar al siguiente diagrama.

![Asociaciones](media/data-collection-rule-azure-monitor-agent/associations.png)

## <a name="create-using-the-azure-portal"></a>Creación mediante Azure Portal
Puede usar Azure Portal para crear una regla de recopilación de datos y asociar las máquinas virtuales de su suscripción a esa regla. El agente de Azure Monitor se instalará automáticamente y se creará una identidad administrada para las máquinas virtuales que no lo tengan instalado.

En el menú **Azure Monitor** de Azure Portal, seleccione las **reglas de recopilación de datos** en la sección **Configuración**. Haga clic en **Agregar** para agregar una nueva asignación y regla de recopilación de datos.

[![Reglas de recopilación de datos](media/azure-monitor-agent/data-collection-rules.png)](media/azure-monitor-agent/data-collection-rules.png#lightbox)

Haga clic en **Agregar** para crear una nueva regla y un conjunto de asociaciones. Dé un **nombre de regla** y especifique una **suscripción** y **un grupo de recursos**. Esto especifica dónde se creará la regla de recopilación de datos. Las máquinas virtuales y sus asociaciones pueden estar en cualquier suscripción o grupo de recursos del inquilino.

[![Nombre de la regla de recopilación de datos](media/azure-monitor-agent/data-collection-rule-basics.png)](media/azure-monitor-agent/data-collection-rule-basics.png#lightbox)

En la pestaña **Máquinas virtuales**, agregue las máquinas virtuales que deben tener aplicada la regla de recopilación de datos. El agente de Azure Monitor se instalará en las máquinas virtuales que no lo tengan instalado.

[![Máquinas virtuales de reglas de recopilación de datos](media/azure-monitor-agent/data-collection-rule-virtual-machines.png)](media/azure-monitor-agent/data-collection-rule-virtual-machines.png#lightbox)

En la pestaña **Collect and deliver** (Recopilar y entregar), haga clic en **Add data source** (Agregar origen de datos) para agregar un origen de datos y un conjunto de destino. Seleccione un **tipo de origen de datos** y se mostrarán los detalles correspondientes. Para los contadores de rendimiento, puede seleccionar entre un conjunto predefinido de objetos y su frecuencia de muestreo. En el caso de los eventos, puede seleccionar entre un conjunto de registros o instalaciones y el nivel de gravedad. 

[![Origen de datos básico](media/azure-monitor-agent/data-collection-rule-data-source-basic.png)](media/azure-monitor-agent/data-collection-rule-data-source-basic.png#lightbox)


Para especificar otros registros y contadores de rendimiento, seleccione **Custom** (Personalizado). Después, puede especificar un [XPath](https://www.w3schools.com/xml/xpath_syntax.asp) para los valores específicos que se van a recopilar. Consulte la [regla de recopilación de datos de ejemplo](data-collection-rule-overview.md#sample-data-collection-rule) para ver ejemplos.

[![Origen de datos personalizado](media/azure-monitor-agent/data-collection-rule-data-source-custom.png)](media/azure-monitor-agent/data-collection-rule-data-source-custom.png#lightbox)

En la pestaña **Destination** (Destino), agregue uno o varios destinos para el origen de datos. Los orígenes de datos de eventos de Windows y Syslog solo pueden enviar a registros de Azure Monitor. Los contadores de rendimiento pueden enviar a métricas de Azure Monitor y registros de Azure Monitor.

[![Destination](media/azure-monitor-agent/data-collection-rule-destination.png)](media/azure-monitor-agent/data-collection-rule-destination.png#lightbox)

Haga clic en **Agregar origen de datos** (Agregar origen de datos) y, después, en **Revisar y crear** para revisar los detalles de la regla de recopilación de datos y la asociación con el conjunto de máquinas virtuales. Haga clic en **Crear** para crearlo.

> [!NOTE]
> Cuando se hayan creado las asociaciones y la regla de recopilación de datos, los datos pueden tardar hasta cinco minutos en enviarse a los destinos.

## <a name="createusingrestapi"></a>Creación mediante la API REST
Siga los pasos que se indican a continuación para crear una DCR y asociaciones mediante la API REST. 
1. Cree manualmente el archivo DCR con el formato JSON que se muestra en  [DCR de ejemplo](data-collection-rule-overview.md#sample-data-collection-rule).
2. Cree la regla con la  [API REST](https://docs.microsoft.com/rest/api/monitor/datacollectionrules/create#examples).
3. Cree una asociación para cada máquina virtual a la regla de recopilación de datos mediante la  [API REST](https://docs.microsoft.com/rest/api/monitor/datacollectionruleassociations/create#examples).

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [agente de Azure Monitor](azure-monitor-agent-overview.md).
- Más información sobre las [reglas de recopilación de datos](data-collection-rule-overview.md).
