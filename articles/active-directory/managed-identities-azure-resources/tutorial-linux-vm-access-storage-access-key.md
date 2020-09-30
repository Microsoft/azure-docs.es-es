---
title: 'Tutorial`:` Uso de identidades administradas para acceder a Azure Storage mediante una clave de acceso (Linux): Azure AD'
description: Este tutorial contiene directrices acerca de cómo utilizar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Storage.
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
ms.date: 03/04/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 312ca5f693c42d70ecb5e210ef0d30df5c9739bb
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90968700"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-storage-via-access-key"></a>Tutorial: Uso de identidades administradas asignadas por el sistema de una máquina virtual Linux para acceder a Azure Storage utilizando una clave de acceso

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial, se muestra cómo usar una identidad administrada asignada por el sistema de una máquina virtual Linux para recuperar las claves de acceso de la cuenta de almacenamiento. Puede usar una clave de acceso de almacenamiento de la forma habitual al realizar operaciones de almacenamiento, por ejemplo, al usar el SDK de Storage. En este tutorial se cargan y descargan blobs mediante la CLI de Azure. Aprenderá a:

> [!div class="checklist"]
> * Conceder acceso a la máquina virtual a las claves de acceso a la cuenta de almacenamiento en Resource Manager 
> * Obtener un token de acceso mediante la identidad de la máquina virtual y utilizarlo para recuperar las claves de acceso a la cuenta de almacenamiento desde Resource Manager  

## <a name="prerequisites"></a>Prerequisites

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
2. Haga clic en el vínculo **Contenedores** a la izquierda, en "Blob service".
3. Haga clic en **+ Contenedor** en la parte superior de la página y se abrirá un panel "Nuevo contenedor".
4. Asigne un nombre al contenedor, seleccione un nivel de acceso y, a continuación, haga clic en **Aceptar**. El nombre especificado se utilizará más adelante en el tutorial. 

    ![Creación de contenedores de almacenamiento](./media/msi-tutorial-linux-vm-access-storage/create-blob-container.png)

## <a name="grant-your-vms-system-assigned-managed-identity-access-to-use-storage-account-access-keys"></a>Concesión de acceso a la identidad administrada asignada por el sistema de la máquina virtual para usar las claves de acceso de la cuenta de almacenamiento

En este paso, va a conceder a la identidad administrada asignada por el sistema de la máquina virtual acceso a las claves de la cuenta de almacenamiento.   

1. Vuelva a la cuenta de almacenamiento recién creada.
2. Haga clic en el vínculo **Control de acceso (IAM)** en el panel izquierdo.  
3. Haga clic en **+ Agregar asignación de rol** en la parte superior de la página para agregar una asignación de roles nueva para la máquina virtual.
4. Establezca **Rol** en "Rol de servicio del operador de claves de cuentas de almacenamiento", en el lado derecho de la página. 
5. En el menú desplegable siguiente, establezca **Asignar acceso a** en el recurso "Máquina virtual".  
6. A continuación, asegúrese de que la suscripción adecuada aparece en el menú desplegable **Suscripción** y después, establezca **Grupo de recursos** en "Todos los grupos de recursos".  
7. Por último, en **Seleccionar**, elija la máquina virtual Linux en el menú desplegable y haga clic en **Guardar**. 

    ![Texto alternativo de imagen](./media/msi-tutorial-linux-vm-access-storage/msi-storage-role.png)

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
    > En la siguiente respuesta, el elemento access_token se ha abreviado.
    
    ```bash
    {"access_token":"eyJ0eXAiOiJ...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://management.azure.com",
    "token_type":"Bearer"} 
     ```
    
## <a name="get-storage-account-access-keys-from-azure-resource-manager-to-make-storage-calls"></a>Obtención de las claves de almacenamiento desde Azure Resource Manager para realizar llamadas al almacenamiento  

Ahora utilice CURL para llamar a Resource Manager mediante el token de acceso que se recuperó en la sección anterior, para recuperar la clave de acceso de almacenamiento. Una vez que tenemos la clave de acceso de almacenamiento, podemos hacer llamadas a operaciones de carga y descarga del almacenamiento. Asegúrese de reemplazar los valores de los parámetros `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>` y `<STORAGE ACCOUNT NAME>` con sus propios valores. Reemplace el valor de `<ACCESS TOKEN>` con el token de acceso que se recuperó anteriormente:

```bash 
curl https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE ACCOUNT NAME>/listKeys?api-version=2016-12-01 --request POST -d "" -H "Authorization: Bearer <ACCESS TOKEN>" 
```

> [!NOTE]
> El texto de la dirección URL anterior distingue mayúsculas de minúsculas, por lo que tiene que fijarse en si usa mayúsculas o minúsculas para los grupos de recursos para reflejarlo adecuadamente según corresponda. Además, es importante saber que se trata de una solicitud POST, no una solicitud GET, y asegurarse de que pasa un valor para capturar un límite de longitud con -d que puede ser NULL.  

La respuesta CURL le ofrece la lista de claves:  

```bash 
{"keys":[{"keyName":"key1","permissions":"Full","value":"iqDPNt..."},{"keyName":"key2","permissions":"Full","value":"U+uI0B..."}]} 
```
Cree un archivo blob de ejemplo para cargar en el contenedor de almacenamiento de blobs. En una máquina virtual Linux puede hacer esto con el siguiente comando. 

```bash
echo "This is a test file." > test.txt
```

A continuación, autentíquese con el comando `az storage` de la CLI con la clave de acceso al almacenamiento y cargue el archivo en el contenedor de blobs. Para este paso, necesitará [instalar la CLI de Azure más reciente](/cli/azure/install-azure-cli) en su máquina virtual, si no lo ha hecho ya.
 

```azurecli
az storage blob upload -c <CONTAINER NAME> -n test.txt -f test.txt --account-name <STORAGE ACCOUNT NAME> --account-key <STORAGE ACCOUNT KEY>
```

Respuesta: 

```JSON
Finished[#############################################################]  100.0000%
{
  "etag": "\"0x8D4F9929765C139\"",
  "lastModified": "2017-09-12T03:58:56+00:00"
}
```

Además, puede cargar el archivo usando la CLI de Azure y autenticando con la clave de acceso al almacenamiento. 

Solicitud: 

```azurecli
az storage blob download -c <CONTAINER NAME> -n test.txt -f test-download.txt --account-name <STORAGE ACCOUNT NAME> --account-key <STORAGE ACCOUNT KEY>
```

Respuesta: 

```JSON
{
  "content": null,
  "metadata": {},
  "name": "test.txt",
  "properties": {
    "appendBlobCommittedBlockCount": null,
    "blobType": "BlockBlob",
    "contentLength": 21,
    "contentRange": "bytes 0-20/21",
    "contentSettings": {
      "cacheControl": null,
      "contentDisposition": null,
      "contentEncoding": null,
      "contentLanguage": null,
      "contentMd5": "LSghAvpnElYyfUdn7CO8aw==",
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
    "etag": "\"0x8D5067F30D0C283\"",
    "lastModified": "2017-09-28T14:42:49+00:00",
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

En este tutorial, ha aprendido a utilizar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Storage utilizando una clave de acceso.  Para obtener más información sobre las claves de acceso de Azure Storage, vea:

> [!div class="nextstepaction"]
>[Administración de las claves de acceso de almacenamiento](../../storage/common/storage-account-create.md)
