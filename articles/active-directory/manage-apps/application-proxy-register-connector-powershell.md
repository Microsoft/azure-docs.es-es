---
title: Instalación silenciosa del conector del proxy de la aplicación de Azure AD | Microsoft Docs
description: Explica cómo realizar una instalación no atendida del conector de Proxy de aplicación de Azure AD para proporcionar acceso remoto seguro a las aplicaciones locales.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/17/2018
ms.author: mimart
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0eb3e52dfd02bd7948f1b5ffd908ac1255118008
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65782919"
---
# <a name="create-an-unattended-installation-script-for-the-azure-ad-application-proxy-connector"></a>Creación de un script de instalación desatendida para el conector del proxy de la aplicación de Azure AD

Este tema le ayuda a crear un script de Windows PowerShell que hace posible la instalación desatendida y el registro para el conector del proxy de la aplicación de Azure AD.

Esta capacidad resulta útil cuando desea hacer lo siguiente:

* Instale el conector en los servidores de Windows que no tienen interfaz de usuario habilitada o a los que no puede acceder con Escritorio remoto.
* Instalar y registrar muchos conectores a la vez.
* Integrar la instalación del conector y el registro como parte de otro procedimiento.
* Crear una imagen de servidor estándar que contiene los bits de conector, pero no está registrada.

Para el [conector del Proxy de aplicación](application-proxy-connectors.md) para trabajar, debe estar registrado con el directorio de Azure AD mediante un administrador de la aplicación y una contraseña. Normalmente, esta información se especifica durante la instalación del conector en un cuadro de diálogo emergente, pero también puede usar PowerShell para automatizar este proceso.

Hay dos pasos para una instalación desatendida. Primero, instale el conector. Después, registre el conector con Azure AD. 

## <a name="install-the-connector"></a>Instalación del conector
Siga estos pasos para instalar el conector sin registrarlo:

1. Abra el símbolo del sistema.
2. Ejecute el siguiente comando, en el que el parámetro /q significa instalación desatendida. Una instalación desatendida no le pide que acepte el contrato de licencia para el usuario final.
   
        AADApplicationProxyConnectorInstaller.exe REGISTERCONNECTOR="false" /q

## <a name="register-the-connector-with-azure-ad"></a>Registro del conector con Azure AD
Hay dos métodos posibles para registrar el conector:

* Registro del conector mediante un objeto de credenciales de Windows PowerShell
* Registro del conector mediante un token creado sin conexión

### <a name="register-the-connector-using-a-windows-powershell-credential-object"></a>Registro del conector mediante un objeto de credenciales de Windows PowerShell
1. Cree un objeto de credenciales de Windows PowerShell `$cred` que contiene un nombre de usuario y una contraseña administrativos para el directorio. Ejecute el comando siguiente, reemplazando *\<username\>* y *\<password\>* por los valores correspondientes:
   
        $User = "<username>"
        $PlainPassword = '<password>'
        $SecurePassword = $PlainPassword | ConvertTo-SecureString -AsPlainText -Force
        $cred = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $User, $SecurePassword
2. Vaya a **C:\Archivos de programa\Microsoft AAD App Proxy Connector** y ejecute el siguiente script mediante el objeto `$cred` que creó:
   
        .\RegisterConnector.ps1 -modulePath "C:\Program Files\Microsoft AAD App Proxy Connector\Modules\" -moduleName "AppProxyPSModule" -Authenticationmode Credentials -Usercredentials $cred -Feature ApplicationProxy

### <a name="register-the-connector-using-a-token-created-offline"></a>Registro del conector mediante un token creado sin conexión
1. Cree un token sin conexión mediante la clase AuthenticationContext con los valores de este fragmento de código o con los cmdlets de PowerShell que tiene a continuación:

    **Con C#:**

        using System;
        using System.Diagnostics;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;

        class Program
        {
        #region constants
        /// <summary>
        /// The AAD authentication endpoint uri
        /// </summary>
        static readonly Uri AadAuthenticationEndpoint = new Uri("https://login.microsoftonline.com/common/oauth2/token?api-version=1.0");

        /// <summary>
        /// The application ID of the connector in AAD
        /// </summary>
        static readonly string ConnectorAppId = "55747057-9b5d-4bd4-b387-abf52a8bd489";

        /// <summary>
        /// The reply address of the connector application in AAD
        /// </summary>
        static readonly Uri ConnectorRedirectAddress = new Uri("urn:ietf:wg:oauth:2.0:oob");

        /// <summary>
        /// The AppIdUri of the registration service in AAD
        /// </summary>
        static readonly Uri RegistrationServiceAppIdUri = new Uri("https://proxy.cloudwebappproxy.net/registerapp");

        #endregion

        #region private members
        private string token;
        private string tenantID;
        #endregion

        public void GetAuthenticationToken()
        {
            AuthenticationContext authContext = new AuthenticationContext(AadAuthenticationEndpoint.AbsoluteUri);

            AuthenticationResult authResult = authContext.AcquireToken(RegistrationServiceAppIdUri.AbsoluteUri,
                ConnectorAppId,
                ConnectorRedirectAddress,
                PromptBehavior.Always);

            if (authResult == null || string.IsNullOrEmpty(authResult.AccessToken) || string.IsNullOrEmpty(authResult.TenantId))
            {
                Trace.TraceError("Authentication result, token or tenant id returned are null");
                throw new InvalidOperationException("Authentication result, token or tenant id returned are null");
            }

            token = authResult.AccessToken;
            tenantID = authResult.TenantId;
        }

    **Con PowerShell:**

        # Locate AzureAD PowerShell Module
        # Change Name of Module to AzureAD after what you have installed
        $AADPoshPath = (Get-InstalledModule -Name AzureAD).InstalledLocation
        # Set Location for ADAL Helper Library
        $ADALPath = $(Get-ChildItem -Path $($AADPoshPath) -Filter Microsoft.IdentityModel.Clients.ActiveDirectory.dll -Recurse ).FullName | Select-Object -Last 1
        
        # Add ADAL Helper Library
        Add-Type -Path $ADALPath
        
        #region constants
        
        # The AAD authentication endpoint uri
        [uri]$AadAuthenticationEndpoint = "https://login.microsoftonline.com/common/oauth2/token?api-version=1.0/" 
        
        # The application ID of the connector in AAD
        [string]$ConnectorAppId = "55747057-9b5d-4bd4-b387-abf52a8bd489"
        
        # The reply address of the connector application in AAD
        [uri]$ConnectorRedirectAddress = "urn:ietf:wg:oauth:2.0:oob" 
        
        # The AppIdUri of the registration service in AAD
        [uri]$RegistrationServiceAppIdUri = "https://proxy.cloudwebappproxy.net/registerapp"
        
        #endregion
        
        #region GetAuthenticationToken
        
        # Set AuthN context
        $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList $AadAuthenticationEndpoint
        
        # Build platform parameters
        $promptBehavior = [Microsoft.IdentityModel.Clients.ActiveDirectory.PromptBehavior]::Always
        $platformParam = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.PlatformParameters" -ArgumentList $promptBehavior
        
        # Do AuthN and get token
        $authResult = $authContext.AcquireTokenAsync($RegistrationServiceAppIdUri.AbsoluteUri, $ConnectorAppId, $ConnectorRedirectAddress, $platformParam).Result
        
        # Check AuthN result
        If (($authResult) -and ($authResult.AccessToken) -and ($authResult.TenantId) ) {
        $token = $authResult.AccessToken
        $tenantId = $authResult.TenantId
        }
        Else {
        Write-Output "Authentication result, token or tenant id returned are null"
        }
        
        #endregion

2. Una vez que tenga el token, cree SecureString con dicho token:

   `$SecureToken = $Token | ConvertTo-SecureString -AsPlainText -Force`

3. Ejecute el siguiente comando de Windows PowerShell, reemplazando \<tenant GUID\> por su identificador de directorio:

   `.\RegisterConnector.ps1 -modulePath "C:\Program Files\Microsoft AAD App Proxy Connector\Modules\" -moduleName "AppProxyPSModule" -Authenticationmode Token -Token $SecureToken -TenantId <tenant GUID> -Feature ApplicationProxy`

## <a name="next-steps"></a>Pasos siguientes 
* [Publicar aplicaciones mediante su propio nombre de dominio](application-proxy-configure-custom-domain.md)
* [Habilitar el inicio de sesión único](application-proxy-configure-single-sign-on-with-kcd.md)
* [Solucionar los problemas que tiene con el Proxy de aplicación](application-proxy-troubleshoot.md)


