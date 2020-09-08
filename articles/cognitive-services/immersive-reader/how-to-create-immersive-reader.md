---
title: Creación de un recurso del Lector inmersivo
titleSuffix: Azure Cognitive Services
description: En este artículo se mostrará cómo crear un nuevo recurso de Lector inmersivo con un subdominio personalizado y, luego, configurar Azure AD en el inquilino de Azure.
services: cognitive-services
author: rwaller
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: conceptual
ms.date: 07/22/2019
ms.author: rwaller
ms.openlocfilehash: 66a2fde47f71536661431959b957246e28c81d6a
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88639821"
---
# <a name="create-an-immersive-reader-resource-and-configure-azure-active-directory-authentication"></a>Creación de un recurso del Lector inmersivo y configuración de la autenticación de Azure Active Directory

En este artículo, se proporciona un script que creará un recurso del Lector inmersivo y configurará la autenticación de Azure Active Directory (Azure AD). Cada vez que se crea un recurso del Lector inmersivo, ya sea con este script o en el portal, también debe configurarse con permisos de Azure AD. Este script le ayudará a hacerlo.

El script está diseñado para crear y configurar automáticamente todos los recursos necesarios del Lector inmersivo y de Azure AD en un solo paso. Sin embargo, también puede configurar la autenticación de Azure AD para un recurso existente del Lector inmersivo, si, por ejemplo, ya ha creado uno en Azure Portal.

En el caso de algunos clientes, puede que sea necesario crear varios recursos del Lector inmersivo, para desarrollo frente a producción, o quizás para varias regiones diferentes en las que se implemente el servicio. En esos casos, puede volver y usar el script varias veces para crear distintos recursos del Lector inmersivo y configurarlos con los permisos de Azure AD.

El script está diseñado para ser flexible. En primer lugar, buscará los recursos existentes del Lector inmersivo y de Azure AD en su suscripción, y solo los creará según sea necesario si aún no existen. Si es la primera vez que crea un recurso del Lector inmersivo, el script hará todo lo que necesita. Si desea usarlo solo para configurar Azure AD para un recurso existente del Lector inmersivo creado en el portal, también se encargará. Asimismo, se puede usar para crear y configurar varios recursos del Lector inmersivo.

## <a name="set-up-powershell-environment"></a>Configuración del entorno de PowerShell

1. Para empezar, abra [Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview). Asegúrese de que Cloud Shell está establecido en PowerShell en la lista desplegable de la parte superior izquierda o escriba `pwsh`.

1. Copie y pegue el siguiente fragmento de código en el shell.

    ```azurepowershell-interactive
    function Create-ImmersiveReaderResource(
        [Parameter(Mandatory=$true, Position=0)] [String] $SubscriptionName,
        [Parameter(Mandatory=$true)] [String] $ResourceName,
        [Parameter(Mandatory=$true)] [String] $ResourceSubdomain,
        [Parameter(Mandatory=$true)] [String] $ResourceSKU,
        [Parameter(Mandatory=$true)] [String] $ResourceLocation,
        [Parameter(Mandatory=$true)] [String] $ResourceGroupName,
        [Parameter(Mandatory=$true)] [String] $ResourceGroupLocation,
        [Parameter(Mandatory=$true)] [String] $AADAppDisplayName="ImmersiveReaderAAD",
        [Parameter(Mandatory=$true)] [String] $AADAppIdentifierUri,
        [Parameter(Mandatory=$true)] [String] $AADAppClientSecret,
        [Parameter(Mandatory=$true)] [String] $AADAppClientSecretExpiration
    )
    {
        $unused = ''
        if (-not [System.Uri]::TryCreate($AADAppIdentifierUri, [System.UriKind]::Absolute, [ref] $unused)) {
            throw "Error: AADAppIdentifierUri must be a valid URI"
        }

        Write-Host "Setting the active subscription to '$SubscriptionName'"
        $subscriptionExists = Get-AzSubscription -SubscriptionName $SubscriptionName
        if (-not $subscriptionExists) {
            throw "Error: Subscription does not exist"
        }
        az account set --subscription $SubscriptionName

        $resourceGroupExists = az group exists --name $ResourceGroupName
        if ($resourceGroupExists -eq "false") {
            Write-Host "Resource group does not exist. Creating resource group"
            $groupResult = az group create --name $ResourceGroupName --location $ResourceGroupLocation
            if (-not $groupResult) {
                throw "Error: Failed to create resource group"
            }
            Write-Host "Resource group created successfully"
        }

        # Create an Immersive Reader resource if it doesn't already exist
        $resourceId = az cognitiveservices account show --resource-group $ResourceGroupName --name $ResourceName --query "id" -o tsv
        if (-not $resourceId) {
            Write-Host "Creating the new Immersive Reader resource '$ResourceName' (SKU '$ResourceSKU') in '$ResourceLocation' with subdomain '$ResourceSubdomain'"
            $resourceId = az cognitiveservices account create `
                            --name $ResourceName `
                            --resource-group $ResourceGroupName `
                            --kind ImmersiveReader `
                            --sku $ResourceSKU `
                            --location $ResourceLocation `
                            --custom-domain $ResourceSubdomain `
                            --query "id" `
                            -o tsv

            if (-not $resourceId) {
                throw "Error: Failed to create Immersive Reader resource"
            }
            Write-Host "Immersive Reader resource created successfully"
        }

        # Create an Azure Active Directory app if it doesn't already exist
        $clientId = az ad app show --id $AADAppIdentifierUri --query "appId" -o tsv
        if (-not $clientId) {
            Write-Host "Creating new Azure Active Directory app"
            $clientId = az ad app create --password $AADAppClientSecret --end-date "$AADAppClientSecretExpiration" --display-name $AADAppDisplayName --identifier-uris $AADAppIdentifierUri --query "appId" -o tsv

            if (-not $clientId) {
                throw "Error: Failed to create Azure Active Directory app"
            }
            Write-Host "Azure Active Directory app created successfully."
            Write-Host "NOTE: To manage your Active Directory app client secrets after this Immersive Reader Resource has been created please visit https://portal.azure.com and go to Home -> Azure Active Directory -> App Registrations -> $AADAppDisplayName -> Certificates and Secrets blade -> Client Secrets section" -ForegroundColor Yellow
        }

        # Create a service principal if it doesn't already exist
        $principalId = az ad sp show --id $AADAppIdentifierUri --query "objectId" -o tsv
        if (-not $principalId) {
            Write-Host "Creating new service principal"
            az ad sp create --id $clientId | Out-Null
            $principalId = az ad sp show --id $AADAppIdentifierUri --query "objectId" -o tsv

            if (-not $principalId) {
                throw "Error: Failed to create new service principal"
            }
            Write-Host "New service principal created successfully"
        }

        # Sleep for 5 seconds to allow the new service principal to propagate
        Write-Host "Sleeping for 5 seconds"
        Start-Sleep -Seconds 5

        Write-Host "Granting service principal access to the newly created Immersive Reader resource"
        $accessResult = az role assignment create --assignee $principalId --scope $resourceId --role "Cognitive Services User"
        if (-not $accessResult) {
            throw "Error: Failed to grant service principal access"
        }
        Write-Host "Service principal access granted successfully"

        # Grab the tenant ID, which is needed when obtaining an Azure AD token
        $tenantId = az account show --query "tenantId" -o tsv

        # Collect the information needed to obtain an Azure AD token into one object
        $result = @{}
        $result.TenantId = $tenantId
        $result.ClientId = $clientId
        $result.ClientSecret = $AADAppClientSecret
        $result.Subdomain = $ResourceSubdomain

        Write-Host "Success! " -ForegroundColor Green -NoNewline
        Write-Host "Save the following JSON object to a text file for future reference:"
        Write-Output (ConvertTo-Json $result)
    }
    ```

1. Ejecute la función `Create-ImmersiveReaderResource` y proporcione los parámetros según corresponda.

    ```azurepowershell-interactive
    Create-ImmersiveReaderResource
      -SubscriptionName '<SUBSCRIPTION_NAME>' `
      -ResourceName '<RESOURCE_NAME>' `
      -ResourceSubdomain '<RESOURCE_SUBDOMAIN>' `
      -ResourceSKU '<RESOURCE_SKU>' `
      -ResourceLocation '<RESOURCE_LOCATION>' `
      -ResourceGroupName '<RESOURCE_GROUP_NAME>' `
      -ResourceGroupLocation '<RESOURCE_GROUP_LOCATION>' `
      -AADAppDisplayName '<AAD_APP_DISPLAY_NAME>' `
      -AADAppIdentifierUri '<AAD_APP_IDENTIFIER_URI>' `
      -AADAppClientSecret '<AAD_APP_CLIENT_SECRET>'
      -AADAppClientSecretExpiration '<AAD_APP_CLIENT_SECRET_Expiration>'
    ```

    | Parámetro | Comentarios |
    | --- | --- |
    | SubscriptionName |Nombre de la suscripción a Azure que se va a usar para el recurso del Lector inmersivo. Debe tener una suscripción para poder crear un recurso. |
    | nombreDelRecurso |  Debe ser alfanumérico y puede contener "-", siempre y cuando "-" no sea el primer ni el último carácter. No puede tener una longitud de más de 63 caracteres.|
    | ResourceSubdomain |Subdominio personalizado necesario para el recurso del Lector inmersivo. El SDK usa el subdominio al llamar al servicio del Lector inmersivo para iniciar el Lector. El subdominio debe ser único globalmente. El subdominio debe ser alfanumérico y puede contener "-", siempre y cuando "-" no sea el primer ni el último carácter. No puede tener una longitud de más de 63 caracteres. Este parámetro es opcional si el recurso ya existe. |
    | ResourceSKU |Opciones: `S0`. Visite nuestra [página de precios de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/immersive-reader/) para obtener más información sobre cada SKU disponible. Este parámetro es opcional si el recurso ya existe. |
    | ResourceLocation |Opciones: `eastus`, `eastus2`, `southcentralus`, `westus`, `westus2`, `australiaeast`, `southeastasia`, `centralindia`, `japaneast`, `northeurope`, `uksouth`, `westeurope`. Este parámetro es opcional si el recurso ya existe. |
    | ResourceGroupName |Los recursos se crean en grupos de recursos dentro de las suscripciones. Suministre el nombre de un grupo de recursos existente. Si el grupo de recursos todavía no existe, se creará uno con este nombre. |
    | ResourceGroupLocation |Si el grupo de recursos no existe, debe proporcionar una ubicación donde se creará el grupo. Ejecute `az account list-locations` para obtener una lista de ubicaciones. Use la propiedad *name* (sin espacios) del resultado devuelto. Este parámetro es opcional si el grupo de recursos ya existe. |
    | AADAppDisplayName |El nombre para mostrar de la aplicación de Azure Active Directory. Si no se encuentra una aplicación de Azure AD existente, se creará una con este nombre. Este parámetro es opcional si la aplicación de Azure AD ya existe. |
    | AADAppIdentifierUri |El URI de la aplicación de Azure AD. Si no se encuentra una aplicación de Azure AD existente, se creará una con este URI. Por ejemplo, `https://immersivereaderaad-mycompany`. |
    | AADAppClientSecret |Una contraseña que cree usted que se usará más adelante para autenticarse al adquirir un token para iniciar el Lector inmersivo. La contraseña debe tener al menos 16 caracteres, contener al menos 1 carácter especial y al menos 1 carácter numérico. Para administrar secretos de cliente de aplicaciones de Azure AD después de haber creado este recurso, visite https://portal.azure.com y vaya a Inicio > Azure Active Directory-> Registros de aplicaciones-> `[AADAppDisplayName]` -> hoja Certificados y secretos -> sección Secretos de clientes (como se muestra en la captura de pantalla "Administración de los secretos de aplicación de Azure AD" a continuación). |
    | AADAppClientSecretExpiration |La fecha u hora tras las cuales expirará `[AADAppClientSecret]` (por ejemplo, "2020-12-31T11:59:59+00:00" o "2020-12-31"). |

    Administración de los secretos de aplicación de Azure AD

    ![Hoja Certificados y secretos de Azure Portal](./media/client-secrets-blade.png)

1. Copie la salida JSON en un archivo de texto para usarla en otro momento. La salida debe tener un aspecto similar al siguiente.

    ```json
    {
      "TenantId": "...",
      "ClientId": "...",
      "ClientSecret": "...",
      "Subdomain": "..."
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

* Consulte el [inicio rápido de Node.js](./quickstarts/client-libraries.md?pivots=programming-language-nodejs) para ver qué más puede hacer con el SDK del Lector inmersivo con Node.js
* Vea el [tutorial de Android](./tutorial-android.md) para ver qué más puede hacer con el SDK del Lector inmersivo con Java o Kotlin para Android
* Vea el [tutorial de iOS](./tutorial-ios.md) para ver qué más puede hacer con el SDK del Lector inmersivo con Swift para iOS
* Vea el [tutorial para Python](./tutorial-python.md) para consultar qué más puede hacer con el SDK del Lector inmersivo con Python.
* Explorar el [SDK del Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk) y agregar la [Referencia del SDK del Lector inmersivo](./reference.md)




