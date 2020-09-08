---
title: 'Inicio rápido: Creación de un blob con Azure Portal'
titleSuffix: Azure Storage
description: En esta guía de inicio rápido, utilizará Azure Portal en el almacenamiento de objetos (Blob). Después, puede usar Azure Portal para cargar un blob en Azure Storage, descargar un blob o enumerar los blobs de un contenedor.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.date: 04/16/2020
ms.author: tamram
ms.openlocfilehash: f2e18b060aabcb849fb8e17722c530d199ebdbb8
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067784"
---
# <a name="quickstart-upload-download-and-list-blobs-with-the-azure-portal"></a>Inicio rápido: Carga, descarga y enumeración de blobs con Azure Portal

En esta guía de inicio rápido, aprenderá a usar [Azure Portal](https://portal.azure.com/) para crear un contenedor en Azure Storage y para cargar y descargar los blobs en bloques en ese contenedor.

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

## <a name="create-a-container"></a>Crear un contenedor

Para crear un contenedor en Azure Portal, siga estos pasos:

1. Vaya a la nueva cuenta de almacenamiento en Azure Portal.
2. En el menú izquierdo de la cuenta de almacenamiento, desplácese a la sección **Blob service** y seleccione **Containers**.
3. Seleccione el botón **+ Contenedor**.
4. Escriba un nombre para el nuevo contenedor. El nombre del contenedor debe escribirse en minúsculas, comenzar por una letra o un número, y solo puede incluir letras, números y el carácter de guión (-). Para más información sobre la nomenclatura de contenedores y blobs, consulte [Naming and Referencing Containers, Blobs, and Metadata](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Asignación de nombres y referencia a contenedores, blobs y metadatos).
5. Establezca el nivel de acceso público al contenedor. El nivel predeterminado es **Private (no anonymous access)** [Privado (sin acceso anónimo)].
6. Seleccione **Aceptar** para crear el contenedor.

    ![Captura de pantalla que muestra cómo crear un contenedor en Azure Portal](media/storage-quickstart-blobs-portal/create-container.png)

## <a name="upload-a-block-blob"></a>Carga de un blob en bloques

Los blobs en bloques se componen de bloques de datos que se ensamblan para formar un blob. En la mayoría de los escenarios en los que se usa Blog Storage se emplean blobs en bloques. Resultan ideales para almacenar datos de texto y binarios en la nube, como archivos, imágenes y vídeos. Esta guía de inicio rápido muestra cómo trabajar con blobs en bloques.

Para cargar un blob en bloques en el nuevo contenedor en Azure Portal, siga estos pasos:

1. En Azure Portal, vaya al contenedor que creó en la sección anterior.
1. Seleccione el contenedor para mostrar una lista de los blobs que contiene. Este contenedor es nuevo, por lo que aún no contiene blobs.
1. Seleccione el botón **Cargar** para abrir la hoja Cargar y examine el sistema de archivos local para buscar un archivo para cargarlo como un blob en bloques. Opcionalmente, puede expandir la sección de **opciones avanzadas** para configurar otros valores de la operación de carga.

    ![Captura de pantalla que muestra cómo cargar un blob desde la unidad local](media/storage-quickstart-blobs-portal/upload-blob.png)

1. Seleccione botón **Upload** para cargar el blob.
1. Cargue tantos blobs como desee de esta manera. Verá que ahora aparecen los nuevos blobs dentro del contenedor.

## <a name="download-a-block-blob"></a>Descarga de un blob en bloques

Puede descargar un blob en bloques para mostrarlo en el explorador o guardarlo en el sistema de archivos local. Para descargar un blob en bloques, siga estos pasos:

1. Vaya a la lista de blobs que cargó en la sección anterior.
1. Haga clic con el botón derecho en el blob que va a descargar y seleccione **Descargar**.

    ![Captura de pantalla que muestra cómo descargar un blob](media/storage-quickstart-blobs-portal/download-blob.png)

## <a name="delete-a-block-blob"></a>Eliminación de un blob en bloques

Puede descargar un blob en bloques para mostrarlo en el explorador o guardarlo en el sistema de archivos local. Para descargar un blob en bloques, siga estos pasos:

1. Vaya a la lista de blobs que cargó en la sección anterior.
1. Seleccione los blobs que desea eliminar y seleccione **Eliminar** en la barra de acciones superior.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar todos los recursos creados en este inicio rápido, basta con eliminar el contenedor. También se eliminarán todos los blobs del contenedor.

Para eliminar el contenedor:

1. En Azure Portal, vaya a la lista de contenedores en la cuenta de almacenamiento.
1. Seleccione el contenedor que desee eliminar.
1. Seleccione el botón **Más** ( **...** ) y, después, **Eliminar**.
1. Confirme que desea eliminar el contenedor.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido ha aprendido a transferir archivos entre un disco local y Azure Blob Storage mediante Azure Portal. Para más información sobre cómo trabajar con Blob Storage, continúe con los procedimientos de Blob Storage.

> [!div class="nextstepaction"]
> [Procedimientos de las operaciones de Blob Storage](storage-dotnet-how-to-use-blobs.md)
