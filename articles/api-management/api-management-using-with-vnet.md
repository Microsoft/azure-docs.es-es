---
title: Usar Azure API Management con redes virtuales
description: Obtenga información sobre cómo configurar una conexión a una red virtual en Azure API Management y acceder a servicios web con esta.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/22/2020
ms.author: apimpm
ms.custom: references_regions
ms.openlocfilehash: ee23b2bc58f8c1f15a7e51b05dee954c1e584293
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489629"
---
# <a name="how-to-use-azure-api-management-with-virtual-networks"></a>Usar Azure API Management con redes virtuales
Azure Virtual Network (redes virtuales) le permiten colocar cualquier recurso de Azure en una red distinta de Internet que se pueda enrutar y a la que controle el acceso. Después, estas redes se pueden conectar a sus redes locales mediante diversas tecnologías de VPN. Para más información sobre Azure Virtual Network, vea: [Información general sobre Azure Virtual Network](../virtual-network/virtual-networks-overview.md).

Azure API Management se puede implementar dentro de la red virtual (VNET), por lo que puede tener acceso a los servicios back-end dentro de la red. El portal para desarrolladores y la puerta de enlace de API pueden configurarse para que sea accesible desde Internet o solo dentro de la red virtual.

> [!NOTE]
> La dirección URL del documento de importación de API debe estar hospedada en una dirección de Internet de acceso público.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [premium-dev.md](../../includes/api-management-availability-premium-dev.md)]

## <a name="prerequisites"></a>Requisitos previos

Para seguir los pasos que se describen en este artículo, debe tener:

+ Una suscripción de Azure activa.

    [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

+ Una instancia de APIM. Para más información, vea [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).

## <a name="enable-vnet-connection"></a><a name="enable-vpn"> </a>Habilitar la conexión de VNET

### <a name="enable-vnet-connectivity-using-the-azure-portal"></a>Habilitación de la conectividad de VNET mediante Azure Portal

1. Vaya a [Azure Portal](https://portal.azure.com) para buscar la instancia API Management. Busque y seleccione **Servicios API Management**.

2. Elija su instancia de API Management.

3. Seleccione **Red virtual**.
4. Configure la instancia de API Management que se va a implementar dentro de la red virtual.

    ![Menú Red virtual de API Management][api-management-using-vnet-menu]
5. Seleccione el tipo de acceso que prefiera:

    * **Off**: Este es el valor predeterminado. API Management no se implementa en una red virtual.

    * **Externas**: la puerta de enlace de API Management y el portal para desarrolladores son accesibles públicamente desde Internet con un equilibrador de carga externo. La puerta de enlace puede acceder a recursos dentro de la red virtual.

        ![Emparejamiento público][api-management-vnet-public]

    * **Internas**: la puerta de enlace de API Management y el portal para desarrolladores solo son accesibles desde la red virtual con un equilibrador de carga interno. La puerta de enlace puede acceder a recursos dentro de la red virtual.

        ![Emparejamiento privado][api-management-vnet-private]

6. Si seleccionó **Externo** o **Interno**, verá una lista de todas las regiones donde se aprovisiona el servicio de API Management. Elija una **Ubicación** y, luego, la **Red virtual** y la **Subred**. La lista de redes virtuales se rellena con redes virtuales de Resource Manager y clásicas disponibles en las suscripciones de Azure que se configuran en la región que va a configurar.

    > [!IMPORTANT]
    > Al implementar una instancia de Azure API Management en una VNET de Resource Manager, el servicio debe estar en una subred dedicada que no contiene ningún otro recurso excepto instancias de Azure API Management. Si se intenta implementar una instancia de Azure API Management en una subred de VNET de Resource Manager que contiene otros recursos, se producirá un error en la implementación.

    Después, seleccione **Aplicar**. La página **Red virtual** de la instancia API Management se actualiza con las opciones de red virtual y subred nueva.

    ![Selección de una VPN][api-management-setup-vpn-select]

7. En la barra de navegación superior, seleccione **Guardar** y, luego, **Aplicar configuración de red**.

> [!NOTE]
> Tenga en cuenta que la dirección VIP de la instancia de API Management cambiará cada vez que se habilita o deshabilita VNET.
> La dirección VIP también cambia cuando se mueve API Management de **externo** a **interno** o viceversa.
>

> [!IMPORTANT]
> Si elimina API Management de una red virtual o cambia aquella en la que se implementa, la red virtual usada anteriormente puede permanecer bloqueada hasta seis horas. Durante este periodo no será posible eliminar la red virtual ni implementar un nuevo recurso en ella. Este comportamiento se aplica a los clientes que usan la versión 2018-01-01 y anteriores de la API. En los clientes que usan la versión 2019-01-01 y posteriores de la API, la red virtual se libera en cuanto se elimina el servicio de API Management asociado.

## <a name="enable-vnet-connection-using-powershell-cmdlets"></a><a name="enable-vnet-powershell"> </a>Habilitar una conexión de VNET con cmdlets de PowerShell
También puede habilitar la conectividad de VNET con los cmdlets de PowerShell.

* **Crear un servicio de API Management dentro de una red virtual**: use el cmdlet [New-AzApiManagement](/powershell/module/az.apimanagement/new-azapimanagement) para crear un servicio Azure API Management dentro de una red virtual.

* **Implementar un servicio existente de API Management dentro de una VNET**: use el cmdlet [Update-AzApiManagementRegion](/powershell/module/az.apimanagement/update-azapimanagementregion) para mover un servicio Azure API Management existente dentro de una red virtual.

## <a name="connect-to-a-web-service-hosted-within-a-virtual-network"></a><a name="connect-vnet"> </a>Conectar a un servicio web hospedado en una red virtual
Después de conectar el servicio API Management a la VNET, se accede a los servicios de back-end de la misma forma que a los servicios públicos. Solo tiene que escribir la dirección IP local o el nombre de host (si se ha configurado un servidor DNS para la VNET) del servicio web en el campo **Dirección URL de servicio web** al crear una API o editar una existente.

![Agregar una API desde VPN][api-management-setup-vpn-add-api]

## <a name="common-network-configuration-issues"></a><a name="network-configuration-issues"> </a>Problemas comunes de configuración de red
A continuación se muestra una lista de problemas de errores de configuración comunes que pueden producirse al implementar el servicio de API Management en una red virtual.

* **Configuración del servidor DNS personalizado**: el servicio de API Management depende de varios servicios de Azure. Cuando API Management está hospedado en una red virtual con un servidor DNS personalizado, necesita resolver los nombres de host de esos servicios de Azure. Siga [estas](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) instrucciones sobre la configuración de DNS personalizado. Vea la siguiente tabla de puertos y otros requisitos de red a efectos de referencia.

> [!IMPORTANT]
> Si planea utilizar un servidor DNS personalizado para la red virtual, debe configurarlo **antes** de implementar en él un servicio API Management. En caso contrario, deberá actualizar el servicio API Management cada vez que cambie los servidores DNS mediante la ejecución de la [operación Aplicar configuración de red](/rest/api/apimanagement/2019-12-01/apimanagementservice/applynetworkconfigurationupdates)

* **Puertos necesarios para API Management**: el tráfico entrante y saliente en la subred en la que se implementa API Management puede controlarse mediante el [grupo de seguridad de red][Network Security Group]. Si alguno de estos puertos no está disponible, es posible que API Management no funcione correctamente y sea inaccesible. Tener bloqueados uno o varios de estos puertos es otro problema común de una configuración incorrecta cuando se usa API Management con una red virtual.

<a name="required-ports"> </a> Cuando la instancia del servicio API Management se hospeda en una red virtual, se usan los puertos de la tabla siguiente.

| Puertos de origen/destino | Dirección          | Protocolo de transporte |   [Etiquetas de servicio](../virtual-network/security-overview.md#service-tags) <br> Origen/destino   | Propósito (\*)                                                 | Tipo de red virtual |
|------------------------------|--------------------|--------------------|---------------------------------------|-------------------------------------------------------------|----------------------|
| * / [80], 443                  | Entrada            | TCP                | INTERNET/VIRTUAL_NETWORK            | Comunicación de cliente con Administración de API                      | Externo             |
| * / 3443                     | Entrada            | TCP                | ApiManagement / VIRTUAL_NETWORK       | Punto de conexión de administración para Azure Portal y PowerShell         | Externa e interna  |
| * / 443                  | Salida           | TCP                | VIRTUAL_NETWORK/Storage             | **Dependencia de Azure Storage**                             | Externa e interna  |
| * / 443                  | Salida           | TCP                | VIRTUAL_NETWORK / AzureActiveDirectory | [Azure Active Directory](api-management-howto-aad.md) (si procede)                   | Externa e interna  |
| * / 1433                     | Salida           | TCP                | VIRTUAL_NETWORK / SQL                 | **Acceso a los puntos de conexión de Azure SQL**                           | Externa e interna  |
| * / 5671, 5672, 443          | Salida           | TCP                | VIRTUAL_NETWORK/EventHub            | Dependencia de la [directiva de registro en el centro de eventos](api-management-howto-log-event-hubs.md) y el agente de supervisión | Externa e interna  |
| * / 445                      | Salida           | TCP                | VIRTUAL_NETWORK/Storage             | Dependencia del recurso compartido de archivos de Azure para [GIT](api-management-configuration-repository-git.md)                      | Externa e interna  |
| * / 443, 12 000                     | Salida           | TCP                | VIRTUAL_NETWORK / AzureCloud            | Extensión Estado y supervisión         | Externa e interna  |
| * / 1886, 443                     | Salida           | TCP                | VIRTUAL_NETWORK / AzureMonitor         | Publicar [métricas y registros de diagnóstico](api-management-howto-use-azure-monitor.md), [Resource Health](../service-health/resource-health-overview.md) y [Application Insights](api-management-howto-app-insights.md)                   | Externa e interna  |
| * / 25, 587, 25028                       | Salida           | TCP                | VIRTUAL_NETWORK/INTERNET            | Conexión a la retransmisión de SMTP para enviar correos electrónicos                    | Externa e interna  |
| * / 6381 - 6383              | Entrada y salida | TCP                | VIRTUAL_NETWORK/VIRTUAL_NETWORK     | Acceso al servicio de Redis para las directivas de [almacenamiento en memoria caché](api-management-caching-policies.md) entre máquinas         | Externa e interna  |
| * / 4290              | Entrada y salida | UDP                | VIRTUAL_NETWORK/VIRTUAL_NETWORK     | Sincronización de contadores para las directivas de [límite de velocidad](api-management-access-restriction-policies.md#LimitCallRateByKey) entre máquinas         | Externa e interna  |
| * / *                        | Entrada            | TCP                | AZURE_LOAD_BALANCER / VIRTUAL_NETWORK | Equilibrador de carga de la infraestructura de Azure                          | Externa e interna  |

>[!IMPORTANT]
> Los puertos para los que el *Propósito* está en **negrita** son necesarios para que el servicio API Management se implemente correctamente. Sin embargo, si se bloquean los otros puertos, se producirá la **degradación** de la capacidad de usar y **supervisar el servicio en ejecución y proporcionar el SLA comprometido**.

+ **Funcionalidad de TLS**: para permitir la creación y validación de la cadena de certificados TLS/SSL, el servicio API Management necesita conectividad de red saliente a ocsp.msocsp.com, mscrl.microsoft.com y crl.microsoft.com. Esta dependencia no es obligatoria, si los certificados que cargue en API Management contienen la cadena completa de la raíz de la entidad de certificación.

+ **Acceso DNS**: Se requiere acceso saliente en el puerto 53 para establecer la comunicación con los servidores DNS. Si existe un servidor DNS personalizado en el otro punto de conexión de una puerta de enlace de VPN, el servidor DNS debe estar accesible desde la subred que alberga la API Management.

+ **Supervisión de métricas y estado**: conectividad de red saliente a puntos de conexión de supervisión de Azure, que se resuelven en los siguientes dominios. Como se muestra en la tabla, estas direcciones URL se representan en la etiqueta de servicio de AzureMonitor para su uso con grupos de seguridad de red.

    | Entorno de Azure | Puntos de conexión                                                                                                                                                                                                                                                                                                                                                              |
    |-------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | Azure Public      | <ul><li>gcs.prod.monitoring.core.windows.net(**nuevo**)</li><li>prod.warmpath.msftcloudes.com (**próximamente en desuso**)</li><li>global.prod.microsoftmetrics.com (**nuevo**)</li><li>global.metrics.nsatc.net (**próximamente en desuso**)</li><li>shoebox2.prod.microsoftmetrics.com (**nuevo**)</li><li>shoebox2.metrics.nsatc.net (**próximamente en desuso**)</li><li>shoebox2-red.prod.microsoftmetrics.com</li><li>shoebox2-black.prod.microsoftmetrics.com</li><li>shoebox2-red.shoebox2.metrics.nsatc.net</li><li>shoebox2-black.shoebox2.metrics.nsatc.net</li><li>prod3.prod.microsoftmetrics.com (**nuevo**)</li><li>prod3.metrics.nsatc.net (**próximamente en desuso**)</li><li>prod3-black.prod.microsoftmetrics.com (**nuevo**)</li><li>prod3-black.prod3.metrics.nsatc.net (**próximamente en desuso**)</li><li>prod3-red.prod.microsoftmetrics.com (**nuevo**)</li><li>prod3-red.prod3.metrics.nsatc.net (**próximamente en desuso**)</li><li>gcs.prod.warm.ingestion.monitoring.azure.com</li></ul> |
    | Azure Government  | <ul><li>fairfax.warmpath.usgovcloudapi.net</li><li>global.prod.microsoftmetrics.com (**nuevo**)</li><li>global.metrics.nsatc.net (**próximamente en desuso**)</li><li>shoebox2.prod.microsoftmetrics.com (**nuevo**)</li><li>shoebox2.metrics.nsatc.net (**próximamente en desuso**)</li><li>shoebox2-red.prod.microsoftmetrics.com</li><li>shoebox2-black.prod.microsoftmetrics.com</li><li>shoebox2-red.shoebox2.metrics.nsatc.net</li><li>shoebox2-black.shoebox2.metrics.nsatc.net</li><li>prod3.prod.microsoftmetrics.com (**nuevo**)</li><li>prod3.metrics.nsatc.net (**próximamente en desuso**)</li><li>prod3-black.prod.microsoftmetrics.com</li><li>prod3-red.prod.microsoftmetrics.com</li><li>prod5.prod.microsoftmetrics.com</li><li>prod5-black.prod.microsoftmetrics.com</li><li>prod5-red.prod.microsoftmetrics.com</li><li>gcs.prod.warm.ingestion.monitoring.azure.us</li></ul>                                                                                                                                                                                                                                                |
    | Azure China 21Vianet     | <ul><li>mooncake.warmpath.chinacloudapi.cn</li><li>global.prod.microsoftmetrics.com (**nuevo**)</li><li>global.metrics.nsatc.net (**próximamente en desuso**)</li><li>shoebox2.prod.microsoftmetrics.com (**nuevo**)</li><li>shoebox2.metrics.nsatc.net (**próximamente en desuso**)</li><li>shoebox2-red.prod.microsoftmetrics.com</li><li>shoebox2-black.prod.microsoftmetrics.com</li><li>shoebox2-red.shoebox2.metrics.nsatc.net</li><li>shoebox2-black.shoebox2.metrics.nsatc.net</li><li>prod3.prod.microsoftmetrics.com (**nuevo**)</li><li>prod3.metrics.nsatc.net (**próximamente en desuso**)</li><li>prod3-black.prod.microsoftmetrics.com</li><li>prod3-red.prod.microsoftmetrics.com</li><li>prod5.prod.microsoftmetrics.com</li><li>prod5-black.prod.microsoftmetrics.com</li><li>prod5-red.prod.microsoftmetrics.com</li><li>gcs.prod.warm.ingestion.monitoring.azure.cn</li></ul>                                                                                                                                                                                                                                                |

  >[!IMPORTANT]
  > El cambio de clústeres anterior con la zona DNS **.nsatc.net** a **.microsoftmetrics.com** es principalmente un cambio de DNS. La dirección IP del clúster no cambiará.

+ **Etiquetas de servicio regional**: Las reglas de NSG que permiten la conectividad saliente a las etiquetas de servicio de Storage, SQL y Event Hubs pueden usar las versiones regionales correspondientes de las etiquetas de la región que contiene la instancia de API Management (por ejemplo, Storage.WestUS para una instancia de API Management en la región Oeste de EE. UU.). En las implementaciones para varias regiones, el NSG de cada región debe permitir el tráfico a las etiquetas de servicio de esa región y de la región primaria.

+ **Retransmisión de SMTP**: conectividad de red de salida para la retransmisión de SMTP, que se resuelve en el host `smtpi-co1.msn.com`, `smtpi-ch1.msn.com`, `smtpi-db3.msn.com`, `smtpi-sin.msn.com` y `ies.global.microsoft.com`.

+ **CAPTCHA del portal para desarrolladores**: Conectividad de red saliente para el CAPTCHA del portal para desarrolladores, que se resuelve en los host `client.hip.live.com` y `partner.hip.live.com`.

+ **Diagnósticos de Azure Portal**: para permitir el flujo de registros de diagnóstico desde Azure Portal al usar la extensión API Management desde dentro de una red Virtual, se requiere el acceso saliente a `dc.services.visualstudio.com` en el puerto 443. Esto ayuda a solucionar los problemas que pueden surgir al usar la extensión.

+ **Azure Load Balancer**: Permitir la solicitud de entrada desde la etiqueta de servicio `AZURE_LOAD_BALANCER` no es un requisito para la SKU `Developer`, ya que solo implementamos una unidad de proceso detrás de ella. Pero la entrada desde [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md) se convierte en crítica al escalar a una SKU superior como `Premium`, ya que un error en el sondeo de estado de Load Balancer produce un error en una implementación.

+ **Application Insights**: Si la supervisión de [Azure Application Insights](api-management-howto-app-insights.md) está habilitada en API Management, tendremos que permitir la conectividad de salida hacia el [punto de conexión de telemetría](/azure/azure-monitor/app/ip-addresses#outgoing-ports) desde la red virtual. 

+ **Forzar la tunelización del tráfico al firewall local mediante la aplicación virtual de red o de Express Route**: Una configuración común de los clientes es definir su propia ruta predeterminada (0.0.0.0/0) que fuerza a todo el tráfico de la subred delegada de API Management a pasar a través de un firewall local o a una aplicación virtual de red. El flujo de tráfico interrumpe invariablemente la conectividad con Azure API Management porque el tráfico saliente está bloqueado de forma local o porque se usa NAT para convertirlo en un conjunto de direcciones irreconocibles que no funcionan con varios puntos de conexión de Azure. La solución requiere que se hagan un par de cosas:

  * Habilite los puntos de conexión de servicio en la subred en la que se ha implementado el servicio API Management. [Los puntos de conexión de servicio][ServiceEndpoints] deben habilitarse para Azure SQL, Azure Storage, Azure Event Hubs y Azure ServiceBus. La habilitación de los puntos de conexión directamente desde la subred delegada de API Management a estos servicios les permite utilizar la red troncal de Microsoft Azure, que proporciona un enrutamiento óptimo para el tráfico de servicios. Si usa puntos de conexión de servicio con una API Management con túnel forzado, el tráfico de servicios de Azure anterior no se enruta a través de tunelización forzada. El resto del tráfico de dependencia del servicio API Management se enruta con tunelización forzada y no se puede perder o este servicio no funcionaría correctamente.
    
  * Todo el tráfico del plano de control desde Internet al punto de conexión de administración del servicio API Management se enruta a través de un conjunto específico de IP de entrada hospedadas en API Management. Cuando el tráfico se produce con tunelización forzada, no se asignarán simétricamente las respuestas a estas direcciones IP de origen de entrada. Para superar la limitación, necesitamos agregar las siguientes rutas definidas por el usuario ([UDR][UDRs]) para dirigir el tráfico de vuelta a Azure mediante el establecimiento del destino de estas rutas de host a "Internet". El conjunto de direcciones IP de entrada para el tráfico del plano de control está documentado en [Direcciones IP del plano de control](#control-plane-ips).

  * Para otras dependencias de servicios API Management con tunelización forzada, debería haber una forma de resolver el nombre de host y llegar hasta el punto de conexión. Entre ellas se incluyen las siguientes:
      - Supervisión de métricas y estado
      - Diagnósticos de Azure Portal
      - Retransmisión de SMTP
      - CAPTCHA del portal para desarrolladores

## <a name="troubleshooting"></a><a name="troubleshooting"> </a>Solución de problemas
* **Programa de instalación inicial**: cuando la implementación inicial del servicio API Management en una subred no se realiza correctamente, se recomienda implementar una máquina virtual en la misma subred. Siga con el escritorio remoto en la máquina virtual y compruebe que hay conectividad cada uno de los siguientes recursos de la suscripción de Azure.
    * Azure Storage Blob
    * Azure SQL Database
    * Tabla de Azure Storage

  > [!IMPORTANT]
  > Una vez que valide la conectividad, asegúrese de quitar todos los recursos implementados en la subred, antes de implementar API Management en ella.

* **Actualizaciones incrementales**: Al realizar cambios en la red, consulte [NetworkStatus API](/rest/api/apimanagement/2019-12-01/networkstatus) para validar si el servicio API Management no ha perdido el acceso a cualquiera de los recursos críticos de los que depende. El estado de conectividad debe actualizarse cada 15 minutos.

* **Vínculos de navegación de recursos**: cuando se implementan en la subred de red virtual del estilo de Resource Manager, API Management reserva la subred creando un vínculo de navegación de recursos. Si la subred ya contiene un recurso de un proveedor distinto, la implementación **producirá un error**. De forma similar, al eliminar un servicio API Management o moverlo a una subred diferente, se quitará ese vínculo de navegación de recursos.

## <a name="subnet-size-requirement"></a><a name="subnet-size"> </a> Requisitos de tamaño de subred
Azure reserva algunas direcciones IP dentro de cada subred y estas direcciones no se pueden usar. La primera y la última dirección IP de las subredes están reservadas para la conformidad con el protocolo, junto con otras tres direcciones usadas para los servicios de Azure. Para más información, consulte [¿Hay alguna restricción en el uso de direcciones IP dentro de estas subredes?](../virtual-network/virtual-networks-faq.md#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets)

Además de las direcciones IP que usa la infraestructura de Azure VNET, cada instancia de API Management de la subred usa dos direcciones IP por unidad de SKU Premium y una dirección IP adicional para la SKU de desarrollador. Cada instancia reserva una dirección IP adicional para el equilibrador de carga externo. Cuando se implementa en la red virtual interna, requiere una dirección IP adicional para el equilibrador de carga interno.

Dado el cálculo anterior, el tamaño mínimo de la subred en la que se puede implementar API Management es /29, lo que proporciona tres direcciones IP utilizables.

Cada unidad de escalado adicional de API Management requiere dos direcciones IP más.

## <a name="routing"></a><a name="routing"> </a> Enrutamiento
+ También se reservará una dirección IP pública (VIP) con equilibrio de carga para proporcionar acceso a todos los puntos de conexión del servicio.
+ Se usará una dirección IP de un intervalo de IP de subred (DIP) para el acceso a los recursos dentro de la red virtual y una dirección IP pública (VIP) para el acceso a los recursos fuera de la red virtual.
+ La dirección IP pública con equilibrio de carga puede encontrarse en la hoja Información general/nformación esencial en Azure Portal.

## <a name="limitations"></a><a name="limitations"> </a>Limitaciones
* Una subred que contenga instancias de API Management no puede contener otros tipos de recursos de Azure.
* La subred y el servicio API Management tienen que estar en la misma suscripción.
* Una subred que contenga instancias de API Management no se puede mover a otras suscripciones.
* Para implementaciones de API Management de varias regiones configuradas en el modo de red virtual interna, los usuarios son responsables de administrar el equilibrio de carga a través de varias regiones, ya que son los propietarios del enrutamiento.
* La conectividad de un recurso en una VNET emparejada globalmente a otra región con el servicio API Management en modo interno no funciona debido a la limitación de la plataforma. Para obtener más información, consulte el apartado [Los recursos en una red virtual no pueden comunicarse con la dirección IP de un equilibrador de carga interno de Azure en la red virtual emparejada](../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints).

## <a name="control-plane-ip-addresses"></a><a name="control-plane-ips"> </a> Direcciones IP del plano de control

Las direcciones IP se dividen según el **entorno de Azure**. Cuando se permiten las solicitudes entrantes, la dirección IP marcada con **Global** debe estar en la lista blanca junto con la dirección IP específica de la **región**.

| **Entorno de Azure**|   **Región**|  **Dirección IP**|
|-----------------|-------------------------|---------------|
| Azure Public| Centro-sur de EE. UU. (global)| 104.214.19.224|
| Azure Public| Centro-norte de EE. UU. (global)| 52.162.110.80|
| Azure Public| Centro-Oeste de EE. UU.| 52.253.135.58|
| Azure Public| Centro de Corea del Sur| 40.82.157.167|
| Azure Public| Oeste de Reino Unido| 51.137.136.0|
| Azure Public| Japón Occidental| 40.81.185.8|
| Azure Public| Centro-Norte de EE. UU| 40.81.47.216|
| Azure Public| Sur de Reino Unido| 51.145.56.125|
| Azure Public| Oeste de la India| 40.81.89.24|
| Azure Public| Este de EE. UU.| 52.224.186.99|
| Azure Public| Oeste de Europa| 51.145.179.78|
| Azure Public| Japón Oriental| 52.140.238.179|
| Azure Public| Centro de Francia| 40.66.60.111|
| Azure Public| Este de Canadá| 52.139.80.117|
| Azure Public| Norte de Emiratos Árabes Unidos| 20.46.144.85|
| Azure Public| Sur de Brasil| 191.233.24.179|
| Azure Public| Sudeste de Asia| 40.90.185.46|
| Azure Public| Norte de Sudáfrica| 102.133.130.197|
| Azure Public| Centro de Canadá| 52.139.20.34|
| Azure Public| Corea del Sur| 40.80.232.185|
| Azure Public| Centro de la India| 13.71.49.1|
| Azure Public| Oeste de EE. UU.| 13.64.39.16|
| Azure Public| Sudeste de Australia| 20.40.160.107|
| Azure Public| Centro de Australia| 20.37.52.67|
| Azure Public| Sur de la India| 20.44.33.246|
| Azure Public| Centro de EE. UU.| 13.86.102.66|
| Azure Public| Este de Australia| 20.40.125.155|
| Azure Public| Oeste de EE. UU. 2| 51.143.127.203|
| Azure Public| EUAP de Este de EE. UU. 2| 52.253.229.253|
| Azure Public| EUAP del centro de EE. UU.| 52.253.159.160|
| Azure Public| Centro-sur de EE. UU.| 20.188.77.119|
| Azure Public| Este de EE. UU. 2| 20.44.72.3|
| Azure Public| Norte de Europa| 52.142.95.35|
| Azure Public| Este de Asia| 52.139.152.27|
| Azure Public| Sur de Francia| 20.39.80.2|
| Azure Public| Oeste de Suiza| 51.107.96.8|
| Azure Public| Centro de Australia 2| 20.39.99.81|
| Azure Public| Centro de Emiratos Árabes Unidos| 20.37.81.41|
| Azure Public| Norte de Suiza| 51.107.0.91|
| Azure Public| Oeste de Sudáfrica| 102.133.0.79|
| Azure Public| Centro-oeste de Alemania| 51.116.96.0|
| Azure Public| Norte de Alemania| 51.116.0.0|
| Azure Public| Este de Noruega| 51.120.2.185|
| Azure Public| Oeste de Noruega| 51.120.130.134|
| Azure China 21Vianet| Norte de China (Global)| 139.217.51.16|
| Azure China 21Vianet| Este de China (Global)| 139.217.171.176|
| Azure China 21Vianet| Norte de China| 40.125.137.220|
| Azure China 21Vianet| Este de China| 40.126.120.30|
| Azure China 21Vianet| Norte de China 2| 40.73.41.178|
| Azure China 21Vianet| Este de China 2| 40.73.104.4|
| Azure Government| USGov Virginia (Global)| 52.127.42.160|
| Azure Government| USGov Texas (Global)| 52.127.34.192|
| Azure Government| USGov Virginia| 52.227.222.92|
| Azure Government| USGov Iowa| 13.73.72.21|
| Azure Government| USGov: Arizona| 52.244.32.39|
| Azure Government| USGov: Texas| 52.243.154.118|
| Azure Government| Departamento de Defensa de centro de EE. UU.| 52.182.32.132|
| Azure Government| Departamento de Defensa del este de EE. UU| 52.181.32.192|

## <a name="related-content"></a><a name="related-content"> </a>Contenido relacionado
* [Conexión de una red virtual a back-end mediante VPN Gateway](../vpn-gateway/design.md#s2smulti)
* [Conexión a una red virtual a partir de diferentes modelos de implementación](../vpn-gateway/vpn-gateway-connect-different-deployment-models-powershell.md)
* [Uso de API Inspector para hacer un seguimiento de las llamadas en Azure API Management](api-management-howto-api-inspector.md)
* [Preguntas más frecuentes acerca de Virtual Network](../virtual-network/virtual-networks-faq.md)
* [Etiquetas de servicio](../virtual-network/security-overview.md#service-tags)

[api-management-using-vnet-menu]: ./media/api-management-using-with-vnet/api-management-menu-vnet.png
[api-management-setup-vpn-select]: ./media/api-management-using-with-vnet/api-management-using-vnet-select.png
[api-management-setup-vpn-add-api]: ./media/api-management-using-with-vnet/api-management-using-vnet-add-api.png
[api-management-vnet-private]: ./media/api-management-using-with-vnet/api-management-vnet-internal.png
[api-management-vnet-public]: ./media/api-management-using-with-vnet/api-management-vnet-external.png

[Enable VPN connections]: #enable-vpn
[Connect to a web service behind VPN]: #connect-vpn
[Related content]: #related-content

[UDRs]: ../virtual-network/virtual-networks-udr-overview.md
[Network Security Group]: ../virtual-network/security-overview.md
[ServiceEndpoints]: ../virtual-network/virtual-network-service-endpoints-overview.md
[ServiceTags]: ../virtual-network/security-overview.md#service-tags
