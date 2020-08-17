---
title: 'Tutorial: Introducción a la creación de un área de trabajo de Synapse'
description: En este tutorial, aprenderá a crear un área de trabajo de Synapse, un grupo de SQL y un grupo de Apache Spark.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.date: 07/20/2020
ms.openlocfilehash: b4d48dcc8f09ae8e2ec3bb198f8864de1c945682
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87093903"
---
# <a name="create-a-synapse-workspace"></a>Creación de un área de trabajo de Synapse

En este tutorial, aprenderá a crear un área de trabajo de Synapse, un grupo de SQL y un grupo de Apache Spark. 

## <a name="prepare-a-storage-account"></a>Preparación de una cuenta de almacenamiento

1. Abra [Azure Portal](https://portal.azure.com).
1. Cree una cuenta de almacenamiento con la siguiente configuración:

    |Pestaña|Configuración | Valor sugerido | Descripción |
    |---|---|---|---|
    |Aspectos básicos|**Nombre de cuenta de almacenamiento**| Elija cualquier nombre.| En este documento, usaremos el nombre **contosolake**.|
    |Aspectos básicos|**Tipo de cuenta**| **StorageV2** ||
    |Aspectos básicos|**Ubicación**|Elija cualquier ubicación.| Se recomienda que el área de trabajo de Azure Synapse Analytics y la cuenta de Azure Data Lake Storage Gen2 se encuentren en la misma región.|
    |Avanzado|**Data Lake Storage Gen2**|**Enabled**| Azure Synapse solo funciona con cuentas de almacenamiento en las que esté habilitado este valor.|
    |||||

1. Después de crear la cuenta de almacenamiento, seleccione **Control de acceso (IAM)** en el panel izquierdo. Luego, asigne los siguientes roles o asegúrese de que ya estén asignados:
    * Asígnese el rol **Propietario**.
    * Asígnese el rol **Propietario de datos de Storage Blob**.
1. En el panel izquierdo, seleccione **Contenedores** y cree un contenedor.
1. Puede asignar cualquier nombre al contenedor. En este documento, llamaremos **users** al contenedor.
1. Acepte el valor predeterminado **Nivel de acceso público** y, después, seleccione **Crear**.

En el paso siguiente, configurará el área de trabajo de Azure Synapse para usar esta cuenta de almacenamiento como su cuenta de almacenamiento "principal" y el contenedor para almacenar los datos del área de trabajo. El área de trabajo almacena datos en tablas de Apache Spark. Almacena los registros de aplicaciones de Spark en una carpeta denominada **/synapse/workspacename**.

## <a name="create-a-synapse-workspace"></a>Creación de un área de trabajo de Synapse

1. Abra [Azure Portal](https://portal.azure.com) y, en la parte superior, busque **Synapse**.
1. En los resultados de la búsqueda, en **Servicios**, seleccione **Azure Synapse Analytics (versión preliminar de las áreas de trabajo)** .
1. Seleccione **Agregar** para crear un área de trabajo con esta configuración:

    |Pestaña|Configuración | Valor sugerido | Descripción |
    |---|---|---|---|
    |Aspectos básicos|**Workspace name** (Nombre del área de trabajo)|Puede asignarle un nombre.| En este documento, usaremos **myworkspace**.|
    |Aspectos básicos|**Región**|Coincidencia de la región de la cuenta de almacenamiento.|

1. En **Select Data Lake Storage Gen 2** (Seleccionar Data Lake Storage Gen 2), seleccione la cuenta y el contenedor que creó anteriormente.
1. Seleccione **Revisar y crear** > **Crear**. El área de trabajo estará lista en unos minutos.

## <a name="verify-access-to-the-storage-account"></a>Comprobación del acceso a la cuenta de almacenamiento

Es posible que las identidades administradas del área de trabajo de Azure Synapse ya tengan acceso a la cuenta de almacenamiento. Siga estos pasos para asegurarse:

1. Abra [Azure Portal](https://portal.azure.com) y la cuenta de almacenamiento principal elegida para el área de trabajo.
1. Seleccione **Control de acceso (IAM)** en el panel izquierdo.
1. Asigne los siguientes roles o asegúrese de que ya estén asignados. Se usa el mismo nombre para la identidad y para el nombre del área de trabajo.
    * Para el rol **Colaborador de datos de Storage Blob** de la cuenta de almacenamiento, asigne **myworkspace** como identidad del área de trabajo.
    * Asigne **myworkspace** como el nombre del área de trabajo.

1. Seleccione **Guardar**.

## <a name="open-synapse-studio"></a>Abrir Synapse Studio

Una vez creada el área de trabajo de Azure Synapse, hay dos maneras de abrir Synapse Studio:

* Abrir el área de trabajo de Synapse en [Azure Portal](https://portal.azure.com). En la parte superior de la sección **Información general**, seleccione **Iniciar Synapse Studio**.
* Vaya a `https://web.azuresynapse.net` e inicie sesión en el área de trabajo.

## <a name="create-a-sql-pool"></a>Creación de un grupo de SQL

1. En Synapse Studio, en el panel izquierdo, seleccione **Administrar** > **Grupos de SQL**.
1. Seleccione **Nuevo** y especifique estos valores:

    |Configuración | Valor sugerido | 
    |---|---|---|
    |**Nombre del grupo de SQL**| **SQLDB1**|
    |**Nivel de rendimiento**|**DW100C**|
    |||

1. Seleccione **Revisar y crear** > **Crear**. El grupo de SQL estará listo en unos minutos. El grupo de SQL se asocia con una base de datos del grupo de SQL también denominada **SQLDB1**.

Los grupos de SQL consumen recursos facturables mientras están activos. Puede pausar los grupos más adelante para reducir los costos.

## <a name="create-an-apache-spark-pool"></a>Creación de un grupo de Apache Spark

1. En Synapse Studio, en el lado izquierdo, seleccione **Administrar** > **Grupos de Apache Spark**.
1. Seleccione **Nuevo** y especifique estos valores:

    |Configuración | Valor sugerido | 
    |---|---|---|
    |**Nombre del grupo de Apache Spark**|**Spark1**
    |**Tamaño del nodo**| **Pequeño**|
    |**Número de nodos**| Establezca el valor mínimo en 3 y el máximo en 3|

1. Seleccione **Revisar y crear** > **Crear**. El grupo de Apache Spark estará listo en unos segundos.

> [!NOTE]
> A pesar del nombre, un grupo de Apache Spark no es como un grupo de SQL. Es solo un conjunto de metadatos básicos que se usan para indicar al área de trabajo de Azure Synapse cómo interactuar con Spark.

Al ser metadatos, los grupos de Spark no se pueden iniciar ni detener.

Cuando se realiza una actividad de Spark en Azure Synapse, se especifica el grupo de Spark que se va a usar. El grupo indica a Azure Synapse el número de recursos de Spark que se van a usar. Solo paga por los recursos que utiliza. Al dejar de usar el grupo de forma activa, los recursos agotarán el tiempo de espera y se reciclarán automáticamente.

> [!NOTE]
> Las bases de datos de Spark se crean de forma independiente de los grupos de Spark. Un área de trabajo siempre tiene una base de datos de Spark denominada **default**. Puede crear bases de datos de Spark adicionales.

## <a name="the-sql-on-demand-pool"></a>El grupo de SQL a petición

Todas las áreas de trabajo incluyen un grupo precompilado llamado **SQL on-demand**. Este grupo no se puede eliminar. Este grupo permite trabajar con SQL sin tener que crear ni pensar en la administración de un grupo de SQL en Azure Synapse.

A diferencia de los otros tipos de grupos, la facturación de SQL on-demand se basa en la cantidad de datos que se examinan para ejecutar la consulta (no en el número de recursos usados para ello).

* SQL on-demand tiene sus propias bases de datos de SQL On-Demand, que existen de forma independiente de cualquier grupo de SQL On-Demand.
* Un área de trabajo siempre tiene exactamente un grupo de SQL On-Demand denominado **SQL on-demand**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Análisis mediante un grupo de SQL](get-started-analyze-sql-pool.md)
