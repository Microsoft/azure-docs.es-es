---
title: Cómo usar valores con nombre en las directivas de Azure API Management
description: Aprenda a usar valores con nombre en las directivas de Azure API Management. Los valores con nombre pueden contener cadenas literales y expresiones de directiva.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/08/2020
ms.author: apimpm
ms.openlocfilehash: 3f317276ae92e6121d519553b7883677dab89705
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852198"
---
# <a name="how-to-use-named-values-in-azure-api-management-policies"></a>Cómo usar valores con nombre en las directivas de Azure API Management

En API Management, las directivas constituyen una funcionalidad eficaz del sistema que permite a Azure Portal cambiar el comportamiento de la API mediante la configuración. Las directivas son una colección de declaraciones que se ejecutan secuencialmente en la solicitud o respuesta de una API. Las instrucciones de las directivas se pueden crear con valores de texto literal, expresiones de directiva y valores con nombre.

Cada instancia del servicio de API Management tiene una colección de pares clave-valor, que se denominan valores con nombre y que son globales para la instancia del servicio. No se impone ningún límite en el número de elementos de la colección. Los valores con nombre se pueden usar para administrar valores de cadena constantes en todas las directivas y la configuración de API. Cada valor con nombre puede tener los siguientes atributos:

| Atributo      | Tipo            | Descripción                                                                                                                            |
| -------------- | --------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `Display name` | string          | Se usa para hacer referencia al valor con nombre en las directivas. Una cadena que contenga entre 1 y 256 caracteres. Solo se permiten letras, números, punto y guión. |
| `Value`        | string          | Valor real. No debe estar vacío ni contener solo espacios en blanco. 4096 caracteres como máximo.                                        |
| `Secret`       | boolean         | Determina si el valor es secreto y si se debe cifrar.                                                               |
| `Tags`         | matriz de cadena | Se usa para filtrar la lista de valores con nombre. Hasta 32 etiquetas.                                                                                    |

![Valores con nombre](./media/api-management-howto-properties/named-values.png)

Los valores con nombre pueden contener cadenas literales y [expresiones de directiva](./api-management-policy-expressions.md). Por ejemplo, el valor de `Expression` es una expresión de directiva que devuelve una cadena que contiene la fecha y la hora actuales. El valor con nombre `Credential` está marcado como secreto, por lo que su valor no se muestra de forma predeterminada.

| Nombre       | Value                      | Secreto | Etiquetas          |
| ---------- | -------------------------- | ------ | ------------- |
| Value      | 42                         | False  | vital-numbers |
| Credential: | ••••••••••••••••••••••     | True   | security      |
| Expression | @(DateTime.Now.ToString()) | False  |               |

> [!NOTE]
> En lugar de los valores con nombre almacenados en un servicio de API Management, puede usar los valores almacenados en el servicio [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) como se muestra en este [ejemplo](https://github.com/Azure/api-management-policy-snippets/blob/master/examples/Look%20up%20Key%20Vault%20secret%20using%20Managed%20Service%20Identity.policy.xml).

## <a name="to-add-and-edit-a-named-value"></a>Para agregar y editar un valor con nombre

![Agregar un valor con nombre](./media/api-management-howto-properties/add-property.png)

1. Seleccione **API** en **API MANAGEMENT**.
2. Seleccione **Valores con nombre**.
3. Presione **+Agregar**.

    Los campos Nombre y Valor son necesarios. Si el valor es un secreto, marque la casilla _Este valor es un secreto_. Escriba una o varias etiquetas opcionales que le ayuden a organizar los valores con nombre y haga clic en Guardar.

4. Haga clic en **Crear**.

Una vez creado el valor con nombre, es posible editarlo haciendo clic en él. Si cambia el nombre del valor con nombre, las directivas que hagan referencia a ese valor con nombre se actualizarán automáticamente para utilizar el nuevo nombre.

## <a name="to-delete-a-named-value"></a>Para eliminar un valor con nombre

Para eliminar un valor con nombre, haga clic en **Eliminar** junto al valor con nombre que se va a eliminar.

> [!IMPORTANT]
> Si se hace referencia al valor con nombre mediante alguna directiva,no podrá eliminarlo correctamente hasta que lo quite de todas las directivas que lo utilicen.

## <a name="to-search-and-filter-named-values"></a>Para buscar y filtrar valores con nombre

En la pestaña **Valores con nombre** se incluyen las funcionalidades de búsqueda y filtrado que le ayudan a administrar los valores con nombre. Para filtrar la lista de valores con nombre por el nombre, escriba un término de búsqueda en el cuadro de texto **Buscar propiedad** . Para mostrar todos los valores con nombre, borre el cuadro de texto **Search property** (Buscar propiedad) y pulse la tecla Entrar.

Para filtrar la lista por etiqueta, escriba una o varias etiquetas en el cuadro de texto **Filtrar por etiquetas** . Para mostrar todos los valores con nombre, borre el cuadro de texto **Filtrar por etiquetas** y pulse la tecla Entrar.

## <a name="to-use-a-named-value"></a>Para usar un valor con nombre

Para utilizar un valor con nombre en una directiva, coloque su nombre dentro de un par doble de llaves (como `{{ContosoHeader}}`), de la misma forma que se muestra en el ejemplo siguiente:

```xml
<set-header name="{{ContosoHeader}}" exists-action="override">
  <value>{{ContosoHeaderValue}}</value>
</set-header>
```

En este ejemplo, `ContosoHeader` se utiliza como el nombre de un encabezado en una directiva `set-header`, y `ContosoHeaderValue` se utiliza como valor de ese encabezado. Cuando esta directiva se evalúa durante una solicitud o responde a la pasarela de API Management, `{{ContosoHeader}}` y `{{ContosoHeaderValue}}` se reemplazan por sus respectivos valores.

Los valores con nombre se pueden utilizar como atributo completo o como valores de elemento (tal como se muestra en el ejemplo anterior), pero también se pueden insertar o combinar con parte de una expresión de texto literal, como en el ejemplo siguiente: `<set-header name = "CustomHeader{{ContosoHeader}}" ...>`

Los valores con nombre también pueden contener expresiones de directiva. En el ejemplo siguiente, se utiliza la propiedad `ExpressionProperty`.

```xml
<set-header name="CustomHeader" exists-action="override">
    <value>{{ExpressionProperty}}</value>
</set-header>
```

Cuando esta directiva se evalúa, se reemplaza `{{ExpressionProperty}}` por su valor: `@(DateTime.Now.ToString())`. Puesto que el valor es una expresión de directiva, la expresión se evalúa y la directiva continúa con su ejecución.

Puede probarlo en el portal para desarrolladores si llama a una operación que tenga dentro de su ámbito una directiva con valores con nombre. En el ejemplo siguiente, se llama a una operación con las dos directivas `set-header` de ejemplo anteriores con valores con nombre. Tenga en cuenta que la respuesta contiene dos encabezados personalizados que se han configurado mediante directivas con valores con nombre.

![Portal para desarrolladores][api-management-send-results]

Si busca en el [seguimiento de API Inspector](api-management-howto-api-inspector.md) una llamada que incluya las dos directivas de ejemplo anteriores con valores con nombre, verá las dos directivas `set-header` con los valores con nombre insertados, así como la evaluación de la expresión de directiva para el valor con nombre que la contiene.

![Seguimiento de API Inspector][api-management-api-inspector-trace]

Aunque los valores con nombre pueden contener expresiones de directiva, no pueden contener otros valores con nombre. En caso de usarse texto que contenga una referencia a valores con nombre para un valor, como `Text: {{MyProperty}}`, dicha referencia no se resolverá ni reemplazará.

## <a name="next-steps"></a>Pasos siguientes

-   Obtenga más información sobre cómo trabajar con directivas
    -   [Directivas de Azure API Management](api-management-howto-policies.md)
    -   [Referencia de la directiva](./api-management-policies.md)
    -   [Expresiones de directiva](./api-management-policy-expressions.md)

[api-management-send-results]: ./media/api-management-howto-properties/api-management-send-results.png
[api-management-properties-filter]: ./media/api-management-howto-properties/api-management-properties-filter.png
[api-management-api-inspector-trace]: ./media/api-management-howto-properties/api-management-api-inspector-trace.png
