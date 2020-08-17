---
title: 'Tutorial: Protección de un centro virtual mediante Azure Firewall Manager'
description: En este tutorial aprenderá a proteger un centro virtual con Azure Firewall Manager mediante Azure Portal.
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: tutorial
ms.date: 07/29/2020
ms.author: victorh
ms.openlocfilehash: 458ebe14e77c7b190a5c4cdd9b408396589d5d27
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420828"
---
# <a name="tutorial-secure-your-virtual-hub-using-azure-firewall-manager"></a>Tutorial: Protección de un centro virtual mediante Azure Firewall Manager

Mediante Azure Firewall Manager puede crear centros virtuales protegidos y así proteger el tráfico en la nube destinado a direcciones IP privadas, PaaS de Azure e Internet. El enrutamiento del tráfico al firewall es automático, por lo que no es necesario crear rutas definidas por el usuario (UDR).

![protección de la red en la nube](media/secure-cloud-network/secure-cloud-network.png)

Firewall Manager también admite una arquitectura de red virtual de centro. Para ver una comparación entre los tipos de arquitectura de red virtual de centro y centro virtual protegido, consulte [¿Cuáles son las opciones de arquitectura de Azure Firewall Manager?](vhubs-and-vnets.md)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear la red virtual de tipo hub-and-spoke
> * Crear un centro virtual protegido
> * Conectar las redes virtuales de tipo hub-and-spoke
> * Enrutamiento del tráfico al centro
> * Implementación de los servidores
> * Creación de una directiva de firewall y protección del centro
> * Probar el firewall

## <a name="create-a-hub-and-spoke-architecture"></a>Creación de una arquitectura en estrella tipo hub-and-spoke

En primer lugar, cree redes virtuales de radio en las que pueda colocar los servidores.

### <a name="create-two-spoke-virtual-networks-and-subnets"></a>Creación de dos redes virtuales de radio y subredes

Cada una de las dos redes virtuales tendrá un servidor de carga de trabajo y se protegerá con el firewall.

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
2. En **Redes**, seleccione **Red virtuales**.
2. En **Suscripción**, seleccione la suscripción.
1. En **Grupo de recursos**, seleccione **Crear nuevo** y escriba **fw-manager** como nombre y seleccione **Aceptar**.
2. En **Nombre**, escriba **Spoke-01**.
3. En **Región**, seleccione **(EE. UU.) Este de EE. UU.** .
4. Seleccione **Siguiente: Direcciones IP**.
1. En **Espacio de direcciones**, escriba **10.1.0.0/16**.
3. Haga clic en **Agregar subred**.
4. Escriba **Workload-01-SN**.
5. En **Intervalo de direcciones de subred**, escriba **10.1.1.0/24**.
6. Seleccione **Agregar**.
1. Seleccione **Revisar y crear**.
2. Seleccione **Crear**.

Repita este procedimiento para crear otra red virtual parecida:

Nombre: **Spoke-02**<br>
Espacio de direcciones: **10.2.0.0/16**<br>
Nombre de subred: **Workload-02-SN**<br>
Intervalo de direcciones de subred: **10.2.1.0/24**

### <a name="create-the-secured-virtual-hub"></a>Creación del centro virtual protegido

Cree el centro virtual protegido con Firewall Manager.

1. En la página principal de Azure Portal, seleccione **Todos los servicios**.
2. En el cuadro de búsqueda, escriba **Firewall Manager** y seleccione **Firewall Manager**.
3. En la página **Firewall Manager**, seleccione **Ver los centros virtuales seguros**.
4. En la página **Firewall Manager | Centros virtuales protegidos**, seleccione **Crear un nuevo centro virtual protegido**.
5. En **Grupo de recursos**, seleccione **fw-manager**.
7. En **Región**, seleccione **Este de EE. UU.** .
1. En **Nombre del centro virtual protegido**, escriba **Hub-01**.
2. En **Espacio de direcciones del concentrador**, escriba **10.0.0.0/16**.
3. Como nombre de la nueva WAN virtual, escriba **Vwan-01**.
4. Deje desactivada la casilla **Incluir VPN Gateway para habilitar asociados de seguridad de confianza**.
5. Seleccione **Siguiente: Azure Firewall**.
6. Acepte el valor predeterminado **Azure Firewall** **Habilitado** y seleccione **Siguiente: Asociados de seguridad de confianza**.
7. Acepte el valor predeterminado **Asociado de seguridad de confianza** **Deshabilitado** y seleccione **Siguiente: Review + create** (Revisar y crear).
8. Seleccione **Crear**. La implementación tardará unos 30 minutos.

Ahora puede obtener la dirección IP pública del firewall.

1. Una vez finalizada la implementación, en Azure Portal, seleccione **Todos los servicios**.
1. Escriba **firewall manager** y después seleccione **Firewall Manager**.
2. Seleccione **Centros virtuales protegidos**.
3. Seleccione **hub-01**.
7. Seleccione **Configuración de IP pública**.
8. Anote la dirección IP pública para usarla más tarde.

### <a name="connect-the-hub-and-spoke-virtual-networks"></a>Conexión de las redes virtuales de tipo hub-and-spoke

Ahora, podrá emparejar las redes virtuales de tipo hub-and-spoke.

1. Seleccione el grupo de recursos **fw-manager** y, después, seleccione la WAN virtual **Vwan-01**.
2. En **Conectividad**, seleccione **Conexiones de red virtual**.
3. Seleccione **Agregar conexión**.
4. En **Nombre de conexión**, escriba **hub-spoke-01**.
5. En **Centros**, seleccione **Hub-01**.
6. En **Grupo de recursos**, seleccione **fw-manager**.
7. En **Red virtual**, seleccione **Spoke-01**.
8. Seleccione **Crear**.

Repita el procedimiento para conectar la red virtual **Spoke-02** con el nombre de conexión **hub-spoke-02**.

### <a name="configure-the-hub-and-spoke-routing"></a>Configuración del enrutamiento de tipo hub-and-spoke

En Azure Portal, abra una instancia de Cloud Shell y ejecute el siguiente comando de Azure PowerShell para configurar el enrutamiento de tipo hub-and-spoke necesario. En las conexiones de radios/ramas del mismo nivel es preciso establecer la propagación en **NO**. Así se evita la comunicación universal entre los radios y, en su lugar, enruta el tráfico al firewall a través de la ruta predeterminada.

```azurepowershell
$noneRouteTable = Get-AzVHubRouteTable -ResourceGroupName fw-manager `
                  -HubName hub-01 -Name noneRouteTable
$vnetConns = Get-AzVirtualHubVnetConnection -ResourceGroupName fw-manager `
             -ParentResourceName hub-01

$vnetConn = $vnetConns[0]
$vnetConn.RoutingConfiguration.PropagatedRouteTables.Ids = @($noneRouteTable)
$vnetConn.RoutingConfiguration.PropagatedRouteTables.Labels = @("none")
Update-AzVirtualHubVnetConnection -ResourceGroupName fw-manager `
   -ParentResourceName hub-01 -Name $vnetConn.Name `
   -RoutingConfiguration $vnetConn.RoutingConfiguration

$vnetConn = $vnetConns[1]
$vnetConn.RoutingConfiguration.PropagatedRouteTables.Ids = @($noneRouteTable)
$vnetConn.RoutingConfiguration.PropagatedRouteTables.Labels = @("none")
Update-AzVirtualHubVnetConnection -ResourceGroupName fw-manager `
   -ParentResourceName hub-01 -Name $vnetConn.Name -RoutingConfiguration $vnetConn.RoutingConfiguration
```

## <a name="deploy-the-servers"></a>Implementación de los servidores

1. En Azure Portal, seleccione **Crear un recurso**.
2. Seleccione **Windows Server 2016 Datacenter** en la lista **Popular**.
3. Especifique estos valores para la máquina virtual:

   |Configuración  |Value  |
   |---------|---------|
   |Resource group     |**fw-manager**|
   |Nombre de la máquina virtual     |**Srv-workload-01**|
   |Region     |**(EE. UU.) Este de EE. UU.**|
   |Nombre de usuario del administrador     |Escriba un nombre de usuario|
   |Contraseña     |Escriba una contraseña|

4. En **Reglas de puerto de entrada**, para **Puertos de entrada públicos**, seleccione **Ninguno**.
6. Acepte los restantes valores predeterminados y seleccione **Siguiente: Discos**.
7. Acepte los valores predeterminados del disco y seleccione **Siguiente: Redes**.
8. Seleccione **Spoke-01** para la red virtual y seleccione **Workload-01-SN** para la subred.
9. En **IP pública**, seleccione **Ninguno**.
11. Acepte los restantes valores predeterminados y seleccione **Siguiente: Administración**.
12. Seleccione **Desactivar** para deshabilitar los diagnósticos de arranque. Acepte los restantes valores predeterminados y seleccione **Revisar y crear**.
13. Revise la configuración en la página de resumen y seleccione **Crear**.

Use la información de la tabla siguiente para configurar otra máquina virtual llamada **Srv-Workload-02**. El resto de la configuración es la misma que la de la máquina virtual **Srv-workload-01**.

|Configuración  |Value  |
|---------|---------|
|Virtual network|**Spoke-02**|
|Subnet|**Workload-02-SN**|

Una vez implementados los servidores, seleccione un recurso de servidor y en **Redes** tenga en cuenta la dirección IP privada de cada servidor.

## <a name="create-a-firewall-policy-and-secure-your-hub"></a>Creación de una directiva de firewall y protección del centro

Una directiva de firewall define colecciones de reglas para dirigir el tráfico en uno o varios centros virtuales protegidos. Creará la directiva de firewall y, a continuación, protegerá el centro.

1. En Firewall Manager, seleccione **Ver las directivas de Azure Firewall**.
2. Seleccione **Crear una directiva de Azure Firewall**.
3. En **Detalles de la directiva**, como **Nombre** escriba **Policy-01** y como **Región** seleccione **Este de EE. UU.**
4. Seleccione **Siguiente: Configuración DNS (versión preliminar)** .
1. Seleccione **Siguiente: Las reglas**.
2. En la pestaña **Reglas**, seleccione **Agregar una colección de reglas**.
3. En la página **Agregar una colección de reglas**, escriba **App-RC-01** en **Nombre**.
4. En **Tipo de colección de reglas**, seleccione **Aplicación**.
5. En **Prioridad**, escriba **100**.
6. Asegúrese de que el valor de **Rule collection action** (Acción de la colección de reglas) es **Permitir**.
7. Como **Nombre** de la regla, escriba **Allow-msft**.
8. Para **Tipo de origen**, seleccione **Dirección IP**.
9. Para **Origen**, escriba **\*** .
10. En **Protocolo**, escriba **http,https**.
11. Asegúrese de que **Tipo de destino** es **FQDN**.
12. En **Destino**, escriba **\*.microsoft.com**.
13. Seleccione **Agregar**.

Agregue una regla DNAT para poder conectar un escritorio remoto a la máquina virtual **Srv-Workload-01**.

1. Seleccione **Agregar una colección de reglas**.
2. En **Nombre**, escriba **DNAT-rdp**.
3. En **Tipo de colección de reglas**, seleccione **DNAT**.
4. En **Prioridad**, escriba **100**.
5. Como **Nombre** de la regla, escriba **Allow-rdp**.
6. Para **Tipo de origen**, seleccione **Dirección IP**.
7. Para **Origen**, escriba **\*** .
8. En **Protocolo**, seleccione **TCP**.
9. En **Puertos de destino**, escriba **3389**.
10. En **Tipo de destino**, seleccione **Dirección IP**.
11. En **Destino**, escriba la dirección IP pública del firewall que anotó anteriormente.
12. En **Dirección traducida**, escriba la dirección IP privada para **Srv-Workload-01** que anotó anteriormente.
13. En **Puerto traducido**, escriba **3389**.
14. Seleccione **Agregar**.

Agregue una regla de red para conectar un escritorio remoto de **Srv-Workload-01** a **Srv-Workload-02**.

1. Seleccione **Agregar una colección de reglas**.
2. En **Nombre**, escriba **vnet-rdp**.
3. En **Tipo de colección de reglas**, seleccione **Red**.
4. En **Prioridad**, escriba **100**.
5. Como **Nombre** de la regla, escriba **Allow-vnet**.
6. Para **Tipo de origen**, seleccione **Dirección IP**.
7. Para **Origen**, escriba **\*** .
8. En **Protocolo**, seleccione **TCP**.
9. En **Puertos de destino**, escriba **3389**.
9. En **Tipo de destino**, seleccione **Dirección IP**.
10. En **Destino**, escriba la dirección IP privada de **Srv-Workload-02** que anotó anteriormente.
11. Seleccione **Agregar**.
1. Seleccione **Siguiente: Inteligencia sobre amenazas**.
2. Seleccione **Siguiente: Centros de conectividad**.
3. En la pestaña **Centros de conectividad**, seleccione **Asociar centros virtuales**.
4. Seleccione **Hub-01** y, luego, seleccione **Agregar**.
5. Seleccione **Revisar + crear**.
6. Seleccione **Crear**.

Esta acción puede tardar unos cinco minutos o más en finalizar.

## <a name="route-traffic-to-your-hub"></a>Enrutamiento del tráfico al centro

Ahora debe asegurarse de que el tráfico de red se enruta a través del firewall.

1. En Firewall Manager, seleccione **Centros virtuales protegidos**.
2. Seleccione **Hub-01**.
3. En **Ajustes**, seleccione **Configuración de seguridad**.
4. En **Tráfico de Internet**, seleccione **Azure Firewall**.
5. En **Private traffic** (Tráfico privado), seleccione **Send via Azure Firewall** (Enviar a través de Azure Firewall).
10. Compruebe que la conexión **hub-spoke** muestra **Tráfico de Internet** como **Protegido**.
11. Seleccione **Guardar**.


## <a name="test-your-firewall"></a>Prueba del firewall

Para probar las reglas de firewall, deberá conectar un escritorio remoto mediante la dirección IP pública del firewall, que se conecta mediante NAT a **Srv-Workload-01**. Desde allí, deberá usar un explorador para probar la regla de aplicación y conectar un escritorio remoto a **Srv-Workload-02** para probar la regla de red.

### <a name="test-the-application-rule"></a>Prueba de la regla de aplicación

Ahora, pruebe las reglas de firewall para confirmar que funcionan según lo previsto.

1. Conecte un escritorio remoto a la dirección IP pública del firewall e inicie sesión.

3. Abra Internet Explorer y vaya a https://www.microsoft.com.
4. Seleccione **Aceptar** > **Cerrar** en las alertas de seguridad de Internet Explorer.

   Debería ver la página principal de Microsoft.

5. Vaya a https://www.google.com.

   El firewall debería bloquearle.

Con ello, ha comprobado que la regla de aplicación de firewall funciona:

* Puede navegar al FQDN permitido pero no a ningún otro.

### <a name="test-the-network-rule"></a>Prueba de la regla de red

Ahora pruebe la regla de red.

- Abra un escritorio remoto conectado a la dirección IP privada de **Srv-Workload-02**.

   Un escritorio remoto debe conectarse a **Srv-Workload-02**.

Con ello, ha comprobado que la regla de red de firewall funciona:
* Puede conectar un escritorio remoto a un servidor ubicado en otra red virtual.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre los asociados de seguridad de confianza](trusted-security-partners.md)
