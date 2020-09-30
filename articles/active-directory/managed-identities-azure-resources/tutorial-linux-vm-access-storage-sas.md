---
title: 'Tutorial: Acceso a Azure Storage mediante credenciales de SAS (Linux): Azure AD'
description: En este tutorial, se explica cómo se utilizan las identidades administradas asignadas por el sistema de una máquina virtual Linux para acceder a Azure Storage utilizando las credenciales de SAS en lugar de una clave de acceso de la cuenta de almacenamiento.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: daveba
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/20/2017
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3b0bb70b82e8c34c50743bf56069488e2d4c4e39
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90968777"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-identity-to-access-azure-storage-via-a-sas-credential"></a>Tutorial: Uso de una identidad asignada por el sistema de una máquina virtual Linux para acceder a Azure Storage con las credenciales de SAS

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial, se explica cómo usar las identidades administradas asignadas por el sistema de una máquina virtual Linux para obtener las credenciales de Firma de acceso compartido (SAS) del almacenamiento. En concreto, una [credencial SAS de servicio](../../storage/common/storage-sas-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#types-of-shared-access-signatures). 

> [!NOTE]
> La clave SAS generada en este tutorial no se restringirá ni limitará a la máquina virtual.  

Una credencial SAS de servicio permite conceder acceso limitado a los objetos de una cuenta de almacenamiento, durante un período de tiempo limitado y con un servicio específico (en nuestro caso, el servicio blob), sin exponer una clave de acceso a la cuenta. Puede usar una credencial SAS de la forma habitual al realizar operaciones de almacenamiento, por ejemplo, al usar el SDK de Storage. En este tutorial, mostramos cómo cargar y descargar un blob mediante la CLI de Azure Storage. Aprenderá a:


> [!div class="checklist"]
> * Crear una cuenta de almacenamiento
> * Creación de un contenedor de blobs en la cuenta de almacenamiento
> * Conceder acceso a la máquina virtual a una SAS de cuenta de almacenamiento en Resource Manager 
> * Obtener un token de acceso mediante la identidad de la máquina virtual y utilizarlo para recuperar la credencial SAS desde Resource Manager 

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento 

Si aún no tiene una, creará ahora una cuenta de almacenamiento.  También puede omitir este paso y conceder a la identidad administrada asignada por el sistema de la máquina virtual acceso a las claves de una cuenta de almacenamiento existente. 

1. Haga clic en el botón **+/Crear nuevo servicio** de la esquina superior izquierda de Azure Portal.
2. Haga clic en **Storage**, a continuación, en **Cuenta de almacenamiento** y se mostrará un nuevo panel "Crear cuenta de almacenamiento".
3. Escriba un **Nombre** para la cuenta de almacenamiento, que utilizará más adelante.  
4. **Modelo de implementación** y **Clase de cuenta** debe establecerse en "Resource Manager" y "Uso General", respectivamente. 
5. Asegúrese de que **Suscripción** y **Grupo de recursos** coinciden con los que especificó cuando creó la máquina virtual en el paso anterior.
6. Haga clic en **Crear**.

    ![Creación de una nueva cuenta de almacenamiento](./media/msi-tutorial-linux-vm-access-storage/msi-storage-create.png)

## <a name="create-a-blob-container-in-the-storage-account"></a>Creación de un contenedor de blobs en la cuenta de almacenamiento

Más adelante se cargará y descargará un archivo a la nueva cuenta de almacenamiento. Dado que los archivos requieren almacenamiento de blobs, es necesario crear un contenedor de blobs en el que almacenar el archivo.

1. Vuelva a la cuenta de almacenamiento recién creada.
2. Haga clic en el vínculo **Contenedores** en el panel de la izquierda, en "Blob service".
3. Haga clic en **+ Contenedor** en la parte superior de la página y se abrirá un panel "Nuevo contenedor".
4. Asigne un nombre al contenedor, seleccione un nivel de acceso y, a continuación, haga clic en **Aceptar**. El nombre especificado se utilizará más adelante en el tutorial. 

    ![Creación de contenedores de almacenamiento](./media/msi-tutorial-linux-vm-access-storage/create-blob-container.png)

## <a name="grant-your-vms-system-assigned-managed-identity-access-to-use-a-storage-sas"></a>Concesión de acceso a la identidad administrada asignada por el sistema de la máquina virtual para usar una SAS de almacenamiento 

Azure Storage no admite la autenticación de Azure AD de forma nativa.  No obstante, puede usar una identidad administrada asignada por el sistema de la máquina virtual para recuperar una SAS de almacenamiento de Resource Manager y usarla para acceder al almacenamiento.  En este paso, va a conceder a la identidad administrada asignada por el sistema de la máquina virtual acceso a la SAS de la cuenta de almacenamiento.   

1. Vuelva a la cuenta de almacenamiento recién creada.
2. Haga clic en el vínculo **Control de acceso (IAM)** en el panel izquierdo.  
3. Haga clic en **+ Agregar asignación de rol** en la parte superior de la página para agregar una asignación de roles nueva para la máquina virtual.
4. Establezca **Rol** en "Colaborador de la cuenta de almacenamiento", en el lado derecho de la página. 
5. En el menú desplegable siguiente, establezca **Asignar acceso a** en el recurso "Máquina virtual".  
6. A continuación, asegúrese de que la suscripción adecuada aparece en el menú desplegable **Suscripción** y después, establezca **Grupo de recursos** en "Todos los grupos de recursos".  
7. Por último, en **Seleccionar**, elija la máquina virtual Linux en el menú desplegable y haga clic en **Guardar**.  

    ![Texto alternativo de imagen](./media/msi-tutorial-linux-vm-access-storage/msi-storage-role-sas.png)

## <a name="get-an-access-token-using-the-vms-identity-and-use-it-to-call-azure-resource-manager"></a>Obtención de un token de acceso utilizando la identidad de la máquina virtual y su uso para llamar a Azure Resource Manager

En el resto del tutorial, vamos a trabajar desde la máquina virtual que se creó anteriormente.

Para completar estos pasos, necesitará un cliente SSH. Si usa Windows, puede usar el cliente SSH en el [Subsistema de Windows para Linux](/windows/wsl/install-win10). Si necesita ayuda para configurar las claves del cliente de SSH, consulte [Uso de SSH con Windows en Azure](../../virtual-machines/linux/ssh-from-windows.md) o [Creación y uso de un par de claves SSH pública y privada para máquinas virtuales Linux en Azure](../../virtual-machines/linux/mac-create-ssh-keys.md).

1. En Azure Portal, vaya a **Máquinas virtuales**, vaya a la máquina virtual Linux y, a continuación, desde la página **Información general**, haga clic en **Conectar** en la parte superior. Copie la cadena para conectarse a la máquina virtual. 
2. Conéctese a la máquina virtual mediante un cliente SSH.  
3. A continuación, se le pedirá que escriba la **contraseña** que agregó al crear la **máquina virtual Linux**. Debe haber iniciado sesión correctamente.  
4. Utilice CURL para obtener un token de acceso para Azure Resource Manager.  

    La solicitud CURL y la respuesta para el token de acceso están a continuación:
    
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true    
    ```
    
    > [!NOTE]
    > En la solicitud anterior, el valor del parámetro "resource" tiene que coincidir exactamente con el que Azure AD espera. Al usar el id. de recurso de Azure Resource Manager, debe incluir la barra diagonal final en el URI.
    > En la siguiente respuesta, el elemento access_token se ha acortado para abreviar.
    
    ```bash
    {"access_token":"eyJ0eXAiOiJ...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://management.azure.com",
    "token_type":"Bearer"} 
     ```

## <a name="get-a-sas-credential-from-azure-resource-manager-to-make-storage-calls"></a>Obtención de una credencial SAS desde Azure Resource Manager para realizar llamadas de almacenamiento

Ahora vamos a utilizar CURL para realizar una llamada a Resource Manager mediante el token de acceso que se recuperó en la sección anterior, a fin de crear una credencial SAS de almacenamiento. Una vez que tenemos la credencial SAS de almacenamiento, podemos hacer llamadas a operaciones de carga y descarga de almacenamiento.

Para esta solicitud, vamos a usar los parámetros de solicitud HTTP de seguimiento para crear la credencial SAS:

```JSON
{
    "canonicalizedResource":"/blob/<STORAGE ACCOUNT NAME>/<CONTAINER NAME>",
    "signedResource":"c",              // The kind of resource accessible with the SAS, in this case a container (c).
    "signedPermission":"rcw",          // Permissions for this SAS, in this case (r)ead, (c)reate, and (w)rite.  Order is important.
    "signedProtocol":"https",          // Require the SAS be used on https protocol.
    "signedExpiry":"<EXPIRATION TIME>" // UTC expiration time for SAS in ISO 8601 format, for example 2017-09-22T00:06:00Z.
}
```

Estos parámetros se incluyen en el cuerpo POST de la solicitud para la credencial SAS. Para más información sobre los parámetros para crear una credencial SAS, consulte la [referencia de REST de SAS del servicio de lista](/rest/api/storagerp/storageaccounts/listservicesas).

Use la siguiente solicitud CURL para obtener la credencial SAS. Asegúrese de reemplazar los valores de los parámetros `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>`, `<STORAGE ACCOUNT NAME>`, `<CONTAINER NAME>` y `<EXPIRATION TIME>` por sus propios valores. Reemplace el valor de `<ACCESS TOKEN>` con el token de acceso que se recuperó anteriormente:

```bash 
curl https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE ACCOUNT NAME>/listServiceSas/?api-version=2017-06-01 -X POST -d "{\"canonicalizedResource\":\"/blob/<STORAGE ACCOUNT NAME>/<CONTAINER NAME>\",\"signedResource\":\"c\",\"signedPermission\":\"rcw\",\"signedProtocol\":\"https\",\"signedExpiry\":\"<EXPIRATION TIME>\"}" -H "Authorization: Bearer <ACCESS TOKEN>"
```

> [!NOTE]
> El texto de la dirección URL anterior distingue mayúsculas de minúsculas, por lo que tiene que fijarse en si usa mayúsculas o minúsculas para los grupos de recursos para reflejarlo adecuadamente según corresponda. Además, es importante saber que se trata de una solicitud POST, no es una solicitud GET.

La respuesta CURL devuelve la credencial SAS:  

```bash 
{"serviceSasToken":"sv=2015-04-05&sr=c&spr=https&st=2017-09-22T00%3A10%3A00Z&se=2017-09-22T02%3A00%3A00Z&sp=rcw&sig=QcVwljccgWcNMbe9roAJbD8J5oEkYoq%2F0cUPlgriBn0%3D"} 
```

Cree un archivo blob de ejemplo para cargar en el contenedor de almacenamiento de blobs. En una máquina virtual Linux puede hacer esto con el siguiente comando. 

```bash
echo "This is a test file." > test.txt
```

A continuación, autentíquese con el comando `az storage` de la CLI con la credencial SAS y cargue el archivo en el contenedor de blobs. Para este paso, necesitará [instalar la CLI de Azure más reciente](/cli/azure/install-azure-cli) en su máquina virtual, si no lo ha hecho ya.

```azurecli
 az storage blob upload --container-name 
                        --file 
                        --name
                        --account-name 
                        --sas-token
```

Respuesta: 

```JSON
Finished[#############################################################]  100.0000%
{
  "etag": "\"0x8D4F9929765C139\"",
  "lastModified": "2017-09-21T03:58:56+00:00"
}
```

Además, puede descargar el archivo usando la CLI de Azure y autenticándose con la credencial SAS. 

Solicitud: 

```azurecli
az storage blob download --container-name
                         --file 
                         --name 
                         --account-name
                         --sas-token
```

Respuesta: 

```JSON
{
  "content": null,
  "metadata": {},
  "name": "testblob",
  "properties": {
    "appendBlobCommittedBlockCount": null,
    "blobType": "BlockBlob",
    "contentLength": 16,
    "contentRange": "bytes 0-15/16",
    "contentSettings": {
      "cacheControl": null,
      "contentDisposition": null,
      "contentEncoding": null,
      "contentLanguage": null,
      "contentMd5": "Aryr///Rb+D8JQ8IytleDA==",
      "contentType": "text/plain"
    },
    "copy": {
      "completionTime": null,
      "id": null,
      "progress": null,
      "source": null,
      "status": null,
      "statusDescription": null
    },
    "etag": "\"0x8D4F9929765C139\"",
    "lastModified": "2017-09-21T03:58:56+00:00",
    "lease": {
      "duration": null,
      "state": "available",
      "status": "unlocked"
    },
    "pageBlobSequenceNumber": null,
    "serverEncrypted": false
  },
  "snapshot": null
}
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Storage utilizando las credenciales de SAS.  Para obtener más información sobre SAS de Azure Storage, vea:

> [!div class="nextstepaction"]
>[Uso de Firmas de acceso compartido (SAS)](../../storage/common/storage-sas-overview.md)
