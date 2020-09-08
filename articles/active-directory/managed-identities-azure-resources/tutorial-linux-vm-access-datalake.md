---
title: 'Tutorial`:` Uso de identidades administradas para acceder a Azure Data Lake Store (Linux): Azure AD'
description: En este tutorial, se explica cómo se utiliza una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Data Lake Store.
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
ms.date: 01/10/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: d465419dfe36fd5dd67abdef22a6f54fba69a98e
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267469"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-data-lake-store"></a>Tutorial: Uso de identidades administradas asignadas por el sistema de una máquina virtual Linux para acceder a Azure Data Lake Store

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial se muestra cómo usar una identidad administrada asignada por el sistema para una máquina virtual Linux a fin de acceder a Azure Data Lake Store. Aprenderá a: 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Conceder a la máquina virtual acceso a Azure Data Lake Store.
> * Obtener un token de acceso utilizando la identidad administrada asignada por el sistema de la máquina virtual para acceder a Azure Data Lake Store.

## <a name="prerequisites"></a>Prerequisites

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

## <a name="grant-access"></a>Conceder acceso

En esta sección se muestra cómo conceder a la máquina virtual el acceso a los archivos y las carpetas de Azure Data Lake Store. En este paso, puede usar una instancia ya existente de Data Lake Store o crear una nueva. Para crear una instancia de Data Lake Store mediante Azure Portal, siga la [guía de inicio rápido de Azure Data Lake Store](../../data-lake-store/data-lake-store-get-started-portal.md). También se pueden encontrar guías de inicio rápido que utilizan la CLI de Azure y Azure PowerShell en la [documentación de Azure Data Lake Store](../../data-lake-store/data-lake-store-overview.md).

En Data Lake Store, cree una nueva carpeta y conceda a la identidad administrada asignada por el sistema de la máquina virtual Linux permisos de lectura, escritura y ejecución de archivos en dicha carpeta:

1. En Azure Portal, seleccione **Data Lake Store** en el panel izquierdo.
2. Seleccione la instancia de Data Lake Store que desea utilizar.
3. Seleccione **Explorador de datos** en la barra de comandos.
4. Se selecciona la carpeta raíz de la instancia de Data Lake Store. Seleccione **Acceso** en la barra de comandos.
5. Seleccione **Agregar**.  En el cuadro **Seleccionar**, escriba el nombre de la máquina virtual, por ejemplo **DevTestVM**. Seleccione la máquina virtual en los resultados de la búsqueda y, a continuación, haga clic en **Seleccionar**.
6. Haga clic en **Seleccionar permisos**.  Seleccione **Lectura** y **Ejecutar**, agréguelos a **Esta carpeta** y agréguelos como **un solo permiso de acceso**. Seleccione **Aceptar**.  El permiso se debe haber agregado correctamente.
7. Cierre el panel **Acceso**.
8. En este tutorial, cree una nueva carpeta. Seleccione **Nueva carpeta** en la barra de comandos y asígnele un nombre, por ejemplo **TestFolder**.  Seleccione **Aceptar**.
9. Seleccione la carpeta que creó y, a continuación, seleccione **Acceso** en la barra de comandos.
10. De forma similar al paso 5, seleccione **Agregar**. En el cuadro **Seleccionar**, escriba el nombre de la máquina virtual. Seleccione la máquina virtual en los resultados de la búsqueda y, a continuación, haga clic en **Seleccionar**.
11. De forma similar al paso 6, seleccione **Seleccionar permisos**. Seleccione **Lectura**, **Escritura** y **Ejecutar**, agréguelos a **Esta carpeta** y agréguelos como **Una entrada de permiso de acceso y una entrada de permiso predeterminado**. Seleccione **Aceptar**.  El permiso se debe haber agregado correctamente.

Ahora, Managed Identities for Azure Resources puede realizar todas las operaciones en los archivos de la carpeta que ha creado. Para más información sobre cómo administrar el acceso a Data Lake Store, consulte [Control de acceso en Data Lake Store](../../data-lake-store/data-lake-store-access-control.md).

## <a name="get-an-access-token"></a>Obtención de un token de acceso 

En esta sección se muestra cómo obtener un token de acceso y llamar al sistema de archivos de Data Lake Store. Azure Data Lake Store admite de forma nativa la autenticación de Azure AD, por lo que puede aceptar directamente los tokens de acceso obtenidos con Managed Identities for Azure Resources. Para autenticarse en el sistema de archivos de Data Lake Store, se envía un token de acceso emitido por Azure AD al punto de conexión del sistema de archivos de Data Lake Store. El token de acceso está en un encabezado de autorización en el formato "Bearer \<ACCESS_TOKEN_VALUE\>".  Para más información sobre la compatibilidad de Data Lake Store con la autenticación de Azure AD, consulte [Autenticación con Data Lake Store mediante Azure Active Directory](../../data-lake-store/data-lakes-store-authentication-using-azure-active-directory.md).

En este tutorial, se autenticará en la API de REST del sistema de archivos de Data Lake Store mediante cURL para realizar solicitudes de REST.

> [!NOTE]
> Los SDK de cliente del sistema de archivos de Data Lake Store todavía no son compatibles con Managed Identities for Azure Resources.

Para completar estos pasos, necesitará un cliente SSH. Si usa Windows, puede usar el cliente SSH en el [Subsistema de Windows para Linux](/windows/wsl/about). Si necesita ayuda para configurar las claves del cliente de SSH, consulte [Uso de claves de SSH con Windows en Azure](../../virtual-machines/linux/ssh-from-windows.md) o [Creación y uso de un par de claves SSH pública y privada para máquinas virtuales Linux en Azure](../../virtual-machines/linux/mac-create-ssh-keys.md).

1. En el portal, vaya a la máquina virtual Linux. En **Información general**, seleccione **Conectar**.  
2. Conéctese a la máquina virtual con el cliente SSH de su elección. 
3. En la ventana del terminal, mediante cURL, realice una solicitud al punto de conexión de las identidades administradas locales de los recursos de Azure y obtener un token de acceso para el sistema de archivos de Data Lake Store. El identificador de recurso de Data Lake Store es `https://datalake.azure.net/`.  Es importante incluir la barra diagonal final en el identificador del recurso.
    
   ```bash
   curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatalake.azure.net%2F' -H Metadata:true   
   ```
    
   Una respuesta correcta devuelve el token de acceso que se utiliza para autenticarse en Data Lake Store:

   ```bash
   {"access_token":"eyJ0eXAiOiJ...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1508119757",
    "not_before":"1508115857",
    "resource":"https://datalake.azure.net/",
    "token_type":"Bearer"}
   ```

4. Con CURL, realice una solicitud al punto de conexión de REST del sistema de archivos de Data Lake Store para enumerar las carpetas que contiene la carpeta raíz. Se trata de una manera sencilla de comprobar que todo está configurado correctamente. Copie el valor del token de acceso del paso anterior. Es importante que la cadena "Bearer" en el encabezado de autorización tenga la "B" en mayúscula. Puede encontrar el nombre de la instancia de Data Lake Store en la sección **Información general** del panel **Data Lake Store** de Azure Portal.

   ```bash
   curl https://<YOUR_ADLS_NAME>.azuredatalakestore.net/webhdfs/v1/?op=LISTSTATUS -H "Authorization: Bearer <ACCESS_TOKEN>"
   ```
    
   Una respuesta correcta tiene el siguiente aspecto:

   ```bash
   {"FileStatuses":{"FileStatus":[{"length":0,"pathSuffix":"TestFolder","type":"DIRECTORY","blockSize":0,"accessTime":1507934941392,"modificationTime":1508105430590,"replication":0,"permission":"770","owner":"bd0e76d8-ad45-4fe1-8941-04a7bf27f071","group":"bd0e76d8-ad45-4fe1-8941-04a7bf27f071"}]}}
   ```

5. Ahora puede intentar cargar un archivo en su instancia de Data Lake Store. En primer lugar, cree un archivo para cargarlo.

   ```bash
   echo "Test file." > Test1.txt
   ```

6. Con CURL, realice una solicitud al punto de conexión de REST del sistema de archivos de Data Lake Store para cargar el archivo en la carpeta que creó anteriormente. La carga implica una redirección y cURL sigue la redirección automáticamente. 

   ```bash
   curl -i -X PUT -L -T Test1.txt -H "Authorization: Bearer <ACCESS_TOKEN>" 'https://<YOUR_ADLS_NAME>.azuredatalakestore.net/webhdfs/v1/<FOLDER_NAME>/Test1.txt?op=CREATE' 
   ```

    Una respuesta correcta tiene el siguiente aspecto:

   ```bash
   HTTP/1.1 100 Continue
   HTTP/1.1 307 Temporary Redirect
   Cache-Control: no-cache, no-cache, no-store, max-age=0
   Pragma: no-cache
   Expires: -1
   Location: https://mytestadls.azuredatalakestore.net/webhdfs/v1/TestFolder/Test1.txt?op=CREATE&write=true
   x-ms-request-id: 756f6b24-0cca-47ef-aa12-52c3b45b954c
   ContentLength: 0
   x-ms-webhdfs-version: 17.04.22.00
   Status: 0x0
   X-Content-Type-Options: nosniff
   Strict-Transport-Security: max-age=15724800; includeSubDomains
   Date: Sun, 15 Oct 2017 22:10:30 GMT
   Content-Length: 0
       
   HTTP/1.1 100 Continue
       
   HTTP/1.1 201 Created
   Cache-Control: no-cache, no-cache, no-store, max-age=0
   Pragma: no-cache
   Expires: -1
   Location: https://mytestadls.azuredatalakestore.net/webhdfs/v1/TestFolder/Test1.txt?op=CREATE&write=true
   x-ms-request-id: af5baa07-3c79-43af-a01a-71d63d53e6c4
   ContentLength: 0
   x-ms-webhdfs-version: 17.04.22.00
   Status: 0x0
   X-Content-Type-Options: nosniff
   Strict-Transport-Security: max-age=15724800; includeSubDomains
   Date: Sun, 15 Oct 2017 22:10:30 GMT
   Content-Length: 0
   ```

Mediante otras API del sistema de archivos de Data Lake Store, puede anexar o descargar archivos, entre otras muchas cosas.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Data Lake Store. Para obtener más información sobre Azure Data Lake Store, vea:

> [!div class="nextstepaction"]
>[Azure Data Lake Store](../../data-lake-store/data-lake-store-overview.md)