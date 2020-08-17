---
title: 'Inquilino de Azure AD para las conexiones VPN de usuario: Autenticación de Azure AD'
description: Puede usar la VPN de usuario de WAN virtual de Azure (punto a sitio) para conectarse a la red virtual mediante la autenticación de Azure AD
titleSuffix: Azure Virtual WAN
services: virtual-wan
author: kumudD
ms.service: virtual-wan
ms.topic: how-to
ms.date: 03/19/2020
ms.author: alzam
ms.openlocfilehash: e88437dc03772348ebbe0d179afc7fd4ddd24bd9
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86507563"
---
# <a name="prepare-azure-active-directory-tenant-for-user-vpn-openvpn-protocol-connections"></a>Preparación de un inquilino de Azure Active Directory para conexiones de VPN de usuario del protocolo OpenVPN

Al conectarse a su centro virtual mediante el protocolo IKEv2, puede usar la autenticación basada en certificados o la autenticación RADIUS. Sin embargo, cuando use el protocolo OpenVPN, también puede usar la autenticación de Azure Active Directory. Este artículo le ayuda a configurar un inquilino de Azure AD para la VPN de usuario de WAN virtual (punto a sitio) mediante la autenticación de OpenVPN.

> [!NOTE]
> La autenticación de Azure AD solo se admite para las conexiones de protocolo de OpenVPN&reg;.
>

## <a name="1-create-the-azure-ad-tenant"></a><a name="tenant"></a>1. Cree un inquilino Azure AD

Compruebe que tiene un inquilino de Azure AD. Si no tiene ningún inquilino de Azure AD, para crearlo siga los pasos del artículo [Creación de un nuevo inquilino](../active-directory/fundamentals/active-directory-access-create-new-tenant.md):

* Nombre de la organización
* Nombre de dominio inicial

Ejemplo:

   ![Nuevo inquilino Azure AD](./media/openvpn-create-azure-ad-tenant/newtenant.png)

## <a name="2-create-azure-ad-tenant-users"></a><a name="users"></a>2. Crear usuario de inquilino Azure AD

A continuación, cree dos cuentas de usuario en el inquilino de Azure AD recién creado, una cuenta de administrador global y una cuenta de usuario. La cuenta de usuario se puede usar para probar la autenticación de OpenVPN y la cuenta de administrador global se usará para conceder consentimiento al registro de la aplicación VPN de Azure. Después de crear una cuenta de usuario de Azure AD, asigne un **rol de directorio** al usuario para delegar permisos administrativos.

Siga los pasos descritos en [este artículo](../active-directory/fundamentals/add-users-azure-active-directory.md) para crear los dos usuarios para su inquilino de Azure AD. Asegúrese de cambiar el **rol de directorio** de una de las cuentas creadas a **administrador global**.

## <a name="3-grant-consent-to-the-azure-vpn-app-registration"></a><a name="enable-authentication"></a>3. Concesión de consentimiento al registro de la aplicación VPN de Azure

1. Inicie sesión en Azure Portal como un usuario al que se le ha asignado el rol de **administrador global**.

2. A continuación, conceda el consentimiento del administrador a su organización; de este modo, la aplicación de VPN de Azure podrá iniciar sesión y leer los perfiles de usuario. Copie y pegue la dirección URL que pertenece a la ubicación de implementación en la barra de direcciones del explorador:

    Público

    ```
    https://login.microsoftonline.com/common/oauth2/authorize?client_id=41b23e61-6c1e-4545-b367-cd054e0ed4b4&response_type=code&redirect_uri=https://portal.azure.com&nonce=1234&prompt=admin_consent
    ````

    Azure Government

    ```
    https://login-us.microsoftonline.com/common/oauth2/authorize?client_id=51bb15d4-3a4f-4ebf-9dca-40096fe32426&response_type=code&redirect_uri=https://portal.azure.us&nonce=1234&prompt=admin_consent
    ````

    Microsoft Cloud Germany

    ```
    https://login-us.microsoftonline.de/common/oauth2/authorize?client_id=538ee9e6-310a-468d-afef-ea97365856a9&response_type=code&redirect_uri=https://portal.microsoftazure.de&nonce=1234&prompt=admin_consent
    ````

    Azure China 21Vianet

    ```
    https://https://login.chinacloudapi.cn/common/oauth2/authorize?client_id=49f817b6-84ae-4cc0-928c-73f27289b3aa&response_type=code&redirect_uri=https://portal.azure.cn&nonce=1234&prompt=admin_consent
    ```

3. Seleccione la cuenta del **Administrador global** si se le solicita.

    ![Identificador de directorio](./media/openvpn-create-azure-ad-tenant/pick.png)

4. Seleccione **Aceptar** cuando se le solicite.

    ![Accept](./media/openvpn-create-azure-ad-tenant/accept.jpg)

5. En su instancia de Azure AD, en **Aplicaciones empresariales**, ahora debería mostrarse la **VPN de Azure** en la lista.

    ![VPN de Azure](./media/openvpn-create-azure-ad-tenant/azurevpn.png)

## <a name="next-steps"></a>Pasos siguientes

Para conectarse a las redes virtuales mediante la autenticación de Azure AD, debe crear una configuración de VPN de usuario y asociarla a un centro virtual. Consulte [Configuración de la autenticación de Azure AD para la conexión de punto a sitio con Azure](virtual-wan-point-to-site-azure-ad.md).
