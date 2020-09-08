---
title: Tutorial sobre el uso de las referencias de Key Vault de Azure App Configuration en una aplicación de ASP.NET Core | Microsoft Docs
description: En este tutorial, aprenderá a usar las referencias de Key Vault de Azure App Configuration desde una aplicación de ASP.NET Core
services: azure-app-configuration
documentationcenter: ''
author: lisaguthrie
manager: maiye
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 04/08/2020
ms.author: lcozzens
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 3e6403f41d8e4b52ca64e9fa452524fa25efe870
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213260"
---
# <a name="tutorial-use-key-vault-references-in-an-aspnet-core-app"></a>Tutorial: Uso de referencias de Key Vault en una aplicación de ASP.NET Core

En este tutorial aprenderá a usar el servicio Azure App Configuration junto con Azure Key Vault. App Configuration y Key Vault son servicios complementarios que se usan de forma conjunta en la mayoría de las implementaciones de aplicaciones.

App Configuration ayuda a usar juntos los servicios, mediante la creación de claves que hacen referencia a valores almacenados en Key Vault. Cuando App Configuration crea estas claves, almacena los URI de los valores de Key Vault, en lugar de los propios valores.

La aplicación usa el proveedor de cliente de App Configuration para recuperar las referencias de Key Vault, tal como hace para cualquier otra clave almacenada en App Configuration. En este caso, los valores almacenados en App Configuration son los URI que hacen referencia a los valores de Key Vault. No son valores ni credenciales de Key Vault. Dado que el proveedor de cliente reconoce las claves como referencias de Key Vault, utiliza Key Vault para recuperar sus valores.

La aplicación es responsable de autenticarse correctamente en App Configuration y en Key Vault. Los dos servicios no se comunican directamente.

En este tutorial se muestra cómo se implementan las referencias de Key Vault en el código. Se basa en la aplicación web que se introdujo en los inicios rápidos. Antes de continuar, finalice primero el tutorial [Creación de una aplicación ASP.NET Core con Azure App Configuration](./quickstart-aspnet-core-app.md).

Para realizar los pasos de este tutorial, puede usar cualquier editor de código. Por ejemplo, [Visual Studio Code](https://code.visualstudio.com/) es un editor de código multiplataforma que está disponible para los sistemas operativos Windows, macOS y Linux.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una clave de App Configuration que hace referencia a un valor almacenado en Key Vault.
> * Acceder al valor de esta clave desde una aplicación web de ASP.NET Core.

## <a name="prerequisites"></a>Prerrequisitos

Antes de iniciar este tutorial, instale el [SDK de .NET Core](https://dotnet.microsoft.com/download).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-a-vault"></a>Creación de un almacén

1. Seleccione la opción **Crear un recurso** en la esquina superior izquierda de Azure Portal:

    ![Salida tras completarse la creación del almacén de claves](./media/quickstarts/search-services.png)
1. En el cuadro de búsqueda, escriba **Key Vault**.
1. En la lista de resultados, seleccione **Key Vaults** a la izquierda.
1. En **Key Vaults**, seleccione **Agregar**.
1. En la sección **Crear almacén de claves**, a la derecha, proporcione la siguiente información:
    - Seleccione **Suscripción** para elegir una suscripción.
    - En **Grupo de recursos**, seleccione **Crear nuevo** y escriba un nombre de grupo de recursos.
    - En **Nombre del almacén de claves** se requiere un nombre único. Para este tutorial, escriba **Contoso-vault2**.
    - En la lista desplegable **Región**, elija una ubicación.
1. Deje las demás opciones de **Crear almacén de claves** con sus valores predeterminados.
1. Seleccione **Crear**.

En este momento, su cuenta de Azure es la única autorizada para acceder a este nuevo almacén.

![Salida tras completarse la creación del almacén de claves](./media/quickstarts/vault-properties.png)

## <a name="add-a-secret-to-key-vault"></a>Incorporación de un secreto a Key Vault

Para agregar un secreto al almacén, debe llevar a cabo algunos pasos adicionales. En este caso, agregará un mensaje que podrá usar para probar la recuperación de Key Vault. El mensaje se denominará **Message** y en él se almacenará el valor "Hello from Key Vault".

1. En las páginas de propiedades de Key Vault, seleccione **Secretos**.
1. Seleccione **Generar o importar**.
1. En el panel **Crear un secreto**, escriba los valores siguientes:
    - **Opciones de carga**: escriba **Manual**.
    - **Name**: escriba **Message**.
    - **Valor**: escriba **Hello from Key Vault**.
1. Deje las demás propiedades de **Crear un secreto** con sus valores predeterminados.
1. Seleccione **Crear**.

## <a name="add-a-key-vault-reference-to-app-configuration"></a>Incorporación de una referencia de Key Vault a App Configuration

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Todos los recursos** y después seleccione la instancia de almacén de App Configuration que creó en el inicio rápido.

1. Seleccione **Explorador de configuración**.

1. Seleccione **+ Crear** > **Referencia del almacén de claves** y, a continuación, especifique los valores siguientes:
    - **Clave**: seleccione **TestApp:Settings:KeyVaultMessage**.
    - **Etiqueta**: deje este valor en blanco.
    - **Suscripción**, **Grupo de recursos** y **Key Vault**: escriba los valores correspondientes a los del almacén de claves que creó en la sección anterior.
    - **Secreto**: seleccione el secreto llamado **Message** que creó en la sección anterior.

## <a name="connect-to-key-vault"></a>Conexión a Key Vault

1. En este tutorial usará una entidad de servicio para la autenticación en Key Vault. Para crear esta entidad de servicio, use el comando [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) de la CLI de Azure:

    ```azurecli
    az ad sp create-for-rbac -n "http://mySP" --sdk-auth
    ```

    Esta operación devuelve una serie de pares clave-valor:

    ```console
    {
    "clientId": "7da18cae-779c-41fc-992e-0527854c6583",
    "clientSecret": "b421b443-1669-4cd7-b5b1-394d5c945002",
    "subscriptionId": "443e30da-feca-47c4-b68f-1636b75e16b3",
    "tenantId": "35ad10f1-7799-4766-9acf-f2d946161b77",
    "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
    "resourceManagerEndpointUrl": "https://management.azure.com/",
    "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
    "galleryEndpointUrl": "https://gallery.azure.com/",
    "managementEndpointUrl": "https://management.core.windows.net/"
    }
    ```

1. Ejecute el siguiente comando para permitir que la entidad de servicio acceda al almacén de claves:

    ```cmd
    az keyvault set-policy -n <your-unique-keyvault-name> --spn <clientId-of-your-service-principal> --secret-permissions delete get list set --key-permissions create decrypt delete encrypt get list unwrapKey wrapKey
    ```

1. Agregue variables de entorno para almacenar los valores de *clientId*, *clientSecret* y *tenantId*.

    #### <a name="windows-command-prompt"></a>[Símbolo del sistema de Windows](#tab/cmd)

    ```cmd
    setx AZURE_CLIENT_ID <clientId-of-your-service-principal>
    setx AZURE_CLIENT_SECRET <clientSecret-of-your-service-principal>
    setx AZURE_TENANT_ID <tenantId-of-your-service-principal>
    ```

    #### <a name="powershell"></a>[PowerShell](#tab/powershell)

    ```PowerShell
    $Env:AZURE_CLIENT_ID = <clientId-of-your-service-principal>
    $Env:AZURE_CLIENT_SECRET = <clientSecret-of-your-service-principal>
    $Env:AZURE_TENANT_ID = <tenantId-of-your-service-principal>
    ```

    #### <a name="bash"></a>[Bash](#tab/bash)

    ```bash
    export AZURE_CLIENT_ID = <clientId-of-your-service-principal>
    export AZURE_CLIENT_SECRET = <clientSecret-of-your-service-principal>
    export AZURE_TENANT_ID = <tenantId-of-your-service-principal>
    ```

    ---

    > [!NOTE]
    > Estas credenciales de Key Vault se usan solo dentro de la aplicación. La aplicación se autentica directamente en Key Vault con estas credenciales. Nunca se pasan al servicio App Configuration.

1. Reinicie el terminal para cargar estas nuevas variables de entorno.

## <a name="update-your-code-to-use-a-key-vault-reference"></a>Actualización del código para usar una referencia de Key Vault

1. Para agregar una referencia a los paquetes de NuGet necesarios, ejecute el siguiente comando:

    ```dotnetcli
    dotnet add package Azure.Identity
    ```

1. Abra *Program.cs* y agregue referencias a los siguientes paquetes necesarios:

    ```csharp
    using Azure.Identity;
    ```

1. Actualice el método `CreateWebHostBuilder` para usar App Configuration; para ello, llame al método `config.AddAzureAppConfiguration`. Incluya la opción `ConfigureKeyVault` y pase las credenciales correctas a Key Vault.

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();

                config.AddAzureAppConfiguration(options =>
                {
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                            .ConfigureKeyVault(kv =>
                            {
                                kv.SetCredential(new DefaultAzureCredential());
                            });
                });
            })
            .UseStartup<Startup>();
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();

                config.AddAzureAppConfiguration(options =>
                {
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                            .ConfigureKeyVault(kv =>
                            {
                                kv.SetCredential(new DefaultAzureCredential());
                            });
                });
            })
            .UseStartup<Startup>());
    ```

1. Al inicializar la conexión a App Configuration, configura la conexión a Key Vault llamando al método `ConfigureKeyVault`. Después de la inicialización, puede acceder a los valores de las referencias de Key Vault de la misma manera que accede a los valores de las claves normales de App Configuration.

    Para ver este proceso en acción, abra *Index.cshtml* en la carpeta **Views** > **Home**. Reemplace su contenido por el código siguiente:

    ```html
    @using Microsoft.Extensions.Configuration
    @inject IConfiguration Configuration

    <style>
        body {
            background-color: @Configuration["TestApp:Settings:BackgroundColor"]
        }
        h1 {
            color: @Configuration["TestApp:Settings:FontColor"];
            font-size: @Configuration["TestApp:Settings:FontSize"]px;
        }
    </style>

    <h1>@Configuration["TestApp:Settings:Message"]
        and @Configuration["TestApp:Settings:KeyVaultMessage"]</h1>
    ```

    Puede acceder al valor de la referencia de Key Vault **TestApp:Settings:KeyVaultMessage** de la misma forma que al valor de configuración **TestApp:Settings:Message**.

## <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación en un entorno local

1. Para compilar la aplicación mediante la CLI de .NET Core, ejecute el siguiente comando en el shell de comandos:

    ```dotnetcli
    dotnet build
    ```

1. Una vez finalizada la compilación, use el siguiente comando para ejecutar la aplicación web localmente:

    ```dotnetcli
    dotnet run
    ```

1. Inicie una ventana del explorador y vaya a `http://localhost:5000`, que es la dirección URL predeterminada de la aplicación web hospedada localmente.

    ![Inicio de la aplicación local del inicio rápido](./media/key-vault-reference-launch-local.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha creado una clave de App Configuration que hace referencia a un valor almacenado en Key Vault. Para aprender a agregar una identidad de servicio administrada por Azure que optimice el acceso a App Configuration y Key Vault, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Integración de identidades administradas](./howto-integrate-azure-managed-service-identity.md)
