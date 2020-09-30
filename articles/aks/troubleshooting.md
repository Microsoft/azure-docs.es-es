---
title: Solucionar problemas comunes de Azure Kubernetes Service
description: Obtenga información sobre cómo solucionar problemas y resolver problemas comunes al usar Azure Kubernetes Service (AKS)
services: container-service
ms.topic: troubleshooting
ms.date: 06/20/2020
ms.openlocfilehash: 855e5e5e23371f600a7e73139f2e6da1eebc91d0
ms.sourcegitcommit: 1fe5127fb5c3f43761f479078251242ae5688386
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/14/2020
ms.locfileid: "90068836"
---
# <a name="aks-troubleshooting"></a>Solución de problemas de AKS

Al crear o administrar clústeres de Azure Kubernetes Service (AKS), en ocasiones pueden surgir problemas. En este artículo se detallan algunos problemas comunes y los pasos para solucionarlos.

## <a name="in-general-where-do-i-find-information-about-debugging-kubernetes-problems"></a>En general, ¿dónde puedo encontrar información sobre la depuración de problemas de Kubernetes?

Pruebe la [guía oficial para la solución de problemas de clústeres de Kubernetes](https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/).
Además, existe una [guía para la solución de problemas](https://github.com/feiskyer/kubernetes-handbook/blob/master/en/troubleshooting/index.md) publicada por un ingeniero de Microsoft en torno a la solución de problemas de pods, nodos, clústeres y otras características.

## <a name="im-getting-a-quota-exceeded-error-during-creation-or-upgrade-what-should-i-do"></a>Obtengo un error de "cuota excedida" durante la creación o actualización. ¿Cuál debo hacer? 

 [Solicitar más núcleos](../azure-portal/supportability/resource-manager-core-quotas-request.md).

## <a name="what-is-the-maximum-pods-per-node-setting-for-aks"></a>¿Cuál es la configuración máxima de pods por nodo de AKS?

La configuración máxima de pods por nodo es 30 de forma predeterminada si implementa un clúster de AKS en Azure Portal.
La configuración máxima de pods por nodo es 110 de forma predeterminada si implementa un clúster de AKS en la CLI de Azure. (Asegúrese de usar la versión más reciente de la CLI de Azure). Se puede cambiar esta configuración mediante la marca `–-max-pods` en el comando `az aks create`.

## <a name="im-getting-an-insufficientsubnetsize-error-while-deploying-an-aks-cluster-with-advanced-networking-what-should-i-do"></a>Recibo el error insufficientSubnetSize al implementar un clúster de AKS con redes avanzadas. ¿Cuál debo hacer?

Este error indica que una subred en uso para un clúster ya no tiene direcciones IP disponibles dentro de su CIDR para una correcta asignación de recursos. En el caso de los clústeres de Kubenet, el requisito es un espacio de IP suficiente para cada nodo del clúster. En el caso de los clústeres de Azure CNI, el requisito es un espacio de IP suficiente para cada nodo y pod del clúster.
Obtenga más información sobre el [diseño de Azure CNI para asignar direcciones IP a pods](configure-azure-cni.md#plan-ip-addressing-for-your-cluster).

Estos errores también se muestran en [Diagnósticos de AKS](./concepts-diagnostics.md), que muestra problemas de manera proactiva, como un tamaño de subred insuficiente.

Los siguientes tres (3) casos provocan un error de tamaño de subred insuficiente:

1. Escala de AKS o escala de AKS Nodepool
   1. Si usa Kubenet, esto sucede cuando el `number of free IPs in the subnet` es **menor que** el `number of new nodes requested`.
   1. Si usa Azure CNI, esto sucede cuando el `number of free IPs in the subnet` es **menor que** el `number of nodes requested times (*) the node pool's --max-pod value`.

1. Actualización de AKS o actualización de AKS Nodepool
   1. Si usa Kubenet, esto sucede cuando el `number of free IPs in the subnet` es **menor que** el `number of buffer nodes needed to upgrade`.
   1. Si usa Azure CNI, esto sucede cuando el `number of free IPs in the subnet` es **menor que** el `number of buffer nodes needed to upgrade times (*) the node pool's --max-pod value`.
   
   De forma predeterminada, los clústeres de AKS establecen un valor de sobrecarga máxima (búfer de actualización) de uno (1), pero este comportamiento de actualización se puede personalizar si se establece el valor de [sobrecarga máxima de un grupo de nodos](upgrade-cluster.md#customize-node-surge-upgrade-preview) que aumentará el número de direcciones IP disponibles necesarias para completar una actualización.

1. Creación de AKS o adición de AKS Nodepool
   1. Si usa Kubenet, esto sucede cuando el `number of free IPs in the subnet` es **menor que** el `number of nodes requested for the node pool`.
   1. Si usa Azure CNI, esto sucede cuando el `number of free IPs in the subnet` es **menor que** el `number of nodes requested times (*) the node pool's --max-pod value`.

La siguiente mitigación se puede llevar a cabo mediante la creación de nuevas subredes. El permiso para crear una nueva subred es necesario para la mitigación debido a la incapacidad de actualizar el rango de CIDR de una subred existente.

1. Vuelva a generar una nueva subred con un rango de CIDR mayor que sea suficiente para los objetivos de la operación:
   1. Cree una nueva subred con un nuevo rango deseado que no se superponga.
   1. Cree un nuevo grupo de nodos en la nueva subred.
   1. Vacíe los pods del grupo de nodos antiguo que reside en la subred antigua que se va a reemplazar.
   1. Elimine la antigua subred y el antiguo grupo de nodos.

## <a name="my-pod-is-stuck-in-crashloopbackoff-mode-what-should-i-do"></a>Mi pod se ha atascado en el modo CrashLoopBackOff. ¿Cuál debo hacer?

Puede haber varios motivos para que el pod se quede atascado en ese modo. Algunas soluciones posibles son:

* El propio pod, mediante el comando `kubectl describe pod <pod-name>`.
* Los registros, mediante el uso de `kubectl logs <pod-name>`.

Para obtener más información sobre cómo solucionar problemas de los pods, consulte cómo [depurar aplicaciones](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/#debugging-pods).

## <a name="im-receiving-tcp-timeouts-when-using-kubectl-or-other-third-party-tools-connecting-to-the-api-server"></a>Recibo `TCP timeouts` al usar `kubectl` u otras herramientas de terceros que se conectan al servidor de API
AKS tiene planos de control de alta disponibilidad que se escalan verticalmente según el número de núcleos para garantizar los objetivos de nivel de servicio (SLO) y los acuerdos de nivel de servicio (SLA). Si se agota el tiempo de espera de las conexiones, compruebe lo siguiente:

- **¿Se agota el tiempo de espera de todos los comandos de la API de manera uniforme o solo unos pocos?** Si solo sucede con unos pocos, el pod `tunnelfront` o el pod `aks-link`, responsables de la comunicación del nodo al plano de control, puede que no estén en estado de ejecución. Asegúrese de que los nodos que hospeden este pod no se usen de manera excesiva ni están bajo estrés. Piense en la posibilidad de moverlos a su propio [grupo de nodos del `system`](use-system-pools.md).
- **¿Ha abierto todos los puertos, FQDN y direcciones IP necesarios indicados en la [documentación de control del tráfico de salida de AKS](limit-egress-traffic.md)?** De lo contrario, se pueden producir errores en varias llamadas a comandos.
- **¿La dirección IP actual está incluida en los [intervalos de IP autorizados de la API](api-server-authorized-ip-ranges.md)?** Si utiliza esta característica y la dirección IP no está incluida en los intervalos, se bloquearán las llamadas. 
- **¿Tiene un cliente o aplicación con fuga de llamadas al servidor de la API?** Asegúrese de usar inspecciones en lugar de llamadas get frecuentes, y de que las aplicaciones de terceros no estén perdiendo dichas llamadas. Por ejemplo, un error en Mixer de Istio hace que se cree una nueva conexión de inspección del servidor de la API cada vez que un secreto se lee internamente. Dado que este comportamiento se produce a intervalos regulares, las conexiones de inspección se acumulan rápidamente y, finalmente, hacen que el servidor de la API se sobrecargue sin importar el patrón de escalado. https://github.com/istio/istio/issues/19481
- **¿Tiene muchas versiones en las implementaciones de Helm?** En este escenario se puede hacer que Tiller use demasiada memoria en los nodos, así como una gran cantidad de `configmaps`, lo que puede provocar picos innecesarios en el servidor de la API. Considere la posibilidad de configurar `--history-max` en `helm init` y aprovechar el nuevo Helm 3. Más información sobre los temas siguientes: 
    - https://github.com/helm/helm/issues/4821
    - https://github.com/helm/helm/issues/3500
    - https://github.com/helm/helm/issues/4543
- **[¿Se está bloqueando el tráfico interno entre los nodos?](#im-receiving-tcp-timeouts-such-as-dial-tcp-node_ip10250-io-timeout)**

## <a name="im-receiving-tcp-timeouts-such-as-dial-tcp-node_ip10250-io-timeout"></a>Estoy recibiendo `TCP timeouts`, como `dial tcp <Node_IP>:10250: i/o timeout`

Estos tiempos de espera pueden estar relacionados con el bloqueo del tráfico interno entre los nodos. Compruebe que no se está bloqueando este tráfico, por ejemplo mediante [grupos de seguridad de red](concepts-security.md#azure-network-security-groups) en la subred de los nodos del clúster.

## <a name="im-trying-to-enable-role-based-access-control-rbac-on-an-existing-cluster-how-can-i-do-that"></a>Estoy intentando habilitar el control de acceso basado en rol (RBAC) en un clúster existente. ¿Cómo puedo hacerlo?

En este momento no se admite la habilitación del control de acceso basado en rol (RBAC) en clústeres existentes; se debe configurar al crear clústeres. RBAC está habilitado de forma predeterminada cuando se usa la CLI, el portal o una versión de API posterior a la `2020-03-01`.

## <a name="i-created-a-cluster-with-rbac-enabled-and-now-i-see-many-warnings-on-the-kubernetes-dashboard-the-dashboard-used-to-work-without-any-warnings-what-should-i-do"></a>He creado un clúster con RBAC habilitado y ahora veo muchas advertencias en el panel de Kubernetes. El panel solía funcionar sin mostrar ninguna advertencia. ¿Cuál debo hacer?

El motivo de las advertencias es que el clúster tiene habilitado RBAC y el acceso al panel ahora está restringido de forma predeterminada. En general, este enfoque es recomendable, puesto que la exposición predeterminada del panel a todos los usuarios del clúster puede dar lugar a amenazas de seguridad. Si desea habilitar el panel, siga los pasos descritos en esta [entrada de blog](https://pascalnaber.wordpress.com/2018/06/17/access-dashboard-on-aks-with-rbac-enabled/).

## <a name="i-cant-get-logs-by-using-kubectl-logs-or-i-cant-connect-to-the-api-server-im-getting-error-from-server-error-dialing-backend-dial-tcp-what-should-i-do"></a>No logro obtener los registros mediante kubectl logs ni puedo conectarme al servidor de API. Recibo el mensaje "Error de servidor: error de marcado al back-end: marcar tcp…" ¿Cuál debo hacer?

Asegúrese de que los puertos 22, 9000 y 1194 están abiertos para conectarse al servidor de la API. Compruebe si el pod `tunnelfront` o `aks-link` se ejecutan en el espacio de nombres *kube-system* mediante el comando `kubectl get pods --namespace kube-system`. Si no se está ejecutando, debe forzar la eliminación del pod y se reiniciará.

## <a name="im-getting-tls-client-offered-only-unsupported-versions-from-my-client-when-connecting-to-aks-api-what-should-i-do"></a>Recibo `"tls: client offered only unsupported versions"` de mi cliente al conectarse a la API de AKS.   ¿Qué debo hacer?

La versión de TLS mínima admitida en AKS es TLS 1.2.

## <a name="im-trying-to-upgrade-or-scale-and-am-getting-a-changing-property-imagereference-is-not-allowed-error-how-do-i-fix-this-problem"></a>Estoy intentando actualizar o escalar y recibo el error `"Changing property 'imageReference' is not allowed"`. ¿Cómo se corrige este problema?

Es posible que reciba este error porque se han modificado las etiquetas de los nodos de agente dentro del clúster de AKS. Modificar y eliminar etiquetas y otras propiedades de recursos en el grupo de recursos MC_* puede provocar resultados inesperados. La modificación de los recursos en el grupo MC_* en el clúster de AKS invalida el objetivo de nivel de servicio (SLO).

## <a name="im-receiving-errors-that-my-cluster-is-in-failed-state-and-upgrading-or-scaling-will-not-work-until-it-is-fixed"></a>Recibo errores que indican que mi clúster presenta errores y que la actualización o el escalado no funcionarán hasta que se corrija.

*Esta ayuda para solucionar problemas se dirige desde https://aka.ms/aks-cluster-failed*

Este error aparece cuando los clústeres entran en un estado con errores por diversos motivos. Siga los pasos que aparecen a continuación para solucionar el estado con errores del clúster antes de reintentar la operación con errores anterior:

1. Hasta que el clúster deje de tener el estado `failed`, las operaciones `upgrade` y `scale` no se realizarán correctamente. Los problemas y resoluciones raíz comunes incluyen:
    * Escalado con una **cuota de proceso insuficiente (CRP)** . Para solucionarlo, primero debe escalar el clúster de vuelta a un estado objetivo estable dentro de la cuota. Luego, siga estos [pasos para solicitar un aumento de cuota de proceso](../azure-portal/supportability/resource-manager-core-quotas-request.md) antes de intentar volver a escalar verticalmente más allá de los límites de cuota iniciales.
    * Escalar un clúster con redes avanzadas y **recursos de subred (redes) insuficientes**. Para solucionarlo, primero debe escalar el clúster de vuelta a un estado objetivo estable dentro de la cuota. Luego, siga estos [pasos para solicitar un aumento de cuota de recursos](../azure-resource-manager/templates/error-resource-quota.md#solution) antes de intentar volver a escalar verticalmente más allá de los límites de cuota iniciales.
2. Una vez que se resuelve la causa subyacente de un error de actualización, el clúster debería tener un estado correcto. Una vez que compruebe el estado correcto, vuelva a intentar la operación original.

## <a name="im-receiving-errors-when-trying-to-upgrade-or-scale-that-state-my-cluster-is-being-upgraded-or-has-failed-upgrade"></a>Recibo errores cuando intento actualizar o escalar que indican que el clúster se está actualizando o que hay un error de actualización.

*Esta ayuda para solucionar problemas se dirige desde https://aka.ms/aks-pending-upgrade*

 No puede tener un grupo de clústeres o de nodos actualizados y escalados a la vez. En su lugar, cada tipo de operación debe completarse en el recurso de destino antes de la siguiente solicitud en ese mismo recurso. Como resultado, las operaciones se ven limitadas cuando tienen lugar o se han intentado operaciones de actualización o de escala. 

Para ayudar a diagnosticar el problema, ejecute `az aks show -g myResourceGroup -n myAKSCluster -o table` para recuperar el estado detallado del clúster. En función del resultado:

* Si el clúster se está actualizando, espere hasta que termine la operación. Si se realizó correctamente, vuelva a intentar realizar la operación que presentó errores.
* Si el clúster no se pudo actualizar, siga los pasos descritos en la sección anterior.

## <a name="can-i-move-my-cluster-to-a-different-subscription-or-my-subscription-with-my-cluster-to-a-new-tenant"></a>¿Puedo mover el clúster a otra suscripción o mi suscripción con mi clúster a un inquilino nuevo?

Si ha migrado el clúster de AKS a otra suscripción o la suscripción del clúster a un nuevo inquilino, el clúster no funcionará porque faltan permisos de identidad del clúster. Debido a esta restricción, **AKS no admite la migración de clústeres entre suscripciones o inquilinos**.

## <a name="im-receiving-errors-trying-to-use-features-that-require-virtual-machine-scale-sets"></a>Recibos errores cuando intento usar características que requieren conjuntos de escalado de máquinas virtuales.

*Esta ayuda para solucionar problemas se dirige desde aka.ms/aks-vmss-enablement*

Puede recibir errores que indiquen que el clúster de AKS no se encuentra en un conjunto de escalado de máquinas virtuales, como en el ejemplo siguiente:

El grupo de agentes **agentpool`<agentpoolname>` definió el escalado automático como habilitado, pero no se encuentra en Virtual Machine Scale Sets**.

Para usar características como el escalador automático de clústeres o varios grupos de nodos, el valor de `vm-set-type` deben ser los conjuntos de escalado de máquinas virtuales.

Siga los pasos de *Antes de empezar* del documento adecuado para crear correctamente un clúster de AKS:

* [Uso del escalador automático del clúster](cluster-autoscaler.md).
* [Creación y uso de varios grupos de nodos](use-multiple-node-pools.md).
 
## <a name="what-naming-restrictions-are-enforced-for-aks-resources-and-parameters"></a>¿Qué restricciones de nomenclatura se aplican para los parámetros y recursos de AKS?

*Esta ayuda para solucionar problemas se dirige desde aka.ms/aks-naming-rules*

Tanto AKS como la plataforma de Azure implementan las restricciones de nomenclatura. Si un parámetro o nombre de recurso infringe una de estas restricciones, se devuelve un error en el que se le pide brindar otra entrada. Se aplican estas directrices de nomenclatura comunes:

* Los nombres de clúster deben tener entre 1 y 63 caracteres. Los únicos caracteres permitidos son letras, números, guiones y guiones bajos. El primer y el último carácter deben ser una letra o un número.
* El nombre del grupo de recursos *MC_* o del nodo de AKS combina el nombre del grupo de recursos y el nombre del recurso. La sintaxis generada automáticamente de `MC_resourceGroupName_resourceName_AzureRegion` no puede tener más de 80 caracteres. Si es necesario, disminuya la longitud del nombre del grupo de recursos o del nombre del clúster de AKS. También puede [personalizar el nombre del grupo de recursos del nodo](cluster-configuration.md#custom-resource-group-name).
* *dnsPrefix* debe empezar y terminar con valores alfanuméricos, y debe tener entre 1 y 54 caracteres. Los caracteres válidos incluyen valores alfanuméricos y guiones (-). *dnsPrefix* no puede incluir caracteres especiales, como un punto (.).
* Los nombres de grupo del nodo de AKS deben estar en minúsculas y tener entre 1 y 11 caracteres, en el caso de grupos de nodos de Linux, y entre 1 y 6 caracteres si son grupos de nodos de Windows. El nombre debe empezar por una letra y los únicos caracteres permitidos son letras y números.
* El *admin-username*, que establece el nombre de usuario de administrador para los nodos de Linux, debe empezar con una letra, solo puede contener letras, números, guiones y caracteres de subrayado, y tiene una longitud máxima de 64 caracteres.

## <a name="im-receiving-errors-when-trying-to-create-update-scale-delete-or-upgrade-cluster-that-operation-is-not-allowed-as-another-operation-is-in-progress"></a>Recibo errores cuando intento crear, actualizar, escalar, eliminar o actualizar un clúster, donde se indica que no se permite la operación porque hay otra en curso.

*Esta ayuda para solucionar problemas se dirige desde aka.ms/aks-pending-operation*

Las operaciones del clúster se limitan cuando todavía hay en curso una operación anterior. Para recuperar un estado detallado del clúster, use el comando `az aks show -g myResourceGroup -n myAKSCluster -o table`. Use el nombre de su propio clúster de AKS y de su propio grupo de recursos, en caso de ser necesario.

En función de la salida del estado del clúster:

* Si el clúster está en cualquier estado de aprovisionamiento distinto de *Succeeded* (Correcto) o *Failed* (Con errores), espere a que se termine la operación (*actualización/creación/escalado/eliminación/migración*). Cuando termine la operación anterior, vuelva a intentar la operación más reciente del clúster.

* Si el clúster presenta errores de actualización, siga los pasos que se describen en [Recibo errores que indican que mi clúster presenta errores y que la actualización o el escalado no funcionarán hasta que se corrija](#im-receiving-errors-that-my-cluster-is-in-failed-state-and-upgrading-or-scaling-will-not-work-until-it-is-fixed).

## <a name="received-an-error-saying-my-service-principal-wasnt-found-or-is-invalid-when-i-try-to-create-a-new-cluster"></a>Al intentar crear un clúster, ha recibido un error que indica que la entidad de servicio no se encontró o no es válida.

Al crear un clúster de AKS, se requiere una entidad de servicio o una identidad administrada para crear recursos en su nombre. AKS puede crear automáticamente una entidad de servicio en el momento de la creación del clúster o recibir una existente. Cuando se usa una creada automáticamente, Azure Active Directory tiene que propagarla a todas las regiones para que la creación se realice correctamente. Si esta propagación tarda demasiado, se producirá un error al validar la creación del clúster, ya que no puede encontrar ninguna entidad de servicio disponible para hacerlo. 

Para resolver este problema, use las siguientes soluciones alternativas:
* Use una entidad de servicio existente que ya se haya propagado entre distintas regiones para pasarla a AKS en el momento de la creación del clúster.
* Si se usan scripts de automatización, agregue demoras entre la creación de la entidad de servicio y la creación del clúster de AKS.
* Si se usa Azure Portal, vuelva a la configuración del clúster durante la creación y reintente la página de validación pocos minutos después.

## <a name="im-getting-aadsts7000215-invalid-client-secret-is-provided-when-using-aks-api-what-should-i-do"></a>Recibo `"AADSTS7000215: Invalid client secret is provided."` al usar la API de AKS.   ¿Qué debo hacer?

Esto suele deberse a la expiración de las credenciales de la entidad de servicio. [Actualice las credenciales de un clúster de AKS.](update-credentials.md)

## <a name="im-receiving-errors-after-restricting-egress-traffic"></a>Recibo errores después de restringir el tráfico de salida

Cuando se restringe el tráfico de salida desde un clúster de AKS, hay puerto de salida / reglas de red y reglas de aplicación /FQDN [requeridos y recomendados opcionales](limit-egress-traffic.md) para AKS. Si la configuración entra en conflicto con alguna de estas reglas, es posible que algunos comandos `kubectl` no funcionen correctamente. También puede ver errores al crear un clúster de AKS.

Compruebe que la configuración no esté en conflicto con ninguno de los puertos de salida, reglas de red ni reglas de aplicación o FQDN requeridos o recomendados opcionales.

## <a name="azure-storage-and-aks-troubleshooting"></a>Solución de problemas de Azure Storage y AKS

### <a name="what-are-the-recommended-stable-versions-of-kubernetes-for-azure-disk"></a>¿Cuáles son las versiones estables recomendadas de Kubernetes para Azure Disk? 

| Versión de Kubernetes | Versión recomendada |
|--|:--:|
| 1.12 | 1.12.9 o posterior |
| 1.13 | 1.13.6 o posterior |
| 1,14 | 1.14.2 o posterior |


### <a name="waitforattach-failed-for-azure-disk-parsing-devdiskazurescsi1lun1-invalid-syntax"></a>Error de WaitForAttach para Azure Disk: análisis de "/dev/disk/azure/scsi1/lun1": sintaxis no válida

En la versión 1.10 de Kubernetes, MountVolume.WaitForAttach puede generar un error al volver a montar un disco de Azure.

En Linux, es posible que vea un error de formato de DevicePath incorrecto. Por ejemplo:

```console
MountVolume.WaitForAttach failed for volume "pvc-f1562ecb-3e5f-11e8-ab6b-000d3af9f967" : azureDisk - Wait for attach expect device path as a lun number, instead got: /dev/disk/azure/scsi1/lun1 (strconv.Atoi: parsing "/dev/disk/azure/scsi1/lun1": invalid syntax)
  Warning  FailedMount             1m (x10 over 21m)   kubelet, k8s-agentpool-66825246-0  Unable to mount volumes for pod
```

En Windows, es posible que vea un error de número de DevicePath(LUN) incorrecto. Por ejemplo:

```console
Warning  FailedMount             1m    kubelet, 15282k8s9010    MountVolume.WaitForAttach failed for volume "disk01" : azureDisk - WaitForAttach failed within timeout node (15282k8s9010) diskId:(andy-mghyb
1102-dynamic-pvc-6c526c51-4a18-11e8-ab5c-000d3af7b38e) lun:(4)
```

Este problema se corrigió en las versiones siguientes de Kubernetes:

| Versión de Kubernetes | Versión corregida |
|--|:--:|
| 1.10 | 1.10.2 o posterior |
| 1.11 | 1.11.0 o posterior |
| 1.12 y posterior | N/D |


### <a name="failure-when-setting-uid-and-gid-in-mountoptions-for-azure-disk"></a>Error al establecer el valor de UID y GID en mountOptions para Azure Disk

De manera predeterminada, Azure Disk usa el sistema de archivos ext4,xfs y mountOptions como uid=x,gid=x no se pueden establecer en el momento del montaje. Por ejemplo, si intenta establecer mountOptions como uid=999,gid=999, vería un error como:

```console
Warning  FailedMount             63s                  kubelet, aks-nodepool1-29460110-0  MountVolume.MountDevice failed for volume "pvc-d783d0e4-85a1-11e9-8a90-369885447933" : azureDisk - mountDevice:FormatAndMount failed with mount failed: exit status 32
Mounting command: systemd-run
Mounting arguments: --description=Kubernetes transient mount for /var/lib/kubelet/plugins/kubernetes.io/azure-disk/mounts/m436970985 --scope -- mount -t xfs -o dir_mode=0777,file_mode=0777,uid=1000,gid=1000,defaults /dev/disk/azure/scsi1/lun2 /var/lib/kubelet/plugins/kubernetes.io/azure-disk/mounts/m436970985
Output: Running scope as unit run-rb21966413ab449b3a242ae9b0fbc9398.scope.
mount: wrong fs type, bad option, bad superblock on /dev/sde,
       missing codepage or helper program, or other error
```

Para mitigar el problema, haga una de las acciones siguientes:

* [Configure el contexto de seguridad para un pod](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) al establecer UID en runAsUser y GID in fsGroup. Por ejemplo, la configuración siguiente establecerá la ejecución de pod como raíz, con lo que será accesible a cualquier archivo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 0
    fsGroup: 0
```

  >[!NOTE]
  > Como GID y UID se montan como raíz o 0 de manera predeterminada. Si GID o UID se establecen como no raíz, por ejemplo 1000, Kubernetes usará `chown` para cambiar todos los directorios y archivos de ese disco. Esta operación puede llevar mucho tiempo y puede hacer que montar el disco sea muy lento.

* Use `chown` en initContainers para establecer GID y UID. Por ejemplo:

```yaml
initContainers:
- name: volume-mount
  image: busybox
  command: ["sh", "-c", "chown -R 100:100 /data"]
  volumeMounts:
  - name: <your data volume>
    mountPath: /data
```

### <a name="azure-disk-detach-failure-leading-to-potential-race-condition-issue-and-invalid-data-disk-list"></a>Error de desasociación de un disco de Azure Disk que lleva a un posible problema de condición de carrera y a una lista de discos de datos no válida

Cuando se produce un error en la desasociación de un disco de Azure Disk, se reintentará hasta seis veces para desasociar el disco mediante el retroceso exponencial. También retendrá un bloqueo de nivel de nodo en la lista de discos de datos durante unos 3 minutos. Si la lista de discos se actualiza manualmente durante ese período, la lista de discos que retiene el bloqueo de nivel de nodo quedará obsoleta y provocará inestabilidad en el nodo.

Este problema se corrigió en las versiones siguientes de Kubernetes:

| Versión de Kubernetes | Versión corregida |
|--|:--:|
| 1.12 | 1.12.9 o posterior |
| 1.13 | 1.13.6 o posterior |
| 1,14 | 1.14.2 o posterior |
| 1.15 y posteriores | N/D |

Si usa una versión de Kubernetes que no tiene la solución para este problema y el nodo tiene una lista de discos obsoletos, para resolverlo, desasocie todos los discos no existentes de la máquina virtual en una operación masiva. **Es posible que se produzca un error al desasociar manualmente los discos no existentes.**

### <a name="large-number-of-azure-disks-causes-slow-attachdetach"></a>Un gran número de discos de Azure provoca asociaciones/desasociaciones lentas

Cuando el número de operaciones de asociación y desasociación de discos de Azure que tienen como destino una máquina virtual de un solo nodo es mayor que 10 o que 3 cuando se dirigen a un único grupo de conjuntos de escalado de máquinas virtuales, pueden ser más lentas de lo esperado, ya que se realizan de forma secuencial. Este problema es una limitación conocida y no hay ninguna solución alternativa en este momento. [Elemento de User Voice para admitir la asociación o desasociación paralelas más allá de un número.](https://feedback.azure.com/forums/216843-virtual-machines/suggestions/40444528-vmss-support-for-parallel-disk-attach-detach-for).

### <a name="azure-disk-detach-failure-leading-to-potential-node-vm-in-failed-state"></a>Error de desasociación de disco de Azure que lleva a una posible máquina virtual de nodo en estado erróneo

En algunos casos, es posible que se produzca un error en la desasociación de discos de Azure y que deje la máquina virtual del nodo en un estado erróneo.

Este problema se corrigió en las versiones siguientes de Kubernetes:

| Versión de Kubernetes | Versión corregida |
|--|:--:|
| 1.12 | 1.12.10 o posterior |
| 1.13 | 1.13.8 o posterior |
| 1,14 | 1.14.4 o posterior |
| 1.15 y posteriores | N/D |

Si usa una versión de Kubernetes que no tiene la solución para este problema y el nodo se encuentra en estado de error, puede resolverlo con la actualización manual del estado de la máquina virtual de una de las maneras siguientes:

* Para un clúster basado en un conjunto de disponibilidad:
    ```azurecli
    az vm update -n <VM_NAME> -g <RESOURCE_GROUP_NAME>
    ```

* Para un clúster basado en VMSS:
    ```azurecli
    az vmss update-instances -g <RESOURCE_GROUP_NAME> --name <VMSS_NAME> --instance-id <ID>
    ```

## <a name="azure-files-and-aks-troubleshooting"></a>Solución de problemas de Azure Files y AKS

### <a name="what-are-the-recommended-stable-versions-of-kubernetes-for-azure-files"></a>¿Cuáles son las versiones estables recomendadas de Kubernetes para Azure Files?
 
| Versión de Kubernetes | Versión recomendada |
|--|:--:|
| 1.12 | 1.12.6 o posterior |
| 1.13 | 1.13.4 o posterior |
| 1,14 | 1.14.0 o posterior |

### <a name="what-are-the-default-mountoptions-when-using-azure-files"></a>¿Cuál es el valor predeterminado de mountOptions cuando se usa Azure Files?

Configuración recomendada:

| Versión de Kubernetes | Valor de fileMode y dirMode|
|--|:--:|
| 1.12.0 - 1.12.1 | 0755 |
| 1.12.2 y versiones posteriores | 0777 |

Las opciones de montaje se pueden especificar en el objeto de clase de almacenamiento. En el ejemplo siguiente se establece *0777*:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azurefile
provisioner: kubernetes.io/azure-file
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=1000
  - gid=1000
  - mfsymlinks
  - nobrl
  - cache=none
parameters:
  skuName: Standard_LRS
```

Algunos valores de *mountOptions* útiles adicionales:

* *mfsymlinks* hará que el montaje de Azure Files (cifs) admita vínculos simbólicos.
* *nobrl* impedirá las solicitudes de bloqueo de intervalo de bytes al servidor. Esta configuración es necesaria para ciertas aplicaciones que se interrumpen con bloqueos de intervalo de bytes obligatorios de estilo CIFS. La mayoría de los servidores CIFS todavía no admiten la solicitud de bloqueos de intervalo de bytes aconsejables. Si no se usa *nobrl* , las aplicaciones que se interrumpen con bloqueos de intervalo de bytes obligatorios de estilo CIFS pueden generar mensajes de error similares a:
    ```console
    Error: SQLITE_BUSY: database is locked
    ```

### <a name="error-could-not-change-permissions-when-using-azure-files"></a>Error "Could not change permissions" (No se pudieron cambiar los permisos) cuando se usa Azure Files

Al ejecutar PostgreSQL en el complemento de Azure Files, puede ver un error similar al siguiente:

```console
initdb: could not change permissions of directory "/var/lib/postgresql/data": Operation not permitted
fixing permissions on existing directory /var/lib/postgresql/data
```

Este error se debe al complemento de Azure Files que usa el protocolo cifs/SMB. Al usar el protocolo cifs/SMB, no se pudieron cambiar los permisos de archivos y directorios después del montaje.

Para resolver este problema, use *subPath*  junto con el complemento de Azure Disk. 

> [!NOTE] 
> Para el tipo de disco ext3/4, se produce un error en el directorio perdido y encontrado después de dar formato al disco.

### <a name="azure-files-has-high-latency-compared-to-azure-disk-when-handling-many-small-files"></a>Azure Files tiene una latencia alta en comparación con Azure Disk al controlar muchos archivos pequeños

En algunos casos, como el control de muchos archivos pequeños, se puede experimentar una latencia elevada al usar Azure Files en comparación con Azure Disk.

### <a name="error-when-enabling-allow-access-allow-access-from-selected-network-setting-on-storage-account"></a>Error al habilitar la configuración "Permitir el acceso desde la red seleccionada" en la cuenta de almacenamiento

Si habilita la opción *Allow access from selected network* (Permitir el acceso desde la red seleccionada) en una cuenta de almacenamiento que se usa para el aprovisionamiento dinámico en AKS, recibirá un error cuando AKS cree un recurso compartido de archivos:

```console
persistentvolume-controller (combined from similar events): Failed to provision volume with StorageClass "azurefile": failed to create share kubernetes-dynamic-pvc-xxx in account xxx: failed to create file share, err: storage: service returned error: StatusCode=403, ErrorCode=AuthorizationFailure, ErrorMessage=This request is not authorized to perform this operation.
```

Este error se debe a que *persistentvolume-controller* de Kubernetes no está en la red que se eligió cuando se estableció la opción *Permitir el acceso desde la red seleccionada*.

Para mitigar el problema, use el [aprovisionamiento estático con Azure Files](azure-files-volume.md).

### <a name="azure-files-fails-to-remount-in-windows-pod"></a>Azure Files no se puede volver a montar en el pod de Windows

Si se elimina un pod de Windows con un montaje de Azure Files y luego se programa para volver a crearlo en el mismo nodo, el montaje generará un error. Este error se debe a un error del comando `New-SmbGlobalMapping` debido a que el montaje de Azure Files ya está montado en el nodo.

Por ejemplo, puede ver un error similar al siguiente:

```console
E0118 08:15:52.041014    2112 nestedpendingoperations.go:267] Operation for "\"kubernetes.io/azure-file/42c0ea39-1af9-11e9-8941-000d3af95268-pvc-d7e1b5f9-1af3-11e9-8941-000d3af95268\" (\"42c0ea39-1af9-11e9-8941-000d3af95268\")" failed. No retries permitted until 2019-01-18 08:15:53.0410149 +0000 GMT m=+732.446642701 (durationBeforeRetry 1s). Error: "MountVolume.SetUp failed for volume \"pvc-d7e1b5f9-1af3-11e9-8941-000d3af95268\" (UniqueName: \"kubernetes.io/azure-file/42c0ea39-1af9-11e9-8941-000d3af95268-pvc-d7e1b5f9-1af3-11e9-8941-000d3af95268\") pod \"deployment-azurefile-697f98d559-6zrlf\" (UID: \"42c0ea39-1af9-11e9-8941-000d3af95268\") : azureMount: SmbGlobalMapping failed: exit status 1, only SMB mount is supported now, output: \"New-SmbGlobalMapping : Generic failure \\r\\nAt line:1 char:190\\r\\n+ ... ser, $PWord;New-SmbGlobalMapping -RemotePath $Env:smbremotepath -Cred ...\\r\\n+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\\r\\n    + CategoryInfo          : NotSpecified: (MSFT_SmbGlobalMapping:ROOT/Microsoft/...mbGlobalMapping) [New-SmbGlobalMa \\r\\n   pping], CimException\\r\\n    + FullyQualifiedErrorId : HRESULT 0x80041001,New-SmbGlobalMapping\\r\\n \\r\\n\""
```

Este problema se corrigió en las versiones siguientes de Kubernetes:

| Versión de Kubernetes | Versión corregida |
|--|:--:|
| 1.12 | 1.12.6 o posterior |
| 1.13 | 1.13.4 o posterior |
| 1.14 y versiones posteriores | N/D |

### <a name="azure-files-mount-fails-because-of-storage-account-key-changed"></a>No se puede montar Azure Files debido a que la clave de la cuenta de almacenamiento cambió

Si la clave de cuenta de almacenamiento cambió, es posible que vea errores de montaje de Azure Files.

Para mitigar el problema, actualice manualmente el campo `azurestorageaccountkey` en el secreto del archivo de Azure con la clave de la cuenta de almacenamiento codificada en Base64.

Para codificar la clave de cuenta de almacenamiento en Base64, puede usar `base64`. Por ejemplo:

```console
echo X+ALAAUgMhWHL7QmQ87E1kSfIqLKfgC03Guy7/xk9MyIg2w4Jzqeu60CVw2r/dm6v6E0DWHTnJUEJGVQAoPaBc== | base64
```

Para actualizar el archivo secreto de Azure, use `kubectl edit secret`. Por ejemplo:

```console
kubectl edit secret azure-storage-account-{storage-account-name}-secret
```

Al cabo de unos minutos, el nodo del agente volverá a intentar el montaje de Azure Files con la clave de almacenamiento actualizada.


### <a name="cluster-autoscaler-fails-to-scale-with-error-failed-to-fix-node-group-sizes"></a>El escalador automático del clúster no se puede escalar con el error que indica que no se pudieron corregir los tamaños del grupo de nodos

Si el escalador automático del clúster no se amplía ni se reduce y aparece un error como el siguiente en los [registros del escalador automático del clúster][view-master-logs].

```console
E1114 09:58:55.367731 1 static_autoscaler.go:239] Failed to fix node group sizes: failed to decrease aks-default-35246781-vmss: attempt to delete existing nodes
```

Este error se debe a una condición de carrera del escalador automático del clúster de subida. En tal caso, el escalador automático del clúster termina con un valor diferente al que se encuentra realmente en el clúster. Para salir de este estado, deshabilite y vuelva a habilitar el [escalador automático del clúster][cluster-autoscaler].

### <a name="slow-disk-attachment-getazuredisklun-takes-10-to-15-minutes-and-you-receive-an-error"></a>Conexión lenta de disco, GetAzureDiskLun tarda entre 10 y 15 minutos y recibe un error.

En las versiones de Kubernetes **anteriores a 1.15.0**, puede recibir un error similar a **Error: WaitForAttach no encuentra el LUN del disco**.  Para evitar este problema, espere unos 15 minutos y vuelva a intentarlo.

<!-- LINKS - internal -->
[view-master-logs]: view-master-logs.md
[cluster-autoscaler]: cluster-autoscaler.md

### <a name="why-do-upgrades-to-kubernetes-116-fail-when-using-node-labels-with-a-kubernetesio-prefix"></a>¿Por qué se produce un error en las actualizaciones a Kubernetes 1.16 al usar etiquetas de nodo con un prefijo kubernetes.io?

A partir de Kubernetes [1.16](https://v1-16.docs.kubernetes.io/docs/setup/release/notes/) [, el kubelet solo puede aplicar un subconjunto definido de etiquetas con el prefijo kubernetes.io](https://github.com/kubernetes/enhancements/blob/master/keps/sig-auth/0000-20170814-bounding-self-labeling-kubelets.md#proposal) a los nodos. AKS no puede quitar etiquetas activas en su nombre sin consentimiento, ya que puede provocar tiempo de inactividad en las cargas de trabajo afectadas.

Como resultado, para mitigar este problema, puede:

1. Actualizar el plano de control del clúster a 1.16 o superior
2. Agregar un nuevo grupo de nodos en 1.16 o una versión posterior sin las etiquetas de kubernetes.io no compatibles
3. Eliminar el antiguo grupo de nodos

AKS está investigando la capacidad de mutar etiquetas activas en un grupo de nodos para mejorar esta mitigación.
