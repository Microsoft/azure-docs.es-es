---
title: Supervisión de flujos de datos de asignación
description: Cómo supervisar visualmente los flujos de datos de asignación en Azure Data Factory
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/19/2020
ms.openlocfilehash: 77dda42b27aa6f5fb505fe65667876523cb3f5d2
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88650900"
---
# <a name="monitor-data-flows"></a>Supervisión de flujos de datos

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Después de haber completado la compilación y depuración del flujo de datos, desea programarlo para ejecutarse según una programación en el contexto de una canalización. Puede programar la canalización de Azure Data Factory mediante desencadenadores. También puede usar la opción Desencadenar ahora desde el generador de canalización de Azure Data Factory para ejecutar una sola ejecución con el fin de probar el flujo de datos dentro del contexto de canalización.

Cuando se ejecuta la canalización, puede supervisar la canalización y todas las actividades contenidas en la canalización, incluida la actividad de Data Flow. Haga clic en el icono de supervisión en el panel izquierdo de la interfaz de usuario de Azure Data Factory. Puede ver una pantalla similar a la siguiente. Los iconos resaltados permiten profundizar en las actividades de la canalización, incluidas las actividades de Data Flow.

![Supervisión de Data Flow](media/data-flow/mon001.png "Supervisión de Data Flow")

Se muestran estadísticas en este nivel, así como los tiempos de ejecución y estado. El identificador de ejecución en el nivel de actividad es diferente al del nivel de canalización. El identificador de ejecución en el nivel anterior es para la canalización. Al seleccionar el icono de las gafas, se obtienen detalles de la ejecución del flujo de datos.

![Supervisión de Data Flow](media/data-flow/monitoring-details.png "Supervisión de Data Flow")

En la vista de supervisión del nodo gráfico, puede ver una versión simplificada de solo lectura del gráfico de flujo de datos.

![Supervisión de Data Flow](media/data-flow/mon003.png "Supervisión de Data Flow")

Este es un vídeo de información general sobre la supervisión del rendimiento de los flujos de datos desde la pantalla de supervisión de ADF:

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4u4mH]

## <a name="view-data-flow-execution-plans"></a>Visualización de planes de ejecución de Data Flow

Cuando se ejecuta Data Flow en Spark, Azure Data Factory determina las rutas de acceso de código óptimo según la integridad del flujo de datos. Además, las rutas de ejecución pueden producirse en distintos nodos de escalabilidad horizontal y particiones de datos. Por lo tanto, el gráfico de supervisión representa el diseño del flujo, teniendo en cuenta la ruta de acceso de ejecución de las transformaciones. Al seleccionar los nodos individuales, puede ver "agrupaciones" que representan el código que se ejecutó juntos en el clúster. Los intervalos y recuentos que ve representan esos grupos en lugar de los pasos individuales del diseño.

![Supervisión de Data Flow](media/data-flow/mon004.png "Supervisión de Data Flow")

* Al seleccionar el espacio abierto de la ventana de supervisión, las estadísticas del panel inferior muestran los recuentos de filas y el tiempo de cada receptor y las transformaciones que dieron lugar a los datos de receptor de linaje de transformación.

* Cuando se seleccionan transformaciones individuales, recibe más comentarios en el panel derecho que muestra estadísticas de partición, recuentos de columna, sesgo (con qué uniformidad se distribuyen los datos entre particiones) y curtosis (cuántos picos tienen los datos).

* Al seleccionar el receptor en la vista del nodo, puede ver el linaje de columna. Hay tres métodos diferentes que las columnas acumulan a lo largo de su flujo de datos para colocarse en el receptor. Son las siguientes:

  * Calculado: Use la columna para el procesamiento condicional o dentro de una expresión en el flujo de datos, pero no la coloque en el receptor.
  * Derivado: La columna es una columna nueva que generó en el flujo, es decir, no estaba presente en el origen.
  * Asignada: La columna se origina desde el origen y la asigna a un campo de receptor.
  * Estado del flujo de datos: el estado actual de la ejecución.
  * Tiempo de inicio del clúster: la cantidad de tiempo necesaria para adquirir el entorno de proceso de Spark JIT para la ejecución del flujo de datos.
  * Número de transformaciones: el número de pasos de transformación que se ejecutan en el flujo
  
![Supervisión de Data Flow](media/data-flow/monitornew.png "Nueva supervisión de Data Flow")

## <a name="total-sink-processing-time-vs-transformation-processing-time"></a>Diferencias entre tiempo total de procesamiento de receptores y tiempo de procesamiento de la transformación

Cada fase de transformación incluye un tiempo total para que se complete, y el tiempo de ejecución de todas las particiones se suma. Al hacer clic en el receptor, verá "Tiempo de procesamiento del receptor". Este valor incluye el total del tiempo de transformación *más* el tiempo de E/S que se tarda en escribir los datos en el almacén de destino. La diferencia entre el tiempo de procesamiento del receptor y el total de la transformación es el tiempo de E/S para escribir los datos.

También puede ver el tiempo detallado del paso de transformación de cada partición si abre la salida JSON desde la actividad de flujo de datos en la vista de supervisión de la canalización de ADF. El archivo JSON contiene el tiempo en milisegundos de cada partición, mientras que la vista de supervisión de la experiencia de usuario es un intervalo agregado de las particiones sumadas:

```
 {
     "stage": 4,
     "partitionTimes": [
          14353,
          14914,
          14246,
          14912,
          ...
         ]
}
```

### <a name="post-processing-time"></a>Tiempo de posprocesamiento

Cuando seleccione un icono de transformación del receptor en el mapa, el panel deslizante de la derecha mostrará un punto de datos adicional denominado "Tiempo de posprocesamiento" en la parte inferior. Esta es la cantidad de tiempo que se dedica a ejecutar el trabajo en el clúster de Spark *después* de cargar, transformar y escribir los datos. Esta cantidad puede incluir el cierre de grupos de conexiones, el apagado de controladores, la eliminación de archivos, la fusión de archivos, etc. Al realizar acciones en el flujo como "migrar archivos" y "enviar a un solo archivo", es probable que se muestre un aumento en el valor de tiempo de posprocesamiento.
  
## <a name="monitor-icons"></a>Iconos de supervisión

Este icono significa que los datos de transformación se almacenaron en caché en el clúster, por lo que los intervalos y la ruta de acceso de ejecución se han tenido en cuenta:

![Supervisión de Data Flow](media/data-flow/mon005.png "Supervisión de Data Flow")

También ve los iconos de círculo verde en la transformación. Representan un recuento del número de receptores en los que fluyen los datos.
