---
title: Reglas de acción para las alertas de Azure Monitor
description: Descripción de qué son las reglas de acción en Azure Monitor y cómo configurarlas y administrarlas.
ms.topic: conceptual
ms.date: 04/25/2019
ms.subservice: alerts
ms.openlocfilehash: 083db4ad046ee586f139309b62eedf0fcc2ffa6a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87045716"
---
# <a name="action-rules-preview"></a>Reglas de acción (versión preliminar)

Las reglas de acción le ayudan a definir o suprimir acciones en cualquier ámbito de Azure Resource Manager (suscripción, grupo de recursos o recurso de destino de Azure). Disponen de varios filtros que le ayudan a reducir el subconjunto específico de instancias de alerta sobre las que quiere actuar.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4rBZ2]

## <a name="why-and-when-should-you-use-action-rules"></a>¿Por qué y cuándo deben usar reglas de acción?

### <a name="suppression-of-alerts"></a>Supresión de alertas

Hay muchos escenarios en los que resulta útil suprimir las notificaciones que generan las alertas. Estos escenarios van desde la supresión durante una ventana de mantenimiento planeado hasta la supresión durante las horas no laborables. Por ejemplo, el equipo responsable de **ContosoVM** desea suprimir las notificaciones de alerta durante el fin de semana próximo, puesto que **ContosoVM** está llevando a cabo un mantenimiento planeado.

Aunque el equipo puede deshabilitar manualmente todas las reglas de alerta configuradas en **ContosoVM** (y habilitarlas de nuevo después del mantenimiento), no es un proceso sencillo. Las reglas de acción le permiten definir la supresión de alertas a escala con la posibilidad de configurar el período de supresión de forma flexible. En el ejemplo anterior, el equipo puede definir una regla de acción en **ContosoVM** que suprime todas las notificaciones de alerta durante el fin de semana.

### <a name="actions-at-scale"></a>Acciones a escala

Aunque las reglas de alertas le ayudan a definir el grupo de acciones que se desencadena cuando se genera la alerta, los clientes a menudo tienden a disponer de un grupo de acciones comunes en su ámbito de operaciones. Por ejemplo, un equipo responsable del grupo de recursos **ContosoRG** probablemente definirá el mismo grupo de acciones para todas las reglas de alerta definidas dentro de **ContosoRG** .

Las reglas de acción le ayudan a simplificar este proceso. Al definir acciones a escala, se puede desencadenar un grupo de acciones para cualquier alerta que se genere en el ámbito configurado. En el ejemplo anterior, el equipo ahora puede definir una regla de acción sobre **ContosoRG** que desencadenará el mismo grupo de acciones para todas las alertas generadas dentro de él.

> [!NOTE]
> Actualmente, las reglas de acción no se aplican a las alertas de Azure Service Health.

## <a name="configuring-an-action-rule"></a>Configuración de una regla de acciones

### <a name="portal"></a>[Portal](#tab/portal)

Para acceder a la característica, puede seleccionar **Administrar acciones** desde la página de inicio de **Alertas** en Azure Monitor. Después, seleccione **Reglas de acción (versión preliminar)** . Para acceder a las reglas, seleccione **Reglas de acción (versión preliminar)** desde el panel de la página de aterrizaje de alertas.

![Reglas de acción desde la página de inicio de Azure Monitor](media/alerts-action-rules/action-rules-landing-page.png)

Seleccione **+ Nueva regla de acción**.

![Incorporación de una nueva regla de acción](media/alerts-action-rules/action-rules-new-rule.png)

Como alternativa, también puede crear una regla de acción al configurar una regla de alerta.

![Incorporación de una nueva regla de acción](media/alerts-action-rules/action-rules-alert-rule.png)

Ahora verá la página de flujo para crear reglas de acción. Configure los elementos siguientes:

![Nuevo flujo de creación de reglas de acción](media/alerts-action-rules/action-rules-new-rule-creation-flow.png)

### <a name="scope"></a>Ámbito

En primer lugar, elija el ámbito (suscripción, grupo de recursos o recurso de destino de Azure). También puede realizar varias selecciones de una combinación de ámbitos dentro de una sola suscripción.

![Ámbito de la regla de acción](media/alerts-action-rules/action-rules-new-rule-creation-flow-scope.png)

### <a name="filter-criteria"></a>Criterios de filtro

Además, puede definir filtros para restringirlos a un subconjunto específico de las alertas.

Los filtros disponibles son:

* **Gravedad**: la opción para seleccionar uno o varios niveles de gravedad de alerta. **Gravedad = Sev1** significa que la regla de acción es aplicable a todas las alertas con un nivel de gravedad Sev1.
* **Supervisar servicio**: filtro basado en el servicio de supervisión de origen. Este filtro también tiene varias selecciones. Por ejemplo, **Supervisar servicio = "Application Insights"** significa que la regla de acción se aplica a todas las alertas basadas en Application Insights.
* **Tipo de recurso**:  filtro según un tipo de recurso específico. Este filtro también tiene varias selecciones. Por ejemplo, **Tipo de recurso = "Virtual Machines"** significa que la regla de acción es aplicable a todas las máquinas virtuales.
* **Id. de regla de alerta**: opción para filtrar por reglas de alertas específicas mediante el identificador de Resource Manager de la regla de alerta.
* **Condición del monitor**:  filtro para las instancias de alerta con la condición de supervisión **Desencadenada** o **Resuelta**.
* **Descripción**: coincidencia regex (expresión regular) que define una coincidencia de cadena con la descripción, definida como parte de la regla de alerta. Por ejemplo, **La descripción contiene "prod**" buscará la coincidencia con todas las alertas que contengan la cadena "prod" en sus descripciones.
* **Contexto de alerta (carga)** : coincidencia regex que define una coincidencia de cadena con los campos de contexto de alerta de la carga de una alerta. Por ejemplo, **Contexto de alerta (carga) contiene "Equipo 01"** buscará la coincidencia con todas las alertas cuyas cargas contengan la cadena "Equipo-01".

Estos filtros se aplican junto con otro. Por ejemplo, si define **Tipo de recurso** = "Virtual Machines" y **"Gravedad" = Sev0**, se filtran todas las alertas con **Sev0** en solo sus máquinas virtuales.

![Filtros de reglas de acción](media/alerts-action-rules/action-rules-new-rule-creation-flow-filters.png)

### <a name="suppression-or-action-group-configuration"></a>Configuración del grupo de acciones o supresiones

A continuación, configure la regla de acción para la supresión de alertas o la compatibilidad con grupos de acciones. No puede elegir ambos. La configuración actúa en todas las instancias de alerta que coincidan con el ámbito y los filtros definidos previamente.

#### <a name="suppression"></a>Supresión

Si selecciona **supresión**, configure la duración de la supresión de notificaciones y acciones. Elija una de las siguientes opciones:
* **Desde ahora (siempre)** : suprime todas las notificaciones de forma indefinida.
* **A la hora programada**: suprime las notificaciones dentro de una duración limitada.
* **Con periodicidad**: suprime las notificaciones de una programación periódica diaria, semanal o mensual.

![Supresión de la regla de acción](media/alerts-action-rules/action-rules-new-rule-creation-flow-suppression.png)

#### <a name="action-group"></a>Grupo de acciones

Si selecciona **Grupo de acciones** en el botón de alternancia, agregue un grupo de acciones existente o cree uno nuevo.

> [!NOTE]
> Puede asociar un solo grupo de acciones a una regla de acción.

![Agregar o crear una regla de acción mediante la selección del grupo de acciones](media/alerts-action-rules/action-rules-new-rule-creation-flow-action-group.png)

### <a name="action-rule-details"></a>Detalles de la regla de acción

Por último, configure los siguientes detalles de la regla de acción:
* Nombre
* Grupo de recursos en el que se guarda.
* Descripción

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Puede crear reglas de acción con la CLI de Azure mediante el comando [az monitor action-rule create](/cli/azure/ext/alertsmanagement/monitor/action-rule#ext-alertsmanagement-az-monitor-action-rule-create).  La referencia a `az monitor action-rule` es solo una de las muchas [referencias de la CLI de Azure para Azure Monitor](/cli/azure/azure-cli-reference-for-monitor).

### <a name="prepare-your-environment"></a>Preparación del entorno

1. [Instalación de la CLI de Azure](/cli/azure/install-azure-cli)

   Si lo prefiere, también puede usar Azure Cloud Shell para completar los pasos de este artículo.  Azure Cloud Shell es un entorno de shell interactivo que se utiliza mediante el explorador.  Inicie Cloud Shell con uno de estos métodos:

   - Vaya a [https://shell.azure.com](https://shell.azure.com) para abrir Cloud Shell.

   - Seleccione el botón **Cloud Shell** en la barra de menús de la esquina superior derecha de [Azure Portal](https://portal.azure.com).

1. Inicie sesión.

   Si está usando una instalación local de la CLI, inicie sesión con el comando [az login](/cli/azure/reference-index#az-login).  Siga los pasos que se muestran en el terminal para completar el proceso de autenticación.

    ```azurecli
    az login
    ```

1. Instale la extensión `alertsmanagement`

   El comando `az monitor action-rule` es una extensión experimental de la CLI de Azure principal. Obtenga más información sobre las referencias de extensión en [Uso de extensiones con la CLI de Azure](/cli/azure/azure-cli-extensions-overview?).

   ```azurecli
   az extension add --name alertsmanagement
   ```

   Se espera la advertencia siguiente.

   ```output
   The installed extension `alertsmanagement` is experimental and not covered by customer support.  Please use with discretion.
   ```

### <a name="create-action-rules-with-the-azure-cli"></a>Creación de reglas de acción con la CLI de Azure

Consulte el contenido de referencia de la CLI de Azure para [az monitor action-rule create](/cli/azure/ext/alertsmanagement/monitor/action-rule#ext-alertsmanagement-az-monitor-action-rule-create) para obtener información sobre los parámetros obligatorios y opcionales.

Cree una regla de acción para suprimir las notificaciones en un grupo de recursos.

```azurecli
az monitor action-rule create --resource-group MyResourceGroupName \
                              --name MyNewActionRuleName \
                              --location Global \
                              --status Enabled \
                              --rule-type Suppression \
                              --scope-type ResourceGroup \
                              --scope /subscriptions/0b1f6471-1bf0-4dda-aec3-cb9272f09590/resourceGroups/MyResourceGroupName \
                              --suppression-recurrence-type Always \
                              --alert-context Contains Computer-01 \
                               --monitor-service Equals "Log Analytics"
```

Cree una regla de acción para suprimir las notificaciones de todas las alertas de Sev4 en todas las VM dentro de la suscripción cada fin de semana.

```azurecli
az monitor action-rule create --resource-group MyResourceGroupName \
                              --name MyNewActionRuleName \
                              --location Global \
                              --status Enabled \
                              --rule-type Suppression \
                              --severity Equals Sev4 \
                              --target-resource-type Equals Microsoft.Compute/VirtualMachines \
                              --suppression-recurrence-type Weekly \
                              --suppression-recurrence 0 6 \
                              --suppression-start-date 12/09/2018 \
                              --suppression-end-date 12/18/2018 \
                              --suppression-start-time 06:00:00 \
                              --suppression-end-time 14:00:00

```

* * *

## <a name="example-scenarios"></a>Escenarios de ejemplo

### <a name="scenario-1-suppression-of-alerts-based-on-severity"></a>Escenario 1: Supresión de alertas en función de la gravedad

Contoso quiere suprimir las notificaciones de todas las alertas de Sev4 en todas las máquinas virtuales dentro de la suscripción **ContosoSub** cada fin de semana.

**Solución:** Cree una regla de acción con:
* Ámbito = **ContosoSub**
* Filtros
    * Gravedad = **Sev4**
    * Tipo de recurso = **Virtual Machines**
* Supresión con la periodicidad establecida en semanal y las opciones **Sábado** y **Domingo** activadas

### <a name="scenario-2-suppression-of-alerts-based-on-alert-context-payload"></a>Escenario 2: Supresión de alertas basada en el contexto de alerta (carga)

Contoso quiere suprimir las notificaciones en todos los registros de alertas generadas para **Equipo 01** en **ContosoSub** indefinidamente ya que va a someterse a mantenimiento.

**Solución:** Cree una regla de acción con:
* Ámbito = **ContosoSub**
* Filtros
    * Supervisar servicio = **Log Analytics**
    * Contexto de alerta (carga) contiene **Equipo 01**
* Supresión establecida en **Desde ahora (siempre)**

### <a name="scenario-3-action-group-defined-at-a-resource-group"></a>Escenario 3: Grupo de acciones definido en un grupo de recursos

Contoso ha definido [una alerta de métricas en el nivel de suscripción](./alerts-metric-overview.md#monitoring-at-scale-using-metric-alerts-in-azure-monitor). Pero quiere definir las acciones que se desencadenan específicamente para las alertas generadas desde el grupo de recursos **ContosoRG**.

**Solución:** Cree una regla de acción con:
* Ámbito = **ContosoRG**
* Sin filtros
* Grupo de acciones establecido en **ContosoActionGroup**

> [!NOTE]
> *Los grupos de acciones definidos dentro de las reglas de acción y las reglas de alertas funcionan de forma independiente, sin desduplicación*. En el escenario descrito anteriormente, si un grupo de acciones se define para la regla de alerta, se desencadena junto con el grupo de acciones definido en la regla de acción.

## <a name="managing-your-action-rules"></a>Administración de las reglas de acción

### <a name="portal"></a>[Portal](#tab/portal)

Puede ver y administrar las reglas de acción desde la vista de lista:

![Vista de lista de reglas de acción](media/alerts-action-rules/action-rules-list-view.png)

Desde aquí, puede habilitar, deshabilitar o eliminar reglas de acción a escala si activa la casilla que hay junto a ellas. Al seleccionar una regla de acción, se abre su página de configuración. La página le ayuda a actualizar la definición de la regla de acción y a habilitarla o deshabilitarla.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Puede ver y administrar las reglas de acción mediante el comando [az monitor action-rule](/cli/azure/ext/alertsmanagement/monitor) de la CLI de Azure.

Antes de administrar las reglas de acción con la CLI de Azure, prepare el entorno según las instrucciones proporcionadas en [Configuración de una regla de acciones](#configuring-an-action-rule).

```azurecli
# List all action rules for a subscription
az monitor action-rule list

# Get details of an action rule
az monitor action-rule show --resource-group MyResourceGroupName --name MyActionRuleName

# Update an action rule.
az monitor action-rule update --resource-group MyResourceGroupName --name MyActionRuleName --status Disabled

# Delete an action rule.
az monitor action-rule delete --resource-group MyResourceGroupName --name MyActionRuleName
```

* * *

## <a name="best-practices"></a>Procedimientos recomendados

Las alertas de registro creadas con la opción [número de resultados](alerts-unified-log.md) generan una sola instancia de alerta con el resultado de búsqueda completo (que, por ejemplo, podría extenderse a varios equipos). En este escenario, si una regla de acción usa el filtro **Contexto de alerta (carga)** , actuará sobre la instancia de alerta siempre que haya una coincidencia. En el escenario 2, descrito anteriormente, si los resultados de la búsqueda de la alerta de registro generada contienen **Equipo 01** y **Equipo 02**, se suprime toda la notificación. No se ha generado ninguna notificación para **Equipo 02**.

![Reglas de acción y alertas del registro (número de resultados)](media/alerts-action-rules/action-rules-log-alert-number-of-results.png)

Para usar mejor las alertas de registro con las reglas de acción, cree alertas de registro con la opción [unidades métricas](alerts-unified-log.md). Con esta opción, se generan instancias de alerta independientes según el campo de grupo definido. A continuación, en el escenario 2, se generan instancias de alerta independientes para **Equipo 01** y **Equipo 02**. Debido a la regla de acción descrita en el escenario, solo se suprime la notificación de **Equipo 01**. La notificación de **Equipo 02** se sigue activando de forma normal.

![Reglas de acción y alertas del registro (número de resultados)](media/alerts-action-rules/action-rules-log-alert-metric-measurement.png)

## <a name="faq"></a>Preguntas más frecuentes

### <a name="while-im-configuring-an-action-rule-id-like-to-see-all-the-possible-overlapping-action-rules-so-that-i-avoid-duplicate-notifications-is-it-possible-to-do-that"></a>Al configurar una regla de acción, me gustaría ver todas las reglas de acción que se pueden superponer de modo que se eviten notificaciones duplicadas. ¿Se puede hacer esto?

Después de definir un ámbito al configurar una regla de acción, puede ver una lista de reglas de acción que se superponen en el mismo ámbito (si la hay). Esta superposición puede ser una de las siguientes opciones:

* Una coincidencia exacta: Por ejemplo, la regla de acción que va a definir y la regla de acción que se superpone están en la misma suscripción.
* Un subconjunto: Por ejemplo, la regla de acción que va a definir se encuentra en una suscripción y la regla de acción que se superpone se encuentra en un grupo de recursos dentro de la suscripción.
* Un superconjunto: Por ejemplo, la regla de acción que va a definir se encuentra en un grupo de recursos y la regla de acción que se superpone se encuentra en la suscripción que contiene el grupo de recursos.
* Una intersección: Por ejemplo, la regla de acción que va a definir se encuentra en **VM1** y **VM2**, y la regla de acción que se superpone, en **VM2** y **VM3**.

![Reglas de acción que se superponen](media/alerts-action-rules/action-rules-overlapping.png)

### <a name="while-im-configuring-an-alert-rule-is-it-possible-to-know-if-there-are-already-action-rules-defined-that-might-act-on-the-alert-rule-im-defining"></a>Al configurar una regla de alerta, ¿es posible saber si ya hay definidas reglas de acción que podrían actuar sobre la regla de alerta que voy a definir?

Después de definir el recurso de destino para la regla de alerta, puede ver la lista de reglas de acción que actúan en el mismo ámbito (si existe); para ello, seleccione **Ver acciones configuradas** en la sección **Acciones**. Esta lista se rellena en función de los siguientes escenarios para el ámbito:

* Una coincidencia exacta: por ejemplo, la regla de alerta que va a definir y la regla de acción están en la misma suscripción.
* Un subconjunto: por ejemplo, la regla de alerta que va a definir se encuentra en una suscripción y la regla de acción se encuentra en un grupo de recursos dentro de la suscripción.
* Un superconjunto: por ejemplo, la regla de alerta que va a definir se encuentra en un grupo de recursos y la regla de acción se encuentra en la suscripción que contiene el grupo de recursos.
* Una intersección: Por ejemplo, la regla de alerta que va a definir se encuentra en **VM1** y **VM2**, y la regla de acción, en **VM2** y **VM3**.

![Reglas de acción que se superponen](media/alerts-action-rules/action-rules-alert-rule-overlapping.png)

### <a name="can-i-see-the-alerts-that-have-been-suppressed-by-an-action-rule"></a>¿Puedo ver las alertas que han sido suprimidas por una regla de acción?

En la [página de lista de alertas](./alerts-managing-alert-instances.md), puede elegir una columna adicional denominada **Estado de supresión**. Si se suprimiera la notificación para una instancia de alerta, mostraría ese estado en la lista.

![Instancias de alertas suprimidas](media/alerts-action-rules/action-rules-suppressed-alerts.png)

### <a name="if-theres-an-action-rule-with-an-action-group-and-another-with-suppression-active-on-the-same-scope-what-happens"></a>Si hay una regla de acción con un grupo de acciones y otra con la supresión activa en el mismo ámbito, ¿qué ocurre?

La supresión siempre tiene prioridad en el mismo ámbito.

### <a name="what-happens-if-i-have-a-resource-thats-monitored-in-two-separate-action-rules-do-i-get-one-or-two-notifications-for-example-vm2-in-the-following-scenario"></a>¿Qué ocurre si tengo un recurso supervisado en dos reglas de acción independientes? ¿Puedo obtener una o dos notificaciones? Por ejemplo, **VM2** en el escenario siguiente:

   `action rule AR1 defined for VM1 and VM2 with action group AG1`

   `action rule AR2 defined for VM2 and VM3 with action group AG1`

Para cada alerta en VM1 y VM3, el grupo de acciones AG1 se desencadenaría una vez. Para cada alerta en **VM2**, el grupo de acciones AG1 se desencadenaría dos veces, ya que las reglas de acción no desduplican las acciones.

### <a name="what-happens-if-i-have-a-resource-monitored-in-two-separate-action-rules-and-one-calls-for-action-while-another-for-suppression-for-example-vm2-in-the-following-scenario"></a>¿Qué ocurre si tengo un recurso supervisado en dos reglas de acción independientes y una llama a la acción mientras la otra lo hace a la supresión? Por ejemplo, **VM2** en el escenario siguiente:

   `action rule AR1 defined for VM1 and VM2 with action group AG1`

   `action rule AR2 defined for VM2 and VM3 with suppression`

Para cada alerta en VM1, el grupo de acciones AG1 se desencadenaría una vez. Se suprimirán las acciones y las notificaciones para todas las alertas en VM2 y VM3.

### <a name="what-happens-if-i-have-an-alert-rule-and-an-action-rule-defined-for-the-same-resource-calling-different-action-groups-for-example-vm1-in-the-following-scenario"></a>¿Qué ocurre si tengo una regla de alerta y una regla de acción definidas para el mismo recurso que llama a grupos de acciones diferentes? Por ejemplo, **VM1** en el escenario siguiente:

   `alert rule rule1 on VM1 with action group AG2`

   `action rule AR1 defined for VM1 with action group AG1`

Para cada alerta en VM1, el grupo de acciones AG1 se desencadenaría una vez. Cada vez que se desencadene la regla de alerta "rule1", también se desencadenará además AG2. Los grupos de acciones definidos dentro de las reglas de acción y las reglas de alertas funcionan de forma independiente, sin desduplicación.

## <a name="next-steps"></a>Pasos siguientes

- [Obtenga más información sobre alertas en Azure](./alerts-overview.md)
