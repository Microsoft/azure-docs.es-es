---
title: 'Página principal personalizada para aplicaciones publicadas: Azure AD Application Proxy'
description: Se explican los conceptos básicos acerca de los conectores del Proxy de aplicación de Azure AD.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/23/2019
ms.author: kenwith
ms.reviewer: harshja
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 155a91ded6c814e2d868e8edd4572459460d006f
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88642068"
---
# <a name="set-a-custom-home-page-for-published-apps-by-using-azure-ad-application-proxy"></a>Establecimiento de una página principal personalizada para aplicaciones publicadas mediante el Proxy de aplicación de Azure AD

En este artículo se explica cómo configurar una aplicación para que dirija a un usuario a una página principal personalizada. Al publicar una aplicación con el proxy de la aplicación, se establece una dirección URL interna pero, a veces, esa no es la primera página que debe ver un usuario. Establezca una página principal personalizada para que un usuario vaya a la página correcta cuando acceda a la aplicación. Un usuario verá la página principal personalizada que ha establecido, independientemente de si accede a la aplicación desde Aplicaciones de Azure Active Directory o desde el iniciador de aplicaciones de Microsoft 365.

Cuando un usuario inicia la aplicación, se le dirige de manera predeterminada a la dirección URL raíz del dominio raíz de la aplicación publicada. La página de aterrizaje normalmente se establece como la dirección URL de la página principal. Use el módulo de Azure AD PowerShell para definir las direcciones URL de una página principal personalizada cuando desee que un usuario de la aplicación llegue a una página concreta de la aplicación.

En este escenario se explica por qué la empresa establecería una página principal personalizada:

- En su red corporativa, un usuario va a `https://ExpenseApp/login/login.aspx` para iniciar sesión y acceder a su aplicación.
- Dado que tiene otros recursos, como imágenes, que el proxy de aplicación necesita para acceder al nivel superior de la estructura de carpetas, publica la aplicación con `https://ExpenseApp` como dirección URL interna.
- La dirección URL externa predeterminada es `https://ExpenseApp-contoso.msappproxy.net`, que no lleva a un usuario externo a la página de inicio de sesión.
- En su lugar, quiere establecer `https://ExpenseApp-contoso.msappproxy.net/login/login.aspx` como la dirección URL de la página principal para que un usuario externo vea primero la página de inicio de sesión.

> [!NOTE]
> Al proporcionar a los usuarios acceso a las aplicaciones publicadas, estas aparecen en [Aplicaciones](../user-help/my-apps-portal-end-user-access.md) y el [iniciador de aplicaciones de Office 365](https://www.microsoft.com/microsoft-365/blog/2016/09/27/introducing-the-new-office-365-app-launcher/).

## <a name="before-you-start"></a>Antes de comenzar

Antes de establecer la dirección URL de la página principal tenga en cuenta los siguientes requisitos:

- La ruta de acceso que especifica debe ser una ruta de acceso de subdominio de la dirección URL del dominio raíz.

  Por ejemplo, si la dirección URL del dominio raíz es `https://apps.contoso.com/app1/`, la dirección URL de la página principal que configure debe empezar con `https://apps.contoso.com/app1/`.

- Si realiza un cambio en la aplicación publicada, este podría restablecer el valor de la dirección URL de la página principal. Si actualiza la aplicación en el futuro, debe volver a comprobar y, si es necesario, actualizar la dirección URL de la página principal.

Puede establecer la dirección URL de la página principal ya sea en Azure Portal o mediante PowerShell.

## <a name="change-the-home-page-in-the-azure-portal"></a>Vaya a la página principal de Azure Portal.

Para cambiar la dirección URL de la página principal de la aplicación en el portal de Azure AD, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador.
1. Seleccione **Azure Active Directory** y **Registros de aplicaciones**. Aparece la lista de aplicaciones registradas.
1. Elija la aplicación en la lista. Aparece una página que muestra los detalles de la aplicación registrada.
1. En **Administrar**, seleccione **Personalización de marca**.
1. Actualice la dirección **URL de página principal** con la nueva ruta de acceso.

   ![Página de personalización de marca para una aplicación registrada que muestra el campo URL de página principal](media/application-proxy-configure-custom-home-page/app-proxy-app-branding.png)

1. Seleccione **Guardar**.

## <a name="change-the-home-page-with-powershell"></a>Cambio de la página principal con PowerShell

Para configurar la página principal de una aplicación con PowerShell, necesitará lo siguiente:

1. Instale el módulo de PowerShell de Azure AD.
1. Busque el valor ObjectId de la aplicación.
1. Actualice la dirección URL de la página principal de la aplicación con comandos de PowerShell.

### <a name="install-the-azure-ad-powershell-module"></a>Instalación del módulo de PowerShell de Azure AD

Para poder definir la dirección URL de la página principal personalizada mediante PowerShell, instale el módulo de Azure AD PowerShell. Puede descargar este paquete desde la [Galería de PowerShell](https://www.powershellgallery.com/packages/AzureAD/2.0.2.16), que utiliza el punto de conexión de Graph API.

Para instalar el paquete, siga estos pasos:

1. Abra una ventana de PowerShell estándar y luego ejecute el siguiente comando:

   ```powershell
   Install-Module -Name AzureAD
   ```

    Si está ejecutando el comando como no administrador, use la opción `-scope currentuser`.

1. Durante la instalación, seleccione **Y** para instalar dos paquetes de Nuget.org. Se requieren ambos paquetes.

### <a name="find-the-objectid-of-the-app"></a>Búsqueda del valor de ObjectID de la aplicación

Puede obtener el valor de ObjectId de la aplicación si busca la aplicación por su nombre para mostrar o su página principal.

1. En la misma ventana de PowerShell, importe el módulo Azure AD.

   ```powershell
   Import-Module AzureAD
   ```

1. Inicie sesión como administrador de inquilinos en el módulo de Azure AD.

   ```powershell
   Connect-AzureAD
   ```

1. Busque la aplicación. En este ejemplo se usa PowerShell para buscar el valor de ObjectId buscando la aplicación con un nombre para mostrar de `SharePoint`.

   ```powershell
   Get-AzureADApplication | Where-Object { $_.DisplayName -eq "SharePoint" } | Format-List DisplayName, Homepage, ObjectId
   ```

   Debe obtener un resultado similar al que se muestra aquí. Copie el GUID de ObjectId para usarlo en la siguiente sección.

   ```console
   DisplayName : SharePoint
   Homepage    : https://sharepoint-iddemo.msappproxy.net/
   ObjectId    : 8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4
   ```

   También podría simplemente extraer la lista de todas las aplicaciones, buscar en ella la aplicación con determinado nombre para mostrar o página principal y copiar el valor de ObjectId de la aplicación una vez que la encuentra.

   ```powershell
   Get-AzureADApplication | Format-List DisplayName, Homepage, ObjectId
   ```

### <a name="update-the-home-page-url"></a>Actualización de la dirección URL de la página principal

Cree la dirección URL de la página principal y actualice la aplicación con ese valor. Siga usando la misma ventana de PowerShell o, s usa una ventana nueva de PowerShell, vuelva a iniciar sesión en el módulo de Azure AD con `Connect-AzureAD`. A continuación, siga estos pasos:

1. Cree una variable que contenga el valor de ObjectId que copió en la sección anterior. (Reemplace el valor de ObjectId que se usa para este ejemplo de SharePoint con el valor de ObjectId de la aplicación).

   ```powershell
   $objguid = "8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4"
   ```

1. Ejecute el comando siguiente para confirmar que tiene la aplicación correcta. La salida debe ser idéntica a la que vio en la sección anterior ([Búsqueda del valor de ObjectId de la aplicación](#find-the-objectid-of-the-app)).

   ```powershell
   Get-AzureADApplication -ObjectId $objguid | Format-List DisplayName, Homepage, ObjectId
   ```

1. Cree un objeto de aplicación en blanco para que contenga los cambios que desea realizar.

   ```powershell
   $appnew = New-Object "Microsoft.Open.AzureAD.Model.Application"
   ```

1. Establezca la dirección URL de la página principal en el valor que quiera. El valor debe ser una ruta de acceso de subdominio de la aplicación publicada. Por ejemplo, si cambia la dirección URL de página principal de `https://sharepoint-iddemo.msappproxy.net/` a `https://sharepoint-iddemo.msappproxy.net/hybrid/`, los usuarios de la aplicación pasan directamente a la página principal personalizada.

   ```powershell
   $homepage = "https://sharepoint-iddemo.msappproxy.net/hybrid/"
   ```

1. Actualice la página principal.

   ```powershell
   Set-AzureADApplication -ObjectId $objguid -Homepage $homepage
   ```

1. Para confirmar que el cambio se realizó correctamente, vuelva a ejecutar este comando del paso 2.

   ```powershell
   Get-AzureADApplication -ObjectId $objguid | Format-List DisplayName, Homepage, ObjectId
   ```

   En nuestro ejemplo, la salida debería aparecer como se muestra a continuación:

   ```console
   DisplayName : SharePoint
   Homepage    : https://sharepoint-iddemo.msappproxy.net/hybrid/
   ObjectId    : 8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4
   ```

1. Reinicie la aplicación para confirmar que la página principal aparece como la primera pantalla, tal como se espera.

> [!NOTE]
> Los cambios realizados en la aplicación pueden restablecer la dirección URL de la página principal. Si la dirección URL de la página principal se restablece, repita los pasos de esta sección para volver a establecerlo de nuevo.

## <a name="next-steps"></a>Pasos siguientes

- [Habilitar el acceso remoto a SharePoint con el Proxy de aplicación de Azure AD](application-proxy-integrate-with-sharepoint-server.md)
- [Tutorial: Adición de una aplicación local para el acceso remoto mediante el proxy de aplicación en Azure Active Directory](application-proxy-add-on-premises-application.md)
