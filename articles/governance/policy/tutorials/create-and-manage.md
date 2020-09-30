---
title: 'Tutorial: Creación de directivas para aplicar el cumplimiento'
description: En este tutorial, utilizará directivas para aplicar estándares, controlar los costos, mantener la seguridad e imponer principios de diseño para toda la empresa.
ms.date: 06/15/2020
ms.topic: tutorial
ms.openlocfilehash: d8dc65d50182b5336a683c2da8e2a5d8ebb9e849
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89650076"
---
# <a name="tutorial-create-and-manage-policies-to-enforce-compliance"></a>Tutorial: Creación y administración de directivas para aplicar el cumplimiento

Comprender cómo se crean y administran las directivas en Azure es importante para mantener el cumplimiento de los estándares corporativos y los contratos de nivel de servicio. En este tutorial, aprenderá a usar Azure Policy para realizar algunas de las tareas más comunes relacionadas con la creación, la asignación y la administración de directivas en toda la organización, como, por ejemplo:

> [!div class="checklist"]
> - Asignar una directiva para aplicar una condición a los recursos que se creen en el futuro
> - Crear y asignar una definición de iniciativa para realizar un seguimiento del cumplimiento para varios recursos
> - Resolución de un recurso que no cumpla o que sea denegado
> - Implementar una nueva directiva en toda la organización

Si desea asignar una directiva para identificar el estado de cumplimiento actual de los recursos existentes, en los artículos de inicio rápido se examina cómo hacerlo.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="assign-a-policy"></a>Asignación de una directiva

El primer paso para aplicar cumplimientos con Azure Policy es asignar una definición de directiva. Una definición de directiva precisa en qué condiciones se aplica una directiva y qué efecto debe tener. En este ejemplo, asigne la definición de directiva integrada llamada _Heredar una etiqueta del grupo de recursos si falta_ para agregar la etiqueta especificada con su valor del grupo de recursos primario a los recursos nuevos o actualizados a los que les falta la etiqueta.

1. Vaya a Azure Portal para asignar directivas. Busque y seleccione **Directiva**.

   :::image type="content" source="../media/create-and-manage/search-policy.png" alt-text="Captura de pantalla de búsqueda de la directiva en la barra de búsqueda." border="false":::

1. Seleccione **Asignaciones** en el panel izquierdo de la página de Azure Policy. Una asignación es una directiva que se asignó para que se lleve a cabo dentro de un ámbito específico.

   :::image type="content" source="../media/create-and-manage/select-assignments.png" alt-text="Captura de pantalla de la selección del nodo de asignaciones en la página Información general de la directiva." border="false":::

1. Seleccione **Asignar directiva** en la parte superior de la página **Policy - Asignaciones**.

   :::image type="content" source="../media/create-and-manage/select-assign-policy.png" alt-text="Captura de pantalla de la selección del botón Asignar directiva en la página Asignaciones." border="false":::

1. En la página **Asignar directiva** y en la pestaña **Aspectos básicos**, seleccione los puntos suspensivos y seleccione una suscripción o un grupo de administración como **Ámbito**. Opcionalmente, seleccione un grupo de recursos. Un ámbito determina en qué recursos o agrupación de recursos se implementa la asignación de directiva.
   Luego elija **Seleccionar** en la parte inferior de la página **Ámbito**.

   En este ejemplo se usa la suscripción de **Contoso**. Su suscripción variará.

1. Los recursos se pueden excluir según el **ámbito**. Las **exclusiones** comienzan en un nivel inferior al nivel del **ámbito**. Las **exclusiones** son opcionales, así que déjelas en blanco por ahora.

1. Seleccione los puntos suspensivos de **Definición de directiva** para abrir la lista de definiciones disponibles. Puede filtrar el campo **Tipo** de la definición de directiva por _Integrada_ para verlas todas y leer sus descripciones.

1. Seleccione **Heredar una etiqueta del grupo de recursos si falta**. Si no la encuentra inmediatamente, escriba **heredar una etiqueta** en el cuadro de búsqueda y, después, presione ENTRAR o haga clic fuera del cuadro de búsqueda.
   Elija **Seleccionar** en la parte inferior de la página **Definiciones disponibles** después de encontrar y seleccionar la definición de directiva.

   :::image type="content" source="../media/create-and-manage/select-available-definition.png" alt-text="Captura de pantalla del filtro de búsqueda al seleccionar una definición de directiva.":::

1. **Nombre de asignación** se rellena automáticamente con el nombre de directiva seleccionado, pero puede cambiarlo. En este ejemplo, deje _Heredar una etiqueta del grupo de recursos si falta_. También puede agregar una **Descripción** opcional. La descripción ofrece detalles sobre esta asignación de directiva.

1. Deje **Cumplimiento de directivas** como _Habilitado_. Cuando esta _Deshabilitado_, esta configuración permite probar el resultado de la directiva sin desencadenar el efecto. Para más información, consulte [Modo de cumplimiento](../concepts/assignment-structure.md#enforcement-mode).

1. **Asignado por**: se rellena automáticamente en función de quién ha iniciado sesión. Este campo es opcional, así que se pueden especificar valores personalizados.

1. Seleccione la pestaña **Parámetros** en la parte superior del asistente.

1. En **Nombre de etiqueta**, escriba _Entorno_.

1. Seleccione la pestaña **Corrección** en la parte superior del asistente.

1. Deje sin marcar **Crear una tarea de corrección**. Este cuadro permite crear una tarea para modificar los recursos existentes, además de los recursos nuevos o actualizados. Para más información, consulte [Corrección de recursos](../how-to/remediate-resources.md).

1. La casilla **Crear una identidad administrada** se activa automáticamente, ya que esta definición de directiva usa el efecto [modify](../concepts/effects.md#modify). **Permisos** se establece automáticamente en _Colaborador_ según la definición de directiva. Para más información, consulte las [identidades administradas](../../../active-directory/managed-identities-azure-resources/overview.md) y [cómo funciona la seguridad de corrección](../how-to/remediate-resources.md#how-remediation-security-works).

1. Seleccione la pestaña **Revisar y crear** en la parte superior del asistente.

1. Revise las selecciones y, luego, seleccione **Crear** en la parte inferior de la página.

## <a name="implement-a-new-custom-policy"></a>Implementación de una nueva directiva personalizada

Ahora que ha asignado una definición de directiva integrada, puede hacer más cosas con Azure Policy. A continuación, creará una directiva personalizada para ahorrar costos y se asegurará de que las máquinas virtuales creadas en el entorno no puedan estar en la serie G. De este modo, cada vez que un usuario de su organización intente crear una máquina virtual de la serie G, la solicitud le será denegada.

1. Seleccione **Definiciones** en **Creación** en el lado izquierdo de la página de Azure Policy.

   :::image type="content" source="../media/create-and-manage/definition-under-authoring.png" alt-text="Captura de pantalla de la página Definiciones en el grupo de creación." border="false":::

1. Seleccione **+ Definición de directiva** en la parte superior de la página. Este botón abre la página **Definición de directiva**.

1. Escriba la siguiente información:

   - El grupo de administración o la suscripción donde está guardada la definición de directiva. Para la selección use los puntos suspensivos en **Ubicación de definición**.

     > [!NOTE]
     > Si planea aplicar esta definición de directiva a varias suscripciones, la ubicación debe ser un grupo de administración que contenga las suscripciones a las que se asigne la directiva. Lo mismo puede aplicarse a una definición de iniciativa.

   - El nombre de la definición de directiva: _Require VM SKUs not in the G series_ (Requerir SKU de máquina virtual que no sean de la serie G)
   - La descripción de la finalidad de la definición de directiva: _esta definición de directiva exige que todas las máquinas virtuales creadas en este ámbito tengan SKU que no sean de la serie G para reducir el costo._
   - Elija entre las opciones existentes o cree una nueva categoría (como _Proceso_) para esta definición de directiva.
   - Copie el siguiente código JSON y actualícelo según sus necesidades con:
      - Los parámetros de directiva.
      - Las reglas/condiciones de la directiva, en este caso: tamaño de SKU de máquina virtual igual a la serie G
      - El efecto de la directiva, en este caso: **Deny** (Denegar).

   Este es el aspecto que debería tener el JSON. Pegue el código revisado en Azure Portal.

   ```json
   {
       "policyRule": {
           "if": {
               "allOf": [{
                       "field": "type",
                       "equals": "Microsoft.Compute/virtualMachines"
                   },
                   {
                       "field": "Microsoft.Compute/virtualMachines/sku.name",
                       "like": "Standard_G*"
                   }
               ]
           },
           "then": {
               "effect": "deny"
           }
       }
   }
   ```

   La propiedad _field_ de la regla de directiva debe ser un valor admitido. Se puede encontrar una lista completa de valores en los [campos de estructura de definición de directiva](../concepts/definition-structure.md#fields). Un ejemplo de un alias podría ser `"Microsoft.Compute/VirtualMachines/Size"`.

   Para ver más ejemplos de Azure Policy, consulte [Ejemplos de Azure Policy](../samples/index.md).

1. Seleccione **Guardar**.

## <a name="create-a-policy-definition-with-rest-api"></a>Creación de una definición de directiva con la API de REST

Puede crear una directiva con la API REST para definiciones de Azure Policy. La API de REST permite crear y eliminar definiciones de directiva, así como recuperar información sobre las definiciones existentes. Para crear una definición de directiva, use el siguiente ejemplo:

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.authorization/policydefinitions/{policyDefinitionName}?api-version={api-version}
```

Incluya un cuerpo de solicitud similar al ejemplo siguiente:

```json
{
    "properties": {
        "parameters": {
            "allowedLocations": {
                "type": "array",
                "metadata": {
                    "description": "The list of locations that can be specified when deploying resources",
                    "strongType": "location",
                    "displayName": "Allowed locations"
                }
            }
        },
        "displayName": "Allowed locations",
        "description": "This policy enables you to restrict the locations your organization can specify when deploying resources.",
        "policyRule": {
            "if": {
                "not": {
                    "field": "location",
                    "in": "[parameters('allowedLocations')]"
                }
            },
            "then": {
                "effect": "deny"
            }
        }
    }
}
```

## <a name="create-a-policy-definition-with-powershell"></a>Creación de una definición de directiva con PowerShell

Antes de continuar con el ejemplo de PowerShell, asegúrese de que tiene instalada la última versión del módulo Az de Azure PowerShell.

Puede crear una definición de directiva con el cmdlet `New-AzPolicyDefinition`.

Para crear una definición de directiva desde un archivo, pase la ruta de acceso al archivo. Para un archivo externo, use el ejemplo siguiente:

```azurepowershell-interactive
$definition = New-AzPolicyDefinition `
    -Name 'denyCoolTiering' `
    -DisplayName 'Deny cool access tiering for storage' `
    -Policy 'https://raw.githubusercontent.com/Azure/azure-policy-samples/master/samples/Storage/storage-account-access-tier/azurepolicy.rules.json'
```

Para un archivo local, use el ejemplo siguiente:

```azurepowershell-interactive
$definition = New-AzPolicyDefinition `
    -Name 'denyCoolTiering' `
    -Description 'Deny cool access tiering for storage' `
    -Policy 'c:\policies\coolAccessTier.json'
```

Para crear una definición de directiva con una regla insertada, use el ejemplo siguiente:

```azurepowershell-interactive
$definition = New-AzPolicyDefinition -Name 'denyCoolTiering' -Description 'Deny cool access tiering for storage' -Policy '{
    "if": {
        "allOf": [{
                "field": "type",
                "equals": "Microsoft.Storage/storageAccounts"
            },
            {
                "field": "kind",
                "equals": "BlobStorage"
            },
            {
                "field": "Microsoft.Storage/storageAccounts/accessTier",
                "equals": "cool"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}'
```

La salida se almacena en un objeto `$definition`, que se usa durante la asignación de directivas. En el ejemplo siguiente se crea una definición de directiva que incluye parámetros:

```azurepowershell-interactive
$policy = '{
    "if": {
        "allOf": [{
                "field": "type",
                "equals": "Microsoft.Storage/storageAccounts"
            },
            {
                "not": {
                    "field": "location",
                    "in": "[parameters(''allowedLocations'')]"
                }
            }
        ]
    },
    "then": {
        "effect": "Deny"
    }
}'

$parameters = '{
    "allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of locations that can be specified when deploying storage accounts.",
            "strongType": "location",
            "displayName": "Allowed locations"
        }
    }
}'

$definition = New-AzPolicyDefinition -Name 'storageLocations' -Description 'Policy to specify locations for storage accounts.' -Policy $policy -Parameter $parameters
```

### <a name="view-policy-definitions-with-powershell"></a>Visualización de definiciones de directiva con PowerShell

Para ver todas las definiciones de directiva en su suscripción, utilice el siguiente comando:

```azurepowershell-interactive
Get-AzPolicyDefinition
```

Devuelve todas las definiciones de directiva disponibles, incluidas las directivas integradas. Cada directiva se devuelve con el formato siguiente:

```output
Name               : e56962a6-4747-49cd-b67b-bf8b01975c4c
ResourceId         : /providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c
ResourceName       : e56962a6-4747-49cd-b67b-bf8b01975c4c
ResourceType       : Microsoft.Authorization/policyDefinitions
Properties         : @{displayName=Allowed locations; policyType=BuiltIn; description=This policy enables you to
                     restrict the locations your organization can specify when deploying resources. Use to enforce
                     your geo-compliance requirements.; parameters=; policyRule=}
PolicyDefinitionId : /providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c
```

## <a name="create-a-policy-definition-with-azure-cli"></a>Creación de una definición de directiva con la CLI de Azure

Puede crear una definición de directiva mediante la CLI de Azure con el comando `az policy definition`. Para crear una definición de directiva con una regla insertada, use el ejemplo siguiente:

```azurecli-interactive
az policy definition create --name 'denyCoolTiering' --description 'Deny cool access tiering for storage' --rules '{
    "if": {
        "allOf": [{
                "field": "type",
                "equals": "Microsoft.Storage/storageAccounts"
            },
            {
                "field": "kind",
                "equals": "BlobStorage"
            },
            {
                "field": "Microsoft.Storage/storageAccounts/accessTier",
                "equals": "cool"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}'
```

### <a name="view-policy-definitions-with-azure-cli"></a>Visualización de definiciones de directiva con la CLI de Azure

Para ver todas las definiciones de directiva en su suscripción, utilice el siguiente comando:

```azurecli-interactive
az policy definition list
```

Devuelve todas las definiciones de directiva disponibles, incluidas las directivas integradas. Cada directiva se devuelve con el formato siguiente:

```json
{
    "description": "This policy enables you to restrict the locations your organization can specify when deploying resources. Use to enforce your geo-compliance requirements.",
    "displayName": "Allowed locations",
    "id": "/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c",
    "name": "e56962a6-4747-49cd-b67b-bf8b01975c4c",
    "policyRule": {
        "if": {
            "not": {
                "field": "location",
                "in": "[parameters('listOfAllowedLocations')]"
            }
        },
        "then": {
            "effect": "Deny"
        }
    },
    "policyType": "BuiltIn"
}
```

## <a name="create-and-assign-an-initiative-definition"></a>Creación y asignación de una definición de iniciativa

Con una definición de iniciativa, puede agrupar varias definiciones de directiva para lograr un objetivo general. Una iniciativa evalúa los recursos dentro del ámbito de la asignación de cara al cumplimiento de las directivas incluidas. Para más información sobre las definiciones de iniciativa, consulte [Introducción a Azure Policy](../overview.md).

### <a name="create-an-initiative-definition"></a>Creación de una definición de iniciativa

1. Seleccione **Definiciones** en **Creación** en el lado izquierdo de la página de Azure Policy.

   :::image type="content" source="../media/create-and-manage/definition-under-authoring.png" alt-text="Captura de pantalla de la página Definiciones en el grupo de creación." border="false":::

1. Seleccione **+ Initiative Definition** (+ Definición de iniciativa) en la parte superior de la página para abrir la página **Definición de iniciativa** (Definición de iniciativa).

   :::image type="content" source="../media/create-and-manage/initiative-definition.png" alt-text="Captura de pantalla de la página de definición de iniciativa y las propiedades que se van a establecer." border="false":::

1. Use **Ubicación de definición** para seleccionar un grupo de administración o una suscripción para almacenar la definición. Si ha establecido el ámbito de la página anterior a un único grupo de administración o suscripción, **Ubicación de definición** se rellena automáticamente. Una vez seleccionado, se rellena el campo **Definiciones disponibles**.

1. Defina los campos **Nombre** y **Descripción** de la iniciativa.

   En este ejemplo se garantiza que los recursos guardan conformidad con las definiciones de directiva sobre protección. Denomine a la iniciativa **Get Secure** (Estar protegido) y la descripción sería: **This initiative has been created to handle all policy definitions associated with securing resources** (Esta iniciativa se ha creado para administrar todas las definiciones de directiva asociadas con la protección de los recursos).

1. En **Categoría**, elija entre las opciones existentes o cree una nueva categoría.

1. Examine la lista **Definiciones disponibles** [mitad derecha de la página **Initiative definition** (definición de iniciativa)] y seleccione las definiciones de directiva que le gustaría agregar a esta iniciativa. En el caso de la iniciativa **Get Secure** (Estar protegido), para agregar las siguientes definiciones de directiva integradas, seleccione **+** junto a la información de definición de directiva o seleccione una fila de definición de directiva y, luego, la opción **+ Agregar** de la página de detalles:

   - Ubicaciones permitidas
   - Supervisión de la falta de Endpoint Protection en Azure Security Center
   - Se deben proteger las reglas del grupo de seguridad de red para máquinas virtuales accesibles desde Internet
   - Azure Backup debe estar habilitado para Virtual Machines.
   - El cifrado de discos debe aplicarse en máquinas virtuales

   Después de seleccionar la definición de directiva de la lista, se agrega cada una debajo de **Categoría**.

   :::image type="content" source="../media/create-and-manage/initiative-definition-2.png" alt-text="Captura de pantalla de los parámetros y definiciones de valor en la página de definición de iniciativa." border="false":::

1. Si una definición de directiva que se va a agregar a la iniciativa tiene parámetros, se muestran bajo el nombre de la directiva en el área **Categoría**. El _valor_ puede establecerse en "Establecer valor" (de forma rígida para todas las asignaciones de esta iniciativa) o en "Utilice el parámetro de iniciativa" (establecido durante cada asignación de iniciativa). Si se selecciona "Establecer valor", la lista desplegable situada a la derecha de _Valores_ permite escribir o seleccionar los valores. Si se selecciona "Utilice el parámetro de iniciativa", se muestra una nueva sección llamada **Parámetros de iniciativa** que permite definir el parámetro que se va a establecer durante la asignación de iniciativa. Los valores permitidos en este parámetro de iniciativa pueden restringir aún más lo que puede establecerse durante la asignación de iniciativa.

   :::image type="content" source="../media/create-and-manage/initiative-definition-3.png" alt-text="Captura de pantalla de las opciones de los valores permitidos para un parámetro de efecto en la página definición de iniciativa." border="false":::

   > [!NOTE]
   > En el caso de algunos parámetros `strongType`, la lista de valores no se puede determinar automáticamente. En estos casos, se muestran puntos suspensivos a la derecha de la fila de parámetros. Al seleccionarlos se abre la página "Ámbito de parámetro (&lt;nombre del parámetro&gt;)". En esta página, seleccione la suscripción que se usará para proporcionar las opciones de valor. Este ámbito de parámetro solo se usa durante la creación de la definición de iniciativa y no influye en la evaluación de directivas ni en el ámbito de la iniciativa cuando se asigna.

   Establezca el parámetro "Ubicaciones permitidas" en "Este de EE. UU. 2" y deje el resto con el valor predeterminado "AuditifNotExists".

1. Seleccione **Guardar**.

#### <a name="create-a-policy-initiative-definition-with-azure-cli"></a>Creación de una definición de iniciativa de directiva con la CLI de Azure

Puede crear una definición de iniciativa de directiva mediante la CLI de Azure con el comando `az policy set-definition`. Para crear una definición de iniciativa de directiva con una definición de directiva existente, use el ejemplo siguiente:

```azurecli-interactive
az policy set-definition create -n readOnlyStorage --definitions '[
        {
            "policyDefinitionId": "/subscriptions/mySubId/providers/Microsoft.Authorization/policyDefinitions/storagePolicy",
            "parameters": { "storageSku": { "value": "[parameters(\"requiredSku\")]" } }
        }
    ]' \
    --params '{ "requiredSku": { "type": "String" } }'
```

#### <a name="create-a-policy-initiative-definition-with-azure-powershell"></a>Creación de una definición de iniciativa de directiva con Azure PowerShell

Puede crear una definición de iniciativa de directiva mediante Azure PowerShell con el cmdlet `New-AzPolicySetDefinition`. Para crear una definición de iniciativa de directiva con una definición de directiva existente, use el siguiente archivo de definición de iniciativa de directiva como `VMPolicySet.json`:

```json
[
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
        "parameters": {
            "tagName": {
                "value": "Business Unit"
            },
            "tagValue": {
                "value": "Finance"
            }
        }
    },
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/464dbb85-3d5f-4a1d-bb09-95a9b5dd19cf"
    }
]
```

```azurepowershell-interactive
New-AzPolicySetDefinition -Name 'VMPolicySetDefinition' -Metadata '{"category":"Virtual Machine"}' -PolicyDefinition C:\VMPolicySet.json
```

### <a name="assign-an-initiative-definition"></a>Asignación de una definición de iniciativa

1. Seleccione **Definiciones** en **Creación** en el lado izquierdo de la página de Azure Policy.

1. Busque la definición de directiva **Get Secure** (Estar protegido) que creó anteriormente y selecciónela. Seleccione **Asignar** en la parte superior de la página para abrir la página **Get Secure: Assign initiative** (Estar protegido: asignar iniciativa).

   :::image type="content" source="../media/create-and-manage/assign-definition.png" alt-text="Captura de pantalla del botón Asignar en la página de definición de iniciativa." border="false":::

   También puede hacer clic con el botón derecho en la fila seleccionada o seleccionar los puntos suspensivos al final de la fila de un menú contextual. A continuación, seleccione **Asignar**.

   :::image type="content" source="../media/create-and-manage/select-right-click.png" alt-text="Captura de pantalla del menú contextual de una iniciativa para seleccionar la funcionalidad de asignación." border="false":::

1. Rellene la página **Get Secure: Assign Initiative** (Estar protegido: asignar iniciativa) con la siguiente información de ejemplo. Puede usar su propia información.

   - Ámbito: el grupo de administración o la suscripción donde guardó la iniciativa se convertirán en predeterminados.
     También puede cambiar ámbito para asignar la iniciativa a un grupo de recursos o una suscripción de la ubicación de almacenamiento.
   - Exclusiones: configure los recursos del ámbito para evitar que se les aplique la asignación de iniciativa.
   - Nombre de la asignación y definición de la iniciativa: Get Secure (Estar protegido) (rellenado previamente como nombre de la iniciativa que se va a asignar).
   - Descripción: esta asignación de iniciativa está adaptada para aplicar este grupo de definiciones de directiva.
   - Aplicación de directivas: Deje el valor predeterminado _Habilitado_.
   - Asignada por: se rellena automáticamente en función de quién ha iniciado sesión. Este campo es opcional, así que se pueden especificar valores personalizados.

1. Seleccione la pestaña **Parámetros** en la parte superior del asistente. Si configuró un parámetro de iniciativa en pasos anteriores, establezca un valor aquí.

1. Seleccione la pestaña **Corrección** en la parte superior del asistente. Deje desactivada la casilla **Crear una identidad administrada**. Esta casilla se _debe_ activar cuando la directiva o la iniciativa que se asigna incluye una directiva con los efectos [deployIfNotExists](../concepts/effects.md#deployifnotexists) o [modify](../concepts/effects.md#modify). Como no es el caso de la directiva usada en este tutorial, déjela en blanco. Para más información, consulte las [identidades administradas](../../../active-directory/managed-identities-azure-resources/overview.md) y [cómo funciona la seguridad de corrección](../how-to/remediate-resources.md#how-remediation-security-works).

1. Seleccione la pestaña **Revisar y crear** en la parte superior del asistente.

1. Revise las selecciones y, luego, seleccione **Crear** en la parte inferior de la página.

## <a name="check-initial-compliance"></a>Comprobación del cumplimiento inicial

1. Seleccione **Cumplimiento** en el panel izquierdo de la página de Azure Policy.

1. Busque la iniciativa **Get Secure** (Estar protegido). Es probable que _Estado de compatibilidad_ se encuentre aún como **No iniciado**.
   Seleccione la iniciativa para obtener información detallada sobre el progreso de la asignación.

   :::image type="content" source="../media/create-and-manage/compliance-status-not-started.png" alt-text="Captura de pantalla de la página de cumplimiento de la iniciativa que muestra las evaluaciones de asignación en un estado no iniciado." border="false":::

1. Una vez que se haya completado la asignación de la iniciativa, la página de conformidad se actualiza con el _Estado de compatibilidad_**Conforme**.

   :::image type="content" source="../media/create-and-manage/compliance-status-compliant.png" alt-text="Captura de pantalla de la página de cumplimiento de la iniciativa que muestra las evaluaciones de asignación en un estado conforme." border="false":::

1. Al seleccionar cualquier directiva de la página de cumplimiento de la iniciativa se abre la página de detalles de cumplimiento de esa directiva. Esta página proporciona los detalles de cumplimiento a nivel de recurso.

## <a name="exempt-a-non-compliant-or-denied-resource-using-exclusion"></a>Exclusión de un recurso no conforme o denegado mediante Exclusión

Después de asignar una iniciativa de directiva para exigir una ubicación específica, cualquier otro recurso creado en una ubicación diferente se deniega. En esta sección se va a examinar cómo resolver una solicitud denegada para crear un recurso mediante la creación de una exclusión de un único grupo de recursos. La exclusión impide el cumplimiento de la directiva (o iniciativa) en ese grupo de recursos. En el ejemplo siguiente, se permite cualquier ubicación en el grupo de recursos excluido. Una exclusión se puede aplicar a una suscripción, a un grupo de recursos o a recursos individuales.

Las implementaciones que ha impedido una directiva o iniciativa asignada se pueden ver en el grupo de recursos de destino de la implementación: seleccione **Implementaciones** en el lado izquierdo de la página y luego seleccione el valor de **Nombre de la implementación** de la implementación con errores. El recurso denegado se muestra con el estado _Prohibido_. Para determinar la directiva o iniciativa y la asignación que denegaron el recurso, seleccione **Failed. Click here for details ->** (Error. Haga clic aquí para ver los detalles ->) en la página de información general de la implementación. Una ventana se abre en el lado derecho de la página con la información sobre el error. En **Detalles del error** se muestran los identificadores únicos globales de los objetos de directiva relacionados.

:::image type="content" source="../media/create-and-manage/rg-deployment-denied.png" alt-text="Captura de pantalla de una implementación con errores denegada por una asignación de directiva." border="false":::

En la página de Azure Policy: Seleccione **Cumplimiento** en el lado izquierdo de la página y seleccione la iniciativa **Get Secure** (Estar protegido). En esta página, hay un aumento en el recuento de **denegaciones** para los recursos bloqueados. En la pestaña **Eventos** se muestran detalles sobre quién intentó crear o implementar el recurso que se denegó en la definición de directiva.

:::image type="content" source="../media/create-and-manage/compliance-overview.png" alt-text="Captura de pantalla de la pestaña Eventos y detalles del evento de directiva en la página cumplimiento de la iniciativa." border="false":::

En este ejemplo, Trent Baker, uno de los especialistas en virtualización sénior de Contoso, estaba haciendo el trabajo requerido. Es necesario conceder a Trent un espacio para una excepción. Cree el grupo de recursos **LocationsExcluded** y, luego, concédale una excepción a esta asignación de directiva.

### <a name="update-assignment-with-exclusion"></a>Actualización de la asignación con la exclusión

1. Seleccione **Asignaciones** en **Creación** en el lado izquierdo de la página de Azure Policy.

1. Examine todas las asignaciones de directiva y abra la que se llama _Get Secure_ (Estar protegido).

1. Para establecer el valor de **Exclusión**, seleccione los puntos suspensivos y seleccione el grupo de recursos que quiere excluir; en este ejemplo, _LocationsExcluded_. Seleccione **Agregar al ámbito seleccionado** y, luego, seleccione **Guardar**.

   :::image type="content" source="../media/create-and-manage/request-exclusion.png" alt-text="Captura de pantalla de la opción Exclusiones de la página de asignación de la iniciativa para agregar un grupo de recursos excluidos a la asignación de directiva." border="false":::

   > [!NOTE]
   > Dependiendo de la definición de directiva y de su efecto, es posible que la exclusión también se conceda a recursos específicos dentro de un grupo de recursos en el ámbito de la asignación. Dado que se usó un efecto **deny** en este tutorial, no tendría sentido establecer la exclusión en un recurso concreto que ya existe.

1. Seleccione **Revisar y guardar** y, luego, elija **Guardar**.

En esta sección, resolvió la solicitud denegada creando una exclusión en un único grupo de recursos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si terminó de trabajar con los recursos de este tutorial, use los pasos siguientes para eliminar todas las asignaciones o definiciones de directiva creadas anteriormente:

1. Seleccione **Definiciones** (o **Asignaciones** si trata de eliminar una asignación) en **Creación** en el lado izquierdo de la página de Azure Policy.

1. Busque la nueva definición de iniciativa o directiva (o asignación) que quiere quitar.

1. Haga clic con el botón derecho en la fila o seleccione los puntos suspensivos al final de la definición (o asignación) y elija **Eliminar definición** (o **Eliminar asignación** ).

## <a name="review"></a>Revisar

En este tutorial, ha realizado correctamente las tareas siguientes:

> [!div class="checklist"]
> - Ha asignado una directiva para aplicar una condición a los recursos que se creen en el futuro
> - Ha creado y asignado una definición de iniciativa para realizar un seguimiento del cumplimiento para varios recursos
> - Ha resuelto un recurso que no cumplía o que se había denegado
> - Ha implementado una nueva directiva en toda la organización

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las estructuras de las definiciones de directiva, consulte este artículo:

> [!div class="nextstepaction"]
> [Estructura de definición de Azure Policy](../concepts/definition-structure.md)
