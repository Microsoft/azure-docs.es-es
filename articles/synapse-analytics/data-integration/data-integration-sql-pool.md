---
title: Ingesta de datos en un grupo de SQL
description: Aprenda a ingerir datos en un grupo de SQL en Azure Synapse Analytics
services: synapse-analytics
author: djpmsft
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: daperlov
ms.reviewer: jrasnick
ms.openlocfilehash: 02efaf3f0382a7af63717e777036637de2bbec25
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90033207"
---
# <a name="ingest-data-into-a-sql-pool"></a>Ingesta de datos en un grupo de SQL

En este artículo, aprenderá a ingerir datos desde una cuenta de almacenamiento de Azure Data Lake Gen2 en un grupo de SQL mediante Azure Synapse Analytics.

## <a name="prerequisites"></a>Prerrequisitos

* **Suscripción de Azure**: Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* **Cuenta de Azure Storage**: Puede usar Azure Data Lake Storage Gen2 como almacén de datos de *origen*. Si no tiene una cuenta de almacenamiento, consulte [Creación de una cuenta de Azure Storage](../../storage/blobs/data-lake-storage-quickstart-create-account.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) para crear una.
* **Azure Synapse Analytics**: Puede usar un grupo de SQL como almacén de datos *receptor*. Si no tiene una instancia de Azure Synapse Analytics, consulte [Creación de un grupo de SQL](../../azure-sql/database/single-database-create-quickstart.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) para ver los pasos para crear una.

## <a name="create-linked-services"></a>Crear servicios vinculados

En Azure Synapse Analytics, un servicio vinculado es donde se define la información de conexión a otros servicios. En esta sección, agregará un servicio vinculado de Azure Synapse Analytics y Azure Data Lake Storage Gen2.

1. Abra la experiencia del usuario de Azure Synapse Studio y vaya a la pestaña **Administrar**.
1. En **Conexiones externas**, seleccione **Servicios vinculados**.
1. Para agregar un servicio vinculado, seleccione **Nuevo**.
1. Seleccione el icono de Azure Data Lake Storage Gen2 de la lista y seleccione **Continuar**.
1. Especifique las credenciales de autenticación. Actualmente, la clave de cuenta, la entidad de servicio y la identidad administrada son los tipos de autenticación admitidos. Seleccione Probar conexión para verificar que las credenciales son correctas. Cuando haya terminado, seleccione **Crear**.
1. Repita los pasos 3-5, pero, en lugar de Azure Data Lake Storage Gen2, seleccione el icono de Azure Synapse Analytics y escriba las credenciales de conexión correspondientes. En el caso de Azure Synapse Analytics, se admiten actualmente la autenticación de SQL, la identidad administrada y la entidad de servicio.

## <a name="create-pipeline"></a>Creación de una canalización

Una canalización contiene el flujo lógico para una ejecución de un conjunto de actividades. En esta sección, creará una canalización que contiene una actividad de copia que ingiere datos de ADLS Gen2 en un grupo de SQL.

1. Vaya a la pestaña **Orchestrate** (Orquestar). Seleccione el icono de signo de más situado junto al encabezado de canalizaciones y seleccione **Canalización**.
1. En **Mover y transformar** en el panel de actividades, arrastre **Copiar datos** al lienzo de la canalización.
1. Seleccione la actividad de copia y vaya a la pestaña **Origen**. Seleccione **Nuevo** para crear un conjunto de datos de origen.
1. Seleccione Azure Data Lake Storage Gen2 como almacén de datos y después seleccione Continuar.
1. Seleccione DelimitedText como formato y después seleccione Continuar.
1. En el panel Establecer propiedades, seleccione el servicio vinculado de ADLS que creó. Especifique la ruta de acceso del archivo de los datos de origen y especifique si la primera fila tiene un encabezado. Puede importar el esquema del almacén de archivos o de un archivo de ejemplo. Seleccione Aceptar cuando termine.
1. Haga clic en la pestaña **Receptor**. Seleccione **Nuevo** para crear un conjunto de datos de receptor.
1. Seleccione Azure Synapse Analytics como almacén de datos y después seleccione Continuar.
1. En el panel Establecer propiedades, seleccione el servicio vinculado de Azure Synapse Analytics que creó. Si está escribiendo en una tabla existente, selecciónela del menú desplegable. De lo contrario, active **Editar** y escriba el nombre de la nueva tabla. Seleccione Aceptar cuando haya terminado.
1. Si va a crear una tabla, habilite **Auto create table** (Creación automática de tabla) en el campo opción de tabla.

## <a name="debug-and-publish-pipeline"></a>Depuración y publicación de la canalización

Una vez que haya terminado de configurar la canalización, puede ejecutar una depuración antes de publicar los artefactos para verificar que todo esté correcto.

1. Para depurar la canalización, seleccione **Depurar** en la barra de herramientas. Verá el estado de ejecución de la canalización en la pestaña **Output** (Salida) en la parte inferior de la ventana. 
1. Una vez que la canalización se pueda ejecutar correctamente, en la barra de herramientas superior, seleccione **Publicar todo**. Esta acción publica las entidades (conjuntos de datos y canalizaciones) que creó en el servicio de Synapse Analytics.
1. Espere a que aparezca el mensaje **Successfully published** (Publicado correctamente). Para ver los mensajes de notificación, seleccione el botón de campana en la esquina superior derecha. 


## <a name="trigger-and-monitor-the-pipeline"></a>Activación y supervisión de la canalización

En este paso, desencadenará manualmente la canalización que publicó en el paso anterior. 

1. Seleccione **Add Trigger** (Agregar desencadenador) en la barra de herramientas y, después, seleccione **Trigger Now** (Desencadenar ahora). En la página **Pipeline Run** (Ejecución de canalización), seleccione **Finish** (Finalizar).  
1. Vaya a la pestaña **Supervisar** que se encuentra en la barra lateral izquierda. Verá una ejecución de canalización que se desencadena de forma manual. Puede usar los vínculos de la columna **Actions** (Acciones) para ver los detalles de la actividad y volver a ejecutar la canalización.
1. Para ver las ejecuciones de actividad asociadas con la ejecución de canalización, seleccione el vínculo **View Activity Runs** (Ver ejecuciones de actividad) en la columna **Actions** (Acciones). En este ejemplo, solo hay una actividad, así que solo verá una entrada en la lista. Para ver detalles sobre la operación de copia, seleccione el vínculo **Details** (Detalles) (icono de gafas) en la columna **Actions** (Acciones). Seleccione **Pipeline Runs** (Ejecuciones de canalización) en la parte superior para volver a la vista Pipeline Runs (Ejecuciones de canalización). Para actualizar la vista, seleccione **Refresh** (Actualizar).
1. Verifique que los datos se han escrito correctamente en el grupo de SQL.


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la integración de datos para Synapse Analytics, consulte el artículo [Ingesta de datos en Azure Data Lake Storage Gen2](data-integration-data-lake.md).
