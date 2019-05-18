---
title: Asignación de transformación de fila Alter del flujo de datos de Azure Data Factory
description: Cómo actualizar bases de datos destino mediante Azure Data Factory asignación flujo Alter fila transformación de datos
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/12/2019
ms.openlocfilehash: f0ac5bb36079983b10e4d86cc776bd4e5ee6817d
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65520157"
---
# <a name="azure-data-factory-alter-row-transformation"></a>Transformación de fila Alter factoría de datos de Azure

Usar la transformación de fila Alter para establecer directivas de insert, delete, update y upsert en filas. Puede agregar condiciones de uno a varios como expresiones. Cada una de esas condiciones puede dar lugar a una fila (o filas) que se inserta, actualiza, elimina o upsert. ALTER fila puede generar las acciones de DDL y DML en la base de datos.

[!INCLUDE [notes](../../includes/data-factory-data-flow-preview.md)]

![Modificar la configuración de la fila](media/data-flow/alter-row1.png "Alter Settings de fila")

> [!NOTE]
> Las transformaciones de filas ALTER solo funcionará en la base de datos receptores en el flujo de datos. Las acciones que se asignan a las filas (insert, update, delete, upsert) no se producirán durante las sesiones de depuración. Debe agregar una tarea de ejecución de flujo de datos a una canalización y usar la depuración de la canalización o desencadenadores para aplicar las directivas de la fila de alter en las tablas de base de datos.

## <a name="view-policies"></a>Ver directivas

Cambiar el modo de flujo de depuración de datos en y, a continuación, ver los resultados de las directivas de la fila alter en el panel de vista previa de datos. Ejecución de una fila de alter en el modo de datos de flujo de depuración no producirá las acciones de DDL o DML con respecto a su destino. Con el fin de que esas acciones que se produzca, ejecute el flujo de datos dentro de una actividad de flujo de datos ejecutar dentro de una canalización.

![Modificar las directivas de la fila](media/data-flow/alter-row3.png "modificar directivas de fila")

Esto le permitirá comprobar y ver el estado de cada fila en función de sus condiciones. Representa el icono para cada instrucción insert, update, delete y upsert acción que tendrá lugar en el flujo de datos, que indica qué acción llevará a cabo cuando se ejecuta el flujo de datos dentro de una canalización.

## <a name="sink-settings"></a>Configuración de receptor

Debe tener una base de datos de tipo de fila Alter funcione de receptor. En el receptor de la configuración, debe establecer cada acción que se permita.

![ALTER receptor fila](media/data-flow/alter-row2.png "Alter receptor de fila")

Es el comportamiento predeterminado en el flujo de datos de ADF con receptores de la base de datos insertar filas. Si desea permitir que las actualizaciones, realiza una operación Upsert y eliminaciones, así, también debe comprobar estos cuadros en el receptor para permitir que las acciones.

> [!NOTE]
> Si su inserciones, actualizaciones o realiza una operación Upsert modifica el esquema de la tabla de destino en el receptor, se producirá un error en el flujo de datos. Para modificar el esquema de destino en la base de datos, debe elegir la opción "Volver a crear la tabla" en el receptor. Esto se quite y vuelva a crear la tabla con la nueva definición de esquema.

## <a name="next-steps"></a>Pasos siguientes

Después de la transformación Alter fila, puede desear [los datos de receptor en un almacén de datos de destino](data-flow-sink.md).
