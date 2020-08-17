---
title: Traslado de archivos entre almacenes basados en archivos
description: Obtenga información sobre cómo usar una plantilla de solución para mover archivos entre almacenamiento basado en archivos con Azure Data Factory.
services: data-factory
author: dearandyxu
ms.author: yexu
ms.reviewer: ''
manager: shwang
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 7/12/2019
ms.openlocfilehash: f6baea73c0c4964bb3937304603a2a92a13d52b2
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86522727"
---
# <a name="move-files-with-azure-data-factory"></a>Movimiento de archivos con Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

La actividad de copia de ADF tiene compatibilidad integrada en el escenario de "traslado" al copiar archivos binarios entre almacenes de almacenamiento.  Para habilitarlo, establezca "deleteFilesAfterCompletion" en true para la actividad de copia. De este modo, la actividad de copia eliminará los archivos del almacén de origen de los datos una vez completado el trabajo. 

En este artículo se describe una plantilla de solución como otro enfoque que aprovecha el flujo de control flexible de ADF junto con la actividad de copia y la actividad de eliminación para lograr el mismo escenario. Uno de los escenarios comunes de uso de esta plantilla: los archivos se colocan continuamente en una carpeta de aterrizaje del almacén de origen. Mediante la creación de un desencadenador de programación, la canalización de ADF puede mover de forma periódica esos archivos del almacén de origen al de destino.  Para "mover los archivos", la canalización de ADF los obtiene de la carpeta de aterrizaje, los copia en otra carpeta del almacén de destino y, después, elimina los mismos archivos de la carpeta de aterrizaje en el almacén de origen.

> [!NOTE]
> Tenga en cuenta que esta plantilla está diseñada para mover archivos en lugar de carpetas.  Tendrá que tener mucho cuidado si quiere mover la carpeta cambiando el conjunto de datos para que solo contenga una ruta de carpeta y, después, usar la actividad de copia y la de eliminación para hacer referencia al mismo conjunto de datos que representa una carpeta. El motivo es que tiene que asegurarse de que NO lleguen nuevos archivos a la carpeta entre la operación de copia y la operación de eliminación. Si llegan archivos nuevos a la carpeta en el momento en que la actividad de copia acaba de completar el trabajo de copia pero la actividad de eliminación aún no ha empezado, es posible que la actividad de eliminación elimine este nuevo archivo entrante que NO se ha copiado aún en el destino mediante la eliminación de toda la carpeta.

## <a name="about-this-solution-template"></a>Acerca de esta plantilla de solución

Esta plantilla obtiene los archivos del almacén basado en archivos de origen. Después, mueve cada uno al almacén de destino.

La plantilla contiene cinco actividades:
- **GetMetadata** obtiene la lista de objetos, incluidos los archivos y las subcarpetas de la carpeta en el almacén de origen. No recuperará los objetos de forma recursiva. 
- **Filter** filtra la lista de objetos de la actividad **GetMetadata** para seleccionar solo los archivos. 
- **ForEach** obtiene la lista de archivos de la actividad **Filter** y, después, recorre en iteración la lista y pasa cada archivo a las actividades Copy y Delete.
- **Copy** copia un archivo del origen en el almacén de destino.
- **Delete** elimina el mismo archivo del almacén de origen.

La plantilla define cuatro parámetros:
- *SourceStore_Location* es la ruta de acceso de la carpeta del almacén de origen desde el que quiere mover los archivos. 
- *SourceStore_Directory* es la ruta de acceso de la subcarpeta del almacén de origen desde el que quiere mover los archivos.
- *DestinationStore_Location* es la ruta de acceso de la carpeta del almacén de destino a donde quiere mover los archivos. 
- *DestinationStore_Directory* es la ruta de acceso de la subcarpeta del almacén de destino a donde quiere mover los archivos.

## <a name="how-to-use-this-solution-template"></a>Uso de esta plantilla de solución

1. Vaya a la plantilla **Mover archivos**. Seleccione la conexión existente o cree **una** al almacén de archivos de origen desde el que quiera mover los archivos. Tenga en cuenta que **DataSource_Folder** y **DataSource_File** hacen referencia a la misma conexión del almacén de archivos de origen.

    ![Creación de una nueva conexión con el origen](media/solution-template-move-files/move-files1.png)

2. Seleccione la conexión existente o cree **una** al almacén de archivos de destino al que quiera mover los archivos.

    ![Creación de una nueva conexión con el destino](media/solution-template-move-files/move-files2.png)

3. Seleccione la pestaña **Usar esta plantilla**.
    
4. Verá la canalización, como en el ejemplo siguiente:

    ![Visualización de la canalización](media/solution-template-move-files/move-files4.png)

5. Seleccione **Depurar**, escriba los **parámetros** y, a continuación, seleccione **Finalizar**.   Los parámetros son las rutas de acceso de la carpeta desde la que quiere mover los archivos y a la que quiere moverlos. 

    ![Ejecución de la canalización](media/solution-template-move-files/move-files5.png)

6. Revise el resultado.

    ![Revisión del resultado](media/solution-template-move-files/move-files6.png)

## <a name="next-steps"></a>Pasos siguientes

- [Copia de archivos nuevos y cambiados por LastModifiedDate con Azure Data Factory](solution-template-copy-new-files-lastmodifieddate.md)

- [Copia de archivos de varios contenedores con Azure Data Factory](solution-template-copy-files-multiple-containers.md)