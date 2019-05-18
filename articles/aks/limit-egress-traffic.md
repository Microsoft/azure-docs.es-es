---
title: Restringir el tráfico de salida en Azure Kubernetes Service (AKS)
description: Obtenga información sobre qué puertos y direcciones son necesarias para controlar el tráfico de salida en Azure Kubernetes Service (AKS)
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 05/14/2019
ms.author: iainfou
ms.openlocfilehash: de0ba13a527569e446a44c275b7323d4487f53b6
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780310"
---
# <a name="preview---limit-egress-traffic-for-cluster-nodes-and-control-access-to-required-ports-and-services-in-azure-kubernetes-service-aks"></a>Versión preliminar: limitar el tráfico de salida para los nodos del clúster y controlar el acceso a los puertos necesarios y los servicios de Azure Kubernetes Service (AKS)

De forma predeterminada, los clústeres de AKS tiene acceso de internet saliente (salida) ilimitado. Este nivel de acceso de red permite a los nodos y servicios que necesitará ejecutar para tener acceso a recursos externos según sea necesario. Si desea restringir el tráfico de salida, un número limitado de puertos y direcciones debe ser accesible para mantener las tareas de mantenimiento del clúster en buen estado. El clúster, a continuación, está configurado para usar únicamente imágenes de contenedor del sistema básico de registro de contenedor de Microsoft (MCR) o Azure Container Registry (ACR), los repositorios públicos no externos.

Este artículo detalla qué puertos de red y nombres de dominio completo (FQDN) son obligatorios y opcionales si restringe el tráfico de salida en un clúster de AKS.  Esta funcionalidad actualmente está en su versión preliminar.

> [!IMPORTANT]
> Características de versión preliminar AKS son autoservicio y participación. Las versiones preliminares se proporcionan para recopilar comentarios y los errores de nuestra comunidad. Sin embargo, no se admiten por soporte técnico de Azure. Si crea un clúster, o agregar estas características para clústeres existentes, ese clúster no se admite hasta que la característica ya no está en versión preliminar y se aprueba para disponibilidad general (GA).
>
> Si tiene problemas con las características de vista previa, [abra una incidencia en el repositorio de GitHub de AKS] [ aks-github] con el nombre de la característica de vista previa en el título del error.

## <a name="before-you-begin"></a>Antes de empezar

Necesita la CLI de Azure versión 2.0.61 o posterior instalado y configurado. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure][install-azure-cli].

Para crear un clúster de AKS que puede limitar el tráfico de salida, habilitar una marca de características en su suscripción. Este registro de la función configura los clústeres AKS que ha creado para usar imágenes de contenedor del sistema básico de MCR o ACR. Para registrar el *AKSLockingDownEgressPreview* marca de características, use la [register de la característica de az] [ az-feature-register] comando tal como se muestra en el ejemplo siguiente:

```azurecli-interactive
az feature register --name AKSLockingDownEgressPreview --namespace Microsoft.ContainerService
```

Tarda unos minutos en que el estado muestre *Registrado*. Puede comprobar el estado de registro mediante el [lista de características de az] [ az-feature-list] comando:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AKSLockingDownEgressPreview')].{Name:name,State:properties.state}"
```

Cuando esté listo, actualizar el registro de la *Microsoft.ContainerService* proveedor de recursos mediante el uso de la [registrar proveedor de az] [ az-provider-register] comando:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

## <a name="egress-traffic-overview"></a>Descripción del tráfico de salida

Para fines operativos y administración, los nodos en un clúster de AKS deben tener acceso a determinados puertos y los nombres de dominio completo (FQDN). Estas acciones podrían ser para comunicarse con el servidor de API, o descargue e instale las actualizaciones de seguridad de nodo y de componentes principales del clúster de Kubernetes. De forma predeterminada, el tráfico de internet (saliente) de salida no está restringido para los nodos en un clúster de AKS. El clúster puede extraer del sistema base imágenes de contenedor de repositorios externos.

Para aumentar la seguridad de su clúster de AKS, es posible que desea restringir el tráfico de salida. El clúster está configurado para sistema básico de extracción de imágenes de contenedor de MCR o ACR. Si bloquea el tráfico de salida de esta manera, debe definir puertos específicos y FQDN para permitir que los nodos AKS para comunicarse correctamente con los servicios externos necesarios. Sin estos puertos autorizados y los FQDN, los nodos de AKS no pueden comunicarse con el servidor de API o instalar los componentes principales.

Puede usar [Firewall de Azure] [ azure-firewall] o un dispositivo de firewall 3rd-party para proteger el tráfico de salida y definirlos requiere los puertos y direcciones.

En AKS, hay dos conjuntos de puertos y direcciones:

* El [requería puertos y direcciones para los clústeres AKS](#required-ports-and-addresses-for-aks-clusters) se detallan los requisitos mínimos para el tráfico de salida autorizados.
* El [recomienda opcional direcciones y puertos para los clústeres AKS](#optional-recommended-addresses-and-ports-for-aks-clusters) no son necesarios para todos los escenarios, pero la integración con otros servicios como Azure Monitor no funcionará correctamente. Revise esta lista de los puertos opcionales y el FQDN y autorizar a cualquiera de los servicios y componentes que se usan en el clúster de AKS.

> [!NOTE]
> Limitar el tráfico de salida solo funciona en clústeres AKS nuevos creados después de habilitar el registro de la marca de característica. Para los clústeres existentes, [realizar una operación de actualización de clúster] [ aks-upgrade] utilizando la `az aks upgrade` antes de limitar el tráfico de salida del comando.

## <a name="required-ports-and-addresses-for-aks-clusters"></a>Puertos y direcciones para los clústeres AKS

Los siguientes puertos de salida / reglas de red son necesarias para un clúster de AKS:

* El puerto TCP *443*
* El puerto TCP *9000*

El siguiente FQDN / reglas de aplicación son necesarias:

| FQDN                      | Port      | Uso      |
|---------------------------|-----------|----------|
| *.azmk8s.io               | HTTPS:443 | Esta dirección es el punto de conexión de servidor de API. |
| aksrepos.azurecr.io       | HTTPS:443 | Esta dirección es necesaria para el acceso a las imágenes en Azure Container Registry (ACR). |
| * .blob.core.windows.net   | HTTPS:443 | Esta dirección es el almacén de back-end para las imágenes almacenadas en ACR. |
| mcr.microsoft.com         | HTTPS:443 | Esta dirección es necesaria para el acceso a las imágenes en el registro de contenedor de Microsoft (MCR). |
| management.azure.com      | HTTPS:443 | Esta dirección es necesaria para las operaciones de Kubernetes GET o PUT. |
| login.microsoftonline.com | HTTPS:443 | Esta dirección es necesaria para la autenticación de Azure Active Directory. |

## <a name="optional-recommended-addresses-and-ports-for-aks-clusters"></a>Opcional recomienda direcciones y puertos para los clústeres AKS

Los siguientes puertos de salida / reglas de red no son necesarias para los clústeres de AKS funcionar correctamente, pero se recomiendan:

* El puerto UDP *123* para la sincronización de hora NTP
* El puerto UDP *53* para DNS

El siguiente FQDN / reglas de aplicación se recomiendan para que clústeres AKS funcionar correctamente:

| FQDN                                    | Port      | Uso      |
|-----------------------------------------|-----------|----------|
| *.ubuntu.com                            | HTTP:80   | Esta dirección permite a los nodos del clúster de Linux descargar las revisiones de seguridad requeridos y actualizaciones. |
| packages.microsoft.com                  | HTTPS:443 | Esta dirección es el repositorio de paquetes de Microsoft usa para almacenar en caché *apt-get* operaciones. |
| dc.services.visualstudio.com            | HTTPS:443 | Se recomienda para las métricas correctas y la supervisión mediante Azure Monitor. |
| *.opinsights.azure.com                  | HTTPS:443 | Se recomienda para las métricas correctas y la supervisión mediante Azure Monitor. |
| *.monitoring.azure.com                  | HTTPS:443 | Se recomienda para las métricas correctas y la supervisión mediante Azure Monitor. |
| gov-prod-policy-data.trafficmanager.net | HTTPS:443 | Esta dirección se utiliza para el correcto funcionamiento de la directiva de Azure (actualmente en versión preliminar de AKS). |
| apt.dockerproject.org                   | HTTPS:443 | Esta dirección se utiliza para la instalación de controlador correcto y la operación en los nodos basados en GPU. |
| nvidia.github.io                        | HTTPS:443 | Esta dirección se utiliza para la instalación de controlador correcto y la operación en los nodos basados en GPU. |

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido qué puertos y direcciones para permitir si restringir el tráfico de salida para el clúster. También puede definir cómo se pueden comunicar los pods a sí mismos y qué restricciones tienen dentro del clúster. Para obtener más información, consulte [proteger el tráfico entre pods mediante las directivas de redes en AKS][network-policy].

<!-- LINKS - external -->
[aks-github]: https://github.com/azure/aks/issues]

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[network-policy]: use-network-policies.md
[azure-firewall]: ../firewall/overview.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[aks-upgrade]: upgrade-cluster.md
