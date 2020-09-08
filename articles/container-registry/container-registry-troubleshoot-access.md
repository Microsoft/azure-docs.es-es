---
title: Solución de problemas de red con el registro
description: Síntomas, causas y resolución de problemas comunes al acceder a un registro de contenedor de Azure en una red virtual o detrás de un firewall
ms.topic: article
ms.date: 08/11/2020
ms.openlocfilehash: 227eeeadb2aef4b4d3feb7923a198b129a6267d3
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88226978"
---
# <a name="troubleshoot-network-issues-with-registry"></a>Solución de problemas de red con el registro

Este artículo le ayuda a solucionar problemas que pueden surgir al acceder a un registro de contenedor de Azure en una red virtual o detrás de un firewall. 

## <a name="symptoms"></a>Síntomas

Puede encontrarse con uno o varios de los siguientes:

* No es posible insertar o extraer imágenes, y se recibe un error `dial tcp: lookup myregistry.azurecr.io`
* No es posible insertar o extraer imágenes, y se recibe un error de la CLI de Azure `Could not connect to the registry login server`
* No es posible extraer imágenes del registro en Azure Kubernetes Service u otro servicio de Azure
* No se puede acceder a un registro detrás de un proxy HTTPS, y se recibe el error `Error response from daemon: login attempt failed with status: 403 Forbidden`
* No se puede acceder a la configuración del registro ni verla en Azure Portal; tampoco se puede administrar el registro mediante la CLI de Azure
* No se puede agregar o modificar la configuración de red virtual o las reglas de acceso público
* ACR Tasks no puede insertar ni extraer imágenes
* Azure Security Center no pueden examinar imágenes en el registro o los resultados de los exámenes no aparecen en Azure Security Center

## <a name="causes"></a>Causas

* Un firewall o proxy de cliente impide el acceso: [solución](#configure-client-firewall-access)
* Las reglas de acceso de la red pública en el registro impiden el acceso: [solución](#configure-public-access-to-registry)
* La configuración de la red virtual impide el acceso: [solución](#configure-vnet-access)
* Intente integrar Azure Security Center con un registro que tenga un punto de conexión privado o un punto de conexión de servicio: [solución](#configure-image-scanning-solution)

## <a name="further-diagnosis"></a>Diagnóstico detallado 

Ejecute el comando [az acr check-helth](/cli/azure/acr#az-acr-check-health) para tener más información sobre el estado del entorno y el acceso opcional a un registro de destino. Por ejemplo, diagnostique ciertos problemas de configuración o conectividad de la red. 

Puede encontrar ejemplos de comandos en [Comprobación del mantenimiento de un registro de contenedor de Azure](container-registry-check-health.md). Si se notifican errores, revise la [referencia de error](container-registry-health-error-reference.md) y las siguientes secciones para ver las soluciones recomendadas.

> [!NOTE]
> Algunos síntomas de conectividad de red también pueden producirse cuando hay problemas con la autenticación o la autorización del registro. Consulte [Solución de problemas de inicio de sesión del registro](container-registry-troubleshoot-login.md).

## <a name="potential-solutions"></a>Posibles soluciones

### <a name="configure-client-firewall-access"></a>Configurar el acceso del firewall del cliente

Para acceder a un registro desde detrás de un firewall de cliente o de un servidor proxy, configure las reglas de firewall para acceder a los puntos de conexión de REST y de datos del registro. Si están habilitados [puntos de conexión de datos dedicados](container-registry-firewall-access-rules.md#enable-dedicated-data-endpoints), necesitará reglas para acceder a:

* Punto de conexión de REST: `<registryname>.azurecr.io`
* Puntos de conexión de datos: `<registry-name>.<region>.data.azurecr.io`

En el caso de un registro con replicación geográfica, configure el acceso al punto de conexión de datos en cada réplica regional.

Detrás del proxy HTTPS, compruebe que tanto el cliente como el demonio de Docker estén configurados para admitir el comportamiento del proxy.

Los registros de recursos del registro de la tabla ContainerRegistryLoginEvents pueden ayudar a diagnosticar los intentos de conexión bloqueados.

Vínculos relacionados:

* [Configuración de reglas para acceder a un registro de contenedor de Azure detrás de un firewall](container-registry-firewall-access-rules.md)
* [Configuración del proxy HTTP/HTTPS](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)
* [Replicación geográfica de Azure Container Registry](container-registry-geo-replication.md)
* [Registros de Azure Container Registry para la evaluación y auditoría de diagnóstico](container-registry-diagnostics-audit-logs.md)

### <a name="configure-public-access-to-registry"></a>Configurar el acceso público al registro

Si tiene acceso a un registro a través de Internet, confirme que el registro permite el acceso a la red pública desde el cliente. De forma predeterminada, un registro de contenedor de Azure permite el acceso a los puntos de conexión del registro público desde todas las redes. Un registro puede limitar el acceso a redes o direcciones IP seleccionadas. 

Si el registro está configurado para una red virtual con un punto de conexión de servicio, al deshabilitar el acceso a la red pública también se deshabilita el acceso a través del punto de conexión de servicio. Si el registro está configurado para una red virtual con Private Link, las reglas de red IP no se aplican a los puntos de conexión privados del registro. 

Vínculos relacionados:

* [Configuración de reglas de red de dirección IP pública](container-registry-access-selected-networks.md)
* [Conexión privada a un registro de contenedor de Azure mediante Azure Private Link](container-registry-private-link.md)
* [Restricción del acceso a un registro de contenedor mediante un punto de conexión de servicio en una red virtual de Azure](container-registry-vnet.md)


### <a name="configure-vnet-access"></a>Configurar el acceso a la red virtual

Confirme que la red virtual está configurada con un punto de conexión privado para Private Link o un punto de conexión de servicio (versión preliminar). Actualmente, no se admite un punto de conexión de Azure Bastion.

Revise las reglas del grupo de seguridad de red y las etiquetas de servicio que se usan para limitar el tráfico que va desde otros recursos de la red hacia el registro. 

Si se ha configurado un punto de conexión de servicio al registro, confirme que se agrega una regla de red al registro que permita el acceso desde la subred de esa red. El punto de conexión de servicio solo admite el acceso desde máquinas virtuales y clústeres de AKS en la red.

Si Azure Firewall o una solución similar está configurada en la red, compruebe que el tráfico de salida de otros recursos, como un clúster de AKS, está habilitado para llegar a los puntos de conexión del registro.

Vínculos relacionados:

* [Conexión privada a un registro de contenedor de Azure mediante Azure Private Link](container-registry-private-link.md)
* [Restricción del acceso a un registro de contenedor mediante un punto de conexión de servicio en una red virtual de Azure](container-registry-vnet.md)
* [Reglas de red de salida y FQDN necesarios para clústeres de AKS](../aks/limit-egress-traffic.md#required-outbound-network-rules-and-fqdns-for-aks-clusters)
* [Kubernetes: Resolución de depuración de DNS](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)
* [Etiquetas de servicio de red virtual](../virtual-network/service-tags-overview.md)

### <a name="configure-image-scanning-solution"></a>Configurar la solución de examen de imágenes

Si el registro está configurado con un punto de conexión privado o un punto de conexión de servicio, actualmente no se puede integrar con el análisis de imágenes de Azure Security Center. Opcionalmente, puede configurar otras soluciones de examen de imágenes disponibles en Azure Marketplace, entre las que se incluyen:

* [Aqua Cloud Native Security Platform](https://azuremarketplace.microsoft.com/marketplace/apps/aqua-security.aqua-security)
* [Twistlock Enterprise Edition](https://azuremarketplace.microsoft.com/marketplace/apps/twistlock.twistlock)

Vínculos relacionados:

* [Análisis de imágenes de Azure Container Registry por Security Center](../security-center/azure-container-registry-integration.md)
* [Envío de comentarios](https://feedback.azure.com/forums/347535-azure-security-center/suggestions/41091577-enable-vulnerability-scanning-for-images-that-are)


## <a name="advanced-troubleshooting"></a>Pasos detallados para solucionar problemas de conexión a Escritorio remoto a máquinas virtuales Windows en Azure

Si la [colección de registros de recursos](container-registry-diagnostics-audit-logs.md) está habilitada en el registro, revise el registro ContainterRegistryLoginEvents. Este registro almacena los eventos y el estado de autenticación, incluida la identidad entrante y la dirección IP. Consulte el registro para conocer los [errores de autenticación del registro](container-registry-diagnostics-audit-logs.md#registry-authentication-failures). 

Vínculos relacionados:

* [Registros para la evaluación y auditoría de diagnóstico](container-registry-diagnostics-audit-logs.md)
* [Preguntas frecuentes sobre Container Registry](container-registry-faq.md)
* [Base de referencia de Azure Security para Azure Container Registry](security-baseline.md)
* [Procedimientos recomendados para Azure Container Registry](container-registry-best-practices.md)

## <a name="next-steps"></a>Pasos siguientes

Si no encuentra aquí una solución para su problema, vea las siguientes opciones.

* Otros temas de solución de problemas del registro incluyen:
  * [Solución de problemas de inicio de sesión del registro](container-registry-troubleshoot-login.md) 
  * [Solución de problemas de rendimiento del registro](container-registry-troubleshoot-performance.md)
* Opciones de [soporte técnico de la comunidad](https://azure.microsoft.com/support/community/)
* [Preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/products/)
* [Abrir una incidencia de soporte técnico](https://azure.microsoft.com/support/create-ticket/)


