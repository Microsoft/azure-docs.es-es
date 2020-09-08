---
title: Conexión de aplicaciones a Instancia administrada de SQL
titleSuffix: Azure SQL Managed Instance
description: En este artículo se describe la conexión de la aplicación a Instancia administrada de Azure SQL.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova, carlrab, vanto
ms.date: 11/09/2018
ms.openlocfilehash: a04f4879bbd06c2fa6c1af921d7adafdef9417d6
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88871452"
---
# <a name="connect-your-application-to-azure-sql-managed-instance"></a>Conexión de la aplicación a Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Hoy en día puede elegir entre varias opciones sobre cuándo y cómo hospedar la aplicación.

Puede elegir hospedar la aplicación en la nube mediante Azure App Service o algunas de las opciones integradas de red virtual de Azure, como Azure App Service Environment, Azure Virtual Machines y conjuntos de escalado de máquinas virtuales. También podría elegir un enfoque de nube híbrida y mantener las aplicaciones en local.

Sea cual sea su opción, puede conectarla a Instancia administrada de Azure SQL. 

![Alta disponibilidad](./media/connect-application-instance/application-deployment-topologies.png)

En este artículo se describe cómo conectar una aplicación a Instancia administrada de Azure SQL en varios escenarios de aplicación diferentes. 

## <a name="connect-inside-the-same-vnet"></a>Conexión dentro de la misma red virtual

La conexión de una aplicación dentro de la misma red virtual que Instancia administrada de SQL es el escenario más sencillo. Las máquinas virtuales dentro de la red virtual pueden conectarse directamente entre sí si se encuentran en subredes diferentes. Esto significa que para conectar una aplicación dentro de Azure Application Environment o de una máquina virtual solo hace falta establecer correctamente la cadena de conexión.  

## <a name="connect-inside-a-different-vnet"></a>Conexión dentro de una red virtual diferente

Conectar una aplicación cuando reside en una red virtual diferente a la de Instancia administrada de SQL es un poco más complejo porque esta opción tiene direcciones IP privadas en su propia red virtual. Para conectarse, una aplicación necesita acceso a la red virtual donde está implementada la opción Instancia administrada de SQL. Por lo tanto, primero debe establecer una conexión entre la aplicación y la red virtual de Instancia administrada de SQL. Las redes virtuales no tienen por qué estar en la misma suscripción para que este escenario funcione.

Para conectar redes virtuales hay dos opciones:

- [Emparejamiento de VNet de Azure](../../virtual-network/virtual-network-peering-overview.md)
- Puerta de enlace VPN de red virtual a red virtual ([Azure Portal](../../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md), [PowerShell](../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md), [CLI de Azure](../../vpn-gateway/vpn-gateway-howto-vnet-vnet-cli.md))

El emparejamiento es la opción preferida porque se usa la red troncal de Microsoft y, por lo tanto, desde la perspectiva de la conectividad, no hay diferencias notables de latencia entre las máquinas virtuales de la red virtual emparejada y la propia. El emparejamiento de red virtual está restringido a las redes de la misma región.  

> [!IMPORTANT]
> El escenario de emparejamiento de red virtual para Instancia administrada de SQL está restringido a las redes de la misma región debido a las [restricciones del emparejamiento de red virtual global](../../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints). Consulte también la sección correspondiente del artículo [Preguntas más frecuentes (P+F) acerca de Azure Virtual Network](https://docs.microsoft.com/azure/virtual-network/virtual-networks-faq#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers) para más información. 

## <a name="connect-from-on-premises"></a>Conexión desde el entorno local 

También puede conectar la aplicación local a Instancia administrada de SQL. El acceso a esta opción solo se puede realizar mediante una dirección IP privada. Para hacerlo desde el entorno local, debe establecer una conexión de sitio a sitio entre la aplicación y la red virtual de Instancia administrada de SQL.

Hay dos opciones para conectarse de forma local a una red virtual de Azure:

- Conexión VPN de sitio a sitio ([Azure Portal](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md), [PowerShell](../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md), [CLI de Azure](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md))
- Conexión de [Azure ExpressRoute](../../expressroute/expressroute-introduction.md)  

Si ha establecido una conexión local a Azure correctamente y no puede establecer una conexión a Instancia administrada de SQL, compruebe si el firewall tiene abierta la conexión saliente en el puerto 1433 de SQL, así como el intervalo de puertos 11000-11999 para el redireccionamiento.

## <a name="connect-the-developer-box"></a>Conexión del cuadro de desarrollador

También es posible conectar el cuadro de desarrollador a Instancia administrada de SQL. El acceso a Instancia administrada de SQL solo se puede realizar mediante una dirección IP privada, por ello, para hacerlo desde el cuadro de desarrollador, primero debe realizar una conexión entre el cuadro de desarrollador y la red virtual de Instancia administrada de SQL. Para ello, configure una conexión de punto a sitio a una red virtual mediante la autenticación de certificados de Azure nativa. Para más información, consulte [Configuración de una conexión de punto a sitio para conectarse a Instancia administrada de Azure SQL desde el equipo local](point-to-site-p2s-configure.md).

## <a name="connect-with-vnet-peering"></a>Conexión con emparejamiento de red virtual

Otro escenario implementado por los clientes es cuando una puerta de enlace de VPN está instalada en una red virtual y una suscripción distintas a aquellas que hospedan Instancia administrada de SQL. Entonces, las dos redes virtuales se emparejan. El siguiente diagrama de arquitectura de ejemplo muestra cómo se puede realizar esta implementación.

![Emparejamiento de redes virtuales de Azure](./media/connect-application-instance/vnet-peering.png)

Una vez configurada la infraestructura básica, deberá modificar alguna configuración para que la puerta de enlace de VPN pueda ver las direcciones IP en la red virtual que hospeda Instancia administrada de SQL. Para ello, realice los siguientes cambios específicos en la **Configuración de emparejamiento**.

1. En la red virtual que hospeda la puerta de enlace de VPN, vaya a **Emparejamientos**, vaya a la conexión de red virtual emparejada de Instancia administrada de SQL y, luego, haga clic en **Permitir tránsito de puerta de enlace**.
2. En la red virtual que hospeda Instancia administrada de SQL, vaya a **Emparejamientos**, vaya a la conexión de red virtual emparejada de la puerta de enlace de VPN y, luego, haga clic en **Usar puertas de enlace remotas**.

## <a name="connect-azure-app-service"></a>Conexión de Azure App Service 

También puede conectar una aplicación hospedada por Azure App Service. El acceso a Instancia administrada de SQL se puede realizar únicamente mediante una dirección IP privada, por ello, para hacerlo desde Azure App Service, primero debe realizar una conexión entre la aplicación y la red virtual de Instancia administrada de SQL. Consulte [Integración de aplicaciones con una red virtual de Azure](../../app-service/web-sites-integrate-with-vnet.md).  

Para solucionar problemas, consulte [Solución de problemas con redes virtuales y aplicaciones](../../app-service/web-sites-integrate-with-vnet.md#troubleshooting). Si no se puede establecer una conexión, intente [sincronizar la configuración de red](azure-app-sync-network-configuration.md).

Un caso especial de conexión de Azure App Service a Instancia administrada de SQL es cuando se integra Azure App Service en una red emparejada a una red virtual de Instancia administrada de SQL. Ese caso requiere la siguiente configuración:

- La red virtual de Instancia administrada de SQL no debe tener una puerta de enlace.  
- La red virtual de Instancia administrada de SQL debe tener establecida la opción `Use remote gateways`.
- La red virtual emparejada debe tener establecida la opción `Allow gateway transit`.

Este escenario se ilustra en el diagrama siguiente:

![emparejamiento de aplicación integrada](./media/connect-application-instance/integrated-app-peering.png)

>[!NOTE]
>La característica de integración de la red virtual no integra una aplicación con una red virtual que tenga una puerta de enlace de ExpressRoute. Incluso si la puerta de enlace de ExpressRoute está configurada en modo de coexistencia, la integración de la red virtual no funcionará. Si necesita acceder a recursos mediante una conexión de ExpressRoute, puede usar App Service Environment, que funciona en la red virtual.

## <a name="troubleshooting-connectivity-issues"></a>Solución de problemas de conectividad

Para solucionar problemas de conectividad, consulte la siguiente información:

- Si no puede conectarse a Instancia administrada de SQL desde una máquina virtual de Azure dentro de la misma red virtual, pero una subred diferente, compruebe que no haya un grupo de seguridad de red establecido en la subred de la máquina virtual que bloquee el acceso. Además, abra la conexión saliente en el puerto 1433 de SQL, así como los puertos del intervalo 11000-11999, ya que son necesarios para conectarse mediante redireccionamiento dentro del límite de Azure.
- Asegúrese de que la propagación de BGP está establecida en **Habilitado** para la tabla de rutas asociada a la red virtual.
- Si utiliza VPN de P2S, compruebe la configuración en Azure Portal para ver si detecta números de **entrada/salida**. Los números distintos de cero indican que Azure enruta el tráfico hacia y desde un entorno local.

   ![Números de entrada/salida](./media/connect-application-instance/ingress-egress-numbers.png)

- Compruebe que la máquina cliente (que ejecuta el cliente de VPN) tenga entradas de ruta para todas las redes virtuales a las que necesita acceso. Las rutas se almacenan en `%AppData%\ Roaming\Microsoft\Network\Connections\Cm\<GUID>\routes.txt`.

   ![route.txt](./media/connect-application-instance/route-txt.png)

   Como se muestra en esta imagen, hay dos entradas para cada red virtual implicada y una tercera entrada para el punto de conexión de VPN que está configurado en el portal.

   Otra manera de comprobar las rutas es mediante el comando siguiente. La salida muestra las rutas a las diversas subredes:

   ```cmd
   C:\ >route print -4
   ===========================================================================
   Interface List
   14...54 ee 75 67 6b 39 ......Intel(R) Ethernet Connection (3) I218-LM
   57...........................rndatavnet
   18...94 65 9c 7d e5 ce ......Intel(R) Dual Band Wireless-AC 7265
   1...........................Software Loopback Interface 1
   Adapter===========================================================================

   IPv4 Route Table
   ===========================================================================
   Active Routes:
   Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0       10.83.72.1     10.83.74.112     35
         10.0.0.0    255.255.255.0         On-link       172.26.34.2     43
         10.4.0.0    255.255.255.0         On-link       172.26.34.2     43
   ===========================================================================
   Persistent Routes:
   None
   ```

- Si usa el emparejamiento de red virtual, asegúrese de que ha seguido las instrucciones para configurar [Permitir tránsito de puerta de enlace y Usar puertas de enlace remotas](#connect-from-on-premises).

- Si usa el emparejamiento de red virtual para conectar una aplicación hospedada por Azure App Service, y la red virtual de Instancia administrada de SQL tiene un intervalo de direcciones IP públicas, asegúrese de que la configuración de la aplicación hospedada permita que el tráfico saliente se enrute a redes con direcciones IP públicas. Siga las instrucciones que se indican en [Integración con red virtual regional](../../app-service/web-sites-integrate-with-vnet.md#regional-vnet-integration).

## <a name="required-versions-of-drivers-and-tools"></a>Versiones necesarias de los controladores y las herramientas

Si quiere conectarse a Instancia administrada de SQL, se recomiendan las siguientes versiones mínimas de las herramientas y los controladores:

| Controlador/Herramienta | Versión |
| --- | --- |
|.NET Framework | 4.6.1 o .NET Core |
|Controlador ODBC| v17 |
|Controlador PHP| 5.2.0 |
|Controlador JDBC| 6.4.0 |
|Controlador de Node.js| 2.1.1 |
|Controlador de OLEDB| 18.0.2.0 |
|SSMS| 18.0 o [superior](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) |
|[SMO](https://docs.microsoft.com/sql/relational-databases/server-management-objects-smo/sql-server-management-objects-smo-programming-guide) | [150](https://www.nuget.org/packages/Microsoft.SqlServer.SqlManagementObjects) o superior |

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre Instancia administrada de SQL, consulte [¿Qué es Instancia administrada de SQL?](sql-managed-instance-paas-overview.md)
- Se puede encontrar un tutorial que muestra cómo crear una instancia administrada en [Creación de instancias administradas](instance-create-quickstart.md).
