---
title: 'Exportación de datos: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar el módulo Exportación de datos en Azure Machine Learning para guardar los resultados, los datos intermedios y los datos de trabajo de las canalizaciones en los destinos de almacenamiento en la nube fuera de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 07/28/2020
ms.openlocfilehash: 5964ed9773d45b809fc79252eabc754b1b04af84
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907986"
---
# <a name="export-data-module"></a>Módulo Exportación de datos

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Utilice este módulo para guardar los resultados, los datos intermedios y los datos de trabajo de las canalizaciones en los destinos de almacenamiento en la nube. 

Este módulo admite la exportación de los datos a los siguientes servicios de datos en la nube:

- Azure Blob Container
- Recurso compartido de archivos de Azure
- Azure Data Lake Storage Gen1
- Azure Data Lake Storage Gen2
- Azure SQL Database

Antes de exportar los datos, debe registrar un almacén de datos en el área de trabajo de Azure Machine Learning. Para más información, consulte [Acceso a los datos en los servicios de almacenamiento de Azure](../how-to-access-data.md).

## <a name="how-to-configure-export-data"></a>Procedimiento para configurar la exportación de datos

1. Agregue el módulo **Exportación de datos** a la canalización en el diseñador. Puede encontrar este módulo en la categoría **Entrada y salida**.

1. Conecte **Exportación de datos** al módulo que contiene los datos que desea exportar.

1. Seleccione **Exportación de datos** para abrir el panel **Propiedades**.

1. En **Almacén de datos**, seleccione un almacén de datos existente en la lista desplegable. También puede crear un nuevo almacén de datos. Para ver cómo, consulte [Acceso a los datos en los servicios de almacenamiento de Azure](../how-to-access-data.md).

    > [!NOTE]
    > No se admite la exportación de datos de un tipo determinado a una columna de SQL Database especificada como otro tipo de datos. No es necesario que la tabla de destino exista primero.

1. La casilla **Regenerate output** (Regenerar salida), decide si se debe ejecutar el módulo para regenerar la salida en tiempo de ejecución. 

    De forma predeterminada, no está seleccionada, lo que significa que si el módulo se ha ejecutado con los mismos parámetros anteriormente, el sistema reutilizará la salida de la última ejecución para reducir el tiempo de ejecución. 

    Si se selecciona, el sistema volverá a ejecutar el módulo para regenerar la salida.

1. Defina la ruta del almacén de datos en el que se encuentran los datos. La ruta es una ruta de acceso relativa. No se permiten las rutas de acceso vacías o de dirección URL.


1. Para **Formato de archivo**, seleccione el formato en el que se deben almacenar los datos.
 
1. Envíe la canalización.

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
