---
title: Determinación de las causas de incumplimiento
description: Cuando un recurso no es compatible, hay muchos motivos posibles para ello. Descubra qué es lo que provoca que no sea compatible.
ms.date: 07/06/2020
ms.topic: how-to
ms.openlocfilehash: a666da4ecb97c24e7176e6c7cfbe2ee24f46f1b7
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89648568"
---
# <a name="determine-causes-of-non-compliance"></a>Determinación de las causas de incumplimiento

Cuando se determina que un recurso de Azure no cumple con una regla de directiva, resulta útil entender con qué parte de la regla no es compatible el recurso. También resulta útil para comprender cuál es el cambio que modificó un recurso compatible anteriormente y lo transformó en no compatible. Hay dos maneras de obtener esta información:

- [Detalles de cumplimiento](#compliance-details)
- [Historial de cambios (versión preliminar)](#change-history)

## <a name="compliance-details"></a>Detalles de cumplimiento

Cuando un recurso no es compatible, los detalles de cumplimiento para ese recurso estarán disponibles en la página **Cumplimiento de directiva**. El panel de detalles de cumplimiento incluye la siguiente información:

- Detalles de recursos como el nombre, el tipo, la ubicación y el identificador del recurso.
- Estado de compatibilidad y marca de tiempo de la última evaluación para la asignación de la directiva actual.
- Una lista de _razones_ por las que el recurso no es compatible.

> [!IMPORTANT]
> Puesto que los detalles de cumplimiento para un recurso _no compatible_ muestran el valor correcto de las propiedades en ese recurso, el usuario debe realizar la operación de **lectura** en el **tipo** de recurso. Por ejemplo, si el recurso _no compatible_ es **Microsoft.Compute/virtualMachines**, el usuario debe realizar la operación **Microsoft.Compute/virtualMachines/read**. Si el usuario no realiza la operación necesaria, se muestra un error de acceso.

Para ver los detalles de cumplimiento, siga estos pasos:

1. Inicie el servicio Azure Policy en Azure Portal. Para ello, seleccione **Todos los servicios** y, a continuación, busque y seleccione **Directiva**.

1. En la página **Información general** o **Cumplimiento**, seleccione una directiva que sea con un **estado de compatibilidad** que sea _No compatible_.

1. En la pestaña **Compatibilidad de recursos** de **Cumplimiento de directiva**, haga clic con el botón derecho o seleccione los puntos suspensivos de un recurso en un **estado de compatibilidad** que sea _No compatible_. A continuación, seleccione **Ver detalles de cumplimiento**.

   :::image type="content" source="../media/determine-non-compliance/view-compliance-details.png" alt-text="Captura de pantalla del vínculo &quot;Ver detalles de cumplimiento&quot; en la pestaña Cumplimiento de recursos." border="false":::

1. La página **Detalles de cumplimiento** muestra información de la última evaluación del recurso en la asignación de directiva actual. En este ejemplo, el campo **Microsoft.Sql/servers/version** tiene que ser _12.0_ y se espera que la definición de política sea _14.0_. Si el recurso no es compatible por varias razones, estas se muestran en este panel.

   :::image type="content" source="../media/determine-non-compliance/compliance-details-pane.png" alt-text="Captura de pantalla del vínculo &quot;Ver detalles de cumplimiento&quot; en la pestaña Cumplimiento de recursos." border="false":::

   Para una definición de directiva **auditIfNotExists** o **deployIfNotExists**, los detalles incluyen la propiedad **details.type** y cualquier propiedad opcional. Para obtener una lista, consulte [auditIfNotExists propiedades](../concepts/effects.md#auditifnotexists-properties) y [deployIfNotExists propiedades](../concepts/effects.md#deployifnotexists-properties). **Último recurso evaluado** es un recurso relacionado de la sección de **detalles** de la definición.

   Definición **deployIfNotExists** parcial de ejemplo:

   ```json
   {
       "if": {
           "field": "type",
           "equals": "[parameters('resourceType')]"
       },
       "then": {
           "effect": "DeployIfNotExists",
           "details": {
               "type": "Microsoft.Insights/metricAlerts",
               "existenceCondition": {
                   "field": "name",
                   "equals": "[concat(parameters('alertNamePrefix'), '-', resourcegroup().name, '-', field('name'))]"
               },
               "existenceScope": "subscription",
               "deployment": {
                   ...
               }
           }
       }
   }
   ```

   :::image type="content" source="../media/determine-non-compliance/compliance-details-pane-existence.png" alt-text="Captura de pantalla del vínculo &quot;Ver detalles de cumplimiento&quot; en la pestaña Cumplimiento de recursos." border="false":::

> [!NOTE]
> Para proteger los datos, cuando un valor de propiedad es _secreto_, el valor actual muestra asteriscos.

Estos detalles explican por qué un recurso no es compatible actualmente, pero no muestran cuándo se realizó el cambio en el recurso que provocó que se transformara en no compatible. Para obtener esa información, consulte [Historial de cambios (versión preliminar)](#change-history) a continuación.

### <a name="compliance-reasons"></a>Razones de cumplimiento

La siguiente matriz asigna cada _motivo_ posible a la [condición](../concepts/definition-structure.md#conditions) responsable en la definición de la directiva:

|Motivo | Condición |
|-|-|
|El valor actual debe contener el valor de destino como una clave. |containsKey o **no** notContainsKey |
|El valor actual debe contener el valor de destino. |contains o **no** notContains |
|El valor actual debe ser igual al valor de destino. |equals o **no** notEquals |
|El valor actual debe ser inferior al valor de destino. |less o **no** greaterOrEquals |
|El valor actual debe ser superior o igual al valor de destino. |greaterOrEquals o **no** less |
|El valor actual debe ser superior al valor de destino. |greater o **no** lessOrEquals |
|El valor actual debe ser inferior o igual al valor de destino. |lessOrEquals o **no** greater |
|El valor actual debe existir. |exists |
|El valor actual debe estar en el valor de destino. |in o **no** notIn |
|El valor actual debe ser como el valor de destino. |like o **no** notLike |
|El valor actual debe coincidir con distinción de mayúsculas y minúsculas con el valor de destino. |match o **no** notMatch |
|El valor actual debe coincidir sin distinción de mayúsculas y minúsculas con el valor de destino. |matchInsensitively o **no** notMatchInsensitively |
|El valor actual no debe contener el valor de destino como clave. |notContainsKey o **no** containsKey|
|El valor actual no debe contener el valor de destino. |notContains o **no** contains |
|El valor actual no debe ser igual que el valor de destino. |notEquals o **no** equals |
|El valor actual no debe existir. |**no** exists  |
|El valor actual no debe estar en el valor de destino. |notIn o **no** in |
|El valor actual no debe ser como el valor de destino. |notLike o **no** like |
|El valor actual no debe coincidir con distinción de mayúsculas y minúsculas con el valor de destino. |notMatch o **no** match |
|El valor actual no debe coincidir sin distinción de mayúsculas y minúsculas con el valor de destino. |notMatchInsensitively o **no** matchInsensitively |
|Ninguno de los recursos relacionados coincide con los detalles de vigencia de la definición de directiva. |Un recurso del tipo definido en **then.details.type** y relacionado con el recurso definido en la porción **if** de la regla de directiva no existe. |

## <a name="compliance-details-for-guest-configuration"></a>Detalles de cumplimiento de la configuración de invitado

Para las directivas _auditIfNotExists_ de la categoría _Configuración de invitados_, podría haber varias configuraciones evaluadas dentro de la máquina virtual y tendrá que ver los detalles por configuración. Por ejemplo, si va a realizar una auditoría de una lista de directivas de contraseñas y solo una de ellas tiene el estado _No compatible_, debe saber qué directivas de contraseñas específicas no cumplen los requisitos y por qué.

También es posible que no disponga de acceso para iniciar sesión en la VM directamente, pero tiene que informar sobre el motivo por el que la VM _no es compatible_.

### <a name="azure-portal"></a>Azure portal

Para empezar, siga los mismos pasos de la sección anterior para ver los detalles de cumplimiento de directivas.

En la vista del panel Detalles de cumplimiento, seleccione el vínculo **Último recurso evaluado**.

:::image type="content" source="../media/determine-non-compliance/guestconfig-auditifnotexists-compliance.png" alt-text="Captura de pantalla del vínculo &quot;Ver detalles de cumplimiento&quot; en la pestaña Cumplimiento de recursos." border="false":::

La página **Asignación de invitado** muestra todos los detalles de cumplimiento disponibles. Cada fila de la vista representa una evaluación que se realizó dentro de la máquina. En la columna **Motivo**, se muestra una frase que describe el motivo por el que Asignación de invitado es _No compatible_. Por ejemplo, si audita directivas de contraseñas, la columna **Motivo** mostraría un texto que incluye el valor actual de cada configuración.

:::image type="content" source="../media/determine-non-compliance/guestconfig-compliance-details.png" alt-text="Captura de pantalla del vínculo &quot;Ver detalles de cumplimiento&quot; en la pestaña Cumplimiento de recursos." border="false":::

### <a name="azure-powershell"></a>Azure PowerShell

También puede ver los detalles de cumplimiento en Azure PowerShell. En primer lugar, asegúrese de tener el módulo de configuración de invitado instalado.

```azurepowershell-interactive
Install-Module Az.GuestConfiguration
```

Puede ver el estado actual de todas las asignaciones de invitado para una VM mediante el comando siguiente:

```azurepowershell-interactive
Get-AzVMGuestPolicyStatus -ResourceGroupName <resourcegroupname> -VMName <vmname>
```

```output
PolicyDisplayName                                                         ComplianceReasons
-----------------                                                         -----------------
Audit that an application is installed inside Windows VMs                 {[InstalledApplication]bwhitelistedapp}
Audit that an application is not installed inside Windows VMs.            {[InstalledApplication]NotInstalledApplica...
```

Para ver solo la frase del _motivo_ que describe la causa por la que la VM _no es compatible_, vuelva solo a la propiedad secundaria del motivo.

```azurepowershell-interactive
Get-AzVMGuestPolicyStatus -ResourceGroupName <resourcegroupname> -VMName <vmname> | % ComplianceReasons | % Reasons | % Reason
```

```output
The following applications are not installed: '<name>'.
```

También puede generar un historial de cumplimiento de las asignaciones de invitados en el ámbito de la máquina. La salida de este comando incluye los detalles de cada informe para la VM.

> [!NOTE]
> La salida puede devolver un gran volumen de datos. Se recomienda almacenar la salida en una variable.

```azurepowershell-interactive
$guestHistory = Get-AzVMGuestPolicyStatusHistory -ResourceGroupName <resourcegroupname> -VMName <vmname>
$guestHistory
```

```output
PolicyDisplayName                                                         ComplianceStatus ComplianceReasons StartTime              EndTime                VMName LatestRepor
                                                                                                                                                                  tId
-----------------                                                         ---------------- ----------------- ---------              -------                ------ -----------
[Preview]: Audit that an application is installed inside Windows VMs      NonCompliant                       02/10/2019 12:00:38 PM 02/10/2019 12:00:41 PM VM01  ../17fg0...
<truncated>
```

Para simplificar esta vista, use el parámetro **ShowChanged**. La salida de este comando solo incluye los informes que tuvieron un cambio en el estado de compatibilidad.

```azurepowershell-interactive
$guestHistory = Get-AzVMGuestPolicyStatusHistory -ResourceGroupName <resourcegroupname> -VMName <vmname> -ShowChanged
$guestHistory
```

```output
PolicyDisplayName                                                         ComplianceStatus ComplianceReasons StartTime              EndTime                VMName LatestRepor
                                                                                                                                                                  tId
-----------------                                                         ---------------- ----------------- ---------              -------                ------ -----------
Audit that an application is installed inside Windows VMs                 NonCompliant                       02/10/2019 10:00:38 PM 02/10/2019 10:00:41 PM VM01  ../12ab0...
Audit that an application is installed inside Windows VMs.                Compliant                          02/09/2019 11:00:38 AM 02/09/2019 11:00:39 AM VM01  ../e3665...
Audit that an application is installed inside Windows VMs                 NonCompliant                       02/09/2019 09:00:20 AM 02/09/2019 09:00:23 AM VM01  ../15ze1...
```

## <a name="change-history-preview"></a><a name="change-history"></a>Historial de cambios (versión preliminar)

Como parte de una nueva **versión preliminar pública**, los últimos 14 días del historial de cambios están disponibles para todos los recursos de Azure que admiten la [eliminación de modo completa](../../../azure-resource-manager/templates/complete-mode-deletion.md). El historial de cambios proporciona información acerca de cuándo se detectó un cambio y una _diferencia visual_ para cada cambio. Se desencadena una detección de cambios cuando se agregan, eliminan o modifican las propiedades de Azure Resource Manager.

1. Inicie el servicio Azure Policy en Azure Portal. Para ello, seleccione **Todos los servicios** y, a continuación, busque y seleccione **Directiva**.

1. En la página **Información general** o **Cumplimiento**, seleccione una directiva con cualquier **estado de compatibilidad**.

1. En la pestaña **Compatibilidad de recursos**, de la página **Cumplimiento de directiva**, seleccione un recurso.

1. Seleccione la pestaña **Historial de cambios (versión preliminar)** en la página **Compatibilidad de recursos**. Se muestra una lista de cambios detectados, si existe alguna.

   :::image type="content" source="../media/determine-non-compliance/change-history-tab.png" alt-text="Captura de pantalla del vínculo &quot;Ver detalles de cumplimiento&quot; en la pestaña Cumplimiento de recursos." border="false":::

1. Seleccione uno de los cambios detectados. Las _diferencias visuales_ para el recurso se presentan en la página **Historial de cambios**.

   :::image type="content" source="../media/determine-non-compliance/change-history-visual-diff.png" alt-text="Captura de pantalla del vínculo &quot;Ver detalles de cumplimiento&quot; en la pestaña Cumplimiento de recursos." border="false":::

Las _diferencias visuales_ ayudan a identificar los cambios de un recurso. Los cambios detectados pueden no estar relacionados con el estado de compatibilidad actual del recurso.

[Azure Resource Graph](../../resource-graph/overview.md) proporciona los datos del historial de cambios. Para consultar esta información fuera de Azure Portal, consulte [Obtención de cambios de recursos](../../resource-graph/how-to/get-resource-changes.md).

## <a name="next-steps"></a>Pasos siguientes

- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](get-compliance-data.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
