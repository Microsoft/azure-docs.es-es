---
title: Asignación de flujos de datos
description: Introducción a los flujos de datos de asignación en Azure Data Factory
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 06/09/2020
ms.openlocfilehash: 850879675d4554329f24c86f2ac28660b303084c
ms.sourcegitcommit: 5f7b75e32222fe20ac68a053d141a0adbd16b347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87475573"
---
# <a name="what-are-mapping-data-flows"></a>¿Qué son los flujos de datos de asignación?

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

La asignación de flujos de datos es una transformación de datos diseñada visualmente en Azure Data Factory. Los flujos de datos permiten a los ingenieros de datos desarrollar una lógica de transformación de datos gráfica sin necesidad de escribir código. Los flujos de datos resultantes se ejecutan como actividades en las canalizaciones de Azure Data Factory que usan clústeres de Apache Spark con escalabilidad horizontal. Las actividades de flujo de datos se pueden captar mediante las funcionalidades de programación, control, flujo y supervisión existentes en Data Factory.

Los flujos de datos de asignación proporcionan una experiencia completamente visual que no requiere programación. Los flujos de datos se ejecutan en el clúster de ejecución durante el procesamiento de datos de escalabilidad horizontal. Asimismo, Azure Data Factory controla toda la traducción de código, la optimización de rutas de acceso y la ejecución de los trabajos de flujo de datos.

![Architecture](media/data-flow/adf-data-flows.png "Architecture")

## <a name="getting-started"></a>Introducción

Para crear un flujo de datos, seleccione el signo más en **Factory Resources** (Recursos de fábrica) y, a continuación, seleccione **Data Flow**. 

![Nuevo flujo de datos](media/data-flow/newdataflow2.png "nuevo flujo de datos")

Con esta acción, accederá al lienzo de flujo de datos, donde podrá crear la lógica de transformación. Seleccione **Agregar origen** para comenzar a configurar la transformación de origen. Para más información, consulte [Transformación de origen](data-flow-source.md).

## <a name="data-flow-canvas"></a>Lienzo de flujo de datos

El lienzo de flujo de datos está dividido en tres partes: la barra superior, el gráfico y el panel de configuración. 

![Lienzo](media/data-flow/canvas1.png "Lienzo")

### <a name="graph"></a>Grafo

En el gráfico se muestra el flujo de transformación. Muestra el linaje de los datos de origen a medida que fluyen hacia uno o varios receptores. Para agregar un nuevo origen, seleccione **Agregar origen**. Para agregar una nueva transformación, seleccione el signo más situado en la parte inferior derecha de una transformación existente.

![Lienzo](media/data-flow/canvas2.png "Lienzo")

### <a name="azure-integration-runtime-data-flow-properties"></a>Propiedades de flujo de datos de Azure Integration Runtime

![Botón Depurar](media/data-flow/debugbutton.png "Botón Depurar")

Al empezar a trabajar con flujos de datos en ADF, querrá activar el modificador "Depurar" para los flujos de datos en la parte superior de la interfaz de usuario del explorador. Esto pone a punto un clúster de Spark que se usa para la depuración interactiva, las vistas previas de datos y las ejecuciones de depuración de canalizaciones. Para establecer el tamaño del clúster que se está usando, elija una opción de [Azure Integration Runtime](concepts-integration-runtime.md) personalizada. La sesión de depuración permanece activa hasta 60 minutos después de la última ejecución de la canalización de depuración o la última vista previa de datos.

Al poner en marcha las canalizaciones con actividades de flujo de datos, ADF usa las instancias de Azure Integration Runtime asociadas a la [actividad](control-flow-execute-data-flow-activity.md) de la propiedad "Ejecutar el".

La instancia predeterminada de Azure Integration Runtime es un único clúster de nodos de trabajo de 4 núcleos diseñado para permitirle obtener una vista previa de los datos y ejecutar rápidamente las canalizaciones de depuración con unos costos mínimos. Establezca una configuración de Azure IR mayor si va a realizar operaciones en conjuntos de datos de gran tamaño.

Puede indicar a ADF que mantenga un grupo de recursos de clúster (VM). Para ello, establezca un TTL en las propiedades de flujo de datos de Azure IR. Esta acción dará lugar a una ejecución más rápida del trabajo en las actividades posteriores.

#### <a name="azure-integration-runtime-and-data-flow-strategies"></a>Estrategias de Azure Integration Runtime y flujos de datos

##### <a name="execute-data-flows-in-parallel"></a>Ejecutar flujos de datos en paralelo

Si ejecuta flujos de datos en una canalización en paralelo, ADF pone a punto clústeres de Spark independientes para cada ejecución de actividad en función de la configuración de la instancia de Azure Integration Runtime asociada a cada actividad. Para diseñar ejecuciones en paralelo en canalizaciones de ADF, agregue sus actividades de flujo de datos sin restricciones de precedencia en la interfaz de usuario.

De estas tres opciones, esta es la que probablemente se ejecuta en menos tiempo. Sin embargo, cada flujo de datos paralelo se ejecuta al mismo tiempo en clústeres diferentes, por lo que el orden de los eventos no es determinista.

Si está ejecutando las actividades de flujo de datos en paralelo dentro de las canalizaciones, se recomienda no utilizar TTL. Esta acción se debe a que las ejecuciones paralelas del flujo de datos que usan simultáneamente la misma instancia de Azure Integration Runtime da como resultado varias instancias de grupo semiactivas para la factoría de datos.

##### <a name="overload-single-data-flow"></a>Sobrecargar un flujo de datos único

Si coloca toda la lógica dentro de un flujo de datos único, ADF se ejecuta íntegramente en ese mismo contexto de ejecución de trabajo en una única instancia de clúster de Spark.

Es posible que el seguimiento y la solución de problemas de esta opción sean más difíciles, ya que las reglas de negocio y la lógica de negocios se pueden mezclar. Esta opción tampoco proporciona mucha capacidad de reutilización.

##### <a name="execute-data-flows-sequentially"></a>Ejecución secuencial de flujos de datos

Si ejecuta las actividades de flujo de datos en secuencia en la canalización y ha establecido un TTL en la configuración de Azure IR, ADF volverá a usar los recursos de proceso (VM), lo que dará lugar a tiempos de ejecución posteriores más rápidos. Seguirá recibiendo un nuevo contexto de Spark para cada ejecución.

De estas tres opciones, esta acción es la que probablemente tarda más tiempo en ejecutarse de un extremo a otro. Pero proporciona una separación limpia de las operaciones lógicas en cada paso del flujo de datos.

### <a name="configuration-panel"></a>Panel de configuración

En el panel configuración se muestra la configuración específica de la transformación seleccionada actualmente. Si no se ha seleccionado ninguna transformación, se muestra el flujo de datos. En la configuración general del flujo de datos, puede modificar el nombre y la descripción en la pestaña **General** o agregar parámetros en la pestaña **Parámetros**. Para más información, consulte [Parámetros de asignación de Data Flow](parameters-data-flow.md).

Cada transformación contiene al menos cuatro pestañas de configuración.

#### <a name="transformation-settings"></a>Configuración de la transformación

La primera pestaña del panel de configuración de cada transformación contiene los valores específicos de esa transformación. Para más información, consulte la página de documentación de la transformación.

![Pestaña de configuración de origen](media/data-flow/source1.png "Pestaña de configuración de origen")

#### <a name="optimize"></a>Optimización

La pestaña **Optimizar** contiene valores opcionales para configurar los esquemas de partición. Para obtener más información sobre cómo optimizar los flujos de datos, consulte la [guía de rendimiento de flujos de datos de asignación](concepts-data-flow-performance.md).

![Optimize](media/data-flow/optimize.png "Optimización") (Optimizar)

#### <a name="inspect"></a>Inspeccionar

La pestaña **Inspeccionar** proporciona una vista de los metadatos del flujo de datos que se está transformando. Puede ver el número de columnas, las columnas que han cambiado, las columnas que se han agregado, los tipos de datos, el orden de las columnas y las referencias de las columnas. **Inspeccionar** es una vista de solo lectura de los metadatos. Para ver los metadatos en el panel **Inspeccionar**, no es preciso que el modo de depuración esté habilitado.

![Inspeccionar](media/data-flow/inspect1.png "Inspeccionar")

Al cambiar la forma de los datos mediante transformaciones, verá que los cambios de los metadatos fluyen por el panel **Inspeccionar**. Si no hay un esquema definido en la transformación de origen, los metadatos no estarán visibles en el panel **Inspeccionar**. La falta de metadatos es habitual en escenarios de desviación en el esquema.

#### <a name="data-preview"></a>Vista previa de datos

Si el modo de depuración está activado, en la pestaña **Vista previa de los datos**, podrá ver una instantánea interactiva de los datos en cada transformación. Para más información, consulte [Vista previa de los datos en modo de depuración](concepts-data-flow-debug-mode.md#data-preview).

### <a name="top-bar"></a>Barra superior

La barra superior contiene acciones que afectan a todo el flujo de datos, como la operación de guardar y la validación. Puede alternar entre el modo de gráfico y el modo de configuración con los botones **Mostrar gráfico** y **Ocultar gráfico**.

![Ocultar gráfico](media/data-flow/hideg.png "Hide graph (Ocultar gráfico)")

Si oculta el gráfico, puede navegar lateralmente por los nodos de transformación con los botones **Anterior** y **Siguiente**.

![Botones Anterior y Siguiente](media/data-flow/showhide.png "Botones Anterior y Siguiente")

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a crear una [transformación de origen](data-flow-source.md).
* Aprenda a crear flujos de datos en [modo de depuración](concepts-data-flow-debug-mode.md).
