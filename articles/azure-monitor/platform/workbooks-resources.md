---
title: Parámetros de recursos de los libros de Azure Monitor
description: Obtenga información sobre cómo usar los parámetros de recursos para permitir la selección de recursos de los libros. Use los parámetros de recursos para establecer el ámbito del que se van a obtener los datos.
services: azure-monitor
author: mrbullwinkle
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/23/2019
ms.author: mbullwin
ms.openlocfilehash: a4d4e095e065e9f505ba1b9b46f0d31fb1783eb2
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90972861"
---
# <a name="workbook-resource-parameters"></a>Parámetros de recursos de libro

Los parámetros de recursos permiten la selección de recursos de los libros. Esto es útil a la hora de establecer el ámbito del que se van a obtener los datos. Un ejemplo sería permitir que los usuarios seleccionen el conjunto de máquinas virtuales que usarán los gráficos más tarde al presentar los datos.

Los valores de los selectores de recursos pueden provenir del contexto del libro, de la lista estática o de las consultas de Azure Resource Graph.

## <a name="creating-a-resource-parameter-workbook-resources"></a>Creación de un parámetro de recurso (recursos de libro)
1. Comience con un libro vacío en modo de edición.
2. Elija _Agregar parámetros_ desde los vínculos del libro.
3. Haga clic en el botón azul _Agregar parámetro_.
4. En el panel de nuevo parámetro que aparece, escriba:
    1. Nombre de parámetro: `Applications`
    2. Tipo de parámetro: `Resource picker`
    3. Obligatorio: `checked`
    4. Permitir selecciones múltiples: `checked`
5. Obtener datos de: `Workbook Resources`
6. Incluir solo los tipos de recursos: `Application Insights`
7. Elija "Guardar" en la barra de herramientas para crear el parámetro.

![Imagen que muestra la creación de un parámetro de recurso mediante recursos de libro](./media/workbooks-resources/resource-create.png)

## <a name="creating-a-resource-parameter-azure-resource-graph"></a>Creación de un parámetro de recurso (Azure Resource Graph)
1. Comience con un libro vacío en modo de edición.
2. Elija _Agregar parámetros_ desde los vínculos del libro.
3. Haga clic en el botón azul _Agregar parámetro_.
4. En el panel de nuevo parámetro que aparece, escriba:
    1. Nombre de parámetro: `Applications`
    2. Tipo de parámetro: `Resource picker`
    3. Obligatorio: `checked`
    4. Permitir selecciones múltiples: `checked`
5. Obtener datos de: `Query`
    1. Tipo de consulta: `Azure Resource Graph`
    2. Suscripciones: `Use default subscriptions`
    3. En el control de consulta, agregue este fragmento de código.
    ```kusto
    where type == 'microsoft.insights/components'
    | project value = id, label = name, selected = false, group = resourceGroup
    ```
7. Elija "Guardar" en la barra de herramientas para crear el parámetro.

![Imagen que muestra la creación de un parámetro de recurso mediante Azure Resource Graph](./media/workbooks-resources/resource-query.png)

> [!NOTE]
> Azure Resource Graph no está disponible en todas las nubes. Si elige este enfoque, asegúrese de que se admita en la nube de destino.

[Documentación de Azure Resource Graph](../../governance/resource-graph/overview.md)

## <a name="creating-a-resource-parameter--json-list"></a>Creación de un parámetro de recurso (lista JSON)
1. Comience con un libro vacío en modo de edición.
2. Elija _Agregar parámetros_ desde los vínculos del libro.
3. Haga clic en el botón azul _Agregar parámetro_.
4. En el panel de nuevo parámetro que aparece, escriba:
    1. Nombre de parámetro: `Applications`
    2. Tipo de parámetro: `Resource picker`
    3. Obligatorio: `checked`
    4. Permitir selecciones múltiples: `checked`
5. Obtener datos de: `JSON`
    1. En el control de contenido, agregue este fragmento de código JSON.
    ```json
    [
        { "value":"/subscriptions/<sub-id>/resourceGroups/<resource-group>/providers/<resource-type>/acmeauthentication", "label": "acmeauthentication", "selected":true, "group":"Acme Backend" },
        { "value":"/subscriptions/<sub-id>/resourceGroups/<resource-group>/providers/<resource-type>/acmeweb", "label": "acmeweb", "selected":false, "group":"Acme Frontend" }
    ]
    ```
    2. Presione el botón azul _Actualizar_.
6. Opcionalmente, establezca `Include only resource types` en _Application Insights_.
7. Elija "Guardar" en la barra de herramientas para crear el parámetro.

## <a name="referencing-a-resource-parameter"></a>Referencia a un parámetro de recurso
1. Agregue un control de consulta al libro y seleccione un recurso de Application Insights.
2. Use la lista desplegable _Application Insights_ para enlazar el parámetro al control. Con ello, se establece el ámbito de la consulta en los recursos devueltos por el parámetro en tiempo de ejecución.
4. En el control KQL, agregue este fragmento de código.
    ```kusto
    requests
    | summarize Requests = count() by appName, name
    | order by Requests desc
    ```
5. Ejecute la consulta para ver los resultados. 

![Imagen que muestra un parámetro de recurso al que se hace referencia en un control de consulta](./media/workbooks-resources/resource-reference.png)

> Este enfoque se puede usar para enlazar recursos a otros controles, como las métricas.

## <a name="resource-parameter-options"></a>Opciones de parámetros de recursos
| Parámetro | Explicación | Ejemplo |
| ------------- |:-------------|:-------------|
| `{Applications}` | Identificador del recurso seleccionado. | _/subscriptions/<sub-id>/resourceGroups/<resource-group>/providers/<resource-type>/acmeauthentication_ |
| `{Applications:label}` | Etiqueta del recurso seleccionado. | `acmefrontend` |
| `{Applications:value}` | Valor del recurso seleccionado. | _'/subscriptions/<sub-id>/resourceGroups/<resource-group>/providers/<resource-type>/acmeauthentication'_ |
| `{Applications:name}` | Nombre del recurso seleccionado. | `acmefrontend` |
| `{Applications:resourceGroup}` | Grupo de recursos del recurso seleccionado. | `acmegroup` |
| `{Applications:resourceType}` | Tipo del recurso seleccionado. | _microsoft.insights/components_ |
| `{Applications:subscription}` | Suscripción del recurso seleccionado. |  |
| `{Applications:grid}` | Cuadrícula que muestra las propiedades del recurso. Es útil para representar en un bloque de texto durante la depuración.  |  |

## <a name="next-steps"></a>Pasos siguientes

* [Comience](workbooks-visualizations.md) a aprender más sobre las muchas opciones de visualizaciones enriquecidas de los libros.
* [Controle](workbooks-access-control.md) y comparta el acceso a los recursos del libro.
