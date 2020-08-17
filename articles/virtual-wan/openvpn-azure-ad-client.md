---
title: 'VPN Gateway: cliente VPN para conexiones de punto a sitio con el protocolo OpenVPN: Autenticación de Azure AD'
description: Puede usar la P2S VPN para conectarse a la red virtual con la autenticación de Azure AD
services: vpn-gateway
author: kumudD
ms.service: virtual-wan
ms.topic: how-to
ms.date: 08/04/2020
ms.author: alzam
ms.openlocfilehash: 97620192fb645dd453d1479d1e755e87cf9afd5c
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87761185"
---
# <a name="configure-a-vpn-client-for-p2s-openvpn-protocol-connections-azure-ad-authentication"></a>Configuración de un cliente VPN para conexiones P2S de protocolo OpenVPN: Autenticación de Azure AD

Este artículo le ayuda a configurar un cliente VPN para conectarse a una red virtual mediante una VPN de punto a sitio y la autenticación Azure Active Directory. Antes de poder conectarse y autenticarse con Azure AD, primero debe configurar el inquilino de Azure AD. Para más información, consulte [Configurar un inquilino de Azure AD](openvpn-azure-ad-tenant.md).

> [!NOTE]
> La autenticación de Azure AD solo se admite para las conexiones de protocolo de OpenVPN®.
>

## <a name="working-with-client-profiles"></a><a name="profile"></a>Trabajar con perfiles de cliente

Para conectarse, tiene que descargar el cliente VPN de Azure y configurar un perfil de cliente VPN en todos los equipos que quieran conectarse a la red virtual. Puede crear un perfil de cliente en un equipo, exportarlo y, después, importarlo en otros equipos.

### <a name="to-download-the-azure-vpn-client"></a>Para descargar el cliente VPN de Azure

Use este [vínculo](https://go.microsoft.com/fwlink/?linkid=2117554) para descargar el cliente VPN de Azure. Asegúrese de que el cliente VPN de Azure tenga permiso para ejecutarse en segundo plano. Para comprobar o habilitar el permiso, siga los pasos a continuación:

1. Vaya a Inicio y, después, seleccione Configuración > Privacidad > Aplicaciones en segundo plano.
2. En Aplicaciones en segundo plano, asegúrese de que la opción **Permitir que las aplicaciones se ejecuten en segundo plano** esté activada.
3. En Elegir qué aplicaciones se pueden ejecutar en segundo plano, cambie la configuración del cliente VPN de Azure a **Activado**.

  ![permiso](./media/openvpn-azure-ad-client/backgroundpermission.png)

### <a name="to-create-a-certificate-based-client-profile"></a><a name="cert"></a>Para crear un perfil de cliente basado en certificados

Cuando trabaje con un perfil basado en certificados, asegúrese de que los certificados adecuados están instalados en el equipo cliente. Para más información acerca de los certificados, consulte [Instalar certificados de cliente](certificates-point-to-site.md).

  ![cert](./media/openvpn-azure-ad-client/create/create-cert1.jpg)

### <a name="to-create-a-radius-client-profile"></a><a name="radius"></a>Para crear un perfil de cliente RADIUS

  ![radio](./media/openvpn-azure-ad-client/create/create-radius1.jpg)
  
> [!NOTE]
> El secreto de servidor se puede exportar en el perfil de cliente de P2S VPN.  [Aquí](about-vpn-profile-download.md) encontrará instrucciones sobre cómo exportar un perfil de cliente.
>

### <a name="to-export-and-distribute-a-client-profile"></a><a name="export"></a>Para exportar y distribuir un perfil de cliente

Una vez que tenga un perfil de trabajo y necesite distribuirlo a otros usuarios, puede exportarlo mediante los siguientes pasos:

1. Resalte el perfil de cliente de VPN que quiere exportar, seleccione el **...** y,luego, seleccione **Exportar**.

    ![exportar](./media/openvpn-azure-ad-client/export/export1.jpg)

2. Seleccione la ubicación en la que desea guardar este perfil, deje el nombre de archivo tal cual y, a continuación, seleccione **Guardar** para guardar el archivo xml.

    ![exportar](./media/openvpn-azure-ad-client/export/export2.jpg)

### <a name="to-import-a-client-profile"></a><a name="import"></a>Para importar un perfil de cliente

1. En la página, seleccione **Importar**.

    ![importación](./media/openvpn-azure-ad-client/import/import1.jpg)

2. Busque el archivo xml de perfil y selecciónelo. Con el archivo seleccionado, seleccione **Abrir**.

    ![importación](./media/openvpn-azure-ad-client/import/import2.jpg)

3. Especifique el nombre del perfil y seleccione **Guardar**.

    ![importación](./media/openvpn-azure-ad-client/import/import3.jpg)

4. Seleccione **Conectar** para conectarse a la VPN.

    ![importación](./media/openvpn-azure-ad-client/import/import4.jpg)

5. Una vez conectado, el icono se volverá verde y dirá**Conectado**.

    ![importación](./media/openvpn-azure-ad-client/import/import5.jpg)

### <a name="to-delete-a-client-profile"></a><a name="delete"></a>Para eliminar un perfil de cliente

1. Seleccione los puntos suspensivos junto al perfil de cliente que desea eliminar. Después, seleccione **Quitar**.

    ![delete](./media/openvpn-azure-ad-client/delete/delete1.jpg)

2. Seleccione **Quitar** para eliminar.

    ![delete](./media/openvpn-azure-ad-client/delete/delete2.jpg)

## <a name="create-a-connection"></a><a name="connection"></a>Crear una conexión

1. En la página, seleccione **+** , después **+ Agregar**.

    ![conexión](./media/openvpn-azure-ad-client/create/create1.jpg)

2. Rellene la información de conexión. Si no está seguro de los valores, póngase en contacto con el administrador. Después de rellenar los valores, seleccione **Guardar**.

    ![conexión](./media/openvpn-azure-ad-client/create/create2.jpg)

3. Seleccione **Conectar** para conectarse a la VPN.

    ![conexión](./media/openvpn-azure-ad-client/create/create3.jpg)

4. Seleccione las credenciales adecuadas y, después, seleccione **Continuar**.

    ![conexión](./media/openvpn-azure-ad-client/create/create4.jpg)

5. Una vez que se haya conectado correctamente, el icono se volverá verde y dirá **Conectado**.

    ![conexión](./media/openvpn-azure-ad-client/create/create5.jpg)

### <a name="to-connect-automatically"></a><a name="autoconnect"></a>Para conectarse automáticamente

Estos pasos le ayudarán a configurar la conexión para que se conecte automáticamente con Always-on.

1. En la página principal del cliente VPN, seleccione **Configuración de VPN**.

    ![auto](./media/openvpn-azure-ad-client/auto/auto1.jpg)

2. Seleccione **Sí** en el cuadro de diálogo cambiar aplicaciones.

    ![auto](./media/openvpn-azure-ad-client/auto/auto2.jpg)

3. Asegúrese de que la conexión que quiere establecer no está conectada y, luego, resalte el perfil y active la casilla **Conectar automáticamente**.

    ![auto](./media/openvpn-azure-ad-client/auto/auto3.jpg)

4. Seleccione **Conectar** para iniciar la conexión VPN.

    ![auto](./media/openvpn-azure-ad-client/auto/auto4.jpg)

## <a name="diagnose-connection-issues"></a><a name="diagnose"></a>Diagnóstico de problemas de conexión

1. Para diagnosticar problemas de conexión, puede usar la herramienta **Diagnosticar**. Seleccione el **...** junto a la conexión VPN que quiere diagnosticar para que se muestre el menú. Después, seleccione **Diagnosticar**.

    ![diagnóstico](./media/openvpn-azure-ad-client/diagnose/diagnose1.jpg)

2. En la página **Propiedades de conexión**, seleccione **Ejecutar diagnóstico**.

    ![diagnóstico](./media/openvpn-azure-ad-client/diagnose/diagnose2.jpg)

3. Inicie sesión con sus credenciales.

    ![diagnóstico](./media/openvpn-azure-ad-client/diagnose/diagnose3.jpg)

4. Ver los resultados del diagnóstico.

    ![diagnóstico](./media/openvpn-azure-ad-client/diagnose/diagnose4.jpg)

## <a name="faq"></a>Preguntas más frecuentes

### <a name="how-do-i-add-dns-suffixes-to-the-vpn-client"></a>¿Cómo agrego sufijos DNS al cliente VPN?

Puede modificar el archivo XML de perfil descargado y agregar las etiquetas **\<dnssuffixes>\<dnssufix> \</dnssufix>\</dnssuffixes>** .

```
<azvpnprofile>
<clientconfig>

    <dnssuffixes>
          <dnssuffix>.mycorp.com</dnssuffix>
          <dnssuffix>.xyz.com</dnssuffix>
          <dnssuffix>.etc.net</dnssuffix>
    </dnssuffixes>
    
</clientconfig>
</azvpnprofile>
```

### <a name="how-do-i-add-custom-dns-servers-to-the-vpn-client"></a>¿Cómo agrego servidores DNS personalizados al cliente VPN?

Puede modificar el archivo XML de perfil descargado y agregar las etiquetas **\<dnsservers>\<dnsserver> \</dnsserver>\</dnsservers>** .

```
<azvpnprofile>
<clientconfig>

    <dnsservers>
        <dnsserver>x.x.x.x</dnsserver>
        <dnsserver>y.y.y.y</dnsserver>
    </dnsservers>
    
</clientconfig>
</azvpnprofile>
```

> [!NOTE]
> El cliente OpenVPN de Azure AD usa las entradas de la Tabla de directivas de resolución de nombres (NRPT) de DNS, lo que significa que los servidores DNS no se mostrarán en la salida de `ipconfig /all`. Para confirmar la configuración de DNS en uso, consulte [Get-DnsClientNrptPolicy](https://docs.microsoft.com/powershell/module/dnsclient/get-dnsclientnrptpolicy?view=win10-ps) en PowerShell.
>

### <a name="how-do-i-add-custom-routes-to-the-vpn-client"></a>¿Cómo agrego rutas personalizadas al cliente VPN?

Puede modificar el archivo XML de perfil descargado y agregar las etiquetas **\<includeroutes>\<route>\<destination>\<mask> \</destination>\</mask>\</route>\</includeroutes>** .

```
<azvpnprofile>
<clientconfig>

    <includeroutes>
        <route>
            <destination>x.x.x.x</destination><mask>24</mask>
        </route>
    </includeroutes>
    
</clientconfig>
</azvpnprofile>
```
### <a name="how-do-i-direct-all-traffic-to-the-vpn-tunnel-force-tunnel"></a>¿Cómo dirijo todo el tráfico al túnel VPN (forzar túnel)?

Puede modificar el archivo XML de perfil descargado y agregar las etiquetas **\<includeroutes>\<route>\<destination>\<mask> \</destination>\</mask>\</route>\</includeroutes>** .

```
<azvpnprofile>
<clientconfig>

    <includeroutes>
        <route>
            <destination>0.0.0.0</destination><mask>1</mask>
        </route>
        <route>
            <destination>128.0.0.0</destination><mask>1</mask>
        </route>
    </includeroutes>
    
</clientconfig>
</azvpnprofile>
```

### <a name="how-do-i-block-exclude-routes-from-the-vpn-client"></a>¿Cómo bloqueo (excluyo) las rutas del cliente VPN?

Puede modificar el archivo XML de perfil descargado y agregar las etiquetas **\<excluderoutes>\<route>\<destination>\<mask> \</destination>\</mask>\</route>\</excluderoutes>** .

```
<azvpnprofile>
<clientconfig>

    <excluderoutes>
        <route>
            <destination>x.x.x.x</destination><mask>24</mask>
        </route>
    </excluderoutes>
    
</clientconfig>
</azvpnprofile>
```
### <a name="can-i-import-the-profile-from-a-command-line-prompt"></a>¿Se puede importar el perfil desde un símbolo de la línea de comandos?

Para importar el perfil desde un símbolo de la línea de comandos, coloque el archivo **azurevpnconfig. XML** descargado en la carpeta **%userprofile%\AppData\Local\Packages\Microsoft. AzureVpn_8wekyb3d8bbwe \LocalState** y ejecute el siguiente comando:

```
azurevpn -i azurevpnconfig.xml 
```
para forzar la importación, use también el modificador **-f**


## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte [crear un inquilino de Azure Active Directory para conexiones VPN abiertas de P2S que usan Autenticación de Azure AD](openvpn-azure-ad-tenant.md).
