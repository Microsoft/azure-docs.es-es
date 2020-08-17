---
title: ¿Qué es el servicio Azure Private Link?
description: Obtenga información sobre el servicio Azure Private Link.
services: private-link
author: sumeetmittal
ms.service: private-link
ms.topic: conceptual
ms.date: 09/16/2019
ms.author: sumi
ms.openlocfilehash: a6bbb2abe24eba96fd2c55b7aaf15ccd8ae33530
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760961"
---
# <a name="what-is-azure-private-link-service"></a>¿Qué es el servicio Azure Private Link?

El servicio Azure Private Link es la referencia a su propio servicio que usa la tecnología de Azure Private Link. El servicio que se ejecuta de forma subyacente a [Azure Standard Load Balancer](../load-balancer/load-balancer-standard-overview.md) se puede habilitar para el acceso a Private Link de modo que los consumidores del servicio puedan tener acceso a este de forma privada desde sus propias redes virtuales. Sus clientes pueden crear un punto de conexión privado dentro de su red virtual y asignarlo a este servicio. En este artículo se explican conceptos relacionados con el lado del proveedor de servicios. 

:::image type="content" source="./media/private-link-service-overview/consumer-provider-endpoint.png" alt-text="Flujo de trabajo del servicio Private Link" border="true":::

*Ilustración: Servicio Azure Private Link.*

## <a name="workflow"></a>Flujo de trabajo

![Flujo de trabajo del servicio Private Link](media/private-link-service-overview/private-link-service-workflow.png)


*Ilustración: Flujo de trabajo del servicio Azure Private Link.*

### <a name="create-your-private-link-service"></a>Creación de su servicio Private Link

- Configure su aplicación para ejecutarla de forma subyacente a una instancia de Standard Load Balancer en su red virtual. Si ya tiene su aplicación configurada de forma subyacente a una instancia de Standard Load Balancer, puede omitir este paso.   
- Cree un servicio Private Link que haga referencia al equilibrador de carga anterior. En el proceso de selección del equilibrador de carga, elija la configuración de IP de front-end donde desea recibir el tráfico. Elija una subred para las direcciones IP de NAT del servicio Private Link. Se recomienda tener al menos ocho direcciones IP de NAT disponibles en la subred. Todo el tráfico de consumidor parecerá originarse en este grupo de direcciones IP privadas del proveedor de servicios. Elija las propiedades o la configuración adecuadas para el servicio Private Link.    

    > [!NOTE]
    > El servicio Azure Private Link solo se admite en Standard Load Balancer. 
    
### <a name="share-your-service"></a>Uso compartido del servicio

Una vez creado un servicio Private Link, Azure generará un moniker con nombre único global llamado "alias" basado en el nombre que proporcione para el servicio. Puede compartir el alias o el URI del recurso del servicio con sus clientes sin conexión. Los consumidores pueden iniciar una conexión de Private Link mediante el alias o el URI del recurso.
 
### <a name="manage-your-connection-requests"></a>Administración de sus solicitudes de conexión

Después de que un consumidor inicie una conexión, el proveedor de servicios puede aceptar o rechazar la solicitud de conexión. Todas las solicitudes de conexión se mostrarán en la propiedad **privateendpointconnections** del servicio Private Link.
 
### <a name="delete-your-service"></a>Eliminación del servicio

Si ya no se usa el servicio Private Link, puede eliminarlo. Sin embargo, antes de hacerlo, asegúrese de que no hay ninguna conexión de punto de conexión privado asociada a este. Puede rechazar todas las conexiones y eliminar el servicio.

## <a name="properties"></a>Propiedades

Un servicio Private Link especifica las siguientes propiedades: 

|Propiedad |Explicación  |
|---------|---------|
|Estado de aprovisionamiento (provisioningState)  |Una propiedad de solo lectura que muestra el estado de aprovisionamiento actual del servicio Private Link. Los estados de aprovisionamiento aplicables son: "Eliminando; Error; Correcto; Actualizando". Si el estado de aprovisionamiento es "Correcto", ha aprovisionado correctamente su servicio Private Link.        |
|Alias (alias)     | Alias es una cadena de solo lectura única global para su servicio. Le ayuda a enmascarar los datos del cliente para su servicio y, al mismo tiempo, crea un nombre fácil de compartir para este. Al crear un servicio Private Link, Azure genera el alias para su servicio, el cual puede compartir con sus clientes. Sus clientes pueden usar este alias para solicitar una conexión a su servicio.          |
|Visibilidad (visibility)     | Visibilidad es la propiedad que controla la configuración de exposición para su servicio Private Link. Los proveedores de servicios pueden elegir limitar la exposición a su servicio a suscripciones con permisos de control de acceso basado en rol (RBAC), un conjunto restringido de suscripciones, o todas las suscripciones de Azure.          |
|Aprobación automática (autoApproval)    |   Aprobación automática controla el acceso automatizado al servicio Private Link. Las suscripciones especificadas en la lista de aprobación automática se aprueban automáticamente al solicitarse una conexión desde puntos de conexión privados de esas suscripciones.          |
|Configuración de IP de front-end de Load Balancer (loadBalancerFrontendIpConfigurations)    |    El servicio Private Link está asociado a la dirección IP de front-end de una instancia de Standard Load Balancer. Todo el tráfico destinado al servicio alcanzará el front-end de SLB. Puede configurar las reglas de SLB para dirigir este tráfico a grupos de back-end adecuados donde se ejecutan sus aplicaciones. Las configuraciones de IP de front-end de Load Balancer son distintas de las configuraciones de IP de NAT.      |
|Configuración de IP de NAT (ipConfigurations)    |    Esta propiedad hace referencia a la configuración de IP de NAT (traducción de direcciones de red) para el servicio Private Link. La IP de NAT se puede elegir desde cualquier subred de la red virtual de un proveedor de servicios. El servicio Private Link realiza traducciones de direcciones de red del lado del destino en el tráfico de Private Link. Esto garantiza que no haya ningún conflicto de IP entre el espacio de direcciones de origen (lado del consumidor) y destino (proveedor de servicios). En el lado de destino (lado del proveedor de servicios), la dirección IP de NAT se mostrará como IP de origen para todos los paquetes recibidos por su servicio e IP de destino para todos los paquetes enviados por su servicio.       |
|Conexiones de punto de conexión privado (privateEndpointConnections)     |  Esta propiedad muestra los puntos de conexión privados que se conectan al servicio Private Link. Varios puntos de conexión privados pueden conectarse al mismo servicio Private Link y el proveedor de servicios puede controlar el estado para los puntos de conexión privados individuales.        |
|Proxy TCP V2 (EnableProxyProtocol)     |  Esta propiedad permite al proveedor de servicios utilizar el proxy TCP v2 para recuperar información de conexión sobre el consumidor del servicio. El proveedor de servicios es responsable de preparar las configuraciones del receptor para que este pueda analizar el encabezado del protocolo de proxy v2.        |
|||


### <a name="details"></a>Detalles

- Se puede acceder al servicio Private Link desde puntos de conexión privados aprobados en cualquier región pública. Se puede alcanzar el punto de conexión privado desde la misma red virtual, redes virtuales emparejadas a nivel regional, global y local mediante conexiones VPN o ExpressRoute privadas. 
 
- Al crear un servicio Private Link, se crea una interfaz de red para el ciclo de vida del recurso. El cliente no puede administrar esta interfaz.
 
- El servicio Private Link debe implementarse en la misma región que la red virtual y Standard Load Balancer.  
 
- Se puede tener acceso a un solo servicio Private Link desde varios puntos de conexión privados pertenecientes a diferentes redes virtuales, suscripciones o inquilinos de Active Directory. La conexión se establece a través de un flujo de trabajo de conexión. 
 
- Pueden crearse varios servicios Private Link en la misma instancia de Standard Load Balancer mediante diferentes configuraciones de IP de front-end. Existen límites en cuanto al número de servicios Private Link que puede crear por Standard Load Balancer y por suscripción. Para más información, consulte el artículo acerca de los  [límites de Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#networking-limits).
 
- El servicio Private Link puede tener más de una configuración de IP de NAT vinculada a él. Elegir más de una configuración de IP de NAT puede ayudar a los proveedores de servicios a escalar. Actualmente, los proveedores de servicios pueden asignar hasta ocho direcciones de IP de NAT por servicio Private Link. Con cada dirección IP de NAT, puede asignar más puertos para sus conexiones TCP y, por tanto, escalar horizontalmente. Tras agregar varias direcciones IP de NAT a un servicio Private Link, no puede eliminar las direcciones IP de NAT. Esto se realiza para garantizar que las conexiones activas no se vean afectadas durante la eliminación de las direcciones IP de NAT.


## <a name="alias"></a>Alias

**Alias** es un nombre único global para el servicio. Le ayuda a enmascarar los datos del cliente para su servicio y, al mismo tiempo, crea un nombre fácil de compartir para este. Al crear un servicio Private Link, Azure genera un alias para su servicio, el cual puede compartir con sus clientes. Sus clientes pueden usar este alias para solicitar una conexión a su servicio.

El alias consta de tres partes: *Prefix*.*GUID*.*Suffix*

- Prefix es el nombre del servicio. Puede elegir su propio prefijo. Tras crear "Alias", no puede cambiarlo, de modo que seleccione su prefijo adecuadamente.  
- La plataforma proporcionará GUID. Esto ayuda a hacer que el nombre sea único a nivel global. 
- Azure anexa Suffix: *region*.azure.privatelinkservice 

Alias completo:  *Prefix*. {GUID}.*region*.azure.privatelinkservice  

## <a name="control-service-exposure"></a>Exposición del servicio de control

El servicio Private Link le proporciona opciones para controlar la exposición de su servicio a través de la configuración "Visibilidad". Puede hacer que el servicio sea privado para su consumo desde las diversas redes virtuales que posee (solo los permisos de RBAC), restringir la exposición a un conjunto limitado de suscripciones de su confianza o hacer que sea público para que todas las suscripciones de Azure puedan solicitar conexiones en el servicio Private Link. Su configuración de visibilidad decide si un consumidor puede conectarse a su servicio o no. 

## <a name="control-service-access"></a>Acceso al servicio de control

Los consumidores con exposición (controlada por la configuración de visibilidad) a su servicio Private Link pueden crear un punto de conexión privado en sus redes virtuales y solicitar una conexión a su servicio Private Link. La conexión de punto de conexión privado se crearán en un estado "Pendiente" del objeto de servicio Private Link. El proveedor de servicios es responsable de actuar en la solicitud de conexión. Puede aprobar la conexión, rechazarla o eliminarla. Solo las conexiones que se aprueban pueden enviar tráfico al servicio Private Link.

La acción de aprobar las conexiones se puede automatizar mediante la propiedad de aprobación automática del servicio Private Link. La aprobación automática es la capacidad de los proveedores de servicios de aprobar de forma previa un conjunto de suscripciones para el acceso automatizado a su servicio. Los clientes tendrán que compartir sus suscripciones sin conexión para que los proveedores de servicios se agreguen a la lista de aprobación automática. La aprobación automática es un subconjunto de la matriz de visibilidad. La visibilidad controla la configuración de exposición, mientras que la aprobación automática controla la configuración de aprobación para su servicio. Si un cliente solicita una conexión desde una suscripción en la lista de aprobación automática, la conexión se aprobará automáticamente y se establecerá. No es necesario que los proveedores de servicios sigan aprobando manualmente la solicitud. Por otro lado, si un cliente solicita una conexión desde una suscripción en la matriz de visibilidad y no en la de aprobación automática, el proveedor de servicios recibirá la solicitud, pero este debe aprobar las conexiones.

## <a name="getting-connection-information-using-tcp-proxy-v2"></a>Cómo obtener información de conexión mediante el proxy TCP V2

Al usar el servicio de hipervínculo privado, la dirección IP de origen de los paquetes procedentes de un punto de conexión privado es traducida a dirección de red (NAT) en el lado del proveedor de servicios mediante la dirección IP NAT asignada desde la red virtual del proveedor. Por lo tanto, las aplicaciones reciben la dirección IP NAT asignada en lugar de la dirección IP de origen real de los consumidores del servicio. Si la aplicación necesita la dirección IP de origen real del lado del consumidor, se puede habilitar el protocolo de proxy en el servicio y recuperar la información a partir del encabezado del protocolo de proxy. Además de la dirección IP de origen, el encabezado del protocolo de proxy también incluye el LinkID del punto de conexión privado. Combinar la dirección IP de origen con el LinkID puede ayudar a los proveedores de servicios a identificar de forma única a sus consumidores. Para más información sobre el protocolo de proxy, [consulte este artículo](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt). 

Esta información se codifica mediante un vector tipo-longitud-valor (TLV) personalizado como se indica a continuación:

Detalles del TLV personalizado:

|Campo |Longitud (octetos)  |Descripción  |
|---------|---------|----------|
|Tipo  |1        |PP2_TYPE_AZURE (0xEE)|
|Length  |2      |Longitud del valor|
|Value  |1     |PP2_SUBTYPE_AZURE_PRIVATEENDPOINT_LINKID (0x01)|
|  |4        |UINT32 (4 bytes) que representan el LINKID del punto de conexión privado. Codificado en formato little endian.|

 > [!NOTE]
 > El proveedor de servicios es responsable de asegurarse de que el servicio que está detrás del equilibrador de carga estándar está configurado para analizar el encabezado del protocolo de proxy según la [especificación](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt) cuando el protocolo de proxy está habilitado en el servicio de vínculo privado. Se producirá un error en la solicitud si la configuración del protocolo de proxy está habilitada en el servicio de vínculo privado pero el servicio del proveedor de servicios no está configurado para analizar el encabezado. Del mismo modo, se producirá un error en la solicitud si el servicio del proveedor de servicios espera un encabezado de protocolo de proxy mientras la configuración no está habilitada en el servicio de vínculo privado. Una vez habilitada la configuración del protocolo de proxy, el encabezado del protocolo de proxy también se incluirá en los sondeos de estado HTTP/TCP desde el host hasta las máquinas virtuales de back-end, aunque no haya información del cliente en el encabezado. 

## <a name="limitations"></a>Limitaciones

A continuación se muestran las limitaciones conocidas al usar el servicio Private Link:
- Solo se admite en Standard Load Balancer 
- Solo admite el tráfico IPv4
- Solo admite el tráfico TCP

## <a name="next-steps"></a>Pasos siguientes
- [Creación de un servicio Private Link mediante Azure PowerShell](create-private-link-service-powershell.md)
- [Creación de un servicio Private Link mediante la CLI de Azure](create-private-link-service-cli.md)
