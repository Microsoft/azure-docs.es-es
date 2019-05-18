---
title: Introducción a los registros de alias de Azure DNS
description: Introducción a la compatibilidad de los registros de alias en Microsoft Azure DNS.
services: dns
author: vhorne
ms.service: dns
ms.topic: article
ms.date: 5/13/2019
ms.author: victorh
ms.openlocfilehash: 847ad271dac4afc8c8baa2faa8702b3a3ab6cefa
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65596708"
---
# <a name="azure-dns-alias-records-overview"></a>Introducción a los registros de alias de Azure DNS

Los registros de alias de Azure DNS son calificaciones en un conjunto de registros de DNS. Pueden hacer referencia a otros recursos de Azure desde dentro de la zona DNS. Por ejemplo, puede crear un conjunto de registros de alias que haga referencia a una dirección IP pública de Azure en lugar de a un registro D. El conjunto de registros de alias señala dinámicamente a una instancia de servicio de direcciones IP públicas de Azure. Como resultado, el conjunto de registros de alias se actualiza sin problemas durante la resolución DNS.

Un conjunto de registros de alias se admite para los siguientes tipos de registros en una zona de Azure DNS: 

- Una 
- AAAA
- CNAME

> [!NOTE]
> Si pretende usar un registro de alias para los tipos de registro A o AAAA para apuntar a un [perfil de Azure Traffic Manager](../traffic-manager/quickstart-create-traffic-manager-profile.md), debe asegurarse de que el perfil de Traffic Manager solo tenga [puntos de conexión externos](../traffic-manager/traffic-manager-endpoint-types.md#external-endpoints). Debe proporcionar la dirección IPv4 o IPv6 para los puntos de conexión externos en Traffic Manager. No se puede usar nombres de dominio completo (FQDN) en los puntos de conexión. Lo ideal es usar direcciones IP estáticas.

## <a name="capabilities"></a>Capacidades

- **Apuntar a un recurso de dirección IP pública desde un conjunto de registros A/AAAA de DNS**. Puede crear un conjunto de registros D/AAAA y hacer que sea un conjunto de registros de alias que apunte a un recurso de dirección IP pública. El conjunto de registros de DNS es automático si la dirección IP pública cambia o se elimina. Se evitan los registros de DNS pendientes que apuntan a direcciones IP incorrectas.

- **Apuntar a un perfil de Traffic Manager desde un conjunto de registros D/AAAA/CNAME de DNS.** Puede crear un conjunto de registros A/AAAA o CNAME y usar los registros de alias para apuntarlo a un perfil de Traffic Manager. Esto resulta especialmente útil si necesita enrutar el tráfico a un vértice de zona, ya que los registros CNAME tradicionales no se admiten para un vértice de zona. Por ejemplo, supongamos que su perfil de Traffic Manager es myprofile.trafficmanager.net y la zona DNS de la empresa es contoso.com. Puede crear un conjunto de registros de alias de tipo A/AAAA para contoso.com (el vértice de zona) y apuntar a myprofile.trafficmanager.net.
- **Seleccione un punto de conexión de Azure Content Delivery Network (CDN)**. Esto es útil al crear los sitios Web estáticos con Azure storage y red CDN de Azure.
- **Apuntar a otro conjunto de registros de DNS dentro de la misma zona**. Los registros de alias pueden hacer referencia a otros conjuntos de registros del mismo tipo. Por ejemplo, un conjunto de registros CNAME de DNS puede ser un alias de otro conjunto de registros CNAME. Esta organización resulta útil si desea que algunos conjuntos de registros sean alias y otros no alias.

## <a name="scenarios"></a>Escenarios

Hay algunos escenarios comunes para los registros de alias.

### <a name="prevent-dangling-dns-records"></a>Impedir que los registros DNS queden pendientes

Un problema común de los registros DNS tradicionales son los registros pendientes. Por ejemplo, los registros DNS que no se han actualizado para reflejar los cambios en las direcciones IP. El error ocurre especialmente con tipos de registros A/AAAA o CNAME.

En el caso de un registro de zona DNS tradicional, si la dirección IP de destino o CNAME ya no existen, el registro DNS asociado debe actualizarse manualmente. En algunas organizaciones, puede que una actualización manual no se produzca a tiempo por problemas de proceso o debido a la separación de roles y niveles de permiso asociados. Por ejemplo, un rol podría tener autoridad para eliminar una dirección IP o CNAME que pertenezca a una aplicación. Pero no tiene autoridad suficiente para actualizar el registro DNS que apunta a esos destinos. Un retraso en la actualización del registro DNS puede provocar una interrupción del servicio para los usuarios.

Los registros de alias impiden referencias pendientes mediante un acoplamiento estrecho del ciclo de vida de un registro DNS con un recurso de Azure. Por ejemplo, considere un registro DNS que se califica como un registro de alias que apunte a una dirección IP pública o a un perfil de Traffic Manager. Si se eliminan los recursos subyacentes, se quita el registro de alias DNS al mismo tiempo.

### <a name="update-dns-record-set-automatically-when-application-ip-addresses-change"></a>Actualización automática del conjunto de registros DNS cuando las direcciones IP de la aplicación cambian

Este escenario es similar al anterior. Puede que una aplicación se mueva o la máquina virtual subyacente se reinicie. Un registro de alias se actualiza entonces automáticamente cuando la dirección IP cambia para el recurso de direcciones IP públicas subyacente. Esto evita posibles riesgos de seguridad de dirigir a los usuarios a otra aplicación asignada a la dirección IP pública anterior.

### <a name="host-load-balanced-applications-at-the-zone-apex"></a>Hospedaje de aplicaciones con equilibrio de carga en el vértice de la zona

El protocolo DNS evita la asignación de registros CNAME en el vértice de zona. Por ejemplo, si el dominio es contoso.com, puede crear registros CNAME para somelable.contoso.com, pero no puede crear CNAME para contoso.com.
Esta restricción presenta un problema para los propietarios de aplicaciones que tienen aplicaciones con equilibrio de carga detrás de [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md). Puesto que el uso de un perfil de Traffic Manager requiere la creación de un registro CNAME, no es posible apuntar al perfil de Traffic Manager desde el vértice de zona.

Este problema puede resolverse mediante los registros de alias. A diferencia de los registros CNAME, se pueden crear registros de alias en el vértice de zona y los propietarios de aplicaciones pueden usarlos para apuntar su registro de vértice de zona a un perfil de Traffic Manager que tiene puntos de conexión externos. Los propietarios de aplicaciones pueden apuntar al mismo perfil de Traffic Manager que se utilice para cualquier otro dominio dentro de su zona DNS.

Por ejemplo, contoso.com y www\.contoso.com puede apuntar al mismo perfil de Traffic Manager. Para obtener más información sobre el uso de registros de alias con perfiles de Azure Traffic Manager, consulte la sección Pasos siguientes.

### <a name="point-zone-apex-to-azure-cdn-endpoints"></a>Seleccione el vértice de zona a puntos de conexión de red CDN de Azure

Al igual que un perfil de Traffic Manager, también puede usar los registros de alias para que señale el vértice de zona DNS a los puntos de conexión de red CDN de Azure. Esto es útil al crear los sitios Web estáticos con Azure storage y red CDN de Azure. A continuación, puede acceder al sitio Web sin anteponiendo "www" al nombre de DNS.

Por ejemplo, si el sitio Web estático se denomina www.contoso.com, los usuarios pueden tener acceso a su sitio mediante contoso.com sin necesidad de anteponer www en el nombre DNS.

Como se describió anteriormente, no se admiten los registros CNAME en el vértice de zona. Por lo tanto, no se puede usar un registro CNAME para que apunte a contoso.com al extremo de CDN. En su lugar, puede usar un registro de alias para que señale el vértice de zona a un punto de conexión CDN directamente.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los registros de alias, consulte los artículos siguientes:

- [Tutorial: Configuración de un registro de alias para hacer referencia a una dirección IP pública de Azure](tutorial-alias-pip.md)
- [Tutorial: Configuración de un registro de alias para admitir nombres de dominio de Apex con Traffic Manager](tutorial-alias-tm.md)
- [Preguntas más frecuentes sobre DNS](https://docs.microsoft.com/azure/dns/dns-faq#alias-records)
