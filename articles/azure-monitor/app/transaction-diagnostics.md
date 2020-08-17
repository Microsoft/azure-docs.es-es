---
title: Diagnósticos de transacción de Azure Application Insights | Microsoft Docs
description: Diagnósticos de transacción de extremo a extremo de Application Insights
ms.topic: conceptual
ms.date: 01/19/2018
ms.reviewer: sdash
ms.openlocfilehash: 552e80d6c90a18180ab41dce72e995b6804a91bc
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318919"
---
# <a name="unified-cross-component-transaction-diagnostics"></a>Diagnósticos de transacción entre componentes unificados

La experiencia de diagnóstico unificada correlaciona automáticamente la telemetría del lado servidor con todos los componentes supervisados de Application Insights en una única visualización. No importa si tiene varios recursos con claves de instrumentación independientes. Application Insights detecta la relación subyacente y le permite diagnosticar fácilmente el componente, la dependencia o la excepción de la aplicación que provocaron una ralentización o un error en la transacción.

## <a name="what-is-a-component"></a>¿Qué es un componente?

Los componentes son partes que se pueden implementar independientemente de su aplicación de microservicios o distribuida. Los equipos de operaciones y los desarrolladores pueden ver el código o acceder a la telemetría que generan estos componentes de la aplicación.

* Los componentes son diferentes de las dependencias externas "observadas", como SQL, EventHub, etc. a las que su equipo u organización no pueden acceder (código o telemetría).
* Los componentes se ejecutan en cualquier número de instancias de rol, servidor o contenedor.
* Los componentes pueden ser claves de instrumentación de Application Insights independientes (incluso aunque las suscripciones sean diferentes) o diferentes roles que informan a una única clave de instrumentación de Application Insights. La nueva experiencia muestra los detalles en todos los componentes, independientemente de cómo se hayan configurado.

> [!NOTE]
> * **¿Faltan los vínculos a elementos relacionados?** Todos los datos de telemetría relacionados se encuentran en las secciones [superior](#cross-component-transaction-chart) e [inferior](#all-telemetry-with-this-operation-id) del lado izquierdo. 

## <a name="transaction-diagnostics-experience"></a>Experiencia de diagnósticos de la transacción
Esta vista tiene cuatro partes principales: una lista de resultados, un gráfico de transacción entre componentes, una lista de secuencia de tiempo de todos los datos de telemetría relacionados con esta operación y el panel de detalles de cualquier elemento de telemetría seleccionado a la izquierda.

![Elementos clave](media/transaction-diagnostics/4partsCrossComponent.png)

## <a name="cross-component-transaction-chart"></a>Gráfico de transacciones entre componentes

Este gráfico proporciona una escala de tiempo con barras horizontales que muestra la duración de las solicitudes y las dependencias entre los componentes. Las excepciones que se recopilan también se marcan en la escala de tiempo.

* La fila superior de este gráfico representa el punto de entrada, la solicitud entrante al primer componente que se llamó en esta transacción. La duración es el tiempo total necesario para que se complete la transacción.
* Las llamadas a las dependencias externas son filas sencillas que no se pueden contraer, con iconos que representan el tipo de dependencia.
* Las llamadas a otros componentes son filas que se pueden contraer. Cada fila corresponde a una operación específica que se invoca en el componente.
* De forma predeterminada, la solicitud, la dependencia o la excepción seleccionadas se muestran en el lado derecho.
* Seleccione cualquier fila para ver sus [detalles a la derecha](#details-of-the-selected-telemetry). 

> [!NOTE]
> Las llamadas a otros componentes tienen dos filas: una fila representa la llamada saliente (dependencia) desde el componente del llamador, y la otra fila corresponde a la solicitud entrante al componente del que recibe la llamada. El icono inicial y los distintos estilos de las barras de duración le ayudan a diferenciarlos.

## <a name="all-telemetry-with-this-operation-id"></a>Todos los datos de telemetría con este identificador de operación

En esta sección se muestra la vista de lista plana en una secuencia de tiempo de todos los datos de telemetría relacionados con esta transacción. También se muestran los eventos personalizados y los seguimientos que no se muestran en el gráfico de transacciones. Puede filtrar esta lista por los datos de telemetría generados por una llamada o componente específicos. Puede seleccionar cualquier elemento de telemetría en esta lista para ver sus [detalles a la derecha](#details-of-the-selected-telemetry).

![Secuencia de tiempo de todos los datos de telemetría](media/transaction-diagnostics/allTelemetryDrawerOpened.png)

## <a name="details-of-the-selected-telemetry"></a>Detalles de la telemetría seleccionada

En este panel contraíble se muestran los detalles de cualquier elemento seleccionado del gráfico de transacciones o de la lista. "Mostrar todo" enumera todos los atributos estándares recopilados. Los atributos personalizados se muestran por separado después del conjunto estándar. Haga clic en "..." debajo de la ventana de seguimiento de la pila para obtener una opción para copiar el seguimiento. Las opciones "Open profiler traces" (Abrir seguimientos de Profiler) o "Abrir instantánea de depuración" muestran los diagnósticos de nivel de código en los paneles de detalles correspondientes.

![Detalle de la excepción](media/transaction-diagnostics/exceptiondetail.png)

## <a name="search-results"></a>Search Results

En este panel contraíble se muestran los otros resultados que cumplen los criterios de filtro. Haga clic en cualquier resultado para actualizar los detalles correspondientes en las tres secciones enumeradas anteriormente. Intentamos buscar ejemplos con más probabilidades de tener detalles disponibles de todos los componentes, aunque el muestreo esté activo en cualquiera de ellos. Se muestran como ejemplos "sugeridos".

![Search Results](media/transaction-diagnostics/searchResults.png)

## <a name="profiler-and-snapshot-debugger"></a>Profiler y Snapshot Debugger

[Application Insights Profiler](./profiler.md) y [Snapshot Debugger](snapshot-debugger.md) le ayudan a realizar diagnósticos al nivel de código para conocer los problemas de rendimiento y errores. Con esta experiencia, podrá ver seguimientos de Profiler o instantáneas de cualquier componente con tan solo un clic.

Si no pudo conseguir que Profiler funcionara, póngase en contacto con **serviceprofilerhelp\@microsoft.com**.

Si no pudo conseguir que Snapshot Debugger funcionara, póngase en contacto con **snapshothelp\@microsoft.com**.

![Integración del generador de perfiles](media/transaction-diagnostics/profilerTraces.png)

## <a name="faq"></a>Preguntas más frecuentes

*Solo veo un componente en el gráfico y el resto solo se muestran como dependencias externas sin detalles acerca de lo que sucedió dentro de esos componentes.*

Posibles razones:

* ¿Existen otros componentes instrumentados con Application Insights?
* ¿Usa la versión más reciente y estable de SDK de Application Insights?
* Si estos componentes son recursos independientes de Application Insights, ¿tiene el acceso requerido para su telemetría?

Si tiene acceso y los componentes se instrumentan con el SDK más reciente de Application Insights, háganoslo saber a través del canal de comentarios en la parte superior derecha.

*Veo filas duplicadas para las dependencias. ¿Es normal?*

Por ahora, presentamos la llamada de dependencia de salida de forma independiente de la solicitud entrante. Por lo general, las dos llamadas son idénticas y solo difiere el valor de duración debido al recorrido de ida y vuelta. El icono inicial y los distintos estilos de las barras de duración le ayudan a diferenciarlos. ¿Le resulta confusa esta presentación de los datos? Envíenos sus comentarios.

*¿Qué hay de los desfases temporales entre las instancias de los diferentes componentes?*

Las escalas de tiempo se ajustan para los desfases temporales en el gráfico de transacciones. Puede ver las marcas de tiempo exactas en el panel de detalles o con Analytics.

*¿Por qué faltan en la nueva experiencia la mayoría de las consultas de elementos relacionadas?*

es así por diseño. Todos los elementos relacionados, en todos los componentes, están disponibles en el lado izquierdo (secciones superior e inferior). La nueva experiencia tiene dos elementos relacionados que no se tratan en el lado izquierdo: todos los datos de telemetría de los cinco minutos anteriores y posteriores a este evento y la escala de tiempo del usuario.

