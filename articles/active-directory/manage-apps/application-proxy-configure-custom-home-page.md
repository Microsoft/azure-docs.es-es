---
title: Establecimiento de una página principal personalizada para aplicaciones publicadas mediante el Proxy de aplicación de Azure AD | Microsoft Docs
description: Se explican los conceptos básicos acerca de los conectores del Proxy de aplicación de Azure AD.
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
ms.date: 04/17/2019
ms.author: mimart
ms.reviewer: harshja
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3fa5c5638da390f4416afc9f4bd9c5d58c34cea8
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65825574"
---
# <a name="set-a-custom-home-page-for-published-apps-by-using-azure-ad-application-proxy"></a>Establecimiento de una página principal personalizada para aplicaciones publicadas mediante el Proxy de aplicación de Azure AD

En este artículo se describe cómo configurar una aplicación para que solicite a un usuario a una página principal personalizada, que pueden diferir dependiendo de si son internos o externos. Cuando se publica una aplicación con el Proxy de aplicación, establecer una dirección URL interna, pero a veces, que no es la página de que un usuario debería ver en primer lugar. Establezca una página principal personalizada para que un usuario obtiene la página correcta cuando accedan a la aplicación. El usuario verá la página principal personalizada que establezca, independientemente de si acceden a la aplicación desde el Panel de acceso de Azure Active Directory o el iniciador de aplicaciones de Office 365.

Cuando un usuario inicia la aplicación, le dirige de forma predeterminada para la dirección URL del dominio raíz de la aplicación publicada. La página de aterrizaje normalmente se establece como la dirección URL de la página principal. Usar el módulo de PowerShell de Azure AD para definir una dirección URL de la página principal personalizada cuando desee que un usuario de la aplicación lleguen a una página específica dentro de la aplicación.

Un escenario de continuación que explica por qué su empresa establecería una página principal personalizada, y por qué es diferente según el tipo de usuario:

- Dado que tiene otros recursos (por ejemplo, imágenes) que el Proxy de aplicación necesita obtener acceso en el nivel superior de la estructura de carpetas, publica la aplicación con `https://ExpenseApp` como la dirección URL interna.
- Sin embargo, dentro de la red corporativa, un usuario visita `https://ExpenseApp/login/login.aspx` para iniciar sesión y acceder a la aplicación.
- La URL externa predeterminada es `https://ExpenseApp-contoso.msappproxy.net`, que no lleva a un usuario externo a la página de inicio de sesión.
- Desea establecer `https://ExpenseApp-contoso.msappproxy.net/login/login.aspx` como la dirección URL de la página principal externo en su lugar, por lo que un usuario externo ve la página de inicio de sesión en primer lugar.

>[!NOTE]
>Al proporcionar a los usuarios acceso a las aplicaciones publicadas, estas aparecen en el [panel de acceso de Azure AD](../user-help/my-apps-portal-end-user-access.md) y el [iniciador de aplicaciones de Office 365](https://www.microsoft.com/microsoft-365/blog/2016/09/27/introducing-the-new-office-365-app-launcher/).

## <a name="before-you-start"></a>Antes de comenzar

Antes de establecer la dirección URL de la página principal tenga en cuenta los siguientes requisitos:

- La ruta de acceso que especifique debe ser una ruta de acceso de subdominio de la dirección URL raíz del dominio.

  Por ejemplo, si la dirección URL raíz del dominio es `https://apps.contoso.com/app1/`, la dirección URL de página principal que configure debe empezar por `https://apps.contoso.com/app1/`.

- Si realiza un cambio en la aplicación publicada, este podría restablecer el valor de la dirección URL de la página principal. Si actualiza la aplicación en el futuro, debe volver a comprobar y, si es necesario, actualizar la dirección URL de la página principal.

Puede cambiar la página principal externa o interna mediante Azure portal o mediante PowerShell.

## <a name="change-the-home-page-in-the-azure-portal"></a>Vaya a la página principal de Azure Portal.

Para cambiar las páginas principales de externas e internas de la aplicación a través del portal de Azure AD, siga estos pasos:

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com/). Aparece el panel del centro de administración de Azure Active Directory.
2. En la barra lateral, seleccione **Azure Active Directory**. Aparece la página de información general de Azure AD.
3. En la barra lateral de información general, seleccione **registros de aplicaciones**. Aparece la lista de aplicaciones registradas.
4. Elija la aplicación en la lista. Aparece una página que muestra los detalles de la aplicación registrada.
5. Seleccione el vínculo situado bajo **URI de redirección**, que muestra el número de URI de redireccionamiento para la web y los tipos de cliente público. Aparece la página de autenticación para la aplicación registrada.
6. En la última fila de la **URI de redirección** de tabla, establezca el **tipo** columna **cliente público (mobile y desktop)** y en el **URI de redireccionamiento**columna, escriba la dirección URL interna que desea usar. Aparece una nueva fila vacía debajo de la fila que acaba de modificar.
7. En la nueva fila, establezca el **tipo** columna **Web**y en el **URI de redireccionamiento** columna, escriba la dirección URL externa que desea usar.
8. Seleccione si desea eliminar las filas existentes de URI de redireccionamiento, el **eliminar** icono (una lata de elementos no utilizados) junto a cada fila no deseada.
9. Seleccione **Guardar**.

## <a name="change-the-home-page-with-powershell"></a>Cambio de la página principal con PowerShell

Para configurar la página principal de una aplicación mediante PowerShell, deberá:

1. Instale el módulo de PowerShell de Azure AD.
2. Buscar el valor de ObjectId de la aplicación.
3. Actualizar la dirección URL de la página principal de la aplicación mediante los comandos de PowerShell.

### <a name="install-the-azure-ad-powershell-module"></a>Instalación del módulo de PowerShell de Azure AD

Para poder definir la dirección URL de la página principal personalizada mediante PowerShell, instale el módulo de Azure AD PowerShell. Puede descargar este paquete desde la [Galería de PowerShell](https://www.powershellgallery.com/packages/AzureAD/2.0.2.16), que utiliza el punto de conexión de Graph API.

Para instalar el paquete, siga estos pasos:

1. Abra una ventana de PowerShell estándar y luego ejecute el siguiente comando:

   ```powershell
   Install-Module -Name AzureAD
   ```

    Si está ejecutando el comando como no administrador, use la opción `-scope currentuser`.

2. Durante la instalación, seleccione **Y** para instalar dos paquetes de Nuget.org. Se requieren ambos paquetes.

### <a name="find-the-objectid-of-the-app"></a>Busque el ObjectId de la aplicación

Obtenga el ObjectId de la aplicación mediante la búsqueda de la aplicación por su nombre para mostrar o la página principal.

1. En la misma ventana de PowerShell, importe el módulo Azure AD.

   ```powershell
   Import-Module AzureAD
   ```

2. Inicie sesión como administrador de inquilinos en el módulo de Azure AD.

   ```powershell
   Connect-AzureAD
   ```

3. Busque la aplicación. Este ejemplo usa PowerShell para buscar el valor de ObjectId mediante la búsqueda de la aplicación con un nombre para mostrar de `SharePoint`.

   ```powershell
   Get-AzureADApplication | Where-Object { $_.DisplayName -eq "SharePoint" } | Format-List DisplayName, Homepage, ObjectId
   ```

   Debe obtener un resultado similar al que se muestra aquí. Copie el GUID de ObjectId para usar en la sección siguiente.

   ```console
   DisplayName : SharePoint
   Homepage    : https://sharepoint-iddemo.msappproxy.net/
   ObjectId    : 8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4
   ```

   Como alternativa, simplemente podría extraer de la lista de todas las aplicaciones, busque en la lista de la aplicación con un nombre de pantalla específico o la página principal y copie el ObjectId de la aplicación una vez que se encuentra la aplicación.

   ```powershell
   Get-AzureADApplication | Format-List DisplayName, Homepage, ObjectId
   ```

### <a name="update-the-home-page-url"></a>Actualización de la dirección URL de la página principal

Cree la dirección URL de la página principal y actualiza la aplicación con ese valor. Seguir usando la misma ventana de PowerShell, o si usa una nueva ventana de PowerShell, inicie sesión en el módulo de Azure AD con `Connect-AzureAD`. A continuación, siga estos pasos:

1. Cree una variable que contenga el valor de ObjectId que copió en la sección anterior. (Reemplace el valor de ObjectId utilizado para en este ejemplo de SharePoint con el valor de ObjectId de la aplicación).

   ```powershell
   $objguid = "8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4"
   ```

2. Confirme que tiene la aplicación correcta, ejecute el comando siguiente. La salida debería ser idéntica a la salida se vio en la sección anterior ([busque el ObjectId de la aplicación](#find-the-objectid-of-the-app)).

   ```powershell
   Get-AzureADApplication -ObjectId $objguid | Format-List DisplayName, Homepage, ObjectId
   ```

3. Cree un objeto de aplicación en blanco para que contenga los cambios que desea realizar.

   ```powershell
   $appnew = New-Object "Microsoft.Open.AzureAD.Model.Application"
   ```

4. Establezca la dirección URL de la página principal en el valor que quiera. El valor debe ser una ruta de acceso de subdominio de la aplicación publicada. Por ejemplo, si cambia la dirección URL de página principal de `https://sharepoint-iddemo.msappproxy.net/` a `https://sharepoint-iddemo.msappproxy.net/hybrid/`, los usuarios de la aplicación pasan directamente a la página principal personalizada.

   ```powershell
   $homepage = "https://sharepoint-iddemo.msappproxy.net/hybrid/"
   ```

5. Realizar la actualización de la página principal.

   ```powershell
   Set-AzureADApplication -ObjectId $objguid -Homepage $homepage
   ```

6. Para confirmar que el cambio fue correcto, ejecute el siguiente comando desde el paso 2 de nuevo.

   ```powershell
   Get-AzureADApplication -ObjectId $objguid | Format-List DisplayName, Homepage, ObjectId
   ```

   En nuestro ejemplo, la salida debería aparecer ahora como sigue:

   ```console
   DisplayName : SharePoint
   Homepage    : https://sharepoint-iddemo.msappproxy.net/hybrid/
   ObjectId    : 8af89bfa-eac6-40b0-8a13-c2c4e3ee22a4
   ```

7. Reinicie la aplicación para confirmar que la página principal aparece como la primera pantalla, según lo previsto.

>[!NOTE]
>Los cambios realizados en la aplicación pueden restablecer la dirección URL de la página principal. Si la dirección URL de la página principal se restablece, repita los pasos de esta sección para volver a establecerlo de nuevo.

## <a name="next-steps"></a>Pasos siguientes

- [Habilitar el acceso remoto a SharePoint con el Proxy de aplicación de Azure AD](application-proxy-integrate-with-sharepoint-server.md)
- [Tutorial: Agregar una aplicación de forma local para el acceso remoto a través del Proxy de aplicación en Azure Active Directory](application-proxy-add-on-premises-application.md)