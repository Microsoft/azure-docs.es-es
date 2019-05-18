---
title: Configurar una transformación de origen en la característica de asignación de flujo de datos de Azure Data Factory
description: Obtenga información sobre cómo configurar una transformación de origen en la asignación de flujo de datos.
author: kromerm
ms.author: makromer
ms.reviewer: douglasl
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/12/2019
ms.openlocfilehash: dc0a6e008c7a1f4fb414f6d8adad3a94abc7a6b2
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65792335"
---
# <a name="source-transformation-for-mapping-data-flow"></a>Transformación de origen para la asignación de flujo de datos 

[!INCLUDE [notes](../../includes/data-factory-data-flow-preview.md)]

Una transformación de origen configura el origen de datos para el flujo de datos. Un flujo de datos puede incluir más de una transformación de origen. Cuando el diseño de datos fluye, siempre empiezan con una transformación de origen.

Cada flujo de datos requiere la transformación de al menos un origen. Agregar orígenes tantos como sea necesario para completar las transformaciones de datos. Puede combinar esos orígenes junto con una transformación de combinación o una transformación Unión.

> [!NOTE]
> Cuando se depura un flujo de datos, los datos se leen desde el origen mediante el uso de la configuración de muestreo o los límites de código fuente de depuración. Para escribir datos en un receptor, debe ejecutar el flujo de datos desde una canalización de la actividad de flujo de datos. 

![Opciones de transformación en la pestaña Configuración de origen del origen de](media/data-flow/source.png "origen")

Asociar su transformación del origen de flujo de datos con exactamente un conjunto de datos de Data Factory. El conjunto de datos define la forma y la ubicación de los datos que desea escribir o leer. Puede usar caracteres comodín y el archivo de listas en el origen para trabajar con más de un archivo a la vez.

## <a name="data-flow-staging-areas"></a>Áreas de almacenamiento provisional de flujo de datos

Flujo de datos funciona con *ensayo* conjuntos de datos que están en Azure. Utilice estos conjuntos de datos de ensayo si se va a transformar los datos. 

Factoría de datos tiene acceso a casi 80 conectores nativos. Para incluir datos de esos otros orígenes en el flujo de datos, utilice la herramienta de la actividad de copia para almacenar provisionalmente datos en una de las áreas de ensayo del conjunto de datos de flujo de datos.

## <a name="options"></a>Opciones

Elegir las opciones de muestreo y el esquema para los datos.

### <a name="allow-schema-drift"></a>Permitir el desfase de esquema
Seleccione **permitir desviación en el esquema** si va a cambiar con frecuencia las columnas de origen. Esta configuración permite que todos los campos de origen entrante fluya a través de las transformaciones al receptor.

### <a name="validate-schema"></a>Validar esquema

Si la versión del origen de datos entrante no coincide con el esquema definido, no podrá ejecutar el flujo de datos.

![Configuración de origen público, que muestra las opciones de validación de esquema, permitir desviación en el esquema y muestreo](media/data-flow/source1.png "origen público 1")

### <a name="sample-the-data"></a>Los datos de ejemplo
Habilitar **muestreo** para limitar el número de filas de su origen. Use esta opción al probar o datos de ejemplo desde el origen con fines de depuración.

## <a name="define-schema"></a>Definir el esquema

Cuando los archivos de origen no son fuertemente tipados (por ejemplo, archivos sin formato en lugar de los archivos de Parquet), defina los tipos de datos para cada campo aquí en la transformación de origen.  

![Configuración de transformación en la pestaña de esquema de definición de origen](media/data-flow/source2.png "origen 2")

Más adelante puede cambiar los nombres de columna en una transformación que seleccione. Utilice una transformación columna derivada para cambiar los tipos de datos. Para los orígenes fuertemente tipados, puede modificar los tipos de datos en una transformación seleccione más adelante. 

![Tipos de datos en una transformación seleccione](media/data-flow/source003.png "tipos de datos")

### <a name="optimize-the-source-transformation"></a>Optimizar la transformación de origen

En el **optimizar** pestaña para la transformación de origen, es posible que vea un **origen** tipo de partición. Esta opción está disponible solo cuando el origen es Azure SQL Database. Esto es porque la factoría de datos intenta establecer conexiones en paralelo para ejecutar consultas grandes en el origen de la base de datos SQL.

![Configuración de la partición de origen](media/data-flow/sourcepart2.png "creación de particiones")

No tienes que crear particiones de datos en el origen de la base de datos SQL, pero las particiones son útiles para las consultas grandes. Puede basar la partición en una columna o una consulta.

### <a name="use-a-column-to-partition-data"></a>Usar una columna de datos de partición

En la tabla de origen, seleccione una columna de partición en. También establece el número máximo de conexiones.

### <a name="use-a-query-to-partition-data"></a>Utilice una consulta para crear particiones de datos

Puede elegir para la partición de las conexiones según una consulta. Simplemente escriba el contenido de un predicado WHERE. Por ejemplo, escriba año 1980 >.

## <a name="source-file-management"></a>Administración de archivos de origen

Elija la configuración para administrar archivos en el origen. 

![Nueva configuración de origen](media/data-flow/source2.png "nueva configuración")

* **Ruta de acceso de comodín**: En la carpeta de origen, elija una serie de archivos que coinciden con un patrón. Este valor invalida cualquier archivo en la definición del conjunto de datos.
* **Lista de archivos**: Se trata de un conjunto de archivos. Cree un archivo de texto que incluye una lista de archivos de ruta de acceso relativa que se va a procesar. Seleccione este archivo de texto.
* **Columna para almacenar el nombre de archivo**: Store el nombre del archivo de origen en una columna de los datos. Escriba aquí un nombre nuevo para almacenar la cadena de nombre de archivo.
* **Tras la finalización**: Optar por no hacer nada con el archivo de origen después de que los datos de ejecuciones de flujo, eliminación el archivo de origen o mover el archivo de origen. Para mover las rutas de acceso son relativas.

### <a name="sql-datasets"></a>Conjuntos de datos SQL

Si el origen está en la base de datos SQL o SQL Data Warehouse, tendrá opciones adicionales para la administración de archivos de origen.

* **Consultar** escriba una consulta SQL para el origen. Este valor invalida cualquier tabla que ha elegido en el conjunto de datos. Tenga en cuenta que **Order By** cláusulas no se admiten aquí. Pero puede establecer una instrucción SELECT FROM completa aquí.
* **Tamaño del lote**: Escriba un tamaño de lote para fragmentar datos grandes en lecturas.

> [!NOTE]
> Las operaciones de archivo se ejecutan solo cuando se inicia el flujo de datos desde una canalización de ejecución (depuración de la canalización o ejecución) que usa la actividad de ejecución de flujo de datos en una canalización. Las operaciones de archivos *no* ejecutar en modo de depuración de flujo de datos.

### <a name="projection"></a>Proyección

Al igual que los esquemas en los conjuntos de datos, la proyección de un origen define las columnas de datos, tipos y formatos del origen de datos. 

![Configuración de la ficha de proyección](media/data-flow/source3.png "proyección")

Si el archivo de texto no tiene ningún esquema definido, seleccione **detectar tipo de datos** para que Data Factory se muestra y deducir los tipos de datos. Seleccione **formato predeterminado de definir** para detectar automáticamente da formato a los datos de forma predeterminada. 

Puede modificar los tipos de datos de columna en una transformación de columna derivada más adelante. Utilice una transformación que seleccione para modificar los nombres de columna.

![Configuración de los formatos de datos predeterminada](media/data-flow/source2.png "los formatos predeterminados")

## <a name="next-steps"></a>Pasos siguientes

Empezar a compilar un [transformación columna derivada](data-flow-derived-column.md) y un [Seleccionar transformación](data-flow-select.md).
