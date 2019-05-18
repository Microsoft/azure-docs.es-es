---
title: Almacén de pares clave-valor de Azure App Configuration | Microsoft Docs
description: Una introducción sobre cómo se almacenan los datos de configuración en Azure App Configuration
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: maiye
editor: ''
ms.service: azure-app-configuration
ms.devlang: na
ms.topic: overview
ms.workload: tbd
ms.date: 04/19/2019
ms.author: yegu
ms.openlocfilehash: 4c741bb86242abfb03d01c902dbaa84d83491dd9
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408743"
---
# <a name="keys-and-values"></a>Claves y valores

Azure App Configuration almacena los datos de configuración como pares clave-valor. Los pares clave-valor constituyen una manera sencilla aunque flexible de representar varios tipos de configuraciones de la aplicación con las que los desarrolladores están familiarizados.

## <a name="keys"></a>Claves

Las claves sirven como nombre de los pares clave-valor y se usan para almacenar y recuperar los valores correspondientes. Es una práctica habitual la organización de las claves en un espacio de nombres jerárquico mediante un delimitador de caracteres como `/` o `:`. Use la convención que resulte más adecuada para su aplicación. Azure App Configuration trata las claves como un todo. No analiza las claves para averiguar cómo están estructurados sus nombres o para aplicar ninguna regla en ellas.

El uso del almacén de configuración dentro de los marcos de trabajo de la aplicación puede dictar esquemas de nombres específicos para los pares clave-valor. Por ejemplo, el marco de trabajo de Spring Cloud de Java define los recursos `Environment` que aportan la configuración a una aplicación de Spring que se va a parametrizar mediante variables entre las que se incluyen el *nombre de la aplicación* y el *perfil*. Las claves de los datos de configuración relacionadas con Spring Cloud empiezan normalmente por estos dos elementos, separados por un delimitador.

Las claves almacenadas en App Configuration distinguen entre mayúsculas y minúsculas y son cadenas basadas en Unicode. Las claves *app1* y *App1* se consideran diferentes en un almacén de App Configuration. Téngalo en cuenta al utilizar las opciones de configuración en una aplicación ya que algunos marcos de trabajo administran las claves de configuración sin hacer distinción entre mayúsculas y minúsculas. Por ejemplo, el sistema de configuración de ASP.NET Core trata las claves como cadenas sin distinción de mayúsculas y minúsculas. Para evitar comportamientos impredecibles al realizar consultas en App Configuration dentro de una aplicación de ASP.NET Core, no utilizan claves que solo se distingan por el uso de mayúsculas o minúsculas.

Puede usar cualquier carácter unicode en los nombres de clave que se especifican en App Configuration, excepto `*`, `,` y `\`. Estos caracteres están reservados. Si tiene que incluir un carácter reservado, deberá especificar un carácter de escape con `\{Reserved Character}`. Hay un límite de tamaño combinado de diez mil caracteres en un par clave-valor. Este límite incluye todos los caracteres de la clave, su valor y todos los atributos opcionales asociados. Dentro de este límite, puede tener múltiples niveles jerárquicos para las claves.

### <a name="design-key-namespaces"></a>Diseño de espacios de nombres de clave

Hay dos enfoques generales en relación con la nomenclatura de las claves que se usa en los datos de configuración: plano o jerárquico. Estos métodos son similares desde la perspectiva del uso de la aplicación, pero la nomenclatura jerárquica ofrece una serie de ventajas:

* Es más fácil de leer. En lugar de una secuencia larga de caracteres, los delimitadores de una función de nombre de clave jerárquico funciona como los espacios de una frase. También proporcionan las divisiones naturales entre palabras.
* Es más fácil de administrar. Un nombre de clave jerárquico representa grupos lógicos de datos de configuración.
* Es más fácil de usar. Resulta más fácil escribir una consulta cuyos patrones coinciden con las claves de una estructura jerárquica y que recupera solo una porción de los datos de configuración. Además, muchas plataformas de programación más recientes tienen una compatibilidad nativa para datos de configuración jerárquicos tal que la aplicación puede hacer uso de conjuntos específicos de configuración.

Puede organizar jerárquicamente las claves de App Configuration de muchas formas. Piense en dichas claves como [identificadores URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier). Cada clave jerárquica es una *ruta* de recurso que consta de uno o varios componentes unidos por delimitadores. Elija qué carácter se debe usar como delimitador en función de qué aplicación, lenguaje o plataforma de programación necesita. Use varios delimitadores para diferentes claves de App Configuration.

Estos son algunos ejemplos de cómo se pueden estructurar los nombres de claves en una jerarquía:

* Según los servicios de los componentes.

        AppName:Service1:ApiEndpoint
        AppName:Service2:ApiEndpoint

* Según las regiones de implementación.

        AppName:Region1:DbEndpoint
        AppName:Region2:DbEndpoint

### <a name="label-keys"></a>Claves de etiqueta

Los valores de clave de App Configuration pueden tener un atributo de etiqueta. Las etiquetas se utilizan para diferenciar los valores de clave con la misma clave. Una clave *app1* con las etiquetas *A* y *B* forma dos claves independientes en un almacén de App Configuration. De forma predeterminada, la etiqueta de un valor de clave está vacía o es `null`.

Las etiquetas proporcionan una manera cómoda de crear variantes de una clave. Un uso habitual de las etiquetas consiste en especificar varios entornos para la misma clave:

    Key = AppName:DbEndpoint & Label = Test
    Key = AppName:DbEndpoint & Label = Staging
    Key = AppName:DbEndpoint & Label = Production

### <a name="version-key-values"></a>Valores de clave de versión

App Configuration no crea valores de clave de versión automáticamente a medida que se modifican. Use etiquetas como una manera de crear varias versiones de un valor de clave. Por ejemplo, puede escribir un número de versión de la aplicación o un identificador de confirmación de Git en las etiquetas para identificar los valores de clave asociados con una compilación de software en particular.

Puede usar cualquier carácter Unicode en las etiquetas, excepto para `*`, `,` y `\`. Estos caracteres están reservados. Para incluir un carácter reservado, debe evitarlo utilizando `\{Reserved Character}`.

### <a name="query-key-values"></a>Valores de clave de consulta

Cada valor de clave se identifica por su clave más una etiqueta que puede ser `null`. Si desea consultar los valores de clave en un almacén de configuración de la aplicación, especifique un modelo. El almacén de configuración de la aplicación devuelve todos los valores de clave que coincidan con el modelo y sus valores y atributos correspondientes. Use los siguientes modelos de claves en las llamadas a la API REST para App Configuration:

| Clave | |
|---|---|
| se omite `key` o `key=*` | Coincide con todas las claves |
| `key=abc` | Coincide con el nombre de clave **abc** exactamente |
| `key=abc*` | Coincide con los nombres de clave que empiezan por **abc** |
| `key=*abc` | Coincide con los nombres de clave que terminan en **abc** |
| `key=*abc*` | Coincide con los nombres de clave que contienen **abc** |
| `key=abc,xyz` | Coincide con los nombres de clave **abc** o **xyz**, limitado a cinco CSV |

También puede incluir los siguientes patrones de etiqueta:

| Etiqueta | |
|---|---|
| se omite `label` o `label=*` | Coincide con cualquier etiqueta, que incluye `null` |
| `label=%00` | Coincide con la etiqueta `null` |
| `label=1.0.0` | Coincide exactamente con la etiqueta **1.0.0** |
| `label=1.0.*` | Coincide con las etiquetas que empiezan por **1.0.** |
| `label=*.0.0` | Coincide con las etiquetas que terminan en **.0.0** |
| `label=*.0.*` | Coincide con las etiquetas que contienen **.0.** |
| `label=%00,1.0.0` | Coincide con las etiquetas `null` o **1.0.1**, limitado a cinco CSV |

## <a name="values"></a>Valores

Los valores asignados a las claves también son cadenas unicode. Puede usar todos los caracteres unicode para los valores. Hay un tipo de contenido opcional definido por el usuario que se asocia a cada valor. Utilice este atributo para almacenar información, por ejemplo, un esquema de codificación, sobre un valor que ayuda a la aplicación a procesarlo correctamente.

Los datos de configuración almacenados en un almacén de configuración de la aplicación, que incluye todas las claves y valores, se cifran en reposo y en tránsito. App Configuration no es una solución de reemplazo para Azure Key Vault. No almacene secretos de aplicación en ella.

## <a name="next-steps"></a>Pasos siguientes

* [Instantánea en un momento dado](./concept-point-time-snapshot.md)  
* [Administración de características](./concept-feature-management.md)  
