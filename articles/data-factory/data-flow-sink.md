---
title: Configurar una transformación de receptor en la característica de asignación de flujo de datos de Azure Data Factory
description: Obtenga información sobre cómo configurar una transformación de receptor en el flujo de datos de asignación.
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/03/2019
ms.openlocfilehash: 4341cbb0e24330d535f5211c088f0068eab33af7
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65596267"
---
# <a name="sink-transformation-for-a-data-flow"></a>Transformación de un flujo de datos de receptor

[!INCLUDE [notes](../../includes/data-factory-data-flow-preview.md)]

Una vez que transforma un flujo de datos, puede recibir los datos en un conjunto de datos de destino. En la transformación de receptor, elija una definición de conjunto de datos para los datos de salida de destino. Puede tener muchas transformaciones de receptor tal y como requiere el flujo de datos.

Para la cuenta para la desviación en el esquema y los cambios en los datos entrantes, los datos de salida de receptor en una carpeta sin un esquema definido en el conjunto de datos de salida. Puede también tener en cuenta los cambios de columna en los orígenes seleccionando **permitir desviación en el esquema** en el origen. A continuación, asignar todos los campos en el receptor.

![Opciones de la pestaña receptor, incluida la opción de asignación automática](media/data-flow/sink1.png "receptor 1")

Para recibir todos los campos de entrada, activar **Autoasignar**. Para elegir los campos en el receptor al destino, o para cambiar los nombres de los campos en el destino, desactive la opción **Autoasignar**. A continuación, abra el **asignación** pestaña asignación de campos de salida.

![Opciones de la pestaña asignación](media/data-flow/sink2.png "receptor 2")

## <a name="output"></a>Salida 
Para Azure Blob storage o tipos de receptores de Data Lake Storage, salida de los datos transformados en una carpeta. Spark genera archivos de datos de salida con particiones según el esquema de partición que utiliza la transformación del receptor. 

Puede establecer el esquema de partición de la **optimizar** ficha. Si desea que la factoría de datos para combinar el resultado en un único archivo, seleccione **única partición**.

![Las opciones en la ficha Optimizar](media/data-flow/opt001.png "opciones de receptor")

## <a name="field-mapping"></a>Asignación de campos

En el **asignación** pestaña de la transformación de receptor, puede asignar las columnas de entrada a la izquierda en los destinos de la derecha. Al receptor de flujos de datos a archivos, Data Factory siempre se escribe nuevos archivos en una carpeta. Cuando se asigna a un conjunto de datos de la base de datos, puede generar una nueva tabla que usa este esquema estableciendo **Guardar directiva** a **sobrescritura**. O insertar filas nuevas en una tabla existente y, a continuación, asignar los campos al esquema existente. 

![La pestaña asignación](media/data-flow/sink2.png "receptores")

En la tabla de asignación puede multiselect para vincular varias columnas, desvincular varias columnas o asignar varias filas en el mismo nombre de columna.

Para asignar siempre el conjunto de campos entrantes a un destino como están y Aceptar completamente las definiciones de esquema flexible, seleccione **permitir desviación en el esquema**.

![La pestaña asignación, que muestra los campos se asignan a columnas del conjunto de datos](media/data-flow/multi1.png "varias opciones")

Para restablecer las asignaciones de columnas, seleccione **volver a asignar**.

![La pestaña receptor](media/data-flow/sink1.png "un receptor")

Seleccione **Validar esquema** un error en el receptor si cambia el esquema.

Seleccione **borrar la carpeta** para truncar el contenido de la carpeta del receptor antes de escribir los archivos de destino en esa carpeta de destino.

## <a name="file-name-options"></a>Opciones de nombre de archivo

Configurar la nomenclatura de archivos: 

   * **Valor predeterminado**: Permitir Spark a los archivos de nombre basados en valores predeterminados de la parte.
   * **Patrón**: Especifique un patrón para los archivos de salida. Por ejemplo, **préstamos [n]** creará loans1.csv loans2.csv y así sucesivamente.
   * **Cada partición**: Escriba un nombre de archivo por partición.
   * **Como los datos de columna**: Establece el archivo de salida en el valor de una columna.
   * **En un único archivo de salida**: Con esta opción, ADF combinará los archivos de salida con particiones en un solo archivo con nombre. Para usar esta opción, el conjunto de datos debe proporcionar un nombre de carpeta. Además, ten en cuenta que esta operación de combinación posiblemente den error en función del tamaño del nodo.

> [!NOTE]
> Inicio de las operaciones de archivo solo cuando se ejecuta la actividad de ejecución de flujo de datos. No se inician en modo de datos de flujo de depuración.

## <a name="database-options"></a>Opciones de base de datos

Elija la configuración de la base de datos:

* **El método Update**: El valor predeterminado es permitir que las inserciones. Borrar **permitir inserción** si desea detener la inserción de nuevas filas del origen de. Para actualizar, upsert, eliminar filas, primero agregue una transformación de alter fila a las filas de la etiqueta para esas acciones. 
* **Vuelva a crear tabla**: Quite o cree la tabla de destino antes de que el flujo de datos finaliza.
* **Truncar una tabla**: Quite todas las filas de la tabla de destino antes de que el flujo de datos finaliza.
* **Tamaño del lote**: Escriba un número para depositar las escrituras en fragmentos. Use esta opción para grandes cargas de datos. 
* **Habilitar el almacenamiento provisional**: Uso de PolyBase al cargar el almacén de datos de Azure como el conjunto de datos receptor.

![La pestaña Configuración, que muestra las opciones de receptor SQL](media/data-flow/alter-row2.png "opciones de SQL")

> [!NOTE]
> En el flujo de datos, puede dirigir la factoría de datos para crear una nueva definición de tabla en la base de datos de destino. Para crear la definición de tabla, establezca un conjunto de datos en la transformación de receptor que tiene un nuevo nombre de tabla. En el conjunto de datos SQL, a continuación el nombre de tabla, seleccione **editar** y escriba un nuevo nombre de tabla. A continuación, en la transformación de receptor, activar **permitir desviación en el esquema**. Establecer **importar esquema** a **ninguno**.

![Configuración del conjunto de datos SQL, que muestra dónde debe editar el nombre de tabla](media/data-flow/dataset2.png "esquema SQL")

> [!NOTE]
> Al actualizar o eliminar filas en el receptor de la base de datos, debe establecer la columna de clave. Esta configuración permite la transformación alter consecutiva determinar la fila única en la biblioteca de movimiento de datos (DML).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado el flujo de datos, agregue un [actividad de flujo de datos a la canalización](concepts-data-flow-overview.md).
