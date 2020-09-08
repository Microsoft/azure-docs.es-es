---
title: 'Tutorial`:` Uso de identidades administradas para acceder a Azure Storage (Linux): Azure AD'
description: Este tutorial contiene directrices acerca de cómo utilizar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Storage.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/14/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9b61d3ed21d053fc7166b47c94a9ec61e355d199
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89263168"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-storage"></a>Tutorial: Uso de identidades administradas asignadas por el sistema de una máquina virtual Linux para acceder a Azure Storage 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial, se explica cómo se utiliza una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Storage. Aprenderá a:

> [!div class="checklist"]
> * Crear una cuenta de almacenamiento
> * Crear un contenedor de blobs en una cuenta de almacenamiento
> * Concesión de acceso a una identidad administrada de una máquina virtual Linux para un contenedor de Azure Storage
> * Obtención de un token de acceso y su uso para llamar a Azure Storage

> [!NOTE]
> La autenticación de Azure Active Directory para Azure Storage está en versión preliminar pública.

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

Para ejecutar los ejemplos de script de la CLI de este tutorial, tiene dos opciones:

- Use [Azure Cloud Shell](~/articles/cloud-shell/overview.md) desde Azure Portal o mediante el botón **Pruébelo**, situado en la esquina superior derecha de cada bloque de código.
- [Instale la versión más reciente de la CLI 2.0](/cli/azure/install-azure-cli) (2.0.23 o posterior), si prefiere usar una consola de la CLI local.

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento 

En esta sección se creará una cuenta de almacenamiento. 

1. Haga clic en el botón **+Crear un recurso** de la esquina superior izquierda de Azure Portal.
2. Haga clic en **Storage** (Almacenamiento) y, luego, en **Storage account - blob, file, table, queue** (Cuenta de almacenamiento: blob, archivo, tabla, cola).
3. En **Name** (Nombre), escriba un nombre para la cuenta de almacenamiento.  
4. **Deployment model** (Modelo de implementación) y **Account kind** (Clase de cuenta) se deben establecer en **Resource Manager** y **Storage (general purpose v1)** (Almacenamiento [de uso general v1]). 
5. Asegúrese de que **Suscripción** y **Grupo de recursos** coinciden con los que especificó cuando creó la máquina virtual en el paso anterior.
6. Haga clic en **Crear**.

    ![Creación de una nueva cuenta de almacenamiento](./media/msi-tutorial-linux-vm-access-storage/msi-storage-create.png)

## <a name="create-a-blob-container-and-upload-a-file-to-the-storage-account"></a>Creación de un contenedor de blobs y carga de un archivo a la cuenta de almacenamiento

Los archivos requieren almacenamiento de blobs, por lo que es necesario crear un contenedor de blobs donde se almacenará el archivo. Luego cargará un archivo en el contenedor de blobs de la cuenta de almacenamiento nueva.

1. Vuelva a la cuenta de almacenamiento recién creada.
2. En **Blob service**, haga clic en **Contenedores**.
3. Haga clic en **+ Contenedor** en la parte superior de la página.
4. En **Nuevo contenedor**, escriba un nombre para el contenedor, y mantenga el valor predeterminado en **Nivel de acceso público**.

    ![Creación de contenedores de almacenamiento](./media/msi-tutorial-linux-vm-access-storage/create-blob-container.png)

5. Elija un editor de su preferencia y cree un archivo denominado *hello world.txt* en la máquina local.  Abra el archivo y agregue el texto (sin comillas) "Hola mundo :)" y guárdelo. 

6. Para cargar el archivo en el contenedor recién creado, haga clic en el nombre del contenedor y en **Cargar**.
7. En el panel **Carga de blob**, en **Archivos**, haga clic en el icono de carpeta y vaya al archivo **hello_world.txt** en la máquina virtual, selecciónelo y haga clic en **Cargar**.

    ![Carga de un archivo de texto](./media/msi-tutorial-linux-vm-access-storage/upload-text-file.png)

## <a name="grant-your-vm-access-to-an-azure-storage-container"></a>Concesión de acceso a un contenedor de Azure Storage para la máquina virtual 

Puede usar la identidad Managed Identity de la máquina virtual para recuperar los datos en Azure Storage Blob.   

1. Vuelva a la cuenta de almacenamiento recién creada.  
2. Haga clic en el vínculo **Control de acceso (IAM)** en el panel izquierdo.  
3. Haga clic en **+ Agregar asignación de rol** en la parte superior de la página para agregar una asignación de roles nueva para la máquina virtual.
4. En la lista desplegable de **Rol**, seleccione **Lector de datos de blobs de almacenamiento**. 
5. En la lista desplegable siguiente, en **Asignar acceso a**, elija **Máquina virtual**.  
6. A continuación, asegúrese de que la suscripción adecuada aparece en el menú desplegable **Suscripción** y establezca **Grupo de recursos** en **Todos los grupos de recursos**.  
7. En **Seleccionar**, elija la máquina virtual y haga clic en **Guardar**.

    ![Asignación de permisos](./media/tutorial-linux-vm-access-storage/access-storage-perms.png)

## <a name="get-an-access-token-and-use-it-to-call-azure-storage"></a>Obtención de un token de acceso y su uso para llamar a Azure Storage

Azure Storage admite de manera nativa la autenticación de Azure AD, por lo que puede aceptar directamente los tokens de acceso obtenidos mediante una identidad Managed Identity. Forma parte de la integración de Azure Storage con Azure AD y es diferente de proporcionar las credenciales en la cadena de conexión.

Para completar los pasos siguientes, deberá trabajar desde la máquina virtual que creó anteriormente y necesitará un cliente SSH para conectarse a ella. Si usa Windows, puede usar el cliente SSH en el [Subsistema de Windows para Linux](/windows/wsl/about). Si necesita ayuda para configurar las claves del cliente de SSH, consulte [Uso de SSH con Windows en Azure](~/articles/virtual-machines/linux/ssh-from-windows.md) o [Creación y uso de un par de claves SSH pública y privada para máquinas virtuales Linux en Azure](~/articles/virtual-machines/linux/mac-create-ssh-keys.md).

1. En Azure Portal, vaya a **Máquinas virtuales**, vaya a la máquina virtual Linux y, a continuación, desde la página **Información general**, haga clic en **Conectar**. Copie la cadena para conectarse a la máquina virtual.
2. **Conéctese** a la máquina virtual con el cliente SSH que elija. 
3. En la ventana del terminal, con CURL, realice una solicitud al punto de conexión local de la identidad administrada local para obtener un token de acceso para Azure Storage.
    
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fstorage.azure.com%2F' -H Metadata:true
    ```
4. Ahora utilice el token de acceso para acceder a Azure Storage, por ejemplo para leer el contenido del archivo de muestra que ha cargado previamente en el contenedor. Reemplace los valores `<STORAGE ACCOUNT>`, `<CONTAINER NAME>` y `<FILE NAME>` por los valores especificados anteriormente, y `<ACCESS TOKEN>` por el token devuelto en el paso anterior.

   ```bash
   curl https://<STORAGE ACCOUNT>.blob.core.windows.net/<CONTAINER NAME>/<FILE NAME> -H "x-ms-version: 2017-11-09" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   La respuesta incluye el contenido del archivo:

   ```bash
   Hello world! :)
   ```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a habilitar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Storage.  Para más información sobre Azure Storage, consulte:

> [!div class="nextstepaction"]
> [Almacenamiento de Azure](../../storage/common/storage-introduction.md)