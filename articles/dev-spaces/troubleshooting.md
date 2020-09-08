---
title: Solución de problemas
services: azure-dev-spaces
ms.date: 09/25/2019
ms.topic: troubleshooting
description: Obtenga información sobre cómo solucionar problemas y resolver problemas comunes al habilitar y usar Azure Dev Spaces
keywords: 'Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores, Helm, service mesh, enrutamiento de service mesh, kubectl, k8s '
ms.openlocfilehash: e26f066294cb0a6a48c5a3299213206fe4226ad0
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88210826"
---
# <a name="azure-dev-spaces-troubleshooting"></a>Solución de problemas de Azure Dev Spaces

Esta guía contiene información sobre problemas habituales que pueden surgir al usar Azure Dev Spaces.

Si tiene algún problema al usar Azure Dev Spaces, cree un [problema en el repositorio de GitHub para Azure Dev Spaces](https://github.com/Azure/dev-spaces/issues).

## <a name="before-you-begin"></a>Antes de empezar

Para solucionar problemas de forma más eficaz, puede ser útil crear registros más detallados para su revisión.

Para Visual Studio, establezca la variable de entorno `MS_VS_AZUREDEVSPACES_TOOLS_LOGGING_ENABLED` en 1. Asegúrese de reiniciar Visual Studio para que la variable de entorno surta efecto. Una vez habilitados, los registros detallados se escribirán en su directorio `%TEMP%\Microsoft.VisualStudio.Azure.DevSpaces.Tools`.

En la CLI, puede generar más información durante la ejecución del comando mediante el uso del conmutador `--verbose`. También puede examinar registros más detallados en `%TEMP%\Azure Dev Spaces`. En un equipo Mac, para encontrar el directorio *TEMP* hay que ejecutar `echo $TMPDIR` desde una ventana de terminal. En un equipo Linux, el directorio *TEMP* es normalmente `/tmp`. Además, compruebe que el registro está habilitado en el [archivo de configuración de la CLI de Azure](/cli/azure/azure-cli-configuration?view=azure-cli-latest#cli-configuration-values-and-environment-variables).

Azure Dev Spaces también funciona mejor cuando se depura una sola instancia o pod. El archivo `azds.yaml` contiene un valor, *replicaCount*, que indica el número de pods que Kubernetes ejecuta para el servicio. Si cambia el valor de *replicaCount* para configurar la aplicación de forma que ejecute varios pods para un servicio determinado, el depurador se asocia al primer pod (cuando se muestran en orden alfabético). El depurador se asocia a un pod diferente cuando se recicla el pod original, lo que da lugar posiblemente a un comportamiento inesperado.

## <a name="common-issues-when-using-local-process-with-kubernetes"></a>Problemas comunes al usar Proceso local con Kubernetes

### <a name="fail-to-restore-original-configuration-of-deployment-on-cluster"></a>No se pudo restaurar la configuración original de la implementación en el clúster

Al usar el cliente Proceso local con Kubernetes, si este se bloquea o se finaliza de forma repentina, es posible que el servicio que redirige el proceso local con Kubernetes no se restaure a su estado original antes de que el proceso local con Kubernetes se conecte a él.

Para corregir este problema, vuelva a implementar el servicio en el clúster.

## <a name="common-issues-when-enabling-azure-dev-spaces"></a>Problemas comunes al habilitar Azure Dev Spaces

### <a name="error-failed-to-create-azure-dev-spaces-controller"></a>Error "Failed to create Azure Dev Spaces controller" (Error al crear el controlador de Azure Dev Spaces)

Es posible que vea este error cuando algo va mal con la creación del controlador. Si es un error transitorio, elimine el controlador y vuelva a crearlo para corregirlo.

También puede intentar eliminar el controlador:

```bash
azds remove -g <resource group name> -n <cluster name>
```

Para eliminar un controlador, use la CLI de Azure Dev Spaces. No es posible eliminar un controlador desde Visual Studio. Tampoco puede instalar la CLI de Azure Dev Spaces en Azure Cloud Shell, así que no podrá eliminar un controlador desde este servicio.

Si no tiene la CLI de Azure Dev Spaces instalada, puede instalarla mediante el comando siguiente y, a continuación, eliminar el controlador:

```azurecli
az aks use-dev-spaces -g <resource group name> -n <cluster name>
```

### <a name="controller-create-failing-because-of-controller-name-length"></a>Error al crear el controlador debido a la longitud del nombre del controlador

El nombre de un controlador de Azure Dev Spaces no puede tener más de 31 caracteres. Si el nombre supera los 31 caracteres, cuando habilite Dev Spaces en un clúster de AKS o cree un controlador, recibirá un error. Por ejemplo:

```console
Failed to create a Dev Spaces controller for cluster 'a-controller-name-that-is-way-too-long-aks-east-us': Azure Dev Spaces Controller name 'a-controller-name-that-is-way-too-long-aks-east-us' is invalid. Constraint(s) violated: Azure Dev Spaces Controller names can only be at most 31 characters long*
```

Para corregir este problema, cree un controlador con un nombre alternativo. Por ejemplo:

```cmd
azds controller create --name my-controller --target-name MyAKS --resource-group MyResourceGroup
```

### <a name="enabling-dev-spaces-failing-when-windows-node-pools-are-added-to-an-aks-cluster"></a>No se puede habilitar Dev Spaces cuando se agregan grupos de nodos de Windows a un clúster de AKS

Actualmente, Azure Dev Spaces está diseñado para ejecutarse solo en pods y nodos de Linux. Cuando haya un clúster de AKS con un grupo de nodos de Windows, debe asegurarse de que los pods de Azure Dev Spaces solo se programan en nodos de Linux. Si un pod de Azure Dev Spaces está programado para ejecutarse en un nodo de Windows, ese pod no se iniciará y se producirá un error al habilitar Dev Spaces.

Para corregir este problema, [agregue un valor taint](../aks/operator-best-practices-advanced-scheduler.md#provide-dedicated-nodes-using-taints-and-tolerations) al clúster de AKS para asegurarse de que los pods de Linux no se programan para ejecutarse en un nodo de Windows.

### <a name="error-found-no-untainted-linux-nodes-in-ready-state-on-the-cluster-there-needs-to-be-at-least-one-untainted-linux-node-in-ready-state-to-deploy-pods-in-azds-namespace"></a>Error "Found no untainted Linux nodes in Ready state on the cluster. There needs to be at least one untainted Linux node in Ready state to deploy pods in 'azds' namespace." (No se encontraron nodos de Linux sin valor taint en estado Listo en el clúster y se necesita al menos uno para implementar pods en el espacio de nombres "azds").

Azure Dev Spaces no pudo crear un controlador en el clúster de AKS porque no encontró un nodo sin valor taint con estado *Listo* donde programar pods. Azure Dev Spaces necesita al menos un nodo de Linux en estado *Listo* que permita programar pods sin especificar tolerations.

Para corregir este problema, [actualice la configuración de taint](../aks/operator-best-practices-advanced-scheduler.md#provide-dedicated-nodes-using-taints-and-tolerations) en el clúster de AKS para asegurarse de que al menos un nodo de Linux permite programar pods sin especificar tolerations. Además, asegúrese de que al menos uno de los nodos de Linux que permitan programar pods sin especificar tolerations tenga el estado *Listo*. Si el nodo está tardando mucho tiempo en alcanzar el estado *Listo*, puede probar a reiniciarlo.

### <a name="error-azure-dev-spaces-cli-not-installed-properly-when-running-az-aks-use-dev-spaces"></a>Error "Azure Dev Spaces CLI not installed properly" (La CLI de Azure Dev Spaces no está instalada correctamente) al ejecutar az aks use-dev-spaces

Una actualización de la CLI de Azure Dev Spaces puede haber cambiado su ruta de acceso de instalación. Si usa una versión de la CLI de Azure anterior a la 2.0.63, puede ver este error. Para mostrar la versión de la CLI de Azure, use `az --version`.

```azurecli
az --version
```

```output
azure-cli                         2.0.60 *
...
```

A pesar del mensaje de error al ejecutar `az aks use-dev-spaces` con una versión de la CLI de Azure anterior a la 2.0.63, la instalación se realiza correctamente. Puede seguir usando `azds` sin ningún problema.

Para corregir este problema, actualice la instalación de la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) a la versión 2.0.63 o posterior. Esta actualización resolverá el mensaje de error que recibe al ejecutar `az aks use-dev-spaces`. Como alternativa, puede continuar usando la versión actual de la CLI y la CLI de Azure Dev Spaces.

### <a name="error-unable-to-reach-kube-apiserver"></a>Error "No se puede conectar con kube-apiserver"

Este error puede aparecer cuando Azure Dev Spaces no puede conectarse al servidor de la API del clúster de AKS.

Si el acceso al servidor de la API del clúster de AKS está bloqueado, o si tiene [intervalos de direcciones IP autorizados en el servidor de la API](../aks/api-server-authorized-ip-ranges.md) que estén habilitados para el clúster de AKS, también deberá [crear](../aks/api-server-authorized-ip-ranges.md#create-an-aks-cluster-with-api-server-authorized-ip-ranges-enabled) o [actualizar](../aks/api-server-authorized-ip-ranges.md#update-a-clusters-api-server-authorized-ip-ranges) el clúster de forma que [permita otros intervalos basados en la región](configure-networking.md#aks-cluster-network-requirements).

Asegúrese de que el servidor de la API está disponible ejecutando comandos kubectl. Si el servidor de la API no está disponible, póngase en contacto con el soporte técnico de AKS y vuelva a intentarlo cuando esté en funcionamiento.

## <a name="common-issues-when-preparing-your-project-for-azure-dev-spaces"></a>Problemas comunes al preparar el proyecto para Azure Dev Spaces

### <a name="warning-dockerfile-could-not-be-generated-due-to-unsupported-language"></a>Advertencia: "Dockerfile could not be generated due to unsupported language" (No se pudo generar un archivo Dockerfile debido a un lenguaje no admitido)
Azure Dev Spaces proporciona compatibilidad nativa para C# y Node.js. Al ejecutar `azds prep` en un directorio que contiene código escrito en uno de estos lenguajes, Azure Dev Spaces crea automáticamente un documento Dockerfile adecuado.

Podrá seguir usando Azure Dev Spaces con código escrito en otros lenguajes, pero deberá crear el documento Dockerfile manualmente antes de ejecutar `azds up` por primera vez.

Si la aplicación se escribe en un lenguaje que Azure Dev Spaces no admite de forma nativa, deberá proporcionar un documento Dockerfile apropiado para compilar una imagen de contenedor que ejecute el código. Docker ofrece una [lista de procedimientos recomendados para escribir un Dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/), así como una [referencia sobre Dockerfile](https://docs.docker.com/engine/reference/builder/) con la que le resultará más fácil escribir un Dockerfile que se ajuste a sus necesidades.

Una vez que disponga del documento Dockerfile adecuado, utilice `azds up` para ejecutar la aplicación en Azure Dev Spaces.

## <a name="common-issues-when-starting-or-stopping-services-with-azure-dev-spaces"></a>Problemas comunes al iniciar o detener servicios con Azure Dev Spaces

### <a name="error-config-file-not-found"></a>Error "Config file not found:" (Archivo de configuración no encontrado:)

Este error puede aparecer al ejecutar `azds up`. Tanto `azds up` como `azds prep` deben ejecutarse desde el directorio raíz del proyecto que desea ejecutar en el espacio de desarrollo.

Para corregir este problema:
1. Cambie el directorio actual a la carpeta raíz que contiene el código de servicio. 
1. Si no tiene un archivo _azds.yaml_ en la carpeta de código, ejecute `azds prep` para generar los recursos de Docker, Kubernetes y Azure Dev Spaces.

### <a name="timeout-at-waiting-for-container-image-build-step-with-aks-virtual-nodes"></a>Con los nodos virtuales de AKS, se agota el tiempo de espera en el paso "Waiting for container image build..." (Esperando la compilación de la imagen de contenedor...)

Este problema se produce cuando intenta usar Dev Spaces para ejecutar un servicio que está configurado para funcionar en un [nodo virtual de AKS](../aks/virtual-nodes-portal.md). Dev Spaces no admite actualmente la compilación o la depuración de servicios en nodos virtuales.

Si ejecuta `azds up` con el modificador `--verbose` o habilita el registro detallado en Visual Studio, verá más detalles:

```cmd
azds up --verbose

Installed chart in 2s
Waiting for container image build...
pods/mywebapi-76cf5f69bb-lgprv: Scheduled: Successfully assigned default/mywebapi-76cf5f69bb-lgprv to virtual-node-aci-linux
Streaming build container logs for service 'mywebapi' failed with: Timed out after 601.3037572 seconds trying to start build logs streaming operation. 10m 1s
Container image build failed
```

El comando anterior muestra que el pod del servicio se ha asignado a *virtual-node-aci-linux*, que es un nodo virtual.

Para corregir este problema, actualice el gráfico de Helm del servicio a fin de quitar cualquier valor *nodeSelector* o *tolerations* que permita que el servicio se ejecute en un nodo virtual. Estos valores se definen normalmente en el archivo `values.yaml` del gráfico.

Podrá seguir usando un clúster de AKS con la característica de nodos virtuales habilitada si el servicio que quiere compilar o depurar mediante Dev Spaces se ejecuta en un nodo de máquina virtual. La configuración predeterminada ejecuta un servicio con Dev Spaces en un nodo de máquina virtual.

### <a name="error-could-not-find-a-ready-tiller-pod-when-launching-dev-spaces"></a>Error "could not find a ready tiller pod" (no se pudo encontrar un pod de Tiller listo) al iniciar Dev Spaces

Este error se produce si el cliente de Helm ya no puede comunicarse con el pod Tiller que se está ejecutando en el clúster.

Para corregir este problema, reinicie los nodos del agente en el clúster.

### <a name="error-release-azds-identifier-spacename-servicename-failed-services-servicename-already-exists-or-pull-access-denied-for-servicename-repository-does-not-exist-or-may-require-docker-login"></a>Error "release azds-\<identifier\>-\<spacename\>-\<servicename\> failed: services '\<servicename\>' already exists" ("Se produjo un error en release azds-\<identifier\>-\<spacename\>-\<servicename\>: los servicios '\<servicename\>' ya existen") o "Pull access denied for \<servicename\>, repository does not exist or may require 'docker login'" ("Se denegó el acceso de extracción para \<servicename\>. El repositorio no existe o es posible que requiera iniciar sesión en Docker").

Estos errores se pueden producir si combina comandos directos de Helm (como `helm install`, `helm upgrade` o `helm delete`) con comandos de Dev Spaces (como `azds up` y `azds down`) en el mismo espacio de desarrollo. Se producen porque Dev Spaces dispone de su propia instancia de Tiller que entrará en conflicto con su propia instancia de Tiller que se ejecuta en el mismo espacio de desarrollo.

Se pueden usar comandos de Helm y comandos de Dev Spaces en el mismo clúster de AKS, pero cada espacio de nombres habilitado para Dev Spaces debería usar unos u otros.

Por ejemplo, suponga que utiliza un comando Helm para ejecutar toda la aplicación en un espacio de desarrollo primario. Puede crear espacios de desarrollo secundarios fuera de ese espacio primario, usar Dev Spaces para ejecutar servicios individuales en los espacios de desarrollo secundarios y probar los servicios juntos. Cuando esté listo para insertar los cambios, utilice un comando de Helm para implementar el código actualizado en el espacio de desarrollo primario. No use `azds up` para ejecutar el servicio actualizado en el espacio de desarrollo primario ya que entrará en conflicto con el servicio que se ejecutó inicialmente mediante Helm.

### <a name="existing-dockerfile-not-used-to-build-a-container"></a>No se ha usado un documento Dockerfile existente para compilar un contenedor

Azure Dev Spaces puede configurarse para que apunte a un _Dockerfile_ específico en el proyecto. Si cree que Azure Dev Spaces no usa el _Dockerfile_ esperado para compilar los contenedores, podría tener que indicar explícitamente a Azure Dev Spaces dónde se encuentra. 

Para corregir este problema, abra el archivo _azds.yaml_ que Azure Dev Spaces generó en el proyecto. Actualice *configurations: develop: build: dockerfile* para que apunte al Dockerfile que quiere usar. Por ejemplo:

```yaml
...
configurations:
  develop:
    build:
      dockerfile: Dockerfile.develop
```

### <a name="error-unauthorized-authentication-required-when-trying-to-use-a-docker-image-from-a-private-registry"></a>Error "unauthorized: authentication required" (no autorizado: autenticación necesaria) al intentar usar una imagen de Docker de un registro privado

Está usando una imagen de Docker de un registro privado que requiere autenticación.

Para corregir este problema, puede permitir que Dev Spaces se autentique en este registro privado y extraiga imágenes de él mediante [imagePullSecrets](https://kubernetes.io/docs/concepts/configuration/secret/#using-imagepullsecrets). Para usar imagePullSecrets, [cree un secreto de Kubernetes](https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod) en el espacio de nombres donde se usa la imagen. Después, proporcione el secreto como imagePullSecret en `azds.yaml`.

A continuación se muestra un ejemplo de cómo especificar imagePullSecrets en `azds.yaml`.

```yaml
kind: helm-release
apiVersion: 1.1
build:
  context: $BUILD_CONTEXT$
  dockerfile: Dockerfile
install:
  chart: $CHART_DIR$
  values:
  - values.dev.yaml?
  - secrets.dev.yaml?
  set:
    # Optional, specify an array of imagePullSecrets. These secrets must be manually created in the namespace.
    # This will override the imagePullSecrets array in values.yaml file.
    # If the dockerfile specifies any private registry, the imagePullSecret for the registry must be added here.
    # ref: https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
    #
    # This uses credentials from secret "myRegistryKeySecretName".
    imagePullSecrets:
      - name: myRegistryKeySecretName
```

> [!IMPORTANT]
> Al establecer imagePullSecrets en `azds.yaml`, se invalida el imagePullSecrets especificado en `values.yaml`.

### <a name="error-service-cannot-be-started"></a>Error "No se puede iniciar el servicio"

Es posible que vea este error cuando el código del servicio no se inicia. La causa suele estar en el código de usuario. Para más información de diagnóstico, habilite un registro más detallado al iniciar el servicio.

En la línea de comandos, use `--verbose` para habilitar un registro más detallado. También puede especificar un formato de salida mediante `--output`. Por ejemplo:

```cmd
azds up --verbose --output json
```

En Visual Studio:

1. Abra **Herramientas > Opciones** y, en **Proyectos y soluciones**, elija **Build and Run** (Compilar y ejecutar).
2. Cambie la configuración de **Detalles de la salida de la compilación del proyecto de MSBuild** a **Detallado** o **Diagnóstico**.

    ![Captura de pantalla del cuadro de diálogo Opciones de herramientas](media/common/VerbositySetting.PNG)

### <a name="rerunning-a-service-after-controller-re-creation"></a>Nueva ejecución de un servicio después de volver a crear el controlador

Puede que reciba el error*El servicio no se puede iniciar* cuando intenta volver a ejecutar un servicio después de haber quitado y luego creado de nuevo el controlador de Azure Dev Spaces asociado a este clúster. En esta situación, la salida detallada contiene el texto siguiente:

```output
Installing Helm chart...
Release "azds-33d46b-default-webapp1" does not exist. Installing it now.
Error: release azds-33d46b-default-webapp1 failed: services "webapp1" already exists
Helm install failed with exit code '1': Release "azds-33d46b-default-webapp1" does not exist. Installing it now.
Error: release azds-33d46b-default-webapp1 failed: services "webapp1" already exists
```

Este error se debe a que al quitar el controlador de Dev Spaces no se quitan los servicios instalados anteriormente por ese controlador. Se produce un error al volver a crear el controlador y luego intentar ejecutar los servicios mediante el nuevo controlador porque los servicios antiguos todavía están en su lugar.

Para solucionar este problema, use el comando `kubectl delete` para quitar manualmente los servicios antiguos del clúster y, después, vuelva a ejecutar Dev Spaces para instalar los nuevos servicios.

### <a name="error-service-cannot-be-started-when-using-multi-stage-dockerfiles"></a>Error "No se puede iniciar el servicio" al usar Dockerfiles de varias fases

Al usar un Dockerfile de varias fases, recibirá el error *El servicio no se puede iniciar*. En esta situación, la salida detallada contiene el texto siguiente:

```cmd
$ azds up -v
Using dev space 'default' with target 'AksClusterName'
Synchronizing files...6s
Installing Helm chart...2s
Waiting for container image build...10s
Building container image...
Step 1/12 : FROM [imagename:tag] AS base
Error parsing reference: "[imagename:tag] AS base" is not a valid repository/tag: invalid reference format
Failed to build container image.
Service cannot be started.
```

Este error se produce porque Azure Dev Spaces no admite actualmente compilaciones en varias fases. Para evitar compilaciones de varias fases, vuelva a escribir el Dockerfile.

### <a name="network-traffic-is-not-forwarded-to-your-aks-cluster-when-connecting-your-development-machine"></a>El tráfico no se reenvía al clúster de AKS cuando se conecta la máquina de desarrollo.

Al usar [Azure Dev Spaces para conectar el clúster de AKS a la máquina de desarrollo](https://code.visualstudio.com/docs/containers/local-process-kubernetes), es posible que se produzca una incidencia que provoque que no se reenvíe el tráfico entre la máquina de desarrollo y el clúster de AKS.

Al conectar el equipo de desarrollo al clúster de AKS, Azure Dev Spaces reenvía el tráfico de red entre el clúster de AKS y el equipo de desarrollo modificando el archivo `hosts` de la máquina de desarrollo. Azure Dev Spaces crea una entrada en `hosts` con la dirección del servicio Kubernetes que se va a reemplazar como nombre de host. Esta entrada se usa con el reenvío de puertos para dirigir el tráfico entre la máquina de desarrollo y el clúster de AKS. Si un servicio de la máquina de desarrollo entra en conflicto con el puerto del servicio Kubernetes que se está reemplazando, Azure Dev Spaces no puede reenviar el tráfico para el servicio Kubernetes. Por ejemplo, el servicio *Windows BranchCache*  normalmente se enlaza a *0.0.0.0:80*, lo que provocará un conflicto en el puerto 80 de todas las direcciones IP locales.

Para corregir esta incidencia, debe detener los servicios o procesos que entren en conflicto con el puerto del servicio Kubernetes que se intenta reemplazar. Puede usar herramientas, como *netstat*, para inspeccionar los servicios o procesos de la máquina de desarrollo que están en conflicto.

Por ejemplo, para detener y deshabilitar el servicio *Windows BranchCache*, haga lo siguiente:
* Ejecute `services.msc` desde el símbolo del sistema.
* Haga clic con el botón derecho en *BranchCache* y seleccione *Propiedades*.
* Haga clic en *Detener*.
* De forma opcional, puede deshabilitarlo estableciendo la opción *Tipo de inicio* en *Deshabilitado*.
* Haga clic en *OK*.

### <a name="error-no-azureassignedidentity-found-for-podazdsazds-webhook-deployment-id-in-assigned-state"></a>Error "no AzureAssignedIdentity found for pod:azds/azds-webhook-deployment-\<id\> in assigned state" (No se encontró ninguna identidad AzureAssignedIdentity para pod:azds/azds-webhook-deployment-\<id\> en el estado asignado).

Al ejecutar un servicio con Azure Dev Spaces en un clúster de AKS con una [identidad administrada](../aks/use-managed-identity.md) e [identidades administradas del pod](../aks/developer-best-practices-pod-security.md#use-pod-managed-identities) instaladas, el proceso puede dejar de responder después del paso de *instalación del gráfico*. Si inspecciona el *azds-injector-webhook* en el espacio de nombres *azds*, es posible que vea este error.

Los servicios que Azure Dev Spaces ejecuta en el clúster usan la identidad administrada del clúster para comunicarse con los servicios de back-end de Azure Dev Spaces fuera del clúster. Cuando se instala la identidad administrada del pod, las reglas de redes se configuran en los nodos del clúster a fin de redirigir todas las llamadas para las credenciales de identidad administrada a un [DaemonSet de Identidad administrada del nodo (NMI) en el clúster](https://github.com/Azure/aad-pod-identity#node-managed-identity). Este DaemonSet de NMI identifica el pod que realiza la llamada y garantiza que este se ha etiquetado correctamente para tener acceso a la identidad administrada solicitada. Azure Dev Spaces no puede detectar si un clúster tiene instalada la identidad administrada del pod y no puede realizar la configuración necesaria para permitir a los servicios Azure Dev Spaces obtener acceso a la identidad administrada del clúster. Dado que los servicios Azure Dev Spaces no se han configurado para tener acceso a la identidad administrada del clúster, el DaemonSet de NMI no les permitirá obtener un token de Azure AD para la identidad administrada y no podrá comunicarse con los servicios de back-end de Azure Dev Spaces.

Para corregir este problema, aplique una [AzurePodIdentityException](https://github.com/Azure/aad-pod-identity/blob/master/docs/readmes/README.app-exception.md) para el *azds-injector-webhook* y actualice los pods instrumentados por Azure Dev Spaces para tener acceso a la identidad administrada.

Cree un archivo denominado *webhookException.yaml* y copie la siguiente definición de código YAML:

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzurePodIdentityException
metadata:
  name: azds-infrastructure-exception
  namespace: azds
spec:
  PodLabels:
    azds.io/uses-cluster-identity: "true"
```

El archivo anterior crea un objeto *AzurePodIdentityException* para el *azds-injector-webhook*. Para implementar este objeto, use `kubectl`:

```cmd
kubectl apply -f webhookException.yaml
```

Para actualizar los pods instrumentados por Azure Dev Spaces para tener acceso a la identidad administrada, actualice el *espacio de nombres* en la siguiente definición de código YAML y use `kubectl` a fin de aplicarla para cada espacio de desarrollo.

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzurePodIdentityException
metadata:
  name: azds-infrastructure-exception
  namespace: myNamespace
spec:
  PodLabels:
    azds.io/instrumented: "true"
```

Como alternativa, puede crear los objetos *AzureIdentity* y *AzureIdentityBinding* y actualizar las etiquetas del pod para cargas de trabajo que se ejecutan en espacios instrumentados por Azure Dev Spaces para tener acceso a la identidad administrada creada por el clúster de AKS.

Para mostrar los detalles de la identidad administrada, ejecute el siguiente comando para el clúster de AKS:

```azurecli
az aks show -g <resourcegroup> -n <cluster> -o json --query "{clientId: identityProfile.kubeletidentity.clientId, resourceId: identityProfile.kubeletidentity.resourceId}"
```

El comando anterior genera los identificadores *clientId* y *resourceId* para la identidad administrada. Por ejemplo:

```json
{
  "clientId": "<clientId>",
  "resourceId": "/subscriptions/<subid>/resourcegroups/<resourcegroup>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<name>"
}
```

Para crear un objeto *AzureIdentity*, cree un archivo denominado *clusteridentity.yaml* y use la siguiente definición de YAML actualizada con los detalles de la identidad administrada del comando anterior:

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzureIdentity
metadata:
  name: my-cluster-mi
spec:
  type: 0
  ResourceID: /subscriptions/<subid>/resourcegroups/<resourcegroup>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<name>
  ClientID: <clientId>
```

Para crear un objeto *AzureIdentityBinding*, cree un archivo denominado *clusteridentitybinding.yaml* y use la siguiente definición de código YAML:

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzureIdentityBinding
metadata:
  name: my-cluster-mi-binding
spec:
  AzureIdentity: my-cluster-mi
  Selector: my-label-value
```

Para implementar los objetos *AzureIdentity* y *AzureIdentityBinding*, use `kubectl`:

```cmd
kubectl apply -f clusteridentity.yaml
kubectl apply -f clusteridentitybinding.yaml
```

Después de implementar los objetos *AzureIdentity* y *AzureIdentityBinding*, cualquier carga de trabajo con la etiqueta *aadpodidbinding: my-label-value* puede tener acceso a la identidad administrada del clúster. Agregue esta etiqueta y vuelva a implementar todas las cargas de trabajo que se ejecutan en cualquier espacio de desarrollo. Por ejemplo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: sample
        aadpodidbinding: my-label-value
    spec:
      [...]
```

## <a name="common-issues-using-visual-studio-and-visual-studio-code-with-azure-dev-spaces"></a>Problemas comunes al usar Visual Studio y Visual Studio Code con Azure Dev Spaces

### <a name="error-required-tools-and-configurations-are-missing"></a>Error "Required tools and configurations are missing" (Faltan herramientas y configuraciones necesarias)

Este error puede producirse al iniciar VS Code: "[Azure Dev Spaces] Faltan las herramientas y configuraciones necesarias para compilar y depurar "[nombre de proyecto]"".
El error significa que azds.exe no está en la variable de entorno PATH, tal como se muestra en VS Code.

Intente iniciar VS Code desde un símbolo del sistema, con la variable de entorno PATH establecida correctamente.

### <a name="error-required-tools-to-build-and-debug-projectname-are-out-of-date"></a>Error "Las herramientas necesarias para compilar y depurar 'nombreDeProyecto' están desactualizadas".

Verá este error en Visual Studio Code si tiene una versión más reciente de la extensión de VS Code para Azure Dev Spaces, pero una versión más antigua de la CLI de Azure Dev Spaces.

Intente descargar e instalar la versión más reciente de la CLI de Azure Dev Spaces:

* [Windows](https://aka.ms/get-azds-windows)
* [Mac](https://aka.ms/get-azds-mac)
* [Linux](https://aka.ms/get-azds-linux)

### <a name="error-failed-to-find-debugger-extension-for-typecoreclr"></a>Error: "Failed to find debugger extension for type:coreclr" (No se pudo encontrar la extensión del depurador para type:coreclr)

Es posible que vea este error al ejecutar el depurador de Visual Studio Code. Puede que no tenga la extensión de VS Code para C# instalada en la máquina de desarrollo. La extensión de C# incluye compatibilidad de depuración para .Net Core (CoreCLR).

Para corregir este problema, instale la [extensión de VS Code para C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).

### <a name="error-configured-debug-type-coreclr-is-not-supported"></a>Error "El tipo de depuración 'coreclr' configurado no es compatible"

Es posible que vea este error al ejecutar el depurador de Visual Studio Code. Puede que no tenga la extensión de VS Code para Azure Dev Spaces instalada en la máquina de desarrollo.

Para corregir este problema, instale la extensión de VS Code para Azure Dev Spaces.

### <a name="error-invalid-cwd-value-src-the-system-cannot-find-the-file-specified-or-launch-program-srcpath-to-project-binary-does-not-exist"></a>Error "Valor 'cwd' no válido '/src'. El sistema no encuentra el archivo especificado." o "launch: program '/src/[ruta de acceso al archivo binario del proyecto]' no existe"

Es posible que vea este error al ejecutar el depurador de Visual Studio Code. De forma predeterminada, la extensión de VS Code utiliza `src` como directorio de trabajo para el proyecto en el contenedor. Si ha actualizado `Dockerfile` para especificar un directorio de trabajo diferente, puede que vea este error.

Para corregir este problema, actualice el archivo `launch.json` en el subdirectorio `.vscode` de su carpeta de proyecto. Cambie la directiva `configurations->cwd` para que apunte al mismo directorio que el `WORKDIR` definido en el `Dockerfile` del proyecto. También es posible que deba actualizar la directiva `configurations->program`.

### <a name="error-the-pipe-program-azds-exited-unexpectedly-with-code-126"></a>Error "El programa de canalización 'azds' se cerró de forma inesperada con el código 126"

Es posible que vea este error al ejecutar el depurador de Visual Studio Code.

Para corregir este problema, cierre y vuelva a abrir Visual Studio Code. Reinicie el depurador.

### <a name="error-internal-watch-failed-watch-enospc-when-attaching-debugging-to-a-nodejs-application"></a>Error "Internal watch failed: watch ENOSPC" (Error durante la inspección interna: inspeccione ENOSPC) al asociar depuración a una aplicación de Node.js

Este error se produce cuando el nodo que ejecuta el pod con la aplicación Node.js que está intentando asociar a un depurador ha superado el valor *fs.inotify.max_user_watches*. En algunos casos, [el valor predeterminado de *fs.inotify.max_user_watches* puede ser demasiado pequeño para controlar la asociación de un depurador directamente a un pod](https://github.com/Azure/AKS/issues/772).

Una solución temporal para resolver este problema consiste en aumentar el valor de *fs.inotify.max_user_watches* en cada nodo del clúster y reiniciar ese nodo para que los cambios surtan efecto.

## <a name="other-common-issues"></a>Otros problemas comunes

### <a name="error-azds-is-not-recognized-as-an-internal-or-external-command-operable-program-or-batch-file"></a>Error "azds no se reconoce como un comando interno o externo, programa o archivo por lotes ejecutable"

Este error puede producirse si `azds.exe` no está instalado o configurado correctamente.

Para corregir este problema:

1. Busque `azds.exe` en la ubicación %ProgramFiles%/Microsoft SDKs\Azure\Azure Dev Spaces CLI. Si está ahí, agregue esa ubicación a la variable de entorno PATH.
2. Si `azds.exe` no está instalado, ejecute el siguiente comando:

    ```azurecli
    az aks use-dev-spaces -n <cluster-name> -g <resource-group>
    ```

### <a name="authorization-error-microsoftdevspacesregisteraction"></a>Error de autorización "Microsoft.DevSpaces/register/action"

Necesita acceso de *propietario* o *colaborador* en su suscripción de Azure para administrar Azure Dev Spaces. Puede ver un error de autorización si intenta administrar Dev Spaces y no tiene acceso como *Propietario* o *Colaborador* a la suscripción de Azure asociada. Por ejemplo:

```output
The client '<User email/Id>' with object id '<Guid>' does not have authorization to perform action 'Microsoft.DevSpaces/register/action' over scope '/subscriptions/<Subscription Id>'.
```

Para corregir este problema, use una cuenta con acceso como *Propietario* o *Colaborador* a la suscripción de Azure, y registre manualmente el espacio de nombres `Microsoft.DevSpaces`:

```azurecli
az provider register --namespace Microsoft.DevSpaces
```

### <a name="new-pods-arent-starting"></a>No se inician los nuevos pods

El inicializador de Kubernetes no puede aplicar PodSpec a los nuevos pods debido a cambios en los permisos de RBAC en el rol *cluster-admin* del clúster. El nuevo pod también puede tener un PodSpec no válido, por ejemplo si la cuenta de servicio asociada con el pod ya no existe. Para ver los pods que están en un estado *pendiente* debido a algún problema del inicializador, use el comando `kubectl get pods`:

```bash
kubectl get pods --all-namespaces --include-uninitialized
```

Este problema puede afectar a los pods de *todos los espacios de nombres* del clúster incluidos aquellos espacios en los que Azure Dev Spaces no está habilitado.

Para corregir este problema, [actualice la CLI de Dev Spaces a la versión más reciente](./how-to/upgrade-tools.md#update-the-dev-spaces-cli-extension-and-command-line-tools) y, después, elimine *azds InitializerConfiguration* del controlador de Azure Dev Spaces:

```azurecli
az aks get-credentials --resource-group <resource group name> --name <cluster name>
```

```bash
kubectl delete InitializerConfiguration azds
```

Una vez que haya eliminado *azds InitializerConfiguration* del controlador de Azure Dev Spaces, use `kubectl delete` para eliminar todos los pods con estado *pendiente*. Una vez que se hayan eliminado todos los pods pendientes, vuelva a implementarlos.

Si los nuevos pods aún siguen en el estado *pendiente* después de una reimplementación, use `kubectl delete` para eliminar todos los *pods* con ese estado. Una vez que se hayan eliminado todos los pods pendientes, elimine el controlador del clúster y vuelva a instalarlo:

```bash
azds remove -g <resource group name> -n <cluster name>
azds controller create --name <cluster name> -g <resource group name> -tn <cluster name>
```

Una vez reinstalado el controlador, vuelva a implementar los pods.

### <a name="incorrect-rbac-permissions-for-calling-dev-spaces-controller-and-apis"></a>Permisos incorrectos de RBAC para llamar al controlador y las API de Dev Spaces

El usuario que accede al controlador de Azure Dev Spaces debe tener acceso de lectura al archivo *kubeconfig* de administración en el clúster de AKS. Por ejemplo, este permiso está disponible en el [rol de administrador del clúster integrado de Azure Kubernetes Service](../aks/control-kubeconfig-access.md#available-cluster-roles-permissions). El usuario que accede al controlador de Azure Dev Spaces debe tener también el rol de Azure de *Colaborador* o *Propietario* del controlador. Encontrará más detalles sobre cómo actualizar los permisos de un usuario para un clúster de AKS [aquí](../aks/control-kubeconfig-access.md#assign-role-permissions-to-a-user-or-group).

Para actualizar el rol de Azure del usuario para el controlador:

1. Inicie sesión en Azure Portal en https://portal.azure.com.
1. Vaya al grupo de recursos que contiene el controlador, que suele ser el mismo que el del clúster de AKS.
1. Marque la casilla *Mostrar tipos ocultos*.
1. Haga clic en el controlador.
1. Abra el panel *Access Control (IAM)* .
1. Haga clic en la pestaña *Asignaciones de roles*.
1. Haga clic en *Agregar* y, a continuación, en *Agregar asignación de roles*.
    * Para *Rol*, seleccione *Colaborador* o *Propietario*.
    * En *Asignar acceso a*, seleccione *Usuario, grupo o entidad de servicio de Azure AD*.
    * En *Seleccionar*, busque el usuario al que desea conceder permisos.
1. Haga clic en *Save*(Guardar).

### <a name="dns-name-resolution-fails-for-a-public-url-associated-with-a-dev-spaces-service"></a>Se produce un error en la resolución de nombres DNS para una dirección URL pública asociada con un servicio Dev Spaces

Puede configurar un punto de conexión de dirección URL pública para el servicio si especifica el modificador `--enable-ingress` para el comando `azds prep` o si activa la casilla `Publicly Accessible` en Visual Studio. El nombre DNS público se registra automáticamente cuando el servicio se ejecuta en Dev Spaces. Si no está registrado, verá un error *No se puede mostrar la página* o *No se puede acceder a este sitio* en el explorador web cuando se conecte a la dirección URL pública.

Para corregir este problema:

* Compruebe el estado de todas las direcciones URL asociadas a los servicios de Dev Spaces:

  ```console
  azds list-uris
  ```

* Si una dirección URL está en estado *Pendiente*, Dev Spaces aún está esperando a que se complete el registro DNS. A veces, el proceso de registro tarda unos minutos en completarse. Dev Spaces también abre un túnel de host local para cada servicio, que puede usar mientras espera el registro del DNS.
* Si una dirección URL permanece en estado *Pendiente* durante más de 5 minutos, puede que indique un problema con el pod de DNS externo que crea al punto de conexión público o con el pod controlador de entrada nginx que adquiere el punto de conexión público. Use los siguientes comandos para eliminar estos pods y permitir que AKS los vuelva a crear automáticamente:
  ```console
  kubectl delete pod -n kube-system -l app=addon-http-application-routing-external-dns
  kubectl delete pod -n kube-system -l app=addon-http-application-routing-nginx-ingress
  ```

### <a name="error-upstream-connect-error-or-disconnectreset-before-headers"></a>Error "upstream connect error or disconnect/reset before headers" (Error de conexión en dirección ascendente o desconexión o restablecimiento antes de los encabezados)

Puede ver este error al intentar acceder al servicio. Por ejemplo, cuando vaya a la dirección URL del servicio en un explorador. Este error indica que el puerto del contenedor no está disponible. Esto puede deberse a los siguientes motivos:

* El contenedor está todavía en proceso de creación e implementación. Este problema puede producirse si ejecuta `azds up` o inicia el depurador y luego intenta acceder al contenedor antes de que se haya implementado correctamente.
* La configuración del puerto no es coherente en _Dockerfile_, el gráfico de Helm y cualquier otro código de servidor que abre un puerto.

Para corregir este problema:

1. Si el contenedor está en proceso de creación o implementación, puede esperar entre dos y tres segundos e intentar acceder al servicio de nuevo. 
1. Compruebe la configuración del puerto en los siguientes recursos:
    * **[Gráfico de Helm](https://docs.helm.sh):** Especificado en `service.port` y `deployment.containerPort` en values.yaml con scaffolding con el comando `azds prep`.
    * Los puertos abiertos en el código de aplicación, por ejemplo, en Node.js: `var server = app.listen(80, function () {...}`

### <a name="the-type-or-namespace-name-mylibrary-couldnt-be-found"></a>No se encontró el tipo o el nombre del espacio de nombres "MyLibrary"

No se encuentra un proyecto de biblioteca que está usando. Con Dev Spaces, el contexto de compilación se sitúa de forma predeterminada en el nivel de proyecto o servicio.  

Para corregir este problema:

1. Modifique el archivo `azds.yaml` para establecer el contexto de compilación en el nivel de la solución.
2. Modifique los archivos `Dockerfile` y `Dockerfile.develop` para que hagan referencia a los archivos del proyecto, por ejemplo, `.csproj`, correctamente en relación con el nuevo contexto de compilación.
3. Agregue un archivo `.dockerignore` en el mismo directorio que el archivo `.sln`.
4. Actualice el archivo `.dockerignore` con entradas adicionales según sea necesario.

Puede ver un ejemplo [aquí](https://github.com/sgreenmsft/buildcontextsample).

### <a name="horizontal-pod-autoscaling-not-working-in-a-dev-space"></a>La escalabilidad horizontal del pod no funciona en un espacio de desarrollo

Al ejecutar un servicio en un espacio de desarrollo, se insertan en el pod del servicio [contenedores adicionales para la instrumentación](how-dev-spaces-works-cluster-setup.md#prepare-your-aks-cluster). Además, todos los contenedores de un pod deben tener solicitudes y límites de recursos establecidos para el escalado automático horizontal del pod.

Para corregir este problema, aplique una solicitud y un límite de recursos a los contenedores de Dev Spaces insertados. Las solicitudes y límites de recursos se pueden aplicar al contenedor en el que se ha producido la inserción (devspaces-proxy) mediante la incorporación de la anotación `azds.io/proxy-resources` a la especificación del pod. El valor debe establecerse en un objeto JSON que representa la sección de recursos de la especificación del contenedor del proxy.

A continuación se muestra un ejemplo de una anotación de recursos de proxy que se va a aplicar a la especificación del pod.
```
azds.io/proxy-resources: "{\"Limits\": {\"cpu\": \"300m\",\"memory\": \"400Mi\"},\"Requests\": {\"cpu\": \"150m\",\"memory\": \"200Mi\"}}"
```

### <a name="enable-azure-dev-spaces-on-an-existing-namespace-with-running-pods"></a>Habilitación de Azure Dev Spaces en un espacio de nombres existente con pods en ejecución

Es posible que ya tenga un clúster de AKS y un espacio de nombres con pods en ejecución en los que desee habilitar Azure Dev Spaces.

Para habilitar Azure Dev Spaces en un espacio de nombres existente de un clúster de AKS, ejecute `use-dev-spaces` y use `kubectl` para reiniciar todos los pods de dicho espacio de nombres.

```azurecli
az aks get-credentials --resource-group MyResourceGroup --name MyAKS
az aks use-dev-spaces -g MyResourceGroup -n MyAKS --space my-namespace --yes
```

```console
kubectl -n my-namespace delete pod --all
```

Una vez que se han reiniciado los pods, puede empezar a usar el espacio de nombres existente con Azure Dev Spaces.

### <a name="enable-azure-dev-spaces-on-aks-cluster-with-restricted-egress-traffic-for-cluster-nodes"></a>Habilitación de Azure Dev Spaces en el clúster de AKS con tráfico de salida restringido para los nodos de clúster

Para habilitar Azure Dev Spaces en un clúster de AKS con el fin de que el tráfico de salida de los nodos de clúster esté restringido, tendrá que permitir los FQDN siguientes:

| FQDN                                    | Port      | Uso      |
|-----------------------------------------|-----------|----------|
| cloudflare.docker.com | HTTPS:443 | Extraer Linux Alpine y otras imágenes de Azure Dev Spaces |
| gcr.io | HTTP:443 | Extraer las imágenes de Helm o Tiller|
| storage.googleapis.com | HTTP:443 | Extraer las imágenes de Helm o Tiller|

Actualice el firewall o la configuración de seguridad para permitir el tráfico de red hacia y desde todos los FQDN anteriores y los [servicios de infraestructura de Azure Dev Spaces](../dev-spaces/configure-networking.md#virtual-network-or-subnet-configurations).

### <a name="error-could-not-find-the-cluster-cluster-in-subscription-subscriptionid"></a>Error "Could not find the cluster \<cluster\>cluster\<subscriptionId\> in subscription subscriptionId" ("No se encontró el clúster \<cluster\>clúster\<subscriptionId\> en la suscripción subscriptionId")

Es posible que vea este error si el archivo kubeconfig tiene como destino un clúster o una suscripción diferente de la que está intentando usar con las herramientas del lado cliente de Azure Dev Spaces. Las herramientas del lado cliente de Azure Dev Spaces replican el comportamiento de *kubectl*, que usa [uno o varios archivos kubeconfig](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) para seleccionar y comunicarse con el clúster.

Para corregir este problema:

* Use `az aks use-dev-spaces -g <resource group name> -n <cluster name>` para actualizar el contexto actual. Este comando también habilita Azure Dev Spaces en el clúster de AKS si aún no está habilitado. También puede usar `kubectl config use-context <cluster name>` como alternativa para actualizar el contexto actual.
* Use `az account show` para mostrar la suscripción de Azure actual que quiere emplear como destino y compruebe que es la correcta. Puede cambiar la suscripción que desea emplear mediante `az account set`.

### <a name="error-using-dev-spaces-after-rotating-aks-certificates"></a>Error al usar Dev Spaces después de rotar los certificados de AKS

Después de [rotar los certificados en el clúster de AKS](../aks/certificate-rotation.md), se producirá un error en ciertas operaciones, como `azds space list` y `azds up`. También debe actualizar los certificados en el controlador de Azure Dev Spaces después de rotar los certificados en el clúster.

Para corregir este problema, asegúrese de que *kubeconfig* tenga los certificados actualizados mediante `az aks get-credentials` y, a continuación, ejecute el comando `azds controller refresh-credentials`. Por ejemplo:

```azurecli
az aks get-credentials -g <resource group name> -n <cluster name>
```

```console
azds controller refresh-credentials -g <resource group name> -n <cluster name>
```
