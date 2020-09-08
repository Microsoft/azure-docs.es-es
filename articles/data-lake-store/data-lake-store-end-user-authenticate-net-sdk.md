---
title: 'Autenticación de usuario final en .NET con Data Lake Storage Gen1: Azure'
description: Aprenda a lograr la autenticación del usuario final con Azure Data Lake Storage Gen1 mediante Azure Active Directory con SDK de .NET.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: devx-track-csharp
ms.openlocfilehash: 67ba4f12aec9e987d79109b7197d03301bf40650
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89004789"
---
# <a name="end-user-authentication-with-azure-data-lake-storage-gen1-using-net-sdk"></a>Autenticación de usuario final con Azure Data Lake Storage Gen1 mediante el SDK de .NET.
> [!div class="op_single_selector"]
> * [Uso de Java](data-lake-store-end-user-authenticate-java-sdk.md)
> * [Uso de SDK de .NET](data-lake-store-end-user-authenticate-net-sdk.md)
> * [Uso de Python](data-lake-store-end-user-authenticate-python.md)
> * [Uso de la API de REST](data-lake-store-end-user-authenticate-rest-api.md)
> 
>  

En este artículo, aprenderá a usar el SDK de .NET para realizar la autenticación de usuario final con Azure Data Lake Storage Gen1. Para la autenticación entre servicios con Data Lake Storage Gen1. mediante el SDK de .NET, consulte [Service-to-service authentication with Data Lake Storage Gen1 using .NET SDK](data-lake-store-service-to-service-authenticate-net-sdk.md) (Autenticación entre servicios con Data Lake Storage Gen1 mediante el SDK de .NET).

## <a name="prerequisites"></a>Prerrequisitos
* **Visual Studio 2013 o superior**. En las instrucciones siguientes se usa Visual Studio 2019.

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

* **Cree una aplicación "nativa" de Azure Active Directory**. Debe completar los pasos descritos en [Autenticación de usuario final con Data Lake Storage Gen1 mediante Azure Active Directory](data-lake-store-end-user-authenticate-using-active-directory.md).

## <a name="create-a-net-application"></a>Creación de una aplicación .NET
1. En Visual Studio, en el menú **Archivo**, seleccione **Nuevo** y, a continuación, **Proyecto**.
2. Elija **Aplicación de consola (.NET Framework)** y, a continuación, seleccione **Siguiente**.
3. En el **nombre del proyecto**, escriba `CreateADLApplication` y, a continuación, seleccione **Crear**.

4. Agregue los paquetes NuGet al proyecto.

   1. Haga clic con el botón derecho en el Explorador de soluciones y haga clic en **Administrar paquetes de NuGet**.
   2. En la pestaña **Administrador de paquetes NuGet**, asegúrese de que la opción **Origen del paquete** esté establecida en **nuget.org** y que esté activada la casilla **Incluir versión preliminar**.
   3. Busque e instale los siguientes paquetes NuGet:

      * `Microsoft.Azure.Management.DataLake.Store` - En este tutorial se usa v2.1.3 (versión preliminar).
      * `Microsoft.Rest.ClientRuntime.Azure.Authentication` - En este tutorial se usa v2.2.12.

        ![Incorporación de un origen de NuGet](./media/data-lake-store-get-started-net-sdk/data-lake-store-install-nuget-package.png "Creación de una nueva cuenta de Azure Data Lake")
   4. Cierre el **Administrador de paquetes NuGet**.

5. Abrir **Program.cs**
6. Reemplace las instrucciones Using por las siguientes líneas:

    ```csharp
    using System;
    using System.IO;
    using System.Linq;
    using System.Text;
    using System.Threading;
    using System.Collections.Generic;
            
    using Microsoft.Rest;
    using Microsoft.Rest.Azure.Authentication;
    using Microsoft.Azure.Management.DataLake.Store;
    using Microsoft.Azure.Management.DataLake.Store.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```     

## <a name="end-user-authentication"></a>Autenticación de usuario final
Agregue este fragmento de código a su aplicación cliente .NET. Reemplace los valores de marcador de posición por los valores recuperados de una aplicación nativa de Azure AD (se enumera como requisito previo). Este fragmento de código le permite autenticar la aplicación **de manera interactiva** con Data Lake Store Gen1, lo que significa que se le pedirá que escriba sus credenciales de Azure.

Para facilitar su uso, el siguiente fragmento de código emplea valores predeterminados para el identificador de cliente y un URI de redirección que son válidos con cualquier suscripción de Azure. En el siguiente fragmento de código, solo es necesario proporcionar el valor del identificador del inquilino. Puede recuperar el identificador del inquilino siguiendo las instrucciones proporcionadas en [Obtención del identificador de inquilino](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
    
- Reemplace la función Main() por el siguiente contenido:

    ```csharp
    private static void Main(string[] args)
    {
        //User login via interactive popup
        string TENANT = "<AAD-directory-domain>";
        string CLIENTID = "1950a258-227b-4e31-a9cf-717495945fc2";
        System.Uri ARM_TOKEN_AUDIENCE = new System.Uri(@"https://management.core.windows.net/");
        System.Uri ADL_TOKEN_AUDIENCE = new System.Uri(@"https://datalake.azure.net/");
        string MY_DOCUMENTS = System.Environment.GetFolderPath(System.Environment.SpecialFolder.MyDocuments);
        string TOKEN_CACHE_PATH = System.IO.Path.Combine(MY_DOCUMENTS, "my.tokencache");
        var tokenCache = GetTokenCache(TOKEN_CACHE_PATH);
        var armCreds = GetCreds_User_Popup(TENANT, ARM_TOKEN_AUDIENCE, CLIENTID, tokenCache);
        var adlCreds = GetCreds_User_Popup(TENANT, ADL_TOKEN_AUDIENCE, CLIENTID, tokenCache);
    }
    ```

Dos cosas que conviene saber acerca del fragmento de código anterior:

* El fragmento de código anterior utiliza las funciones auxiliares `GetTokenCache` y `GetCreds_User_Popup`. El código de estas funciones auxiliares está disponible [aquí en GitHub](https://github.com/Azure-Samples/data-lake-analytics-dotnet-auth-options#gettokencache).
* Para ayudarle a completar este tutorial con más rapidez, este fragmento de código usa el identificador de cliente de la aplicación nativa que está disponible de manera predeterminada para todas las suscripciones de Azure. Por lo tanto, puede **usar este fragmento de código tal cual en la aplicación**.
* Sin embargo, si desea utilizar su propio identificador de cliente de dominio y de aplicación de Azure AD, debe crear una aplicación nativa de Azure AD y, después, utilizar el identificador de inquilino de Azure AD, el identificador de cliente y el identificador URI de redirección para la aplicación que ha creado. Consulte [Create an Active Directory Application for end-user authentication with Data Lake Storage Gen1](data-lake-store-end-user-authenticate-using-active-directory.md) (Crear una aplicación de Active Directory para la autenticación del usuario final con Data Lake Storage Gen1) para obtener más instrucciones.

  
## <a name="next-steps"></a>Pasos siguientes
En este artículo, aprendió a usar la autenticación de usuario final para autenticarse en Azure Data Lake Storage Gen1 mediante el SDK de .NET. Ahora puede consultar los siguientes artículos, que tratan sobre cómo usar el SDK de .NET con Azure Data Lake Storage Gen1.

* [Operaciones de administración de cuentas en Data Lake Storage Gen1 con el SDK de .NET](data-lake-store-get-started-net-sdk.md)
* [Operaciones de datos en Data Lake Storage Gen1 mediante el SDK de .NET](data-lake-store-data-operations-net-sdk.md)

