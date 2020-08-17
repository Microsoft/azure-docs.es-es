---
title: 'Conexión de redes virtuales clásicas a redes virtuales de Azure Resource Manager: Portal | Microsoft Docs'
description: Pasos para conectar redes virtuales clásicas a redes virtuales de Resource Manager mediante VPN Gateway y el portal
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/24/2019
ms.author: cherylmc
ms.openlocfilehash: d3ab49d0ad24c2b2c8859408ed103178cede5b8b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082110"
---
# <a name="connect-virtual-networks-from-different-deployment-models-using-the-portal"></a>Conexión de redes virtuales a partir de diferentes modelos de implementación con el portal

Este artículo muestra cómo conectar redes virtuales clásicas a redes virtuales de Resource Manager para permitir que los recursos que se encuentran en modelos de implementación independientes se comuniquen entre sí. En los pasos de este artículo se usa fundamentalmente Azure Portal, pero también se puede crear esta configuración con PowerShell si se selecciona el artículo en esta lista.

> [!div class="op_single_selector"]
> * [Portal](vpn-gateway-connect-different-deployment-models-portal.md)
> * [PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
> 
> 

La conexión de una red virtual clásica a una red virtual de Resource Manager es similar a la conexión de una red virtual a una ubicación de sitio local. Ambos tipos de conectividad usan una puerta de enlace de VPN para proporcionar un túnel seguro con IPsec/IKE. Puede crear una conexión entre redes virtuales que estén en diferentes suscripciones y en diferentes regiones. También puede conectar redes virtuales que tengan ya conexiones a redes locales, siempre que la puerta de enlace con la que se hayan configurado sea dinámica o basada en ruta. Para más información acerca de las conexiones de red virtual a red virtual, consulte [P+F sobre conexiones de red virtual a red virtual](#faq) al final de este artículo. 

Si aún no tiene una puerta de enlace de red virtual y no desea crear una, considere la posibilidad de conectar sus redes virtuales mediante Emparejamiento de VNET. El emparejamiento de VNET no usa VPN Gateway. Para más información, consulte [Emparejamiento de VNET](../virtual-network/virtual-network-peering-overview.md).

### <a name="before-you-begin"></a><a name="before"></a>Antes de empezar



* En estos pasos se presume que ya se crearon ambas redes virtuales. Si usa este artículo como ejercicio y no tiene redes virtuales, los pasos incluyen vínculos que le ayudarán a crearlas.
* Compruebe que los intervalos de direcciones de las redes virtuales no se superponen entre sí ni con alguno de los intervalos de otras conexiones con las que puedan estar conectadas las puertas de enlace.
* Instale los cmdlets de PowerShell más recientes para Resource Manager y Service Management (clásico). En este artículo se usan Azure Portal y PowerShell. PowerShell es necesario para crear la conexión desde la red virtual clásica a la red virtual de Resource Manager. Para obtener más información, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/). 

### <a name="example-settings"></a><a name="values"></a>Configuración de ejemplo

Puede usar estos valores para crear un entorno de prueba o hacer referencia a ellos para comprender mejor los ejemplos de este artículo.

**Red virtual clásica**

Nombre de red virtual = ClassicVNet <br>
Espacio de direcciones = 10.0.0.0/24 <br>
Nombre de subred = Subnet-1 <br>
Intervalo de direcciones de subred = 10.0.0.0/27 <br>
Suscripción = la suscripción que desea usar <br>
Grupo de recursos = ClassicRG <br>
Ubicación = Oeste de EE. UU. <br>
GatewaySubnet = 10.0.0.32/28 <br>
Sitio local = RMVNetLocal <br>

**Red virtual de Resource Manager**

Nombre de red virtual = RMVNet <br>
Espacio de direcciones = 192.168.0.0/16 <br>
Grupo de recursos = GR1 <br>
Ubicación = Este de EE. UU. <br>
Nombre de subred = Subnet-1 <br>
Intervalo de direcciones = 192.168.1.0/24 <br>
Subred de la puerta de enlace = 192.168.0.0/26 <br>
Nombre de la puerta de enlace de red virtual = PuertaDeEnlaceRM <br>
Tipo de puerta de enlace = VPN <br>
Tipo de VPN = basada en enrutamiento <br>
SKU = VpnGw1 <br>
Ubicación = Este de EE. UU. <br>
Red virtual = RMVNet <br> (asocie la puerta de enlace de VPN a esta red virtual) Configuración de la primera dirección IP = rmgwpip <br> (dirección IP pública de la puerta de enlace) Puerta de enlace de red local = ClassicVNetLocal <br>
Nombre de conexión = RMtoClassic

### <a name="connection-overview"></a><a name="connectoverview"></a>Información general sobre la conexión

Para esta configuración se crea una conexión de puerta de enlace VPN a través de un túnel VPN de IPsec/IKE entre las redes virtuales. Asegúrese de que ninguno de los intervalos de red virtual se superponga con otro o con cualquiera de las redes locales a las que se conectan.

En la tabla siguiente se muestra un ejemplo de cómo se definen los sitios locales y las redes virtuales de ejemplo:

| Virtual Network | Espacio de direcciones | Region | Se conecta a un sitio de red local |
|:--- |:--- |:--- |:--- |
| ClassicVNet |(10.0.0.0/24) |Oeste de EE. UU. | RMVNetLocal (192.168.0.0/16) |
| RMVNet | (192.168.0.0/16) |Este de EE. UU. |ClassicVNetLocal (10.0.0.0/24) |

## <a name="section-1---configure-the-classic-vnet-settings"></a><a name="classicvnet"></a>Sección 1: Configuración de la red virtual clásica

En esta sección se crea la red virtual clásica, la red local (sitio local) y la puerta de enlace de red virtual. Las capturas de pantalla se proporcionan a modo de ejemplo. Asegúrese de reemplazar los valores por los suyos o use los valores de [ejemplo](#values).

### <a name="1-create-a-classic-vnet"></a>1. <a name="classicvnet"></a>Creación de una red virtual clásica

Si no tiene una red virtual clásica y lleva a cabo estos pasos como ejercicio, puede crear una red virtual según [este artículo](../virtual-network/virtual-networks-create-vnet-classic-pportal.md) y los valores de configuración de [ejemplo](#values) indicados anteriormente.

Si ya tiene una red virtual con una puerta de enlace VPN, compruebe que la puerta de enlace sea dinámica. Si es estática, primero debe eliminar la puerta de enlace de VPN antes de pasar a [configurar el sitio local](#local).

1. Abra [Azure Portal](https://ms.portal.azure.com) e inicie sesión con su cuenta de Azure.
2. Haga clic en **+ Crear un recurso** para abrir la página "Nuevo".
3. En el campo "Buscar en el Marketplace", escriba "Virtual Network". Si en su lugar, selecciona Redes -> Virtual Network, no obtendrá la opción para crear una red virtual clásica.
4. En la lista de resultados, busque "Virtual Network" y haga clic en esa opción para abrir la página Virtual Network. 
5. En la página de red virtual, seleccione "Clásica" para crear una red virtual clásica. Si aquí usa el valor predeterminado, generará en su lugar una red virtual de Resource Manager.

### <a name="2-configure-the-local-site"></a>2. <a name="local"></a>Configuración del sitio local

1. Vaya a **Todos los recursos** y ubique **ClassicVNet** en la lista.
2. Haga clic en **Puerta de enlace** en la sección **Configuración** del menú y, a continuación, haga clic en el banner para crear una puerta de enlace.
  ![Configuración de una puerta de enlace de VPN](./media/vpn-gateway-connect-different-deployment-models-portal/gatewaygraphic.png "Configuración de una instancia de VPN Gateway")
3. En la página **Nueva conexión de VPN**, en la opción **Tipo de conexión**, seleccione **De sitio a sitio**.
4. En **Sitio local**, haga clic en **Configurar los valores obligatorios**. Con esto se abre la página **Sitio local**.
5. En la página **Sitio Local**, cree un nombre para hacer referencia a la red virtual de Resource Manager. Por ejemplo, "RMVNetLocal".
6. Si la puerta de enlace VPN para la red virtual de Resource Manager ya tiene una dirección IP pública, use el valor para el campo **Dirección IP de la VPN Gateway**. Si lleva a cabo estos pasos como ejercicio o todavía no tiene una puerta de enlace de red virtual para la red virtual de Resource Manager, puede inventar una dirección IP de marcador de posición. Asegúrese de que la dirección IP de marcador de posición utiliza un formato válido. Más adelante, reemplaza la dirección IP de marcador de posición por la dirección IP pública de la puerta de enlace de red virtual de Resource Manager.
7. Para **Client Address Space** (Espacio de direcciones de clientes), use los [valores](#connectoverview) de los espacios de direcciones IP de red virtual para la red virtual de Resource Manager. Este valor se usa para especificar los espacios de direcciones a fin de enrutar a la red virtual de Resource Manager. En el ejemplo usamos 192.168.0.0/16, el intervalo de direcciones de RMVNet.
8. Haga clic en **Aceptar** para guardar los valores y volver a la página **Nueva conexión de VPN**.

### <a name="3-create-the-virtual-network-gateway"></a><a name="classicgw"></a>3. Creación de la puerta de enlace de red virtual

1. En la página **Nueva conexión VPN**, active la casilla **Crear puerta de enlace inmediatamente**.
2. Haga clic en **Configuración de puerta de enlace opcional** para abrir la página **Configuración de puerta de enlace**.

   ![Apertura de la página de configuración de las puertas de enlace](./media/vpn-gateway-connect-different-deployment-models-portal/optionalgatewayconfiguration.png "Apertura de la página de configuración de las puertas de enlace")
3. Haga clic en **Subred: configurar los valores obligatorios** para abrir la página **Agregar subred**. El campo **Nombre** ya está establecido en el valor apropiado: **GatewaySubnet**.
4. **Intervalo de direcciones** hace referencia al intervalo para la subred de puerta de enlace. Si bien puede crear una subred de puerta de enlace con un intervalo de direcciones de /29 (3 direcciones), se recomienda crear una subred de puerta de enlace que incluya más direcciones IP. Esto permitirá configuraciones futuras que puedan requerir más direcciones IP disponibles. Si es posible, utilice /27 o /28. Si usa estos pasos como ejercicio, puede hacer referencia a los [valores de ejemplo](#values). En este ejemplo se usa "10.0.0.32/28". Haga clic en **Aceptar** para crear la subred de puerta de enlace.
5. En la página **Configuración de puerta de enlace**, la opción **Tamaño** hace referencia a la SKU de la puerta de enlace. Seleccione la SKU de puerta de enlace para la puerta de enlace VPN.
6. Compruebe que el **Tipo de enrutamiento** sea **Dinámico** y haga clic en **Aceptar** para volver a la página **Nueva conexión de VPN**.
7. En la página **Nueva conexión de VPN**, haga clic en **Aceptar** para empezar a crear la puerta de enlace de VPN. Una puerta de enlace VPN puede tardar hasta 45 minutos en completarse.

### <a name="4-copy-the-virtual-network-gateway-public-ip-address"></a><a name="ip"></a>4. Copia de la dirección IP pública de la puerta de enlace de la red virtual

Una vez que se haya creado la puerta de enlace de la red virtual, puede ver la dirección IP de la puerta de enlace. 

1. Desplácese a la red virtual clásica y haga clic en **Información general**.
2. Haga clic en **Conexiones de VPN** para abrir la página de conexiones. En la página de conexiones de VPN, puede ver la dirección IP pública. Se trata de la dirección IP pública asignada a la puerta de enlace de red virtual. Anote la dirección IP. Se usa en pasos posteriores al trabajar con las opciones de configuración de puerta de enlace de red local de Resource Manager. 
3. Puede ver el estado de las conexiones de puerta de enlace. Observe que el sitio de red local que creó aparece como "Conectando". El estado cambiará una vez creadas las conexiones. Puede cerrar esta página cuando termine de ver el estado.

## <a name="section-2---configure-the-resource-manager-vnet-settings"></a><a name="rmvnet"></a>Sección 2: Configuración de redes virtuales de Resource Manager

En esta sección se crea la puerta de enlace de red virtual y la puerta de enlace de red local para la red de Resource Manager. Las capturas de pantalla se proporcionan a modo de ejemplo. Asegúrese de reemplazar los valores por los suyos o use los valores de [ejemplo](#values).

### <a name="1-create-a-virtual-network"></a>1. Creación de una red virtual

**Valores de ejemplo:**

* Nombre de red virtual = RMVNet <br>
* Espacio de direcciones = 192.168.0.0/16 <br>
* Grupo de recursos = GR1 <br>
* Ubicación = Este de EE. UU. <br>
* Nombre de subred = Subnet-1 <br>
* Intervalo de direcciones = 192.168.1.0/24 <br>

Si no tiene una red virtual de Resource Manager y va a ejecutar estos pasos como ejercicio, cree una red virtual con los pasos que se indican en [Creación de una red virtual](../virtual-network/quick-create-portal.md) y use los valores de ejemplo.

### <a name="2-create-a-virtual-network-gateway"></a><a name="creategw"></a>2. Creación de una puerta de enlace de red virtual

En este paso, se crea la puerta de enlace para la red virtual. La creación de una puerta de enlace suele tardar 45 minutos o más, según la SKU de la puerta de enlace seleccionada.

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-portal-include.md)]

**Valores de ejemplo:**

* Nombre de la puerta de enlace de red virtual = PuertaDeEnlaceRM <br>
* Tipo de puerta de enlace = VPN <br>
* Tipo de VPN = basada en enrutamiento <br>
* SKU = VpnGw1 <br>
* Ubicación = Este de EE. UU. <br>
* Red virtual = RMVNet <br>
* Subred de la puerta de enlace = 192.168.0.0/26 <br>
* Configuración de primera dirección IP = rmgwpip <br>

[!INCLUDE [vpn-gateway-add-gw-rm-portal](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

### <a name="3-create-a-local-network-gateway"></a><a name="createlng"></a>3. Creación de una puerta de enlace de red local

**Valores de ejemplo:** Puerta de enlace de red local = LocalRedVClásica

| Virtual Network | Espacio de direcciones | Region | Se conecta a un sitio de red local |Dirección IP pública de puerta de enlace|
|:--- |:--- |:--- |:--- |:--- |
| ClassicVNet |(10.0.0.0/24) |Oeste de EE. UU. | RMVNetLocal (192.168.0.0/16) |La dirección IP pública que se asigna a la puerta de enlace ClassicVNet|
| RMVNet | (192.168.0.0/16) |Este de EE. UU. |ClassicVNetLocal (10.0.0.0/24) |La dirección IP pública que se asigna a la puerta de enlace RMVNet.|

La puerta de enlace de red local especifica el intervalo de direcciones y la dirección IP pública asociados a la red virtual clásica y su puerta de enlace de red virtual. Si lleva a cabo estos pasos como un ejercicio, consulte los valores de ejemplo.

[!INCLUDE [vpn-gateway-add-lng-rm-portal](../../includes/vpn-gateway-add-lng-rm-portal-include.md)]

## <a name="section-3---modify-the-classic-vnet-local-site-settings"></a><a name="modifylng"></a>Sección 3: modificación de la configuración del sitio local de la red virtual clásica

En esta sección se reemplaza la dirección IP de marcador de posición que usó al especificar la configuración del sitio local con la dirección IP de puerta de enlace VPN de Resource Manager. Esta sección utiliza los cmdlets de PowerShell (SM) clásicos.

1. En Azure Portal, vaya a la red virtual clásica.
2. En la página de la red virtual, haga clic en **Información general**.
3. En la sección **Conexiones VPN**, haga clic en el nombre del sitio local en el gráfico.

   ![VPN-connections](./media/vpn-gateway-connect-different-deployment-models-portal/vpnconnections.png "VPN Connections")
4. En la página **Conexiones de VPN de sitio a sitio**, haga clic en el nombre del sitio.

   ![Site-name](./media/vpn-gateway-connect-different-deployment-models-portal/sitetosite3.png "Nombre del sitio local")
5. En la página de conexión del sitio local, haga clic en el nombre del sitio local para abrir la página **Sitio local**.

   ![Open-local-site](./media/vpn-gateway-connect-different-deployment-models-portal/openlocal.png "Sitio local abierto")
6. En la página **Sitio local**, reemplace la **dirección IP de la puerta de enlace de VPN** por la dirección IP de la puerta de enlace de Resource Manager.

   ![Gateway-ip-address](./media/vpn-gateway-connect-different-deployment-models-portal/gwipaddress.png "Dirección IP de la puerta de enlace")
7. Haga clic en **Aceptar** para actualizar la dirección IP.

## <a name="section-4---create-resource-manager-to-classic-connection"></a><a name="RMtoclassic"></a>Sección 4: Creación de conexión de red virtual de Resource Manager a red virtual clásica

En estos pasos se configura la conexión desde la red virtual de Resource Manager a la red virtual clásica con Azure Portal.

1. En **Todos los recursos**, ubique la puerta de enlace de red local. En el ejemplo, la puerta de enlace de red local es **ClassicVNetLocal**.
2. Haga clic en **Configuración** y compruebe que el valor de dirección IP es la puerta de enlace VPN para la red virtual clásica. Actualice si es necesario y, luego, haga clic en **Guardar**. Cierre la página.
3. En **Todos los recursos**, haga clic en la puerta de enlace de red local.
4. Haga clic en **Conexiones** para abrir la página Conexiones.
5. En la página **Conexiones**, haga clic en **+** para agregar una conexión.
6. En la página **Agregar conexión**, asigne un nombre a la conexión. Por ejemplo, "RMtoClassic".
7. En esta página ya está seleccionada la opción **De sitio a sitio**.
8. Seleccione la puerta de enlace de red virtual que desea asociar con este sitio.
9. Cree una **clave compartida**. Esta clave se usa también en la conexión que crea desde la red virtual clásica a la red virtual de Resource Manager. Puede generar la clave o inventar una. En el ejemplo usamos "abc123", pero puede (y debe) usar un valor más complejo.
10. Haga clic en **Aceptar** para crear la conexión.

## <a name="section-5---create-classic-to-resource-manager-connection"></a><a name="classictoRM"></a>Sección 5: Creación de conexión de red virtual clásica a red virtual de Resource Manager

En estos pasos se configura la conexión desde la red virtual clásica a la red virtual de Resource Manager. Estos pasos requieren PowerShell. Esta conexión no se puede crear en el portal. Asegúrese de que ha descargado e instalado los cmdlets de PowerShell tanto del modelo clásico (SM) como del modelo de Resource Manager (RM).

### <a name="1-connect-to-your-azure-account"></a>1. Conexión a la cuenta de Azure

Abra la consola de PowerShell con derechos elevados e inicie sesión en la cuenta de Azure. Después de iniciar la sesión, se descarga la configuración de la cuenta para que esté disponible para Azure PowerShell. El siguiente cmdlet pide las credenciales de inicio de sesión de la cuenta de Azure para el modelo de implementación de Resource Manager:

```powershell
Connect-AzAccount
```

Obtenga una lista de las suscripciones de Azure.

```powershell
Get-AzSubscription
```

Si tiene varias suscripciones, seleccione la que quera usar.

```powershell
Select-AzSubscription -SubscriptionName "Name of subscription"
```

A continuación, inicie sesión para usar los cmdlets de PowerShell clásicos (administración de servicios). Para agregar su cuenta de Azure del modelo de implementación clásica, use el siguiente comando:

```powershell
Add-AzureAccount
```

Obtenga una lista de las suscripciones. Este paso puede ser necesario al agregar los cmdlets de Service Management, dependiendo de su módulo de Azure.

```powershell
Get-AzureSubscription
```

Si tiene varias suscripciones, seleccione la que quera usar.

```powershell
Select-AzureSubscription -SubscriptionName "Name of subscription"
```

### <a name="2-view-the-network-configuration-file-values"></a>2. Visualización de los valores de archivo de configuración de red

Cuando crea una red virtual en Azure Portal, el nombre completo que Azure usa no aparece en Azure Portal. Por ejemplo, una red virtual que parece llamarse "ClassicVNet" en Azure Portal puede que tenga un nombre mucho más largo en el archivo de configuración de la red. El nombre debería parecerse al siguiente: "Group ClassicRG ClassicVNet". En estos pasos se descarga el archivo de configuración de red y se ven los valores.

Cree un directorio en el equipo y, a continuación, exporte el archivo de configuración de red al directorio. En este ejemplo, se exporta el archivo de configuración de red a C:\AzureNet.

```powershell
Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
```

Abra el archivo con un editor de texto y consulte el nombre de la red virtual clásica. Use los nombres que aparecen en el archivo de configuración de red cuando ejecute los cmdlets de PowerShell.

- Los nombres de las redes virtuales aparecen como **VirtualNetworkSite name =**
- Los nombres de los sitios aparecen como **LocalNetworkSite name=**

### <a name="3-create-the-connection"></a>3. Creación de la conexión

Establezca la clave compartida y cree la conexión desde la red virtual clásica a la red virtual de Resource Manager. No se puede establecer la clave compartida mediante el portal. Asegúrese de ejecutar estos pasos mientras la sesión esté iniciada con la versión clásica de los cmdlets de PowerShell. Para ello, use **Add-AzureAccount**. De lo contrario, no podrá establecer '-AzureVNetGatewayKey'.

- En este ejemplo, **-VNetName** es el nombre de la red virtual clásica tal como se encuentra en el archivo de configuración de red. 
- **-LocalNetworkSiteName** es el nombre que especificó para el sitio local, según se encontró en el archivo de configuración de red.
- **-SharedKey** es un valor que se puede generar y especificar. En este ejemplo, hemos utilizado *abc123* pero puede generar algo más complejo. Lo importante es que el valor que especifique aquí debe ser el mismo que el que se especificó al crear la conexión entre la red virtual de Resource Manager y la red virtual clásica.

```powershell
Set-AzureVNetGatewayKey -VNetName "Group ClassicRG ClassicVNet" `
-LocalNetworkSiteName "172B9E16_RMVNetLocal" -SharedKey abc123
```

## <a name="section-6---verify-your-connections"></a><a name="verify"></a>Sección 6: Comprobación de las conexiones

Puede comprobar las conexiones mediante Azure Portal o PowerShell. Al comprobar, es posible que necesite esperar un minuto o dos mientras se crea la conexión. Cuando una conexión se realiza correctamente, el estado de conectividad cambia de "Conectando" a "Conectado".

### <a name="to-verify-the-connection-from-your-classic-vnet-to-your-resource-manager-vnet"></a>Comprobación de la conexión de la red virtual clásica a la red virtual de Resource Manager

[!INCLUDE [vpn-gateway-verify-connection-azureportal-classic](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]

### <a name="to-verify-the-connection-from-your-resource-manager-vnet-to-your-classic-vnet"></a>Comprobación de la conexión de la red virtual de Resource Manager a la red virtual clásica

[!INCLUDE [vpn-gateway-verify-connection-portal-rm](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## <a name="vnet-to-vnet-faq"></a><a name="faq"></a>P+F sobre conexiones de red virtual a red virtual

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]
