---
title: Bloqueo de conexiones para conectores de API específicos
description: Restricción de la creación y el uso de conexiones de API en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: deli, logicappspm
ms.topic: conceptual
ms.date: 07/23/2020
ms.openlocfilehash: cccc45f182f3ae826440df8bc163080b82226c9f
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87172076"
---
# <a name="block-connections-created-by-connectors-in-azure-logic-apps"></a>Bloqueo de conexiones creadas por conectores en Azure Logic Apps

Si su organización no permite la conexión a recursos restringidos o no aprobados mediante el uso de sus conectores en Azure Logic Apps, puede bloquear la capacidad para crear y usar esas conexiones en flujos de trabajo de aplicaciones lógicas. Con [Azure Policy](../governance/policy/overview.md) puede definir y aplicar [directivas](../governance/policy/overview.md#policy-definition) que impiden la creación o el uso de conexiones para los conectores que desea bloquear. Por ejemplo, por motivos de seguridad, es posible que desee bloquear las conexiones a plataformas de medios sociales específicos u otros servicios y sistemas.

En este tema se muestra cómo configurar una directiva que bloquea conexiones específicas mediante Azure Portal, pero puede crear definiciones de directivas de otras maneras; por ejemplo, a través de la API REST de Azure, Azure PowerShell, la CLI de Azure y plantillas de Azure Resource Manager. Para más información, consulte el [Tutorial: Creación y administración de directivas para aplicar el cumplimiento](../governance/policy/tutorials/create-and-manage.md).

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar, si no tiene una suscripción a Azure.

* El identificador de referencia del conector que desea bloquear. Para obtener más información, consulte [Búsqueda del identificador de referencia del conector](#connector-reference-ID).

<a name="connector-reference-ID"></a>

## <a name="find-connector-reference-id"></a>Búsqueda del identificador de referencia del conector

Si ya tiene una aplicación lógica con la conexión que desea bloquear, siga los [pasos para Azure Portal](#connector-ID-portal). De lo contrario, siga estos pasos.

1. Visite la [lista de conectores de Logic Apps](/connectors/connector-reference/connector-reference-logicapps-connectors).

1. Busque la página de referencia del conector que desea bloquear.

   Por ejemplo, si quiere bloquear el conector de Instagram, que está en desuso, vaya a esta página:

   `https://docs.microsoft.com/connectors/instagram/`

1. En la dirección URL de la página, copie y guarde el identificador de referencia del conector al final sin la barra diagonal (`/`), por ejemplo, `instagram`.

   Más adelante, al crear la definición de directiva, usará este identificador en la instrucción de condición de la definición, por ejemplo:

   `"like": "*managedApis/instagram"`

<a name="connector-ID-portal"></a>

### <a name="azure-portal"></a>Azure portal

1. En [Azure Portal](https://portal.azure.com), busque y abra la aplicación lógica.

1. En el menú de la aplicación lógica, seleccione **Vista de código de la aplicación lógica** para ver la definición de JSON de la aplicación lógica.

   ![Apertura de "Vista de código de la aplicación lógica" para buscar el identificador del conector](./media/block-connections-connectors/code-view-connector-id.png)

1. Busque el objeto `parameters` que contiene el objeto `$connections`, que incluye un objeto `{connection-name}` para cada conexión de la aplicación lógica y especifica información sobre esa conexión:

   ```json
   {
      "parameters": {
         "$connections": {
            "value" : {
               "{connection-name}": {
                  "connectionId": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group-name}/providers/Microsoft.Web/connections/{connection-name}",
                  "connectionName": "{connection-name}",
                  "id": "/subscriptions/{Azure-subscription-ID}/providers/Microsoft.Web/locations/{Azure-region}/managedApis/{connection-name}"
               }
            }
         }
      }
   }
   ```

   Por ejemplo, para el conector de Instagram, busque el objeto `instagram`, que identifica una conexión a Instagram:

   ```json
   {
      "parameters": {
         "$connections": {
            "value" : {
               "instagram": {
                  "connectionId": "/subscriptions/xxxxxXXXXXxxxxxXXXXXxxxxxXXXXX/resourceGroups/MyLogicApp-RG/providers/Microsoft.Web/connections/instagram",
                  "connectionName": "instagram",
                  "id": "/subscriptions/xxxxxXXXXXxxxxxXXXXXxxxxxXXXXX/providers/Microsoft.Web/locations/westus/managedApis/instagram"
               }
            }
         }
      }
   }
   ```

1. Para la conexión que desea bloquear, busque la propiedad y el valor de `id`, que sigue este formato: 

   `"id": "/subscriptions/{Azure-subscription-ID}/providers/Microsoft.Web/locations/{Azure-region}/managedApis/{connection-name}"`

   Por ejemplo, a continuación se muestra la propiedad `id` y el valor de una conexión a Instagram:

   `"id": "/subscriptions/xxxxxXXXXXxxxxxXXXXXxxxxxXXXXX/providers/Microsoft.Web/locations/westus/managedApis/instagram"`

1. En el valor de la propiedad `id`, copie y guarde el identificador de referencia del conector al final, por ejemplo, `instagram`.

   Más adelante, al crear la definición de directiva, usará este identificador en la instrucción de condición de la definición, por ejemplo:

   `"like": "*managedApis/instagram"`

<a name="create-policy-connections"></a>

## <a name="create-policy-to-block-creating-connections"></a>Creación de directiva para bloquear la creación de conexiones

Para bloquear la creación de una conexión en una aplicación lógica, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). En el cuadro de búsqueda del portal, escriba `policy` y seleccione **Directiva**.

   ![En Azure Portal, búsqueda y selección de "directiva"](./media/block-connections-connectors/find-select-azure-policy.png)

1. En el menú **Directiva**, en **Creación**, seleccione **Definiciones** >  **+ Definición de directiva**.

   ![Selección de "Definiciones" > "+ Definición de directiva"](./media/block-connections-connectors/add-new-policy-definition.png)

1. En **Definición de directiva**, proporcione la información de la definición de directiva, en función de las propiedades descritas en el ejemplo:

   ![Propiedades de definición de directiva](./media/block-connections-connectors/policy-definition-create-connections-1.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Ubicación de definición** | Sí | <*Azure-subscription-name*> | La suscripción de Azure que se usará para la definición de la directiva. <p><p>1. Para buscar la suscripción, seleccione el botón de puntos suspensivos ( **...** ). <br>2. En la lista **Suscripción**, busque y seleccione su suscripción. <br>3. Cuando finalice, haga clic en **Seleccionar**. |
   | **Nombre** | Sí | <*policy-definition-name*> | Nombre que se va a usar para la definición de directiva. |
   | **Descripción** | No | <*policy-definition-name*> | Descripción de la definición de directiva. |
   | **Categoría** | Sí | **Aplicaciones lógicas** | Nombre de una categoría existente o nueva categoría de la definición de directiva. |
   | **Aplicación de directivas** | Sí | **Enabled** | Esta configuración especifica si se debe habilitar o deshabilitar la definición de directiva al guardar el trabajo. |
   ||||

1. En **REGLA DE DIRECTIVA**, el cuadro de edición JSON se rellena previamente con una plantilla de definición de directiva. Reemplace esta plantilla por su [definición de directiva](../governance/policy/concepts/definition-structure.md) en función de las propiedades descritas en la tabla siguiente y siguiendo esta sintaxis:

   ```json
   {
      "mode": "All",
      "policyRule": {
         "if": {
            "field": "Microsoft.Web/connections/api.id",
            "like": "*managedApis/{connector-name}"
         },
         "then": {
            "effect": "deny"
         }
      },
      "parameters": {}
    }
    ```

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | `mode` | `All` | Modo que determina los tipos de recursos que evalúa la directiva. <p><p>En este escenario se establece `mode` en `All`, que aplica la directiva a los grupos de recursos, las suscripciones y todos los tipos de recursos de Azure. <p><p>Para obtener más información, vea [Estructura de definición de Azure Policy: modo](../governance/policy/concepts/definition-structure.md#mode). |
   | `if` | `{condition-to-evaluate}` | La condición que determina cuándo aplicar la regla de directiva. <p><p>En este escenario, `{condition-to-evaluate}` determina si el valor de `api.id` en `Microsoft.Web/connections/api.id` coincide con `*managedApis/{connector-name}`, que especifica un valor de carácter comodín (*). <p><p>Para obtener más información, vea [Estructura de definición de Azure Policy: regla de directiva](../governance/policy/concepts/definition-structure.md#policy-rule). |
   | `field` | `Microsoft.Web/connections/api.id` | Valor `field` que se va a comparar con la condición. <p><p>En este escenario, `field` usa el [*alias*](../governance/policy/concepts/definition-structure.md#aliases), `Microsoft.Web/connections/api.id`, para tener acceso al valor de la propiedad del conector, `api.id`. |
   | `like` | `*managedApis/{connector-name}` | Operador lógico y valor que se va a usar para comparar el valor de `field`. <p><p>En este escenario, el operador `like` y el carácter comodín (*) se aseguran de que la regla funciona independientemente de la región, y la cadena, `*managedApis/{connector-name}`, es el valor que debe coincidir, donde `{connector-name}` es el identificador del conector que desea bloquear. <p><p>Por ejemplo, supongamos que desea bloquear la creación de conexiones a las plataformas o bases de datos de medios sociales: <p><p>- Twitter: `twitter` <br>- Instagram: `instagram` <br>- Facebook: `facebook` <br>- Pinterest: `pinterest` <br>- SQL Server o Azure SQL: `sql` <p><p>Para buscar estos identificadores de conector, vea [Búsqueda del identificador de referencia del conector](#connector-reference-ID) anteriormente en este tema. |
   | `then` | `{effect-to-apply}` | Efecto que se va a aplicar cuando se cumpla la condición de `if`. <p><p>En este escenario, `{effect-to-apply}` es bloquear y generar un error en una solicitud u operación que no cumpla con la directiva. <p><p>Para obtener más información, vea [Estructura de definición de Azure Policy: regla de directiva](../governance/policy/concepts/definition-structure.md#policy-rule). |
   | `effect` | `deny` | `effect` es bloquear la solicitud, lo cual sirve para crear la conexión especificada. <p><p>Para obtener más información, consulte [Comprender los efectos de Azure Policy: denegar](../governance/policy/concepts/effects.md#deny). |
   ||||

   Por ejemplo, supongamos que desea bloquear la creación de conexiones con el conector de Instagram. Esta es la definición de directiva que puede usar:

   ```json
   {
      "mode": "All",
      "policyRule": {
         "if": {
            "field": "Microsoft.Web/connections/api.id",
            "like": "*managedApis/instagram"
         },
         "then": {
            "effect": "deny"
         }
      },
      "parameters": {}
   }
   ```

   Esta es la manera en que aparece el cuadro **REGLA DE DIRECTIVA**:

   ![Regla para la definición de directiva](./media/block-connections-connectors/policy-definition-create-connections-2.png)

   Si hay varios conectores, puede agregar más condiciones, por ejemplo:

   ```json
   {
      "mode": "All",
      "policyRule": {
         "if": {
            "anyOf": [
               {
                  "field": "Microsoft.Web/connections/api.id",
                  "like": "*managedApis/instagram"
               },
               {
                  "field": "Microsoft.Web/connections/api.id",
                  "like": "*managedApis/twitter"
               },
               {
                  "field": "Microsoft.Web/connections/api.id",
                  "like": "*managedApis/facebook"
               },
               {
                  "field": "Microsoft.Web/connections/api.id",
                  "like": "*managedApis/pinterest"
               }
            ]
         },
         "then": {
            "effect": "deny"
         }
      },
      "parameters": {}
    }
    ```

1. Cuando finalice, seleccione **Guardar**. Después de guardar la definición de la directiva, Azure Policy genera y agrega más valores de propiedad a la definición de la directiva.

1. A continuación, para asignar la definición de directiva donde desea aplicar la directiva, [cree una asignación de directiva](#create-policy-assignment).

Para obtener más información sobre las definiciones de Azure Policy, consulte estos temas:

* [Definición de estructura de Policy](../governance/policy/concepts/definition-structure.md)
* [Tutorial: Creación y administración de directivas para aplicar el cumplimiento](../governance/policy/tutorials/create-and-manage.md)
* [Definiciones de directivas integradas de Azure Policy en Azure Logic Apps](../logic-apps/policy-samples.md)

<a name="create-policy-connector-usage"></a>

## <a name="create-policy-to-block-using-connections"></a>Creación de directiva para bloquear mediante conexiones

Cuando se crea una conexión dentro de una aplicación lógica, dicha conexión existe como recurso independiente de Azure. Si elimina solo la aplicación lógica, la conexión no se eliminará automáticamente y seguirá existiendo hasta que se elimine. Es posible que tenga un escenario en el que la conexión ya existe o donde tenga que crear la conexión para usarla fuera de una aplicación lógica. Todavía puede bloquear la capacidad de usar una conexión existente en una aplicación lógica mediante la creación de una directiva que impida guardar aplicaciones lógicas que tengan la conexión restringida o no aprobada.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). En el cuadro de búsqueda del portal, escriba `policy` y seleccione **Directiva**.

   ![En Azure Portal, búsqueda y selección de "directiva"](./media/block-connections-connectors/find-select-azure-policy.png)

1. En el menú **Directiva**, en **Creación**, seleccione **Definiciones** >  **+ Definición de directiva**.

   ![Selección de "Definiciones" > "+ Definición de directiva"](./media/block-connections-connectors/add-new-policy-definition.png)

1. En **Definición de directiva**, proporcione la información de la definición de directiva, en función de las propiedades descritas en el ejemplo, y siga usando Instagram como en el ejemplo:

   ![Propiedades de definición de directiva](./media/block-connections-connectors/policy-definition-using-connections-1.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Ubicación de definición** | Sí | <*Azure-subscription-name*> | Suscripción de Azure que se usará para la definición de la directiva. <p><p>1. Para buscar la suscripción, seleccione el botón de puntos suspensivos ( **...** ). <br>2. En la lista **Suscripción**, busque y seleccione su suscripción. <br>3. Cuando finalice, haga clic en **Seleccionar**. |
   | **Nombre** | Sí | <*policy-definition-name*> | Nombre que se va a usar para la definición de directiva. |
   | **Descripción** | No | <*policy-definition-name*> | Descripción de la definición de directiva. |
   | **Categoría** | Sí | **Aplicaciones lógicas** | Nombre de una categoría existente o nueva categoría de la definición de directiva. |
   | **Aplicación de directivas** | Sí | **Enabled** | Esta configuración especifica si se debe habilitar o deshabilitar la definición de directiva al guardar el trabajo. |
   ||||

1. En **REGLA DE DIRECTIVA**, el cuadro de edición JSON se rellena previamente con una plantilla de definición de directiva. Reemplace esta plantilla por su [definición de directiva](../governance/policy/concepts/definition-structure.md) en función de las propiedades descritas en la tabla siguiente y siguiendo esta sintaxis:

   ```json
   {
      "mode": "All",
      "policyRule": {
         "if": {
            "value": "[string(field('Microsoft.Logic/workflows/parameters'))]",
            "contains": "{connector-name}"
         },
         "then": {
            "effect": "deny"
         }
      },
      "parameters": {}
    }
    ```

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | `mode` | `All` | Modo que determina los tipos de recursos que evalúa la directiva. <p><p>En este escenario se establece `mode` en `All`, que aplica la directiva a los grupos de recursos, las suscripciones y todos los tipos de recursos de Azure. <p><p>Para obtener más información, vea [Estructura de definición de Azure Policy: modo](../governance/policy/concepts/definition-structure.md#mode). |
   | `if` | `{condition-to-evaluate}` | Condición que determina cuándo aplicar la regla de directiva. <p><p>En este escenario, `{condition-to-evaluate}` determina si la salida de la cadena de `[string(field('Microsoft.Logic/workflows/parameters'))]` contiene la cadena `{connector-name}`. <p><p>Para obtener más información, vea [Estructura de definición de Azure Policy: regla de directiva](../governance/policy/concepts/definition-structure.md#policy-rule). |
   | `value` | `[string(field('Microsoft.Logic/workflows/parameters'))]` | Valor que se va a comparar con la condición. <p><p>En este escenario, `value` es la salida de cadena de `[string(field('Microsoft.Logic/workflows/parameters'))]`, que convierte el objeto `$connectors` dentro del objeto `Microsoft.Logic/workflows/parameters` en una cadena. |
   | `contains` | `{connector-name}` | Operador lógico y valor que se va a usar para comparar con la propiedad `value`. <p><p>En este escenario, el operador `contains` se asegura de que la regla funciona independientemente de dónde aparezca `{connector-name}`, donde la cadena, `{connector-name}`, es el identificador del conector que desea restringir o bloquear. <p><p>Por ejemplo, supongamos que desea bloquear el uso de conexiones a las plataformas o bases de datos de medios sociales: <p><p>- Twitter: `twitter` <br>- Instagram: `instagram` <br>- Facebook: `facebook` <br>- Pinterest: `pinterest` <br>- SQL Server or Azure SQL: `sql` <p><p>Para buscar estos identificadores de conector, vea [Búsqueda del identificador de referencia del conector](#connector-reference-ID) anteriormente en este tema. |
   | `then` | `{effect-to-apply}` | Efecto que se va a aplicar cuando se cumpla la condición de `if`. <p><p>En este escenario, `{effect-to-apply}` es bloquear y generar un error en una solicitud u operación que no cumpla con la directiva. <p><p>Para obtener más información, vea [Estructura de definición de Azure Policy: regla de directiva](../governance/policy/concepts/definition-structure.md#policy-rule). |
   | `effect` | `deny` | `effect` es `deny` o bloquear la solicitud para guardar una aplicación lógica que usa la conexión especificada. <p><p>Para obtener más información, consulte [Comprender los efectos de Azure Policy: denegar](../governance/policy/concepts/effects.md#deny). |
   ||||

   Por ejemplo, supongamos que desea bloquear el almacenamiento de aplicaciones lógicas que usan conexiones a Instagram. Esta es la definición de directiva que puede usar:

   ```json
   {
      "mode": "All",
      "policyRule": {
         "if": {
            "value": "[string(field('Microsoft.Logic/workflows/parameters'))]",
            "contains": "instagram"
         },
         "then": {
            "effect": "deny"
         }
      },
      "parameters": {}
    }
    ```

   Esta es la manera en que aparece el cuadro **REGLA DE DIRECTIVA**:

   ![Regla para la definición de directiva](./media/block-connections-connectors/policy-definition-using-connections-2.png)

1. Cuando finalice, seleccione **Guardar**. Después de guardar la definición de la directiva, Azure Policy genera y agrega más valores de propiedad a la definición de la directiva.

1. A continuación, para asignar la definición de directiva donde desea aplicar la directiva, [cree una asignación de directiva](#create-policy-assignment).

Para obtener más información sobre las definiciones de Azure Policy, consulte estos temas:

* [Definición de estructura de Policy](../governance/policy/concepts/definition-structure.md)
* [Tutorial: Creación y administración de directivas para aplicar el cumplimiento](../governance/policy/tutorials/create-and-manage.md)
* [Definiciones de directivas integradas de Azure Policy en Azure Logic Apps](../logic-apps/policy-samples.md)

<a name="create-policy-assignment"></a>

## <a name="create-policy-assignment"></a>Creación de asignación de directiva

A continuación, debe asignar la definición de directiva en la que desea aplicar la directiva, por ejemplo, a un único grupo de recursos, varios grupos de recursos, un inquilino de Azure Active Directory (Azure AD) o una suscripción de Azure. Para esta tarea, siga estos pasos a fin de crear una asignación de directiva:

1. Si cerró la sesión, vuelva a iniciar sesión en [Azure Portal](https://portal.azure.com). En el cuadro de búsqueda del portal, escriba `policy` y seleccione **Directiva**.

   ![En Azure Portal, búsqueda y selección de "Directiva"](./media/block-connections-connectors/find-select-azure-policy.png)

1. En el menú **Directiva**, en **Creación**, seleccione **Asignaciones** > **Asignar directiva**.

   ![Selección de "Asignaciones" > "Asignar"](./media/block-connections-connectors/add-new-policy-assignment.png)

1. En **Aspectos básicos**, proporcione esta información para la asignación de directiva:

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Ámbito** | Sí | Recursos en los que desea aplicar la asignación de directiva. <p><p>1. Junto al cuadro **Ámbito**, seleccione el botón de puntos suspensivos ( **...** ). <br>2. En la lista **Suscripción**, seleccione la suscripción de Azure. <br>3. Opcionalmente, en la lista **Grupo de recursos**, seleccione el grupo de recursos. <br>4. Cuando finalice, haga clic en **Seleccionar**. |
   | **Exclusiones** | No | Recursos de Azure que quiera excluir de la asignación de directiva. <p><p>1. Junto al cuadro **Exclusiones**, seleccione el botón de puntos suspensivos ( **...** ). <br>2. En la lista **Recurso**, seleccione el recurso > **Agregar al ámbito seleccionado**. <br>3. Cuando finalice, seleccione **Guardar**. |
   | **Definición de directiva** | Sí | El nombre de la definición de directiva que desea asignar y aplicar. En este ejemplo se continúa con la directiva de ejemplo para bloquear conexiones a Instagram. <p><p>1. Junto al cuadro **Definición de directiva**, seleccione el botón de puntos suspensivos ( **...** ). <br>2. Busque y seleccione la definición de directiva con el filtro **Tipo** o el cuadro **Buscar**. <br>3. Cuando finalice, haga clic en **Seleccionar**. |
   | **Nombre de asignación** | Sí | Nombre que se va a usar para la asignación de directiva, si es diferente de la definición de directiva. |
   | **Id. de asignación** | Sí | Identificador generado automáticamente para la asignación de directiva. |
   | **Descripción** | No | Descripción de la asignación de directiva. |
   | **Aplicación de directivas** | Sí | Valor que habilita o deshabilita la asignación de directiva. |
   | **Asignada por** | No | Nombre de la persona que creó y aplicó la asignación de directiva. |
   ||||

   Por ejemplo, para asignar la directiva a un grupo de recursos de Azure mediante el ejemplo de Instagram:

   ![Propiedades de la asignación de directivas](./media/block-connections-connectors/policy-assignment-basics.png)

1. Seleccione **Revisar y crear** cuando haya terminado.

   Después de crear una directiva, es posible que tenga que esperar hasta 15 minutos para que surta efecto. Los cambios también pueden tener efectos similares en cuanto a desfase temporal.

1. Una vez que la directiva surta efecto, puede [probarla](#test-policy).

Para más información, consulte [Inicio rápido: Creación de una asignación de directiva para identificar recursos no compatibles](../governance/policy/assign-policy-portal.md).

<a name="test-policy"></a>

## <a name="test-the-policy"></a>Prueba de la directiva

Para probar la directiva, empiece a crear una conexión con el conector ahora restringido en el Diseñador de aplicación lógica. Siguiendo con el ejemplo de Instagram, al iniciar sesión en Instagram, se obtiene este error que hace que la aplicación lógica no pueda crear la conexión:

![Error de conexión debido a aplicación de directiva](./media/block-connections-connectors/connection-failure-message.png)

El mensaje incluye esta información:

| Descripción | Contenido |
|-------------|---------|
| Motivo del error | `"Resource 'instagram' was disallowed by policy."` |
| Nombre de asignación | `"Block Instagram connections"` |
| Id. de asignación | `"/subscriptions/xxxxxXXXXXxxxxxXXXXXxxxxxXXXXX/resourceGroups/MyLogicApp-RG/providers/Microsoft.Authorization/policyAssignments/4231890fc3bd4352acb0b673"` |
| Identificador de la definición de directiva | `"/subscriptions/xxxxxXXXXXxxxxxXXXXXxxxxxXXXXX/providers/Microsoft.Authorization/policyDefinitions/b5ddcfec-1b24-4cac-a353-360846a59f24"` |
|||

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información acerca de [Azure Policy](../governance/policy/overview.md).
