---
title: 'VPN Gateway: inquilino de Azure AD para las conexiones VPN de punto a sitio: Autenticación de Azure AD'
description: Puede usar la P2S VPN para conectarse a la red virtual con la autenticación de Azure AD
services: vpn-gateway
author: kumudD
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 04/17/2020
ms.author: alzam
ms.openlocfilehash: 74999b2bf1a34e3c7b8190dd04206b2b541c465f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087040"
---
# <a name="create-an-azure-active-directory-tenant-for-p2s-openvpn-protocol-connections"></a>Creación de un inquilino de Azure Active Directory para conexiones del protocolo P2S OpenVPN

Al conectarse a la red virtual, puede usar la autenticación basada en certificados o la autenticación RADIUS. Sin embargo, cuando use el protocolo de VPN abierto, también puede usar la autenticación de Azure Active Directory. Este artículo le ayuda a configurar un inquilino de Azure AD para la autenticación de P2S VPN abierta.

> [!NOTE]
> La autenticación de Azure AD solo se admite para las conexiones de protocolo de OpenVPN®.
>


## <a name="1-verify-azure-ad-tenant"></a><a name="tenant"></a>1. Comprobación del inquilino de Azure AD

Compruebe que tiene un inquilino de Azure AD. Si no tiene ningún inquilino de Azure AD, para crearlo siga los pasos del artículo [Creación de un nuevo inquilino](../active-directory/fundamentals/active-directory-access-create-new-tenant.md):

* Nombre organizativo
* Nombre de dominio inicial

Ejemplo:

   ![Nuevo inquilino Azure AD](./media/openvpn-create-azure-ad-tenant/newtenant.png)

## <a name="2-create-azure-ad-tenant-users"></a><a name="users"></a>2. Crear usuario de inquilino Azure AD

El inquilino de Azure AD necesita las siguientes cuentas: una cuenta de administrador global y una cuenta de usuario maestro. La cuenta de usuario principal se usa como cuenta de inserción maestra (cuenta de servicio). Al crear una cuenta de usuario de inquilino Azure AD, ajuste el rol de directorio para el tipo de usuario que desea crear.

Siga los pasos descritos en [este artículo](../active-directory/fundamentals/add-users-azure-active-directory.md) para crear al menos dos usuarios para su inquilino de Azure AD. Asegúrese de cambiar el **rol de directorio** para crear los tipos de cuenta:

* Administrador global
* Usuario

## <a name="3-enable-azure-ad-authentication-on-the-vpn-gateway"></a><a name="enable-authentication"></a>3. Habilitación de la autenticación de Azure AD

1. Busque el ID. de directorio del directorio que desea utilizar para la autenticación. Aparece en la sección propiedades de la página Active Directory.

    ![Identificador de directorio](./media/openvpn-create-azure-ad-tenant/directory-id.png)

2. Copie el ID. del directorio.

3. Inicie sesión en el Azure Portal como un usuario al que se le ha asignado el rol de **administrador global**.

4. Después, ceda el consentimiento del administrador. Copie y pegue la dirección URL que pertenece a la ubicación de implementación en la barra de direcciones del explorador:

    Público

    ```
    https://login.microsoftonline.com/common/oauth2/authorize?client_id=41b23e61-6c1e-4545-b367-cd054e0ed4b4&response_type=code&redirect_uri=https://portal.azure.com&nonce=1234&prompt=admin_consent
    ````

    Azure Government

    ```
   https://login.microsoftonline.us/common/oauth2/authorize?client_id=51bb15d4-3a4f-4ebf-9dca-40096fe32426&response_type=code&redirect_uri=https://portal.azure.us&nonce=1234&prompt=admin_consent
    ````

    Microsoft Cloud Germany

    ```
    https://login-us.microsoftonline.de/common/oauth2/authorize?client_id=538ee9e6-310a-468d-afef-ea97365856a9&response_type=code&redirect_uri=https://portal.microsoftazure.de&nonce=1234&prompt=admin_consent
    ````

    Azure China 21Vianet

    ```
    https://login.chinacloudapi.cn/common/oauth2/authorize?client_id=49f817b6-84ae-4cc0-928c-73f27289b3aa&response_type=code&redirect_uri=https://portal.azure.cn&nonce=1234&prompt=admin_consent
    ```

5. Seleccione la cuenta del **Administrador global** si se le solicita.

    ![Identificador de directorio](./media/openvpn-create-azure-ad-tenant/pick.png)

6. Seleccione **Aceptar** cuando se le solicite.

    ![Accept](./media/openvpn-create-azure-ad-tenant/accept.jpg)

7. En el Azure AD, en **Aplicaciones empresariales**, verá la **VPN de Azure** en la lista.

    ![VPN de Azure](./media/openvpn-create-azure-ad-tenant/azurevpn.png)
    
8. Si no dispone de un entorno de punto a sitio funcionando, siga las instrucciones para crear uno. Consulte [Creación de una VPN de punto a sitio](vpn-gateway-howto-point-to-site-resource-manager-portal.md) para crear una puerta de enlace de VPN de punto a sitio. 

    > [!IMPORTANT]
    > La SKU de nivel Básico no es compatible con OpenVPN.

9. Habilite la autenticación de Azure AD en la puerta de enlace de VPN. Para ello, navegue hasta **Configuración de punto a sitio** y seleccione **OpenVPN (SSL)** como **Tipo de túnel**. Seleccione **Azure Active Directory** como **Tipo de autenticación**, a continuación, rellene la información de la sección **Azure Active Directory**.

    ![VPN de Azure](./media/openvpn-create-azure-ad-tenant/azure-ad-auth-portal.png)


   > [!NOTE]
   > Asegúrese de incluir la barra diagonal al final del valor `AadIssuerUri`. De lo contrario, se puede producir un error en la conexión.

10. Para crear y descargar el perfil, haga clic en el vínculo **Descargar cliente VPN**.

11. Extraiga el archivo zip descargado.

12. Busque la carpeta "AzureVPN" descomprimida.

13. Anote la ubicación del archivo "azurevpnconfig. xml". Azurevpnconfig. xml contiene la configuración de la conexión VPN y se puede importar directamente en la aplicación del cliente VPN Azure. También puede distribuir este archivo a todos los usuarios que necesiten conectarse a través del correo electrónico u otros medios. El usuario necesitará credenciales de Azure AD válidas para conectarse correctamente.

## <a name="next-steps"></a>Pasos siguientes

Para conectarse a la red virtual, debe crear y configurar un perfil de cliente de VPN. Consulte [Configurar un cliente VPN para conexiones P2S VPN](openvpn-azure-ad-client.md).
