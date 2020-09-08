---
author: erhopf
ms.author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 05/11/2020
ms.openlocfilehash: 235b7946fbcfc2322878428cce72e77ecceb9cfc
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88011018"
---
## <a name="authenticate-with-azure-active-directory"></a>Autenticación mediante Azure Active Directory

> [!IMPORTANT]
> 1. Actualmente, **solo** Computer Vision API, Face API, Text Analytics API, Lector inmersivo, Form Recognizer, Anomaly Detector y todos los servicios de Bing, excepto Bing Custom Search, admiten la autenticación mediante Azure Active Directory (AAD).
> 2. La autenticación con AAD siempre debe usarse junto con el nombre de subdominio personalizado de su recurso de Azure. Los [puntos de conexión regionales](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-custom-subdomains#is-there-a-list-of-regional-endpoints) no admiten la autenticación con AAD.

En las secciones anteriores, le mostramos cómo realizar la autenticación en Azure Cognitive Services mediante una clave de suscripción de un solo servicio o de varios servicios. Aunque estas claves proporcionan un camino rápido y fácil para iniciar el desarrollo, se quedan cortas en escenarios más complejos que requieren control de acceso basados en roles de Azure (RBAC de Azure). Echemos un vistazo a lo que se necesita para autenticarse con Azure Active Directory (AAD).

En las secciones siguientes, usará el entorno de Azure Cloud Shell o la CLI de Azure para crear un subdominio, asignar roles y obtener un token de portador para llamar a Cognitive Services de Azure. Si se bloquea, se proporcionan vínculos en cada sección con todas las opciones disponibles para cada comando de Azure Cloud Shell o la CLI de Azure.

### <a name="create-a-resource-with-a-custom-subdomain"></a>Creación de un recurso con un subdominio personalizado

El primer paso es crear un subdominio personalizado. Si desea usar un recurso de Cognitive Services existente que no tenga un nombre de subdominio personalizado, siga las instrucciones de [Subdominios personalizados para Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-custom-subdomains#how-does-this-impact-existing-resources) para habilitar el subdominio personalizado para el recurso.

1. Para empezar, abra Azure Cloud Shell, Después [seleccione una suscripción](https://docs.microsoft.com/powershell/module/az.accounts/set-azcontext?view=azps-3.3.0):

   ```powershell-interactive
   Set-AzContext -SubscriptionName <SubscriptionName>
   ```

2. A continuación, [cree un recurso de Cognitive Services](https://docs.microsoft.com/powershell/module/az.cognitiveservices/new-azcognitiveservicesaccount?view=azps-1.8.0) con un subdominio personalizado. El nombre del subdominio debe ser único globalmente y no puede incluir caracteres especiales, como: ".", "!", ",".

   ```powershell-interactive
   $account = New-AzCognitiveServicesAccount -ResourceGroupName <RESOURCE_GROUP_NAME> -name <ACCOUNT_NAME> -Type <ACCOUNT_TYPE> -SkuName <SUBSCRIPTION_TYPE> -Location <REGION> -CustomSubdomainName <UNIQUE_SUBDOMAIN>
   ```

3. Si se realiza correctamente, el **punto de conexión** debe mostrar el nombre de subdominio único del recurso.


### <a name="assign-a-role-to-a-service-principal"></a>Asignación de un rol a una entidad de servicio

Ahora que tiene un subdominio personalizado asociado al recurso, deberá asignar un rol a una entidad de servicio.

> [!NOTE]
> Tenga en cuenta que las asignaciones de roles de Azure pueden tardar hasta cinco minutos en propagarse.

1. En primer lugar, vamos a registrar una [aplicación de AAD](https://docs.microsoft.com/powershell/module/Az.Resources/New-AzADApplication?view=azps-1.8.0).

   ```powershell-interactive
   $SecureStringPassword = ConvertTo-SecureString -String <YOUR_PASSWORD> -AsPlainText -Force

   $app = New-AzADApplication -DisplayName <APP_DISPLAY_NAME> -IdentifierUris <APP_URIS> -Password $SecureStringPassword
   ```

   Va a necesitar el valor de **ApplicationID** en el paso siguiente.

2. Después, tiene que [crear una entidad de servicio](https://docs.microsoft.com/powershell/module/az.resources/new-azadserviceprincipal?view=azps-1.8.0) para la aplicación de AAD.

   ```powershell-interactive
   New-AzADServicePrincipal -ApplicationId <APPLICATION_ID>
   ```

   >[!NOTE]
   > Si registra una aplicación en Azure Portal, este paso se completa automáticamente.

3. El último paso es [asignar el rol "Usuario de Cognitive Services"](https://docs.microsoft.com/powershell/module/az.Resources/New-azRoleAssignment?view=azps-1.8.0) a la entidad de servicio (en el ámbito del recurso). Mediante la asignación de un rol, concede acceso a la entidad de servicio a este recurso. Puede conceder a la misma entidad de servicio acceso a varios recursos de la suscripción.
   >[!NOTE]
   > Se utiliza el valor de ObjectId de la entidad de servicio, no el valor de ObjectId de la aplicación.
   > ACCOUNT_ID será el identificador de recurso de Azure de la cuenta de Cognitive Services que ha creado. Puede encontrar el identificador de recurso de Azure en las "propiedades" del recurso en Azure Portal.

   ```azurecli-interactive
   New-AzRoleAssignment -ObjectId <SERVICE_PRINCIPAL_OBJECTID> -Scope <ACCOUNT_ID> -RoleDefinitionName "Cognitive Services User"
   ```

### <a name="sample-request"></a>Solicitud de ejemplo

En este ejemplo, se usa una contraseña para autenticar la entidad de servicio. El token proporcionado se usa para llamar a la API Computer Vision.

1. Obtenga el **TenantId**:
   ```powershell-interactive
   $context=Get-AzContext
   $context.Tenant.Id
   ```

2. Obtenga un token:
   > [!NOTE]
   > Si está usando Azure Cloud Shell, la clase `SecureClientSecret` no está disponible. 

   #### <a name="powershell"></a>[PowerShell](#tab/powershell)
   ```powershell-interactive
   $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList "https://login.windows.net/<TENANT_ID>"
   $secureSecretObject = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.SecureClientSecret" -ArgumentList $SecureStringPassword   
   $clientCredential = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential" -ArgumentList $app.ApplicationId, $secureSecretObject
   $token=$authContext.AcquireTokenAsync("https://cognitiveservices.azure.com/", $clientCredential).Result
   $token
   ```
   
   #### <a name="azure-cloud-shell"></a>[Azure Cloud Shell](#tab/azure-cloud-shell)
   ```Azure Cloud Shell
   $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList "https://login.windows.net/<TENANT_ID>"
   $clientCredential = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential" -ArgumentList $app.ApplicationId, <YOUR_PASSWORD>
   $token=$authContext.AcquireTokenAsync("https://cognitiveservices.azure.com/", $clientCredential).Result
   $token
   ``` 

   ---

3. Llame a la API Computer Vision:
   ```powershell-interactive
   $url = $account.Endpoint+"vision/v1.0/models"
   $result = Invoke-RestMethod -Uri $url  -Method Get -Headers @{"Authorization"=$token.CreateAuthorizationHeader()} -Verbose
   $result | ConvertTo-Json
   ```

Como alternativa, la entidad de servicio se puede autenticar con un certificado. Además de la entidad de servicio, la entidad de seguridad de usuario también se admite si tiene permisos delegados a través de otra aplicación de AAD. En este caso, en lugar de contraseñas o certificados, se les pedirá a los usuarios la autenticación en dos fases al adquirir el token.

## <a name="authorize-access-to-managed-identities"></a>Autorización para obtener acceso a identidades administradas
 
Cognitive Services admite la autenticación de Azure Active Directory (Azure AD) con [identidades administradas para los recursos de Azure](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview). Las identidades administradas para recursos de Azure pueden autorizar el acceso a los recursos de Cognitive Services con credenciales de Azure AD desde aplicaciones que se ejecutan en máquinas virtuales (VM) de Azure, aplicaciones de función, conjuntos de escalado de máquinas virtuales y otros servicios. Si usa identidades administradas para recursos de Azure junto con autenticación de Azure AD, puede evitar el almacenamiento de credenciales con las aplicaciones que se ejecutan en la nube.  

### <a name="enable-managed-identities-on-a-vm"></a>Habilitación de identidades administradas en una máquina virtual

Para poder usar identidades administradas para recursos de Azure a fin de autorizar el acceso los recursos de Cognitive Services desde la VM, primero debe habilitar las identidades administradas para los recursos de Azure en la VM. Para aprender a habilitar las identidades administradas para los recursos de Azure, consulte:

- [Azure Portal](https://docs.microsoft.com/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm)
- [Azure PowerShell](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm)
- [CLI de Azure](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm)
- [Plantilla de Azure Resource Manager](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm)
- [Bibliotecas cliente de Azure Resource Manager](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm)

Para más información sobre las identidades administradas, consulte [Identidades administradas para recursos de Azure](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).
