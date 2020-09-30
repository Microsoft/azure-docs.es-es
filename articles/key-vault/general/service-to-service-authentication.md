---
title: Autenticación entre servicios en Azure Key Vault mediante .NET
description: Utilice la biblioteca Microsoft.Azure.Services.AppAuthentication para autenticar en Azure Key Vault mediante .NET.
keywords: credenciales locales de autenticación de azure key-vault
author: msmbaldwin
services: key-vault
ms.author: mbaldwin
ms.date: 08/08/2020
ms.topic: how-to
ms.service: key-vault
ms.subservice: general
ms.openlocfilehash: 860f9b0e49423b5d144d56ecd965153f7a362d87
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89180922"
---
# <a name="service-to-service-authentication-to-azure-key-vault-using-net"></a>Autenticación entre servicios en Azure Key Vault mediante .NET

> [!NOTE]
> Ya no se recomienda usar **Microsoft.Azure.Services.AppAuthentication** con el nuevo SDK de Key Vault. Se reemplaza por la nueva biblioteca de identidades de Azure **DefaultAzureCredentials**, disponible para .NET, Java, TypeScript y Python, y debe usarse para todo el desarrollo nuevo. Puede encontrar más información aquí: [Autenticación y el SDK de Azure](https://devblogs.microsoft.com/azure-sdk/authentication-and-the-azure-sdk/).

Para autenticarse en Azure Key Vault, necesitará una credencial de Azure Active Directory (Azure AD), un secreto compartido o un certificado.

La administración de estas credenciales puede ser complicada. Resulta tentador agrupar las credenciales en una aplicación incluyéndolas en archivos de configuración o de origen. La biblioteca `Microsoft.Azure.Services.AppAuthentication` para .NET simplifica este problema. Utiliza las credenciales del desarrollador para la autenticación durante el desarrollo local. Cuando la solución se implementa más adelante en Azure, la biblioteca cambia automáticamente a las credenciales de la aplicación. El uso de credenciales de desarrollador durante el desarrollo local es más seguro porque no es necesario crear credenciales de Azure AD o compartir credenciales entre los programadores.

La biblioteca `Microsoft.Azure.Services.AppAuthentication` administra la autenticación automáticamente, que a su vez le permite centrarse en la solución, en lugar de en las credenciales. Admite el desarrollo local con Microsoft Visual Studio, la CLI de Azure o la autenticación integrada de Azure AD. Cuando se implementa en un recurso de Azure que admite una identidad administrada, la biblioteca usa automáticamente [identidades administradas para los recursos de Azure](../../active-directory/msi-overview.md). No se requieren cambios de configuración o código. La biblioteca también admite el uso directo de las [credenciales de cliente](../../azure-resource-manager/resource-group-authenticate-service-principal.md) de Azure AD cuando una identidad administrada no está disponible o cuando no se puede determinar el contexto de seguridad del desarrollador durante el desarrollo local.

## <a name="prerequisites"></a>Prerrequisitos

- [Visual Studio 2019](https://www.visualstudio.com/downloads/) o [Visual Studio 2017 v15.5](https://blogs.msdn.microsoft.com/visualstudio/2017/10/11/visual-studio-2017-version-15-5-preview/).

- La extensión de autenticación de la aplicación para Visual Studio, disponible como una extensión independiente para Visual Studio 2017 Update 5 e incluida con el producto en Update 6 y versiones posteriores. Con la Update 6 o posterior, puede comprobar la instalación de la extensión de autenticación de la aplicación si selecciona las herramientas de desarrollo de Azure en el instalador de Visual Studio.

## <a name="using-the-library"></a>Uso de la biblioteca

Para aplicaciones. NET, la manera más sencilla de trabajar con una identidad administrada es con el paquete `Microsoft.Azure.Services.AppAuthentication`. Aquí tiene información sobre cómo empezar:

1. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Administrar paquetes NuGet para la solución** para agregar referencias a los paquetes NuGet [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) y [Microsoft.Azure.KeyVault](https://www.nuget.org/packages/Microsoft.Azure.KeyVault) para el proyecto.

1. Agregue el siguiente código:

    ``` csharp
    using Microsoft.Azure.Services.AppAuthentication;
    using Microsoft.Azure.KeyVault;

    // Instantiate a new KeyVaultClient object, with an access token to Key Vault
    var azureServiceTokenProvider1 = new AzureServiceTokenProvider();
    var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider1.KeyVaultTokenCallback));

    // Optional: Request an access token to other Azure services
    var azureServiceTokenProvider2 = new AzureServiceTokenProvider();
    string accessToken = await azureServiceTokenProvider2.GetAccessTokenAsync("https://management.azure.com/").ConfigureAwait(false);
    ```

No es preciso comprobar la expiración del token antes de llamar al método `GetAccessTokenAsync`, ya que `AzureServiceTokenProvider` almacena el token en la memoria caché y lo recupera de Azure AD inmediatamente antes de que expire. 

El método `GetAccessTokenAsync` requiere un identificador de recursos. Para obtener más información sobre los servicios de Microsoft Azure, consulte [¿Qué es Managed Identities for Azure Resources?](../../active-directory/msi-overview.md).

## <a name="local-development-authentication"></a>Autenticación de desarrollo local

Para el desarrollo local, hay dos escenarios de autenticación principales: la [autenticación en los servicios de Azure](#authenticating-to-azure-services) y la [autenticación en los servicios personalizados](#authenticating-to-custom-services).

### <a name="authenticating-to-azure-services"></a>Autenticación en los servicios de Azure

Los equipos locales no admiten identidades administradas para recursos de Azure. Como resultado, la biblioteca `Microsoft.Azure.Services.AppAuthentication` utiliza las credenciales de desarrollador para la ejecución en el entorno de desarrollo local. Cuando la solución se implementa en Azure, la biblioteca usa una identidad administrada para cambiar a un flujo de concesión de credenciales de cliente de OAuth 2.0. Este enfoque significa que puede probar el mismo código local y remotamente sin preocuparse.

Para el desarrollo local, `AzureServiceTokenProvider` captura tokens mediante **Visual Studio**, la **interfaz de línea de comandos de Azure** (CLI) o **la autenticación integrada de Azure AD**. Cada opción se prueba secuencialmente y la biblioteca usa la primera opción correcta. Si no funciona ninguna opción, se produce una excepción `AzureServiceTokenProviderException` con información detallada.

#### <a name="authenticating-with-visual-studio"></a>Autenticación con Visual Studio

Para autenticar mediante Visual Studio:

1. Inicie sesión en Visual Studio y use **Herramientas**&nbsp;>&nbsp;**Opciones** para abrir **Opciones**.

1. Seleccione **Autenticación de servicio de Azure**, elija una cuenta para desarrollo local y seleccione **Aceptar**.

Si experimenta problemas con Visual Studio, como errores relacionados con el archivo del proveedor de tokens, revise atentamente los pasos anteriores.

Puede que sea necesario volver a autenticar el token del desarrollador. Para ello, seleccione **Herramientas**&nbsp;>&nbsp;**Opciones** y, a continuación, seleccione **Autenticación&nbsp;de servicio&nbsp;de Azure**. Busque un vínculo para **volver a autenticar** en la cuenta seleccionada. Selecciónelo para autenticar.

#### <a name="authenticating-with-azure-cli"></a>Autenticación con la CLI de Azure

Para usar la CLI de Azure para el desarrollo local, asegúrese de que tiene la versión [CLI de Azure v2.0.12](/cli/azure/install-azure-cli) o posterior.

Para usar la CLI de Azure:

1. Busque la CLI de Azure en la barra de tareas de Windows para abrir el **símbolo del sistema de Microsoft Azure**.

1. Inicie sesión en Azure Portal: *az login* para iniciar sesión en Azure.

1. Escriba *az account get-access-token --resource https:\//vault.azure.net* para verificar el acceso. Si recibe un error, compruebe que se ha instalado correctamente la versión adecuada de la CLI de Azure.

   Si la CLI de Azure no está instalada en el directorio predeterminado, recibirá un error que le notificará que `AzureServiceTokenProvider` no encuentra la ruta de acceso para la CLI de Azure. Use la variable de entorno **AzureCLIPath** para definir la carpeta de instalación de la CLI de Azure. `AzureServiceTokenProvider` agrega el directorio especificado en la variable de entorno **AzureCLIPath** a la variable de entorno **Path** cuando sea necesario.

1. Si ha iniciado la sesión en la CLI de Azure con varias cuentas o la cuenta tiene acceso a varias suscripciones, debe especificar la suscripción que se va a usar. Escriba el comando *az account set --subscription <subscription-id>* .

Este comando genera un resultado solo en caso de error. Para comprobar la configuración de la cuenta actual, escriba el comando `az account list`.

#### <a name="authenticating-with-azure-ad-authentication"></a>Autenticación con la autenticación de Azure AD

Para usar la autenticación de Azure AD, compruebe lo siguiente:

- La instancia de Active Directory local se sincroniza con Azure AD. Para más información, consulte [¿Qué es la identidad híbrida con Azure Active Directory?](../../active-directory/connect/active-directory-aadconnect.md).

- El código se ejecuta en un equipo unido a un dominio.

### <a name="authenticating-to-custom-services"></a>Autenticación en los servicios personalizados

Cuando un servicio llama a los servicios de Azure, los pasos anteriores funcionan porque los servicios de Azure permiten el acceso a los usuarios y a las aplicaciones.

Al crear un servicio que llama a un servicio personalizado, use las credenciales de cliente de Azure AD para la autenticación de desarrollo local. Hay dos opciones:

- Usar a una entidad de servicio para iniciar sesión en Azure:

    1. Crear una entidad de servicio. Para más información, vea [Creación de una entidad de servicio de Azure con la CLI de Azure](/cli/azure/create-an-azure-service-principal-azure-cli).

    1. Use la CLI de Azure para iniciar sesión con el siguiente comando:

        ```azurecli
        az login --service-principal -u <principal-id> --password <password> --tenant <tenant-id> --allow-no-subscriptions
        ```

        Puesto que la entidad de servicio puede no tener acceso a la suscripción, use el argumento `--allow-no-subscriptions`.

- Usar variables de entorno para especificar los detalles de la entidad de servicio. Para más información, consulte [Ejecución de la aplicación con una entidad de servicio](#running-the-application-using-a-service-principal).

Una vez que haya iniciado sesión en Azure, `AzureServiceTokenProvider` usa la entidad de servicio para recuperar un token para el desarrollo local.

Este enfoque se aplica solo a desarrollo local. Cuando la solución se implementa en Azure, la biblioteca cambia a una identidad administrada para la autenticación.

## <a name="running-the-application-using-managed-identity-or-user-assigned-identity"></a>Ejecución de la aplicación con una identidad administrada o una identidad asignada por el usuario

Cuando se ejecuta el código en una instancia de Azure App Service o en una máquina virtual de Azure con una identidad administrada habilitada, la biblioteca usa automáticamente dicha identidad. No es necesario realizar cambios en el código, pero la identidad administrada debe tener permisos *GET* para el almacén de claves. Puede conceder a la identidad administrada permisos *GET* mediante las *directivas de acceso* del almacén de claves.

También puede autenticarse con una identidad asignada por el usuario. Para obtener más información sobre las identidades asignadas por el usuario, consulte el artículo sobre [las identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types). Para autenticarse con una identidad asignada por el usuario, debe especificar el identificador de cliente de la identidad asignada por el usuario en la cadena de conexión. La cadena de conexión se especifica en [Compatibilidad con la cadena de conexión](#connection-string-support).

## <a name="running-the-application-using-a-service-principal"></a>Ejecución de la aplicación con una entidad de servicio

Puede ser necesario crear una credencial de cliente de Azure AD para autenticar. Esta situación puede producirse en los ejemplos siguientes:

- El código se ejecuta en un entorno de desarrollo local, pero no en la identidad del desarrollador. Service Fabric, por ejemplo, usa la [cuenta de NetworkService](../../service-fabric/service-fabric-application-secret-management.md) para el desarrollo local.

- El código se ejecuta en un entorno de desarrollo local y se autentica en un servicio personalizado, por lo que no puede usar la identidad de desarrollador.

- Se ejecuta el código en un recurso de Azure Compute que aún no admite identidades administradas para recursos de Azure, como Azure Batch.

Hay tres métodos principales para usar una entidad de servicio para ejecutar la aplicación. Para usar cualquiera de ellos, primero debe crear una entidad de servicio. Para más información, vea [Creación de una entidad de servicio de Azure con la CLI de Azure](/cli/azure/create-an-azure-service-principal-azure-cli).

### <a name="use-a-certificate-in-local-keystore-to-sign-into-azure-ad"></a>Uso de un certificado en el almacén de claves local para iniciar sesión en Azure AD

1. Cree un certificado de entidad de servicio mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) de la CLI de Azure.

    ```azurecli
    az ad sp create-for-rbac --create-cert
    ```

    Este comando un archivo .pem (clave privada) que se almacena en el directorio particular. Implemente este certificado en los almacenes *LocalMachine* o *CurrentUser*.

    > [!Important]
    > El comando de la CLI genera un archivo .pem, pero Windows solo proporciona compatibilidad nativa con los certificados PFX. Para generar un certificado PFX en su lugar, use los comandos de PowerShell que se muestran aquí: [Creación de una entidad de servicio con un certificado autofirmado](../../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-self-signed-certificate). Estos comandos también implementan el certificado automáticamente.

1. Establezca una variable de entorno denominada **AzureServicesAuthConnectionString** para el siguiente valor:

    ```azurecli
    RunAs=App;AppId={AppId};TenantId={TenantId};CertificateThumbprint={Thumbprint};
          CertificateStoreLocation={CertificateStore}
    ```

    Reemplace *{AppId}* , *{TenantId}* y *{Thumbprint}* por los valores generados en el paso 1. Reemplace *{CertificateStore}* por *LocalMachine*` o *CurrentUser*, según su plan de implementación.

1. Ejecute la aplicación.

### <a name="use-a-shared-secret-credential-to-sign-into-azure-ad"></a>Uso de una credencial compartida secreta para iniciar sesión en Azure AD

1. Cree un certificado de entidad de servicio con una contraseña mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) de la CLI de Azure con el parámetro --sdk-auth.

    ```azurecli
    az ad sp create-for-rbac --sdk-auth
    ```

1. Establezca una variable de entorno denominada **AzureServicesAuthConnectionString** para el siguiente valor:

    ```azurecli
    RunAs=App;AppId={AppId};TenantId={TenantId};AppKey={ClientSecret}
    ```

    Reemplace _{AppId}_ , _{TenantId}_ y _{ClientSecret}_ por los valores generados en el paso 1.

1. Ejecute la aplicación.

Una vez que todo está configurado correctamente, no es necesario realizar más cambios de código. `AzureServiceTokenProvider` usa la variable de entorno y el certificado para autenticar en Azure AD.

### <a name="use-a-certificate-in-key-vault-to-sign-into-azure-ad"></a>Uso de un certificado en Key Vault para iniciar sesión en Azure AD

Esta opción le permite almacenar el certificado de cliente de una entidad de servicio en Key Vault y usarlo para la autenticación de la entidad de servicio. Puede usar esta opción en los escenarios siguientes:

- Autenticación local, en la que quiere autenticarse mediante una entidad de servicio explícita y quiere guardar la credencial de la entidad de servicio de forma segura en un almacén de claves. La cuenta de desarrollador debe tener acceso al almacén de claves.

- Autenticación desde Azure, donde quiere usar la credencial explícita y quiere guardar la credencial de la entidad de servicio de forma segura en un almacén de claves. Puede usar esta opción para un escenario entre inquilinos. La identidad administrada debe tener acceso al almacén de claves.

La identidad administrada o la identidad del desarrollador deben tener permiso para recuperar el certificado de cliente de Key Vault. La biblioteca AppAuthentication usa el certificado recuperado como credencial de cliente de la entidad de servicio.

Para usar un certificado de cliente para la autenticación de la entidad de servicio:

1. Cree un certificado de entidad de servicio y almacénelo automáticamente en su Key Vault. Use el comando [az ad sp create-for-rbac --keyvault \<keyvaultname> --cert \<certificatename> --create-cert --skip-assignment](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) de la CLI de Azure:

    ```azurecli
    az ad sp create-for-rbac --keyvault <keyvaultname> --cert <certificatename> --create-cert --skip-assignment
    ```

    El identificador del certificado será una URL con el formato `https://<keyvaultname>.vault.azure.net/secrets/<certificatename>`

1. Reemplace `{KeyVaultCertificateSecretIdentifier}` en esta cadena de conexión por el identificador del certificado:

    ```azurecli
    RunAs=App;AppId={TestAppId};KeyVaultCertificateSecretIdentifier={KeyVaultCertificateSecretIdentifier}
    ```

    Por ejemplo, si el almacén de claves se llamaba *myKeyVault* y ha creado un certificado denominado *myCert*, el identificador del certificado sería:

    ```azurecli
    RunAs=App;AppId={TestAppId};KeyVaultCertificateSecretIdentifier=https://myKeyVault.vault.azure.net/secrets/myCert
    ```

## <a name="connection-string-support"></a>Compatibilidad de la cadena de conexión

De forma predeterminada, `AzureServiceTokenProvider` intenta los siguientes métodos de autenticación, en orden, para recuperar un token:

- [Una identidad administrada para recursos de Azure](../..//active-directory/managed-identities-azure-resources/overview.md)
- Autenticación de Visual Studio
- [Autenticación de la CLI de Azure](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest)
- [Autenticación integrada de Windows](/aspnet/web-api/overview/security/integrated-windows-authentication)

Para controlar el proceso, utilice una cadena de conexión pasada al constructor `AzureServiceTokenProvider` o especificada en la variable de entorno *AzureServicesAuthConnectionString*.  Se admiten las siguientes opciones:

| Opción de la cadena de conexión | Escenario | Comentarios|
|:--------------------------------|:------------------------|:----------------------------|
| `RunAs=Developer; DeveloperTool=AzureCli` | Desarrollo local | `AzureServiceTokenProvider` usa AzureCli para obtener el token. |
| `RunAs=Developer; DeveloperTool=VisualStudio` | Desarrollo local | `AzureServiceTokenProvider` usa Visual Studio para obtener el token. |
| `RunAs=CurrentUser` | Desarrollo local | No se admite en .NET Core. `AzureServiceTokenProvider` usa la autenticación integrada de Azure AD para obtener el token. |
| `RunAs=App` | [Identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/index.yml) | `AzureServiceTokenProvider` usa una identidad administrada para obtener el token. |
| `RunAs=App;AppId={ClientId of user-assigned identity}` | [Identidad asignada por el usuario para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) | `AzureServiceTokenProvider` usa una identidad asignada por el usuario para obtener el token. |
| `RunAs=App;AppId={TestAppId};KeyVaultCertificateSecretIdentifier={KeyVaultCertificateSecretIdentifier}` | Autenticación en servicios personalizados | `KeyVaultCertificateSecretIdentifier` es el identificador secreto del certificado. |
| `RunAs=App;AppId={AppId};TenantId={TenantId};CertificateThumbprint={Thumbprint};CertificateStoreLocation={LocalMachine or CurrentUser}`| Entidad de servicio | `AzureServiceTokenProvider` usa el certificado para obtener el token desde Azure AD. |
| `RunAs=App;AppId={AppId};TenantId={TenantId};CertificateSubjectName={Subject};CertificateStoreLocation={LocalMachine or CurrentUser}` | Entidad de servicio | `AzureServiceTokenProvider` usa el certificado para obtener el token desde Azure AD.|
| `RunAs=App;AppId={AppId};TenantId={TenantId};AppKey={ClientSecret}` | Entidad de servicio |`AzureServiceTokenProvider` usa el secreto para obtener el token desde Azure AD. |

## <a name="samples"></a>Ejemplos

Para ver la biblioteca `Microsoft.Azure.Services.AppAuthentication` en acción, consulte los siguientes ejemplos de código.

- [Use una identidad administrada para recuperar un secreto de almacén de Azure Key Vault en tiempo de ejecución](https://github.com/Azure-Samples/app-service-msi-keyvault-dotnet).

- [Implemente mediante programación una plantilla de Azure Resource Manager desde una máquina virtual de Azure con una identidad administrada](https://github.com/Azure-Samples/windowsvm-msi-arm-dotnet).

- [Use el ejemplo .NET Core y una identidad administrada para llamar a los servicios de Azure desde una máquina virtual Linux de Azure](https://github.com/Azure-Samples/linuxvm-msi-keyvault-arm-dotnet/).

## <a name="appauthentication-troubleshooting"></a>Solución de problemas de AppAuthentication

### <a name="common-issues-during-local-development"></a>Problemas comunes durante el desarrollo local

#### <a name="azure-cli-is-not-installed-youre-not-logged-in-or-you-dont-have-the-latest-version"></a>La CLI de Azure no está instalada, no se ha iniciado sesión o no se cuenta con la versión más reciente

Ejecute *az account get-access-token* para ver si la CLI de Azure muestra un token automáticamente. Si indica que **no se encontró dicho programa**, instale la [versión más reciente de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest). Es posible que se le pida que inicie sesión.

#### <a name="azureservicetokenprovider-cant-find-the-path-for-azure-cli"></a>AzureServiceTokenProvider no puede encontrar la ruta de acceso de la CLI de Azure

AzureServiceTokenProvider busca la CLI de Azure en sus ubicaciones de instalación predeterminadas. Si no se puede encontrar la CLI de Azure, establezca la variable de entorno **AzureCLIPath** en la carpeta de instalación de la CLI de Azure. AzureServiceTokenProvider agregará la variable de entorno a la variable de entorno Path.

#### <a name="youre-logged-into-azure-cli-using-multiple-accounts-the-same-account-has-access-to-subscriptions-in-multiple-tenants-or-you-get-an-access-denied-error-when-trying-to-make-calls-during-local-development"></a>Ha iniciado sesión en la CLI de Azure con varias cuentas, la misma cuenta tiene acceso a las suscripciones de varios inquilinos o se recibe un error de acceso denegado al intentar realizar llamadas durante el desarrollo local

Con la CLI de Azure, establezca la suscripción predeterminada en una que tenga la cuenta que quiere usar. La suscripción debe estar en el mismo inquilino que el recurso al que quiere acceder: **az account set --subscription [subscription-id]** . Si no se muestra ninguna salida, significa que se ha realizado correctamente. Compruebe que la cuenta correcta es ahora el valor predeterminado mediante **az account list**.

### <a name="common-issues-across-environments"></a>Problemas comunes en los entornos

#### <a name="unauthorized-access-access-denied-forbidden-or-similar-error"></a>Acceso no autorizado, acceso denegado, prohibido o un error similar

La entidad de seguridad utilizada no tiene acceso al recurso al que intenta acceder. Conceda a la cuenta de usuario o al "colaborador" de MSI de App Service acceso a un recurso. La decisión depende de si está ejecutando el ejemplo en el equipo local o implementado en Azure en App Service. Algunos recursos, como los almacenes de claves, también tienen sus propias [directivas de acceso](https://docs.microsoft.com/azure/key-vault/general/secure-your-key-vault#data-plane-and-access-policies), que se usan para conceder acceso a las entidades de seguridad, como usuarios, aplicaciones y grupos.

### <a name="common-issues-when-deployed-to-azure-app-service"></a>Problemas comunes al realizar la implementación en Azure App Service

#### <a name="managed-identity-isnt-set-up-on-the-app-service"></a>La identidad administrada no se configura en App Service.

Compruebe si las variables de entorno MSI_ENDPOINT y MSI_SECRET existen mediante [Kudu debug console](https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/). Si estas variables de entorno no existen, la identidad administrada no está habilitada en App Service.

### <a name="common-issues-when-deployed-locally-with-iis"></a>Problemas comunes al realizar la implementación localmente con IIS

#### <a name="cant-retrieve-tokens-when-debugging-app-in-iis"></a>No se pueden recuperar los tokens al depurar la aplicación en IIS.

De manera predeterminada, AppAuth se ejecuta en un contexto de usuario diferente en IIS. Este es el motivo por el que no tiene acceso para usar la identidad de desarrollador para recuperar los tokens de acceso. Puede configurar IIS para que se ejecute con su contexto de usuario con los dos pasos siguientes:
- Configure el grupo de aplicaciones para que la aplicación web se ejecute como su cuenta de usuario actual. Puede obtener más información [aquí](https://docs.microsoft.com/iis/manage/configuring-security/application-pool-identities#configuring-iis-application-pool-identities)
- Configure "setProfileEnvironment" en "True". Puede obtener más información [aquí](https://docs.microsoft.com/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration). 

    - Vaya a %windir%\System32\inetsrv\config\applicationHost.config.
    - Busque "setProfileEnvironment". Si está establecido en "False", cámbielo a "True". Si no está presente, agréguelo como atributo al elemento processModel (/configuration/system.applicationHost/applicationPools/applicationPoolDefaults/processModel/@setProfileEnvironment) y establézcalo en "True".

- Obtenga más información sobre las [identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/index.yml).
- Más información sobre los [escenarios de autenticación de Azure AD](../../active-directory/develop/active-directory-authentication-scenarios.md).
