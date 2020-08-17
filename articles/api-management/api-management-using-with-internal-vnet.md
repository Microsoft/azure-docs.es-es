---
title: Uso de Azure API Management con redes virtuales internas
titleSuffix: Azure API Management
description: Aprenda a instalar y configurar Azure API Management en una red virtual interna.
services: api-management
documentationcenter: ''
author: vladvino
manager: kjoshi
editor: ''
ms.assetid: dac28ccf-2550-45a5-89cf-192d87369bc3
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/31/2019
ms.author: apimpm
ms.openlocfilehash: ac9554ae5ca151a377395d3b16598f7070c87d10
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87386051"
---
# <a name="using-azure-api-management-service-with-an-internal-virtual-network"></a>Uso del servicio Azure API Management con una red virtual interna
Con Azure Virtual Network, Azure API Management puede administrar las API que no están accesibles desde Internet. Para establecer la conexión, hay una serie de tecnologías de VPN disponibles. API Management puede implementarse de dos modos en una red virtual:
* Externo
* Interno

Cuando API Management se implementa en el modo de red virtual interna, todos los puntos de conexión de servicio (la puerta de enlace de proxy, el portal para desarrolladores, la administración directa y Git) solamente están visibles en una red virtual en la que usted controla el acceso. Ninguno de los puntos de conexión de servicio está registrado en el servidor DNS público.

> [!NOTE]
> Dado que no hay ninguna entrada DNS para los puntos de conexión de servicio, estos puntos de conexión no serán accesibles hasta que [se configure DNS](#apim-dns-configuration) para la red virtual.

Si utiliza API Management en modo interno, puede conseguir los siguientes escenarios:

* Puede conseguir que terceras personas puedan obtener acceso de forma segura a las API hospedadas en el centro de datos privado desde fuera de este centro utilizando conexiones de sitio a sitio o conexiones de VPN de Azure ExpressRoute.
* Puede permitir escenarios de nube híbrida exponiendo las API basadas en la nube y las API locales a través de una puerta de enlace común.
* Puede administrar las API hospedadas en varias ubicaciones geográficas mediante un único punto de conexión de puerta de enlace.

[!INCLUDE [premium-dev.md](../../includes/api-management-availability-premium-dev.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para seguir los pasos que se describen en este artículo, debe tener:

+ **Una suscripción activa a Azure**.

    [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

+ **Una instancia de Azure API Management**. Para más información, vea [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).
+ Cuando se implementa un servicio API Management en una red virtual, se usa una [lista de puertos](./api-management-using-with-vnet.md#required-ports) que deben abrirse. 

## <a name="creating-an-api-management-in-an-internal-virtual-network"></a><a name="enable-vpn"> </a>Creación de una instancia de API Management en una red virtual interna
En las redes virtuales internas, el servicio API Management se hospeda detrás de un [equilibrador de carga interno (clásico)](/azure/load-balancer/load-balancer-get-started-ilb-classic-cloud). Esta es la única opción disponible y no se puede cambiar.

### <a name="enable-a-virtual-network-connection-using-the-azure-portal"></a>Habilitación de una conexión de red virtual mediante Azure Portal

1. Navegue a la instancia de Azure API Management en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Red virtual**.
3. Configure la instancia de API Management que se va a implementar dentro de la red virtual.

    ![Menú de configuración de una instancia de Azure API Management en una red virtual interna][api-management-using-internal-vnet-menu]

4. Seleccione **Guardar**.

Después de que la implementación se realiza correctamente, verá la dirección IP virtual **privada** y la dirección IP virtual **pública** del servicio API Management en la hoja de información general. La dirección IP virtual **privada** es una dirección IP con equilibrio de carga dentro de la subred delegada de API Management a través de la que se puede acceder a los puntos de conexión `gateway`, `portal`, `management` y `scm`. La dirección IP virtual **pública** se usa **solo** con el tráfico del plano de control al punto de conexión `management` a través del puerto 3443 y puede bloquearse hasta la etiqueta de servicio [ApiManagement][ServiceTags].

![Panel de API Management con una red virtual interna configurada][api-management-internal-vnet-dashboard]

> [!NOTE]
> La consola de prueba disponible en Azure Portal no funcionará con un servicio implementado en una red virtual **interna**, ya que la dirección Url de puerta de enlace no está registrada en el DNS público. En su lugar, debe usar la consola de prueba que se ofrece en el **Portal para desarrolladores**.

### <a name="enable-a-virtual-network-connection-by-using-powershell-cmdlets"></a>Habilitación de una conexión de red virtual mediante cmdlets de PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

También puede habilitar la conectividad de la red virtual utilizando cmdlets de PowerShell.

* Cree un servicio API Management dentro de una red virtual: Use el cmdlet [New AzApiManagement](/powershell/module/az.apimanagement/new-azapimanagement) para crear un servicio Azure API Management dentro de una red virtual y configurarlo para utilizar el tipo de red virtual interna.

* Actualice una implementación existente de un servicio API Management dentro de una red virtual: Use el cmdlet [Update-AzApiManagementRegion](/powershell/module/az.apimanagement/update-azapimanagementregion) para mover un servicio API Management existente en una red virtual y configurarlo para utilizar el tipo de red virtual interna.

## <a name="dns-configuration"></a><a name="apim-dns-configuration"></a>Configuración de DNS
Cuando API Management está en modo de red virtual externa, el DNS está administrado por Azure. En el modo de red virtual interna, es usted quien tiene que administrar su propio DNS.

> [!NOTE]
> El servicio API Management no escucha las solicitudes procedentes de direcciones IP. Solo responde a las solicitudes dirigidas al nombre de host establecido en los puntos de conexión de servicio. Estos puntos de conexión pueden ser la puerta de enlace, Azure Portal y el portal del desarrollador, el punto de conexión de administración directa y GIT.

### <a name="access-on-default-host-names"></a>Acceso de nombres de host predeterminados
Cuando se crea un servicio API Management, llamado por ejemplo "contosointernalvnet", se configuran de forma predeterminada los siguientes puntos de conexión de servicio:

   * Puerta de enlace o proxy: contosointernalvnet.azure-api.net

   * El portal para desarrolladores: contosointernalvnet.portal.azure-api.net

   * El nuevo portal para desarrolladores: contosointernalvnet.developer.azure-api.net

   * Punto de conexión de administración directa: contosointernalvnet.management.azure-api.net

   * Git: contosointernalvnet.scm.azure-api.net

Para obtener acceso a estos puntos de conexión de servicio de API Management, puede crear una máquina virtual en una subred conectada a la red virtual en la que está implementado API Management. Suponiendo que la dirección IP virtual interna del servicio sea 10.1.0.5, puede asignar el archivo del host, %SystemDrive%\drivers\etc\hosts, del modo siguiente:

   * 10.1.0.5 contosointernalvnet.azure-api.net

   * 10.1.0.5 contosointernalvnet.portal.azure-api.net

   * 10.1.0.5     contosointernalvnet.developer.azure-api.net

   * 10.1.0.5 contosointernalvnet.management.azure-api.net

   * 10.1.0.5 contosointernalvnet.scm.azure-api.net

Así, podrá tener acceso a todos los puntos de conexión de servicio desde la máquina virtual que ha creado.
Si utiliza un servidor DNS personalizado en una red virtual, también puede crear registros D de DNS y acceder a estos puntos de conexión desde cualquier lugar de la red virtual.

### <a name="access-on-custom-domain-names"></a>Acceso de nombres de dominio personalizados

1. Si no quiere acceder al servicio API Management con los nombres de host predeterminados, puede definir nombres de dominio personalizados para todos los puntos de conexión de servicio, tal y como se indica en la siguiente ilustración:

   ![Configuración de un dominio personalizado para API Management][api-management-custom-domain-name]

2. A continuación, puede crear registros en el servidor DNS para acceder a los puntos de conexión que solo están accesibles desde dentro de la red virtual.

## <a name="routing"></a><a name="routing"> </a> Enrutamiento

* Se reservará una dirección IP virtual *privada* con equilibrio de carga desde el intervalo de subred y se usará para acceder a los puntos de conexión de servicio de API Management desde la red virtual. Esta dirección IP *privada* puede encontrarse en la hoja de información general del servicio en Azure Portal. Esta dirección debe registrarse en los servidores DNS usados por la red virtual.
* También se reservará una dirección IP *pública* (VIP) con equilibrio de carga para proporcionar acceso al punto de conexión de servicio de administración a través del puerto 3443. Esta dirección IP *pública* puede encontrarse en la hoja de información general del servicio en Azure Portal. La dirección IP *pública* se usa solo con el tráfico del plano de control al punto de conexión `management` a través del puerto 3443 y puede bloquearse hasta la etiqueta de servicio [ApiManagement][ServiceTags].
* Las direcciones IP del intervalo IP de subred (DIP) se asignarán a cada máquina virtual del servicio y se usarán para acceder a los recursos dentro de la red virtual. Una dirección IP pública (VIP) se usará para acceder a los recursos fuera de la red virtual. Si se usan listas de restricciones de IP para proteger los recursos dentro de la red virtual, se debe especificar el intervalo entero de la subred donde se implementa el servicio API Management para conceder o restringir el acceso desde el servicio.
* Las direcciones IP privadas y públicas con equilibrio de carga pueden encontrarse en la hoja de información general de Azure Portal.
* Si el servicio se quitó de la red virtual y luego se volvió a agregar a ella, pueden cambiar las direcciones IP asignadas para el acceso público y privado. Si esto sucede, puede ser necesario actualizar los registros de DNS, las reglas de enrutamiento y las listas de restricciones de IP dentro de la red virtual.

## <a name="related-content"></a><a name="related-content"> </a>Contenido relacionado
Para más información, vea los siguientes artículos:
* [Problemas comunes de configuración de red al establecer Azure API Management en una red virtual][Common network configuration problems]
* [Preguntas más frecuentes (P+F) acerca de Azure Virtual Network](../virtual-network/virtual-networks-faq.md)
* [Creating a record in DNS](/previous-versions/windows/it-pro/windows-2000-server/bb727018(v=technet.10)) (Creación de un registro en DNS)

[api-management-using-internal-vnet-menu]: ./media/api-management-using-with-internal-vnet/api-management-using-with-internal-vnet.png
[api-management-internal-vnet-dashboard]: ./media/api-management-using-with-internal-vnet/api-management-internal-vnet-dashboard.png
[api-management-custom-domain-name]: ./media/api-management-using-with-internal-vnet/api-management-custom-domain-name.png

[Create API Management service]: get-started-create-service-instance.md
[Common network configuration problems]: api-management-using-with-vnet.md#network-configuration-issues

[ServiceTags]: ../virtual-network/security-overview.md#service-tags
