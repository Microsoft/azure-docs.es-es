---
title: Acceso a Azure Storage mediante una identidad administrada asignada por el sistema de una máquina virtual Windows | Microsoft Docs
description: Este tutorial contiene directrices acerca de cómo utilizar una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure Storage.
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
ms.date: 01/14/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: de1cc69b3cfdac307edf6dfe999a5d538c2cb811
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89263185"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-storage"></a>Tutorial: Uso de una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure Storage

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial se muestra cómo usar una identidad administrada asignada por el sistema en una máquina virtual Windows para acceder a Azure Storage. Aprenderá a:

> [!div class="checklist"]
> * Crear un contenedor de blobs en una cuenta de almacenamiento
> * Conceder a la identidad administrada asignada por el sistema de la máquina virtual Windows acceso a una cuenta de almacenamiento
> * Obtener un acceso y usarlo para llamar a Azure Storage

> [!NOTE]
> La autenticación de Azure Active Directory para Azure Storage está en versión preliminar pública.

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]



## <a name="enable"></a>Habilitar

[!INCLUDE [msi-tut-enable](../../../includes/active-directory-msi-tut-enable.md)]



## <a name="grant-access"></a>Conceder acceso


### <a name="create-storage-account"></a>Crear cuenta de almacenamiento

En esta sección se creará una cuenta de almacenamiento.

1. Haga clic en el botón **+Crear un recurso** de la esquina superior izquierda de Azure Portal.
2. Haga clic en **Storage** (Almacenamiento) y, luego, en **Storage account - blob, file, table, queue** (Cuenta de almacenamiento: blob, archivo, tabla, cola).
3. En **Name** (Nombre), escriba un nombre para la cuenta de almacenamiento.
4. **Deployment model** (Modelo de implementación) y **Account kind** (Clase de cuenta) se deben establecer en **Resource Manager** y **Storage (general purpose v1)** (Almacenamiento [de uso general v1]).
5. Asegúrese de que **Suscripción** y **Grupo de recursos** coinciden con los que especificó cuando creó la máquina virtual en el paso anterior.
6. Haga clic en **Crear**.

    ![Creación de una nueva cuenta de almacenamiento](./media/msi-tutorial-linux-vm-access-storage/msi-storage-create.png)

### <a name="create-a-blob-container-and-upload-a-file-to-the-storage-account"></a>Creación de un contenedor de blobs y carga de un archivo a la cuenta de almacenamiento

Los archivos requieren almacenamiento de blobs, por lo que es necesario crear un contenedor de blobs donde se almacenará el archivo. Luego cargará un archivo en el contenedor de blobs de la cuenta de almacenamiento nueva.

1. Vuelva a la cuenta de almacenamiento recién creada.
2. En **Blob service**, haga clic en **Contenedores**.
3. Haga clic en **+ Contenedor** en la parte superior de la página.
4. En **Nuevo contenedor**, escriba un nombre para el contenedor, y mantenga el valor predeterminado en **Nivel de acceso público**.

    ![Creación de contenedores de almacenamiento](./media/msi-tutorial-linux-vm-access-storage/create-blob-container.png)

5. Elija un editor de su preferencia y cree un archivo denominado *hello world.txt* en la máquina local. Abra el archivo y agregue el texto (sin comillas) "Hola mundo :)" y guárdelo.
6. Para cargar el archivo en el contenedor recién creado, haga clic en el nombre del contenedor y en **Cargar**.
7. En el panel **Carga de blob**, en **Archivos**, haga clic en el icono de carpeta y vaya al archivo **hello_world.txt** en la máquina virtual, selecciónelo y haga clic en **Cargar**.
    ![Carga de un archivo de texto](./media/msi-tutorial-linux-vm-access-storage/upload-text-file.png)

### <a name="grant-access"></a>Conceder acceso

En esta sección se muestra cómo conceder acceso a un contenedor de Azure Storage para la máquina virtual. Puede usar la identidad administrada asignada por el sistema de la máquina virtual para recuperar los datos en Azure Storage Blob.

1. Vuelva a la cuenta de almacenamiento recién creada.
2. Haga clic en el vínculo **Control de acceso (IAM)** en el panel izquierdo.
3. Haga clic en **+ Agregar asignación de rol** en la parte superior de la página para agregar una asignación de roles nueva para la máquina virtual.
4. En la lista desplegable de **Rol**, seleccione **Lector de datos de blobs de almacenamiento**.
5. En la lista desplegable siguiente, en **Asignar acceso a**, elija **Máquina virtual**.
6. A continuación, asegúrese de que la suscripción adecuada aparece en el menú desplegable **Suscripción** y establezca **Grupo de recursos** en **Todos los grupos de recursos**.
7. En **Seleccionar**, elija la máquina virtual y haga clic en **Guardar**.

    ![Asignación de permisos](./media/tutorial-linux-vm-access-storage/access-storage-perms.png)

## <a name="access-data"></a>Acceso a los datos 

Azure Storage admite de manera nativa la autenticación de Azure AD, por lo que puede aceptar directamente los tokens de acceso obtenidos mediante una identidad administrada. Forma parte de la integración de Azure Storage con Azure AD y es diferente de proporcionar las credenciales en la cadena de conexión.

Este es un ejemplo de código .NET para abrir una conexión a Azure Storage con un token de acceso y luego leer el contenido del archivo que creó anteriormente. Este código se debe ejecutar en la máquina virtual para poder acceder al punto de conexión de la identidad administrada de la máquina virtual. Se requiere .NET framework 4.6 o posterior para usar el método de token de acceso. Reemplace el valor de `<URI to blob file>` según corresponda. Para obtener este valor, vaya al archivo que creó y cargó en el almacenamiento de blobs y copie la **dirección URL** bajo **Propiedades** en la página **Información general**.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.IO;
using System.Net;
using System.Web.Script.Serialization;
using Microsoft.WindowsAzure.Storage.Auth;
using Microsoft.WindowsAzure.Storage.Blob;

namespace StorageOAuthToken
{
    class Program
    {
        static void Main(string[] args)
        {
            //get token
            string accessToken = GetMSIToken("https://storage.azure.com/");

            //create token credential
            TokenCredential tokenCredential = new TokenCredential(accessToken);

            //create storage credentials
            StorageCredentials storageCredentials = new StorageCredentials(tokenCredential);

            Uri blobAddress = new Uri("<URI to blob file>");

            //create block blob using storage credentials
            CloudBlockBlob blob = new CloudBlockBlob(blobAddress, storageCredentials);

            //retrieve blob contents
            Console.WriteLine(blob.DownloadText());
            Console.ReadLine();
        }

        static string GetMSIToken(string resourceID)
        {
            string accessToken = string.Empty;
            // Build request to acquire MSI token
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=" + resourceID);
            request.Headers["Metadata"] = "true";
            request.Method = "GET";

            try
            {
                // Call /token endpoint
                HttpWebResponse response = (HttpWebResponse)request.GetResponse();

                // Pipe response Stream to a StreamReader, and extract access token
                StreamReader streamResponse = new StreamReader(response.GetResponseStream());
                string stringResponse = streamResponse.ReadToEnd();
                JavaScriptSerializer j = new JavaScriptSerializer();
                Dictionary<string, string> list = (Dictionary<string, string>)j.Deserialize(stringResponse, typeof(Dictionary<string, string>));
                accessToken = list["access_token"];
                return accessToken;
            }
            catch (Exception e)
            {
                string errorText = String.Format("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
                return accessToken;
            }
        }
    }
}
```

La respuesta incluye el contenido del archivo:

`Hello world! :)`


## <a name="disable"></a>Disable

[!INCLUDE [msi-tut-disable](../../../includes/active-directory-msi-tut-disable.md)]



## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a habilitar una identidad asignada por el sistema de una máquina virtual Windows para acceder a Azure Storage.  Para más información sobre Azure Storage, consulte:

> [!div class="nextstepaction"]
> [Almacenamiento de Azure](../../storage/common/storage-introduction.md)