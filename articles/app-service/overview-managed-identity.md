---
title: Identidades administradas
description: Obtenga información sobre cómo funcionan las identidades administradas en Azure App Service y Azure Functions, cómo configurar una identidad administrada y generar un token para un recurso de back-end.
author: mattchenderson
ms.topic: article
ms.date: 05/27/2020
ms.author: mahender
ms.reviewer: yevbronsh
ms.custom: devx-track-csharp, devx-track-python, devx-track-azurepowershell
ms.openlocfilehash: 98cec9fc30b4840a763358c0b3cd76659ff865d7
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89070367"
---
# <a name="how-to-use-managed-identities-for-app-service-and-azure-functions"></a>Cómo usar identidades administradas para App Service y Azure Functions

En este tema se muestra cómo crear una identidad administrada para las aplicaciones App Service y Azure Functions y cómo usarla para acceder a otros recursos. 

> [!Important] 
> Las identidades administradas de App Service y Azure Functions no se comportarán según lo esperado si la aplicación se migra entre suscripciones e inquilinos. La aplicación tiene que obtener una nueva identidad, para lo cual se deshabilita y vuelve a habilitar la característica. Consulte [Eliminación de una identidad](#remove) a continuación. Los recursos de nivel inferior también han de tener directivas de acceso actualizadas para utilizar la nueva identidad.

[!INCLUDE [app-service-managed-identities](../../includes/app-service-managed-identities.md)]

## <a name="add-a-system-assigned-identity"></a>Adición de una identidad asignada por el sistema

Para crear una aplicación con una identidad asignada por el sistema se requiere la configuración de una propiedad adicional en la aplicación.

### <a name="using-the-azure-portal"></a>Uso de Azure Portal

Para configurar una identidad administrada en el portal, primero creará una aplicación como lo hace normalmente y, a continuación, habilitará la característica.

1. Cree una aplicación en el portal como lo haría normalmente. Navegue hasta el portal.

2. Si utiliza una aplicación de función, vaya a **Características de la plataforma**. Para otros tipos de aplicación, desplácese hacia abajo hasta el grupo **Configuración** en el panel de navegación izquierdo.

3. Seleccione **Identidad**.

4. En la pestaña **Asignado por el sistema**, cambie **Estado** a **Activado**. Haga clic en **Save**(Guardar).

    ![Identidad administrada en App Service](media/app-service-managed-service-identity/system-assigned-managed-identity-in-azure-portal.png)


> [!NOTE] 
> Para encontrar la identidad administrada de la aplicación web o de ranura en Azure Portal, busque en la sección **Configuración de usuario** en **Aplicaciones empresariales**. Normalmente, el nombre de la ranura es similar a `<app name>/slots/<slot name>`.


### <a name="using-the-azure-cli"></a>Uso de la CLI de Azure

Para configurar una identidad administrada mediante la CLI de Azure, deberá usar el comando `az webapp identity assign` en una aplicación existente. Tiene tres opciones para ejecutar los ejemplos de esta sección:

- Usar [Azure Cloud Shell](../cloud-shell/overview.md) desde Azure Portal.
- Use Azure Cloud Shell integrado mediante el botón "Pruébelo", situado en la esquina superior derecha de cada bloque de código.
- [Instale la versión más reciente de la CLI de Azure](/cli/azure/install-azure-cli) (2.0.31 o posterior) si prefiere usar una consola de CLI local. 

Los siguientes pasos le guiarán por la creación de una aplicación web y la asignación a la misma de una identidad mediante la CLI:

1. Si usa la CLI de Azure en una consola local, lo primero que debe hacer es iniciar sesión en Azure mediante el [inicio de sesión de az](/cli/azure/reference-index#az-login). Use una cuenta asociada a la suscripción de Azure en la que desea implementar la aplicación:

    ```azurecli-interactive
    az login
    ```

2. Cree una aplicación web mediante la CLI. Para más ejemplos de cómo utilizar la CLI con App Service, consulte los [ejemplos de la CLI de App Service](../app-service/samples-cli.md):

    ```azurecli-interactive
    az group create --name myResourceGroup --location westus
    az appservice plan create --name myPlan --resource-group myResourceGroup --sku S1
    az webapp create --name myApp --resource-group myResourceGroup --plan myPlan
    ```

3. Ejecute el comando `identity assign` para crear la identidad de esta aplicación:

    ```azurecli-interactive
    az webapp identity assign --name myApp --resource-group myResourceGroup
    ```

### <a name="using-azure-powershell"></a>Uso de Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Los siguientes pasos le guiarán por la creación de una aplicación y la asignación de una identidad a esta mediante Azure PowerShell: Las instrucciones para crear una aplicación web y una aplicación de funciones son diferentes.

#### <a name="using-azure-powershell-for-a-web-app"></a>Uso de Azure PowerShell para una aplicación web

1. Si es necesario, instale Azure PowerShell con las instrucciones que se encuentra en la [guía de Azure PowerShell](/powershell/azure/) y luego ejecute `Login-AzAccount` para crear una conexión con Azure.

2. Cree una aplicación web mediante Azure PowerShell. Para ver más ejemplos de cómo utilizar Azure PowerShell con App Service, consulte los [ejemplos de PowerShell de App Service](../app-service/samples-powershell.md):

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create an App Service plan in Free tier.
    New-AzAppServicePlan -Name $webappname -Location $location -ResourceGroupName $resourceGroupName -Tier Free

    # Create a web app.
    New-AzWebApp -Name $webappname -Location $location -AppServicePlan $webappname -ResourceGroupName $resourceGroupName
    ```

3. Ejecute el comando `Set-AzWebApp -AssignIdentity` para crear la identidad de esta aplicación:

    ```azurepowershell-interactive
    Set-AzWebApp -AssignIdentity $true -Name $webappname -ResourceGroupName $resourceGroupName 
    ```

#### <a name="using-azure-powershell-for-a-function-app"></a>Uso de Azure PowerShell para una aplicación de funciones

1. Si es necesario, instale Azure PowerShell con las instrucciones que se encuentra en la [guía de Azure PowerShell](/powershell/azure/) y luego ejecute `Login-AzAccount` para crear una conexión con Azure.

2. Cree una aplicación de funciones con Azure PowerShell. Para obtener más ejemplos de cómo usar Azure PowerShell con Azure Functions, vea la [referencia de Az.Functions](/powershell/module/az.functions/?view=azps-4.1.0#functions):

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create a storage account.
    New-AzStorageAccount -Name $storageAccountName -ResourceGroupName $resourceGroupName -SkuName $sku

    # Create a function app with a system-assigned identity.
    New-AzFunctionApp -Name $functionAppName -ResourceGroupName $resourceGroupName -Location $location -StorageAccountName $storageAccountName -Runtime $runtime -IdentityType SystemAssigned
    ```

También puede actualizar una aplicación de funciones existente mediante `Update-AzFunctionApp` en su lugar.

### <a name="using-an-azure-resource-manager-template"></a>Uso de una plantilla de Azure Resource Manager

Se puede utilizar una plantilla de Azure Resource Manager para automatizar la implementación de los recursos de Azure. Para obtener más información acerca de la implementación en App Service y Functions, vea [Automating resource deployment in App Service](../app-service/deploy-complex-application-predictably.md) (Automatización de la implementación de recursos en App Service) y [Automatización de la implementación de recursos para una aplicación de función en Azure Functions](../azure-functions/functions-infrastructure-as-code.md).

Se puede crear cualquier recurso de tipo `Microsoft.Web/sites` con una identidad mediante la inclusión de la siguiente propiedad en la definición de recursos:

```json
"identity": {
    "type": "SystemAssigned"
}
```

> [!NOTE]
> Una aplicación puede tener identidades asignadas por el sistema y asignadas por el usuario al mismo tiempo. En este caso, la propiedad `type` sería `SystemAssigned,UserAssigned`.

Al agregar el tipo asignado por el sistema se indica a Azure que debe crear y administrar la identidad para la aplicación.

Por ejemplo, una aplicación web podría tener el aspecto siguiente:

```json
{
    "apiVersion": "2016-08-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('appName')]",
    "location": "[resourceGroup().location]",
    "identity": {
        "type": "SystemAssigned"
    },
    "properties": {
        "name": "[variables('appName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "hostingEnvironment": "",
        "clientAffinityEnabled": false,
        "alwaysOn": true
    },
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]"
    ]
}
```

Cuando se crea el sitio, tiene las siguientes propiedades adicionales:

```json
"identity": {
    "type": "SystemAssigned",
    "tenantId": "<TENANTID>",
    "principalId": "<PRINCIPALID>"
}
```

La propiedad tenantId identifica a qué inquilino de Azure AD pertenece la identidad. El valor principalId es un identificador único para la nueva identidad de la aplicación. En Azure AD, la entidad de servicio tiene el mismo nombre que asignó a la instancia de App Service o Azure Functions.

Si posteriormente tiene que hacer referencia a estas propiedades en la plantilla, puede hacerlo mediante la [función de plantilla `reference()`](../azure-resource-manager/templates/template-functions-resource.md#reference) con la marca `'Full'`, tal y como se muestra en este ejemplo:

```json
{
    "tenantId": "[reference(resourceId('Microsoft.Web/sites', variables('appName')), '2018-02-01', 'Full').identity.tenantId]",
    "objectId": "[reference(resourceId('Microsoft.Web/sites', variables('appName')), '2018-02-01', 'Full').identity.principalId]",
}
```

## <a name="add-a-user-assigned-identity"></a>Adición de una identidad asignada por el usuario

La creación de una aplicación con una identidad asignada por el usuario requiere que se cree la identidad y luego se agregue su identificador de recurso a la configuración de la aplicación.

### <a name="using-the-azure-portal"></a>Uso de Azure Portal

En primer lugar, tendrá que crear un recurso de identidad asignada por el usuario.

1. Cree un recurso de identidad administrada asignada por el usuario según [estas instrucciones](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity).

2. Cree una aplicación en el portal como lo haría normalmente. Navegue hasta el portal.

3. Si utiliza una aplicación de función, vaya a **Características de la plataforma**. Para otros tipos de aplicación, desplácese hacia abajo hasta el grupo **Configuración** en el panel de navegación izquierdo.

4. Seleccione **Identidad**.

5. En la pestaña **Usuario asignado**, haga clic **Agregar**.

6. Busque la identidad que creó anteriormente y selecciónela. Haga clic en **Agregar**.

    ![Identidad administrada en App Service](media/app-service-managed-service-identity/user-assigned-managed-identity-in-azure-portal.png)

### <a name="using-azure-powershell"></a>Uso de Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Los siguientes pasos le guiarán por la creación de una aplicación y la asignación de una identidad a esta mediante Azure PowerShell:

> [!NOTE]
> La versión actual de los commandlets de Azure PowerShell para Azure App Service no admite identidades asignadas por el usuario. Las instrucciones siguientes son para Azure Functions.

1. Si es necesario, instale Azure PowerShell con las instrucciones que se encuentra en la [guía de Azure PowerShell](/powershell/azure/) y luego ejecute `Login-AzAccount` para crear una conexión con Azure.

2. Cree una aplicación de funciones con Azure PowerShell. Para obtener más ejemplos de cómo usar Azure PowerShell con Azure Functions, vea la [referencia de Az.Functions](/powershell/module/az.functions/?view=azps-4.1.0#functions). El siguiente script también utiliza `New-AzUserAssignedIdentity`, que se debe instalar por separado según el procedimiento de [Crear, enumerar y eliminar una identidad administrada que haya asignado el usuario mediante Azure PowerShell](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md).

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create a storage account.
    New-AzStorageAccount -Name $storageAccountName -ResourceGroupName $resourceGroupName -SkuName $sku

    # Create a user-assigned identity. This requires installation of the "Az.ManagedServiceIdentity" module.
    $userAssignedIdentity = New-AzUserAssignedIdentity -Name $userAssignedIdentityName -ResourceGroupName $resourceGroupName

    # Create a function app with a user-assigned identity.
    New-AzFunctionApp -Name $functionAppName -ResourceGroupName $resourceGroupName -Location $location -StorageAccountName $storageAccountName -Runtime $runtime -IdentityType UserAssigned -IdentityId $userAssignedIdentity.Id
    ```

También puede actualizar una aplicación de funciones existente mediante `Update-AzFunctionApp` en su lugar.

### <a name="using-an-azure-resource-manager-template"></a>Uso de una plantilla de Azure Resource Manager

Se puede utilizar una plantilla de Azure Resource Manager para automatizar la implementación de los recursos de Azure. Para obtener más información acerca de la implementación en App Service y Functions, vea [Automating resource deployment in App Service](../app-service/deploy-complex-application-predictably.md) (Automatización de la implementación de recursos en App Service) y [Automatización de la implementación de recursos para una aplicación de función en Azure Functions](../azure-functions/functions-infrastructure-as-code.md).

Se puede crear cualquier recurso de tipo `Microsoft.Web/sites` con una identidad incluyendo el siguiente bloque en la definición del recurso, que reemplaza `<RESOURCEID>` por el identificador de recurso de la identidad deseada:

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {}
    }
}
```

> [!NOTE]
> Una aplicación puede tener identidades asignadas por el sistema y asignadas por el usuario al mismo tiempo. En este caso, la propiedad `type` sería `SystemAssigned,UserAssigned`.

Al agregar el tipo asignado por el usuario se indica a Azure que use la identidad asignada por el usuario especificada para la aplicación.

Por ejemplo, una aplicación web podría tener el aspecto siguiente:

```json
{
    "apiVersion": "2016-08-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('appName')]",
    "location": "[resourceGroup().location]",
    "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
            "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]": {}
        }
    },
    "properties": {
        "name": "[variables('appName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "hostingEnvironment": "",
        "clientAffinityEnabled": false,
        "alwaysOn": true
    },
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]"
    ]
}
```

Cuando se crea el sitio, tiene las siguientes propiedades adicionales:

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {
            "principalId": "<PRINCIPALID>",
            "clientId": "<CLIENTID>"
        }
    }
}
```

El valor principalId es un identificador único de la identidad que se usa para la administración de Azure AD. El valor clientId es un identificador único de la nueva identidad de la aplicación que se usa para especificar qué identidad utilizar durante las llamadas de runtime.

## <a name="obtain-tokens-for-azure-resources"></a>Obtención de tokens para recursos de Azure

Una aplicación puede utilizar su identidad administrada para obtener tokens de acceso a otros recursos protegidos por Azure AD, como Azure Key Vault. Estos tokens representan a la aplicación que accede al recurso, y no a un usuario específico de la aplicación. 

Es posible que tenga que configurar el recurso de destino para permitir el acceso desde la aplicación. Por ejemplo, si se solicita un token para acceder a Key Vault, debe asegurarse de que ha agregado una directiva de acceso que incluya la identidad de la aplicación. De lo contrario, las llamadas a Key Vault se rechazarán, incluso si incluyen el token. Para más información sobre los recursos que admiten tokens de Azure Active Directory, consulte [Servicios de Azure que admiten autenticación de Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

> [!IMPORTANT]
> Los servicios de back-end para identidades administradas mantienen una memoria caché por URI de recurso durante unas ocho horas. Si actualiza la directiva de acceso de un recurso de destino determinado y recupera inmediatamente un token para ese recurso, es posible que siga obteniendo un token en caché con permisos obsoletos hasta la expiración del token. Actualmente no existe ninguna forma de forzar la actualización de un token.

Hay un protocolo de REST sencillo para obtener un token en App Service y Azure Functions. Se puede usar con todas las aplicaciones y todos los lenguajes. En el caso de .NET y Java, el SDK de Azure ofrece una abstracción sobre este protocolo y facilita una experiencia de desarrollo local.

### <a name="using-the-rest-protocol"></a>Uso del protocolo de REST

> [!NOTE]
> Una versión anterior de este protocolo, con la versión de API "2017-09-01", utilizó el encabezado `secret` en lugar de `X-IDENTITY-HEADER` y solo aceptó la propiedad `clientid` para el valor asignado por el usuario. También devuelve el valor `expires_on` en un formato de marca de tiempo. Se puede usar MSI_ENDPOINT como alias para IDENTITY_ENDPOINT y MSI_SECRET como alias para IDENTITY_HEADER. Esta versión del protocolo es necesaria actualmente en los planes de hospedaje de Consumo para Linux.

Una aplicación con una identidad administrada tiene dos variables de entorno definidas:

- IDENTITY_ENDPOINT: dirección URL del servicio de token local.
- IDENTITY_HEADER: encabezado que se usa para ayudar a mitigar los ataques de falsificación de solicitudes del servidor (SSRF). La plataforma se encarga de cambiarlo.

La variable **IDENTITY_ENDPOINT** es una dirección URL local desde la que la aplicación puede solicitar tokens. Para obtener un token para un recurso, realice una solicitud HTTP GET para este punto de conexión, incluyendo los parámetros siguientes:

> | Nombre de parámetro    | En     | Descripción                                                                                                                                                                                                                                                                                                                                |
> |-------------------|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
> | resource          | Consultar  | El URI del recurso de Azure AD del recurso para el que se debe obtener un token. Este podría ser uno de los [servicios de Azure que admiten la autenticación de Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication) o cualquier otro URI de recurso.    |
> | api-version       | Consultar  | La versión de la API de token que se usará. Utilice la versión "2019-08-01" u otra posterior, a menos que use Consumo para Linux, que en la actualidad solo ofrece la versión "2017-09-01" (ver nota anterior).                                                                                                                                                                                                                                                                 |
> | X-IDENTITY-HEADER | Encabezado | El valor de la variable de entorno IDENTITY_HEADER. Este encabezado se utiliza para ayudar a mitigar los ataques de falsificación de solicitudes del servidor (SSRF).                                                                                                                                                                                                    |
> | client_id         | Consultar  | (Opcional) El identificador del cliente de la identidad asignada por el usuario que se va a usar. No se puede usar en una solicitud que incluya `principal_id`, `mi_res_id` o `object_id`. Si se omiten todos los parámetros de identificador (`client_id`, `principal_id`, `object_id` y `mi_res_id`), se usa la identidad asignada por el sistema.                                             |
> | principal_id      | Consultar  | (Opcional) El identificador de la entidad de seguridad de la identidad asignada por el usuario que se va a usar. `object_id` es un alias que se puede utilizar en su lugar. No se puede usar en una solicitud que incluya client_id, mi_res_id u object_id. Si se omiten todos los parámetros de identificador (`client_id`, `principal_id`, `object_id` y `mi_res_id`), se usa la identidad asignada por el sistema. |
> | mi_res_id         | Consultar  | (Opcional) El identificador del recurso de Azure de la identidad asignada por el usuario que se va a usar. No se puede usar en una solicitud que incluya `principal_id`, `client_id` o `object_id`. Si se omiten todos los parámetros de identificador (`client_id`, `principal_id`, `object_id` y `mi_res_id`), se usa la identidad asignada por el sistema.                                      |

> [!IMPORTANT]
> Si intenta obtener tokens para las identidades asignadas por el usuario, debe incluir una de las propiedades opcionales. De lo contrario, el servicio de token intentará obtener un token para una identidad que haya asignado el sistema, la cual puede existir o no.

Una respuesta 200 OK incluye un cuerpo JSON con las siguientes propiedades:

> | Nombre de propiedad | Descripción                                                                                                                                                                                                                                        |
> |---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
> | access_token  | El token de acceso solicitado. El servicio web de llamada puede usar este token para autenticarse en el servicio web de recepción.                                                                                                                               |
> | client_id     | El identificador de cliente de la identidad que se ha utilizado.                                                                                                                                                                                                       |
> | expires_on    | La hora a la que expira el token de acceso. La fecha se representa como el número de segundos desde "1970-01-01T0:0:0Z UTC" (se corresponde con la notificación `exp` del token).                                                                                |
> | not_before    | El intervalo de tiempo cuando el token de acceso tiene efecto y se puede aceptar. La fecha se representa como el número de segundos desde "1970-01-01T0:0:0Z UTC" (se corresponde con la notificación `nbf` del token).                                                      |
> | resource      | El recurso para el que se solicitó el token de acceso, que coincide con el parámetro de la cadena de consulta `resource` de la solicitud.                                                                                                                               |
> | token_type    | Indica el valor de tipo de token. El único tipo que admite Azure AD es FBearer. Para más información sobre los tokens de portador, consulte [The OAuth2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) (Marco de autorización de OAuth2.0: uso del token de portador [RFC 6750]). |

Esta respuesta es la misma que la [respuesta para la solicitud de token de acceso de servicio a servicio de Azure AD](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md#service-to-service-access-token-response).

### <a name="rest-protocol-examples"></a>Ejemplos de protocolo de REST

Una solicitud de ejemplo puede tener el siguiente aspecto:

```http
GET /MSI/token?resource=https://vault.azure.net&api-version=2019-08-01 HTTP/1.1
Host: localhost:4141
X-IDENTITY-HEADER: 853b9a84-5bfa-4b22-a3f3-0b9a43d9ad8a
```

Y una respuesta de ejemplo puede tener el siguiente aspecto:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "access_token": "eyJ0eXAi…",
    "expires_on": "1586984735",
    "resource": "https://vault.azure.net",
    "token_type": "Bearer",
    "client_id": "5E29463D-71DA-4FE0-8E69-999B57DB23B0"
}
```

### <a name="code-examples"></a>Ejemplos de código

# <a name="net"></a>[.NET](#tab/dotnet)

> [!TIP]
> Para los lenguajes. NET, también puede usar [Microsoft.Azure.Services.AppAuthentication](#asal) en lugar de crear esta solicitud personalmente.

```csharp
private readonly HttpClient _client;
// ...
public async Task<HttpResponseMessage> GetToken(string resource)  {
    var request = new HttpRequestMessage(HttpMethod.Get, 
        String.Format("{0}/?resource={1}&api-version=2019-08-01", Environment.GetEnvironmentVariable("IDENTITY_ENDPOINT"), resource));
    request.Headers.Add("X-IDENTITY-HEADER", Environment.GetEnvironmentVariable("IDENTITY_HEADER"));
    return await _client.SendAsync(request);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const rp = require('request-promise');
const getToken = function(resource, cb) {
    let options = {
        uri: `${process.env["IDENTITY_ENDPOINT"]}/?resource=${resource}&api-version=2019-08-01`,
        headers: {
            'X-IDENTITY-HEADER': process.env["IDENTITY_HEADER"]
        }
    };
    rp(options)
        .then(cb);
}
```

# <a name="python"></a>[Python](#tab/python)

```python
import os
import requests

identity_endpoint = os.environ["IDENTITY_ENDPOINT"]
identity_header = os.environ["IDENTITY_HEADER"]

def get_bearer_token(resource_uri):
    token_auth_uri = f"{identity_endpoint}?resource={resource_uri}&api-version=2019-08-01"
    head_msi = {'X-IDENTITY-HEADER':identity_header}

    resp = requests.get(token_auth_uri, headers=head_msi)
    access_token = resp.json()['access_token']

    return access_token
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
$resourceURI = "https://<AAD-resource-URI-for-resource-to-obtain-token>"
$tokenAuthURI = $env:IDENTITY_ENDPOINT + "?resource=$resourceURI&api-version=2019-08-01"
$tokenResponse = Invoke-RestMethod -Method Get -Headers @{"X-IDENTITY-HEADER"="$env:IDENTITY_HEADER"} -Uri $tokenAuthURI
$accessToken = $tokenResponse.access_token
```

---

### <a name="using-the-microsoftazureservicesappauthentication-library-for-net"></a><a name="asal"></a>Uso de la biblioteca Microsoft.Azure.Services.AppAuthentication para .NET

En el caso de aplicaciones y funciones de .NET, la manera más sencilla de trabajar con una identidad administrada es usar el paquete Microsoft.Azure.Services.AppAuthentication. Esta biblioteca también le permite probar el código localmente en la máquina de desarrollo, con su cuenta de usuario de Visual Studio, la [CLI de Azure](/cli/azure) o la autenticación integrada de Active Directory. Cuando se hospeda en la nube, de forma predeterminada, se utilizará una identidad asignada por el sistema. Sin embargo, este comportamiento puede personalizarse utilizando una variable de entorno de cadena de conexión que haga referencia al identificador de cliente de una identidad asignada por el usuario. Para más información sobre las opciones de desarrollo con esta biblioteca, consulte la [referencia de Microsoft.Azure.Services.AppAuthentication]. En esta sección se muestra cómo empezar a usar la biblioteca en su código.

1. Agregue referencias a los paquetes de NuGet [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) y a otros paquetes necesarios para la aplicación. El ejemplo siguiente también usa [Microsoft.Azure.KeyVault](https://www.nuget.org/packages/Microsoft.Azure.KeyVault).

2. Agregue el código siguiente a la aplicación, modificándolo para que tenga como destino el recurso correcto. En este ejemplo se muestran dos formas de trabajar con Azure Key Vault:

    ```csharp
    using Microsoft.Azure.Services.AppAuthentication;
    using Microsoft.Azure.KeyVault;
    // ...
    var azureServiceTokenProvider = new AzureServiceTokenProvider();
    string accessToken = await azureServiceTokenProvider.GetAccessTokenAsync("https://vault.azure.net");
    // OR
    var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));
    ```

Si desea utilizar una identidad administrada asignada por el usuario, puede establecer la opción `AzureServicesAuthConnectionString` de la aplicación en `RunAs=App;AppId=<clientId-guid>`. Sustituya `<clientId-guid>` por el identificador de cliente de la identidad que desee usar. Puede definir varias cadenas de conexión de este tipo utilizando valores personalizados en la configuración de la aplicación y pasando estos valores al constructor AzureServiceTokenProvider.

```csharp
    var identityConnectionString1 = Environment.GetEnvironmentVariable("UA1_ConnectionString");
    var azureServiceTokenProvider1 = new AzureServiceTokenProvider(identityConnectionString1);
    
    var identityConnectionString2 = Environment.GetEnvironmentVariable("UA2_ConnectionString");
    var azureServiceTokenProvider2 = new AzureServiceTokenProvider(identityConnectionString2);
```

Para más información sobre la configuración de AzureServiceTokenProvider y las operaciones que expone, consulte la [referencia de Microsoft.Azure.Services.AppAuthentication] y el [ejemplo de App Service y KeyVault con MSI .NET](https://github.com/Azure-Samples/app-service-msi-keyvault-dotnet).

### <a name="using-the-azure-sdk-for-java"></a>Uso del SDK de Azure para Java

Para aplicaciones y funciones de Java, la manera más sencilla de trabajar con una identidad administrada es con el [SDK de Azure para Java](https://github.com/Azure/azure-sdk-for-java). En esta sección se muestra cómo empezar a usar la biblioteca en su código.

1. Agregue una referencia a la [biblioteca de Azure SDK](https://mvnrepository.com/artifact/com.microsoft.azure/azure). Para los proyectos de Maven, puede agregar este fragmento de código a la sección `dependencies` del archivo POM del proyecto:

    ```xml
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure</artifactId>
        <version>1.23.0</version>
    </dependency>
    ```

2. Use el objeto `AppServiceMSICredentials` para la autenticación. En este ejemplo se muestra cómo se puede utilizar este mecanismo para trabajar con Azure Key Vault:

    ```java
    import com.microsoft.azure.AzureEnvironment;
    import com.microsoft.azure.management.Azure;
    import com.microsoft.azure.management.keyvault.Vault
    //...
    Azure azure = Azure.authenticate(new AppServiceMSICredentials(AzureEnvironment.AZURE))
            .withSubscription(subscriptionId);
    Vault myKeyVault = azure.vaults().getByResourceGroup(resourceGroup, keyvaultName);

    ```


## <a name="remove-an-identity"></a><a name="remove"></a>Eliminación de una identidad

Para quitar una identidad asignada por el sistema, deshabilite la función mediante el portal, PowerShell o la CLI de la misma forma en que se creó. Las identidades asignadas por el usuario se pueden quitar individualmente. Para quitar todas las identidades, establezca el tipo de identidad en "Ninguno".

Al quitar una identidad asignada por el sistema de esta manera también se eliminará de Azure AD. Las identidades asignadas por el sistema también se quitan automáticamente de Azure AD cuando se elimina el recurso de la aplicación.

Para quitar todas las identidades de una [plantilla de ARM](#using-an-azure-resource-manager-template):

```json
"identity": {
    "type": "None"
}
```

Para quitar todas las identidades de Azure PowerShell (solo Azure Functions):

```azurepowershell-interactive
# Update an existing function app to have IdentityType "None".
Update-AzFunctionApp -Name $functionAppName -ResourceGroupName $resourceGroupName -IdentityType None
```

> [!NOTE]
> También hay una configuración de aplicación que se puede establecer, WEBSITE_DISABLE_MSI, que simplemente deshabilita el servicio de token local. Sin embargo, deja la identidad en su lugar, y las herramientas seguirán mostrando la identidad administrada como "activada" o "habilitada". Como resultado, no se recomienda el uso de esta configuración.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Acceso seguro a SQL Database mediante una identidad administrada](app-service-web-tutorial-connect-msi.md)

[Referencia de Microsoft.Azure.Services.AppAuthentication]: https://go.microsoft.com/fwlink/p/?linkid=862452
