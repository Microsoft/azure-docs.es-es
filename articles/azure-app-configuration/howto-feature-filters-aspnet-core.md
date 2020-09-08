---
title: Uso de filtros de características para habilitar una característica para un subconjunto de usuarios
titleSuffix: Azure App Configuration
description: Obtenga información sobre cómo usar los filtros de características y habilitar una característica para un subconjunto de usuarios
ms.service: azure-app-configuration
ms.custom: devx-track-csharp
author: lisaguthrie
ms.author: lcozzens
ms.topic: conceptual
ms.date: 3/9/2020
ms.openlocfilehash: 5b2eb942581f6e4163012b0f767d04c02689bb7b
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88206759"
---
# <a name="use-feature-filters-to-enable-a-feature-for-a-subset-of-users"></a>Uso de filtros de características para habilitar una característica para un subconjunto de usuarios

Las marcas de característica permiten activar o desactivar la funcionalidad de la aplicación. Una marca de característica simple está activada o desactivada. La aplicación siempre se comporta de la misma manera. Por ejemplo, podría implementar una nueva característica detrás de una marca de característica. Cuando la marca de característica está habilitada, todos los usuarios ven la nueva característica. Al deshabilitar la marca de característica, se oculta la nueva característica.

Por el contrario, una _marca de característica condicional_ permite que la marca de característica se habilite o deshabilite de forma dinámica. La aplicación puede comportarse de forma diferente, dependiendo de los criterios de la marca de característica. Supongamos que desea mostrar la nueva característica a un pequeño subconjunto de usuarios en primer lugar. Una marca de característica condicional permite habilitar la marca de característica para algunos usuarios al deshabilitarla para otras personas. _Los filtros de características_ determinan el estado de la marca de característica cada vez que se evalúa.

La biblioteca `Microsoft.FeatureManagement` incluye dos filtros de características:

- `PercentageFilter` habilita la marca de característica en función de un porcentaje.
- `TimeWindowFilter` habilita la marca de característica durante un período de tiempo especificado.

También puede crear su propio filtro de características que implementa la [interfaz Microsoft.FeatureManagement.IFeatureFilter](/dotnet/api/microsoft.featuremanagement.ifeaturefilter).

## <a name="registering-a-feature-filter"></a>Registro de un filtro de características

Para registrar un filtro de características, llame al método `AddFeatureFilter` y especifique el nombre del filtro de características. Por ejemplo, el código siguiente registra `PercentageFilter`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews();
    services.AddFeatureManagement().AddFeatureFilter<PercentageFilter>();
}
```

## <a name="configuring-a-feature-filter-in-azure-app-configuration"></a>Configuración de un filtro de características en Azure App Configuration

Algunos filtros de características tienen opciones de configuración adicionales. Por ejemplo, `PercentageFilter` activa una característica basada en un porcentaje. Tiene una configuración que define el porcentaje que se va a usar.

Puede configurar estas opciones para las marcas de característica definidas en Azure App Configuration. Por ejemplo, siga estos pasos para usar `PercentageFilter` y habilitar la marca de característica para el 50 % de las solicitudes a una aplicación web:

1. Siga las instrucciones del artículo [Guía de inicio rápido: Agregue marcas de característica a una aplicación ASP.NET Core](./quickstart-feature-flag-aspnet-core.md) para crear una aplicación web con una marca de característica.

1. En Azure Portal, vaya al almacén de configuración y haga clic en **Feature Manager** (Administrador de características).

1. Haga clic en el menú contextual de la marca de característica *Beta* que creó en la guía de inicio rápido. Haga clic en **Editar**.

    > [!div class="mx-imgBorder"]
    > ![Editar marca de característica Beta](./media/edit-beta-feature-flag.png)

1. En la pantalla **Editar**, seleccione el botón de opción **Activar** si aún no está seleccionado. A continuación, haga clic en el botón **Agregar filtro**. (La etiqueta del botón de opción **Activar** cambiará a **Condicional**).

1. En el campo **Clave**, escriba *Microsoft.Percentage*.

    > [!div class="mx-imgBorder"]
    > ![Agregar filtro de características](./media/feature-flag-add-filter.png)

1. Haga clic en el menú contextual situado junto a la clave de filtro de características. Haga clic en **Editar parámetros**.

    > [!div class="mx-imgBorder"]
    > ![Edición de parámetros de filtro de características](./media/feature-flag-edit-filter-parameters.png)

1. Mantenga el puntero sobre el encabezado **Nombre** para que aparezcan cuadros de texto en la cuadrícula. Escriba un **Nombre** de *Valor* y **Valor** de 50. El campo **Valor** indica el porcentaje de solicitudes para las que habilitar el filtro de características.

    > [!div class="mx-imgBorder"]
    > ![Establecimiento de parámetros de filtro de características](./media/feature-flag-set-filter-parameters.png)

1. Haga clic en **Aplicar** para volver a la pantalla **Editar marca de característica**. A continuación, haga clic en **Aplicar** de nuevo para guardar la configuración de la marca de característica.

1. El **Estado** de la marca de característica aparece ahora como *Condicional*. Este estado indica que la marca de característica se habilitará o deshabilitará en función de cada solicitud, según los criterios que el filtro de características aplique.

    > [!div class="mx-imgBorder"]
    > ![Marca de característica condicional](./media/feature-flag-filter-enabled.png)

## <a name="feature-filters-in-action"></a>Filtros de características en acción

Para ver los efectos de esta marca de característica, inicie la aplicación y presione el botón **Actualizar** en el explorador varias veces. Verá que el elemento *Beta* aparece en la barra de herramientas alrededor del 50 % del tiempo. Está oculto el resto del tiempo, porque el `PercentageFilter` desactiva la característica *Beta* para un subconjunto de solicitudes. En el vídeo siguiente se muestra este comportamiento en acción.

> [!div class="mx-imgBorder"]
> ![PercentageFilter en acción](./media/feature-flags-percentagefilter.gif)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción a la administración de características](./concept-feature-management.md)
