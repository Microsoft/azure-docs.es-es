---
title: Asignación de costos de Azure
description: En este artículo se explica cómo crear reglas de asignación de costos para distribuir los costos de las suscripciones, grupos de recursos o etiquetas a otros usuarios.
author: bandersmsft
ms.author: banders
ms.date: 08/11/2020
ms.topic: how-to
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: benshy
ms.openlocfilehash: 025b9b7e503f38a111bd158f17b7fbeec5b23579
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684990"
---
# <a name="create-and-manage-azure-cost-allocation-rules-preview"></a>Creación y administración de reglas de asignación de costos de Azure (versión preliminar)

A menudo, las grandes empresas tienen servicios o recursos de Azure que se administran centralmente, pero que los usan diferentes departamentos internos o unidades de negocio. Lo habitual es que el equipo que los administra de forma centralizada quiera repercutir el costo de los servicios compartidos a los departamentos internos o a las unidades de negocio de la organización que los usan de forma activa. Este artículo le ayudará no solo a conocer el concepto de asignación de costos en Azure Cost Management, sino también a usar dicha asignación.

Con la asignación de costos, es posible reasignar o distribuir los costos de los servicios compartidos de unas suscripciones, grupos de recursos o etiquetas a otras suscripciones, grupos de recursos o etiquetas de la organización. La asignación de costos traslada los costos de los servicios compartidos a otra suscripción, otros grupos de recursos u otras etiquetas, que son propiedad de los departamentos internos o las unidades de negocio que los consumen. En otras palabras, la asignación de costos ayuda a administrar y mostrar la _responsabilidad de los costos_ de un lugar en otro.

La asignación de costos no afecta a su facturación. Las responsabilidades de facturación no cambian. El objetivo principal de la asignación de costos es ayudarle a repercutir los costos a otros usuarios. Todos los procesos de contracargo se producen en su organización fuera de Azure. La asignación de costos le ayuda a repercutir los costos, ya que los muestra a medida que se reasignan o distribuyen.

Los costos asignados se muestran en el análisis de costos. Se muestran como elementos adicionales asociados a las suscripciones de destino, los grupos de recursos o las etiquetas que se especifiquen al crear una regla de asignación de costos.

> [!NOTE]
> La característica de asignación de costos de Azure Cost Management está actualmente en versión preliminar pública. Es posible que algunas características de Cost Management no sean compatibles o que sus funcionalidades estén limitadas.

## <a name="prerequisites"></a>Requisitos previos

- En la actualidad, la asignación de costos solo pueden usarla los clientes con un [Contrato de cliente de Microsoft](https://azure.microsoft.com/pricing/purchase-options/microsoft-customer-agreement/) o un [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/).
- Para crear o administrar una regla de asignación de costos, se debe usar una cuenta de administrador de empresa para los [Contratos Enterprise](../manage/understand-ea-roles.md). O bien se debe ser propietario de una [cuenta de facturación](../manage/understand-mca-roles.md) para los Contratos de cliente de Microsoft.

## <a name="create-a-cost-allocation-rule"></a>Creación de una regla de asignación de costos

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com/).
2. Vaya a **Administración de costos + facturación** > **Administración de costos**.
3. En **Configuración** > **Configuración**, seleccione **Asignación de costos (versión preliminar)** .
4. Asegúrese de que se selecciona la cuenta de facturación o inscripción de EA correctas.
5. Seleccione **+Agregar**.
6. Escriba un texto descriptivo para el nombre de la regla de asignación de costos.

:::image type="content" source="./media/allocate-costs/rule-name.png" alt-text="Ejemplo que muestra la creación de un nombre de regla" lightbox="./media/allocate-costs/rule-name.png" :::

La fecha de inicio de la evaluación de la regla se usa para generar porcentajes de asignación de costos que se rellenan previamente.

1. Seleccione **Agregar orígenes** y, después, seleccione las suscripciones, los grupos de recursos o las etiquetas en la que se eligen los costos que se van a distribuir.
2. Seleccione **Agregar destinos** y, después, seleccione las suscripciones, los grupos de recursos o las etiquetas que van a recibir los costos asignados.
3. Si necesita crear más reglas de asignación de costos, repita este proceso.

## <a name="configure-the-allocation-percentage"></a>Configuración del porcentaje de asignación

Configure el porcentaje de asignación para definir la forma en que los costos se dividen proporcionalmente entre los destinos especificados. Puede definir de forma manual los porcentajes de números enteros para crear una regla de asignación. O bien, puede dividir los costos de manera proporcional en función del uso actual del proceso, almacenamiento o red entre los destinos especificados.

Al distribuir los costos por costo de proceso, costo del almacenamiento o costo de red, el porcentaje proporcional se deduce mediante la evaluación de los costos del destino seleccionado. Los costos se asocian con el tipo de recurso del mes de facturación actual.

Al distribuir los costos de forma proporcional al costo total, el porcentaje proporcional se asigna por la suma o costo total de los destinos seleccionados en el mes de facturación actual.

:::image type="content" source="./media/allocate-costs/cost-distribution.png" alt-text="Ejemplo que muestra el porcentaje de asignación" lightbox="./media/allocate-costs/cost-distribution.png" :::

Una vez que se establecen, los porcentajes rellenados previamente se quedan fijos. Se usan para todas las asignaciones en curso. Los porcentajes solo cambian cuando la regla se actualiza manualmente.

1. Seleccione una de las siguientes opciones en la lista **Prerrellenar porcentaje**.
    - **Distribuir uniformemente**: cada uno de los destinos recibe una proporción igual del costo total.
    - **Costo total**: crea una relación proporcional a los destinos en función de su costo total. La relación se usa para distribuir los costos de los orígenes seleccionados.
    - **Costo de proceso**: crea una relación proporcional a los destinos en función de su costo de proceso de Azure (los tipos de recursos del espacio de nombres [Microsoft.Compute](https://docs.microsoft.com/azure/templates/microsoft.compute/allversions)). La relación se usa para distribuir los costos de los orígenes seleccionados.
    - **Costo de almacenamiento**: crea una relación proporcional a los destinos en función de su costo de proceso de Azure (los tipos de recursos del espacio de nombres [Microsoft.Storage](https://docs.microsoft.com/azure/templates/microsoft.storage/allversions)). La relación se usa para distribuir los costos de los orígenes seleccionados.
    - **Costo de red**: crea una relación proporcional a los destinos en función de su costo de red de Azure (los tipos de recursos del espacio de nombres [Microsoft.Network](https://docs.microsoft.com/azure/templates/microsoft.network/allversions)). La relación se usa para distribuir los costos de los orígenes seleccionados.
    - **Personalizado**: permite especificar manualmente un porcentaje de un número entero. El total especificado debe sumar un 100 %.
1. Cuando la regla se haya configurado, seleccione **Crear**.

La regla de asignación empieza a procesarse. Cuando la regla está activa, todos los costos del origen seleccionado se asignan a los destinos especificados.

> [!NOTE] 
> El procesamiento de la regla nueva puede tardar hasta dos horas en completarse y en que la regla esté activa.

## <a name="verify-the-cost-allocation-rule"></a>Verificación de la regla de asignación de costos

Cuando la regla de asignación de costos está activa, los costos de los orígenes seleccionados se distribuyen a los destinos de asignación especificados. Use la siguiente información para verificar que el costo se ha asignado correctamente a los destinos.

### <a name="view-cost-allocation-for-a-subscription"></a>Visualización de la asignación de costos de una suscripción

Vea el impacto de la regla de asignación en el análisis de costos. En Azure Portal, vaya a [Suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade). Seleccione en la lista una suscripción que sea el destino de una regla de asignación de costos activa. Luego seleccione **Análisis de costos** en el menú. En Análisis de costos, seleccione **Agrupar por** y luego seleccione **Asignación de costos**. La vista resultante muestra un rápido desglose de costos generado por la suscripción. También se muestran los costos asignados a la suscripción, como en la siguiente imagen.

:::image type="content" source="./media/allocate-costs/cost-breakdown.png" alt-text="Ejemplo que muestra el desglose de costos" lightbox="./media/allocate-costs/cost-breakdown.png" :::

### <a name="view-cost-allocation-for-a-resource-group"></a>Visualización de la asignación de costos de un grupo de recursos

Use un proceso similar para ver el impacto de una regla de asignación de costos en un grupo de recursos. En Azure Portal, vaya a [Grupo de recursos](https://portal.azure.com/#blade/HubsExtension/BrowseResourceGroups). Seleccione un grupo de recursos en la lista una suscripción que sea el destino de una regla de asignación de costos activa. Luego seleccione **Análisis de costos** en el menú. En Análisis de costos, seleccione **Agrupar por** y luego seleccione **Asignación de costos**. La vista muestra un rápido desglose de costos generado por el grupo de recursos. También se muestra el costo asignado al grupo de recursos.

### <a name="view-cost-allocation-for-tags"></a>Visualización de la asignación de costos de etiquetas

En Azure Portal, vaya a **Administración de costos + facturación** > **Administración de costos** > **Análisis de costos**. En Análisis de costos, seleccione **Agregar filtro**. Seleccione **Etiqueta**, elija la clave de la etiqueta y los valores de la etiqueta a los que se haya asignado algún costo.

:::image type="content" source="./media/allocate-costs/tagged-costs.png" alt-text="Ejemplo que muestra los costos de los elementos con etiqueta" lightbox="./media/allocate-costs/tagged-costs.png" :::

## <a name="edit-an-existing-cost-allocation-rule"></a>Edición de una regla de asignación existente

Puede editar una regla de asignación de costos para cambiar el origen o el destino, si desea actualizar el porcentaje rellenado en las opciones de proceso, almacenamiento o red. Las reglas se editan de la misma forma que se crean. La modificación de las reglas existentes puede tardar hasta dos horas en reprocesarse.

## <a name="frequently-asked-questions-faq"></a>Preguntas más frecuentes

Las siguientes secciones contienen preguntas habituales acerca de la asignación de costos.

### <a name="what-are-the-current-limitations-with-cost-allocation-in-public-preview"></a>¿Cuáles son las limitaciones actuales de la asignación de costos en versión preliminar pública?
<a name="limitations"></a>

Actualmente, la asignación de costos se admite en Cost Management por análisis de costo, presupuesto y vistas de previsión. Los costos asignados también se muestran en la lista de suscripciones y en la página de información general de Suscripciones.

Los elementos siguientes no son compatibles actualmente con la versión preliminar pública de asignación de costos:

- [Exportaciones](tutorial-export-acm-data.md) programadas
- Datos que expone [Usage Details](https://docs.microsoft.com/rest/api/consumption/usagedetails/list) API
- Área de suscripciones de facturación
- [Aplicación Azure Cost Management para Power BI](https://appsource.microsoft.com/product/power-bi/costmanagement.azurecostmanagementapp)
- [Conector de Power BI Desktop](https://docs.microsoft.com/power-bi/connect-data/desktop-connect-azure-cost-management)

### <a name="are-costs-factored-into-budgets-and-forecast-views"></a>¿Se incluyen los costos en las vistas de presupuestos y previsiones?
<a name="budgets-forecast"></a>

Sí. Los costos asignados se incluyen en los presupuestos y las previsiones, y cuentan con su respaldo. Las vistas de los presupuestos y previsiones muestran los costos que tienen asignados tal como se haya configurado en las reglas de asignación de costos.

### <a name="if-a-cost-allocation-rule-is-deleted-what-happens"></a>¿Qué sucede si se elimina una regla de asignación de costos?
<a name="delete-rule"></a>

Cuando se elimina una regla de asignación de costos, se quitan del mes de facturación actual y abierto todos los costos que se han asignado a los destinos. Si la regla de asignación de costos lleva varios meses vigente, los datos de asignación del historial de los meses anteriores permanecen tal y como se establecieron originalmente con la regla de asignación.

### <a name="why-is-an-enrollment-admin-or-a-billing-account-admin-needed-to-create-cost-allocation-rules"></a>¿Por qué se necesita un administrador de inscripciones o un administrador de cuentas de facturación para crear reglas de asignación de costos?
<a name="why-admin"></a>

Las reglas de asignación de costos se crean en el ámbito de las inscripciones (Contrato Enterprise) o en el ámbito de la cuenta de facturación (Contrato de cliente de Microsoft). Los permisos para realizar cambios en estos ámbitos requieren privilegios de administrador de facturación.

### <a name="why-are-sources-and-targets-limited-to-25-per-rule"></a>¿Por qué se impone una limitación de 25 orígenes y destinos por regla?
<a name="source-target-rule-limit"></a>

Esta es una limitación de la versión preliminar para garantizar que tanto la asignación de costos como la escalabilidad funcionen correctamente. Es probable que los límites aumenten o desaparezcan cuando la asignación de costos pase a disponibilidad general (GA).

### <a name="what-can-happen-if-cost-allocation-rules-sourcestargets-overlap"></a>¿Qué puede ocurrir si se superponen las reglas de asignación de costos (orígenes y destinos)?
<a name="rule-overlap"></a>

No se recomiendan aquellas reglas que tengan orígenes o destinos superpuestos. Las reglas de asignación de costos se aplican en orden según su fecha de creación, por lo que si se produce cualquier superposición, tendrá prioridad la que tenga la fecha anterior.

## <a name="next-steps"></a>Pasos siguientes

- Cree o actualice reglas de asignación mediante la [API REST de asignación de costos](https://go.microsoft.com/fwlink/?linkid=2135004)
- Obtenga más información sobre la [optimización de la inversión en la nube con Azure Cost Management](cost-mgt-best-practices.md)