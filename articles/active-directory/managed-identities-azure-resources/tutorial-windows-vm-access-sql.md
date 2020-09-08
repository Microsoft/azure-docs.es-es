---
title: Tutorial`:` Uso de una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure SQL
description: Este tutorial contiene directrices acerca de cómo utilizar una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure SQL Database.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/14/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: d576fb4f5dea10a2adf0d7488aa422e1397fd6d1
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89255756"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-sql"></a>Tutorial: Uso de una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure SQL

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial se muestra cómo usar una identidad asignada por el sistema en una máquina virtual de Windows para acceder a Azure SQL Database. Las identidades de MSI son administradas automáticamente por Azure y le permiten autenticar los servicios que admiten la autenticación de Azure AD sin necesidad de insertar credenciales en el código. Aprenderá a:

> [!div class="checklist"]
>
> * Dar a una máquina virtual acceso a Azure SQL Database
> * Habilitación de la autenticación de Azure AD
> * Cree un usuario contenido en la base de datos que represente la identidad asignada por el sistema de la máquina virtual.
> * Obtener un token de acceso mediante la identidad de máquina virtual y usarlo para hacer consultas a Azure SQL Database

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

## <a name="enable"></a>Habilitar

[!INCLUDE [msi-tut-enable](../../../includes/active-directory-msi-tut-enable.md)]

## <a name="grant-access"></a>Conceder acceso

Para conceder a la máquina virtual acceso a una base de datos de Azure SQL Server, puede usar un [servidor SQL lógico](../../azure-sql/database/logical-servers.md) existente o crear uno. Para crear un servidor y una base de datos con Azure Portal, siga esta [Guía de inicio rápido de Azure SQL](../../azure-sql/database/single-database-create-quickstart.md). También hay guías de inicio rápido que utilizan la CLI de Azure y Azure PowerShell en la [documentación de Azure SQL](/azure/sql-database/).

Hay dos pasos para conceder a la máquina virtual acceso a una base de datos:

1. Habilite la autenticación de Azure AD para el servidor.
2. Cree un **usuario contenido** en la base de datos que represente la identidad asignada por el sistema de la máquina virtual.

### <a name="enable-azure-ad-authentication"></a>Habilitación de la autenticación de Azure AD

**Para [configurar la autenticación de Azure AD](../../azure-sql/database/authentication-aad-configure.md):**

1. En Azure Portal, seleccione **Servidores SQL Server** en el panel de navegación izquierdo.
2. Haga clic en el servidor SQL para habilitarlo para la autenticación de Azure AD.
3. En la sección **Configuración** de la hoja, haga clic en **Administrador de Active Directory**.
4. En la barra de comandos, haga clic en **Establecer administrador**.
5. Seleccione una cuenta de usuario de Azure AD para que se convierta en administrador del servidor y haga clic en **Seleccionar.**
6. En la barra de comandos, haga clic en **Guardar**

### <a name="create-contained-user"></a>Creación del usuario contenido

En esta sección se muestra cómo crear un usuario contenido en la base de datos que represente la identidad asignada por el sistema de la máquina virtual. En este paso, necesita [Microsoft SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS). Antes de comenzar, también puede ser útil revisar los artículos siguientes para obtener información sobre la integración de Azure AD:

- [Autenticación universal con SQL Database y Azure Synapse Analytics (compatibilidad de SSMS con MFA)](../../azure-sql/database/authentication-mfa-ssms-overview.md)
- [Configuración y administración de la autenticación de Azure Active Directory con SQL Database o Azure Synapse Analytics](../../azure-sql/database/authentication-aad-configure.md)

SQL DB requiere nombres para mostrar de AAD únicos. Con esto, las cuentas de AAD tales como usuarios, grupos y entidades de servicio (aplicaciones) y nombres de máquina virtual habilitados para la identidad administrada deben definirse de forma única en AAD en relación con sus nombres para mostrar. SQL DB comprueba el nombre para mostrar de AAD durante la creación de T-SQL de dichos usuarios y, si no es único, el comando no puede solicitar que se proporcione un nombre para mostrar de AAD único para una cuenta especificada.

**Para crear un usuario contenido:**

1. Inicie SQL Server Management Studio.
2. En el cuadro de diálogo **Conectar al servidor**, escriba el nombre de su servidor en el campo **Nombre del servidor**.
3. En el campo **Autenticación**, seleccione **Active Directory - Universal compatible con MFA**.
4. En el campo **Nombre de usuario**, escriba el nombre de la cuenta de Azure AD que estableció como administradora del servidor, por ejemplo,helen@woodgroveonline.com
5. Haga clic en **Opciones**.
6. En el campo **Conectar una base de datos**, escriba el nombre de la base de datos que no es del sistema y que desea configurar.
7. Haga clic en **Conectar**. Complete el proceso de inicio de sesión.
8. En el **Explorador de objetos**, expanda la carpeta **Bases de datos**.
9. Haga clic con el botón derecho en una base de datos de usuario y haga clic en **Nueva consulta**.
10. En la ventana de consulta, escriba la línea siguiente y haga clic en **Ejecutar** en la barra de herramientas:

    > [!NOTE]
    > `VMName` en el siguiente comando es el nombre de la máquina virtual en la que habilitó la identidad asignada por el sistema en la sección de requisitos previos.

    ```sql
    CREATE USER [VMName] FROM EXTERNAL PROVIDER
    ```

    El comando se completará correctamente y se creará el usuario contenido para la identidad asignada por el sistema de la máquina virtual.
11. Limpie la ventana de consulta, escriba la línea siguiente y haga clic en **Ejecutar** en la barra de herramientas:

    > [!NOTE]
    > `VMName` en el siguiente comando es el nombre de la máquina virtual en la que habilitó la identidad asignada por el sistema en la sección de requisitos previos.

    ```sql
    ALTER ROLE db_datareader ADD MEMBER [VMName]
    ```

    El comando debería completarse correctamente, concediendo al usuario contenido la capacidad de leer la base de datos completa.

El código que se ejecuta en la máquina virtual ahora puede obtener un token con su identidad administrada asignada por el sistema y usarlo para autenticarse en el servidor.

## <a name="access-data"></a>Acceso a los datos

En esta sección se muestra cómo obtener un token de acceso con una identidad administrada asignada por el sistema de la máquina virtual y usarla para llamar a Azure SQL. Azure SQL admite de forma nativa la autenticación de Azure AD, por lo que puede aceptar directamente los tokens de acceso obtenidos con la característica Managed Identities for Azure Resources. Se usa el método **token de acceso** para la creación de una conexión a SQL. Forma parte de la integración de Azure SQL con Azure AD y es diferente de proporcionar las credenciales en la cadena de conexión.

Este es un ejemplo de código .NET para abrir una conexión a SQL con un token de acceso. El código se debe ejecutar en la máquina virtual para poder acceder al punto de conexión de la identidad administrada asignada por el sistema de la máquina virtual. Se requiere **.NET Framework 4.6** o **.NET Core 2.2** o posterior para usar el método de token de acceso. Reemplace los valores de SQL-SERVERNAME y DATABASE en consecuencia. Tenga en cuenta el identificador de recurso para Azure SQL es `https://database.windows.net/`.

```csharp
using System.Net;
using System.IO;
using System.Data.SqlClient;
using System.Web.Script.Serialization;

//
// Get an access token for SQL.
//
HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://database.windows.net/");
request.Headers["Metadata"] = "true";
request.Method = "GET";
string accessToken = null;

try
{
    // Call managed identities for Azure resources endpoint.
    HttpWebResponse response = (HttpWebResponse)request.GetResponse();

    // Pipe response Stream to a StreamReader and extract access token.
    StreamReader streamResponse = new StreamReader(response.GetResponseStream());
    string stringResponse = streamResponse.ReadToEnd();
    JavaScriptSerializer j = new JavaScriptSerializer();
    Dictionary<string, string> list = (Dictionary<string, string>) j.Deserialize(stringResponse, typeof(Dictionary<string, string>));
    accessToken = list["access_token"];
}
catch (Exception e)
{
    string errorText = String.Format("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
}

//
// Open a connection to the server using the access token.
//
if (accessToken != null) {
    string connectionString = "Data Source=<AZURE-SQL-SERVERNAME>; Initial Catalog=<DATABASE>;";
    SqlConnection conn = new SqlConnection(connectionString);
    conn.AccessToken = accessToken;
    conn.Open();
}
```

Como alternativa, un método rápido para probar la configuración de extremo a otro extremo sin tener que escribir e implementar una aplicación en la máquina virtual es usar PowerShell.

1. En el portal, vaya a **Virtual Machines** y diríjase a la máquina virtual Windows. A continuación, en **Introducción**, haga clic en **Conectar**.
2. Escriba su **nombre de usuario** y **contraseña** que agregó cuando creó la máquina virtual Windows.
3. Ahora que ha creado una **conexión a Escritorio remoto** con la máquina virtual, abra **PowerShell** en la sesión remota.
4. Mediante `Invoke-WebRequest` de PowerShell, realice una solicitud al punto de conexión de la identidad administrada local para obtener un token de acceso para Azure SQL.

    ```powershell
        $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatabase.windows.net%2F' -Method GET -Headers @{Metadata="true"}
    ```

    Convierta la respuesta de un objeto JSON a un objeto de PowerShell.

    ```powershell
    $content = $response.Content | ConvertFrom-Json
    ```

    Extraiga el token de acceso de la respuesta.

    ```powershell
    $AccessToken = $content.access_token
    ```

5. Abra una conexión con el servidor. No olvide reemplazar los valores de AZURE-SQL-SERVERNAME y DATABASE.

    ```powershell
    $SqlConnection = New-Object System.Data.SqlClient.SqlConnection
    $SqlConnection.ConnectionString = "Data Source = <AZURE-SQL-SERVERNAME>; Initial Catalog = <DATABASE>"
    $SqlConnection.AccessToken = $AccessToken
    $SqlConnection.Open()
    ```

    Después, cree una consulta y envíela al servidor. No olvide reemplazar el valor de TABLE.

    ```powershell
    $SqlCmd = New-Object System.Data.SqlClient.SqlCommand
    $SqlCmd.CommandText = "SELECT * from <TABLE>;"
    $SqlCmd.Connection = $SqlConnection
    $SqlAdapter = New-Object System.Data.SqlClient.SqlDataAdapter
    $SqlAdapter.SelectCommand = $SqlCmd
    $DataSet = New-Object System.Data.DataSet
    $SqlAdapter.Fill($DataSet)
    ```

Examine el valor de `$DataSet.Tables[0]` para ver los resultados de la consulta.

## <a name="disable"></a>Disable

[!INCLUDE [msi-tut-disable](../../../includes/active-directory-msi-tut-disable.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar una identidad administrada asignada por el sistema para acceder a Azure SQL Database. Para más información acerca de Azure SQL Database, consulte:

> [!div class="nextstepaction"]
> [Azure SQL Database](../../azure-sql/database/sql-database-paas-overview.md)