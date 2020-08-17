---
title: Creación de una aplicación web para el Conector de administración de servicios
description: Cree una aplicación web de Service Manager con un script automatizado para conectar con Conector de administración de servicios de TI en Azure y supervise y administre de manera centralizada de los elementos de trabajo ITSM.
ms.subservice: logs
ms.topic: conceptual
author: nolavime
ms.author: v-jysur
ms.date: 01/23/2018
ms.openlocfilehash: 46c219a82d45129ed0856d22d88be819938e5240
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325311"
---
# <a name="create-service-manager-web-app-using-the-automated-script"></a>Creación de una aplicación web de Service Manager mediante el script automatizado

Use el siguiente script para crear la aplicación web para la instancia de Service Manager. Aquí puede obtener más información sobre la conexión de Service Manager. [Aplicación web de Service Manager](./itsmc-connections.md#create-and-deploy-service-manager-web-app-service)

Ejecute el script proporcionando los siguientes detalles necesarios:

- Detalles de la suscripción de Azure
- Definición de un nombre de grupo de recursos
- Location
- Detalles del servidor de Service Manager (nombre del servidor, dominio, nombre de usuario y contraseña)
- Prefijo de nombre de sitio de la aplicación web
- Espacio de nombres de ServiceBus.

El script creará la aplicación web con el nombre que especificó (junto con algunas cadenas adicionales para hacerlo único). Genera la **dirección URL de la aplicación web**, el **id. de cliente** y el **secreto de cliente**.

Guarde estos valores, ya que los necesitará cuando cree una conexión con el Conector de Administración de servicios de TI.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

 Windows Management Framework 5.0 o posterior.
Windows 10 tiene la versión 5.1 de forma predeterminada. Puede descargar el marco desde [aquí](https://www.microsoft.com/download/details.aspx?id=50395):

Use el siguiente script:

```powershell
####################################
# User Configuration Section Begins
####################################

# Subscription name in Azure account. Check in Azure Portal.
$azureSubscriptionName = ""

# Resource group name for resource deployment. Could be an existing resource group or a new one to be created.
$resourceGroupName = ""

# Location for existing resource group or new resource group deployment
################################### List of available regions #################################################
# centralus,eastasia,southeastasia,eastus,eastus2,westus,westus2,northcentralus,southcentralus,westcentralus,
# northeurope,westeurope,japaneast,japanwest,brazilsouth,australiasoutheast,australiaeast,westindia,southindia,
# centralindia,canadacentral,canadaeast,uksouth,ukwest.
###############################################################################################################
$location = ""

# Service Manager Authentication Settings
$serverName = ""
$domain = ""
$username = ""
$password = ""


# Azure site Name Prefix. Default is "smoc". It can be configured to any desired value.
$siteNamePrefix = ""

# Service Bus namespace. Please provide an already existing service bus namespace.
# If it doesn't exist, a new one will be created with name $siteName + "sbn" which can also be later reused for any other hybrid connections.
$serviceName = ""

##################################
# User Configuration Section Ends
##################################

################
# Installations
################

# Allowing the execution of the script for current user.  
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser -Force

Write-Host "Checking for required modules..."
if(!(Get-PackageProvider -Name NuGet))
{
   Write-Host "Installing NuGet Package Provider..."
   Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Scope CurrentUser -Force -WarningAction SilentlyContinue
}
$module = Get-Module -ListAvailable -Name Az

if(!$module -or ($module[0].Version.Major -lt 1))
{
    Write-Host "Installing Az Module..."  
    try
    {
        # In case of Win 10 Anniversary update
        Install-Module Az -MinimumVersion 1.0 -Scope CurrentUser -Force -WarningAction SilentlyContinue -AllowClobber
    }
    catch
    {
        Install-Module Az -MinimumVersion 1.0 -Scope CurrentUser -Force -WarningAction SilentlyContinue
    }

}

Write-Host "Requirement check complete!!"

#############
# Parameters
#############

$errorActionPreference = "Stop"

$templateUri = "https://raw.githubusercontent.com/Azure/SMOMSConnector/master/azuredeploy.json"

if(!$siteNamePrefix)
{
    $siteNamePrefix = "smoc"
}

Connect-AzAccount

$context = Set-AzContext -SubscriptionName $azureSubscriptionName -WarningAction SilentlyContinue

$resourceProvider = Get-AzResourceProvider -ProviderNamespace Microsoft.Web

if(!$resourceProvider -or $resourceProvider[0].RegistrationState -ne "Registered")
{
    try
    {
        Write-Host "Registering Microsoft.Web Resource Provider"
        Register-AzResourceProvider -ProviderNamespace Microsoft.Web
    }
    catch
    {
        Write-Host "Failed to Register Microsoft.Web Resource Provider. Please register it in Azure Portal."
        exit
    }   
}
do
{
    $rand = Get-Random -Maximum 32000

    $siteName = $siteNamePrefix + $rand
    
    $resource = Get-AzResource -Name $siteName -ResourceType Microsoft.Web/sites

}while($resource)

$azureSite = "https://"+$siteName+".azurewebsites.net"

##############
# MAIN Begins
##############

# Web App Deployment
####################

$tenant = $context.Tenant.Id
if(!$tenant)
{
    #For backward compatibility with older versions
    $tenant = $context.Tenant.TenantId
}
try
{
    Get-AzResourceGroup -Name $resourceGroupName
}
catch
{
    New-AzResourceGroup -Location $location -Name $resourceGroupName
}

Write-Output "Web App Deployment in progress...."

New-AzResourceGroupDeployment -TemplateUri $templateUri -siteName $siteName -ResourceGroupName $resourceGroupName

Write-Output "Web App Deployed successfully!!"

# AAD Authentication
####################

Add-Type -AssemblyName System.Web

$secret = [System.Web.Security.Membership]::GeneratePassword(30,2).ToString()
$clientSecret = $secret | ConvertTo-SecureString -AsPlainText -Force


try
{

    Write-Host "Creating AzureAD application..."

    $adApp = New-AzADApplication -DisplayName $siteName -HomePage $azureSite -IdentifierUris $azureSite -Password $clientSecret

    Write-Host "AzureAD application created successfully!!"
}
catch
{
    # Delete the deployed web app if Azure AD application fails
    Remove-AzResource -ResourceGroupName $resourceGroupName -ResourceName $siteName -ResourceType Microsoft.Web/sites -Force

    Write-Host "Failure occurred in Azure AD application....Try again!!"

    exit

}

$clientId = $adApp.ApplicationId

$servicePrincipal = New-AzADServicePrincipal -ApplicationId $clientId

# Web App Configuration
#######################
try
{

    Write-Host "Configuring deployed Web-App..."
    $webApp = Get-AzWebAppSlot -ResourceGroupName $resourceGroupName -Name $siteName -Slot production -WarningAction SilentlyContinue

    $appSettingList = $webApp.SiteConfig.AppSettings

    $appSettings = @{}
    ForEach ($item in $appSettingList) {
        $appSettings[$item.Name] = $item.Value
    }

    $appSettings['ida:Tenant'] = $tenant
    $appSettings['ida:Audience'] = $azureSite
    $appSettings['ida:ServerName'] = $serverName
    $appSettings['ida:Domain'] = $domain
    $appSettings['ida:Username'] = $userName
    $appSettings['ida:WhitelistedClientId'] = $clientId

    $connStrings = @{}
    $kvp = @{"Type"="Custom"; "Value"=$password}
    $connStrings['ida:Password'] = $kvp

    Set-AzWebAppSlot -ResourceGroupName $resourceGroupName -Name $siteName -AppSettings $appSettings -ConnectionStrings $connStrings -Slot production -WarningAction SilentlyContinue

}
catch
{
    Write-Host "Web App configuration failed. Please ensure all values are provided in Service Manager Authentication Settings in User Configuration Section"

    # Delete the AzureRm AD Application if configuration fails
    Remove-AzADApplication -ObjectId $adApp.ObjectId -Force

    # Delete the deployed web app if configuration fails
    Remove-AzResource -ResourceGroupName $resourceGroupName -ResourceName $siteName -ResourceType Microsoft.Web/sites -Force

    exit
}


# Relay Namespace
###################

if(!$serviceName)
{
    $serviceName = $siteName + "sbn"
}

$resourceProvider = Get-AzResourceProvider -ProviderNamespace Microsoft.Relay

if(!$resourceProvider -or $resourceProvider[0].RegistrationState -ne "Registered")
{
    try
    {
        Write-Host "Registering Microsoft.Relay Resource Provider"
        Register-AzResourceProvider -ProviderNamespace Microsoft.Relay
    }
    catch
    {
        Write-Host "Failed to Register Microsoft.Relay Resource Provider. Please register it in Azure Portal."
    }   
}

$resource = Get-AzResource -Name $serviceName -ResourceType Microsoft.Relay/namespaces

if(!$resource)
{
    $serviceName = $siteName + "sbn"
    $properties = @{
        "sku" = @{
            "name"= "Standard"
            "tier"= "Standard"
            "capacity"= 1
         }
    }
    try
    {
        Write-Host "Creating Service Bus namespace..."
        New-AzResource -ResourceName $serviceName -Location $location -PropertyObject $properties -ResourceGroupName $resourceGroupName -ResourceType Microsoft.Relay/namespaces -ApiVersion 2016-07-01 -Force
    }
    catch
    {
        $err = $TRUE
        Write-Host "Creation of Service Bus Namespace failed...Please create it manually from Azure Portal.`n"
    }

}

Write-Host "Note: Please Configure Hybrid connection in the Networking section of the web application in Azure Portal to link to the on-premises system.`n"
Write-Host "App Details"
Write-Host "============"
Write-Host "App Name:"  $siteName
Write-Host "Client Id:"  $clientId
Write-Host "Client Secret:"  $secret
Write-Host "URI:"  $azureSite
if(!$err)
{
    Write-Host "ServiceBus Namespace:"  $serviceName  
}
```

## <a name="next-steps"></a>Pasos siguientes
[Configuración de la conexión híbrida](./itsmc-connections.md#configure-the-hybrid-connection).

