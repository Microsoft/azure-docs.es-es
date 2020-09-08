---
title: Información sobre la administración de características mediante Azure App Configuration
description: Activar y desactivar características mediante Azure App Configuration
author: lisaguthrie
ms.author: lcozzens
ms.service: azure-app-configuration
ms.custom: devx-track-dotnet
ms.topic: conceptual
ms.date: 02/20/2020
ms.openlocfilehash: b77f0063b37adbfaecaff68387e858d0077561b3
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212654"
---
# <a name="feature-management-overview"></a>Introducción a la administración de características

Tradicionalmente, enviar una nueva característica de aplicación requiere una reimplementación completa de la propia aplicación. La prueba de una característica requiere con frecuencia varias implementaciones de la aplicación.  Cada implementación puede cambiar la característica o exponer la característica a distintos clientes para realizar pruebas.  

La administración de características es una práctica moderna de desarrollo de software que separa el lanzamiento de las características de la implementación del código y permite hacer cambios rápidos relacionados con la disponibilidad de las características a petición. Usa una técnica denominada *marcas de características* (también conocida como *activación/desactivación de funcionalidad* o *modificador de características*, entre otros) para administrar el ciclo de vida de una característica dinámicamente.

La administración de características ayuda a los desarrolladores a abordar los siguientes problemas:

* **Administración de la rama de código**: Use las marcas de características para ajustar la nueva funcionalidad de la aplicación que actualmente está en fase de desarrollo. Esta funcionalidad se "oculta" de forma predeterminada. La característica se puede enviar de forma segura, aunque esté sin terminar, y permanecer inactiva en producción. Este enfoque, denominado *implementación oscura*, puede liberar todo el código al final de cada ciclo de desarrollo. Ya no es necesario mantener las ramas de código en varios ciclos de desarrollo, ya que una característica determinada requiere más de un ciclo para completarse.
* **Realice pruebas en producción**: Use las marcas de características para conceder acceso anticipado a la nueva funcionalidad en producción. Por ejemplo, puede limitar el acceso a los miembros del equipo o evaluadores beta internos. Estos usuarios obtendrán una experiencia de producción de fidelidad total, en lugar de una experiencia simulada o parcial en un entorno de prueba.
* **Lanzamiento como paquete piloto**: Use las marcas de características para implementar una nueva funcionalidad en los usuarios finales de manera incremental. Puede centrarse en un pequeño porcentaje de la población de usuarios en primer lugar y aumentar ese porcentaje gradualmente con el tiempo.
* **Conmutador de eliminación instantánea**: Las marcas de características proporcionan una red de seguridad inherente para liberar la nueva funcionalidad. Puede activar y desactivar las características de la aplicación sin tener que volver a implementar ningún código. Si es necesario, puede deshabilitar rápidamente una característica sin volver a generar e implementar la aplicación.
* **Activación selectiva**: Use las marcas de características para segmentar los usuarios y ofrecer un conjunto específico de características a cada grupo. Puede tener una característica que funcione solo en un determinado explorador web. Puede definir una marca de características para que solo los usuarios de ese explorador puedan ver y usar la característica. Con este enfoque, puede ampliar fácilmente la lista de exploradores admitidos más adelante sin tener que cambiar ningún código.

## <a name="basic-concepts"></a>Conceptos básicos

Estos son varios términos nuevos relacionados con la administración de características:

* **Marca de características**: una marca de características es una variable con un estado binario de *activado* o *desactivado*. La marca de características también tiene un bloque de código asociado. El estado de la marca de características se desencadena si el bloque de código se ejecuta.
* **Administrador de características**: Un administrador de características es un paquete de aplicación que controla el ciclo de vida de todas las marcas de características en una aplicación. El administrador de características también proporciona funcionalidad adicional, incluyendo el almacenamiento en caché de las marcas de características y la actualización de sus estados.
* **Filtro**: Un filtro es una regla para evaluar el estado de una marca de características. Los filtros posibles incluyen grupos de usuarios, tipos de dispositivos o de exploradores, ubicaciones geográficas y ventanas de tiempo.

Una implementación eficaz de la administración de características consta de al menos dos componentes que funcionan conjuntamente:

* Una aplicación que hace uso de las marcas de características.
* Un repositorio independiente que almacena las marcas de características y sus estados actuales.

## <a name="using-feature-flags-in-your-code"></a>Uso de marcas de características en el código

El patrón básico para implementar las marcas de características en una aplicación es sencillo. Una marca de características es una variable de estado booleana que controla una instrucción condicional en el código:

```csharp
if (featureFlag) {
    // Run the following code
}
```

Puede establecer el valor de `featureFlag` estáticamente.

```csharp
bool featureFlag = true;
```

Puede evaluar el estado de la marca según ciertas reglas:

```csharp
bool featureFlag = isBetaUser();
```

Puede ampliar el condicional para establecer el comportamiento de la aplicación para cualquier estado:

```csharp
if (featureFlag) {
    // This following code will run if the featureFlag value is true
} else {
    // This following code will run if the featureFlag value is false
}
```

## <a name="feature-flag-repository"></a>Repositorio de marcas de características

Para usar marcas de características de forma eficaz, debe externalizar todas las marcas de características usadas en una aplicación. Esto le permite cambiar los estados de las marcas de características sin modificar y volver a implementar la propia aplicación.

Azure App Configuration proporciona un repositorio centralizado para las marcas de características. Puede usarla para definir distintos tipos de marcas de características y manipular sus estados con rapidez y confianza. Luego, puede usar las bibliotecas de App Configuration con diversos marcos de lenguajes de programación para acceder fácilmente a estas marcas de características desde su aplicación.

En [Use feature flags in an ASP.NET Core app](./use-feature-flags-dotnet-core.md) (Uso de marcas de características en una aplicación de ASP.NET Core) se muestra cómo se usan juntos el proveedor de App Configuration de .NET Core y las bibliotecas de administración de características para implementar las marcas de características para la aplicación web de ASP.NET.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Add feature flags to an ASP.NET Core web app](./quickstart-feature-flag-aspnet-core.md) (Adición de marcas de características a una aplicación web ASP.NET Core)  
