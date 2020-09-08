---
title: Funcionamiento del enrutamiento con Azure Dev Spaces
services: azure-dev-spaces
ms.date: 03/24/2020
ms.topic: conceptual
description: Describe los procesos que posibilitan el funcionamiento de Azure Dev Spaces y cómo funciona el enrutamiento.
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores
ms.openlocfilehash: abdd8a573b7e120c5494b8878c2eb19813fc9507
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213409"
---
# <a name="how-routing-works-with-azure-dev-spaces"></a>Funcionamiento del enrutamiento con Azure Dev Spaces

Azure Dev Spaces le ofrece varias formas de iterar y depurar rápidamente las aplicaciones Kubernetes y colaborar con su equipo en un clúster de Azure Kubernetes Service (AKS). Una vez que el proyecto se ejecuta en un espacio de desarrollo, Azure Dev Spaces proporciona funcionalidades de red y enrutamiento adicionales para el proyecto.

En este artículo se describe cómo funciona el enrutamiento con Dev Spaces.

## <a name="how-routing-works"></a>Cómo funciona el enrutamiento

Los espacios de desarrollo se basan en AKS y usan los mismos [conceptos de redes](../aks/concepts-network.md). Azure Dev Spaces también tiene un servicio *ingressmanager* centralizado e implementa su propio controlador de entrada para el clúster de AKS. El servicio *ingressmanager* supervisa los clústeres de AKS con espacios de desarrollo y aumenta el controlador de entrada de Azure Dev Spaces en el clúster con objetos de entrada para el enrutamiento a los pods de la aplicación. El contenedor devspaces-proxy de cada pod agrega un encabezado HTTP `azds-route-as` para el tráfico HTTP a un espacio de desarrollo en función de la dirección URL. Por ejemplo, una solicitud a la dirección URL *http://azureuser.s.default.serviceA.fedcba09...azds.io* obtendría un encabezado HTTP con `azds-route-as: azureuser`. El contenedor devspaces-proxy no agregará un encabezado `azds-route-as` si ya hay alguno.

Cuando se realiza una solicitud HTTP a un servicio desde fuera del clúster, la solicitud se pasa al controlador de entrada. El controlador de entrada enruta la solicitud directamente al pod adecuado según sus objetos de entrada y sus reglas. El contenedor devspaces-proxy del pod recibe la solicitud, agrega el encabezado `azds-route-as` en función de la dirección URL y, después, enruta la solicitud al contenedor de la aplicación.

Cuando se realiza una solicitud HTTP a un servicio desde otro servicio del clúster, la solicitud pasa primero por el contenedor devspaces-proxy del servicio que realiza la llamada. El contenedor devspaces-proxy examina la solicitud HTTP y comprueba el encabezado `azds-route-as`. En función del encabezado, el contenedor devspaces-proxy buscará la dirección IP del servicio asociado con el valor del encabezado. Si se encuentra una dirección IP, el contenedor devspaces-proxy vuelve a enrutar la solicitud a esa dirección IP. Si no se encuentra ninguna dirección IP, el contenedor devspaces-proxy enruta la solicitud al contenedor primario de la aplicación.

Por ejemplo, los servicios *serviceA* y *serviceB* de la aplicación se implementan en un espacio de desarrollo primario denominado *default*. *serviceA* se basa en *serviceB* y realiza llamadas HTTP a este servicio. Azure User crea un espacio de desarrollo secundario basado en el espacio *default* denominado *azureuser*. Azure User también implementa su propia versión de *serviceA* en su espacio secundario. Cuando se realiza una solicitud a *http://azureuser.s.default.serviceA.fedcba09...azds.io* :

![Enrutamiento de Azure Dev Spaces](media/how-dev-spaces-works/routing.svg)

1. El controlador de entrada busca la dirección IP para el pod asociado a la dirección URL, que es *serviceA.azureuser*.
1. El controlador de entrada encuentra la dirección IP para el pod en el espacio de desarrollo de Azure User y enruta la solicitud al pod *serviceA.azureuser*.
1. El contenedor devspaces-proxy del pod *serviceA.azureuser* recibe la solicitud y agrega `azds-route-as: azureuser` como encabezado HTTP.
1. El contenedor devspaces-proxy del pod *serviceA.azureuser* enruta la solicitud al contenedor de la aplicación *serviceA* del pod *serviceA.azureuser*.
1. La aplicación *serviceA* del pod *serviceA.azureuser* realiza una llamada a *serviceB*. La aplicación *serviceA* también contiene código para conservar el encabezado `azds-route-as` existente, que en este caso es `azds-route-as: azureuser`.
1. El contenedor devspaces-proxy del pod *serviceA.azureuser* recibe la solicitud y busca la dirección IP de *serviceB* según el valor del encabezado `azds-route-as`.
1. El contenedor devspaces-proxy del pod *serviceA.azureuser* no encuentra una dirección IP para *serviceB.azureuser*.
1. El contenedor devspaces-proxy del pod *serviceA.azureuser* busca la dirección IP de *serviceB* en el espacio primario, que es *serviceB.default*.
1. El contenedor devspaces-proxy del pod *serviceA.azureuser* encuentra la dirección IP de *serviceB.default* y enruta la solicitud al pod *serviceB.default*.
1. El contenedor devspaces-proxy del pod *serviceB.default* recibe la solicitud y la enruta al contenedor de la aplicación *serviceB* del pod *serviceB.default*.
1. La aplicación *serviceB* del pod *serviceB.default* devuelve una respuesta al pod *serviceA.azureuser*.
1. El contenedor devspaces-proxy del pod *serviceA.azureuser* recibe la respuesta y la enruta al contenedor de la aplicación *serviceA* del pod *serviceA.azureuser*.
1. La aplicación *serviceA* recibe la respuesta y, después, devuelve su propia respuesta.
1. El contenedor devspaces-proxy del pod *serviceA.azureuser* recibe la respuesta del contenedor de la aplicación *serviceA* y la enruta al autor de la llamada original fuera del clúster.

El restante tráfico TCP que no es HTTP pasa a través del controlador de entrada y los contenedores devspaces-proxy sin modificar.

## <a name="sharing-a-dev-space"></a>Uso compartido de un espacio de desarrollo

Al trabajar con un equipo, puede compartir un espacio de desarrollo con el equipo al completo y crear espacios de desarrollo derivados. Cualquier persona con acceso de colaborador al grupo de recursos del espacio de desarrollo puede usar un espacio de desarrollo.

También puede crear un espacio de desarrollo que se derive de otro espacio de desarrollo. Al crear un espacio de desarrollo derivado, se agrega la etiqueta *azds.io/parent-space=PARENT-SPACE-NAME* al espacio de nombres del espacio de desarrollo derivado. Además, todas las aplicaciones del espacio de desarrollo primario se comparten con el espacio de desarrollo derivado. Si implementa una versión actualizada de una aplicación en el espacio de desarrollo derivado, solo existirá en este espacio y el espacio de desarrollo primario no se verá afectado. Puede tener un máximo de tres niveles de espacios de desarrollo derivados o espacios *primarios principales*.

El espacio de desarrollo derivado también enrutará las solicitudes de forma inteligente entre sus propias aplicaciones y las aplicaciones compartidas desde su elemento primario. Para llevar a cabo el enrutamiento, se intenta enrutar la solicitud a una aplicación del espacio de desarrollo derivado y se recurre a la aplicación compartida del espacio de desarrollo primario. El enrutamiento recurrirá a la aplicación compartida del espacio primario principal si la aplicación no está en el espacio primario.

Por ejemplo:
* El espacio de desarrollo *predeterminado* tiene las aplicaciones *serviceA* y *serviceB*.
* El espacio de desarrollo *azureuser* se deriva de *default*.
* Se implementa una versión actualizada de *serviceA* en *azureuser*.

Al usar *azureuser*, todas las solicitudes a *serviceA* se enrutarán a la versión actualizada en *azureuser*. Una solicitud a *serviceB* primero intentará enrutarse a la versión de *azureuser* de *serviceB*. Dado que no existe, se enrutará a la versión *default* de *serviceB*. Si se quita la versión de *azureuser* de *serviceA*, todas las solicitudes a *serviceA* recurrirán a usar la versión *default* de *serviceA*.

## <a name="next-steps"></a>Pasos siguientes

Para ver algunos ejemplos de cómo Azure Dev Spaces usa el enrutamiento para proporcionar una iteración y un desarrollo rápidos, consulte [Funcionamiento del proceso local con Kubernetes][how-it-works-local-process-kubernetes], [Funcionamiento de la depuración remota del código con Azure Dev Spaces][how-it-works-remote-debugging] y [Acciones de GitHub y Azure Kubernetes Service][pr-flow].


[helm-upgrade]: https://helm.sh/docs/intro/using_helm/#helm-upgrade-and-helm-rollback-upgrading-a-release-and-recovering-on-failure
[how-it-works-local-process-kubernetes]: /visualstudio/containers/overview-local-process-kubernetes
[how-it-works-remote-debugging]: how-dev-spaces-works-remote-debugging.md
[pr-flow]: how-to/github-actions.md
[troubleshooting]: troubleshooting.md