---
title: 'Virtual WAN: inquilino de Azure AD para distintos grupos de usuarios: Autenticación de Azure AD'
description: Puede usar la P2S VPN para conectarse a la red virtual con la autenticación de Azure AD
services: virtual-wan
author: kumudD
ms.service: virtual-wan
ms.topic: how-to
ms.date: 03/19/2020
ms.author: alzam
ms.openlocfilehash: 5ca57ccc40669a607cd0541dc738e3a3eacf3e88
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86507699"
---
# <a name="create-an-azure-active-directory-ad-tenant-for-p2s-openvpn-protocol-connections"></a>Creación de un inquilino de Azure Active Directory (AD) para conexiones del protocolo P2S OpenVPN

Al conectarse a la red virtual, puede usar la autenticación basada en certificados o la autenticación RADIUS. Sin embargo, cuando use el protocolo de VPN abierto, también puede usar la autenticación de Azure Active Directory. Si quiere que un conjunto de usuarios diferente pueda conectarse a diferentes puertas de enlace, puede registrar varias aplicaciones en AD y vincularlas a diferentes puertas de enlace.

Este artículo le ayuda a configurar un inquilino de Azure AD para la autenticación de OpenVPN de P2S y a crear y registrar varias aplicaciones en Azure AD a fin de permitir un acceso distinto a los diferentes usuarios y grupos.

> [!NOTE]
> La autenticación de Azure AD solo se admite para las conexiones de protocolo de OpenVPN&reg;.
>

[!INCLUDE [create](../../includes/openvpn-azure-ad-tenant-multi-app.md)]

## <a name="6-create-a-new-p2s-configuration"></a><a name="site"></a>6. Creación de una nueva configuración de P2S

Una configuración de P2S define los parámetros para conectarse a los clientes remotos.

1. Establezca las siguientes variables y reemplace los valores según sea necesario para su entorno.

   ```azurepowershell-interactive
   $aadAudience = "00000000-abcd-abcd-abcd-999999999999"
   $aadIssuer = "https://sts.windows.net/00000000-abcd-abcd-abcd-999999999999/"
   $aadTenant = "https://login.microsoftonline.com/00000000-abcd-abcd-abcd-999999999999"    
   ```

2. Ejecute los siguientes comandos para crear la configuración:

   ```azurepowershell-interactive
   $aadConfig = New-AzVpnServerConfiguration -ResourceGroupName <ResourceGroup> -Name newAADConfig -VpnProtocol OpenVPN -VpnAuthenticationType AAD -AadTenant $aadTenant -AadIssuer $aadIssuer -AadAudience $aadAudience -Location westcentralus
   ```

   > [!NOTE]
   > No use el id. de la aplicación del cliente VPN de Azure en los comandos anteriores: Concederá acceso a todos los usuarios a la puerta de enlace. Use el identificador de las aplicaciones que registró.

## <a name="7-edit-hub-assignment"></a><a name="hub"></a>7. Edición de la asignación del concentrador

1. Vaya a la hoja **Centros de conectividad** de la red virtual WAN.

2. Seleccione el concentrador al que desea asociar la configuración del servidor VPN y haga clic en los puntos suspensivos (...).

    ![nuevo sitio](media/openvpn-azure-ad-tenant-multi-app/p2s4.jpg)

3. Haga clic en **Editar centro de conectividad virtual**.

4. Active la casilla **Incluir puerta de enlace de punto a sitio** y elija la **unidad de escalado de puerta de enlace** que quiera.

    ![nuevo sitio](media/openvpn-azure-ad-tenant-multi-app/p2s2.jpg)

5. Especifique el **Grupo de direcciones** del que se asignarán direcciones IP a los clientes VPN.

6. Haga clic en **Confirmar**.

7. La operación puede tardar un máximo de 30 minutos en completarse.

## <a name="8-download-vpn-profile"></a><a name="device"></a>8. Descarga del perfil de VPN

Use el perfil de VPN para configurar los clientes.

1. En la página de su red WAN virtual, haga clic en **Configuraciones de VPN de usuario**.

2. En la parte superior de la página, haga clic en **Download user VPN config** (Descargar configuración de VPN de usuario).

3. Una vez el archivo se haya terminado de crear, puede hacer clic en el vínculo para descargarlo.

4. Use el archivo de perfil para configurar los clientes de VPN.

5. Extraiga el archivo zip descargado.

6. Busque la carpeta "AzureVPN" descomprimida.

7. Anote la ubicación del archivo "azurevpnconfig.xml". Azurevpnconfig. xml contiene la configuración de la conexión VPN y se puede importar directamente en la aplicación del cliente VPN Azure. También puede distribuir este archivo a todos los usuarios que necesiten conectarse a través del correo electrónico u otros medios. El usuario necesitará credenciales de Azure AD válidas para conectarse correctamente.

## <a name="9-configure-user-vpn-clients"></a>9. Configuración de clientes VPN de usuario

Para conectarse, debe descargar el cliente de VPN de Azure e importar el perfil de cliente de VPN que descargó en los pasos anteriores en todos los equipos que quieran conectarse a la red virtual.

> [!NOTE]
> La autenticación de Azure AD solo se admite para las conexiones de protocolo de OpenVPN&reg;.
>

#### <a name="to-download-the-azure-vpn-client"></a>Para descargar el cliente VPN de Azure

Use este [vínculo](https://go.microsoft.com/fwlink/?linkid=2117554) para descargar el cliente VPN de Azure.

#### <a name="to-import-a-client-profile"></a><a name="import"></a>Para importar un perfil de cliente

1. En la página, seleccione **Importar**.

    ![importación](./media/openvpn-azure-ad-tenant-multi-app/import/import1.jpg)

2. Busque el archivo xml de perfil y selecciónelo. Con el archivo seleccionado, seleccione **Abrir**.

    ![importación](./media/openvpn-azure-ad-tenant-multi-app/import/import2.jpg)

3. Especifique el nombre del perfil y seleccione **Guardar**.

    ![importación](./media/openvpn-azure-ad-tenant-multi-app/import/import3.jpg)

4. Seleccione **Conectar** para conectarse a la VPN.

    ![importación](./media/openvpn-azure-ad-tenant-multi-app/import/import4.jpg)

5. Una vez conectado, el icono se volverá verde y dirá**Conectado**.

    ![importación](./media/openvpn-azure-ad-tenant-multi-app/import/import5.jpg)

#### <a name="to-delete-a-client-profile"></a><a name="delete"></a>Para eliminar un perfil de cliente

1. Seleccione los puntos suspensivos junto al perfil de cliente que quiera eliminar. Después, seleccione **Quitar**.

    ![delete](./media/openvpn-azure-ad-tenant-multi-app/delete/delete1.jpg)

2. Seleccione **Quitar** para eliminar.

    ![delete](./media/openvpn-azure-ad-tenant-multi-app/delete/delete2.jpg)

#### <a name="to-diagnose-connection-issues"></a><a name="diagnose"></a>Para diagnosticar problemas de conexión

1. Para diagnosticar problemas de conexión, puede usar la herramienta **Diagnosticar**. Seleccione los puntos suspensivos (...) junto a la conexión de VPN que desea diagnosticar para que se muestre el menú. Después, seleccione **Diagnosticar**.

    ![diagnóstico](./media/openvpn-azure-ad-tenant-multi-app/diagnose/diagnose1.jpg)

2. En la página **Propiedades de conexión**, seleccione **Ejecutar diagnóstico**.

    ![diagnóstico](./media/openvpn-azure-ad-tenant-multi-app/diagnose/diagnose2.jpg)

3. Inicie sesión con sus credenciales.

    ![diagnóstico](./media/openvpn-azure-ad-tenant-multi-app/diagnose/diagnose3.jpg)

4. Ver los resultados del diagnóstico.

    ![diagnóstico](./media/openvpn-azure-ad-tenant-multi-app/diagnose/diagnose4.jpg)

## <a name="10-view-your-virtual-wan"></a><a name="viewwan"></a>10. Visualizar la instancia de Virtual WAN

1. Vaya a la instancia de Virtual WAN.

2. En la página Información general, cada punto del mapa representa un concentrador.

3. En la sección de concentradores y conexiones, puede ver estado del concentrador, sitio, región, estado de la conexión VPN y bytes de entrada y salida.

## <a name="clean-up-resources"></a><a name="cleanup"></a>Limpieza de recursos

Cuando ya no necesite estos recursos, puede usar [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) para quitar el grupo de recursos y todos los recursos que contiene. Reemplace "myResourceGroup" con el nombre del grupo de recursos y ejecute el siguiente comando de PowerShell:

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Virtual WAN, consulte la página [Introducción a Virtual WAN](virtual-wan-about.md).
