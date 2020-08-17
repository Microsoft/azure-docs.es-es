---
title: Cuadernos de Synapse Studio
description: En este artículo, aprenderá a crear y desarrollar cuadernos de Azure Synapse Studio (versión preliminar) para la preparación y visualización de datos.
services: synapse analytics
author: ruixinxu
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 05/01/2020
ms.author: ruxu
ms.reviewer: ''
ms.custom: devx-track-python
ms.openlocfilehash: 6e0062450889a2bbbdfcd47137ffbe36b83cae57
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87849105"
---
# <a name="create-develop-and-maintain-synapse-studio-preview-notebooks-in-azure-synapse-analytics"></a>Creación, desarrollo y mantenimiento de cuadernos de Synapse Studio (versión preliminar) en Azure Synapse Analytics

Un cuaderno de Synapse Studio (versión preliminar) es una interfaz web para crear archivos que contengan código activo, visualizaciones y texto narrativo. Los cuadernos son un buen lugar para validar ideas y aplicar experimentos rápidos para sacar conclusiones a partir de los datos. Los cuadernos también se usan ampliamente en la preparación de datos, la visualización de datos, el aprendizaje automático y otros escenarios de macrodatos.

Con un cuaderno de Azure Synapse Studio, puede hacer lo siguiente:

* Empezar a trabajar sin esfuerzo alguno de configuración.
* Mantener los datos protegidos con las características de seguridad empresarial integradas.
* Analizar datos en formatos sin procesar (CSV, TXT, JSON, etc.), formatos de archivos procesados (parquet, Delta Lake, ORC, etc.) y archivos de datos tabulares de SQL en Spark y SQL.
* Ser productivo con funcionalidades de creación mejoradas y visualización de datos integrada.

En este artículo se describe cómo usar los cuadernos en Azure Synapse Studio.

## <a name="create-a-notebook"></a>Creación de un cuaderno

Hay dos formas de crear un cuaderno. Puede crear un cuaderno o importar uno existente en un área de trabajo de Azure Synapse desde el **Explorador de objetos**. Los cuadernos de Azure Synapse Studio pueden reconocer archivos IPYNB estándar de Jupyter Notebook.

![synapse-create-import-notebook](./media/apache-spark-development-using-notebooks/synapse-create-import-notebook.png)

## <a name="develop-notebooks"></a>Desarrollo de cuadernos

Los cuadernos se componen de celdas, que son bloques de código individuales o texto que se pueden ejecutar de manera independiente o como grupo.

### <a name="add-a-cell"></a>Adición de una celda

Hay varias maneras de agregar una nueva celda a un cuaderno.

1. Expanda el botón superior izquierdo **+ Celda** y seleccione **Agregar celda de código** o **Agregar celda de texto**.

    ![add-cell-with-cell-button](./media/apache-spark-development-using-notebooks/synapse-add-cell-1.png)

2. Mantenga el puntero sobre el espacio entre dos celdas y seleccione **Agregar código** o **Agregar texto**.

    ![add-cell-between-space](./media/apache-spark-development-using-notebooks/synapse-add-cell-2.png)

3. Utilice las [teclas de método abreviado en el modo de comando](#shortcut-keys-under-command-mode). Presione **A** para insertar una celda sobre la celda actual. Presione **B** para insertar una celda debajo de la celda actual.

### <a name="set-a-primary-language"></a>Definición del lenguaje principal

Los cuadernos de Azure Synapse Studio admiten cuatro lenguajes de Apache Spark:

* pySpark (Python)
* Spark (Scala)
* SparkSQL
* .NET para Apache Spark (C#)

Desde la lista desplegable de la barra de comandos superior, puede establecer el lenguaje principal para las nuevas celdas que se agreguen.

   ![default-synapse-language](./media/apache-spark-development-using-notebooks/synapse-default-language.png)

### <a name="use-multiple-languages"></a>Uso de varios lenguajes

Para usar varios lenguajes en un cuaderno, puede especificar el comando magic de lenguaje correcto al principio de una celda. En la tabla siguiente se enumeran los comandos magic para cambiar el lenguaje de las celdas.

|Comando magic |Idioma | Descripción |  
|---|------|-----|
|%%pyspark| Python | Ejecute una consulta de **Python** en SparkContext.  |
|%%spark| Scala | Ejecute una consulta de **Scala** en SparkContext.  |  
|%%sql| SparkSQL | Ejecute una consulta de **SparkSQL** en SparkContext.  |
|%%csharp | .NET para Spark C# | Ejecute una consulta de **.NET para Spark C#** en Spark Context. |

La imagen siguiente es un ejemplo de cómo se puede escribir una consulta de PySpark con el comando magic **%%pyspark** o una consulta de SparkSQL con el comando magic **%%sql** en un cuaderno de **Spark (Scala)** . Tenga en cuenta que el lenguaje principal del cuaderno está establecido en pySpark.

   ![synapse-spark-magics](./media/apache-spark-development-using-notebooks/synapse-spark-magics.png)

### <a name="use-temp-tables-to-reference-data-across-languages"></a>Uso de tablas temporales para hacer referencia a datos entre lenguajes

No puede hacer referencia a datos o variables directamente en distintos lenguajes en un cuaderno de Synapse Studio. En Spark, se puede hacer referencia a una tabla temporal entre lenguajes. Este es un ejemplo de cómo leer un DataFrame de `Scala` en `PySpark` y `SparkSQL` mediante una tabla temporal de Spark como solución alternativa.

1. En la celda 1, lea un DataFrame de un conector de grupo de SQL mediante Scala y cree una tabla temporal.

   ```scala
   %%scala
   val scalaDataFrame = spark.read.option("format", "DW connector predefined type")
   scalaDataFrame.registerTempTable( "mydataframetable" )
   ```

2. En la celda 2, consulte los datos mediante Spark SQL.
   
   ```sql
   %%sql
   SELECT * FROM mydataframetable
   ```

3. En la celda 3, use los datos de PySpark.

   ```python
   %%pyspark
   myNewPythonDataFrame = spark.sql("SELECT * FROM mydataframetable")
   ```

### <a name="ide-style-intellisense"></a>IntelliSense de estilo IDE

Los cuadernos de Azure Synapse Studio se integran en el editor Monaco para incluir IntelliSense de estilo IDE en el editor de celdas. El resaltado de la sintaxis, el marcador de errores y la finalización automática de código le ayudan a escribir código y a identificar problemas más rápido.

Las características de IntelliSense tienen distintos niveles de madurez para distintos lenguajes. Use la tabla a continuación para ver lo que se admite.

|Lenguajes| Resaltado de sintaxis | Creador de errores de sintaxis  | Finalización de código de sintaxis | Finalización de código de sintaxis| Finalización de código de funciones del sistema| Finalización de código de funciones del usuario| Sangría inteligente | Plegado de código|
|--|--|--|--|--|--|--|--|--|
|PySpark (Python)|Sí|Sí|Sí|Sí|Sí|Sí|Sí|Sí|
|Spark (Scala)|Sí|Sí|Sí|Sí|-|-|-|Sí|
|SparkSQL|Sí|Sí|-|-|-|-|-|-|
|.NET para Spark ( C# )|Sí|-|-|-|-|-|-|-|

### <a name="format-text-cell-with-toolbar-buttons"></a>Formato de celdas de texto con botones de la barra de herramientas

Puede usar los botones de formato en la barra de herramientas de celdas de texto para completar acciones comunes de marcado. Incluye texto en negrita, cursiva, inserción de fragmentos de código, inserción de listas sin ordenar, inserción de listas ordenadas e inserción de imágenes desde direcciones URL.

  ![synapse-text-cell-toolbar](./media/apache-spark-development-using-notebooks/synapse-text-cell-toolbar.png)

### <a name="undo-cell-operations"></a>Deshacer operaciones en celdas
Haga clic en el botón **Deshacer** o presione **CTRL+Z** para revocar la operación más reciente en la celda. Ahora puede deshacer hasta las 20 acciones de celda históricas más recientes. 

   ![synapse-undo-cells](./media/apache-spark-development-using-notebooks/synapse-undo-cells.png)

### <a name="move-a-cell"></a>Movimiento de una celda

Seleccione los puntos suspensivos (…) para tener acceso al menú de acciones de celda adicionales en el extremo derecho. A continuación, seleccione **Move cell up** (Subir celda) o **Move cell down** (Bajar celda) para desplazar la celda actual. 

También puede utilizar las [teclas de método abreviado en el modo de comando](#shortcut-keys-under-command-mode). Presione **CTRL+Alt+↑** para subir la celda actual. Presione **CTRL+Alt+↓** para bajar la celda actual.

   ![move-a-cell](./media/apache-spark-development-using-notebooks/synapse-move-cells.png)

### <a name="delete-a-cell"></a>Eliminación de una celda

Para eliminar una celda, seleccione los puntos suspensivos (…) para tener acceso al menú de acciones de celda adicionales en el extremo derecho y, a continuación, seleccione **Eliminar celda**. 

También puede utilizar las [teclas de método abreviado en el modo de comando](#shortcut-keys-under-command-mode). Presione **D,D** para eliminar la celda actual.
  
   ![delete-a-cell](./media/apache-spark-development-using-notebooks/synapse-delete-cell.png)

### <a name="collapse-a-cell-input"></a>Contracción de una entrada de celda
Haga clic en el botón de flecha situado en la parte inferior de la celda actual para contraerla. Para expandirla, haga clic en el botón de flecha mientras la celda está contraída.

   ![collapse-cell-input](./media/apache-spark-development-using-notebooks/synapse-collapse-cell-input.gif)

### <a name="collapse-a-cell-output"></a>Contracción de una salida de celda

Haga clic en el botón **Collapse output** (Contraer salida) situado en la parte superior izquierda de la salida de la celda actual para contraerla. Para expandirla, haga clic en **Show cell output** (Mostrar salida de la celda) mientras la salida de la celda está contraída.

   ![collapse-cell-output](./media/apache-spark-development-using-notebooks/synapse-collapse-cell-output.gif)

## <a name="run-notebooks"></a>Ejecución de cuadernos

Puede ejecutar las celdas de código en el cuaderno individualmente o todas a la vez. El estado y el progreso de cada celda se representan en el cuaderno.

### <a name="run-a-cell"></a>Ejecución de una celda

Hay varias maneras de ejecutar el código de una celda.

1. Mantenga el puntero sobre la celda que desea ejecutar y seleccione el botón **Ejecutar celda** o presione **CTRL+Entrar**.

   ![run-cell-1](./media/apache-spark-development-using-notebooks/synapse-run-cell.png)


2. Para tener acceso al menú de acciones de celda adicionales en el extremo derecho, seleccione los puntos suspensivos ( **…** ). Luego, seleccione **Ejecutar celda**.

   ![run-cell-2](./media/apache-spark-development-using-notebooks/synapse-run-cell-2.png)
   
3. Utilice las [teclas de método abreviado en el modo de comando](#shortcut-keys-under-command-mode). Presione **Mayús+Entrar** para ejecutar la celda actual y seleccionar la celda a continuación. Presione **Mayús+Entrar** para ejecutar la celda actual e insertar una nueva celda a continuación.


### <a name="run-all-cells"></a>Ejecución de todas las celdas
Haga clic en el botón **Ejecutar todo** para ejecutar todas las celdas del cuaderno actual en secuencia.

   ![run-all-cells](./media/apache-spark-development-using-notebooks/synapse-run-all.png)

### <a name="run-all-cells-above-or-below"></a>Ejecución de todas las celdas encima o debajo

Para tener acceso al menú de acciones de celda adicionales en el extremo derecho, seleccione los puntos suspensivos ( **…** ). A continuación, seleccione **Run cells above** (Ejecutar las celdas encima) para ejecutar todas las celdas situadas encima de la actual en secuencia. Seleccione **Run cells below** (Ejecutar las celdas debajo) para ejecutar todas las celdas situadas debajo de la actual en secuencia.

   ![run-cells-above-or-below](./media/apache-spark-development-using-notebooks/synapse-run-cells-above-or-below.png)


### <a name="cancel-all-running-cells"></a>Cancelación de todas las celdas en ejecución
Haga clic en el botón **Cancelar todo** para cancelar las celdas en ejecución o las celdas que esperan en la cola. 
   ![cancel-all-cells](./media/apache-spark-development-using-notebooks/synapse-cancel-all.png) 

### <a name="cell-status-indicator"></a>Indicador de estado de la celda

Debajo de la celda se muestra un estado de ejecución detallado de la celda para ayudarle a ver su progreso actual. Una vez que se complete la ejecución de la celda, aparecerá un resumen de la ejecución con la duración total y la hora de finalización; este resumen se mantendrá allí para futuras referencias.

![cell-status](./media/apache-spark-development-using-notebooks/synapse-cell-status.png)

### <a name="spark-progress-indicator"></a>Indicador de progreso de Spark

Un cuaderno de Azure Synapse Studio se basa únicamente en Spark. Las celdas de código se ejecutan en el grupo de Spark de manera remota. Aparece un indicador de progreso del trabajo de Spark con una barra de progreso en tiempo real que le ayudará a entender el estado de ejecución del trabajo.
El número de tareas por cada trabajo o etapa ayuda a identificar el nivel paralelo del trabajo de Spark. También puede profundizar en la interfaz de usuario de Spark de un trabajo o fase específicos a través de un clic en el vínculo del nombre del trabajo o de la fase.


![spark-progress-indicator](./media/apache-spark-development-using-notebooks/synapse-spark-progress-indicator.png)

### <a name="spark-session-config"></a>Configuración de la sesión de Spark

Puede especificar la duración del tiempo de espera, el número y el tamaño de los ejecutores que se van a dar a la sesión actual de Spark en **Configure session** (Configurar sesión). Reinicie la sesión de Spark para que surtan efecto los cambios de configuración. Se borran todas las variables del cuaderno almacenadas en la memoria caché.

[![session-management](./media/apache-spark-development-using-notebooks/synapse-spark-session-management.png)](./media/apache-spark-development-using-notebooks/synapse-spark-session-management.png#lightbox)

Un recomendador de sesión de Spark ahora está disponible en el panel de configuración de la sesión de Spark. Puede seleccionar un grupo de Spark directamente desde el panel de configuración de la sesión y ver cuántos nodos están en uso y cuántos ejecutores restantes están disponibles. Esta información puede ayudar a establecer el tamaño de la sesión correctamente en lugar de tener que modificarlo una y otra vez.

![session-recommend](./media/apache-spark-development-using-notebooks/synapse-spark-session-recommender.png)


## <a name="bring-data-to-a-notebook"></a>Traslado de los datos a un cuaderno

Puede cargar datos desde Azure Blob Storage, Azure Data Lake Store Gen2 y el grupo de SQL, tal como se muestra en los ejemplos de código siguientes.

### <a name="read-a-csv-from-azure-data-lake-store-gen2-as-a-spark-dataframe"></a>Lectura de un archivo CSV desde Azure Data Lake Store Gen2 como DataFrame de Spark

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
account_name = "Your account name"
container_name = "Your container name"
relative_path = "Your path"
adls_path = 'abfss://%s@%s.dfs.core.windows.net/%s' % (blob_container_name, blob_account_name,  blob_relative_path)

spark.conf.set("fs.azure.account.auth.type.%s.dfs.core.windows.net" %account_name, "SharedKey")
spark.conf.set("fs.azure.account.key.%s.dfs.core.windows.net" %account_name ,"Your ADLSg2 Primary Key")

df1 = spark.read.option('header', 'true') \
                .option('delimiter', ',') \
                .csv(adls_path + '/Testfile.csv')

```

#### <a name="read-a-csv-from-azure-blob-storage-as-a-spark-dataframe"></a>Lectura de un archivo CSV desde Azure Blob Storage como DataFrame de Spark

```python

from pyspark.sql import SparkSession
from pyspark.sql.types import *

blob_account_name = "Your blob account name"
blob_container_name = "Your blob container name"
blob_relative_path = "Your blob relative path"
blob_sas_token = "Your blob sas token"

wasbs_path = 'wasbs://%s@%s.blob.core.windows.net/%s' % (blob_container_name, blob_account_name, blob_relative_path)
spark.conf.set('fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name), blob_sas_token)

df = spark.read.option("header", "true") \
            .option("delimiter","|") \
            .schema(schema) \
            .csv(wasbs_path)

```

### <a name="read-data-from-the-primary-storage-account"></a>Lectura de datos desde la cuenta de almacenamiento principal

Puede tener acceso directamente a los datos de la cuenta de almacenamiento principal. No hay necesidad de proporcionar las claves secretas. En Explorador de datos, haga clic con el botón derecho en un archivo y seleccione **Nuevo cuaderno** para ver un nuevo cuaderno con el extractor de datos generado automáticamente.

![data-to-cell](./media/apache-spark-development-using-notebooks/synapse-data-to-cell.png)

## <a name="visualize-data-in-a-notebook"></a>Visualización de datos en un cuaderno

### <a name="produce-rendered-table-view"></a>Generación de una vista tabular representada

Se proporciona una vista tabular de resultados con la opción de crear un gráfico de barras, un gráfico de líneas, un gráfico circular, un gráfico de dispersión y un gráfico de áreas. Puede visualizar los datos sin tener que escribir código. Los gráficos pueden personalizarse en **Opciones de gráfico**. 

De forma predeterminada, la salida de los comandos magic **%%sql** aparece en la vista de tabla representada. Puede llamar a <code>display(df)</code> en DataFrames de Spark o a una función de Resilient Distributed Datasets (RDD) para generar la vista tabular representada.

   [![builtin-charts](./media/apache-spark-development-using-notebooks/synapse-builtin-charts.png)](./media/apache-spark-development-using-notebooks/synapse-builtin-charts.png#lightbox)

### <a name="visualize-built-in-charts-from-large-scale-dataset"></a>Visualización de gráficos integrados a partir de un conjunto de datos a gran escala 

De forma predeterminada, la función <code>display(df)</code> solo tomará las primeras 1000 filas de los datos para representar los gráficos. Marque **Aggregation over all results** (Agregación de todos los resultados) y haga clic en el botón **Aplicar**; el gráfico se creará a partir de todo el conjunto de datos. Un trabajo de Spark se desencadenará cuando se cambie la configuración del gráfico; tardará un tiempo en completar el cálculo y representarlo. 
    [![builtin-charts-aggregation-all](./media/apache-spark-development-using-notebooks/synapse-builtin-charts-aggregation-all.png)](./media/apache-spark-development-using-notebooks/synapse-builtin-charts-aggregation-all.png#lightbox)


### <a name="visualize-data-statistic-information"></a>Visualización de la información estadística de los datos
Puede usar <code>display(df, summary = true)</code> para comprobar el resumen de las estadísticas de una instancia de DataFrame de Spark determinada que incluya el nombre de columna, el tipo de columna, los valores únicos y los valores que faltan para cada columna. También puede seleccionar una columna específica para ver el valor mínimo, el valor máximo, el valor medio y la desviación estándar.
    [ ![builtin-charts-summary](./media/apache-spark-development-using-notebooks/synapse-builtin-charts-summary.png) ](./media/apache-spark-development-using-notebooks/synapse-builtin-charts-summary.png#lightbox)

### <a name="render-html-or-interactive-libraries"></a>Representación de bibliotecas HTML o interactivas

Puede representar bibliotecas HTML o interactivas, como **bokeh**, mediante **displayHTML()** .

La imagen siguiente es un ejemplo del trazado de glifos en un mapa mediante **bokeh**.

   ![bokeh-example](./media/apache-spark-development-using-notebooks/synapse-bokeh-image.png)
   

Ejecute el siguiente código de ejemplo para dibujar la imagen anterior.

```python
from bokeh.plotting import figure, output_file
from bokeh.tile_providers import get_provider, Vendors
from bokeh.embed import file_html
from bokeh.resources import CDN
from bokeh.models import ColumnDataSource

tile_provider = get_provider(Vendors.CARTODBPOSITRON)

# range bounds supplied in web mercator coordinates
p = figure(x_range=(-9000000,-8000000), y_range=(4000000,5000000),
           x_axis_type="mercator", y_axis_type="mercator")
p.add_tile(tile_provider)

# plot datapoints on the map
source = ColumnDataSource(
    data=dict(x=[ -8800000, -8500000 , -8800000],
              y=[4200000, 4500000, 4900000])
)

p.circle(x="x", y="y", size=15, fill_color="blue", fill_alpha=0.8, source=source)

# create an html document that embeds the Bokeh plot
html = file_html(p, CDN, "my plot1")

# display this html
displayHTML(html)

```

## <a name="save-notebooks"></a>Guardado de cuadernos

Puede guardar un único cuaderno o todos los cuadernos en el área de trabajo.

1. Para guardar los cambios realizados en un solo cuaderno, seleccione el botón **Publicar** en la barra de comandos del cuaderno.

   ![publish-notebook](./media/apache-spark-development-using-notebooks/synapse-publish-notebook.png)

2. Para guardar todos los cuadernos en el área de trabajo, seleccione el botón **Publicar todo** en la barra de comandos del área de trabajo. 

   ![publish-all](./media/apache-spark-development-using-notebooks/synapse-publish-all.png)

En las propiedades del cuaderno, puede configurar si incluir la salida de la celda al guardar.

   ![notebook-properties](./media/apache-spark-development-using-notebooks/synapse-notebook-properties.png)

## <a name="magic-commands"></a>Comandos magic
Puede usar los comandos magic de Jupyter que ya conoce en los cuadernos de Azure Synapse Studio. Consulte en la lista a continuación los comandos magic disponibles actualmente. Indíquenos sus casos de uso en GitHub para que podamos seguir generando más comandos magic para satisfacer sus necesidades.

Comandos magic de línea disponibles: [%lsmagic](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-lsmagic), [%time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time) y [%time it](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit)

Comandos magic de celda disponibles: [%%time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [%%timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit), [%%capture](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-capture), [%%writefile](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-writefile), [%%sql](#use-multiple-languages), [%%pyspark](#use-multiple-languages), [%%spark](#use-multiple-languages), [%%csharp](#use-multiple-languages)


## <a name="orchestrate-notebook"></a>Organización de cuadernos

### <a name="add-a-notebook-to-a-pipeline"></a>Adición de un cuaderno a una canalización

Haga clic en el botón **Agregar a la canalización** en la esquina superior derecha para agregar un cuaderno a una canalización existente o crear una canalización.

![add-to-pipeline](./media/apache-spark-development-using-notebooks/add-to-pipeline.png)

### <a name="designate-a-parameters-cell"></a>Designación de una celda de parámetros

Para parametrizar el cuaderno, seleccione los puntos suspensivos (…) para acceder al menú de acciones de celda adicionales en el extremo derecho. A continuación, seleccione **Toggle parameter cell** (Alternar celda de parámetros) para designar la celda como la celda de parámetros.

![toggle-parameter](./media/apache-spark-development-using-notebooks/toggle-parameter-cell.png)

Azure Data Factory busca la celda de parámetros y la trata como valores predeterminados para los parámetros que se pasan en tiempo de ejecución. El motor de ejecución agregará una nueva celda debajo de la celda de parámetros con parámetros de entrada para sobrescribir los valores predeterminados. Cuando no se designa ninguna celda de parámetros, la celda insertada se insertará en la parte superior del cuaderno.

### <a name="assign-parameters-values-from-a-pipeline"></a>Asignación de valores de parámetros de una canalización

Una vez que haya creado un cuaderno con parámetros, podrá ejecutarlo desde una canalización con la actividad de cuadernos de Azure Synapse. Después de agregar la actividad al lienzo de la canalización, podrá establecer los valores de los parámetros en la sección **Parámetros base** de la pestaña **Configuración**. 

![assign-parameter](./media/apache-spark-development-using-notebooks/assign-parameter.png)

Al asignar valores de parámetros, puede usar el [lenguaje de expresiones de canalización](../../data-factory/control-flow-expression-language-functions.md) o las [variables del sistema](../../data-factory/control-flow-system-variables.md).



## <a name="shortcut-keys"></a>Teclas de método abreviado

De forma similar a los cuadernos de Jupyter Notebook, los cuadernos de Azure Synapse Studio tienen una interfaz de usuario modal. El teclado realiza diferentes acciones en función del modo en que se encuentre la celda del cuaderno. Los cuadernos de Synapse Studio admiten los siguientes dos modos para una celda de código determinada: modo de comando y modo de edición.

1. Una celda se encuentra en modo de comando cuando no hay ningún cursor de texto que le pida que escriba. Cuando una celda está en modo de comando, puede editar el cuaderno en su conjunto, pero no escribir en celdas individuales. Para ingresar al modo de comando, presione `ESC` o use el mouse para hacer clic fuera del área del editor de una celda.

   ![command-mode](./media/apache-spark-development-using-notebooks/synapse-command-mode2.png)

2. El modo de edición se indica mediante un cursor de texto que le pide que escriba en el área del editor. Cuando una celda se encuentra en modo de edición, puede escribir en la celda. Para ingresar al modo de edición, presione `Enter` o use el mouse para hacer clic en el área del editor de una celda.
   
   ![edit-mode](./media/apache-spark-development-using-notebooks/synapse-edit-mode2.png)

### <a name="shortcut-keys-under-command-mode"></a>Teclas de método abreviado en el modo de comando

Con los siguientes métodos abreviados de teclado, puede navegar y ejecutar código más fácilmente en cuadernos de Azure Synapse.

| Acción |Accesos directos de cuadernos de Synapse Studio  |
|--|--|
|Ejecutar la celda actual y seleccionar la que está a continuación | Mayús+Entrar |
|Ejecutar la celda actual e insertar una a continuación | Alt+Entrar |
|Seleccionar la celda anterior| Arriba |
|Seleccionar la celda siguiente| Bajar |
|Insertar una celda encima| A |
|Insertar una celda debajo| B |
|Extender las celdas seleccionadas encima| Mayús+Arriba |
|Extender las celdas seleccionadas debajo| Mayús+Abajo|
|Subir celda| CTRL+Alt+↑ |
|Bajar celda| CTRL+Alt+↓ |
|Eliminar celdas seleccionadas| D, D |
|Cambiar al modo de edición| Escriba |

### <a name="shortcut-keys-under-edit-mode"></a>Teclas de método abreviado en el modo de edición

Con los siguientes métodos abreviados de teclado, puede navegar y ejecutar código más fácilmente en cuadernos de Azure Synapse en el modo de edición.

| Acción |Accesos directos de cuadernos de Synapse Studio  |
|--|--|
|Subir el cursor | Arriba |
|Bajar el cursor|Bajar|
|Deshacer|CTRL+Z|
|Rehacer|CTRL+Y|
|Comentar y quitar comentario|CTRL+/|
|Eliminar palabra anterior|CTRL+Retroceso|
|Eliminar palabra posterior|CTRL+Supr|
|Ir al inicio de la celda|CTRL+Inicio|
|Ir al final de la celda |CTRL+Fin|
|Ir una palabra a la izquierda|CTRL+Izquierda|
|Ir una palabra a la derecha|CTRL+Derecha|
|Seleccionar todo|CTRL+A|
|Aplicar sangría| CTRL+]|
|Desaplicar sangría|CTRL+[|
|Cambiar al modo de comando| Esc |

## <a name="next-steps"></a>Pasos siguientes
- [Consulte los cuadernos de ejemplo de Synapse](https://github.com/Azure-Samples/Synapse/tree/master/Notebooks)
- [Inicio rápido: Creación de un grupo de Apache Spark (versión preliminar) en Azure Synapse Analytics mediante herramientas web](../quickstart-apache-spark-notebook.md)
- [Qué es Apache Spark en Azure Synapse Analytics](apache-spark-overview.md)
- [Uso de .NET para Apache Spark con Azure Synapse Analytics](spark-dotnet.md)
- [Documentación de .NET para Apache Spark](/dotnet/spark?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics)
