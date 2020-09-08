---
title: Configuración de red para Azure Dev Spaces en distintas topologías de red
services: azure-dev-spaces
ms.date: 03/17/2020
ms.topic: conceptual
description: Se describen los requisitos de red para ejecutar Azure Dev Spaces en Azure Kubernetes Services.
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores, CNI, Kubenet, SDN, red
ms.openlocfilehash: 0d9ebbec3e3c07a466acb58e88b67e6a32a20edb
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88214174"
---
# <a name="configure-networking-for-azure-dev-spaces-in-different-network-topologies"></a>Configuración de red para Azure Dev Spaces en distintas topologías de red

Azure Dev Spaces se ejecuta en clústeres de Azure Kubernetes Service (AKS) con la configuración de red predeterminada. Si quiere cambiar la configuración de red del clúster de AKS, como poner el clúster detrás de un firewall, usar grupos de seguridad de red o directivas de red, debe incorporar consideraciones adicionales para ejecutar Azure Dev Spaces.

![Configuración de redes virtuales](media/configure-networking/virtual-network-clusters.svg)

## <a name="virtual-network-or-subnet-configurations"></a>Configuraciones de la red virtual o subred

El clúster de AKS puede tener una configuración diferente de red virtual o subred para restringir el tráfico de entrada o salida para el clúster de AKS. Por ejemplo, el clúster puede estar detrás de un firewall, como Azure Firewall, o puede usar grupos de seguridad de red o roles personalizados para restringir el tráfico de red. Puede encontrar un ejemplo de configuración de red en el [repositorio de ejemplos de Azure Dev Spaces en GitHub][sample-repo].

Azure Dev Spaces tiene ciertos requisitos para el tráfico de red de *entrada y salida*, así como para el tráfico de *solo entrada*. Si usa Azure Dev Spaces en un clúster de AKS con una configuración de red virtual o subred que restringe el tráfico para el clúster de AKS, debe seguir los siguientes requisitos de tráfico de entrada y salida y de solo entrada para que Azure Dev Spaces funcione correctamente.

### <a name="ingress-and-egress-network-traffic-requirements"></a>Requisitos de tráfico de red de entrada y salida

Azure Dev Spaces necesita tráfico de entrada y salida para los FQDN siguientes:

| FQDN                       | Port       | Uso      |
|----------------------------|------------|----------|
| cloudflare.docker.com      | HTTPS: 443 | Para extraer imágenes de Docker para Azure Dev Spaces |
| gcr.io                     | HTTPS: 443 | Para extraer imágenes de Helm para Azure Dev Spaces |
| storage.googleapis.com     | HTTPS: 443 | Para extraer imágenes de Helm para Azure Dev Spaces |

Actualice el firewall o la configuración de seguridad para permitir el tráfico de red hacia y desde todos los FQDN anteriores y los [servicios de infraestructura de Azure Dev Spaces][service-tags]. Por ejemplo, si usa un firewall para proteger la red, se deben agregar los FQDN anteriores a la regla de aplicación del firewall, y la etiqueta de servicio de Azure Dev Spaces también se deberá [agregar al firewall][firewall-service-tags]. Estas dos actualizaciones del firewall son obligatorias para permitir el tráfico hacia y desde estos dominios.

### <a name="ingress-only-network-traffic-requirements"></a>Requisitos de tráfico de red de solo entrada

Azure Dev Spaces proporciona enrutamiento de nivel de espacio de nombres de Kubernetes y acceso público a los servicios mediante su propio FQDN. Para que ambas características funcionen, actualice el firewall o la configuración de red para permitir la entrada pública en la dirección IP externa del controlador de entrada de Azure Dev Spaces del clúster. También puede crear un [equilibrador de carga interno][aks-internal-lb] y agregar una regla NAT en el firewall para traducir la dirección IP pública del firewall a la dirección IP del equilibrador de carga interno. También puede usar [traefik][traefik-ingress] o [NGINX][nginx-ingress] para crear un controlador de entrada personalizado.

## <a name="aks-cluster-network-requirements"></a>Requisitos de red del clúster de AKS

AKS permite usar [directivas de red][aks-network-policies] para controlar el tráfico de entrada y salida entre pods en un clúster, así como el tráfico de salida de un pod. Azure Dev Spaces tiene ciertos requisitos para el tráfico de red de *entrada y salida*, así como para el tráfico de *solo entrada*. Si usa Azure Dev Spaces en un clúster de AKS con directivas de red de AKS, debe seguir los siguientes requisitos de tráfico de entrada y salida y de solo entrada para que Azure Dev Spaces funcione correctamente.

### <a name="ingress-and-egress-network-traffic-requirements"></a>Requisitos de tráfico de red de entrada y salida

Azure Dev Spaces le permite comunicarse directamente con un pod en un espacio de desarrollo en el clúster para la depuración. Para que esta característica funcione, agregue una directiva de red que permita la comunicación de entrada y salida con las direcciones IP de la infraestructura de Azure Dev Spaces, que puede [variar por región][service-tags].

### <a name="ingress-only-network-traffic-requirements"></a>Requisitos de tráfico de red de solo entrada

Azure Dev Spaces proporciona enrutamiento entre los pods entre espacios de nombres. Por ejemplo, los espacios de nombres con Azure Dev Spaces habilitado pueden tener una relación de elementos primarios y secundarios, lo que permite enrutar el tráfico de red entre los pods entre los espacios de nombres primarios y secundarios. También Azure Dev Spaces expone puntos de conexión de servicio mediante su propio FQDN. Para configurar maneras diferentes de exponer servicios y cómo afecta al enrutamiento de nivel de espacio de nombres, consulte [Usar diferentes opciones de punto de conexión][endpoint-options].

## <a name="using-azure-cni"></a>Uso de CNI de Azure

De manera predeterminada, los clústeres de AKS se configuran para usar [Kubenet][aks-kubenet] para redes, que funciona con Azure Dev Spaces. También puede configurar el clúster de AKS para usar la [interfaz de red de contenedor (CNI) de Azure][aks-cni]. Para usar Azure Dev Spaces con CNI de Azure en el clúster de AKS, permita que la red virtual y los espacios de direcciones de subred tengan hasta 10 direcciones IP privadas para los pods implementados por Azure Dev Spaces. Puede encontrar más información sobre cómo permitir las direcciones IP privadas en la [documentación de CNI de Azure de AKS][aks-cni-ip-planning].

## <a name="using-api-server-authorized-ip-ranges"></a>Uso de intervalos IP autorizados de servidor de API

Los clústeres de AKS permiten configurar medidas de seguridad adicionales que limiten la dirección IP que puede interactuar con los clústeres, por ejemplo, el uso de redes virtuales personalizadas o la [protección del acceso al servidor de API mediante intervalos IP autorizados][aks-ip-auth-ranges]. Para utilizar Azure Dev Spaces cuando se usen estas medidas de seguridad adicionales al [crear][aks-ip-auth-range-create] el clúster, debe [permitir intervalos adicionales según la región][service-tags]. También puede [actualizar][aks-ip-auth-range-update] un clúster existente para permitir esos intervalos adicionales. También debe permitir que la dirección IP de cualquier máquina de desarrollo que se conecte al clúster de AKS para la depuración se conecte al servidor de API.

## <a name="using-aks-private-clusters"></a>Uso de clústeres privados de AKS

En este momento, Azure Dev Spaces no es compatible con los [clústeres privados de AKS][aks-private-clusters].

## <a name="using-different-endpoint-options"></a>Usar diferentes opciones de punto de conexión

Azure Dev Spaces tiene la opción de exponer los puntos de conexión de los servicios que se ejecutan en AKS. Al habilitar Azure Dev Spaces en el clúster, tiene las siguientes opciones para configurar el tipo de punto de conexión para el clúster:

* Un punto de conexión *público*, que es el valor predeterminado, implementa un controlador de entrada con una dirección IP pública. La dirección IP pública se registra en el DNS del clúster, lo que permite el acceso público a los servicios mediante una dirección URL. Puede ver esta dirección URL mediante `azds list-uris`.
* Un punto de conexión *privado* implementa un controlador de entrada con una dirección IP privada. Con una dirección IP privada, el equilibrador de carga del clúster solo es accesible desde dentro de la red virtual del clúster. La dirección IP privada del equilibrador de carga se registra en el DNS del clúster para que se pueda tener acceso a los servicios dentro de la red virtual del clúster mediante una dirección URL. Puede ver esta dirección URL mediante `azds list-uris`.
* Si se establece *ninguno* para la opción de punto de conexión, no se implementará ningún controlador de entrada. Sin el controlador de entrada implementado, las [funciones de enrutamiento de Azure Dev Spaces][dev-spaces-routing] no funcionarán. Opcionalmente, puede implementar su propia solución de controlador de entrada mediante [traefik][traefik-ingress] o [NGINX][nginx-ingress], lo que permitirá que las funciones de enrutamiento vuelvan a funcionar.

Para configurar la opción de punto de conexión, use *-e* o *--punto de conexión* al habilitar Azure Dev Spaces en el clúster. Por ejemplo:

> [!NOTE]
> La opción de punto de conexión requiere que se ejecute CLI de Azure versión 2.2.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

```azurecli
az aks use-dev-spaces -g MyResourceGroup -n MyAKS -e private
```

## <a name="client-requirements"></a>Requisitos del cliente

Azure Dev Spaces usa herramientas del lado cliente, como la extensión de la CLI de Azure Dev Spaces, la extensión de Visual Studio Code y la extensión de Visual Studio, para comunicarse con el clúster de AKS para la depuración. Para usar las herramientas del lado cliente de Azure Dev Spaces, permita el tráfico desde las máquinas de desarrollo a la [infraestructura de Azure Dev Spaces][dev-spaces-allow-infrastructure]. Si usa [intervalo IP autorizados del servidor de API][auth-range-section], también debe permitir que la dirección IP de cualquier máquina de desarrollo que se conecte al clúster de AKS para la depuración se conecte al servidor de API.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre cómo funciona Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Funcionamiento de Azure Dev Spaces](how-dev-spaces-works.md)

[aks-cni]: ../aks/configure-azure-cni.md
[aks-cni-ip-planning]: ../aks/configure-azure-cni.md#plan-ip-addressing-for-your-cluster
[aks-kubenet]: ../aks/configure-kubenet.md
[aks-internal-lb]: ../aks/internal-lb.md
[aks-ip-auth-ranges]: ../aks/api-server-authorized-ip-ranges.md
[aks-ip-auth-range-create]: ../aks/api-server-authorized-ip-ranges.md#create-an-aks-cluster-with-api-server-authorized-ip-ranges-enabled
[aks-ip-auth-range-update]: ../aks/api-server-authorized-ip-ranges.md#update-a-clusters-api-server-authorized-ip-ranges
[aks-network-policies]: ../aks/use-network-policies.md
[aks-private-clusters]: ../aks/private-clusters.md
[auth-range-section]: #using-api-server-authorized-ip-ranges
[azure-cli-install]: /cli/azure/install-azure-cli
[dev-spaces-allow-infrastructure]: #virtual-network-or-subnet-configurations
[dev-spaces-routing]: how-dev-spaces-works-routing.md
[endpoint-options]: #using-different-endpoint-options
[firewall-service-tags]: ../firewall/service-tags.md
[traefik-ingress]: how-to/ingress-https-traefik.md
[nginx-ingress]: how-to/ingress-https-nginx.md
[sample-repo]: https://github.com/Azure/dev-spaces/tree/master/advanced%20networking
[service-tags]: ../virtual-network/service-tags-overview.md#available-service-tags