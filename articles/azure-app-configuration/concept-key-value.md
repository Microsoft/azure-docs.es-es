---
title: Descripción del almacén de pares clave-valor de Azure App Configuration
description: Describa el almacenamiento de pares clave-valor en Azure App Configuration, que almacena los datos de configuración como pares clave-valor. Los pares clave-valor son una representación de la configuración de la aplicación.
author: lisaguthrie
ms.author: lcozzens
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 02/19/2020
ms.openlocfilehash: b1998532c3d9e4272d91280d57d9ea2f6e7a262c
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88586399"
---
# <a name="keys-and-values"></a>Claves y valores

Azure App Configuration almacena los datos de configuración como claves y valores. Las claves y los valores son una representación sencilla y flexible de la configuración de la aplicación usada por los desarrolladores.

## <a name="keys"></a>Claves

Las claves sirven como identificadores de las claves y los valores y se usan para almacenar y recuperar los valores correspondientes. Es una práctica habitual la organización de las claves en un espacio de nombres jerárquico mediante un delimitador de caracteres como `/` o `:`. Use una convención más adecuada para su aplicación. Azure App Configuration trata las claves como un todo. No analiza las claves para averiguar cómo están estructurados sus nombres o para aplicar ninguna regla en ellas.

Este es un ejemplo de nombres de clave estructurados en una jerarquía basada en servicios de componentes:

```aspx
    AppName:Service1:ApiEndpoint
    AppName:Service2:ApiEndpoint
```

El uso de datos de configuración dentro de los marcos de las aplicaciones puede dictar esquemas de nomenclatura específicos para las claves y los valores. Por ejemplo, el marco de trabajo de Spring Cloud de Java define los recursos `Environment` que suministran la configuración a una aplicación de Spring.  Esta se va a parametrizar mediante variables entre las que se incluyen el *nombre de la aplicación* y el *perfil*. Las claves de los datos de configuración relacionadas con Spring Cloud empiezan normalmente por estos dos elementos, separados por un delimitador.

Las claves almacenadas en App Configuration distinguen entre mayúsculas y minúsculas y son cadenas basadas en Unicode. Las claves *app1* y *App1* se consideran diferentes en un almacén de App Configuration. Téngalo en cuenta al utilizar las opciones de configuración en una aplicación ya que algunos marcos de trabajo administran las claves de configuración sin hacer distinción entre mayúsculas y minúsculas. No se recomienda usar mayúsculas y minúsculas para diferenciar las claves.

En los nombres de clave se puede usar cualquier carácter Unicode, excepto `%`. Un nombre de clave no puede ser `.` ni `..`. Hay un límite de tamaño combinado de 10 KB en una clave y un valor. Este límite incluye todos los caracteres de la clave, su valor y todos los atributos opcionales asociados. Dentro de este límite, puede tener múltiples niveles jerárquicos para las claves.

### <a name="design-key-namespaces"></a>Diseño de espacios de nombres de clave

Hay dos enfoques generales en relación con la nomenclatura de las claves que se usa en los datos de configuración: plano o jerárquico. Estos métodos son similares desde la perspectiva del uso de la aplicación, pero la nomenclatura jerárquica ofrece una serie de ventajas:

* Es más fácil de leer. Los delimitadores de un nombre de clave jerárquica funcionan como espacios en una oración. También proporcionan las divisiones naturales entre palabras.
* Es más fácil de administrar. Un nombre de clave jerárquico representa grupos lógicos de datos de configuración.
* Es más fácil de usar. Resulta más fácil escribir una consulta cuyos patrones coinciden con las claves de una estructura jerárquica y que recupera solo una porción de los datos de configuración. Además, muchas plataformas de programación más recientes tienen una compatibilidad nativa para datos de configuración jerárquicos tal que la aplicación puede hacer uso de conjuntos específicos de configuración.

Puede organizar jerárquicamente las claves de App Configuration de muchas formas. Piense en dichas claves como [identificadores URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier). Cada clave jerárquica es una *ruta* de recurso que consta de uno o varios componentes unidos por delimitadores. Elija qué carácter se debe usar como delimitador en función de qué aplicación, lenguaje o plataforma de programación necesita. Use varios delimitadores para diferentes claves de App Configuration.

### <a name="label-keys"></a>Claves de etiqueta

Los pares clave-valor de App Configuration pueden tener opcionalmente un atributo label. Las etiquetas se utilizan para diferenciar los pares clave-valor con la misma clave. Una clave *app1* con las etiquetas *A* y *B* forma dos claves independientes en un almacén de App Configuration. De forma predeterminada, una clave y un valor no tienen etiqueta. Para hacer referencia de forma explícita a una clave y un valor sin etiqueta, use `\0` (URL codificada como `%00`).

Las etiquetas proporcionan una manera cómoda de crear variantes de una clave. Un uso habitual de las etiquetas consiste en especificar varios entornos para la misma clave:

```
    Key = AppName:DbEndpoint & Label = Test
    Key = AppName:DbEndpoint & Label = Staging
    Key = AppName:DbEndpoint & Label = Production
```

### <a name="version-key-values"></a>Versiones de claves y valores

Use etiquetas como una manera de crear varias versiones de una clave y un valor. Por ejemplo, puede escribir un número de versión de la aplicación o un identificador de confirmación de Git en las etiquetas para identificar los pares clave-valor asociados con una compilación de software concreta.

> [!NOTE]
> Si busca versiones modificadas, App Configuration conserva todos los cambios de una clave y un valor ocurridos en el último período de tiempo determinado de forma automática. Vea [Instantánea en un momento dado](./concept-point-time-snapshot.md) para obtener más detalles.

### <a name="query-key-values"></a>Consulta de claves y valores

Cada par clave-valor se identifica de forma exclusiva mediante su clave más una etiqueta que puede ser `\0`. Para consultar claves y valores de un almacén de App Configuration, especifique un patrón. El almacén de App Configuration devuelve todos los pares clave-valor que coinciden con el patrón, incluidos sus valores y atributos correspondientes. Use los siguientes modelos de claves en las llamadas a la API REST para App Configuration:

| Clave | Descripción |
|---|---|
| se omite `key` o `key=*` | Coincide con todas las claves |
| `key=abc` | Coincide con el nombre de clave **abc** exactamente |
| `key=abc*` | Coincide con los nombres de clave que empiezan por **abc** |
| `key=abc,xyz` | Coincide con los nombres de clave **abc** o **xyz**. Tiene un límite de cinco archivos .csv |

También puede incluir los siguientes patrones de etiqueta:

| Etiqueta | Descripción |
|---|---|
| se omite `label` o `label=*` | Coincide con cualquier etiqueta, que incluye `\0` |
| `label=%00` | Coincide con la etiqueta `\0` |
| `label=1.0.0` | Coincide exactamente con la etiqueta **1.0.0** |
| `label=1.0.*` | Coincide con las etiquetas que empiezan por **1.0.** |
| `label=%00,1.0.0` | Coincide con las etiquetas `\0` o **1.0.0**, limitado a cinco CSV |

> [!NOTE]
> `*`, `,` y `\` son caracteres reservados de las consultas. Si se usa un carácter reservado en los nombres o las etiquetas de las claves, debe aplicarle escape mediante `\{Reserved Character}` en las consultas.

## <a name="values"></a>Valores

Los valores asignados a las claves también son cadenas unicode. Puede usar todos los caracteres unicode para los valores.

### <a name="use-content-type"></a>Uso de Content-Type
Cada clave y valor de App Configuration tiene un atributo content-type. Si quiere, puede usar este atributo para almacenar información sobre el tipo de valor de un par clave-valor que ayude a la aplicación a procesarlo correctamente. Puede usar cualquier formato para content-type. App Configuration usa [tipos de medios]( https://www.iana.org/assignments/media-types/media-types.xhtml) (también conocidos como tipos MIME) para tipos de datos integrados como marcas de características, referencias de Key Vault y valores y claves JSON.

## <a name="next-steps"></a>Pasos siguientes

* [Instantánea en un momento dado](./concept-point-time-snapshot.md)
* [Administración de características](./concept-feature-management.md)
* [Control de eventos](./concept-app-configuration-event.md)
