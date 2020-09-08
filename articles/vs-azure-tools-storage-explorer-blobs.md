---
title: Administración de recursos de Azure Blob Storage con el Explorador de Storage Azure | Microsoft Docs
description: Administre recursos de Azure Blob Storage con el Explorador de Storage. Cree un contenedor de blobs, vea el contenido del contenedor de blobs, elimine o copie un contenedor de blobs, entre otras acciones.
services: storage
documentationcenter: na
author: cawaMS
manager: paulyuk
editor: ''
ms.assetid: 2f09e545-ec94-4d89-b96c-14783cc9d7a9
ms.service: storage
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/21/2019
ms.author: cawa
ms.openlocfilehash: 07652f2ac88475664f30e8189204b5649f966d16
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88033621"
---
# <a name="manage-azure-blob-storage-resources-with-storage-explorer"></a>Administración de recursos de Azure Blob Storage con el Explorador de Azure Storage

## <a name="overview"></a>Información general

[Azure Blob Storage](storage/blobs/storage-dotnet-how-to-use-blobs.md) es un servicio para almacenar grandes cantidades de datos sin estructura, como texto o datos binarios, a los que puede acceder desde cualquier lugar del mundo a través de HTTP o HTTPS.
Puede usar el almacenamiento de blobs para exponer datos públicamente o para almacenar datos de la aplicación de manera privada. En este artículo, aprenderá a usar el Explorador de Azure Storage para trabajar con blobs y contenedores de blobs.

## <a name="prerequisites"></a>Prerrequisitos

Para completar los pasos de este artículo, necesitará:

* [Descarga e instalación del Explorador de Storage](https://www.storageexplorer.com)
* [Conectarse a una cuenta o a un servicio de almacenamiento de Azure](vs-azure-tools-storage-manage-with-storage-explorer.md#connect-to-a-storage-account-or-service)

## <a name="create-a-blob-container"></a>Creación de un contenedor de blobs

Todos los blobs deben residir en un contenedor de blobs, que no es más que una agrupación lógica de blobs. Una cuenta puede contener un número ilimitado de contenedores y cada contenedor puede almacenar un número ilimitado de blobs.

Los siguientes pasos muestran cómo crear un contenedor de blob en el Explorador de Azure Storage.

1. Abra el Explorador de Storage.
2. En el panel izquierdo, expanda la cuenta de almacenamiento en la que desea crear el contenedor de blobs.
3. Haga clic con el botón derecho en **Contenedores de blobs** y, en el menú contextual, seleccione **Crear contenedor de blobs**.

   ![Menú contextual Crear contenedores de blobs][0]
4. Aparecerá un cuadro de texto debajo de la carpeta **Contenedores de blob** . Escriba el nombre del contenedor de blobs. Consulte [Crear un contenedor](storage/blobs/storage-quickstart-blobs-dotnet.md#create-a-container) para obtener más información sobre las reglas y restricciones en la nomenclatura de los contenedores de blobs.

   ![Crear cuadro de texto Contenedores de blobs][1]
5. Presione **ENTRAR** cuando termine para crear el contenedor de blobs o **Esc** para cancelar la operación. Una vez que el contenedor de blobs se haya creado correctamente, se mostrará en la carpeta **Contenedores de Blob** de la cuenta de almacenamiento seleccionada.

   ![Contenedor de blobs creado][2]

## <a name="view-a-blob-containers-contents"></a>Visualización del contenido de un contenedor de blobs

Los contenedores de blobs contienen blobs y carpetas (que también contienen blobs).

Los siguientes pasos muestran cómo ver el contenido de un contenedor de blobs en el Explorador de Azure Storage:

1. Abra el Explorador de Storage.
2. En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el contenedor de blobs que desea ver.
3. Expanda **Contenedores de blob**de la cuenta de almacenamiento.
4. Haga clic con el botón derecho en el contenedor de blobs que desea ver y (en el menú contextual) seleccione **Abrir editor de contenedores de blobs**.
   También puede hacer doble clic en el contenedor de blobs que desea ver.

   ![Menú contextual Abrir editor de contenedor de blobs][19]
5. El panel principal mostrará el contenido del contenedor de blobs.

   ![Editor de contenedor de blobs][3]

## <a name="delete-a-blob-container"></a>un contenedor de blobs

Los contenedores de blobs se pueden crear fácilmente y eliminarse según sea necesario. (Para ver cómo eliminar blobs individuales, consulte la sección [Administración de blobs de un contenedor de blobs](#managing-blobs-in-a-blob-container)).

Los siguientes pasos muestran cómo eliminar un contenedor de blobs en el Explorador de Azure Storage:

1. Abra el Explorador de Storage.
2. En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el contenedor de blobs que desea ver.
3. Expanda **Contenedores de blob**de la cuenta de almacenamiento.
4. Haga clic con el botón derecho en el contenedor de blobs que desea eliminar y (en el menú contextual) seleccione **Eliminar**.
   También puede presionar **Suprimir** para eliminar el contenedor de blobs actualmente seleccionado.

   ![Menú contextual Eliminar contenedores de blobs][4]
5. Haga clic en **Sí** en el cuadro de diálogo de confirmación.

   ![Cuadro de diálogo de confirmación Eliminar contenedores de blobs][5]

## <a name="copy-a-blob-container"></a>Copia de un contenedor de blobs

Explorador de Azure Storage permite copiar un contenedor de blobs en el Portapapeles y, a continuación, pegarlo en otra cuenta de almacenamiento. (Para ver cómo copiar blobs individuales, consulte la sección [Administración de blobs de un contenedor de blobs](#managing-blobs-in-a-blob-container)).

Los siguientes pasos muestran cómo copiar un contenedor de blobs de una cuenta de almacenamiento a otra.

1. Abra el Explorador de Storage.
2. En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el contenedor de blobs que desea copiar.
3. Expanda **Contenedores de blob**de la cuenta de almacenamiento.
4. Haga clic con el botón derecho en el contenedor de blobs que desea copiar y (en el menú contextual) seleccione **Copy Blob Container**(Copiar contenedor de blobs).

   ![Menú contextual Copiar contenedor de blobs][6]
5. Haga clic con el botón derecho en la cuenta de almacenamiento de "destino" deseada en la que desea pegar el contenedor de blobs y (en el menú contextual) seleccione **Paste Blob Container**(Pegar contenedor de blobs).

   ![Menú contextual Pegar contenedor de blobs][7]

## <a name="get-the-sas-for-a-blob-container"></a>Obtención de la SAS para un contenedor de blobs

Una [firma de acceso compartido (SAS)](storage/common/storage-dotnet-shared-access-signature-part-1.md) ofrece acceso delegado a los recursos en la cuenta de almacenamiento.
Esto significa que puede conceder permisos limitados de los clientes a objetos en su cuenta de almacenamiento durante un período específico y con un conjunto determinado de permisos sin tener que compartir las claves de acceso a las cuentas.

Los siguientes pasos muestran cómo crear una SAS para un contenedor de blobs:

1. Abra el Explorador de Storage.
2. En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el contenedor de blobs del que desea obtener una SAS.
3. Expanda **Contenedores de blob**de la cuenta de almacenamiento.
4. Haga clic con el botón derecho en el contenedor de blobs deseado y (en el menú contextual) seleccione **Get Shared Access Signature**(Obtener firma de acceso compartido).

   ![Menú contextual Obtener SAS][8]
5. En el cuadro de diálogo **Firma de acceso compartido** , especifique la directiva, las fechas de inicio y vencimiento, la zona horaria y los niveles de acceso que desea para el recurso.

   ![Opciones de Obtener SAS][9]
6. Cuando haya terminado de especificar las opciones de SAS, seleccione **Crear**.
7. Después, un segundo cuadro de diálogo **Firma de acceso compartido** mostrará el contenedor de blobs junto con la dirección URL y las QueryStrings que se pueden utilizar para acceder al recurso de almacenamiento.
   Seleccione **Copiar** junto a la dirección URL que desea copiar al Portapapeles.

   ![Copiar direcciones URL de SAS][10]
8. Cuando haya terminado, seleccione **Cerrar**.

## <a name="manage-access-policies-for-a-blob-container"></a>Administración de directivas de acceso para un contenedor de blobs

Los siguientes pasos muestran cómo administrar (agregar y quitar) las directivas de acceso de un contenedor de blobs:

1. Abra el Explorador de Storage.
2. En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el contenedor de blobs cuyas directivas de acceso desea administrar.
3. Expanda **Contenedores de blob**de la cuenta de almacenamiento.
4. Seleccione el contenedor de blobs deseado y (en el menú contextual) seleccione **Manage Access Policies**(Administrar directivas de acceso).

   ![Menú contextual Administrar directivas de acceso][11]
5. El cuadro de diálogo **Directivas de acceso** enumerará las directivas de acceso creadas para el contenedor de blobs seleccionado.

   ![Opciones de Directiva de acceso][12]
6. Siga estos pasos en función de la tarea de administración de directivas de acceso:

   * **Agregar una nueva directiva de acceso**: seleccione **Agregar**. Una vez generada, el cuadro de diálogo **Directivas de acceso** mostrará la directiva de acceso recién agregada (con la configuración predeterminada).
   * **Editar una directiva de acceso**: realice las modificaciones que desee y seleccione **Guardar**.
   * **Quitar una directiva de acceso**: seleccione **Quitar** junto a la directiva de acceso que desea quitar.

## <a name="set-the-public-access-level-for-a-blob-container"></a>Establecimiento del nivel de acceso público para un contenedor de blobs

De manera predeterminada, todos los contenedores de blobs se establecen en "Sin acceso público".

Los siguientes pasos muestran cómo especificar un nivel de acceso público para un contenedor de blobs.

1. Abra el Explorador de Storage.
2. En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el contenedor de blobs cuyas directivas de acceso desea administrar.
3. Expanda **Contenedores de blob**de la cuenta de almacenamiento.
4. Seleccione el contenedor de blobs deseado y (en el menú contextual) seleccione **Set Public Access Level**(Establecer nivel de acceso público).

   ![Menú contextual Establecer nivel de acceso público][13]
5. En el cuadro de diálogo **Set Container Public Access Level** (Establecer nivel de acceso público de contenedor), especifique el nivel de acceso deseado.

   ![Opciones de Establecer nivel de acceso público][14]
6. Seleccione **Aplicar**.

## <a name="managing-blobs-in-a-blob-container"></a>Administración de blobs de un contenedor de blobs

Una vez que haya creado un contenedor de blobs, puede cargar un blob en él, descargar un blob en el equipo local, abrir un blob en el equipo local, etc.

Los siguientes pasos muestran cómo administrar los blobs (y carpetas) en un contenedor de blobs.

1. Abra el Explorador de Storage.
2. En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el contenedor de blobs que desea administrar.
3. Expanda **Contenedores de blob**de la cuenta de almacenamiento.
4. Haga doble clic en el contenedor de blobs que desea ver.
5. El panel principal mostrará el contenido del contenedor de blobs.

   ![Ver contenedor de blobs][3]
6. El panel principal mostrará el contenido del contenedor de blobs.
7. Siga estos pasos según la tarea que desea realizar:

   * **Cargar archivos en un contenedor de blobs**

     1. En la barra de herramientas del panel principal, seleccione **Cargar** y, luego, **Cargar archivos** en el menú desplegable.

        ![Menú Cargar archivos][15]
     2. En el cuadro de diálogo **Cargar archivos**, seleccione el botón de puntos suspensivos (**...**) a la derecha del cuadro de texto **Archivos** para seleccionar los archivos que desea cargar.

        ![Opciones de Cargar archivos][16]
     3. Especifique el **tipo de blob**. Consulte [Crear un contenedor](storage/blobs/storage-quickstart-blobs-dotnet.md#create-a-container) para obtener más información.
     4. Opcionalmente, especifique la carpeta de destino en la que se cargarán los archivos seleccionados. Si la carpeta de destino no existe, se creará.
     5. Seleccione **Cargar**.
   * **Cargar una carpeta en un contenedor de blobs**

     1. En la barra de herramientas del panel principal, seleccione **Cargar**, y luego **Cargar carpeta** en el menú desplegable.

        ![Menú Cargar carpeta][17]
     2. En el cuadro de diálogo **Cargar carpeta**, seleccione el botón de puntos suspensivos (**...**) a la derecha del cuadro de texto **Carpeta** para seleccionar la carpeta cuyo contenido desea cargar.

        ![Opciones de Cargar carpeta][18]
     3. Especifique el **tipo de blob**. Consulte [Crear un contenedor](storage/blobs/storage-quickstart-blobs-dotnet.md#create-a-container) para obtener más información.
     4. Opcionalmente, especifique la carpeta de destino en la que se cargará el contenido de la carpeta seleccionada. Si la carpeta de destino no existe, se creará.
     5. Seleccione **Cargar**.
   * **Descargar un blob en el equipo local**

     1. Seleccione el blob que desea descargar.
     2. En la barra de herramientas del panel principal, seleccione **Descargar**.
     3. En el cuadro de diálogo **Specify where to save the downloaded blob** (Especificar dónde se guarda el blob descargado), especifique la ubicación en que desea descargar el blob y el nombre que desee darle.  
     4. Seleccione **Guardar**.
   * **Abrir un blob en el equipo local**

     1. Seleccione el blob que desea abrir.
     2. En la barra de herramientas del panel principal, seleccione **Abrir**.
     3. El blob se descargará y se abrirá con la aplicación asociada con el tipo de archivo subyacente del blob.
   * **Copiar un blob en el Portapapeles**

     1. Seleccione el blob que desea copiar.
     2. En la barra de herramientas del panel principal, seleccione **Copiar**.
     3. En el panel izquierdo, navegue a otro contenedor de blobs y haga doble clic en él para verlo en el panel principal.
     4. En la barra de herramientas del panel principal, seleccione **Pegar** para crear una copia del blob.
   * **Eliminar un blob**

     1. Seleccione el blob que desea eliminar.
     2. En la barra de herramientas del panel principal, seleccione **Eliminar**.
     3. Haga clic en **Sí** en el cuadro de diálogo de confirmación.

## <a name="next-steps"></a>Pasos siguientes

* Vea las [notas de la versión y los vídeos más recientes del Explorador de Storage](https://www.storageexplorer.com).
* Obtenga información acerca de cómo [crear aplicaciones con blobs, tablas, colas y archivos de Azure](https://azure.microsoft.com/documentation/services/storage/).

[0]: ./media/vs-azure-tools-storage-explorer-blobs/blob-containers-create-context-menu.png
[1]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-create.png
[2]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-create-done.png
[3]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-editor.png
[4]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-delete-context-menu.png
[5]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-delete-confirmation.png
[6]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-copy-context-menu.png
[7]: ./media/vs-azure-tools-storage-explorer-blobs/blob-containers-paste-context-menu.png
[8]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-get-sas-context-menu.png
[9]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-get-sas-options.png
[10]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-get-sas-urls.png
[11]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-manage-access-policies-context-menu.png
[12]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-manage-access-policies-options.png
[13]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-set-public-access-level-context-menu.png
[14]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-set-public-access-level-options.png
[15]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-files-menu.png
[16]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-files-options.png
[17]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-folder-menu.png
[18]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-folder-options.png
[19]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-open-editor-context-menu.png
