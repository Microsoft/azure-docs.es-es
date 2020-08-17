---
title: Conversión de JSON y XML con plantillas de Liquid
description: Transformación de JSON y XML usando plantillas de Liquid como asignaciones en Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, daviburg, logicappspm
ms.topic: article
ms.date: 07/31/2020
ms.openlocfilehash: 5aa6b3717925146607f3785ad5ea5fb940e8c236
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503412"
---
# <a name="transform-json-and-xml-using-liquid-templates-as-maps-in-azure-logic-apps"></a>Transformación de JSON y XML usando plantillas de Liquid como asignaciones en Azure Logic Apps

Cuando quiera realizar transformaciones JSON básicas en las aplicaciones lógicas, puede usar [operaciones de datos](../logic-apps/logic-apps-perform-data-operations.md) nativas, como **Compose** (Redactar) o **Parse JSON** (Analizar JSON). En las transformaciones de JSON a JSON avanzadas y complejas que tienen elementos, como iteraciones, flujos de control y variables, cree y use plantillas que describan estas transformaciones mediante [Liquid](https://shopify.github.io/liquid/), el lenguaje de plantilla de código abierto. También puede [realizar otras transformaciones](#other-transformations), por ejemplo, JSON a texto, XML a JSON y XML a texto.

Antes de poder realizar una transformación Liquid en la aplicación lógica, primero debe crear una plantilla de Liquid que defina la asignación que quiere. Luego, [cargue la plantilla como un asignación](../logic-apps/logic-apps-enterprise-integration-maps.md) en la [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md). Al agregar la acción **Transform JSON to JSON - Liquid** (Transformar de JSON a JSON: Liquid) a la aplicación lógica, puede seleccionar la plantilla de Liquid como asignación de la acción que se va a usar.

En este artículo se muestra cómo completar estas tareas:

* Crear una plantilla de Liquid.
* Agregar la plantilla a la cuenta de integración.
* Agregar la acción de transformación Liquid a la aplicación lógica.
* Seleccionar la plantilla como la asignación que se quiere usar.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* Una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md).

* Conocimientos básicos sobre el [lenguaje de plantilla de Liquid](https://shopify.github.io/liquid/)

  > [!NOTE]
  > La acción **Transform JSON to JSON - Liquid** (Transformar de JSON a JSON: Liquid) sigue la [implementación DotLiquid para Liquid](https://github.com/dotliquid/dotliquid), que, en determinados casos, difiere de la [implementación Shopify para Liquid](https://shopify.github.io/liquid). Para más información, consulte [Consideraciones sobre las plantillas de Liquid](#template-considerations).

## <a name="create-the-template"></a>Creación de la plantilla

1. Cree la plantilla de Liquid que va a usar como asignación de la transformación JSON. Puede usar la herramienta de edición que prefiera.

   En este ejemplo, cree la plantilla de Liquid de ejemplo que se describe en esta sección:

   ```json
   {%- assign deviceList = content.devices | Split: ', ' -%}

   {
      "fullName": "{{content.firstName | Append: ' ' | Append: content.lastName}}",
      "firstNameUpperCase": "{{content.firstName | Upcase}}",
      "phoneAreaCode": "{{content.phone | Slice: 1, 3}}",
      "devices" : [
         {%- for device in deviceList -%}
            {%- if forloop.Last == true -%}
            "{{device}}"
            {%- else -%}
            "{{device}}",
            {%- endif -%}
         {%- endfor -%}
      ]
   }
   ```

1. Guarde la plantilla con la extensión `.liquid`. En este ejemplo se usa `SimpleJsonToJsonTemplate.liquid`.

## <a name="upload-the-template"></a>Carga de la plantilla

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure.

1. En el cuadro de búsqueda de Azure Portal, escriba `integration accounts` y seleccione **Cuentas de integración**.

   ![Buscar "cuentas de integración"](./media/logic-apps-enterprise-integration-liquid-transform/find-integration-accounts.png)

1. Busque y seleccione su cuenta de integración.

   ![Seleccionar cuenta de integración](./media/logic-apps-enterprise-integration-liquid-transform/select-integration-account.png)

1. En el panel **Información general**, en **Componentes**, seleccione **Asignaciones**.

    ![Selección del icono "Asignaciones"](./media/logic-apps-enterprise-integration-liquid-transform/select-maps-tile.png)

1. En el panel **Asignaciones**, seleccione **Agregar** y proporcione estos detalles para la asignación:

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **Nombre** | `JsonToJsonTemplate` | Nombre de la asignación, que es "JsonToJsonTemplate" en este ejemplo |
   | **Tipo de asignación** | **liquid** | Tipo de la asignación. Para la transformación de JSON a JSON, debe seleccionar **liquid**. |
   | **Map** | `SimpleJsonToJsonTemplate.liquid` | Archivo de asignación o plantilla de Liquid existente para su uso en la transformación, "SimpleJsonToJsonTemplate.liquid" en este ejemplo. Puede usar el selector de archivos para buscar el archivo. Para conocer los límites de tamaño de las asignaciones, consulte [Límites y configuración](../logic-apps/logic-apps-limits-and-config.md#artifact-capacity-limits). |
   |||

   ![Adición de la plantilla de Liquid](./media/logic-apps-enterprise-integration-liquid-transform/add-liquid-template.png)

## <a name="add-the-liquid-transformation-action"></a>Adición de la acción de transformación de Liquid

1. En Azure Portal, siga estos pasos para [crear una aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md).

1. En el Diseñador de aplicaciones lógicas, agregue el [desencadenador de solicitud](../connectors/connectors-native-reqres.md#add-request) a la aplicación lógica.

1. En el desencadenador, elija **New step** (Nuevo paso). En el cuadro de búsqueda, escriba `liquid` como filtro y seleccione esta acción: **Transformar de JSON a JSON - Liquid**

   ![Búsqueda y selección de la acción Liquid](./media/logic-apps-enterprise-integration-liquid-transform/search-action-liquid.png)

1. Abra la lista **Asignación** y seleccione la plantilla de Liquid, que, en este ejemplo, es "JsonToJsonTemplate".

   ![Selección de asignación](./media/logic-apps-enterprise-integration-liquid-transform/select-map.png)

   Si la lista de asignaciones está vacía, es probable que la aplicación lógica no esté vinculada a la cuenta de integración. 
   Para vincular la aplicación lógica a la cuenta de integración que tiene la plantilla o asignación Liquid, siga estos pasos:

   1. En el menú de la aplicación lógica, seleccione **Configuración del flujo de trabajo**.

   1. En la lista **Seleccione una cuenta de integración**, seleccione su cuenta de integración y seleccione **Guardar**.

      ![Vincular la aplicación lógica a la cuenta de integración](./media/logic-apps-enterprise-integration-liquid-transform/link-integration-account.png)

1. Ahora, agregue la propiedad **Content** a esta acción. Abra la lista **Agregar nuevo parámetro** y seleccione **Content**.

   ![Adición de la propiedad "Content" a la acción](./media/logic-apps-enterprise-integration-liquid-transform/add-content-property-to-action.png)

1. Para establecer el valor de la propiedad **Content**, haga clic dentro del cuadro **Content** para que aparezca la lista de contenido dinámico. Seleccione el token **Body**, que representa la salida del contenido del cuerpo del desencadenador.

   ![Selección del token "Body" para el valor de la propiedad "Content"](./media/logic-apps-enterprise-integration-liquid-transform/select-body.png)

   Cuando haya terminado, la acción debe ser parecida a la de este ejemplo:

   ![Acción "Transformar de JSON a JSON" finalizada](./media/logic-apps-enterprise-integration-liquid-transform/finished-transform-action.png)

## <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

Publique la entrada JSON en la aplicación lógica mediante [Postman](https://www.getpostman.com/postman) o una herramienta parecida. La salida JSON transformada de la aplicación lógica tiene este aspecto:

![Salida de ejemplo](./media/logic-apps-enterprise-integration-liquid-transform/example-output-jsontojson.png)

<a name="template-considerations"></a>

## <a name="liquid-template-considerations"></a>Consideraciones sobre las plantillas de Liquid

* Las plantillas de Liquid siguen los [límites de tamaño de archivo de las asignaciones](../logic-apps/logic-apps-limits-and-config.md#artifact-capacity-limits) de Azure Logic Apps.

* La acción **Transform JSON to JSON - Liquid** (Transformar de JSON a JSON: Liquid) sigue la [implementación DotLiquid para Liquid](https://github.com/dotliquid/dotliquid). Esta implementación es un puerto a .NET Framework desde la [implementación Shopify para Liquid](https://shopify.github.io/liquid/) y varía en [casos concretos](https://github.com/dotliquid/dotliquid/issues).

  Estas son las diferencias conocidas:

  * La acción **Transform JSON to JSON - Liquid** (Transformar de JSON a JSON: Liquid) genera de forma nativa una cadena, que puede incluir JSON, XML, HTML, etc. La acción de Liquid solo indica que la salida de texto esperada de la plantilla de Liquid es una cadena JSON. La acción indica a la aplicación lógica que analice la entrada como un objeto JSON y que aplique un contenedor para que Liquid pueda interpretar la estructura JSON. Después de la transformación, la acción indica a la aplicación lógica que analice la salida de texto de Liquid a JSON.

    DotLiquid no comprende JSON de forma nativa, por lo que debe asegurarse de escapar el carácter de barra diagonal inversa (`\`) y cualquier otro carácter JSON reservado.

  * Si la plantilla usa [filtros de Liquid](https://shopify.github.io/liquid/basics/introduction/#filters), asegúrese de seguir las [convenciones de nomenclatura de DotLiquid y C#](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Designers#filter-and-output-casing), que usan *mayúscula al principio de la oración*. En todas las transformaciones Liquid, asegúrese de que los nombres de los filtros de la plantilla también usen mayúscula al principio. De lo contrario, los filtros no funcionarán.

    Por ejemplo, al usar el filtro `replace`, use `Replace`, no `replace`. La misma regla se aplica si se prueban ejemplos en [DotLiquid online](http://dotliquidmarkup.org/try-online). Para más información, consulte [Filtros de Liquid para Shopify](https://shopify.dev/docs/themes/liquid/reference/filters) y [Filtros de Liquid para DotLiquid](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Developers#create-your-own-filters). La especificación Shopify incluye ejemplos para cada filtro, por lo que, para realizar comparaciones, puede probar estos ejemplos en [Probar DotLiquid en línea](https://dotliquidmarkup.org/try-online).

  * Actualmente, el filtro `json` de los filtros de extensión Shopify [no está implementado en DotLiquid](https://github.com/dotliquid/dotliquid/issues/384). Normalmente, este filtro se usaría para preparar la salida de texto para el análisis de cadenas JSON; en cambio, debe usar el filtro `Replace`.

  * El filtro `Replace` estándar de la [implementación DotLiquid](https://github.com/dotliquid/dotliquid/blob/b6a7d992bf47e7d7dcec36fb402f2e0d70819388/src/DotLiquid/StandardFilters.cs#L425) usa la [coincidencia de expresiones regulares (RegEx)](/dotnet/standard/base-types/regular-expression-language-quick-reference), mientras que la [implementación Shopify](https://shopify.github.io/liquid/filters/replace/) usa la [coincidencia de cadenas simples](https://github.com/Shopify/liquid/issues/202). Ambas implementaciones parecen funcionar de la misma manera hasta que se usa un carácter reservado de RegEx o un carácter de escape en el parámetro match.

    Por ejemplo, para escapar el carácter de escape reservado de barra diagonal inversa de RegEx (`\`), use `| Replace: '\\', '\\'` y no `| Replace: '\', '\\'`. En estos ejemplos se muestra que el filtro `Replace` se comporta de forma diferente cuando se intenta escapar el carácter de barra diagonal inversa. Aunque esta versión funciona correctamente:

    `{ "SampleText": "{{ 'The quick brown fox "jumped" over the sleeping dog\\' | Replace: '\\', '\\' | Replace: '"', '\"'}}"}`

    Con este resultado:

    `{ "SampleText": "The quick brown fox \"jumped\" over the sleeping dog\\\\"}`

    Esta versión no funciona bien:

    `{ "SampleText": "{{ 'The quick brown fox "jumped" over the sleeping dog\\' | Replace: '\', '\\' | Replace: '"', '\"'}}"}`

    Con este error:

    `{ "SampleText": "Liquid error: parsing "\" - Illegal \ at end of pattern."}`

    Para más información, consulte [El filtro estándar Replace usa la coincidencia de patrones de RegEx...](https://github.com/dotliquid/dotliquid/issues/385).

  * El filtro `Sort` de la [implementación DotLiquid](https://github.com/dotliquid/dotliquid/blob/b6a7d992bf47e7d7dcec36fb402f2e0d70819388/src/DotLiquid/StandardFilters.cs#L326) ordena los elementos de una matriz o colección por propiedad, pero con estas diferencias:<p>

    * Sigue el [comportamiento sort_natural de Shopify](https://shopify.github.io/liquid/filters/sort_natural/), no el [comportamiento de ordenación de Shopify](https://shopify.github.io/liquid/filters/sort/).

    * Solo ordena en orden alfanumérico de cadena. Para más información, consulte [Ordenación numérica](https://github.com/Shopify/liquid/issues/980).

    * Usa el orden *distinguir mayúsculas de minúsculas*, no el orden que no realiza esta distinción. Para más información, consulte [El filtro Sort no sigue el comportamiento de mayúsculas/minúsculas de la especificación de Shopify]( https://github.com/dotliquid/dotliquid/issues/393).

<a name="other-transformations"></a>

## <a name="other-transformations-using-liquid"></a>Otras transformaciones mediante Liquid

Liquid no se limita únicamente a transformaciones JSON. También puede usar Liquid para realizar otras transformaciones, por ejemplo:

* [JSON a texto](#json-text)
* [XML a JSON](#xml-json)
* [XML a texto](#xml-text)

<a name="json-text"></a>

### <a name="transform-json-to-text"></a>Transformación de JSON en texto

Esta es la plantilla de Liquid utilizada en este ejemplo:

```json
{{content.firstName | Append: ' ' | Append: content.lastName}}
```

Estas son las salidas y entradas de ejemplo:

![Ejemplo de salida JSON a texto](./media/logic-apps-enterprise-integration-liquid-transform/example-output-jsontotext.png)

<a name="xml-json"></a>

### <a name="transform-xml-to-json"></a>Transformación XML a JSON

Esta es la plantilla de Liquid utilizada en este ejemplo:

``` json
[{% JSONArrayFor item in content -%}
      {{item}}
  {% endJSONArrayFor -%}]
```

El bucle `JSONArrayFor` es un mecanismo de bucle personalizado para la entrada XML, de forma que puede crear cargas JSON que eviten una coma final. Además, la condición `where` de este mecanismo de bucle personalizado utiliza el nombre del elemento XML para la comparación, en lugar del valor del elemento como sucede con otros filtros Liquid. Para más información, consulte [Profundización en la directiva set-body: colecciones de cosas](https://azure.microsoft.com/blog/deep-dive-on-set-body-policy).

Estas son las salidas y entradas de ejemplo:

![Ejemplo de salida XML a JSON](./media/logic-apps-enterprise-integration-liquid-transform/example-output-xmltojson.png)

<a name="xml-text"></a>

### <a name="transform-xml-to-text"></a>Transformación XML a texto

Esta es la plantilla de Liquid utilizada en este ejemplo:

``` json
{{content.firstName | Append: ' ' | Append: content.lastName}}
```

Estas son las salidas y entradas de ejemplo:

![Ejemplo de salida XML a texto](./media/logic-apps-enterprise-integration-liquid-transform/example-output-xmltotext.png)

## <a name="next-steps"></a>Pasos siguientes

* [Lenguaje y ejemplos de Liquid para Shopify](https://shopify.github.io/liquid/basics/introduction/)
* [DotLiquid](http://dotliquidmarkup.org/)
* [Probar DotLiquid en línea](https://dotliquidmarkup.org/try-online)
* [DotLiquid en GitHub](https://github.com/dotliquid/dotliquid)
* [Problemas de DotLiquid en GitHub](https://github.com/dotliquid/dotliquid/issues/)
* Más información sobre las [asignaciones](../logic-apps/logic-apps-enterprise-integration-maps.md)
