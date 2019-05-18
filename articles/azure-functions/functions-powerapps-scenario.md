---
title: Llamada a una función desde PowerApps | Microsoft Docs
description: Cree un conector personalizado y después llame a una función mediante ese conector.
services: functions
keywords: aplicaciones en la nube, servicios en la nube, PowerApps, procesos empresariales, aplicación empresarial
author: ggailey777
manager: jeconnoc
ms.assetid: ''
ms.service: azure-functions
ms.topic: conceptual
ms.date: 12/14/2017
ms.author: glenga
ms.reviewer: sunayv
ms.custom: ''
ms.openlocfilehash: 26f6502f63b39d3f1ecf8dfeb09c8df4daa63b68
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65786105"
---
# <a name="call-a-function-from-powerapps"></a>Llamada a una función desde PowerApps
La plataforma [PowerApps](https://powerapps.microsoft.com) está diseñada para que expertos empresariales creen aplicaciones sin código de aplicación tradicional. Los desarrolladores profesionales pueden usar Azure Functions para ampliar las capacidades de PowerApps, evitándoles los detalles técnicos a los compiladores de aplicaciones de PowerApps.

En este tema compilará una aplicación basada en un escenario de mantenimiento para turbinas eólicas. En este tema, se muestra cómo llamar a la función que ha definido en [Create an OpenAPI definition for a function](functions-openapi-definition.md) (Creación de una definición de OpenAPI para una función). La función determina si resulta rentable una reparación de emergencia en una turbina eólica.

![Aplicación finalizada en PowerApps](media/functions-powerapps-scenario/finished-app.png)

Para obtener información acerca de cómo llamar a la misma función desde Microsoft Flow, consulte [Call a function from Microsoft Flow](functions-flow-scenario.md) (Llamar a una función desde Microsoft Flow).

En este tema, aprenderá cómo:

> [!div class="checklist"]
> * Preparar datos de ejemplo en Excel.
> * Exportar una definición de API.
> * Agregar una conexión a la API.
> * Crear una aplicación y agregar orígenes de datos.
> * Agregar controles para ver los datos en la aplicación.
> * Agregar controles para llamar a la función y mostrar los datos.
> * Ejecutar la aplicación para determinar si una reparación es rentable.

[!INCLUDE [functions-openapi-note](../../includes/functions-openapi-note.md)]

## <a name="prerequisites"></a>Requisitos previos

+ Una [cuenta de PowerApps](https://docs.microsoft.com/powerapps/maker/signup-for-powerapps) activa con las mismas credenciales de inicio de sesión que su cuenta de Azure. 
+ Excel y el [archivo de Excel de ejemplo](https://procsi.blob.core.windows.net/docs/turbine-data.xlsx) que va a utilizar como origen de datos para la aplicación.
+ Complete el tutorial [Create an OpenAPI definition for a function](functions-openapi-definition.md) (Creación de una definición de OpenAPI para una función).

[!INCLUDE [Export an API definition](../../includes/functions-export-api-definition.md)]

## <a name="add-a-connection-to-the-api"></a>Agregar una conexión a la API
La API personalizada (también conocida como conector personalizado) está disponible en PowerApps, pero se debe establecer una conexión a la API para poder usarla en una aplicación.

1. En [web.powerapps.com](https://web.powerapps.com), haga clic en **Conexiones**.

    ![Conexiones de PowerApps](media/functions-powerapps-scenario/powerapps-connections.png)

1. Haga clic en **Nueva conexión**, desplácese hacia abajo hasta el conector **Turbine Repair** (Reparación de turbinas) y haga clic en él.

    ![Nueva conexión](media/functions-powerapps-scenario/new-connection.png)

1. Escriba la clave de API y haga clic en **Crear**.

    ![Crear conexión](media/functions-powerapps-scenario/create-connection.png)

> [!NOTE]
> Si comparte la aplicación con otros usuarios, todas las personas que trabajen en la aplicación o la usen también tendrán que escribir la clave de API para conectarse a la API. Este comportamiento podría cambiar en el futuro y, en ese caso, este tema se actualizaría para reflejar el cambio.

## <a name="create-an-app-and-add-data-sources"></a>Crear una aplicación y agregar orígenes de datos
Ahora está listo para crear la aplicación en PowerApps y agregar los datos de Excel y la API personalizada como orígenes de datos de la aplicación.

1. En [web.powerapps.com](https://web.powerapps.com), elija **Iniciar desde cero** > ![Icono de aplicación para teléfono](media/functions-powerapps-scenario/icon-phone-app.png) (teléfono) > **Crear esta aplicación**.

    ![Iniciar desde cero - aplicación de teléfono](media/functions-powerapps-scenario/create-phone-app.png)

    La aplicación se abre en PowerApps Studio para web. En la siguiente imagen, se muestran las distintas partes de PowerApps Studio.

    ![PowerApps Studio](media/functions-powerapps-scenario/powerapps-studio.png)

    **(A) Barra de navegación izquierda**, en la que se ve una vista jerárquica de todos los controles de cada pantalla

    **(B) Panel central**, que muestra la pantalla en la que está trabajando

    **(C) Panel derecho**, donde se establecen opciones como los orígenes de datos y el diseño

    **(D) Propiedades**, lista desplegable donde se seleccionan las propiedades a las que se aplican las fórmulas

    **(E) Barra de fórmulas**, donde se agregan fórmulas (como en Excel) que definen el comportamiento de la aplicación
    
    **(F) Cinta**, donde se pueden agregar controles y personalizar elementos de diseño

1. Agregue el archivo de Excel como origen de datos.

    Los datos que va a importar tienen el siguiente aspecto:

    ![Datos de Excel para importar](media/functions-powerapps-scenario/excel-table.png)

    1. En el lienzo de la aplicación, elija **conectar a datos**.

    1. En el panel **Datos** haga clic en **Agregar datos estáticos a la aplicación**.

        ![Agregar origen de datos](media/functions-powerapps-scenario/add-static-data.png)

        Normalmente, leería y escribiría datos desde un origen externo, pero está agregando los datos de Excel como datos estáticos porque se trata de un ejemplo.

    1. Vaya al archivo de Excel que ha guardado, seleccione la tabla **Turbines** (Turbinas) y haga clic en **Conectar**.

        ![Agregar origen de datos](media/functions-powerapps-scenario/choose-table.png)


1. Agregue la API personalizada como origen de datos.

    1. En el panel **Datos**, haga clic en **Agregar origen de datos**.

    1. Haga clic en **Turbine Repair** (Reparación de turbinas).

        ![Conector de Turbine Repair (Reparación de turbinas)](media/functions-powerapps-scenario/turbine-connector.png)

## <a name="add-controls-to-view-data-in-the-app"></a>Agregar controles para ver los datos en la aplicación
Ahora que los orígenes de datos están disponibles en la aplicación, agregue una pantalla a la aplicación para que pueda ver los datos de las turbinas.

1. En la pestaña **Inicio**, haga clic en **Nueva pantalla** > **Pantalla de lista**.

    ![Pantalla de lista](media/functions-powerapps-scenario/list-screen.png)

    PowerApps agrega una pantalla que contiene una *galería* para mostrar elementos y otros controles que permiten buscar, ordenar y filtrar.

1. Cambie la barra de título por `Turbine Repair` y cambie el tamaño de la galería, de modo que haya espacio para más controles debajo.

    ![Cambiar el título y cambiar el tamaño de la galería](media/functions-powerapps-scenario/gallery-title.png)

1. Con la galería seleccionada, en el panel derecho, en **Propiedades**, haga clic en **CustomGallerySample**.

    ![Cambiar origen de datos](media/functions-powerapps-scenario/change-data-source.png)

1. En el panel **Datos**, seleccione **Turbinas** en la lista.

    ![Selección de origen de datos](media/functions-powerapps-scenario/select-data-source.png)

    El conjunto de datos no contiene ninguna imagen, así que cambie el diseño para que se ajuste mejor a los datos. 

1. Todavía en el panel **Datos**, cambie **Diseño** por **Título, subtítulo y cuerpo**.

    ![Cambiar el diseño de la galería](media/functions-powerapps-scenario/change-layout.png)

1. Como último paso en el panel **Datos**, cambie los campos que se muestran en la galería.

    ![Cambiar los campos de la galería](media/functions-powerapps-scenario/change-fields.png)
    
    + **Body1** = LastServiceDate
    + **Subtitle2** = ServiceRequired
    + **Title2** = Title 

1. Con la galería seleccionada, establezca la propiedad **TemplateFill** en la fórmula siguiente: `If(ThisItem.IsSelected, Orange, White)`.

    ![Fórmula de relleno de plantilla](media/functions-powerapps-scenario/formula-fill.png)

    Ahora es más fácil ver qué elemento de la galería está seleccionado.

    ![Elemento seleccionado](media/functions-powerapps-scenario/selected-item.png)

1. No necesita la pantalla original en la aplicación. En el panel izquierdo, mantenga el puntero sobre **Screen1**, haga clic en **. . .** y **Eliminar**.

    ![Eliminar pantalla](media/functions-powerapps-scenario/delete-screen.png)

1. Haga clic en **Archivo**y dele un nombre a la aplicación. Haga clic en **Guardar** en el menú izquierdo y, a continuación, haga clic en **Guardar** en la esquina inferior derecha.

Hay muchas otras características de formato que realizaría normalmente en una aplicación de producción, pero pasaremos a la parte importante de este escenario: llamar a la función.

## <a name="add-controls-to-call-the-function-and-display-data"></a>Agregar controles para llamar a la función y mostrar los datos
Tiene una aplicación que muestra datos de resumen de cada turbina, así que es el momento de agregar controles que llamen a la función que ha creado y muestren los datos que se devuelven. Se obtiene acceso a la función según la forma en que se denomina en la definición de OpenAPI; en este caso, es `TurbineRepair.CalculateCosts()`.

1. En la cinta, en la pestaña **Insertar**, haga clic en **Botón**. Después, en la misma pestaña, haga clic en **Etiqueta**

    ![Botón Insertar y etiqueta](media/functions-powerapps-scenario/insert-controls.png)

1. Arrastre el botón y la etiqueta debajo de la galería y cambie el tamaño de la etiqueta. 

1. Seleccione el texto del botón y cámbielo por `Calculate costs`. La aplicación debería ser similar a la siguiente imagen.

    ![Aplicación con botón](media/functions-powerapps-scenario/move-button-label.png)

1. Seleccione el botón y escriba la fórmula siguiente en la propiedad **OnSelect** del botón.

    ```
    If (BrowseGallery1.Selected.ServiceRequired="Yes", ClearCollect(DetermineRepair, TurbineRepair.CalculateCosts({hours: BrowseGallery1.Selected.EstimatedEffort, capacity: BrowseGallery1.Selected.MaxOutput})))
    ```
    Esta fórmula se ejecuta cuando se hace clic en el botón y hace lo siguiente si el elemento de la galería seleccionado tiene un valor **ServiceRequired** de `Yes`:

    + Borra la *colección* `DetermineRepair` para quitar datos de llamadas anteriores. Una colección es una variable tabular.

    + Asigna a la colección los datos devueltos al llamar a la función `TurbineRepair.CalculateCosts()`. 
    
        Los valores pasados a la función proceden de los campos **EstimatedEffort** y **MaxOutput** del elemento seleccionado en la galería. Estos campos no se muestran en la galería, pero están disponibles para usarlos en las fórmulas.

1. Seleccione la etiqueta y escriba la fórmula siguiente en la propiedad **Text** de la etiqueta.

    ```
    "Repair decision: " & First(DetermineRepair).message & " | Cost: " & First(DetermineRepair).costToFix & " | Revenue: " & First(DetermineRepair).revenueOpportunity
    ```
    Esta fórmula usa la función `First()` para obtener acceso a la primera (y única) fila de la colección `DetermineRepair`. Después, muestra los tres valores que devuelve la función: `message`, `costToFix` y `revenueOpportunity`. Estos valores están en blanco antes de que la aplicación se ejecute por primera vez.

    La aplicación completa debería ser similar a la siguiente imagen.

    ![Aplicación finalizada antes de ejecutar](media/functions-powerapps-scenario/finished-app-before-run.png)


## <a name="run-the-app"></a>Ejecución de la aplicación
Tiene una aplicación completa. Ahora es el momento de ejecutarla y ver las llamadas de la función en acción.

1. En la esquina superior derecha de PowerApps Studio, haga clic en el botón de ejecución: ![Botón de ejecución de aplicación](media/functions-powerapps-scenario/f5-arrow-sm.png).

1. Seleccione una turbina con un valor de `Yes` para **ServiceRequired** y después haga clic en el botón **Calculate costs** (Calcular costos). Debería ver un resultado parecido al de la siguiente imagen.

    ![Aplicación finalizada en PowerApps](media/functions-powerapps-scenario/finished-app.png)

1. Pruebe las demás turbinas para ver qué devuelve la función cada vez.

## <a name="next-steps"></a>Pasos siguientes
En este tema, ha aprendido cómo:

> [!div class="checklist"]
> * Preparar datos de ejemplo en Excel.
> * Exportar una definición de API.
> * Agregar una conexión a la API.
> * Crear una aplicación y agregar orígenes de datos.
> * Agregar controles para ver los datos en la aplicación.
> * Agregar controles para llamar a la función y mostrar los datos
> * Ejecutar la aplicación para determinar si una reparación es rentable.

Para obtener más información sobre PowerApps, consulte [Introducción a PowerApps](https://powerapps.microsoft.com/tutorials/getting-started/).

Para obtener información acerca de otro escenario interesante que usa Azure Functions, consulte [Call a function from Microsoft Flow](functions-flow-scenario.md) (Llamar a una función desde Microsoft Flow) y [Creación de una función que se integre con Azure Logic Apps](functions-twitter-email.md).
