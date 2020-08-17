---
title: Extensión de los scripts de U-SQL con Python en Azure Data Lake Analytics
description: Obtenga información sobre cómo ejecutar código Python en scripts de U-SQL mediante Azure Data Lake Analytics
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/20/2017
ms.custom: devx-track-python
ms.openlocfilehash: b15ab268433e4220d499f3e1fe7cb90ffac2a1be
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876024"
---
# <a name="extend-u-sql-scripts-with-python-code-in-azure-data-lake-analytics"></a>Extensión de los scripts de U-SQL con código Python en Azure Data Lake Analytics

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que están instaladas las extensiones de Python en su cuenta de Azure Data Lake Analytics.

* En Azure Portal, vaya a la cuenta de Data Lake Analytics.
* En el menú izquierdo, en **INTRODUCCIÓN**, haga clic en **Scripts de ejemplo**.
* Seleccione **Instalar las extensiones de U-SQL** y luego **Aceptar**.

## <a name="overview"></a>Información general

Las extensiones de Python para U-SQL permiten a los desarrolladores realizar la ejecución en paralelo de forma masiva de código Python. En el ejemplo siguiente se muestran los pasos básicos:

* Uso de la instrucción `REFERENCE ASSEMBLY` para habilitar las extensiones de Python para el script de U-SQL
* Uso de la operación `REDUCE` para la partición de datos de entrada de una clave
* Las extensiones de Python para U-SQL incluyen un reductor integrado (`Extension.Python.Reducer`) que ejecuta código de Python en cada vértice asignado al reductor
* El script de U-SQL contiene el código de Python insertado que tiene una función denominada `usqlml_main` que acepta un DataFrame de Pandas como entrada y devuelve un DataFrame de Pandas como salida.

```usql
REFERENCE ASSEMBLY [ExtPython];
DECLARE @myScript = @"
def get_mentions(tweet):
    return ';'.join( ( w[1:] for w in tweet.split() if w[0]=='@' ) )
def usqlml_main(df):
    del df['time']
    del df['author']
    df['mentions'] = df.tweet.apply(get_mentions)
    del df['tweet']
    return df
";
@t  =
    SELECT * FROM
       (VALUES
           ("D1","T1","A1","@foo Hello World @bar"),
           ("D2","T2","A2","@baz Hello World @beer")
       ) AS date, time, author, tweet );
@m  =
    REDUCE @t ON date
    PRODUCE date string, mentions string
    USING new Extension.Python.Reducer(pyScript:@myScript);
OUTPUT @m
    TO "/tweetmentions.csv"
    USING Outputters.Csv();
```

## <a name="how-python-integrates-with-u-sql"></a>¿Cómo se integra Python con U-SQL?

### <a name="datatypes"></a>Tipos de datos

* Columnas numéricas y de cadena de U-SQL convertidas tal cual entre pandas y U-SQL
* Los valores Null de U-SQL se conviernten a valores `NA` de Pandas y viceversa

### <a name="schemas"></a>Esquemas

* Los vectores de índice en Pandas no son compatibles en U-SQL. Todas las tramas de datos de entrada en la función Python siempre tienen un índice numérico de 64 bits comprendido entre 0 y el número de filas menos 1.
* Los conjuntos de datos de U-SQL no pueden tener nombres de columna duplicados
* Los nombres de columna de conjuntos de datos de U-SQL que no son cadenas.

### <a name="python-versions"></a>Versiones de Python

Se admite solo Python 3.5.1 (compilado para Windows).

### <a name="standard-python-modules"></a>Módulos estándar de Python

Se incluyen todos los módulos estándar de Python.

### <a name="additional-python-modules"></a>Módulos adicionales de Python

Además de las bibliotecas estándar de Python, se incluyen varias bibliotecas de python de uso frecuente:

* Pandas
* numpy
* numexpr

### <a name="exception-messages"></a>Mensajes de excepción

Actualmente, se muestra una excepción en el código de Python como un error genérico de vértices. En el futuro, los mensajes de error de trabajo de U-SQL mostrarán el mensaje de excepción de Python.

### <a name="input-and-output-size-limitations"></a>Limitaciones de tamaño de entrada y salida

Cada vértice tiene una cantidad limitada de memoria asignada a él. Actualmente, ese límite es de 6 GB para AU. Dado que las DataFrames de entrada y salida deben existir en la memoria en el código de Python, el tamaño total de la entrada y salida no puede superar los 6 GB.

## <a name="next-steps"></a>Pasos siguientes

* [Información general de Análisis de Microsoft Azure Data Lake](data-lake-analytics-overview.md)
* [Desarrollo de scripts U-SQL mediante Data Lake Tools for Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
* [Uso de funciones de ventana de U-SQL para trabajos de Análisis de Azure Data Lake](data-lake-analytics-use-window-functions.md)
* [Uso de Herramientas de Azure Data Lake para Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md)
