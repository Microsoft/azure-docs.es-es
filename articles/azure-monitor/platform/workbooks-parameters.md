---
title: Parámetros de creación de los libros de Azure Monitor
description: Obtenga información sobre cómo los parámetros permiten a los autores de libros recopilar la entrada de los consumidores y hacer referencia a ella en otras partes del libro.
services: azure-monitor
author: mrbullwinkle
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/23/2019
ms.author: mbullwin
ms.openlocfilehash: 8be7dd170d5e4e3bddb09bc1b163fba7a841a6b7
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90984563"
---
# <a name="workbook-parameters"></a>Parámetros de libro

Los parámetros permiten a los autores de libros recopilar la entrada de los consumidores y hacer referencia a ella en otras partes del libro, normalmente para definir el ámbito del conjunto de resultados o establecer el objeto visual adecuado. Es una funcionalidad clave que permite a los autores crear experiencias e informes interactivos. 

Los libros permiten determinar el modo en que se presentan los controles de parámetro a los consumidores: cuadro de texto frente a lista desplegable, selección única frente a selección múltiple, valores de texto, JSON, KQL o Azure Resource Graph.  

Entre los tipos de parámetros admitidos se incluyen:
* [Tiempo](workbooks-time.md): permite a los usuarios seleccionar entre intervalos de tiempo rellenados previamente o seleccionar un intervalo personalizado.
* [Lista desplegable](workbooks-dropdowns.md): permite a los usuarios seleccionar un valor o un conjunto de valores.
* [Texto](workbooks-text.md): permite a los usuarios escribir texto arbitrario.
* [Recurso](workbooks-resources.md): permite a los usuarios seleccionar uno o varios recursos de Azure.
* [Suscripción](workbooks-resources.md): permite a los usuarios seleccionar uno o varios recursos de suscripción de Azure.
* Tipo de recurso: permite a los usuarios seleccionar uno o varios valores de tipo de recurso de Azure.
* Ubicación: permite a los usuarios seleccionar uno o varios valores de ubicación de Azure.

Se puede hacer referencia a estos valores de parámetro en otras partes de los libros mediante enlaces o expansiones de valores.

## <a name="creating-a-parameter"></a>Creación de un parámetro
1. Comience con un libro vacío en modo de edición.
2. Elija _Agregar parámetros_ desde los vínculos del libro.
3. Haga clic en el botón azul _Agregar parámetro_.
4. En el panel de nuevo parámetro que aparece, escriba:
    1. Nombre del parámetro: `TimeRange` *(tenga en cuenta que los __nombres__ de parámetros **no pueden** incluir espacios ni caracteres especiales)*
    2. Nombre para mostrar: `Time Range` *(sin embargo, los __nombres para mostrar__ pueden incluir espacios, caracteres especiales, emoji, etc.)*
    2. Tipo de parámetro: `Time range picker`
    3. Obligatorio: `checked`
    4. Intervalos de tiempo disponibles: Última hora, Últimas 12 horas, Últimas 24 horas, Últimas 48 horas, Últimos 3 días, Últimos 7 días y Permitir la selección de intervalos de tiempo personalizados
5. Elija "Guardar" en la barra de herramientas para crear el parámetro.

   ![Imagen que muestra la creación de un parámetro de intervalo de tiempo](./media/workbooks-parameters/time-settings.png)

Este es el aspecto que tendrá el libro en modo de lectura, en el estilo "Pills".

   ![Imagen que muestra un parámetro de intervalo de tiempo en modo de lectura](./media/workbooks-parameters/parameters-time.png)

## <a name="referencing-a-parameter"></a>Referencia a un parámetro
### <a name="via-bindings"></a>Mediante enlaces
1. Agregue un control de consulta al libro y seleccione un recurso de Application Insights.
2. Abra la lista desplegable _Intervalo de tiempo_ y seleccione la opción `Time Range` en la sección de parámetros de la parte inferior.
3. Esta acción enlaza el parámetro de intervalo de tiempo con el intervalo de tiempo del gráfico. El ámbito de tiempo de la consulta de ejemplo es ahora Últimas 24 horas.
4. Ejecución de la consulta para ver los resultados

    ![Imagen que muestra un parámetro de intervalo de tiempo al que se hace referencia mediante enlaces](./media/workbooks-parameters/time-binding.png)

### <a name="in-kql"></a>En KQL
1. Agregue un control de consulta al libro y seleccione un recurso de Application Insights.
2. En KQL, escriba un filtro de ámbito de tiempo mediante el parámetro: `| where timestamp {TimeRange}`.
3. El tiempo de evaluación de consulta se expande a `| where timestamp > ago(1d)`, que es el valor del intervalo de tiempo del parámetro.
4. Ejecución de la consulta para ver los resultados

    ![Imagen que muestra un intervalo de tiempo al que se hace referencia en KQL](./media/workbooks-parameters/time-in-code.png)

### <a name="in-text"></a>En texto 
1. Agregue un control de texto al libro.
2. En el Markdown, escriba `The chosen time range is {TimeRange:label}`.
3. Elija _Edición finalizada_.
4. El control de texto mostrará el texto: _El intervalo de tiempo elegido es Últimas 24 horas_

## <a name="parameter-options"></a>Opciones de parámetro
En la sección _In Text_ se ha usado el elemento `label` del parámetro en lugar de su valor. Los parámetros exponen varias de estas opciones en función de su tipo; por ejemplo, los selectores de intervalo de tiempo permiten valor, etiqueta, consulta, inicio, fin y grano.

Use la sección `Previews` del panel _Editar parámetro_ para ver las opciones de expansión del parámetro:

![Imagen que muestra las opciones de un parámetro de intervalo de tiempo](./media/workbooks-parameters/time-previews.png)

## <a name="next-steps"></a>Pasos siguientes

* [Comience](workbooks-visualizations.md) a aprender más sobre las muchas opciones de visualizaciones enriquecidas de los libros.
* [Controle](workbooks-access-control.md) y comparta el acceso a los recursos del libro.
