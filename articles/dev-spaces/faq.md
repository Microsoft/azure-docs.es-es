---
title: Preguntas más frecuentes sobre Azure Dev Spaces
services: azure-dev-spaces
ms.date: 01/28/2020
ms.topic: conceptual
description: Encuentre respuestas a algunas de las preguntas comunes sobre Azure Dev Spaces.
keywords: 'Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores, Helm, service mesh, enrutamiento de service mesh, kubectl, k8s '
ms.openlocfilehash: 739cfdb4f930d384e180ffd0f3ce2311cd41c70a
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88207996"
---
# <a name="frequently-asked-questions-about-azure-dev-spaces"></a>Preguntas más frecuentes sobre Azure Dev Spaces

Aborda las preguntas más frecuentes sobre Azure Dev Spaces.

## <a name="what-versions-of-kubernetes-are-supported-for-azure-dev-spaces"></a>¿Qué versiones de Kubernetes son compatibles con Azure Dev Spaces?

Azure Dev Spaces admite todas las [versiones de disponibilidad general (GA) de Kubernetes admitidas actualmente en AKS][aks-supported-k8s].

## <a name="which-azure-regions-currently-provide-azure-dev-spaces"></a>¿Qué regiones de Azure proporcionan actualmente Azure Dev Spaces?

Consulte las [regiones admitidas][supported-regions] para obtener una lista completa de las regiones disponibles.

## <a name="can-i-migrate-my-aks-cluster-with-azure-dev-spaces-to-another-region"></a>¿Puedo migrar mi clúster de AKS con Azure Dev Spaces a otra región?

Sí, si quiere trasladar el clúster de AKS con Azure Dev Spaces a otra [región compatible][supported-regions], se recomienda crear un nuevo clúster en la otra región y, a continuación, instalar y configurar Azure Dev Spaces e implementar los recursos y las aplicaciones en el nuevo clúster. Para más información sobre la migración de AKS, consulte [Migrar a Azure Kubernetes Service (AKS)][aks-migration].

## <a name="can-i-use-azure-dev-spaces-with-existing-dockerfiles-or-helm-charts"></a>¿Puedo usar Azure Dev Spaces con gráficos existentes de Dockerfiles o Helm?

Sí, si el proyecto ya tiene un gráfico de Dockerfile o Helm, puede usar esos archivos con Azure Dev Spaces. Al ejecutar `azds prep`, use el parámetro `--chart` y especifique la ubicación del gráfico. Azure Dev Spaces seguirá generando un archivo *azds.yaml* y *Dockerfile.develop*, pero no reemplazará ni modificará un gráfico de Helm o Dockerfile existente. Es posible que tenga que modificar los archivos *azds.yaml* y *Dockerfile.develop* para que todo funcione correctamente con la aplicación existente cuando ejecute `azds up`.

Al usar su propio gráfico de Dockerfile o Helm, existen las siguientes limitaciones:
* Si solo usa un gráfico de Dockerfile, debe incluir todo lo necesario para habilitar escenarios de desarrollo, como el SDK de idioma, y no solo el runtime. Si usa un gráfico de Dockerfile independiente para Azure Dev Spaces, como el archivo Dockerfile.develop, todo lo que necesite para habilitar escenarios de desarrollo debe estar incluido en ese gráfico Dockerfile.
* El gráfico de Helm debe permitir pasar parte o toda la etiqueta de imagen como un valor desde *values.yaml*.
* Si va a modificar un elemento con entrada, también puede actualizar el gráfico de Helm para usar la solución de entrada proporcionada por Azure Dev Spaces.
* Si desea utilizar las [funcionalidades de enrutamiento que Azure Dev Spaces proporciona][dev-spaces-routing], todos los servicios de un proyecto individual deben caber dentro de un único espacio de nombres Kubernetes y deben implementarse con nombres simples, por ejemplo *servicio-a*. En los gráficos de Helm estándar, para llevar a cabo esta actualización de nombres se puede especificar un valor para la propiedad *fullnameOverride*.

## <a name="can-i-modify-the-files-generated-by-azure-dev-spaces"></a>¿Puedo modificar los archivos generados por Azure Dev Spaces?

Sí, puede modificar el archivo *azds.yaml* y el gráfico de Dockerfile y de Helm[ generados por Azure Dev Spaces al preparar el proyecto][dev-spaces-prep]. Al modificar estos archivos, se cambia el modo en que el proyecto se compila y se ejecuta.

## <a name="can-i-use-azure-dev-spaces-without-a-public-ip-address"></a>¿Puedo usar Azure Dev Spaces sin una dirección IP pública?

No, no se puede aprovisionar Azure Dev Spaces en un clúster de AKS sin una dirección IP pública. Una dirección IP pública es [necesaria para el enrutamiento en Azure Dev Spaces][dev-spaces-routing].

## <a name="can-i-use-my-own-ingress-with-azure-dev-spaces"></a>¿Puedo usar mi propia entrada con Azure Dev Spaces?

Sí, puede configurar su propia entrada junto con la que Azure Dev Spaces crea. Por ejemplo, puede usar [traefik][ingress-traefik] o [NGINX][ingress-nginx].

## <a name="can-i-use-https-with-azure-dev-spaces"></a>¿Puedo usar HTTPS con Azure Dev Spaces?

Sí, puede configurar su propia entrada con HTTPS mediante [traefik][ingress-https-traefik] o [NGINX][ingress-https-nginx].

## <a name="can-i-use-azure-dev-spaces-on-a-cluster-that-uses-cni-rather-than-kubenet"></a>¿Puedo usar Azure Dev Spaces en un clúster que use CNI en lugar de kubenet? 

Sí, puede usar Azure Dev Spaces en un clúster de AKS que use CNI para redes. Por ejemplo, puede usar Azure Dev Spaces en un clúster de AKS con [contenedores de Windows existentes][windows-containers] que use CNI para redes. Puede encontrar más información sobre el uso de CNI para redes con Azure Dev Spaces [aquí](configure-networking.md#using-azure-cni).

## <a name="can-i-use-azure-dev-spaces-with-windows-containers"></a>¿Puedo usar Azure Dev Spaces con contenedores de Windows?

Actualmente, Azure Dev Spaces está diseñado para ejecutarse solo en nodos y pods de Linux, pero puede ejecutar Azure Dev Spaces en un clúster de AKS con [contenedores de Windows existentes][windows-containers].

## <a name="can-i-use-azure-dev-spaces-on-aks-clusters-with-api-server-authorized-ip-address-ranges-enabled"></a>¿Puedo usar Azure Dev Spaces en clústeres de AKS con intervalos de direcciones IP autorizados del servidor de API habilitado?

Sí, puede usar Azure Dev Spaces en clústeres de AKS con [intervalos de direcciones IP autorizados del servidor de API][aks-auth-range] habilitado. [Aquí](configure-networking.md#using-api-server-authorized-ip-ranges) puede encontrar más información sobre el uso de un clúster de AKS con intervalos de direcciones IP autorizadas del servidor de API habilitados con Azure Dev Spaces.

## <a name="can-i-use-azure-dev-spaces-on-aks-clusters-with-restricted-egress-traffic-for-cluster-nodes"></a>¿Puedo usar Azure Dev Spaces en clústeres de AKS con tráfico de salida restringido para los nodos de clúster?

Sí, puede usar Azure Dev Spaces en clústeres de AKS con el [tráfico de salida restringido para los nodos de clúster][aks-restrict-egress-traffic] habilitado una vez que se han permitido los FQDN correctos. [Aquí](configure-networking.md#ingress-and-egress-network-traffic-requirements) puede encontrar más información sobre el uso de un clúster de AKS con el tráfico de salida restringido para nodos de clúster habilitado con Azure Dev Spaces.

## <a name="can-i-use-azure-dev-spaces-on-rbac-enabled-aks-clusters"></a>¿Puedo usar Azure Dev Spaces en clústeres de AKS habilitados para RBAC?

Sí, puede usar Azure Dev Spaces en clústeres de AKS con o sin RBAC habilitado.

## <a name="what-happens-when-i-enable-ingress-for-project-in-visual-studio"></a>¿Qué ocurre cuando se habilita la entrada para el proyecto en Visual Studio?

Al usar Visual Studio para preparar el proyecto, tiene la opción de habilitar la entrada para su servicio. La habilitación de la entrada crea un punto de conexión público para acceder a su servicio cuando se ejecuta en el clúster de AKS, lo cual es opcional. Si no habilita la entrada, el servicio solo es accesible desde dentro del clúster de AKS.

## <a name="can-i-use-pod-managed-identities-with-azure-dev-spaces"></a>¿Puedo usar identidades administradas de pod con Azure Dev Spaces?

Sí, puede usar las [identidades administradas de pod][aks-pod-managed-id] en clústeres de AKS con Azure Dev Spaces habilitado, pero hay [pasos de configuración adicionales][dev-spaces-pod-managed-id-steps] después de habilitar Azure Dev Spaces en el clúster con identidades administradas de pod. Si tiene identidades administradas de pod instaladas y desea desinstalarlas, puede encontrar más detalles en las [notas de desinstalación][aks-pod-managed-id-uninstall].

## <a name="can-i-use-azure-dev-spaces-with-multiple-microservices-in-an-application"></a>¿Puedo usar Azure Dev Spaces con varios microservicios en una aplicación?

Sí, puede usar Azure Dev Spaces en una aplicación con varios microservicios, pero debe preparar y ejecutar los microservicios individuales en su raíz. La CLI de Azure Dev Spaces, la extensión de Azure Dev Spaces VS Code y la carga de trabajo de desarrollo de Visual Studio Azure esperan que el archivo *azds.yaml* esté en la raíz del microservicio para poder ejecutarse y depurarse. Consulte la [aplicación de ejemplo Bike Sharing][bike-sharing] para ver un ejemplo de varios microservicios en una sola aplicación.

En Visual Studio Code, es posible [abrir proyectos independientes en una sola área de trabajo][vs-code-multi-root-workspaces] y depurarlos por separado a través de Azure Dev Spaces. Cada uno de los proyectos debe ser independiente y estar preparado para Azure Dev Spaces.

En Visual Studio, es posible configurar soluciones de .NET Core para la depuración a través de Azure Dev Spaces.

## <a name="can-i-use-azure-dev-spaces-with-a-service-mesh"></a>¿Puedo usar Azure Dev Spaces con una red de servicios?

En este momento, no puede usar Azure Dev Spaces con una red de servicios como [Istio][istio] o [Linkerd][linkerd]. Puede ejecutar Azure Dev Spaces y una red de servicios en el mismo clúster de AKS, pero no puede tener Azure Dev Spaces y una red de servicios habilitados en el mismo espacio de nombres.

[aks-auth-range]: ../aks/api-server-authorized-ip-ranges.md
[aks-auth-range-create]: ../aks/api-server-authorized-ip-ranges.md#create-an-aks-cluster-with-api-server-authorized-ip-ranges-enabled
[aks-auth-range-update]: ../aks/api-server-authorized-ip-ranges.md#update-a-clusters-api-server-authorized-ip-ranges
[aks-migration]: ../aks/aks-migration.md
[aks-pod-managed-id]: ../aks/developer-best-practices-pod-security.md#use-pod-managed-identities
[aks-pod-managed-id-uninstall]: https://github.com/Azure/aad-pod-identity#uninstall-notes
[aks-restrict-egress-traffic]: ../aks/limit-egress-traffic.md
[aks-supported-k8s]: ../aks/supported-kubernetes-versions.md#azure-portal-and-cli-versions
[bike-sharing]: https://github.com/Azure/dev-spaces/tree/master/samples/BikeSharingApp
[dev-spaces-pod-managed-id-steps]: troubleshooting.md#error-no-azureassignedidentity-found-for-podazdsazds-webhook-deployment-id-in-assigned-state
[dev-spaces-prep]: how-dev-spaces-works-prep.md
[dev-spaces-routing]: how-dev-spaces-works-routing.md#how-routing-works
[ingress-nginx]: how-to/ingress-https-nginx.md#configure-a-custom-nginx-ingress-controller
[ingress-traefik]: how-to/ingress-https-traefik.md#configure-a-custom-traefik-ingress-controller
[ingress-https-nginx]: how-to/ingress-https-nginx.md#configure-the-nginx-ingress-controller-to-use-https
[ingress-https-traefik]: how-to/ingress-https-traefik.md#configure-the-traefik-ingress-controller-to-use-https
[istio]: https://istio.io/
[linkerd]: https://linkerd.io/
[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[vs-code-multi-root-workspaces]: https://code.visualstudio.com/docs/editor/multi-root-workspaces
[windows-containers]: how-to/run-dev-spaces-windows-containers.md