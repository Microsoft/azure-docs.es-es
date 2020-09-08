---
title: Procesamiento de archivos de texto de longitud fija con flujos de datos de asignación en Azure Data Factory
description: Aprenda a procesar archivos de texto de longitud fija en Azure Data Factory mediante flujos de datos de asignación.
services: data-factory
author: balakreshnan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 8/18/2019
ms.author: makromer
ms.openlocfilehash: 23b812da8c84ebf055ac4eabdc4649828c139a7f
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051022"
---
# <a name="process-fixed-length-text-files-by-using-data-factory-mapping-data-flows"></a>Procesamiento de archivos de texto de longitud fija mediante flujos de datos de asignación de Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Mediante el uso de flujos de datos de asignación en Microsoft Azure Data Factory, puede transformar datos de archivos de texto de ancho fijo. En la tarea siguiente, definiremos un conjunto de datos para un archivo de texto sin un delimitador y, a continuación, configuraremos divisiones de subcadenas basadas en la posición ordinal.

## <a name="create-a-pipeline"></a>Crear una canalización

1. Seleccione **+Nueva canalización** para iniciar una nueva canalización.

2. Agregue una actividad de flujo de datos que se usará para el procesamiento de archivos de ancho fijo:

    ![Canalización de ancho fijo](media/data-flow/fwpipe.png)

3. En la actividad de flujo de datos, seleccione **New Mapping Data Flow** (Nuevo flujo de datos de asignación).

4. Agregue una transformación de origen, una columna derivada, una de selección y una de receptor:

    ![Flujo de datos de ancho fijo](media/data-flow/fw2.png)

5. Configure la transformación de origen para que use un nuevo conjunto de datos, que será del tipo texto delimitado.

6. No establezca ningún delimitador o encabezado de columna.

   Ahora vamos a establecer simplemente puntos iniciales y longitudes de campo para el contenido de este archivo:

    ```
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    ```

7. En la pestaña **Proyección** de la transformación de origen, debería ver una columna de cadena denominada *Column_1*.

8. En la columna derivada, cree una nueva columna.

9. Asignaremos a las columnas nombres sencillos como *col1*.

10. A continuación, en el generador de expresiones, escriba lo siguiente:

    ```substring(Column_1,1,4)```

    ![columna derivada](media/data-flow/fwderivedcol1.png)

11. Repita el paso 10 para todas las columnas que necesite analizar.

12. Seleccione la pestaña **Inspeccionar** para ver las nuevas columnas que se van a generar:

    ![inspeccionar](media/data-flow/fwinspect.png)

13. Use la transformación de selección para quitar todas las columnas que no necesite para la transformación:

    ![seleccionar transformación](media/data-flow/fwselect.png)

14. Use la transformación de receptor para generar los datos en una carpeta:

    ![receptor de ancho fijo](media/data-flow/fwsink.png)

    El resultado tendrá una apariencia similar a la siguiente:

    ![salida de ancho fijo](media/data-flow/fxdoutput.png)

  Los datos de ancho fijo se dividen ahora con cuatro caracteres cada uno y se asignan a Col1, Col2, Col3, Col4, etc. Según el ejemplo anterior, los datos se dividen en cuatro columnas.

## <a name="next-steps"></a>Pasos siguientes

* Compile el resto de la lógica del flujo de datos mediante [transformaciones](concepts-data-flow-overview.md) de flujos de datos de asignación.
