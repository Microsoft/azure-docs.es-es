---
title: Administración de características de Azure App Configuration | Microsoft Docs
description: Información general de cómo se puede usar Azure App Configuration para activar y desactivar las características de la aplicación bajo demanda
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
ms.openlocfilehash: 304ca08275ccf4bf32d38e3fc89dd5cf07460fa4
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413765"
---
# <a name="feature-management"></a>Administración de características

Tradicionalmente, la aplicación de una nueva característica de una aplicación requiere una nueva implementación completa de la propia aplicación. Para probar una característica, es probable que tenga que implementar su aplicación muchas veces para controlar cuándo se hace visible o quién puede verla. La administración de características es una práctica moderna de desarrollo de software que separa el lanzamiento de las características de la implementación del código y permite hacer cambios rápidos relacionados con la disponibilidad de las características bajo demanda. Usa una técnica denominada *marca de características* (también conocida como *activación/desactivación de funcionalidad* o *modificador de características*, entre otros) para administrar el ciclo de vida de una característica dinámicamente. Permite a los desarrolladores hacer frente a los problemas siguientes:

* **Administración de la rama de código**: Las marcas de características se usan con frecuencia para ajustar la nueva funcionalidad de la aplicación que está en desarrollo. Esta funcionalidad se "oculta" de forma predeterminada. Puede enviarla de manera segura, aunque no esté terminada, y permanecerá inactiva en producción. Este enfoque, denominado *implementación oscura*, puede liberar todo el código al final de cada ciclo de desarrollo. Ya no es necesario mantener ninguna rama de código en varios ciclos porque una característica tarde más de un período en completarse.
* **Realice pruebas en producción**: Puede utilizar las marcas de características para conceder acceso anticipado a la nueva funcionalidad en producción. Por ejemplo, puede limitar este acceso solo a los miembros del equipo o evaluadores beta internos. Estos usuarios obtendrán la experiencia de producción de fidelidad total, en lugar de una experiencia simulada o parcial en un entorno de prueba.
* **Lanzamiento como paquete piloto**: También puede utilizar las marcas de características para implementar una nueva funcionalidad a los usuarios finales incrementalmente. Puede dirigirse a un pequeño porcentaje de la población de usuarios en primer lugar y aumentar ese porcentaje gradualmente con el tiempo, una vez que haya adquirido más confianza en la aplicación.
* **Conmutador de eliminación instantánea**: Las marcas de características proporcionan una red de seguridad inherente para liberar la nueva funcionalidad. Con ellas, no solo puede activar, sino también desactivar las características de la aplicación fácilmente. Disfruta de la posibilidad de deshabilitar rápidamente una característica, si es necesario, sin la necesidad de recompilar y volver a implementar su aplicación.
* **Activación selectiva**: Si bien la mayoría de las marcas de características existen hasta que sus funcionalidades asociadas se hayan publicado con éxito, algunas pueden durar mucho tiempo. Puede utilizarlas como una forma de segmentar los usuarios y ofrecer un conjunto específico de características para cada grupo. Por ejemplo, puede tener una característica que funciona en un determinado explorador web. Puede definir una marca de características para que solo los usuarios de ese explorador puedan ver y usar la característica. La ventaja de este enfoque es que puede ampliar fácilmente la lista de exploradores admitidos más adelante sin tener que cambiar ningún código.

## <a name="basic-concepts"></a>Conceptos básicos

Estos son varios nuevos términos relacionados con la administración de características.

* **Marca de características**: Una marca de características es una variable con un estado binario de *on* o *off* que tiene un bloque de código asociado. Su estado decide si el bloque de código se ejecuta o no.
* **Administrador de características**: Un administrador de características es un paquete de aplicación que controla el ciclo de vida de todas las marcas de características en una aplicación. Normalmente proporciona funcionalidad adicional, como almacenamiento en caché de las marcas y actualización de sus estados.
* **Filtro**: Un filtro es una regla para evaluar el estado de una marca de características. Grupo de usuarios, dispositivo o tipo de explorador, ubicación geográfica o período de tiempo son ejemplos de lo que puede representar un filtro.

Una implementación eficaz de la administración de características consta de al menos dos componentes que funcionan conjuntamente: una aplicación que hace uso de las marcas de características y un repositorio independiente que almacena las marcas de características y sus estados actuales. A continuación se ilustran las interacciones.

## <a name="feature-flag-usage-in-code"></a>Uso de la marca de características en el código

El patrón básico para implementar las marcas de características en una aplicación es sencillo. Puede conceptualizarlo como una variable de estado booleana utilizada con una declaración condicional `if` en su código.

```csharp
if (featureFlag) {
    // Run the following code
}
```

En este caso, si `featureFlag` está establecido en `True`, se ejecuta el bloque de código incluido; en caso contrario, se omite. El valor de `featureFlag` se puede establecer de forma estática:

```csharp
bool featureFlag = true;
```

También puede evaluarse según ciertas reglas:

```csharp
bool featureFlag = isBetaUser();
```

Un patrón de marca de características un poco más complicado incluye también una instrucción `else`.

```csharp
if (featureFlag) {
    // This following code will run if the featureFlag value is true
} else {
    // This following code will run if the featureFlag value is false
}
```

Sin embargo, se puede reescribir en el patrón básico. El artículo [Use feature flags in an ASP.NET Core app](./use-feature-flags-dotnet-core.md) (Uso de marcas de características en una aplicación ASP.NET Core) muestra la ventaja de estandarizar un patrón de código simple.

```csharp
if (featureFlag) {
    // This following code will run if the featureFlag value is true
}

if (!featureFlag) {
    // This following code will run if the featureFlag value is false
}
```

## <a name="feature-flag-repository"></a>Repositorio de marcas de características

Para usarlas de forma eficaz, debe externalizar todas las marcas de características utilizadas en una aplicación. Eso le permitirá cambiar los estados de las marcas de características sin modificar y volver a implementar la propia aplicación. Azure App Configuration está diseñado para ser un repositorio centralizado para las marcas de características. Puede utilizarlo para definir los distintos tipos de marcas de características y manipular sus estados con rapidez y confianza. Luego, podrá acceder a estas marcas de características fácilmente desde su aplicación utilizando las bibliotecas de App Configuration para varios marcos de lenguaje de programación. El artículo [Use feature flags in an ASP.NET Core app](./use-feature-flags-dotnet-core.md) (Uso de marcas de características en una aplicación ASP.NET Core) muestra cómo se utilizan el proveedor de App Configuration de .NET Core y las bibliotecas de administración de características como una solución completa para implementar las marcas de características para la aplicación web ASP.NET.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Add feature flags to an ASP.NET Core web app](./quickstart-feature-flag-aspnet-core.md) (Adición de marcas de características a una aplicación web ASP.NET Core)  
