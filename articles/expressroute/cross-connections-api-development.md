---
title: Desarrollo e integración de la API CrossConnnections de Azure ExpressRoute
description: En este artículo se proporciona información general detallada para los asociados de ExpressRoute sobre el tipo de recurso expressRouteCrossConnections.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: duau
ms.openlocfilehash: 25c8288e1804e6a08ae2b5b128ab6fbc699563f9
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89397855"
---
# <a name="expressroute-crossconnnections-api-development-and-integration"></a>Desarrollo e integración de la API CrossConnnections de ExpressRoute

La API de Resource Manager de asociado de ExpressRoute permite a los asociados de ExpressRoute administrar la configuración de nivel 2 y nivel 3 de los circuitos ExpressRoute del cliente. La API de Resource Manager del asociado de ExpressRoute presenta un nuevo tipo de recurso, **expressRouteCrossConnections**. Los asociados usan este recurso para administrar los circuitos ExpressRoute de los clientes.

## <a name="workflow"></a>Flujo de trabajo

El recurso expressRouteCrossConnections es un recurso de propiedad reemplazada del circuito ExpressRoute. Cuando un cliente de Azure crea un circuito ExpressRoute y selecciona un asociado de ExpressRoute específico, Microsoft crea un recurso expressRouteCrossConnections en la suscripción de administración de Azure ExpressRoute del asociado. Al hacerlo, Microsoft define un grupo de recursos en el que se va a crear el recurso expressRouteCrossConnections. El estándar de nomenclatura para el grupo de recursos es **CrossConnection-*PeeringLocation*;** donde PeeringLocation = la ubicación de ExpressRoute. Por ejemplo, si un cliente crea un circuito ExpressRoute en Denver, CrossConnection se creará en la suscripción de Azure del asociado en el siguiente grupo de recursos: **CrossConnnection-Denver**.

Los asociados de ExpressRoute administran la configuración de nivel 2 y nivel 3 emitiendo operaciones REST en el recurso expressRouteCrossConnections.

## <a name="benefits"></a>Ventajas

Ventajas de mover al recurso expressRouteCrossConnections:

* Las futuras mejoras de los asociados de ExpressRoute estarán disponibles en el recurso ExpressRouteCrossConnection.

* Los asociados pueden aplicar el [control de acceso basado en rol de Azure (Azure RBAC)](https://docs.microsoft.com/azure/role-based-access-control/overview) al recurso expressRouteCrossConnection. Estos controles pueden definir permisos para los que las cuentas de usuario pueden modificar el recurso expressRouteCrossConnection y agregar, actualizar o eliminar configuraciones de emparejamiento.

* El recurso expressRouteCrossConnection expone las API que pueden ser útiles para solucionar problemas de conexiones de ExpressRoute. Esto incluye la tabla ARP, el resumen de la tabla de rutas BGP y los detalles de la tabla de rutas BGP. Esta capacidad no es compatible con las API de implementación clásica.

* Los asociados también pueden buscar las comunidades anunciadas en el emparejamiento de Microsoft mediante el recurso *RouteFilter*.

## <a name="api-development-and-integration-steps"></a>Pasos de desarrollo e integración de API

Para desarrollar con Partner API, los asociados de ExpressRoute aprovechan la configuración de un cliente de prueba y un asociado de prueba. La configuración del cliente de prueba se usará para crear circuitos ExpressRoute en ubicaciones de emparejamiento de prueba que se asignan a puertos y dispositivos ficticios. La configuración del asociado de prueba se usa para administrar los circuitos ExpressRoute creados en la ubicación de emparejamiento de prueba.

### <a name="1-enlist-subscriptions"></a>1. Dar de alta suscripciones

Para solicitar la configuración del asociado de prueba y cliente de prueba, dé de alta dos suscripciones a Azure de Pago por uso para el contacto de ingeniería de ExpressRoute:
* **ExpressRoute_API_Dev_Provider_Sub:** Esta suscripción se usará para administrar los circuitos ExpressRoute creados en las ubicaciones de emparejamiento de prueba en dispositivos y puertos ficticios.

* **ExpressRoute_API_Dev_Customer_Sub:** Esta suscripción se usará para crear los circuitos ExpressRoute en las ubicaciones de emparejamiento de prueba que se asignan a dispositivos y puertos ficticios.

Ubicaciones de emparejamiento de prueba: los dispositivos y puertos ficticios no se exponen a los clientes de producción de forma predeterminada. Para crear circuitos ExpressRoute que se asignarán a la configuración de prueba, es necesario habilitar una marca de característica de suscripción.

### <a name="2-register-the-dev_provider-subscription-to-access-the-expressroutecrossconnections-api"></a>2. Registrar la suscripción Dev_Provider para acceder a la API expressRouteCrossConnections

Para acceder a la API expressRouteCrossConnections, la suscripción de asociado debe inscribirse en el **proveedor de recursos Microsoft.Network**. Siga los pasos descritos del artículo [Tipos y proveedores de recursos de Azure](/azure/azure-resource-manager/management/resource-providers-and-types#azure-portal) para completar el proceso de registro.

### <a name="3-set-up-authentication-for-azure-resource-manager-rest-api-calls"></a>3. Configurar la autenticación para las llamadas de API de REST de Azure Resource Manager

La mayoría de los servicios de Azure requieren el código de cliente para autenticarse en Resource Manager, mediante credenciales válidas, antes de llamar a las API de servicio. La autenticación la coordina Azure AD entre los distintos actores y proporciona al cliente un token de acceso como prueba de autenticación.

El proceso de autenticación implica dos pasos principales:

1. [Registrar el cliente](https://docs.microsoft.com/rest/api/azure/#register-your-client-application-with-azure-ad).
2. [Crear la solicitud de acceso](https://docs.microsoft.com/rest/api/azure/#create-the-request).

### <a name="4-provide-network-contributor-permission-to-the-client-application"></a>4. Proporcionar el permiso de colaborador de red a la aplicación cliente

Una vez que la autenticación se haya configurado correctamente, debe conceder acceso de colaborador de red a la aplicación cliente, en Dev_Provider_Sub. Para conceder permiso, inicie sesión en [Azure Portal](https://ms.portal.azure.com/#home) y complete estos pasos:

1. Navegue hasta Suscripciones y seleccione Dev_Provider_Sub.
2. Navegue hasta Control de acceso (IAM).
3. Agregue la asignación de roles.
4. Seleccione el rol Colaborador de red.
5. Asigne acceso al usuario, grupo o entidad de servicio de Azure AD.
6. Seleccione la aplicación cliente.
7. Guardar cambios

### <a name="5-develop"></a>5. Desarrollar

Desarrolle con la [API expressRouteCrossConnections](https://docs.microsoft.com/rest/api/expressroute/expressroutecrossconnections).

## <a name="rest-api"></a>API DE REST

Consulte [API de REST de CrossConnections de ExpressRoute](https://docs.microsoft.com/rest/api/expressroute/expressroutecrossconnections) para obtener documentación de la API de REST.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre todas las API de REST de ExpressRoute, consulte [API de REST de ExpressRoute](https://docs.microsoft.com/rest/api/expressroute/).