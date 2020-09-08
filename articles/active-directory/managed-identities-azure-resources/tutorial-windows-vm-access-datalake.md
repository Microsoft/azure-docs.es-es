---
title: 'Tutorial`:` Uso de identidades administradas para acceder a Azure Data Lake Store (Windows): Azure AD'
description: En este tutorial, se explica cómo se utiliza una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure Data Lake Store.
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
ms.date: 11/14/2018
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: d29edec6145ebc03218264532cae07b6afc9654c
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89254209"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-data-lake-store"></a>Tutorial: Uso de una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure Data Lake Store

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial se muestra cómo usar una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure Data Lake Store. Las identidades de MSI son administradas automáticamente por Azure y le permiten autenticar los servicios que admiten la autenticación de Azure AD sin necesidad de insertar credenciales en el código. Aprenderá a:

> [!div class="checklist"]
> * Conceder a una máquina virtual acceso a Azure Data Lake Store
> * Obtener un token de acceso mediante la identidad de máquina virtual y usarlo para acceder a Azure Data Lake Store

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]



## <a name="enable"></a>Habilitar

[!INCLUDE [msi-tut-enable](../../../includes/active-directory-msi-tut-enable.md)]



## <a name="grant-access"></a>Conceder acceso

Ahora puede conceder a una máquina virtual el acceso a los archivos y las carpetas de una instancia de Azure Data Lake Store.  En este paso, puede usar una instancia ya existente de Data Lake Store o crear una nueva.  Para crear una nueva instancia de Data Lake Store mediante Azure Portal, siga esta [guía de inicio rápido de Azure Data Lake Store](../../data-lake-store/data-lake-store-get-started-portal.md). También se pueden encontrar guías de inicio rápido que utilizan la CLI de Azure y Azure PowerShell en la [documentación de Azure Data Lake Store](../../data-lake-store/data-lake-store-overview.md).

En Data Lake Store, cree una nueva carpeta y conceda a la identidad asignada por el sistema de la máquina virtual permisos de lectura, escritura y ejecución de archivos en dicha carpeta:

1. En Azure Portal, haga clic en **Data Lake Store** en el panel de navegación izquierdo.
2. Haga clic en la instancia de Data Lake Store que quiere usar en este tutorial.
3. Haga clic en **Explorador de datos** en la barra de comandos.
4. Se selecciona la carpeta raíz de Data Lake Store.  Haga clic en **Acceso** en la barra de comandos.
5. Haga clic en **Agregar**.  En el campo **Seleccionar**, escriba el nombre de la máquina virtual, por ejemplo **DevTestVM**.  Haga clic para seleccionar la máquina virtual de los resultados de búsqueda y luego haga clic en **Seleccionar**.
6. Haga clic en **Seleccionar permisos**.  Seleccione **Lectura** y **Ejecutar**, agréguelos a **Esta carpeta** y agréguelos como **un solo permiso de acceso**.  Haga clic en **Aceptar**.  El permiso se debe haber agregado correctamente.
7. Cierre la hoja **Acceso**.
8. En este tutorial, cree una nueva carpeta.  Haga clic en **Nueva carpeta** en la barra de comandos y asígnele un nombre, por ejemplo **TestFolder**.  Haga clic en **Aceptar**.
9. Haga clic en la carpeta que ha creado y, luego, en **Acceso** en la barra de comandos.
10. Igual que en el paso 5, haga clic en **Agregar**, en el campo **Seleccionar** escriba el nombre de su máquina virtual, selecciónelo y haga clic en **Seleccionar**.
11. Igual que en el paso 6, haga clic en **Seleccionar permisos**, seleccione **Lectura**, **Escritura** y **Ejecutar**, agréguelos a **Esta carpeta** y agréguelos como **Una entrada de permiso de acceso y una entrada de permiso predeterminado**.  Haga clic en **Aceptar**.  El permiso se debe haber agregado correctamente.

La identidad administrada asignada por el sistema de la máquina virtual puede realizar ahora todas las operaciones en los archivos de la carpeta que ha creado.  Para más información sobre cómo administrar el acceso a Data Lake Store, lea este artículo sobre el [control de acceso en Data Lake Store](../../data-lake-store/data-lake-store-access-control.md).

## <a name="access-data"></a>Acceso a los datos

Azure Data Lake Store admite de forma nativa la autenticación de Azure AD, por lo que puede aceptar directamente los tokens de acceso obtenidos con la característica Managed Identities for Azure Resources.  Para autenticarse en el sistema de archivos de Data Lake Store, se envía un token de acceso emitido por Azure AD al punto de conexión del sistema de archivos de Data Lake Store, en un encabezado de autorización con el formato "Bearer <ACCESS_TOKEN_VALUE>".  Para más información sobre la compatibilidad de Data Lake Store con la autenticación de Azure AD, lea [Autenticación con Data Lake Store mediante Azure Active Directory](../../data-lake-store/data-lakes-store-authentication-using-azure-active-directory.md).

> [!NOTE]
> Los SDK de cliente del sistema de archivos de Data Lake Store todavía no admiten la característica Managed Identities for Azure Resources.  Este tutorial se actualizará cuando se agregue esta compatibilidad al SDK.

En este tutorial, se autenticará en la API de REST del sistema de archivos de Data Lake Store mediante PowerShell para realizar solicitudes de REST. Para usar la identidad administrada asignada por el sistema de la máquina virtual para la autenticación, es preciso realizar las solicitudes desde la máquina virtual.

1. En el portal, vaya a **Virtual Machines**, vaya a la máquina virtual Windows, y en **Información general**, haga clic en **Conectar**.
2. Escriba su **nombre de usuario** y **contraseña** que agregó cuando creó la máquina virtual Windows. 
3. Ahora que ha creado una **conexión a Escritorio remoto** con la máquina virtual, abra **PowerShell** en la sesión remota. 
4. Mediante `Invoke-WebRequest` de PowerShell, realice una solicitud a las identidades administradas locales del punto de conexión de recursos de Azure y obtenga un token de acceso para Azure Data Lake Store.  El identificador de recurso de Data Lake Store es `https://datalake.azure.net/`.  Data Lake busca una coincidencia exacta en el identificador del recurso y la barra diagonal final es importante.

   ```powershell
   $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatalake.azure.net%2F' -Method GET -Headers @{Metadata="true"}
   ```
    
   Convierta la respuesta de un objeto JSON a un objeto de PowerShell. 
    
   ```powershell
   $content = $response.Content | ConvertFrom-Json
   ```

   Extraiga el token de acceso de la respuesta.
    
   ```powershell
   $AccessToken = $content.access_token
   ```

5. Con el comando "Invoke-WebRequest" de PowerShell, realice una solicitud a su punto de conexión de REST de Data Lake Store para mostrar las carpetas que contiene la carpeta raíz.  Se trata de una manera sencilla de comprobar que todo está configurado correctamente.  Es importante que la cadena "Bearer" en el encabezado de autorización tenga la "B" en mayúscula.  Puede encontrar el nombre de su instancia de Data Lake Store en la sección **Información general** de la hoja de Data Lake Store de Azure Portal.

   ```powershell
   Invoke-WebRequest -Uri https://<YOUR_ADLS_NAME>.azuredatalakestore.net/webhdfs/v1/?op=LISTSTATUS -Headers @{Authorization="Bearer $AccessToken"}
   ```

   Una respuesta correcta tiene el siguiente aspecto:

   ```powershell
   StatusCode        : 200
   StatusDescription : OK
   Content           : {"FileStatuses":{"FileStatus":[{"length":0,"pathSuffix":"TestFolder","type":"DIRECTORY", "blockSize":0,"accessTime":1507934941392, "modificationTime":1507944835699,"replication":0, "permission":"770","ow..."
   RawContent        : HTTP/1.1 200 OK
                       Pragma: no-cache
                       x-ms-request-id: b4b31e16-e968-46a1-879a-3474aa7d4528
                       x-ms-webhdfs-version: 17.04.22.00
                       Status: 0x0
                       X-Content-Type-Options: nosniff
                       Strict-Transport-Security: ma...
   Forms             : {}
   Headers           : {[Pragma, no-cache], [x-ms-request-id, b4b31e16-e968-46a1-879a-3474aa7d4528],
                       [x-ms-webhdfs-version, 17.04.22.00], [Status, 0x0]...}
   Images            : {}
   InputFields       : {}
   Links             : {}
   ParsedHtml        : System.__ComObject
   RawContentLength  : 556
   ```

6. Ahora puede intentar cargar un archivo en su instancia de Data Lake Store.  En primer lugar, cree un archivo para cargarlo.

   ```powershell
   echo "Test file." > Test1.txt
   ```

7. Con el comando `Invoke-WebRequest` de PowerShell, realice una solicitud al punto de conexión de REST de Data Lake Store para cargar el archivo en la carpeta que creó anteriormente.  Esta solicitud implica dos pasos.  En el primer paso, se realiza una solicitud y se obtiene un redirección al lugar donde se debe cargar el archivo.  En el segundo paso, se carga realmente el archivo.  No olvide configurar el nombre de la carpeta y el archivo correctamente si ha usado valores diferentes a los que se muestran en este tutorial. 

   ```powershell
   $HdfsRedirectResponse = Invoke-WebRequest -Uri https://<YOUR_ADLS_NAME>.azuredatalakestore.net/webhdfs/v1/TestFolder/Test1.txt?op=CREATE -Method PUT -Headers @{Authorization="Bearer $AccessToken"} -Infile Test1.txt -MaximumRedirection 0
   ```

   Si examina el valor de `$HdfsRedirectResponse`, debería ser parecido a la siguiente respuesta:

   ```powershell
   PS C:\> $HdfsRedirectResponse

   StatusCode        : 307
   StatusDescription : Temporary Redirect
   Content           : {}
   RawContent        : HTTP/1.1 307 Temporary Redirect
                       Pragma: no-cache
                       x-ms-request-id: b7ab492f-b514-4483-aada-4aa0611d12b3
                       ContentLength: 0
                       x-ms-webhdfs-version: 17.04.22.00
                       Status: 0x0
                       X-Content-Type-Options: nosn...
   Headers           : {[Pragma, no-cache], [x-ms-request-id, b7ab492f-b514-4483-aada-4aa0611d12b3], 
                       [ContentLength, 0], [x-ms-webhdfs-version, 17.04.22.00]...}
   RawContentLength  : 0
   ```

   Para finalizar la carga, envíe una solicitud al punto de conexión de redirección:

   ```powershell
   Invoke-WebRequest -Uri $HdfsRedirectResponse.Headers.Location -Method PUT -Headers @{Authorization="Bearer $AccessToken"} -Infile Test1.txt -MaximumRedirection 0
   ```

   Una respuesta correcta se parece a esta:

   ```powershell
   StatusCode        : 201
   StatusDescription : Created
   Content           : {}
   RawContent        : HTTP/1.1 201 Created
                       Pragma: no-cache
                       x-ms-request-id: 1e70f36f-ead1-4566-acfa-d0c3ec1e2307
                       ContentLength: 0
                       x-ms-webhdfs-version: 17.04.22.00
                       Status: 0x0
                       X-Content-Type-Options: nosniff
                       Strict...
   Headers           : {[Pragma, no-cache], [x-ms-request-id, 1e70f36f-ead1-4566-acfa-d0c3ec1e2307],
                       [ContentLength, 0], [x-ms-webhdfs-version, 17.04.22.00]...}
   RawContentLength  : 0
   ```

Mediante otras API del sistema de archivos de Data Lake Store, puede anexar o descargar archivos, entre otras muchas cosas.


## <a name="disable"></a>Disable

[!INCLUDE [msi-tut-disable](../../../includes/active-directory-msi-tut-disable.md)]


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a usar una identidad administrada asignada por el sistema en una máquina virtual Windows para acceder a Azure Data Lake Store. Para obtener más información sobre Azure Data Lake Store, vea:

> [!div class="nextstepaction"]
>[Azure Data Lake Store](../../data-lake-store/data-lake-store-overview.md)