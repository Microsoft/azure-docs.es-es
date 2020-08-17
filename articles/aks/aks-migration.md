---
title: Migrar a Azure Kubernetes Service (AKS)
description: Migrar a Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 02/25/2020
ms.custom: mvc
ms.openlocfilehash: 9371feb527bbb2d94d43072bb8a44a6705b45055
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87280229"
---
# <a name="migrate-to-azure-kubernetes-service-aks"></a>Migrar a Azure Kubernetes Service (AKS)

Este artículo le ayuda a planear y ejecutar una migración correcta a Azure Kubernetes Service (AKS). Para ayudarle a tomar decisiones clave, en esta guía se proporcionan detalles sobre la configuración recomendada actual para AKS. En este artículo no se tratan todos los escenarios y, cuando sea necesario, se incluyen vínculos a información más detallada para planear una migración correcta.

Este documento se puede usar para ayudar con los siguientes escenarios:

* Migración de un clúster de AKS respaldado por [Conjuntos de disponibilidad](../virtual-machines/windows/tutorial-availability-sets.md) a [Conjuntos de escala de máquinas virtuales](../virtual-machine-scale-sets/overview.md)
* Migración de un clúster de AKS para usar un [equilibrador de carga de SKU estándar](./load-balancer-standard.md)
* Migración desde [Azure Container Service (ACS): que se retirará el 31 de enero de 2020](https://azure.microsoft.com/updates/azure-container-service-will-retire-on-january-31-2020/) a AKS
* Migración desde el [motor de AKS](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-1908) a AKS
* Migración de clústeres de Kubernetes no basados en Azure a AKS
* Traslado de recursos existentes a una región diferente

Al realizar la migración, asegúrese de que la versión de Kubernetes de destino se encuentre en la ventana admitida para AKS. Si usa una versión anterior, para que se admita en AKS es posible que no esté dentro del intervalo admitido y que sea necesario actualizar a versiones de actualización. Para más información, consulte [Versiones de Kubernetes compatibles en AKS](./supported-kubernetes-versions.md).

Si va a migrar a una versión más reciente de Kubernetes, consulte [Versión de Kubernetes y directiva de soporte de asimetría de versiones](https://kubernetes.io/docs/setup/release/version-skew-policy/#supported-versions).

Varias herramientas de código abierto pueden ayudarle con la migración, en función de su escenario:

* [Velero](https://velero.io/) (requiere Kubernetes 1.7+)
* [Extensión de la CLI de Azure Kube](https://github.com/yaron2/azure-kube-cli)
* [ReShifter](https://github.com/mhausenblas/reshifter)

En este artículo se resumen los detalles de la migración para lo siguiente:

> [!div class="checklist"]
> * AKS con Standard Load Balancer y Virtual Machine Scale Sets
> * Servicios de Azure asociados existentes
> * Garantizar cuotas válidas
> * Alta disponibilidad y continuidad empresarial
> * Consideraciones sobre las aplicaciones sin estado
> * Consideraciones sobre las aplicaciones con estado
> * Implementación de la configuración del clúster

## <a name="aks-with-standard-load-balancer-and-virtual-machine-scale-sets"></a>AKS con Standard Load Balancer y Virtual Machine Scale Sets

AKS es un servicio administrado que ofrece funcionalidades únicas con una sobrecarga de administración menor. Como resultado de ser un servicio administrado, debe seleccionar un conjunto de [regiones](./quotas-skus-regions.md) que admita AKS. La transición del clúster existente a AKS puede requerir la modificación de las aplicaciones existentes para que sigan siendo correctas en el plano de control administrado AKS.

Se recomienda usar clústeres de AKS respaldados por [Virtual Machine Scale Sets](../virtual-machine-scale-sets/index.yml) y [Azure Standard Load Balancer](./load-balancer-standard.md) para asegurarse de que se obtengan características como, por ejemplo, [varios grupos de nodos](./use-multiple-node-pools.md), [zona de disponibilidad](../availability-zones/az-overview.md), [intervalos de IP autorizados](./api-server-authorized-ip-ranges.md), [escalador automático de clúster](./cluster-autoscaler.md), [Azure Policy para AKS](../governance/policy/concepts/policy-for-kubernetes.md) y otras características nuevas a medida que se lancen.

Los clústeres de AKS respaldados por [Virtual Machine Availability Sets](../virtual-machines/availability.md#availability-sets) no tienen compatibilidad con muchas de estas características.

En el ejemplo siguiente se crea un clúster de AKS con un único grupo de nodos respaldado por un conjunto de escalado de máquinas virtuales. Usa una instancia de Standard Load Balancer. También habilita la escalabilidad automática del clúster en el grupo de nodos para el clúster y establece un mínimo de *1* y un máximo de *3* nodos:

```azurecli-interactive
# First create a resource group
az group create --name myResourceGroup --location eastus

# Now create the AKS cluster and enable the cluster autoscaler
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 1 \
  --vm-set-type VirtualMachineScaleSets \
  --load-balancer-sku standard \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 3
```

## <a name="existing-attached-azure-services"></a>Servicios de Azure asociados existentes

Al migrar clústeres, es posible que haya asociado servicios externos de Azure. Estos no requieren la recreación de recursos, pero requerirán la actualización de las conexiones de los clústeres anteriores a los nuevos para mantener la funcionalidad.

* Azure Container Registry
* Log Analytics
* Application Insights
* Traffic Manager
* Cuenta de almacenamiento
* Bases de datos externas

## <a name="ensure-valid-quotas"></a>Garantizar cuotas válidas

Dado que las máquinas virtuales adicionales se implementarán en su suscripción durante la migración, debe comprobar que los límites y cuotas son suficientes para estos recursos. Es posible que tenga que solicitar un aumento en la [cuota de vCPU](../azure-portal/supportability/per-vm-quota-requests.md).

Es posible que tenga que solicitar un aumento de las [cuotas de red](../azure-portal/supportability/networking-quota-requests.md) para asegurarse de no agotar las direcciones IP. Para más información, consulte [Redes e intervalos IP para AKS](./configure-kubenet.md).

Para más información, consulte [Azure subscription and service limits](../azure-resource-manager/management/azure-subscription-service-limits.md) (Límites de suscripción y servicio de Azure). Para comprobar las cuotas actuales, en Azure Portal, vaya a la [hoja de suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade), seleccione la suscripción y seleccione **Uso y cuotas**.

## <a name="high-availability-and-business-continuity"></a>Alta disponibilidad y continuidad empresarial

Si la aplicación no puede controlar el tiempo de inactividad, debe seguir los procedimientos recomendados para escenarios de migración de alta disponibilidad.  Los procedimientos recomendados para la planeación de la continuidad empresarial compleja, la recuperación ante desastres y la maximización del tiempo de actividad quedan fuera del ámbito de este documento.  Para más información, consulte [Procedimientos recomendados para continuidad empresarial y recuperación ante desastres en Azure Kubernetes Service (AKS)](./operator-best-practices-multi-region.md).

En aplicaciones complejas se suele migrar a lo largo del tiempo y no de una vez. Es decir, puede que los entornos nuevos y antiguos tengan que comunicarse a través de la red. Es posible que aplicaciones que anteriormente usaban servicios `ClusterIP` para comunicarse deban exponerse como tipo `LoadBalancer` y estar protegidas adecuadamente.

Para completar la migración, querrá dirigir los clientes a los nuevos servicios que se ejecutan en AKS. Recomendamos redirigir el tráfico mediante la actualización de DNS para que apunte a la instancia de Load Balancer que se encuentra frente al clúster de AKS.

[Azure Traffic Manager](../traffic-manager/index.yml) puede dirigir a los clientes a la instancia de la aplicación y los clústeres de Kubernetes deseados.  Azure Traffic Manager es un equilibrador de carga de tráfico basado en DNS que puede distribuir el tráfico de red entre regiones.  Para conseguir los máximos niveles de rendimiento y redundancia, dirija todo el tráfico de la aplicación con Traffic Manager antes de que pase al clúster de AKS.  En una implementación de varios clústeres, los clientes deben conectarse a un nombre DNS de Traffic Manager que apunte a los servicios en cada clúster de AKS. Para definir estos servicios, use los puntos de conexión de Traffic Manager. Cada punto de conexión es la *dirección IP del equilibrador de carga del servicio*. Use esta configuración para dirigir el tráfico de red desde el punto de conexión de Traffic Manager de una región al punto de conexión de otra.

![AKS con Traffic Manager](media/operator-best-practices-bc-dr/aks-azure-traffic-manager.png)

[Azure Front Door Service](../frontdoor/front-door-overview.md) es otra opción para enrutar el tráfico de los clústeres de AKS.  Azure Front Door Service permite definir, administrar y supervisar el enrutamiento global para el tráfico web mediante la optimización para obtener el mejor rendimiento y la conmutación por error global instantánea para alta disponibilidad. 

### <a name="considerations-for-stateless-applications"></a>Consideraciones sobre las aplicaciones sin estado

La migración de aplicaciones sin estado es el caso más sencillo. Aplique las definiciones de recurso (YAML o Helm) al nuevo clúster, asegúrese de que todo funciona según lo previsto y redirija el tráfico para activar el nuevo clúster.

### <a name="considerations-for-stateful-applications"></a>Consideraciones sobre las aplicaciones con estado

Planee meticulosamente la migración de aplicaciones con estado para evitar la pérdida de datos o tiempos de inactividad inesperados.

Si usa Azure Files, puede montar el recurso compartido de archivos como un volumen en el nuevo clúster:
* [Montar instancias estáticas de Azure Files como volumen](./azure-files-volume.md#mount-the-file-share-as-a-volume)

Si usa Azure Managed Disks, solo podrá montar el disco si no está conectado a ninguna VM:
* [Montar Static Azure Disk como volumen](./azure-disk-volume.md#mount-disk-as-volume)

Si ninguno de estos métodos funciona, puede usar las opciones de copia de seguridad y restauración:
* [Velero en Azure](https://github.com/vmware-tanzu/velero-plugin-for-microsoft-azure/blob/master/README.md)

#### <a name="azure-files"></a>Archivos de Azure

A diferencia de los discos, Azure Files puede montarse simultáneamente en varios hosts. En el clúster de AKS, Azure y Kubernetes no le impiden que cree un pod que el clúster de ACS siga usando. Para evitar la pérdida de datos y un comportamiento inesperado, asegúrese de que ambos clústeres no escriben en los mismos archivos al mismo tiempo.

Si la aplicación puede hospedar varias réplicas que apunten al mismo recurso compartido de archivos, siga los pasos de la migración sin estado e implemente las definiciones de YAML en el nuevo clúster. Si no es así, un posible enfoque de migración implica los pasos siguientes:

* Compruebe que la aplicación funciona correctamente.
* Apunte el tráfico en directo al nuevo clúster de AKS.
* Desconecte el clúster anterior.

Si quiere comenzar con un recurso compartido vacío y luego realizar una copia de los datos de origen, puede usar los comandos [`az storage file copy`](/cli/azure/storage/file/copy?view=azure-cli-latest) para migrar los datos.


#### <a name="migrating-persistent-volumes"></a>Migración de volúmenes persistentes

Si migra volúmenes persistentes existentes a AKS, generalmente seguirá estos pasos:

* Desactivar las escrituras en la aplicación. (Este paso es opcional y requiere tiempo de inactividad).
* Tomar instantáneas de los discos.
* Crear discos administrados nuevos a partir de las instantáneas.
* Crear volúmenes persistentes en AKS.
* Actualizar las especificaciones de pod para [usar volúmenes existentes](./azure-disk-volume.md) en lugar de PersistentVolumeClaims (aprovisionamiento estático).
* Implemente la aplicación en AKS.
* Compruebe que la aplicación funciona correctamente.
* Apunte el tráfico en directo al nuevo clúster de AKS.

> [!IMPORTANT]
> Si decide no desactivar las escrituras, deberá replicar los datos en la nueva implementación. En caso contrario, perderá los datos que se escriban después de tomar las instantáneas de disco.

Algunas herramientas de código abierto pueden ayudarle a crear discos administrados y a migrar volúmenes entre clústeres de Kubernetes:

* La [extensión de copia de disco de la CLI de Azure](https://github.com/noelbundick/azure-cli-disk-copy-extension) copia y convierte los discos entre grupos de recursos y regiones de Azure.
* La [extensión de la CLI de Azure Kube](https://github.com/yaron2/azure-kube-cli) enumera los volúmenes de ACS Kubernetes y los migra a un clúster de AKS.


### <a name="deployment-of-your-cluster-configuration"></a>Implementación de la configuración del clúster

Se recomienda usar la canalización existente de integración continua (CI) y entrega continua (CD) para implementar una configuración válida conocida en AKS. Puede usar Azure Pipelines para [compilar e implementar las aplicaciones en AKS](/azure/devops/pipelines/ecosystems/kubernetes/aks-template?view=azure-devops). Clone las tareas de implementación existentes y asegúrese de que `kubeconfig` apunta al nuevo clúster de AKS.

Si no es posible, exporte las definiciones de recursos desde el clúster de Kubernetes existente y luego aplíquelas a AKS. Puede usar `kubectl` para exportar objetos.

```console
kubectl get deployment -o=yaml --export > deployments.yaml
```

### <a name="moving-existing-resources-to-another-region"></a>Movimiento de recursos existentes a otra región

Puede que quiera trasladar el clúster de AKS a [otra región compatible con AKS][region-availability]. Se recomienda crear un nuevo clúster en la otra región y, a continuación, implementar los recursos y las aplicaciones en el nuevo clúster. Además, si tiene servicios como [Azure Dev Spaces][azure-dev-spaces] que se ejecutan en el clúster de AKS, también deberá instalar y configurar esos servicios en el clúster en la nueva región.


En este artículo se proporciona un resumen de los detalles de la migración para:

> [!div class="checklist"]
> * AKS con Standard Load Balancer y Virtual Machine Scale Sets
> * Servicios de Azure asociados existentes
> * Garantizar cuotas válidas
> * Alta disponibilidad y continuidad empresarial
> * Consideraciones sobre las aplicaciones sin estado
> * Consideraciones sobre las aplicaciones con estado
> * Implementación de la configuración del clúster


[region-availability]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[azure-dev-spaces]: ../dev-spaces/index.yml
