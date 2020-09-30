---
title: Recursos compartidos de administración de Azure Data Box Gateway | Microsoft Docs
description: Describe cómo usar Azure Portal para administrar recursos compartidos en Azure Data Box Gateway.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: how-to
ms.date: 03/25/2019
ms.author: alkohli
ms.openlocfilehash: 785fcbbcde0c4efe3c2d1cd84bf19bd03371ab06
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90883549"
---
# <a name="use-the-azure-portal-to-manage-shares-on-your-azure-data-box-gateway"></a>Uso de Azure Portal para administrar recursos compartidos en Azure Data Box Gateway 

En este artículo se describe cómo administrar recursos compartidos en Azure Data Box Gateway. Azure Data Box Gateway se puede administrar a través de Azure Portal o de la interfaz de usuario web local. Use Azure Portal para agregar, eliminar y actualizar recursos compartidos o sincronizar la clave de almacenamiento para la cuenta de almacenamiento asociada con los recursos compartidos.

## <a name="about-shares"></a>Acerca de los recursos compartidos

Para transferir datos a Azure, es preciso crear recursos compartidos en Azure Data Box Gateway, que son recursos compartidos en la nube. Los datos de estos recursos compartidos se cargan automáticamente en la nube y se les aplican todas las funciones de la nube, como la actualización y la sincronización de claves de almacenamiento. Use los recursos compartidos en la nube cuando desee que los datos del dispositivo se inserten automáticamente en su cuenta de almacenamiento en la nube.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Agregar un recurso compartido
> * Eliminación de un recurso compartido
> * Actualización de recursos compartidos
> * Sincronizar clave de almacenamiento


## <a name="add-a-share"></a>Agregar un recurso compartido

Siga estos pasos en Azure Portal para crear un recurso compartido.

1. En Azure Portal, vaya al recurso Data Box Gateway y, después, a **Información general**. Haga clic en **+ Agregar recurso compartido** en la barra de comandos.
2. En **Agregar recurso compartido**, especifique la configuración del recurso compartido. Proporcione un nombre exclusivo para el recurso compartido. 

    ![Hacer clic en agregar recurso compartido](media/data-box-gateway-manage-shares/add-share-1.png)

    Los nombres de recursos compartidos solo pueden contener letras minúsculas, números y guiones. El nombre del recurso compartido debe tener entre 3 y 63 caracteres y empezar por una letra o un número. Antes y después de cada guion debe ir un carácter que no sea otro guión.

3. Seleccione un **tipo** de recurso compartido. El tipo puede ser **SMB** o **NFS** (SMB es el predeterminado). SMB es el estándar para los clientes de Windows y se usa NFS para los clientes de Linux. Dependiendo de si elige recursos compartidos SMB o NFS, las opciones que se presentan son ligeramente diferentes.

4. Especifique la **cuenta de almacenamiento** en que se encuentra el recurso compartido. Se crea un contenedor en la cuenta de almacenamiento con el nombre del recurso compartido si el contenedor no existía previamente. Si el contenedor ya existe, se usará este.  

5. Elija el **servicio de almacenamiento** entre blob en bloques, blobs en páginas o archivos. El tipo de servicio elegido depende de en qué formato desea que los datos residan en Azure. Por ejemplo, en este caso, queremos que los datos residan como **blobs en bloques** en Azure y, por tanto, seleccionamos esa opción. Si elige **Blob en páginas**, debe asegurarse de que los datos tienen una alineación de 512 bytes. Por ejemplo, un VHDX siempre tiene una alineación de 512 bytes.

   > [!IMPORTANT]
   > Asegúrese de que la cuenta de Azure Storage que usa no tiene directivas de inmutabilidad establecidas si va a usarla con un dispositivo Azure Stack Edge Pro o Data Box Gateway. Para más información, consulte [Establecimiento y administración de directivas de inmutabilidad para el almacenamiento de blobs](https://docs.microsoft.com/azure/storage/blobs/storage-blob-immutability-policies-manage).

6. Este paso depende de si está creando un recurso compartido SMB o NFS.
    - **Si crea un recurso compartido SMB**: en el campo **Usuario local con todos los privilegios**, elija entre **Crear nuevo** o **Usar existente**. Si va a crear un nuevo usuario local, indique el **nombre de usuario** y la **contraseña** y, después, confirme la contraseña. Esto asigna los permisos al usuario local. Después de haber asignado los permisos aquí, puede utilizar el Explorador de archivos para modificarlos.

        ![Incorporación de recurso compartido de SMB](media/data-box-gateway-manage-shares/add-share-2.png)

        Si selecciona que se permitan operaciones de solo lectura en los datos de este recurso compartido, puede especificar usuarios de solo lectura.
    - **Si crea un recurso compartido NFS**: debe indicar las **direcciones IP de los clientes autorizados** que pueden acceder al recurso compartido.

        ![Incorporación de un recurso compartido NFS](media/data-box-gateway-manage-shares/add-share-3.png)

7. Haga clic en **Crear** para crear el recurso compartido. Se le notifica que la creación del recurso compartido está en curso. Una vez creado el recurso compartido con la configuración especificada, la hoja **Recursos compartidos** se actualiza para reflejar el nuevo recurso compartido.
 
## <a name="delete-a-share"></a>Eliminación de un recurso compartido

Siga estos pasos en Azure Portal para eliminar un recurso compartido.

1. En la lista de recursos compartidos, seleccione y haga clic en el recurso compartido que desea eliminar.

    ![Seleccionar recurso compartido](media/data-box-gateway-manage-shares/delete-1.png)

2. Haga clic en **Eliminar**. 

    ![Clic en Eliminar](media/data-box-gateway-manage-shares/delete-2.png)

3. Cuando se le pida confirmación, haga clic en **Sí**.

    ![Confirmar eliminación](media/data-box-gateway-manage-shares/delete-3.png)

La lista de recursos compartidos se actualiza para reflejar la eliminación.


## <a name="refresh-shares"></a>Actualización de recursos compartidos

La característica de actualización permite actualizar el contenido de los recursos compartidos locales. Al actualizar un recurso compartido, se inicia una búsqueda para encontrar todos los objetos de Azure, lo que incluye los blobs y archivos que se agregaron a la nube desde la última actualización. Luego, estos archivos adicionales se usan para actualizar el contenido del recurso compartido de forma local en el dispositivo. 

> [!NOTE]
> En una operación de actualización, no se conservan los permisos y las listas de control de acceso (ACL). 

Siga estos pasos en Azure Portal para actualizar un recurso compartido.

1. En Azure Portal, vaya a **Recursos compartidos**. Seleccione y haga clic en el recurso compartido que desea actualizar.

   ![Seleccionar recurso compartido](media/data-box-gateway-manage-shares/refresh-1.png)

2. Haga clic en **Actualizar**. 

   ![Hacer clic en actualizar](media/data-box-gateway-manage-shares/refresh-2.png)
 
3. Cuando se le pida confirmación, haga clic en **Sí**. Se inicia el trabajo de actualización del contenido del recurso compartido local. 

   ![Confirmar actualización](media/data-box-gateway-manage-shares/refresh-3.png)
 
4.   Mientras la actualización está en curso, la opción de actualización está deshabilitada en el menú contextual. Haga clic en la notificación para ver el estado del trabajo de actualización.

5. El tiempo que tarde en completarse la actualización dependerá del número de archivos que haya en el contenedor de Azure, así como de los archivos del dispositivo. Cuando la actualización se haya completado correctamente, se actualizará la marca de tiempo del recurso compartido. Aunque la actualización tenga errores parciales, la operación se considerará correcta y se actualizará la marca de tiempo. 

   ![Marca de tiempo actualizada](media/data-box-gateway-manage-shares/refresh-4.png)
 
Si se produce un error, se genera una alerta, en la que se indica la causa del error y se proporciona una recomendación para solucionarlo. La alerta también incluye vínculos a un archivo que contiene el resumen completo de los errores, incluidos los archivos que no se pudieron actualizar o eliminar.

>[!IMPORTANT]
> En esta versión, no actualice varios recursos compartidos al mismo tiempo.

## <a name="sync-storage-keys"></a>Sincronización de claves de almacenamiento

Si las claves de la cuenta de almacenamiento han rotado, deberá sincronizar las claves del acceso al almacenamiento. La sincronización ayuda al dispositivo a obtener las claves de almacenamiento más recientes de su cuenta de almacenamiento.

Realice los pasos siguientes en Azure Portal para sincronizar la clave de acceso al almacenamiento.

1. Vaya a **Información general** en el recurso. 
2. En la lista de recursos compartidos, elija y haga clic en un recurso compartido asociado con la cuenta de almacenamiento que necesita sincronizar. Haga clic en **Sincronizar clave de almacenamiento**. 

     ![Sincronizar clave de almacenamiento 1](media/data-box-gateway-manage-shares/sync-storage-key-1.png)

3. Haga clic en **Sí** cuando se pida confirmación. Salga del cuadro de diálogo cuando haya finalizado la sincronización.

     ![Sincronizar clave de almacenamiento 1](media/data-box-gateway-manage-shares/sync-storage-key-2.png)

>[!NOTE]
> Esta operación solo debe realizarse una vez en cada cuenta de almacenamiento. No es necesario repetir la acción para todos los recursos compartidos asociados con la misma cuenta de almacenamiento.


## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [administrar usuarios desde Azure Portal](data-box-gateway-manage-users.md).
