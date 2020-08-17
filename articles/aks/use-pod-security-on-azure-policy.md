---
title: Protección de pods con Azure Policy en Azure Kubernetes Service (AKS)
description: Aprenda a proteger pods con Azure Policy en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 07/06/2020
author: jluk
ms.openlocfilehash: 8be0b05c260037bbe8afc92726d81668e1391d4a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87050468"
---
# <a name="secure-pods-with-azure-policy-preview"></a>Protección de pods con Azure Policy (versión preliminar)

Para mejorar la seguridad del clúster de AKS, puede controlar qué funciones se conceden a los pods y si se ejecuta algo que no cumpla la directiva de la empresa. Este acceso se define mediante las directivas integradas proporcionadas por el [complemento de Azure Policy para AKS][kubernetes-policy-reference]. Al proporcionar un control adicional sobre los aspectos de seguridad de la especificación del pod, como los privilegios de usuario raíz, permite una adherencia más estricta de la seguridad y visibilidad sobre lo que se implementa en el clúster. Si un pod no cumple las condiciones especificadas en la directiva, Azure Policy puede impedir que se inicie el pod o marcar una infracción. En este artículo se muestra cómo usar Azure Policy para limitar la implementación de pods en AKS.

> [!IMPORTANT]
> Las características en vista previa de AKS son de autoservicio y se tienen que habilitar. Las versiones preliminares se proporcionan "tal cual" y "como están disponibles", y están excluidas de los contratos de nivel de servicio y la garantía limitada. Las versiones preliminares de AKS reciben cobertura parcial del soporte al cliente en la medida de lo posible. Por lo tanto, estas características no están diseñadas para usarse en producción. Para más información, consulte los siguientes artículos de soporte:
>
> * [Directivas de soporte técnico para AKS][aks-support-policies]
> * [Preguntas más frecuentes de soporte técnico de Azure][aks-faq]

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

### <a name="install-the-azure-policy-add-on-for-aks"></a>Instalación del complemento de Azure Policy para AKS

Para proteger los pods de AKS mediante Azure Policy, debe instalar el complemento de Azure Policy para AKS en un clúster de AKS. Siga estos [pasos para instalar el complemento de Azure Policy](../governance/policy/concepts/policy-for-kubernetes.md#install-azure-policy-add-on-for-aks).

En este documento se supone que dispone de los siguientes elementos, que se implementan en el tutorial vinculado anteriormente.

* Ha registrado los proveedores de recursos `Microsoft.ContainerService` y `Microsoft.PolicyInsights` mediante `az provider register`
* Ha registrado la marca de características en versión preliminar `AKS-AzurePolicyAutoApprove` mediante `az feature register`
* La CLI de Azure instalada con la versión 0.4.53 o posterior de la extensión `aks-preview`
* Un clúster de AKS con la versión 1.15 o superior instalada con el complemento de Azure Policy

## <a name="overview-of-securing-pods-with-azure-policy-for-aks-preview"></a>Introducción a la protección de pods con Azure Policy para AKS (versión preliminar)

>[!NOTE]
> En este documento se detalla cómo usar Azure Policy para proteger los pods, sucesor de la [característica de directiva de seguridad de pod de Kubernetes en versión preliminar](use-pod-security-policies.md).
> **La directiva de seguridad de pod (versión preliminar) y el complemento de Azure Policy para AKS no se pueden habilitar simultáneamente.**
> 
> Si instala el complemento de Azure Policy en un clúster con la directiva de seguridad de pod habilitada, [siga estos pasos para deshabilitar la directiva de seguridad de pod](use-pod-security-policies.md#enable-pod-security-policy-on-an-aks-cluster).

En un clúster de AKS, se usa un controlador de admisión para interceptar las solicitudes al servidor de API cuando se va a crear y actualizar un recurso. A continuación, el controlador de admisión puede *validar* la solicitud de recurso con un conjunto de reglas para determinar si se debería crear.

Anteriormente, la característica [directiva de seguridad de pod (versión preliminar)](use-pod-security-policies.md) se habilitaba mediante el proyecto de Kubernetes para limitar los pods que se pueden implementar. Esta característica ya no está en desarrollo activo desde el proyecto Kubernetes.

Mediante el complemento de Azure Policy, un clúster de AKS puede usar directivas integradas de Azure que protegen los pods y otros recursos de Kubernetes de forma similar a la anterior directiva de seguridad de pod. El complemento de Azure Policy para AKS instala una instancia administrada de [Gatekeeper](https://github.com/open-policy-agent/gatekeeper), un controlador de admisión para la validación. Azure Policy para Kubernetes se basa en el software de código abierto Open Policy Agent,que se basa en el [lenguaje de directivas Rego](../governance/policy/concepts/policy-for-kubernetes.md#policy-language).

En este documento se detalla cómo usar Azure Policy para proteger los pods de un clúster de AKS y se indica cómo migrar desde las directivas de seguridad de pod (versión preliminar).

## <a name="limitations"></a>Limitaciones

* Durante la versión preliminar, se pueden ejecutar un máximo de 200 pods con 20 directivas de Azure Policy para Kubernetes en un solo clúster.
* Se excluyen de la evaluación de las directivas [algunos espacios de nombres del sistema](#namespace-exclusion) que contienen pods administrados por AKS.
* Los pods con Windows [no admiten contextos de seguridad](https://kubernetes.io/docs/concepts/security/pod-security-standards/#what-profiles-should-i-apply-to-my-windows-pods), por lo que muchas de las directivas de Azure solo se aplican a los pods con Linux, como no permitir privilegios de usuario raíz, que no se pueden escalar en pods con Windows.
* La directiva de seguridad de pod y el complemento de Azure Policy para AKS no se pueden habilitar simultáneamente. Si instala el complemento de Azure Policy en un clúster con la directiva de seguridad de pod habilitada, deshabilite la directiva de seguridad de pod con las [instrucciones siguientes](use-pod-security-policies.md#enable-pod-security-policy-on-an-aks-cluster).

## <a name="azure-policies-to-secure-kubernetes-pods"></a>Directivas de Azure para proteger los pods de Kubernetes

Después de instalar el complemento de Azure Policy, no se aplica ninguna directiva de forma predeterminada.

Hay once (11) directivas integradas de Azure individuales y dos (2) iniciativas integradas que protegen de forma específica los pods de un clúster de AKS.
Cada directiva se puede personalizar con un efecto. Aquí se muestra una lista completa de las [directivas de AKS y los efectos admitidos][policy-samples]. Más información sobre los [efectos de Azure Policy](../governance/policy/concepts/effects.md).

Las directivas de Azure se pueden aplicar en el nivel de grupo de administración, suscripción o grupo de recursos. Al asignar una directiva en el nivel de grupo de recursos, asegúrese de que el grupo de recursos del clúster de AKS de destino está seleccionado dentro del ámbito de la directiva. Cada clúster del ámbito asignado con el complemento de Azure Policy instalado está en el ámbito de la directiva.

Si usa [directiva de seguridad de pod (versión preliminar)](use-pod-security-policies.md), obtenga información sobre cómo [migrar a Azure Policy y sobre otras diferencias de comportamiento](#migrate-from-kubernetes-pod-security-policy-to-azure-policy).

### <a name="built-in-policy-initiatives"></a>Iniciativas de directiva integradas

Una iniciativa de Azure Policy es una colección de definiciones de directiva personalizadas para alcanzar un único objetivo general. El uso de iniciativas puede simplificar la administración y asignación de directivas en clústeres de AKS. Una iniciativa existe como un solo objeto. Más información sobre las [iniciativas de Azure Policy](../governance/policy/overview.md#initiative-definition).

Azure Policy para Kubernetes ofrece dos iniciativas integradas que protegen los pods: [línea de base](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) y [restringida](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F42b8ef37-b724-4e24-bbc8-7a7708edfe00).

Ambas iniciativas integradas se crean a partir de las definiciones utilizadas en la [directiva de seguridad de pod de Kubernetes](https://github.com/kubernetes/website/blob/master/content/en/examples/policy/baseline-psp.yaml).

|[Control de directiva de seguridad de pod](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#what-is-a-pod-security-policy)| Vínculo de definición de Azure Policy| [Iniciativa de línea de base](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) | [Iniciativa restringida](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F42b8ef37-b724-4e24-bbc8-7a7708edfe00) |
|---|---|---|---|
|No permitir la ejecución de contenedores con privilegios|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F95edb821-ddaf-4404-9732-666045e056b4)| Sí | Sí
|No permitir el uso compartido de espacios de nombres de host|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47a1ee2f-2a2a-4576-bf2a-e0e36709c2b8)| Sí | Sí
|Restringir todo uso de puertos y redes de host|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F82985f06-dc18-4a48-bc1c-b9f4f0098cfe)| Sí | Sí
|Restringir todo uso del sistema de archivos de host|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F098fc59e-46c7-4d99-9b16-64990e543d75)| Sí | Sí
|Restringir las capacidades de Linux al [conjunto predeterminado](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc26596ff-4d70-4e6a-9a30-c2506bd2f80c) | Sí | Sí
|Restringir el uso de tipos de volúmenes definidos|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F16697877-1118-4fb1-9b65-9898ec2509ec)| - | Sí: los tipos de volúmenes permitidos son `configMap`, `emptyDir`, `projected`, `downwardAPI`, `persistentVolumeClaim`|
|Elevación de privilegios a usuario raíz|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1c6e92c9-99f0-4e55-9cf2-0c234dc48f99) | - | Sí |
|Restringir los identificadores de usuario y de grupo del contenedor|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff06ddb64-5fa3-4b77-b166-acb36f7f6042) | - | Sí|
|Restringir la asignación de un FSGroup que posee los volúmenes del pod|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff06ddb64-5fa3-4b77-b166-acb36f7f6042) | - | Sí: las reglas permitidas son `runAsUser: mustRunAsNonRoot`, `supplementalGroup: mustRunAs 1:65536`, `fsGroup: mustRunAs 1:65535`, `runAsGroup: mustRunAs 1:65535`.  |
|Requiere el perfil de seccomp|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F975ce327-682c-4f2e-aa46-b9598289b86c) | - | Sí, los perfiles permitidos son *`docker/default` o `runtime/default` |

\* docker/default está en desuso en Kubernetes a partir de v1.11

### <a name="additional-optional-policies"></a>Directivas opcionales adicionales

Hay otras directivas de Azure que se pueden aplicar por sí mismas fuera del proceso de aplicación de una iniciativa. Considere la posibilidad de anexar estas directivas a las iniciativas si las iniciativas integradas no cumplen los requisitos.

|[Control de directiva de seguridad de pod](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#what-is-a-pod-security-policy)| Vínculo de definición de Azure Policy| Aplicar además de la iniciativa de línea de base | Aplicar además de la iniciativa restringida |
|---|---|---|---|
|Definir el perfil de AppArmor que usan los contenedores|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F511f5417-5d12-434d-ab2e-816901e72a5e) | Opcional | Opcional |
|Permitir montajes que no sean de solo lectura|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdf49d893-a74c-421d-bc95-c663042e5b80) | Opcional | Opcional |
|Restringir a controladores de FlexVolume específicos|[Nube pública](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff4a8fce0-2dd5-4c21-9a36-8f0ec809d663) | Opcional: use si solo quiere restringir los controladores de FlexVolume, pero no otros establecidos por "Restringir el uso de tipos de volúmenes definidos" | No aplicable: la iniciativa restringida incluye "Restringir el uso de tipos de volúmenes definidos", que prohíbe todos los controladores de FlexVolume |

### <a name="unsupported-built-in-policies-for-managed-aks-clusters"></a>Directivas integradas no compatibles con clústeres de AKS administrados

> [!NOTE]
> Las tres directivas siguientes **no son compatibles con AKS** debido a la personalización de aspectos administrados y protegidos por AKS como servicio administrado. Estas directivas se compilan específicamente para clústeres conectados de Azure Arc con planos de control no administrados.

|[Control de directiva de seguridad de pod](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#what-is-a-pod-security-policy)|
|---|
|Definir el contexto de SELinux personalizado de un contenedor|
|Restringir el perfil de sysctl que usan los contenedores|
|Los tipos de montaje de procedimiento predeterminados se definen para reducir la superficie expuesta a ataques|

<!---
# Removing until custom initiatives are supported the week after preview

#### Custom initiative

If the built-in initiatives to address pod security do not match your requirements, you can choose your own policies to exist in a custom initiative. Read more about [building custom initatives in Azure Policy](../governance/policy/tutorials/create-and-manage#create-and-assign-an-initiative-definition).

--->

### <a name="namespace-exclusion"></a>Exclusión de espacios de nombres

> [!WARNING]
> Los pods de los espacios de nombres administrativos como kube-system deben ejecutarse para que un clúster siga con un funcionamiento correcto. Si se elimina un espacio de nombres necesario de la lista de espacios de nombres excluidos predeterminados, se pueden desencadenar infracciones de directivas debido a un pod del sistema necesario.

AKS requiere que los pods del sistema se ejecuten en un clúster para proporcionar servicios críticos, como la resolución DNS. Las directivas que limitan la funcionalidad del pod pueden afectar a la estabilidad del pod del sistema. Como resultado, los siguientes espacios de nombres se **excluyen de la evaluación de directivas durante las solicitudes de admisión durante la creación, actualización y auditoría de directivas**. Esto fuerza la exclusión de las directivas de Azure de las nuevas implementaciones en estos espacios de nombres.

1. kube-system
1. gatekeeper-system
1. azure-arc
1. aks-periscope

Los espacios de nombres personalizados adicionales se pueden excluir de la evaluación durante la creación, la actualización y la auditoría. Se debe usar si tiene pods especializados que se ejecutan en un espacio de nombres autorizado y desea evitar que se desencadenen infracciones de auditoría.

## <a name="apply-the-baseline-initiative"></a>Aplicación de la iniciativa de línea de base

> [!TIP]
> Todas las directivas tienen como valor predeterminado un efecto de auditoría. Los efectos se pueden actualizar para denegar en cualquier momento mediante Azure Policy.

Para aplicar la iniciativa de línea de base, podemos realizar la asignación mediante Azure Portal.
<!--
1. Navigate to the Policy service in Azure portal
1. In the left pane of the Azure Policy page, select **Definitions**
1. Search for "Baseline Profile" on the search pane to the right of the page
1. Select `Kubernetes Pod Security Standards Baseline Profile for Linux-based workloads` from the `Kubernetes` category
-->
1. Siga [este vínculo a Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) para revisar la iniciativa de línea de base de seguridad de pod.
1. Establezca el valor de **Ámbito** en el nivel de suscripción o solo en el grupo de recursos que contiene los clústeres de AKS con el complemento de Azure Policy habilitado.
1. Seleccione la página **Parámetros** y actualice **Efecto** de `audit` a `deny` para bloquear las nuevas implementaciones que infrinjan la iniciativa de línea de base.
1. Agregue espacios de nombres adicionales para excluirlos de la evaluación durante la creación, la actualización y la auditoría; [algunos espacios de nombres se excluyen forzosamente de la evaluación de directivas.](#namespace-exclusion)
![Actualización del efecto](media/use-pod-security-on-azure-policy/update-effect.png)
1. Seleccione **Revisar y crear** para enviar las directivas.

Confirme que las directivas se aplican al clúster mediante la ejecución de `kubectl get constrainttemplates`.

> [!NOTE]
> Las directivas pueden tardar [hasta 20 minutos en sincronizar](../governance/policy/concepts/policy-for-kubernetes.md#assign-a-built-in-policy-definition) en cada clúster.

La salida debe ser similar a:

```console
$ kubectl get constrainttemplate
NAME                                     AGE
k8sazureallowedcapabilities              30m
k8sazureblockhostnamespace               30m
k8sazurecontainernoprivilege             30m
k8sazurehostfilesystem                   30m
k8sazurehostnetworkingports              30m
```

## <a name="validate-rejection-of-a-privileged-pod"></a>Validación del rechazo de un pod con privilegios

Vamos a probar primero lo que sucede cuando se programa un pod con el contexto de seguridad `privileged: true`. Este contexto de seguridad eleva los privilegios del pod. La iniciativa de línea de base no permite pods con privilegios, por lo que se denegará la solicitud, lo que provocará que se rechace la implementación.

Cree un archivo denominado `nginx-privileged.yaml` y pegue el siguiente manifiesto de YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-privileged
spec:
  containers:
    - name: nginx-privileged
      image: nginx
      securityContext:
        privileged: true
```

Cree el pod con el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl apply -f nginx-privileged.yaml
```

Como se esperaba, el pod no se puede incluir en la programación, como se muestra en la salida del ejemplo siguiente:

```console
$ kubectl apply -f privileged.yaml

Error from server ([denied by azurepolicy-container-no-privilege-00edd87bf80f443fa51d10910255adbc4013d590bec3d290b4f48725d4dfbdf9] Privileged container is not allowed: nginx-privileged, securityContext: {"privileged": true}): error when creating "privileged.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [denied by azurepolicy-container-no-privilege-00edd87bf80f443fa51d10910255adbc4013d590bec3d290b4f48725d4dfbdf9] Privileged container is not allowed: nginx-privileged, securityContext: {"privileged": true}
```

El pod no llega a la fase de programación, por lo que no hay recursos que eliminar para continuar.

## <a name="test-creation-of-an-unprivileged-pod"></a>Prueba de la creación de un pod sin privilegios

En el ejemplo anterior la imagen de contenedor intentó automáticamente utilizar la raíz para enlazar NGINX al puerto 80. La iniciativa de la directiva de línea de base denegó esta solicitud, por lo que no se puede iniciar el pod. Vamos a probar ahora a ejecutar ese mismo pod con NGINX sin acceso con privilegios.

Cree un archivo denominado `nginx-unprivileged.yaml` y pegue el siguiente manifiesto de YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-unprivileged
spec:
  containers:
    - name: nginx-unprivileged
      image: nginx
```

Cree el pod mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl apply -f nginx-unprivileged.yaml
```

Se ha programado correctamente el pod. Al activar el estado del pod mediante el comando [kubectl get pods][kubectl-get], el pod tiene el estado *En ejecución*:

```console
$ kubectl get pods

NAME                 READY   STATUS    RESTARTS   AGE
nginx-unprivileged   1/1     Running   0          18s
```

En este ejemplo se muestra que la iniciativa de línea de base solo afecta a las implementaciones que infringen las directivas de la colección. Las implementaciones permitidas continúan funcionando.

Elimine el pod NGIX sin privilegios mediante el comando [kubectl delete][kubectl-delete] y especifique el nombre del manifiesto de YAML:

```console
kubectl delete -f nginx-unprivileged.yaml
```

## <a name="disable-policies-and-the-azure-policy-add-on"></a>Deshabilitación de directivas y el complemento de Azure Policy

Para eliminar la iniciativa de línea de base:

1. Vaya al panel Directiva en Azure Portal.
1. Seleccione **Asignaciones** en el panel izquierdo.
1. Haga clic en el botón "..." situado junto al perfil de línea de base.
1. Seleccione "Eliminar asignación".

![Eliminar asignación](media/use-pod-security-on-azure-policy/delete-assignment.png)

Para deshabilitar el complemento de Azure Policy, use el comando [az aks disable-addons][az-aks-disable-addons].

```azure-cli
az aks disable-addons --addons azure-policy --name MyAKSCluster --resource-group MyResourceGroup
```

Obtenga información acerca de cómo eliminar el [complemento de Azure Policy desde Azure Portal](../governance/policy/concepts/policy-for-kubernetes.md#remove-the-add-on-from-aks).

## <a name="migrate-from-kubernetes-pod-security-policy-to-azure-policy"></a>Migración desde la directiva de seguridad de pod de Kubernetes a Azure Policy

Para migrar desde la directiva de seguridad de pod, debe realizar las siguientes acciones en un clúster.

1. [Deshabilitar la directiva de seguridad de pod](use-pod-security-policies.md#clean-up-resources) en el clúster
1. Habilitar el [complemento de Azure Policy][kubernetes-policy-reference]
1. Habilitar las directivas de Azure deseadas a partir de las [directivas integradas disponibles][policy-samples]

A continuación se muestra un resumen de los cambios de comportamiento entre la directiva de seguridad de pod y Azure Policy.

|Escenario| Directiva de seguridad de pod | Azure Policy |
|---|---|---|
|Instalación|Habilitar la característica de directiva de seguridad de pod |Habilitar el complemento de Azure Policy
|Implementar directivas| Implementar el recurso de la directiva de seguridad de pod| Asigne las directivas de Azure al ámbito de la suscripción o del grupo de recursos. El complemento de Azure Policy es necesario para las aplicaciones de recursos de Kubernetes.
| Directivas predeterminadas | Cuando la directiva de seguridad de pod está habilitada en AKS, se aplican las directivas predeterminadas Con privilegios y Sin restricciones. | No se aplica ninguna directiva predeterminada al habilitar el complemento de Azure Policy. Debe habilitar explícitamente las directivas en Azure Policy.
| Quién puede crear y asignar directivas | El administrador del clúster crea un recurso de directiva de seguridad de pod | Los usuarios deben tener un rol mínimo con permisos de "propietario" o "colaborador de directiva de recursos" en el grupo de recursos del clúster de AKS. - Mediante la API, los usuarios pueden asignar directivas en el ámbito del recurso del clúster de AKS. El usuario debe tener permisos mínimos de "propietario" o "colaborador de directiva de recursos" en el recurso del clúster de AKS. - En Azure Portal, las directivas se pueden asignar en el nivel de grupo de administración, suscripción o grupo de recursos.
| Autorizar directivas| Los usuarios y las cuentas de servicio requieren permisos explícitos para usar las directivas de seguridad de pod. | No se requiere ninguna asignación adicional para autorizar directivas. Una vez que se han asignado las directivas en Azure, todos los usuarios del clúster pueden usar estas directivas.
| Aplicabilidad de las directivas | El usuario administrador está exento del cumplimiento de las directivas de seguridad de pod. | Todos los usuarios (administradores y no administradores) ven las mismas directivas. No hay ninguna grafía especial basada en los usuarios. La aplicación de las directivas se puede excluir en el nivel de espacio de nombres.
| Ámbito de la directiva | Las directivas de seguridad de pod no tienen espacios de nombres | Las plantillas de restricción usadas por Azure Policy no tienen espacios de nombres.
| Acción de denegación/auditoría/mutación | Las directivas de seguridad de pod solo admiten acciones de denegación. La mutación se puede realizar con valores predeterminados en las solicitudes de creación. La validación se puede realizar durante las solicitudes de actualización.| Azure Policy admite las acciones de denegación y de auditoría. La mutación todavía no se admite, pero está planeada.
| Cumplimiento de la directiva de seguridad de pod | No hay visibilidad del cumplimiento de los pods que existían antes de habilitar la directiva de seguridad de pod. Los pods no compatibles creados después de habilitar las directivas de seguridad de pod se deniegan. | Los pods no compatibles que existían antes de aplicar las directivas de Azure se mostrarían en infracciones de directivas. Los pods no compatibles creados después de habilitar las directivas de Azure se deniegan si las directivas se establecen con efecto de denegación.
| Cómo ver las directivas del clúster | `kubectl get psp` | `kubectl get constrainttemplate` : devuelve todas las directivas.
| Directiva de seguridad de pod estándar: con privilegios | De forma predeterminada, se crea un recurso de directiva de seguridad de pod con privilegios al habilitar la característica. | El modo con privilegios no implica ninguna restricción, por tanto, es equivalente a no tener ninguna asignación de Azure Policy.
| [Directiva de seguridad de pod estándar: línea de base/predeterminada](https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline-default) | El usuario instala un recurso de línea de base de la directiva de seguridad de pod. | Azure Policy proporciona una [iniciativa de línea de base integrada](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) que se asigna a la directiva de seguridad de pod de línea de base.
| [Directiva de seguridad de pod estándar: restringida](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted) | El usuario instala un recurso restringido de la directiva de seguridad de pod. | Azure Policy proporciona una [iniciativa restringida integrada](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F42b8ef37-b724-4e24-bbc8-7a7708edfe00) que se asigna a la directiva de seguridad de pod de restringida.

## <a name="next-steps"></a>Pasos siguientes

En este artículo se ha mostrado cómo aplicar una directiva de Azure que restringe la implementación de pods con privilegios para evitar el uso del acceso con privilegios. Hay muchas directivas que se pueden aplicar, como las que restringen el uso de los volúmenes. Para más información sobre las opciones disponibles, consulte [Documentación de referencia de Azure Policy para Kubernetes][kubernetes-policy-reference].

Para más información sobre la limitación del tráfico del pod, consulte [Protección del tráfico entre pods mediante directivas de red en Azure Kubernetes Service (AKS)][network-policies].

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->
[kubernetes-policy-reference]: ../governance/policy/concepts/policy-for-kubernetes.md
[policy-samples]: policy-samples.md#microsoftcontainerservice
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[network-policies]: use-network-policies.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-aks-update]: /cli/azure/ext/aks-preview/aks#ext-aks-preview-az-aks-update
[az-extension-add]: /cli/azure/extension#az-extension-add
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-disable-addons]: /cli/azure/aks#az-aks-disable-addons
[az-policy-assignment-delete]: /cli/azure/policy/assignment#az-policy-assignment-delete