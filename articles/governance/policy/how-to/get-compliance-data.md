---
title: Obtención de datos de cumplimiento de directiva
description: Las evaluaciones y los efectos de Azure Policy determinan el cumplimiento. Obtenga información sobre cómo obtener los detalles de cumplimiento de los recursos de Azure.
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 2ab75bdab0dcf910da91eb60b5f0cf23892d6c51
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90895434"
---
# <a name="get-compliance-data-of-azure-resources"></a>Obtención de datos de cumplimiento de los recursos de Azure

Una de las ventajas más grandes de Azure Policy es la información y los controles que proporciona sobre los recursos de una suscripción o [grupo de administración](../../management-groups/overview.md) de suscripciones. Este control puede ejercerse de muchas maneras diferentes, por ejemplo: evitar que los recursos se creen en una ubicación incorrecta, forzar un uso común y coherente de las etiquetas, o auditar los recursos existentes para las opciones y configuraciones apropiadas. En todos los casos, los datos se generan en Azure Policy para que pueda conocer el estado de cumplimiento de su entorno.

Hay varias maneras de acceder a la información de cumplimiento generada por la directiva y las asignaciones de iniciativa:

- Mediante [Azure Portal](#portal)
- Mediante scripting de [línea de comandos](#command-line)

Antes de pasar a los métodos que informan sobre el cumplimiento, veamos cuándo se actualiza la información de cumplimiento y la frecuencia y eventos que desencadenan un ciclo de evaluación.

> [!WARNING]
> Si el estado de cumplimiento se notifica como **No registrado**, compruebe que el proveedor de recursos **Microsoft.PolicyInsights** esté registrado y que el usuario tenga los permisos de control de acceso basado en rol (RBAC) adecuados, tal como se describe en [RBAC en Azure Policy](../overview.md#rbac-permissions-in-azure-policy).

## <a name="evaluation-triggers"></a>Desencadenadores de evaluación

Los resultados de un ciclo de evaluación completo están disponibles en el proveedor de recursos `Microsoft.PolicyInsights` por medio de las operaciones `PolicyStates` y `PolicyEvents`. Para más información sobre las operaciones de la API REST Azure Policy Insights, consulte [Azure Policy Insights](/rest/api/policy-insights/).

Las evaluaciones de directivas asignadas e iniciativas se producen como resultado de varios eventos:

- Una directiva o iniciativa se asigna recientemente a un ámbito. La asignación tarda unos 30 minutos en aplicarse al ámbito definido. Una vez que se aplica, comienza el ciclo de evaluación de recursos dentro de ese ámbito con la directiva o iniciativa recién asignadas y, dependiendo de los efectos usados por la directiva o iniciativa, los recursos se marcan como compatibles, no compatibles o exentos. Una directiva o iniciativa grande evaluada en un ámbito amplio de recursos puede tardar bastante tiempo. Por tanto, no hay una predicción de cuándo se completará el ciclo de la evaluación. Una vez completado, los resultados de cumplimiento actualizados están disponibles en el portal y en los SDK.

- Una directiva o iniciativa que ya está asignada a un ámbito se actualiza. El ciclo de evaluación y control de tiempo en este escenario es el mismo que para una nueva asignación a un ámbito.

- Un recurso se implementa o actualiza en un ámbito con una asignación mediante Azure Resource Manager, la API REST o un SDK compatible. En este escenario, el evento de efecto (anexar, auditar, denegar, implementar) y la información de estado de cumplimiento para el recurso individual están disponibles en el portal y en los SDK unos 15 minutos más tarde. Este evento no causa una evaluación de otros recursos.

- Se crea, actualiza o elimina una [directiva de exención](../concepts/exemption-structure.md). En este escenario, se evalúa el ámbito de exención definido en la asignación correspondiente.

- Ciclo de evaluación de cumplimiento estándar. Una vez cada 24 horas, las asignaciones se vuelven a evaluar automáticamente. Una directiva o iniciativa grande de muchos recursos puede tardar bastante tiempo, por lo que no hay una predicción de cuándo se completará el ciclo de evaluación. Una vez completado, los resultados de cumplimiento actualizados están disponibles en el portal y en los SDK.

- Un recurso administrado actualiza el proveedor de recursos [Configuración de invitado](../concepts/guest-configuration.md) con detalles de cumplimiento.

- Examen a petición

### <a name="on-demand-evaluation-scan"></a>Examen de evaluación a petición

Un examen de evaluación de una suscripción o de un grupo de recursos se puede iniciar con la CLI de Azure, Azure PowerShell o con una llamada a la API REST. Este examen es un proceso asincrónico.

#### <a name="on-demand-evaluation-scan---azure-cli"></a>Análisis de evaluación a petición: CLI de Azure

El examen de cumplimiento se inicia con el comando [az policy state trigger-scan](/cli/azure/policy/state#az-policy-state-trigger-scan).

De forma predeterminada, `az policy state trigger-scan` inicia una evaluación de todos los recursos de la suscripción actual. Para iniciar una evaluación en un grupo de recursos específico, use el parámetro **resource-group**. En el ejemplo siguiente se inicia un examen de cumplimiento de la suscripción actual para el grupo de recursos _MyRG_:

```azurecli-interactive
az policy state trigger-scan --resource-group "MyRG"
```

Puede no esperar a que el proceso asincrónico se complete antes de continuar con el parámetro **no-wait**.

#### <a name="on-demand-evaluation-scan---azure-powershell"></a>Análisis de evaluación a petición: Azure PowerShell

El examen de cumplimiento se inicia con el cmdlet [Start-AzPolicyComplianceScan](/powershell/module/az.policyinsights/start-azpolicycompliancescan).

De forma predeterminada, `Start-AzPolicyComplianceScan` inicia una evaluación de todos los recursos de la suscripción actual. Para iniciar una evaluación en un grupo de recursos específico, use el parámetro **ResourceGroupName**. En el ejemplo siguiente se inicia un examen de cumplimiento de la suscripción actual para el grupo de recursos _MyRG_:

```azurepowershell-interactive
Start-AzPolicyComplianceScan -ResourceGroupName 'MyRG'
```

Puede hacer que PowerShell espere a que se complete la llamada asincrónica antes de proporcionar la salida de resultados o hacer que se ejecute en segundo plano como un [trabajo](/powershell/module/microsoft.powershell.core/about/about_jobs). Para usar un trabajo de PowerShell para ejecutar el examen de cumplimiento en segundo plano, use el parámetro **AsJob** y establezca el valor en un objeto, como `$job` en este ejemplo:

```azurepowershell-interactive
$job = Start-AzPolicyComplianceScan -AsJob
```

Para comprobar el estado del trabajo, compruebe el objeto `$job`. El trabajo es del tipo `Microsoft.Azure.Commands.Common.AzureLongRunningJob`. Use `Get-Member` en el objeto `$job` para ver las propiedades y los métodos disponibles.

Mientras se ejecuta el examen de cumplimiento, la comprobación del objeto `$job` genera resultados como los siguientes:

```azurepowershell-interactive
$job

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
2      Long Running O… AzureLongRunni… Running       True            localhost            Start-AzPolicyCompliance…
```

Una vez finalizada esta operación, la propiedad **Estado** cambia a _Completado_.

#### <a name="on-demand-evaluation-scan---rest"></a>Examen de evaluación a petición: REST

En cuanto proceso asincrónico, el punto de conexión de REST que inicia el examen no espera a que esté completo para responder. En su lugar, proporciona un URI para consultar el estado de la evaluación solicitada.

En cada identificador URI de la API REST, hay variables usadas que se deben reemplazar por sus propios valores:

- `{YourRG}`: reemplácelo por el nombre del grupo de recursos
- `{subscriptionId}`: reemplácelo por el identificador de suscripción

El examen admite la evaluación de recursos de una suscripción o de un grupo de recursos. Inicie un examen para el ámbito con un comando **POST** de API REST mediante las siguientes estructuras de URI:

- Subscription

  ```http
  POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/triggerEvaluation?api-version=2019-10-01
  ```

- Resource group

  ```http
  POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{YourRG}/providers/Microsoft.PolicyInsights/policyStates/latest/triggerEvaluation?api-version=2019-10-01
  ```

La llamada devuelve un estado **202 - Aceptado**. En el encabezado de la respuesta se incluye una propiedad **Location** con el formato siguiente:

```http
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/asyncOperationResults/{ResourceContainerGUID}?api-version=2019-10-01
```

`{ResourceContainerGUID}` se genera estáticamente para el ámbito solicitado. Si un ámbito ya está ejecutando un examen a petición, no se iniciará un nuevo examen. En su lugar, se proporciona a la nueva solicitud el mismo URI de **ubicación** de `{ResourceContainerGUID}` para el estado. Un comando **GET** de API REST en el URI **Location** devolverá una respuesta **202 - Aceptado** mientras la evaluación esté en curso. Cuando haya finalizado el examen de evaluación, devolverá un estado **200 OK**. El cuerpo de un examen completo es una respuesta JSON con el estado:

```json
{
    "status": "Succeeded"
}
```

## <a name="how-compliance-works"></a>Cómo funciona el cumplimiento

En una asignación, un recurso es **No compatible** si no cumple las reglas de la iniciativa o la directiva y no está _exento_. En la tabla siguiente se muestra cómo funcionan los distintos efectos de directiva con la evaluación de condición para el estado de cumplimiento resultante:

| Estado del recurso | Efecto | Evaluación de directiva | Estado de cumplimiento |
| --- | --- | --- | --- |
| Exists | Deny, Audit, Append\*, DeployIfNotExist\*, AuditIfNotExist\* | True | Incompatible |
| Exists | Deny, Audit, Append\*, DeployIfNotExist\*, AuditIfNotExist\* | False | Compatible |
| Nuevo | Audit, AuditIfNotExist\* | True | Incompatible |
| Nuevo | Audit, AuditIfNotExist\* | False | Compatible |

\* Los efectos Modify, Append, DeployIfNotExist y AuditIfNotExist requieren que la instrucción IF sea TRUE. Los efectos requieren también que la condición de existencia sea FALSE para ser no compatibles. Si es TRUE, la condición IF desencadena la evaluación de la condición de existencia de los recursos relacionados.

Por ejemplo, suponga que tiene un grupo de recursos (ContosoRG) con varias cuentas de almacenamiento (resaltadas en rojo) expuestas a redes públicas.

:::image type="complex" source="../media/getting-compliance-data/resource-group01.png" alt-text="Diagrama de cuentas de almacenamiento expuestas a redes públicas en el grupo de recursos Contoso R G." border="false":::
   Diagrama que muestra imágenes de cinco cuentas de almacenamiento en el grupo de recursos Contoso R G.  Las cuentas de almacenamiento una y tres son azules, mientras que las cuentas de almacenamiento dos, cuatro y cinco son rojas.
:::image-end:::

En este ejemplo, debe tener cuidado con los riesgos de seguridad. Ahora que ha creado una asignación de directiva, se evalúa en todas las cuentas de almacenamiento incluidas y no exentas en el grupo de recursos ContosoRG. Se auditan las tres cuentas de almacenamiento no compatibles, con lo que sus estados cambian a **No compatible**.

:::image type="complex" source="../media/getting-compliance-data/resource-group03.png" alt-text="Diagrama del cumplimiento de las cuentas de almacenamiento en el grupo de recursos Contoso R G." border="false":::
   Diagrama que muestra imágenes de cinco cuentas de almacenamiento en el grupo de recursos Contoso R G. Las cuentas de almacenamiento uno y tres ahora tienen marcas de verificación verdes debajo, mientras que las cuentas de almacenamiento dos, cuatro y cinco ahora tienen signos de advertencia rojos debajo.
:::image-end:::

Además de **Compatible** y **No compatible**, las directivas y los recursos tienen otros tres estados:

- **Exento**: el recurso está en el ámbito de una asignación, pero tiene una [exención definida](../concepts/exemption-structure.md).
- **En conflicto**: existen dos o más definiciones de directivas con reglas en conflicto. Por ejemplo, dos definiciones anexan la misma etiqueta con valores diferentes.
- **No iniciado**: no se ha iniciado el ciclo de evaluación de la directiva o del recurso.
- **No registrado**: no se ha registrado el proveedor de recursos de Azure Policy o la cuenta con que se ha iniciado sesión no tiene permiso para leer datos de cumplimiento.

Azure Policy usa los campos de **tipo** y **nombre** en la definición para determinar si un recurso coincide. Cuando el recurso coincide, se considera aplicable y tiene un estado de **Compatible**, **No compatible** o **Exento**. Si la única propiedad de la definición es **type** o **name**, todos los recursos incluidos y exentos se consideran aplicables y se evalúan.

El porcentaje de cumplimiento se determina dividiendo los recursos **compatibles** y **exentos** entre los _recursos totales_. Los _recursos totales_ se definen como la suma de los recursos **compatibles**, **no compatibles**, **exentos** y **en conflicto**. La cifra de cumplimiento general es la suma de los distintos recursos que son **compatibles** o **exentos** dividida entre la suma de todos los recursos distintos. En la imagen siguiente, hay 20 recursos diferentes que son aplicables y solo uno es **No compatible**.
Por tanto, el cumplimiento general de los recursos es del 95 % (19 de 20).

:::image type="content" source="../media/getting-compliance-data/simple-compliance.png" alt-text="Captura de pantalla de los detalles de cumplimiento de directiva de la página Cumplimiento." border="false":::

> [!NOTE]
> Cumplimiento normativo de Azure Policy es una característica en versión preliminar. Las propiedades de cumplimiento del SDK y las páginas del portal son diferentes para las iniciativas habilitadas. Para obtener más información, consulte [Cumplimiento normativo](../concepts/regulatory-compliance.md).

## <a name="portal"></a>Portal

Azure Portal presenta de forma gráfica la visualización y el reconocimiento del estado de cumplimiento de normas en su entorno. En la página **Directiva**, la opción **Información general** proporciona detalles sobre el cumplimiento de directivas e iniciativas por parte de los ámbitos disponibles. Junto con el recuento y el estado de cumplimiento por asignación, contiene un gráfico que muestra el cumplimiento durante los últimos siete días. La página **Cumplimiento** contiene gran parte de esta misma información (excepto el gráfico), pero ofrece más opciones de filtrado y ordenación.

:::image type="content" source="../media/getting-compliance-data/compliance-page.png" alt-text="Captura de pantalla de la página Cumplimiento, opciones de filtrado y detalles." border="false":::

Dado que una directiva o iniciativa se puede asignar a distintos ámbitos, la tabla incluye el ámbito de cada asignación y el tipo de definición que se asignó. También se proporciona el número de directivas y recursos no compatibles para cada asignación. Al seleccionar una directiva o una iniciativa de la tabla, se proporciona más información sobre el cumplimiento de esa asignación en particular.

:::image type="content" source="../media/getting-compliance-data/compliance-details.png" alt-text="Captura de pantalla de la página Detalles de cumplimiento, incluidos los recuentos y la información de cumplimiento de los recursos." border="false":::

La lista de recursos de la pestaña **Compatibilidad de recursos** muestra el estado de evaluación de los recursos existentes para la asignación actual. El valor predeterminado de la pestaña es **No compatible**, pero se puede filtrar.
Los eventos (anexar, auditar, denegar, implementar) que desencadena la solicitud para crear un recurso se muestran en la pestaña **Eventos**.

> [!NOTE]
> En el caso de una directiva del motor de AKS, el recurso que se muestra es el grupo de recursos.

:::image type="content" source="../media/getting-compliance-data/compliance-events.png" alt-text="Captura de pantalla de la pestaña Eventos de la página Detalles de cumplimiento." border="false":::

Para los recursos del [modo de proveedor de recursos](../concepts/definition-structure.md#resource-provider-modes), en la pestaña **Resource compliance** (Compatibilidad de recursos), si selecciona el recurso o hace clic con el botón derecho en la fila y selecciona **View compliance details** (Ver detalles de cumplimiento), se abren los detalles de cumplimiento de componentes. Esta página también ofrece pestañas para ver las directivas que se asignan a este recurso, eventos, eventos de componentes e historial de cambios.

:::image type="content" source="../media/getting-compliance-data/compliance-components.png" alt-text="Captura de pantalla de la pestaña Compatibilidad de componentes y detalles de cumplimiento de una asignación de modo de proveedor de recursos." border="false":::

De vuelta a la página de cumplimiento de recursos, haga clic con el botón derecho en la fila del evento sobre el que quiere recopilar información más detallada y seleccione **Show activity logs** (Mostrar los registros de actividad). Se abre la página de registro de actividad y se filtra previamente para mostrar detalles de la asignación y los eventos. El registro de actividad proporciona contexto e información adicionales sobre esos eventos.

:::image type="content" source="../media/getting-compliance-data/compliance-activitylog.png" alt-text="Captura de pantalla del registro de actividad para actividades y evaluaciones de Azure Policy." border="false":::

### <a name="understand-non-compliance"></a>Qué significa no cumplimiento

Cuando se determina que un recurso **no es compatible**, hay muchas razones posibles para ello. Para determinar el motivo de que un recurso **no sea compatible** o para buscar el responsable del cambio, consulte [Determinación del incumplimiento](./determine-non-compliance.md).

## <a name="command-line"></a>Línea de comandos

La misma información que está disponible en el portal se puede recuperar mediante la API REST (incluso con [ARMClient](https://github.com/projectkudu/ARMClient)), Azure PowerShell y la CLI de Azure. Para detalles completos sobre la API REST, consulte la referencia de [Azure Policy Insights](/rest/api/policy-insights/). Las páginas de referencia de la API de REST tienen un botón verde en cada operación para realizar una prueba en el explorador.

Use ARMClient o una herramienta similar para tratar la autenticación en Azure para los ejemplos de la API REST.

### <a name="summarize-results"></a>Resumen de resultados

Con la API REST, el resumen se puede realizar por contenedor, definición o asignación. Este es un ejemplo de resumen en el nivel de suscripción mediante [Summarize For Subscription](/rest/api/policy-insights/policystates/summarizeforsubscription) (Resumen de la suscripción) de Azure Policy Insights:

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/summarize?api-version=2019-10-01
```

La salida resume la suscripción. En la salida de ejemplo siguiente, la compatibilidad resumida está en **value.results.nonCompliantResources** y **value.results.nonCompliantPolicies**. Esta solicitud proporciona más detalles, incluida cada asignación que compone los números no compatibles y la información de definición de cada asignación. Cada objeto de directiva de la jerarquía proporciona un **queryResultsUri** que puede utilizarse para obtener detalles adicionales en ese nivel.

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#summary",
    "@odata.count": 1,
    "value": [{
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#summary/$entity",
        "results": {
            "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=ComplianceState eq 'NonCompliant'",
            "nonCompliantResources": 15,
            "nonCompliantPolicies": 1
        },
        "policyAssignments": [{
            "policyAssignmentId": "/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77",
            "policySetDefinitionId": "",
            "results": {
                "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=ComplianceState eq 'NonCompliant' and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77'",
                "nonCompliantResources": 15,
                "nonCompliantPolicies": 1
            },
            "policyDefinitions": [{
                "policyDefinitionReferenceId": "",
                "policyDefinitionId": "/providers/microsoft.authorization/policydefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "effect": "deny",
                "results": {
                    "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=ComplianceState eq 'NonCompliant' and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77' and PolicyDefinitionId eq '/providers/microsoft.authorization/policydefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62'",
                    "nonCompliantResources": 15
                }
            }]
        }]
    }]
}
```

### <a name="query-for-resources"></a>Consulta de recursos

En el ejemplo anterior, **value.policyAssignments.policyDefinitions.results.queryResultsUri** proporciona un URI de ejemplo para todos los recursos no compatibles de una definición de directiva específica. Al examinar el valor de **$filter**, ComplianceState es igual (eq) a "NonCompliant", se ha especificado PolicyAssignmentId para la definición de directiva y, después, se ha especificado el propio PolicyDefinitionId. La razón para incluir PolicyAssignmentId en el filtro es que PolicyDefinitionId podría existir en varias asignaciones de directivas o de iniciativas con diversos ámbitos. Al especificar PolicyAssignmentId y PolicyDefinitionId, podremos ser explícitos en los resultados que estamos buscando. Anteriormente, para PolicyStates usábamos **más reciente**, que establece automáticamente una ventana temporal **desde** y **hasta** de las últimas 24 horas.

```http
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=ComplianceState eq 'NonCompliant' and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77' and PolicyDefinitionId eq '/providers/microsoft.authorization/policydefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62'
```

La respuesta del ejemplo siguiente se ha reducido a un único recurso no compatible para mayor brevedad. La respuesta detallada tiene varios elementos de datos sobre el recurso, la directiva (o iniciativa) y la asignación. Tenga en cuenta que también puede ver qué parámetros de asignación se han pasado a la definición de directiva.

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
    "@odata.count": 15,
    "value": [{
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
        "timestamp": "2018-05-19T04:41:09Z",
        "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/rg-tags/providers/Microsoft.Compute/virtualMachines/linux",
        "policyAssignmentId": "/subscriptions/{subscriptionId}/resourceGroups/rg-tags/providers/Microsoft.Authorization/policyAssignments/37ce239ae4304622914f0c77",
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
        "effectiveParameters": "",
        "ComplianceState": "NonCompliant",
        "subscriptionId": "{subscriptionId}",
        "resourceType": "/Microsoft.Compute/virtualMachines",
        "resourceLocation": "westus2",
        "resourceGroup": "RG-Tags",
        "resourceTags": "tbd",
        "policyAssignmentName": "37ce239ae4304622914f0c77",
        "policyAssignmentOwner": "tbd",
        "policyAssignmentParameters": "{\"tagName\":{\"value\":\"costCenter\"},\"tagValue\":{\"value\":\"Contoso-Test\"}}",
        "policyAssignmentScope": "/subscriptions/{subscriptionId}/resourceGroups/RG-Tags",
        "policyDefinitionName": "1e30110a-5ceb-460c-a204-c1c3969c6d62",
        "policyDefinitionAction": "deny",
        "policyDefinitionCategory": "tbd",
        "policySetDefinitionId": "",
        "policySetDefinitionName": "",
        "policySetDefinitionOwner": "",
        "policySetDefinitionCategory": "",
        "policySetDefinitionParameters": "",
        "managementGroupIds": "",
        "policyDefinitionReferenceId": ""
    }]
}
```

### <a name="view-events"></a>Ver eventos

Cuando se crea o actualiza un recurso, se genera un resultado de evaluación de directiva. Los resultados se denominan _eventos de directiva_. Utilice el siguiente Uri para ver eventos recientes de directiva asociados a la suscripción.

```http
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyEvents/default/queryResults?api-version=2019-10-01
```

Los resultados deben tener una apariencia similar al ejemplo siguiente:

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyEvents/$metadata#default",
    "@odata.count": 1,
    "value": [{
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyEvents/$metadata#default/$entity",
        "NumAuditEvents": 16
    }]
}
```

Para más información sobre cómo consultar eventos de directiva, consulte el artículo de referencia sobre los [estados de Azure Policy](/rest/api/policy-insights/policyevents).

### <a name="azure-cli"></a>Azure CLI

El grupo de comandos de la [CLI de Azure](/cli/azure/what-is-azure-cli) para Azure Policy cubre la mayoría de las operaciones que están disponibles en REST o Azure PowerShell. Para obtener la lista completa de los comandos disponibles, consulte la [Información general sobre Azure Policy en la CLI de Azure](/cli/azure/policy).

Ejemplo: Obtener el resumen de estado para la directiva asignada superior con el mayor número de recursos no compatibles.

```azurecli-interactive
az policy state summarize --top 1
```

La parte superior de la respuesta es similar a la de este ejemplo:

```json
{
    "odatacontext": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#summary/$entity",
    "odataid": null,
    "policyAssignments": [{
            "policyAssignmentId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/e0704696df5e4c3c81c873e8",
            "policyDefinitions": [{
                "effect": "audit",
                "policyDefinitionGroupNames": [
                    ""
                ],
                "policyDefinitionId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policydefinitions/2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
                "policyDefinitionReferenceId": "",
                "results": {
                    "nonCompliantPolicies": null,
                    "nonCompliantResources": 398,
                    "policyDetails": [{
                        "complianceState": "noncompliant",
                        "count": 1
                    }],
                    "policyGroupDetails": [{
                        "complianceState": "noncompliant",
                        "count": 1
                    }],
                    "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2020-07-14 14:01:22Z&$to=2020-07-15 14:01:22Z and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/e0704696df5e4c3c81c873e8' and PolicyDefinitionId eq '/subscriptions/{subscriptionId}/providers/microsoft.authorization/policydefinitions/2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a'",
                    "resourceDetails": [{
                            "complianceState": "noncompliant",
                            "count": 398
                        },
                        {
                            "complianceState": "compliant",
                            "count": 4
                        }
                    ]
                }
            }],
    ...
```

Ejemplo: Obtener el registro de estado para el recurso evaluado más recientemente (el valor predeterminado es por marca de tiempo en orden descendente).

```azurecli-interactive
az policy state list --top 1
```

```json
[
  {
    "complianceReasonCode": "",
    "complianceState": "Compliant",
    "effectiveParameters": "",
    "isCompliant": true,
    "managementGroupIds": "{managementgroupId}",
    "odatacontext": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
    "odataid": null,
    "policyAssignmentId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/securitycenterbuiltin",
    "policyAssignmentName": "SecurityCenterBuiltIn",
    "policyAssignmentOwner": "tbd",
    "policyAssignmentParameters": "",
    "policyAssignmentScope": "/subscriptions/{subscriptionId}",
    "policyAssignmentVersion": "",
    "policyDefinitionAction": "auditifnotexists",
    "policyDefinitionCategory": "tbd",
    "policyDefinitionGroupNames": [
      ""
    ],
    "policyDefinitionId": "/providers/microsoft.authorization/policydefinitions/aa633080-8b72-40c4-a2d7-d00c03e80bed",
    "policyDefinitionName": "aa633080-8b72-40c4-a2d7-d00c03e80bed",
    "policyDefinitionReferenceId": "identityenablemfaforownerpermissionsmonitoring",
    "policyDefinitionVersion": "",
    "policyEvaluationDetails": null,
    "policySetDefinitionCategory": "security center",
    "policySetDefinitionId": "/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8",
    "policySetDefinitionName": "1f3afdf9-d0c9-4c3d-847f-89da613e70a8",
    "policySetDefinitionOwner": "",
    "policySetDefinitionParameters": "",
    "policySetDefinitionVersion": "",
    "resourceGroup": "",
    "resourceId": "/subscriptions/{subscriptionId}",
    "resourceLocation": "",
    "resourceTags": "tbd",
    "resourceType": "Microsoft.Resources/subscriptions",
    "subscriptionId": "{subscriptionId}",
    "timestamp": "2020-07-15T08:37:07.903433+00:00"
  }
]
```

Ejemplo: Obtener los detalles de todos los recursos de red virtual no compatibles.

```azurecli-interactive
az policy state list --filter "ResourceType eq 'Microsoft.Network/virtualNetworks'"
```

```json
[
  {
    "complianceReasonCode": "",
    "complianceState": "NonCompliant",
    "effectiveParameters": "",
    "isCompliant": false,
    "managementGroupIds": "{managementgroupId}",
    "odatacontext": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
    "odataid": null,
    "policyAssignmentId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/e0704696df5e4c3c81c873e8",
    "policyAssignmentName": "e0704696df5e4c3c81c873e8",
    "policyAssignmentOwner": "tbd",
    "policyAssignmentParameters": "",
    "policyAssignmentScope": "/subscriptions/{subscriptionId}",
    "policyAssignmentVersion": "",
    "policyDefinitionAction": "audit",
    "policyDefinitionCategory": "tbd",
    "policyDefinitionGroupNames": [
      ""
    ],
    "policyDefinitionId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policydefinitions/2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
    "policyDefinitionName": "2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
    "policyDefinitionReferenceId": "",
    "policyDefinitionVersion": "",
    "policyEvaluationDetails": null,
    "policySetDefinitionCategory": "",
    "policySetDefinitionId": "",
    "policySetDefinitionName": "",
    "policySetDefinitionOwner": "",
    "policySetDefinitionParameters": "",
    "policySetDefinitionVersion": "",
    "resourceGroup": "RG-Tags",
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Microsoft.Network/virtualNetworks/RG-Tags-vnet",
    "resourceLocation": "westus2",
    "resourceTags": "tbd",
    "resourceType": "Microsoft.Network/virtualNetworks",
    "subscriptionId": "{subscriptionId}",
    "timestamp": "2020-07-15T08:37:07.901911+00:00"
  }
]
```

Ejemplo: Obtener eventos relacionados con recursos de red virtual no compatibles que se produjeron después de una fecha concreta.

```azurecli-interactive
az policy state list --filter "ResourceType eq 'Microsoft.Network/virtualNetworks'" --from '2020-07-14T00:00:00Z'
```

```json
[
  {
    "complianceReasonCode": "",
    "complianceState": "NonCompliant",
    "effectiveParameters": "",
    "isCompliant": false,
    "managementGroupIds": "{managementgroupId}",
    "odatacontext": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
    "odataid": null,
    "policyAssignmentId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/e0704696df5e4c3c81c873e8",
    "policyAssignmentName": "e0704696df5e4c3c81c873e8",
    "policyAssignmentOwner": "tbd",
    "policyAssignmentParameters": "",
    "policyAssignmentScope": "/subscriptions/{subscriptionId}",
    "policyAssignmentVersion": "",
    "policyDefinitionAction": "audit",
    "policyDefinitionCategory": "tbd",
    "policyDefinitionGroupNames": [
      ""
    ],
    "policyDefinitionId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policydefinitions/2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
    "policyDefinitionName": "2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
    "policyDefinitionReferenceId": "",
    "policyDefinitionVersion": "",
    "policyEvaluationDetails": null,
    "policySetDefinitionCategory": "",
    "policySetDefinitionId": "",
    "policySetDefinitionName": "",
    "policySetDefinitionOwner": "",
    "policySetDefinitionParameters": "",
    "policySetDefinitionVersion": "",
    "resourceGroup": "RG-Tags",
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Microsoft.Network/virtualNetworks/RG-Tags-vnet",
    "resourceLocation": "westus2",
    "resourceTags": "tbd",
    "resourceType": "Microsoft.Network/virtualNetworks",
    "subscriptionId": "{subscriptionId}",
    "timestamp": "2020-07-15T08:37:07.901911+00:00"
  }
]
```

### <a name="azure-powershell"></a>Azure PowerShell

El módulo Azure PowerShell para Azure Policy está disponible en la Galería de PowerShell como [Az.PolicyInsights](https://www.powershellgallery.com/packages/Az.PolicyInsights). Con el uso de PowerShellGet, puede instalar el módulo con `Install-Module -Name Az.PolicyInsights` (asegúrese de tener instalada la última versión de [Azure PowerShell](/powershell/azure/install-az-ps)):

```azurepowershell-interactive
# Install from PowerShell Gallery via PowerShellGet
Install-Module -Name Az.PolicyInsights

# Import the downloaded module
Import-Module Az.PolicyInsights

# Login with Connect-AzAccount if not using Cloud Shell
Connect-AzAccount
```

El módulo incluye los siguientes cmdlets:

- `Get-AzPolicyStateSummary`
- `Get-AzPolicyState`
- `Get-AzPolicyEvent`
- `Get-AzPolicyRemediation`
- `Remove-AzPolicyRemediation`
- `Start-AzPolicyRemediation`
- `Stop-AzPolicyRemediation`

Ejemplo: Obtener el resumen de estado para la directiva asignada superior con el mayor número de recursos no compatibles.

```azurepowershell-interactive
PS> Get-AzPolicyStateSummary -Top 1

NonCompliantResources : 15
NonCompliantPolicies  : 1
PolicyAssignments     : {/subscriptions/{subscriptionId}/resourcegroups/RG-Tags/providers/micros
                        oft.authorization/policyassignments/37ce239ae4304622914f0c77}
```

Ejemplo: Obtener el registro de estado para el recurso evaluado más recientemente (el valor predeterminado es por marca de tiempo en orden descendente).

```azurepowershell-interactive
PS> Get-AzPolicyState -Top 1

Timestamp                  : 5/22/2018 3:47:34 PM
ResourceId                 : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Network/networkInterfaces/linux316
PolicyAssignmentId         : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Authorization/policyAssignments/37ce239ae4304622914f0c77
PolicyDefinitionId         : /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62
ComplianceState            : NonCompliant
SubscriptionId             : {subscriptionId}
ResourceType               : /Microsoft.Network/networkInterfaces
ResourceLocation           : westus2
ResourceGroup              : RG-Tags
ResourceTags               : tbd
PolicyAssignmentName       : 37ce239ae4304622914f0c77
PolicyAssignmentOwner      : tbd
PolicyAssignmentParameters : {"tagName":{"value":"costCenter"},"tagValue":{"value":"Contoso-Test"}}
PolicyAssignmentScope      : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags
PolicyDefinitionName       : 1e30110a-5ceb-460c-a204-c1c3969c6d62
PolicyDefinitionAction     : deny
PolicyDefinitionCategory   : tbd
```

Ejemplo: Obtener los detalles de todos los recursos de red virtual no compatibles.

```azurepowershell-interactive
PS> Get-AzPolicyState -Filter "ResourceType eq '/Microsoft.Network/virtualNetworks'"

Timestamp                  : 5/22/2018 4:02:20 PM
ResourceId                 : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Network/virtualNetworks/RG-Tags-vnet
PolicyAssignmentId         : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Authorization/policyAssignments/37ce239ae4304622914f0c77
PolicyDefinitionId         : /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62
ComplianceState            : NonCompliant
SubscriptionId             : {subscriptionId}
ResourceType               : /Microsoft.Network/virtualNetworks
ResourceLocation           : westus2
ResourceGroup              : RG-Tags
ResourceTags               : tbd
PolicyAssignmentName       : 37ce239ae4304622914f0c77
PolicyAssignmentOwner      : tbd
PolicyAssignmentParameters : {"tagName":{"value":"costCenter"},"tagValue":{"value":"Contoso-Test"}}
PolicyAssignmentScope      : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags
PolicyDefinitionName       : 1e30110a-5ceb-460c-a204-c1c3969c6d62
PolicyDefinitionAction     : deny
PolicyDefinitionCategory   : tbd
```

Ejemplo: Obtener eventos relacionados con recursos de red virtual no compatibles que se produjeron después de una fecha concreta.

```azurepowershell-interactive
PS> Get-AzPolicyEvent -Filter "ResourceType eq '/Microsoft.Network/virtualNetworks'" -From '2018-05-19'

Timestamp                  : 5/19/2018 5:18:53 AM
ResourceId                 : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Network/virtualNetworks/RG-Tags-vnet
PolicyAssignmentId         : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Authorization/policyAssignments/37ce239ae4304622914f0c77
PolicyDefinitionId         : /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62
ComplianceState            : NonCompliant
SubscriptionId             : {subscriptionId}
ResourceType               : /Microsoft.Network/virtualNetworks
ResourceLocation           : eastus
ResourceGroup              : RG-Tags
ResourceTags               : tbd
PolicyAssignmentName       : 37ce239ae4304622914f0c77
PolicyAssignmentOwner      : tbd
PolicyAssignmentParameters : {"tagName":{"value":"costCenter"},"tagValue":{"value":"Contoso-Test"}}
PolicyAssignmentScope      : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags
PolicyDefinitionName       : 1e30110a-5ceb-460c-a204-c1c3969c6d62
PolicyDefinitionAction     : deny
PolicyDefinitionCategory   : tbd
TenantId                   : {tenantId}
PrincipalOid               : {principalOid}
```

El campo **PrincipalOid** se puede utilizar para obtener un usuario específico con el cmdlet `Get-AzADUser` de Azure PowerShell. Reemplace **{principalOid}** por la respuesta que se obtiene del ejemplo anterior.

```azurepowershell-interactive
PS> (Get-AzADUser -ObjectId {principalOid}).DisplayName
Trent Baker
```

## <a name="azure-monitor-logs"></a>Registros de Azure Monitor

Si tiene un [área de trabajo de Log Analytics](../../../azure-monitor/log-query/log-query-overview.md) con `AzureActivity` de la [solución Activity Log Analytics](../../../azure-monitor/platform/activity-log.md) vinculada a su suscripción, también puede ver los resultados de no cumplimiento del ciclo de evaluación mediante consultas sencillas de Kusto y la tabla `AzureActivity`. Con los detalles de los registros de Azure Monitor, se pueden configurar alertas para comprobar la opción de no compatibilidad.

:::image type="content" source="../media/getting-compliance-data/compliance-loganalytics.png" alt-text="Captura de pantalla de registros de Azure Monitor que muestran acciones de Azure Policy en la tabla AzureActivity." border="false":::

## <a name="next-steps"></a>Pasos siguientes

- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](programmatically-create.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
