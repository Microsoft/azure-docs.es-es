---
title: 'Conexión a una red virtual mediante una VPN P2S y autenticación de certificados: portal'
titleSuffix: Azure VPN Gateway
description: Conecte de forma segura los clientes de Windows, Mac OS X y Linux a una red virtual de Azure mediante P2S y certificados autofirmados o emitidos por una entidad de certificación. En este artículo se usa Azure Portal.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: cherylmc
ms.openlocfilehash: 03dbc481950ed2a020a26dc3af8668c516b66115
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89436022"
---
# <a name="configure-a-point-to-site-vpn-connection-to-a-vnet-using-native-azure-certificate-authentication-azure-portal"></a>Configure una conexión VPN de punto a sitio a una red virtual mediante la autenticación nativa de los certificados de Azure: Portal de Azure

Este artículo le ayudará con la conexión segura de clientes que ejecuten Windows, Linux o Mac OS X a una red virtual de Azure. Las conexiones VPN de punto a sitio son útiles cuando desea conectarse a la red virtual desde una ubicación remota, como desde casa o desde una conferencia. También puede usar P2S en lugar de una VPN de sitio a sitio cuando son pocos los clientes que necesitan conectarse a una red virtual. Las conexiones de punto a sitio no requieren un dispositivo VPN ni una dirección IP de acceso público. P2S crea la conexión VPN sobre SSTP (Protocolo de túnel de sockets seguros) o IKEv2. Para más información sobre las conexiones VPN de punto a sitio, consulte [Acerca de las conexiones VPN de punto a sitio](point-to-site-about.md).

![Conexión de un equipo a una red virtual de Azure: diagrama de conexión de punto a sitio](./media/vpn-gateway-howto-point-to-site-resource-manager-portal/p2snativeportal.png)

## <a name="architecture"></a>Architecture

Las conexiones con autenticación mediante certificado de Azure nativas de punto a sitio usan los siguientes elementos, que se configuran en este ejercicio:

* Una puerta de enlace de VPN RouteBased.
* La clave pública (archivo .cer) de un certificado raíz, que se carga en Azure. Una vez cargado el certificado, se considera un certificado de confianza y se usa para la autenticación.
* Un certificado de cliente que se genera a partir del certificado raíz. El certificado de cliente se instalará en todos los equipos cliente que se conecten a la red virtual. Este certificado se usa para la autenticación de cliente.
* Una configuración de cliente VPN. Los archivos de configuración de cliente VPN contienen la información necesaria para que el cliente se conecte a la red virtual. Los archivos configuran el cliente VPN existente que es nativo en el sistema operativo. Cada cliente que se conecta debe configurarse con los valores de los archivos de configuración.

#### <a name="example-values"></a><a name="example"></a>Valores del ejemplo

Puede usar los siguientes valores para crear un entorno de prueba o hacer referencia a ellos para comprender mejor los ejemplos de este artículo:

* **Nombre de la red virtual:** VNet1
* **Espacio de direcciones**: 10.1.0.0/16<br>En este ejemplo, se utiliza solo un espacio de direcciones. Puede tener más de un espacio de direcciones para la red virtual.
* **Nombre de subred:** FrontEnd
* **Intervalo de direcciones de subred:** 10.1.0.0/24
* **Subscription** (Suscripción): si tiene más de una suscripción, compruebe que usa la correcta.
* **Grupos de recursos:** TestRG1
* **Ubicación:** Este de EE. UU.
* **GatewaySubnet:** 10.1.255.0/27<br>
* **Nombre de la puerta de enlace de red virtual:** VNet1GW
* **Tipo de puerta de enlace:** VPN
* **Tipo de VPN:** basada en rutas
* **Nombre de dirección IP pública:** VNet1GWpip
* **Tipo de conexión**: De punto a sitio
* **Grupo de direcciones de clientes:** 172.16.201.0/24<br>Los clientes de VPN que se conectan a la red virtual mediante esta conexión de punto a sitio reciben una dirección IP del grupo de clientes.

## <a name="1-create-a-virtual-network"></a><a name="createvnet"></a>1. Creación de una red virtual

Antes de empezar, compruebe que tiene una suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial).
[!INCLUDE [Basic Point-to-Site VNet](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="2-create-a-virtual-network-gateway"></a><a name="creategw"></a>2. Creación de una puerta de enlace de red virtual

En este paso, se crea la puerta de enlace para la red virtual. La creación de una puerta de enlace suele tardar 45 minutos o más, según la SKU de la puerta de enlace seleccionada.

>[!NOTE]
>La SKU de puerta de enlace de nivel Básico no admite la autenticación de IKEv2 o RADIUS. Si planea que clientes Mac se conecten a la red virtual, no use la SKU de nivel Básico.
>

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-portal-include.md)]

[!INCLUDE [Create a gateway](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

## <a name="3-generate-certificates"></a><a name="generatecert"></a>3. Generación de certificados

Azure usa certificados para autenticar a los clientes para conectarse a una red virtual a través de una conexión VPN de punto a sitio. Una vez que obtenga el certificado raíz, [cargue](#uploadfile) la información de clave pública en Azure. En ese momento, Azure considera que el certificado raíz es "de confianza" para conectarse de P2S a la red virtual. También genere certificados de cliente desde el certificado raíz de confianza y, a continuación, vuelva a instalarlos en cada equipo cliente. El certificado de cliente se utiliza para autenticar al cliente cuando se inicia una conexión con la red virtual. 

### <a name="1-obtain-the-cer-file-for-the-root-certificate"></a><a name="getcer"></a>1. Obtención del archivo .cer del certificado raíz

[!INCLUDE [root-certificate](../../includes/vpn-gateway-p2s-rootcert-include.md)]

### <a name="2-generate-a-client-certificate"></a><a name="generateclientcert"></a>2. Generación de un certificado de cliente

[!INCLUDE [generate-client-cert](../../includes/vpn-gateway-p2s-clientcert-include.md)]

## <a name="4-add-the-client-address-pool"></a><a name="addresspool"></a>4. Incorporación del grupo de direcciones del cliente

El grupo de direcciones de cliente es un intervalo de direcciones IP privadas que usted especifica. Los clientes que se conectan de forma dinámica a través de una VPN de punto a sitio reciben una dirección IP de este intervalo. Use un intervalo de direcciones IP privadas que no se superponga a la ubicación local desde la que se va a conectar ni a la red virtual a la que desea conectarse. Si configura varios protocolos y SSTP es uno de ellos, el grupo de direcciones configurado se divide equitativamente entre los protocolos configurados.

1. Una vez creada la puerta de enlace de red virtual, navegue hasta la sección **Valores** de la página de la puerta de enlace de red virtual. En la sección **Configuración**, seleccione **Configuración de punto a sitio**. Seleccione **Configure now** (Configurar ahora) para abrir la página de configuración.

   ![Página de punto a sitio](./media/vpn-gateway-howto-point-to-site-resource-manager-portal/point-to-site-configure.png "Punto a sitio: configuración ahora")
2. En la página **Configuración de punto a sitio**, puede configurar diversas opciones. Si no ve el tipo de túnel o el tipo de autenticación en esta página, la puerta de enlace usará la SKU básica. La SKU básica no admite la autenticación de IKEv2 o RADIUS. Si quiere usar esta configuración, debe eliminar y volver a crear la puerta de enlace con una SKU de puerta de enlace diferente.

   [![Configuración de punto a sitio](./media/vpn-gateway-howto-point-to-site-resource-manager-portal/certificate-settings-address.png "especificar el grupo de direcciones")](./media/vpn-gateway-howto-point-to-site-resource-manager-portal/certificate-settings-expanded.png#lightbox)
3. En el cuadro **Grupo de direcciones**, agregue el intervalo de direcciones IP privadas que quiere usar. Los clientes VPN reciben de forma dinámica una dirección IP del intervalo que especifique. La máscara de subred mínima es de 29 bits para la configuración activa/pasiva, y de 28 bits para la configuración activa/activa.
4. Vaya a la sección siguiente para configurar el tipo de túnel.

## <a name="5-configure-tunnel-type"></a><a name="tunneltype"></a>5. Configuración del tipo de túnel

Puede seleccionar el tipo de túnel. Las opciones de túnel son OpenVPN, SSTP y IKEv2.

* El cliente Strongswan de Linux y Android, y el cliente VPN IKEv2 nativo de iOS y OSX solo utilizarán el túnel IKEv2 para conectarse.
* Los clientes Windows prueban primero el túnel IKEv2 y, si no se conecta, recurren a SSTP.
* Puede usar el cliente OpenVPN para conectarse con el tipo de túnel OpenVPN.

![Tipo de túnel](./media/vpn-gateway-howto-point-to-site-resource-manager-portal/tunnel.png "especificar el tipo de túnel")

## <a name="6-configure-authentication-type"></a><a name="authenticationtype"></a>6. Configuración del tipo de autenticación

En **Tipo de autenticación**, seleccione **Certificado de Azure**.

  ![Tipo de autenticación](./media/vpn-gateway-howto-point-to-site-resource-manager-portal/authentication-type.png "especificar el tipo de autenticación")

## <a name="7-upload-the-root-certificate-public-certificate-data"></a><a name="uploadfile"></a>7. Cargue la información del certificado de datos público del certificado raíz

Puede cargar más certificados raíz de confianza, hasta un total de 20. Una vez que se han cargado los datos de certificado público, Azure puede usarlos para autenticar a los clientes que tienen instalado un certificado de cliente generado a partir del certificado raíz de confianza. Cargue la información de clave pública del certificado raíz en Azure.

1. Los certificados se agregan en la página **Configuración de punto a sitio** de la sección **Certificado raíz**.
2. Asegúrese de exportar el certificado raíz como archivo X.509 codificado base 64 (.cer). Debe exportar el certificado en este formato para poder abrirlo con un editor de texto.
3. Abra el certificado con un editor de texto como Bloc de notas. Al copiar los datos del certificado, asegúrese de copiar el texto como una línea continua sin retornos de carro ni avances de línea. Es posible que para ver los retornos de carro y los avances de línea deba cambiar la vista del editor de texto de forma que se muestren los símbolos y todos los caracteres. Copie solo la siguiente sección como una línea continua:

   ![Datos del certificado](./media/vpn-gateway-howto-point-to-site-resource-manager-portal/notepadroot.png "copiar los datos del certificado raíz")
4. Pegue los datos del certificado en la sección **Public Certificate Data** (Datos de certificado público). En **Nombre** asigne un nombre al certificado y, luego, seleccione **Guardar**. Puede agregar hasta 20 certificados raíz de confianza.

   ![Pegado de los datos del certificado](./media/vpn-gateway-howto-point-to-site-resource-manager-portal/uploaded.png "pegar los datos del certificado")
5. Seleccione **Guardar** en la parte superior de la página para guardar todos los valores de configuración.

   ![Guardar configuración](./media/vpn-gateway-howto-point-to-site-resource-manager-portal/save.png "guardar la configuración")

## <a name="8-install-an-exported-client-certificate"></a><a name="installclientcert"></a>8. Instalación de un certificado de cliente exportado

Si desea crear una conexión P2S desde un equipo cliente distinto del que usó para generar los certificados de cliente, debe instalar un certificado de cliente. Al instalar un certificado de cliente, necesita la contraseña que se creó cuando se exportó el certificado de cliente.

Asegúrese de que se exportó el certificado de cliente como un .pfx junto con la cadena de certificados completa (que es el valor predeterminado). En caso contrario, la información del certificado raíz no está presente en el equipo cliente y el cliente no podrá realizar la autenticación correctamente.

Para obtener los pasos de instalación, consulte [Instalación de un certificado de cliente](point-to-site-how-to-vpn-client-install-azure-cert.md).

## <a name="9-generate-and-install-the-vpn-client-configuration-package"></a><a name="clientconfig"></a>9. Generación e instalación del paquete de configuración de cliente de VPN

Los archivos de configuración del cliente VPN contienen opciones para configurar los dispositivos para conectarse a una red virtual a través de una conexión de punto a sitio. Para obtener instrucciones para generar e instalar archivos de configuración de cliente VPN, consulte [Creación e instalación de archivos de configuración de cliente VPN para configuraciones de punto a sitio con autenticación con certificados nativos de Azure](point-to-site-vpn-client-configuration-azure-cert.md).

## <a name="10-connect-to-azure"></a><a name="connect"></a>10. Conexión con Azure

### <a name="to-connect-from-a-windows-vpn-client"></a>Para conectarse desde un cliente VPN en Windows

>[!NOTE]
>Debe tener derechos de administrador en el equipo cliente de Windows desde el que se va a conectar.
>
>

1. Para conectarse a su red virtual, en el equipo cliente, vaya a las conexiones VPN y ubique la que creó. Tiene el mismo nombre que su red virtual. Seleccione **Conectar**. Es posible que aparezca un mensaje emergente que haga referencia al uso del certificado. Seleccione **Continuar** para usar privilegios elevados.

2. En la página de estado **Conexión**, seleccione **Conectar** para iniciar la conexión. Si ve una pantalla para **Seleccionar certificado** , compruebe que el certificado de cliente que se muestra es el que desea utilizar para conectarse. Si no es así, use la flecha de lista desplegable para seleccionar el certificado correcto y, luego, seleccione **Aceptar**.

   ![El cliente VPN se conecta a Azure](./media/vpn-gateway-howto-point-to-site-resource-manager-portal/clientconnect.png "conectar")
3. Se ha establecido la conexión.

   ![Conexión establecida](./media/vpn-gateway-howto-point-to-site-resource-manager-portal/connected.png "conexión establecida")

#### <a name="troubleshoot-windows-p2s-connections"></a>Solución de problemas de conexiones de punto a sitio en Windows

[!INCLUDE [verifies client certificates](../../includes/vpn-gateway-certificates-verify-client-cert-include.md)]

### <a name="to-connect-from-a-mac-vpn-client"></a>Para conectarse desde un cliente VPN en Mac

En el cuadro de diálogo Red, localice el perfil de cliente que quiere usar, especifique la configuración de [VpnSettings.xml](point-to-site-vpn-client-configuration-azure-cert.md#installmac) y, después, haga clic en **Conectar**.

Para obtener instrucciones detalladas al respecto, consulte [Instalación: Mac (OS X)](https://docs.microsoft.com/azure/vpn-gateway/point-to-site-vpn-client-configuration-azure-cert#installmac). Si tiene problemas para conectarse, compruebe que la puerta de enlace de red virtual no está usando una SKU de nivel Básico. La SKU de nivel Básico no es compatible con los clientes Mac.

  ![Conexión de Mac](./media/vpn-gateway-howto-point-to-site-rm-ps/applyconnect.png "Conectar")

## <a name="to-verify-your-connection"></a><a name="verify"></a>Para comprobar la conexión

Estas instrucciones se aplican a los clientes Windows.

1. Para comprobar que la conexión VPN está activa, abra un símbolo del sistema con privilegios elevados y ejecute *ipconfig/all*.
2. Vea los resultados. Observe que la dirección IP que recibió es una de las direcciones dentro del grupo de direcciones de cliente de VPN punto a sitio que especificó en la configuración. Los resultados son similares a los del ejemplo siguiente:

   ```
   PPP adapter VNet1:
      Connection-specific DNS Suffix .:
      Description.....................: VNet1
      Physical Address................:
      DHCP Enabled....................: No
      Autoconfiguration Enabled.......: Yes
      IPv4 Address....................: 172.16.201.3(Preferred)
      Subnet Mask.....................: 255.255.255.255
      Default Gateway.................:
      NetBIOS over Tcpip..............: Enabled
   ```

## <a name="to-connect-to-a-virtual-machine"></a><a name="connectVM"></a>Conexión a una máquina virtual

Estas instrucciones se aplican a los clientes Windows.

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm-p2s-include.md)]

## <a name="to-add-or-remove-trusted-root-certificates"></a><a name="add"></a>Para agregar o quitar certificados raíz de confianza

Puede agregar y quitar certificados raíz de confianza de Azure. Al quitar un certificado raíz, los clientes que tienen un certificado generado a partir de dicha raíz no podrán autenticarse y, por tanto, no podrán conectarse. Si desea que un cliente se autentique y se conecte, es preciso que instale un nuevo certificado de cliente generado a partir de un certificado raíz que sea de confianza (se cargue) en Azure.

### <a name="to-add-a-trusted-root-certificate"></a>Para agregar un certificado raíz de confianza

Puede agregar hasta 20 archivos .cer de certificado raíz de confianza a Azure. Para ver instrucciones, consulte la sección [Carga del archivo .cer del certificado raíz](#uploadfile) de este artículo.

### <a name="to-remove-a-trusted-root-certificate"></a>Eliminación de un certificado raíz de confianza

1. Para quitar un certificado raíz de confianza, vaya a la página **Configuración de punto a sitio** para la puerta de enlace de red virtual.
2. En la sección **Certificado raíz** de la página, busque el certificado que desea quitar.
3. Seleccione los puntos suspensivos junto al certificado y, luego, "Quitar".

## <a name="to-revoke-a-client-certificate"></a><a name="revokeclient"></a>Revocación de un certificado de cliente

Puede revocar certificados de cliente. La lista de revocación de certificados permite denegar de forma selectiva la conectividad de punto a sitio basada en certificados de cliente individuales. Esto difiere de la forma en que se quita un certificado raíz de confianza. Si quita de Azure un archivo .cer de certificado raíz de confianza, se revoca el acceso para todos los certificados de cliente generados y firmados con el certificado raíz revocado. Al revocarse un certificado de cliente, en lugar del certificado raíz, se permite que el resto de los certificados que se generaron a partir del certificado raíz sigan usándose para la autenticación.

Lo más habitual es usar el certificado raíz para administrar el acceso a nivel de equipo u organización, mientras que los certificados de cliente revocados se usan para un control de acceso específico para usuarios individuales.

### <a name="revoke-a-client-certificate"></a>Revocar un certificado de cliente

Puede revocar un certificado de cliente si agrega la huella digital a la lista de revocación.

1. Recupere la huella digital del certificado de cliente. Para más información, consulte [Cómo recuperar la huella digital de un certificado](https://msdn.microsoft.com/library/ms734695.aspx).
2. Copie la información en un editor de texto y quite todos los espacios de forma que sea una sola cadena continua.
3. Vaya a la página **Configuración de punto a sitio** de la puerta de enlace de red virtual. Se trata de la misma hoja que utilizó para [cargar un certificado raíz de confianza](#uploadfile).
4. En la sección **Certificados revocados**, especifique un nombre descriptivo para el certificado (no es necesario que sea el CN del certificado).
5. Copie y pegue la cadena de huella digital en el campo **Huella digital**.
6. Se valida la huella digital y se agrega automáticamente a la lista de revocación. Aparece un mensaje en la pantalla que indica que se está actualizando la lista. 
7. Una vez finalizada la actualización, el certificado no se puede usar para conectarse. Los clientes que intenten conectarse con este certificado reciben un mensaje que indica que el certificado ya no es válido.

## <a name="point-to-site-faq"></a><a name="faq"></a>Preguntas más frecuentes sobre la conexión de punto a sitio

[!INCLUDE [Point-to-Site FAQ](../../includes/vpn-gateway-faq-p2s-azurecert-include.md)]

## <a name="next-steps"></a>Pasos siguientes
Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Virtual Machines](https://docs.microsoft.com/azure/) para más información. Para más información acerca de las redes y las máquinas virtuales, consulte [Información general sobre las redes de máquina virtual con Linux y Azure](../virtual-machines/linux/azure-vm-network-overview.md).

Para obtener información sobre solución de problemas de P2S, vea [Solución de problemas: conexión de punto a sitio de Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).
