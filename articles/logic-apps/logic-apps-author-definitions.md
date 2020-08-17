---
title: Creación, edición o extensión de definiciones de flujo de trabajo JSON de la aplicación lógica
description: Cómo escribir, editar y ampliar las definiciones de flujo de trabajo JSON de la aplicación lógica en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 01/01/2018
ms.openlocfilehash: 9163071237041d7c8510a644c573e3763434bb0c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87060668"
---
# <a name="create-edit-or-extend-json-for-logic-app-workflow-definitions-in-azure-logic-apps"></a>Creación, edición o extensión de JSON para definiciones de flujo de trabajo de aplicaciones lógicas en Azure Logic Apps

Al crear soluciones de integración empresarial con flujos de trabajo automatizados en [Azure Logic Apps](../logic-apps/logic-apps-overview.md), las definiciones de aplicaciones lógicas subyacentes usan una notación de objetos JavaScript (JSON) sencilla y declarativa con el [esquema del lenguaje de definición de flujo de trabajo](../logic-apps/logic-apps-workflow-definition-language.md) para su descripción y validación. Estos formatos facilitan la lectura y comprensión de las definiciones de aplicaciones lógicas sin necesidad de saber mucho sobre el código.
Si desea automatizar la creación e implementación de aplicaciones lógicas, puede incluir las definiciones de aplicaciones lógicas como [recursos de Azure](../azure-resource-manager/management/overview.md) en las [plantillas de Azure Resource Manager](../azure-resource-manager/templates/overview.md).
Para crear, administrar e implementar las aplicaciones lógicas, puede usar [Azure PowerShell](/powershell/module/az.logicapp), la [CLI de Azure](../azure-resource-manager/templates/deploy-cli.md) o las [API de REST de Azure Logic Apps](/rest/api/logic/).

Para trabajar con definiciones de aplicaciones lógicas en JSON, abra el editor de la vista Código cuando trabaje en Azure Portal o en Visual Studio, o bien copie la definición en cualquier editor que desee.
Si es la primera vez que interactúa con las aplicaciones lógicas, consulte el artículo sobre [cómo crear la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

> [!NOTE]
> Algunas funcionalidades de Azure Logic Apps, como definir parámetros y varios desencadenadores en las definiciones de aplicaciones lógicas, solo están disponibles en JSON, no en el Diseñador de aplicaciones lógicas.
> Por tanto, para estas tareas, debe trabajar en la vista Código o en otro editor.

## <a name="edit-json---azure-portal"></a>Edición de JSON: Azure Portal

1. Inicie sesión en <a href="https://portal.azure.com" target="_blank">Azure Portal</a>.

2. En el menú de la izquierda, elija **Todos los servicios**.
En el cuadro de búsqueda, busque "Logic Apps" y, a continuación, en los resultados, seleccione la aplicación lógica.

3. En el menú de la aplicación lógica, en **Herramientas de desarrollo**, seleccione **Vista de código de aplicación lógica**.

   El editor de la vista Código se abre y en él se muestra la definición de la aplicación lógica en formato JSON.

## <a name="edit-json---visual-studio"></a>Edición de JSON: Visual Studio

Para poder trabajar en la definición de aplicación lógica en Visual Studio, asegúrese de haber [instalado las herramientas necesarias](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#prerequisites).
Para crear una aplicación lógica con Visual Studio, revise [Inicio rápido: Creación y automatización de tareas, procesos y flujos de trabajo con Azure Logic Apps en Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

En Visual Studio, puede abrir las aplicaciones lógicas creadas e implementadas directamente desde Azure Portal o como proyectos de Azure Resource Manager desde Visual Studio.

1. Abra la solución de Visual Studio o el proyecto del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) que contiene la aplicación lógica.

2. Busque y abra la definición de la aplicación lógica, que aparece de forma predeterminada en una [plantilla de Resource Manager](../azure-resource-manager/templates/overview.md) denominada **LogicApp.json**.
Puede usar y personalizar esta plantilla para implementarla en diferentes entornos.

3. Abra el menú contextual para la definición de la aplicación lógica o la plantilla.
Seleccione **Open with Logic App Designer** (Abrir con Diseñador de aplicación lógica).

   ![Abrir la aplicación lógica en una solución de Visual Studio](./media/logic-apps-author-definitions/open-logic-app-designer.png)

   > [!TIP]
   > Si no tiene este comando en Visual Studio 2019, compruebe que tiene las actualizaciones más recientes de Visual Studio.

4. En la parte inferior del diseñador, seleccione la **vista Código**.

   El editor de la vista Código se abre y en él se muestra la definición de la aplicación lógica en formato JSON.

5. Para volver a la vista del diseñador, en la parte inferior del editor de la vista Código, elija **Diseño**.

## <a name="parameters"></a>Parámetros

Por lo general, el ciclo de vida de una implementación tiene distintos entornos para el desarrollo, las pruebas, el almacenamiento provisional y la producción. Cuando tenga valores que desee reutilizar en la aplicación lógica sin codificar o que varían en función de sus necesidades de implementación, puede crear una [plantilla de Azure Resource Manager](../azure-resource-manager/management/overview.md) para la definición de flujo de trabajo, con el fin de que también pueda automatizar la implementación de aplicaciones lógicas.

Siga estos pasos generales para *parametrizar*, o definir y usar parámetros para esos valores en su lugar. Después, puede proporcionar los valores en un archivo de parámetros independiente que pase esos valores a la plantilla. De esa forma puede cambiar estos valores más fácilmente sin tener que actualizar y volver a implementar la aplicación lógica. Para más información, consulte [Introducción: Implementación automatizada de aplicaciones lógicas con plantillas de Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md).

1. En la plantilla, defina los parámetros de la plantilla y los parámetros de la definición del flujo de trabajo para aceptar los valores que se van a usar tanto en la implementación como en el runtime, respectivamente.

   Los parámetros de la plantilla se definen en una sección de parámetros que está fuera de la definición del flujo de trabajo, mientras que los parámetros de la definición del flujo de trabajo se definen en una sección de parámetros que está dentro de la definición del flujo de trabajo.

1. Reemplace los valores codificados por expresiones que hagan referencia a estos parámetros. Las expresiones de plantilla usan una sintaxis distinta de las expresiones de definición del flujo de trabajo.

   Evite complicar el código por la falta de uso de expresiones de plantilla, que se evalúan en la implementación, dentro de las expresiones de definición de flujo de trabajo, que se evalúan en el runtime. Use solo expresiones de plantilla fuera de la definición de flujo de trabajo. Use solo expresiones de definición de flujo de trabajo dentro de la definición de flujo de trabajo.

   Al especificar los valores de los parámetros de la definición de flujo de trabajo, puede hacer referencia a los parámetros de plantilla mediante el uso de la sección de parámetros que está fuera de la definición de flujo de trabajo, pero dentro de la definición de recurso de su aplicación lógica. De este modo, podrá pasar valores de parámetros de plantilla a los parámetros de definición de flujo de trabajo.

1. Almacene los valores de los parámetros en un [archivo de parámetros](../azure-resource-manager/templates/parameter-files.md) independiente e inclúyalo en la implementación.

## <a name="process-strings-with-functions"></a>Procesamiento de cadenas con funciones

Logic Apps tiene diversas funciones para trabajar con las cadenas.
Por ejemplo, imagine que desea pasar el nombre de una empresa de un pedido a otro sistema.
Sin embargo, no está seguro de cómo se debe controlar adecuadamente la codificación de caracteres.
Podría llevar a cabo la codificación Base64 en esta cadena, pero para evitar que aparezcan caracteres de escape en la dirección URL, puede reemplazar varios caracteres. Además, solo necesita una subcadena para el nombre de la empresa, porque no se usan los cinco primeros caracteres.

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "order": {
      "defaultValue": {
        "quantity": 10,
        "id": "myorder1",
        "companyName": "NAME=Contoso"
      },
      "type": "Object"
    }
  },
  "triggers": {
    "request": {
      "type": "Request",
      "kind": "Http"
    }
  },
  "actions": {
    "order": {
      "type": "Http",
      "inputs": {
        "method": "GET",
        "uri": "https://www.example.com/?id=@{replace(replace(base64(substring(parameters('order').companyName,5,sub(length(parameters('order').companyName), 5) )),'+','-') ,'/' ,'_' )}"
      }
    }
  },
  "outputs": {}
}
```

En estos pasos se describe cómo este ejemplo procesa esta cadena, desde adentro hacia afuera:

```
"uri": "https://www.example.com/?id=@{replace(replace(base64(substring(parameters('order').companyName,5,sub(length(parameters('order').companyName), 5) )),'+','-') ,'/' ,'_' )}"
```

1. Obtenga el valor de [`length()`](../logic-apps/logic-apps-workflow-definition-language.md) para el nombre de la empresa, con lo que se obtiene el número total de caracteres.

2. Para obtener una cadena más corta, reste `5`.

3. Ahora obtiene [`substring()`](../logic-apps/logic-apps-workflow-definition-language.md).
Comience con el índice `5` y vaya al resto de la cadena.

4. Convierta esta subcadena en una cadena [`base64()`](../logic-apps/logic-apps-workflow-definition-language.md).

5. Ahora [`replace()`](../logic-apps/logic-apps-workflow-definition-language.md) todos los caracteres `+` por caracteres `-`.

6. Por último, [`replace()`](../logic-apps/logic-apps-workflow-definition-language.md) todos los caracteres `/` por caracteres `_`.

## <a name="map-list-items-to-property-values-then-use-maps-as-parameters"></a>Asignación de los elementos de lista a valores de propiedad para luego usar los mapas como parámetros

Para obtener resultados distintos en función del valor de una propiedad, puede crear un mapa que coincida con cada valor de propiedad a un resultado y luego usar ese mapa como parámetro.

Por ejemplo, este flujo de trabajo define algunas categorías como parámetros y un mapa que coincide con dichas categorías con una dirección URL específica.
En primer lugar, el flujo de trabajo obtiene una lista de artículos. Luego, el flujo de trabajo usa el mapa para encontrar la dirección URL que coincide con la categoría de cada artículo.

*   La función [`intersection()`](../logic-apps/logic-apps-workflow-definition-language.md) comprueba si la categoría coincide con una categoría definida conocida.

*   Después de obtener una categoría coincidente, el ejemplo extrae el elemento del mapa a través de corchetes: `parameters[...]`

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "specialCategories": {
      "defaultValue": [
        "science",
        "google",
        "microsoft",
        "robots",
        "NSA"
      ],
      "type": "Array"
    },
    "destinationMap": {
      "defaultValue": {
        "science": "https://www.nasa.gov",
        "microsoft": "https://www.microsoft.com/en-us/default.aspx",
        "google": "https://www.google.com",
        "robots": "https://en.wikipedia.org/wiki/Robot",
        "NSA": "https://www.nsa.gov/"
      },
      "type": "Object"
    }
  },
  "triggers": {
    "Request": {
      "type": "Request",
      "kind": "http"
    }
  },
  "actions": {
    "getArticles": {
      "type": "Http",
      "inputs": {
        "method": "GET",
        "uri": "https://ajax.googleapis.com/ajax/services/feed/load?v=1.0&q=https://feeds.wired.com/wired/index"
      }
    },
    "forEachArticle": {
      "type": "foreach",
      "foreach": "@body('getArticles').responseData.feed.entries",
      "actions": {
        "ifGreater": {
          "type": "if",
          "expression": "@greater(length(intersection(item().categories, parameters('specialCategories'))), 0)",
          "actions": {
            "getSpecialPage": {
              "type": "Http",
              "inputs": {
                "method": "GET",
                "uri": "@parameters('destinationMap')[first(intersection(item().categories, parameters('specialCategories')))]"
              }
            }
          }
        }
      },
      "runAfter": {
        "getArticles": [
          "Succeeded"
        ]
      }
    }
  }
}
```

## <a name="get-data-with-date-functions"></a>Obtención de datos con funciones de fecha

Para obtener datos de un origen de datos que no admite *desencadenadores* de manera nativa, en su lugar puede usar las funciones de fecha para trabajar con horas y fecha.
Por ejemplo, con esta expresión se sabe cuánto demoran los pasos de este flujo de trabajo, de adentro hacia afuera:

``` json
"expression": "@less(actions('order').startTime,addseconds(utcNow(),-1))",
```

1. En la acción `order`, extraiga el valor de `startTime`.
2. Obtenga la hora actual con `utcNow()`.
3. Reste un segundo:

   [`addseconds(..., -1)`](../logic-apps/logic-apps-workflow-definition-language.md)

   Puede usar otras unidades de tiempo, como `minutes` o `hours`.

3. Ahora puede comparar estos dos valores.

   Si el primero es inferior al segundo, significa que ha transcurrido más de un segundo desde la primera vez que se realizó el pedido.

Para dar formato a las fechas, puede usar formateadores de cadena. Por ejemplo, para obtener RFC1123, use [`utcnow('r')`](../logic-apps/logic-apps-workflow-definition-language.md).
Más información sobre el [formato de fecha](../logic-apps/logic-apps-workflow-definition-language.md).

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "order": {
      "defaultValue": {
        "quantity": 10,
        "id": "myorder-id"
      },
      "type": "Object"
    }
  },
  "triggers": {
    "Request": {
      "type": "request",
      "kind": "http"
    }
  },
  "actions": {
    "order": {
      "type": "Http",
      "inputs": {
        "method": "GET",
        "uri": "https://www.example.com/?id=@{parameters('order').id}"
      }
    },
    "ifTimingWarning": {
      "type": "If",
      "expression": "@less(actions('order').startTime,addseconds(utcNow(),-1))",
      "actions": {
        "timingWarning": {
          "type": "Http",
          "inputs": {
            "method": "GET",
            "uri": "https://www.example.com/?recordLongOrderTime=@{parameters('order').id}&currentTime=@{utcNow('r')}"
          }
        }
      },
      "runAfter": {
        "order": [
          "Succeeded"
        ]
      }
    }
  },
  "outputs": {}
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Ejecución de pasos en función de una condición (instrucciones condicionales)](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [Realización de pasos en función de los diferentes valores (instrucciones switch)](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [Ejecución y repetición de pasos (bucles)](../logic-apps/logic-apps-control-flow-loops.md)
* [Ejecución o combinación de pasos en paralelo (ramas)](../logic-apps/logic-apps-control-flow-branches.md)
* [Ejecución de pasos en función del estado de las acciones agrupadas (ámbitos)](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
* Más información sobre el [esquema del lenguaje de definición de flujo de trabajo para Azure Logic Apps](../logic-apps/logic-apps-workflow-definition-language.md)
* Más información [sobre los desencadenadores y las acciones de flujo de trabajo para Logic Apps](../logic-apps/logic-apps-workflow-actions-triggers.md)
