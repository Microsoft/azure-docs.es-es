---
title: 'Introducción manual de datos: Referencia del módulo'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo Introducción manual de datos en Azure Machine Learning para crear un conjunto de datos pequeño escribiendo valores. El conjunto de datos puede tener varias columnas.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/22/2020
ms.openlocfilehash: 3bff9a91f06649487560faef3ab554b2a3d56af3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90908079"
---
# <a name="enter-data-manually-module"></a>Módulo Introducción manual de datos

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Utilice el módulo **Introducción manual de datos** para crear un conjunto de datos pequeño escribiendo valores. El conjunto de datos puede tener varias columnas.
  
Este módulo puede ser útil en escenarios como los siguientes:  
  
- Generación de un pequeño conjunto de valores para las pruebas.  
- Creación de una lista breve de etiquetas.  
- Escritura de una lista de nombres de columna para su inserción en un conjunto de datos.

## <a name="create-a-dataset"></a>Crear un conjunto de datos 
  
1. Agregue el módulo [Introducción manual de datos](./enter-data-manually.md) a la canalización. Puede encontrar el módulo en la categoría **Entrada y salida de datos** de Azure Machine Learning. 
  
1. Para **DataFormat**, seleccione una de las siguientes opciones. Estas opciones determinan cómo se deben analizar los datos proporcionados. Los requisitos para cada formato varían en gran medida, por lo que debe asegurarse de leer los temas relacionados.  
  
   - **ARFF**: formato de archivo de relación de atributos usado por Weka.   
   - **CSV**: formato de valores separados por comas. Para obtener más información, consulte [Convert to CSV](./convert-to-csv.md) (Convertir a CSV).    
   - **SVMLight**: formato usado por Vowpal Wabbit y otros marcos de aprendizaje automático.    
   - **TSV**: formato de valores separados por tabulaciones.

   Si elige un formato y no proporciona datos que cumplan las especificaciones de formato, se produce un error en tiempo de ejecución.
  
1. Haga clic dentro del cuadro de texto **Datos** para empezar a escribir datos. Los formatos siguientes requieren una atención especial:  
  
   - **CSV**: para crear varias columnas, pegue texto separado por comas o escriba varias columnas utilizando comas entre los campos.
  
     Si selecciona la opción **HasHeader**, puede usar la primera fila de valores como encabezado de columna.  
  
     Si anula la selección de esta opción, se usarán los nombres de columnas (Col1, Col2, etc.). Puede agregar o cambiar los nombres de columnas más adelante mediante [Editar metadatos](./edit-metadata.md).  
  
   - **TSV**: para crear varias columnas, pegue texto separado por tabulaciones o escriba varias columnas utilizando tabulaciones entre los campos.  
  
     Si selecciona la opción **HasHeader**, puede usar la primera fila de valores como encabezado de columna.  
  
     Si anula la selección de esta opción, se usarán los nombres de columnas (Col1, Col2, etc.). Puede agregar o cambiar los nombres de columnas más adelante mediante [Editar metadatos](./edit-metadata.md).  
  
   - **ARFF**: pegue en un archivo de formato ARFF existente. Si escribe los valores directamente, no olvide agregar el encabezado opcional y los campos de atributos requeridos al principio de los datos. 

     Por ejemplo, las filas de encabezado y de atributo siguientes pueden agregarse a una lista simple. El encabezado de columna sería `SampleText`. Tenga en cuenta que no se admite el tipo Cadena.
    
     ```text
     % Title: SampleText.ARFF  
     % Source: Enter Data module  
     @ATTRIBUTE SampleText NUMERIC  
     @DATA  
     \<type first data row here>  
     ```

   - **SVMLight**: escriba o pegue los valores con el formato SVMLight.  
  
     Por ejemplo, en el ejemplo siguiente se representa el primer par de líneas del conjunto de datos Blood Donation, en formato SVMight:  
  
     ```text  
     # features are [Recency], [Frequency], [Monetary], [Time]  
     1 1:2 2:50 3:12500 4:98   
     1 1:0 2:13 3:3250 4:28   
     ```  
  
     Al ejecutar el módulo [Introducción manual de datos](./enter-data-manually.md), estas líneas se convierten en un conjunto de datos de columnas y valores de índice como sigue:  
  
     |Col1|Col2|Col3|Col4|Etiquetas|  
     |-|-|-|-|-|  
     |0.00016|0.004|0.999961|0.00784|1|  
     |0|0.004|0.999955|0.008615|1|  
  
1. Utilice la tecla ENTRAR después de cada fila para iniciar una nueva línea.      
     
   Si presiona ENTRAR varias veces para agregar varias filas finales vacías, estas filas se quitarán o recortarán.  
  
   Si crea filas con valores que faltan, siempre puede filtrarlas más adelante.  
  
1. Conecte el puerto de salida a otros módulos y ejecute la canalización.  
  
   Para ver el conjunto de datos, haga clic con el botón derecho en el módulo y seleccione **Visualize** (Visualizar).

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 