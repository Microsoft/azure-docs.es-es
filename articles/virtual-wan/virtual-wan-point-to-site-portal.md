---
title: Uso de Azure Virtual WAN para crear una conexión de sitio a sitio a Azure | Microsoft Docs
description: En este tutorial, aprenda a usar Azure Virtual WAN para crear una conexión VPN de punto a sitio a Azure.
services: virtual-wan
author: kumudD
ms.service: virtual-wan
ms.topic: tutorial
ms.date: 06/29/2020
ms.author: alzam
ms.openlocfilehash: 27bb252b857fea2548e89471adcedcbd38b9f8ee
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077358"
---
# <a name="tutorial-create-a-user-vpn-connection-using-azure-virtual-wan"></a>Tutorial: Creación de una conexión VPN de usuario mediante Azure Virtual WAN

Este tutorial muestra cómo usar Virtual WAN para conectarse a los recursos de Azure a través de una conexión VPN de OpenVPN o IPsec/IKE (IKEv2). Este tipo de conexión requiere que se configure un cliente en el equipo cliente. Para más información sobre Virtual WAN, consulte la [Introducción a Virtual WAN](virtual-wan-about.md)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una red de área extensa (WAN)
> * Creación de una configuración de P2S
> * Crear un concentrador
> * Especificación de los servidores DNS
> * Descarga de un perfil de cliente VPN
> * Visualizar la instancia de Virtual WAN

![Diagrama de Virtual WAN](./media/virtual-wan-about/virtualwanp2s.png)

## <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar con la configuración, compruebe que se cumplen los criterios siguientes:

* Tiene una red virtual a la que quiere conectarse. Compruebe que ninguna de las subredes de sus redes locales se superpone a las redes virtuales a las que quiere conectarse. Para crear una red virtual en Azure Portal, consulte este [Inicio rápido](../virtual-network/quick-create-portal.md).

* Su red virtual no tiene ninguna puerta de enlace de red virtual. Si la red virtual tiene alguna puerta de enlace (ya sea VPN o ExpressRoute), tiene que quitarla. Esta configuración requiere que las redes virtuales estén conectadas a la puerta de enlace del centro de conectividad de Virtual WAN.

* Obtenga un intervalo de direcciones IP para la región del concentrador. El centro de conectividad es una red virtual que Virtual WAN crea y usa. El intervalo de direcciones que especifique para el centro de conectividad no se puede superponer a ninguna de las redes virtuales existentes a las que ya esté conectado. Igualmente no se puede superponer a los intervalos de direcciones con las que esté conectadas en el entorno local. Si no está familiarizado con los intervalos de direcciones IP ubicados en la configuración de la red local, póngase de acuerdo con alguien que pueda proporcionarle estos detalles.

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-virtual-wan"></a><a name="wan"></a>Creación de una instancia de Virtual WAN

Desde un explorador, navegue al [Portal de Azure](https://portal.azure.com) e inicie sesión con su cuenta de Azure.

1. Vaya a la página de Virtual WAN. En el portal, seleccione **+Crear un recurso**. Escriba **Virtual WAN** en el cuadro de búsqueda y seleccione **ENTRAR**.
1. Seleccione **Virtual WAN** en los resultados. En la página Virtual WAN, seleccione **Crear** para abrir la página Crear una red WAN.
1. Dentro de la página **Crear una red WAN**, en la pestaña **Aspectos básicos**, rellene los campos siguientes:

   ![Red WAN virtual](./media/virtual-wan-point-to-site-portal/vwan.png)

   * **Suscripción**: seleccione la suscripción que quiere usar.
   * **Grupo de recursos**: cree uno nuevo o utilice uno ya existente.
   * **Ubicación del grupo de recursos**: elija una ubicación para los recursos en la lista desplegable. Una red WAN es un recurso global y no reside en una región determinada. De todas formas, tiene que seleccionar una región con el fin de administrar y ubicar más fácilmente el recurso WAN que cree.
   * **Nombre**: escriba el nombre que desea dar a la WAN.
   * **Tipo:** Estándar. Si crea una WAN básica, solo puede crear un centro de conectividad básico. Los centros de conectividad básicos solo pueden tener conectividad VPN de sitio a sitio.
1. Cuando termine de rellenar los campos, haga clic en **Revisión y creación**.
1. Una vez que se haya superado la validación, seleccione **Crear** para crear la WAN virtual.

## <a name="create-a-p2s-configuration"></a><a name="p2sconfig"></a>Creación de una configuración de P2S

Una configuración de P2S define los parámetros para conectarse a los clientes remotos.

1. Vaya a **Todos los recursos**.
1. Seleccione la red WAN virtual que ha creado.
1. Seleccione **+Crear una configuración de VPN de usuario** en la parte superior de la página para abrir la página **Crear nueva configuración de VPN de usuario**.

   :::image type="content" source="media/virtual-wan-point-to-site-portal/p2s1.jpg" alt-text="Configuraciones de VPN de usuario":::

1. En la página **Crear nueva configuración de VPN de usuario**, rellene los campos siguientes:

   * **Nombre de la configuración**: nombre que desea usar para hacer referencia a su configuración.
   * **Tipo de túnel**: el protocolo que se usará para el túnel.
   * **Nombre del certificado raíz**: nombre descriptivo para el certificado.
   * **Datos de certificado público**: datos del certificado X.509 codificado en Base 64.
  
1. Seleccione **Crear** para crear la configuración.

## <a name="create-hub-with-point-to-site-gateway"></a><a name="hub"></a>Creación de un centro de conectividad con puerta de enlace de punto a sitio

1. En la red WAN virtual, seleccione Centros de conectividad y seleccione **+Nuevo centro de conectividad**.

   :::image type="content" source="media/virtual-wan-point-to-site-portal/hub1.jpg" alt-text="Nuevo centro de conectividad":::

1. En la página Crear centro de conectividad virtual, complete los siguientes campos.

   * **Región**: seleccione la región en la que quiere implementar el centro de conectividad virtual.
   * **Nombre**: escriba el nombre que quiere asignar al centro de conectividad virtual.
   * **Espacio de direcciones privadas del centro de conectividad**: intervalo de direcciones del centro de conectividad en la notación CIDR.

   :::image type="content" source="media/virtual-wan-point-to-site-portal/hub2.jpg" alt-text="Crear centro de conectividad virtual":::

1. En la pestaña "De punto a sitio", rellene los siguientes campos:

   * **Unidades de escalado de puerta de enlace**: que representa la capacidad agregada de la puerta de enlace de VPN del usuario.
   * **Configuración de punto a sitio**: la que creó en el paso anterior.
   * **Grupo de direcciones de clientes**: para los usuarios remotos.
   * **Dirección IP de servidor DNS personalizado**.

   :::image type="content" source="media/virtual-wan-point-to-site-portal/hub-with-p2s.png" alt-text="centro de conectividad con configuración de punto a sitio":::

1. Seleccione **Revisar + crear**.
1. En la página **Validación superada**, seleccione **Crear**.

## <a name="specify-dns-server"></a><a name="dns"></a>Especificación del servidor DNS

Las puertas de enlace VPN del usuario de la red WAN virtual permiten especificar hasta 5 servidores DNS. Puede configurar estos durante el proceso de creación del centro de conectividad o realizar modificaciones posteriormente. Para ello, busque el centro de conectividad virtual. En **VPN de usuario (punto a sitio)** , haga clic en configurar y escriba las direcciones IP de los servidores DNS en los cuadros de texto **Servidores DNS personalizados**.

   :::image type="content" source="media/virtual-wan-point-to-site-portal/custom-dns.png" alt-text="DNS personalizado" lightbox="media/virtual-wan-point-to-site-portal/custom-dns-expand.png":::

## <a name="download-vpn-profile"></a><a name="download"></a>Descarga del perfil de VPN

Use el perfil de VPN para configurar los clientes.

1. En la página de su red WAN virtual, seleccione **Configuraciones de VPN de usuario**.
2. En la parte superior de la página, seleccione **Download user VPN config** (Descargar configuración de VPN de usuario). La descarga de una configuración en el nivel WAN proporciona un perfil integrado de VPN de usuario basado en Traffic Manager. Para más información sobre los perfiles globales o los perfiles basados en un centro de conectividad, consulte [Perfiles de centro de conectividad](https://docs.microsoft.com/azure/virtual-wan/global-hub-profile).   Los escenarios de conmutación por error se simplifican con los perfiles globales.

   Si, por alguna razón, un centro de conectividad no está disponible, la administración de tráfico integrada que proporciona el servicio garantiza la conectividad mediante un centro de conectividad diferente a los recursos de Azure para los usuarios de punto a sitio. Siempre puede descargar una configuración de VPN específica del centro de conectividad. Para ello, vaya hasta el centro de conectividad específico. En **VPN de usuario (punto a sitio)** , descargue el perfil de **VPN de usuario** del centro de conectividad virtual.

1. Una vez que el archivo se haya terminado de crear, puede seleccionar el vínculo para descargarlo.
1. Use el archivo de perfil para configurar los clientes de VPN.

### <a name="configure-user-vpn-clients"></a>Configuración de clientes VPN de usuario

Use el perfil descargado para configurar los clientes de acceso remoto. El procedimiento para cada sistema operativo es diferente, siga las instrucciones correctas a continuación:

#### <a name="microsoft-windows"></a>Microsoft Windows
##### <a name="openvpn"></a>OpenVPN

1. Descargue e instale el cliente OpenVPN desde el sitio web oficial.
1. Descargue el perfil de VPN para la puerta de enlace. Esto puede hacerse desde la pestaña de configuraciones de VPN de usuario de Azure Portal o mediante New-AzureRmVpnClientConfiguration en PowerShell.
1. Descomprima el perfil. Abra el archivo de configuración vpnconfig.ovpn desde la carpeta OpenVPN en el Bloc de notas.
1. Rellene la sección de certificado cliente de P2S con la clave pública del certificado cliente de P2S en Base64. En un certificado con formato PEM, puede abrir el archivo .cer y copiar la clave de base64 entre los encabezados del certificado. Para conocer los pasos, consulte [cómo exportar un certificado para obtener la clave pública codificada](certificates-point-to-site.md).
1. Rellene la sección de la clave privada con la clave privada del certificado cliente de P2S en Base64. Para conocer los pasos, consulte [cómo extraer la clave privada](howto-openvpn-clients.md#windows).
1. No cambie los demás campos. Use los datos de la configuración de entrada del cliente para conectarse a la VPN.
1. Copie el archivo vpnconfig.ovpn en la carpeta C:\Program Files\OpenVPN\config.
1. Haga clic con el botón derecho en el icono OpenVPN en la bandeja del sistema y, después, seleccione **Conectar**.

##### <a name="ikev2"></a>IKEv2

1. Seleccione los archivos de configuración de cliente VPN que correspondan a la arquitectura del equipo Windows. Si la arquitectura de procesador es de 64 bits, elija el paquete del instalador "VpnClientSetupAmd64". En caso de que sea de 32 bits, elija el paquete del instalador "VpnClientSetupX86".
1. Haga doble clic en el paquete para instalarlo. Si ve una ventana emergente de SmartScreen, seleccione **Más información** y, después, **Ejecutar de todas formas**.
1. En el equipo cliente, vaya a **Configuración de red** y haga clic en **VPN**. La conexión VPN muestra el nombre de la red virtual a la que se conecta.
1. Antes de intentar conectarse, compruebe que ha instalado un certificado de cliente en el equipo cliente. Es necesario un certificado de cliente para la autenticación al usar el tipo de autenticación de certificados nativo de Azure. Para más información acerca de cómo generar certificados, consulte [Generación de certificados](certificates-point-to-site.md). Para obtener información acerca de cómo instalar un certificado de cliente, consulte [Instalación de certificados de cliente](../vpn-gateway/point-to-site-how-to-vpn-client-install-azure-cert.md).

## <a name="view-your-virtual-wan"></a><a name="viewwan"></a>Visualización de la instancia de Virtual WAN

1. Vaya a la instancia de Virtual WAN.
1. En la página **Información general**, cada punto del mapa representa un centro de conectividad.
1. En la sección de **centros de conectividad y conexiones**, puede ver estado del centro de conectividad, sitio, región, estado de la conexión VPN y bytes de entrada y salida.

## <a name="clean-up-resources"></a><a name="cleanup"></a>Limpieza de recursos

Cuando ya no necesite estos recursos, puede usar [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) para quitar el grupo de recursos y todos los recursos que contiene. Reemplace "myResourceGroup" con el nombre del grupo de recursos y ejecute el siguiente comando de PowerShell:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Virtual WAN, consulte la página [Introducción a Virtual WAN](virtual-wan-about.md).
