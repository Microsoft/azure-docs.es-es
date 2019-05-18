---
title: Instalación de Istio en Azure Kubernetes Service (AKS)
description: Obtenga información sobre cómo instalar y usar Istio para crear una malla de servicio en un clúster de Azure Kubernetes Service (AKS).
services: container-service
author: paulbouwer
ms.service: container-service
ms.topic: article
ms.date: 04/19/2019
ms.author: pabouwer
ms.openlocfilehash: 12565d2b8004a5119add25473e5b088c9162035f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780500"
---
# <a name="install-and-use-istio-in-azure-kubernetes-service-aks"></a>Instalación y uso de Istio en Azure Kubernetes Service (AKS)

[Istio][istio-github] es una malla de servicio de código abierto que proporciona un conjunto clave de funcionalidades en los microservicios de un clúster de Kubernetes. Estas características incluyen la administración del tráfico, la seguridad e identidad de servicio, el cumplimiento de directivas y la observación. Para más información sobre Istio, vea la documentación del sitio oficial [What is Istio?][istio-docs-concepts] (¿Qué es Istio?).

En este artículo se explica cómo instalar Istio. El Istio `istioctl` binarios de cliente se instala en el equipo cliente y los componentes de Istio se instalan en un clúster de Kubernetes en AKS.

> [!NOTE]
> Estas instrucciones hacen referencia Istio versión `1.1.3`.
>
> El Istio `1.1.x` versiones se han probado por el equipo de Istio frente a las versiones de Kubernetes `1.11`, `1.12`, `1.13`. Puede encontrar versiones adicionales de Istio en [GitHub - libera Istio] [ istio-github-releases] e información sobre cada una de las versiones en [Istio - notas de la versión] [ istio-release-notes].

En este artículo, aprenderá a:

> [!div class="checklist"]
> * Descargar Istio
> * Instalar al cliente de istioctl Istio binario
> * Instale el CRD Istio en AKS
> * Instale los componentes de Istio en AKS
> * Validar la instalación de Istio
> * Acceso a complementos
> * Desinstalar Istio de AKS

## <a name="before-you-begin"></a>Antes de empezar

Los pasos detallados en este artículo se suponen que ha creado un clúster de AKS (Kubernetes `1.11` y versiones posteriores, con RBAC habilitado) y que ha establecido un `kubectl` conexión con el clúster. Si necesita ayuda con cualquiera de estos elementos, vea el [inicio rápido de AKS][aks-quickstart].

Necesitará [Helm] [ helm] siga estas instrucciones e instale Istio. Se recomienda que tiene la versión `2.12.2` o posterior correctamente instalado y configurado en el clúster. Si necesita ayuda con la instalación de Helm, consulte el [instrucciones de instalación de Helm AKS][helm-install].

En este artículo, las instrucciones de instalación de Istio se dividen en varios pasos discretos. El resultado final presenta la misma estructura que las [instrucciones][istio-install-helm] de instalación oficiales de Istio.

## <a name="download-istio"></a>Descargar Istio

En primer lugar, descargue y extraiga la versión más reciente de Istio. Los pasos son un poco diferentes para un shell de bash en MacOS, Linux o el subsistema de Windows para Linux y para un shell de PowerShell. Elija uno de los siguientes pasos de instalación que coincida con el entorno que prefiera:

* [Bash en MacOS, Linux o un subsistema de Windows para Linux](#bash)
* [PowerShell](#powershell)

### <a name="bash"></a>Bash

En MacOS, utilice `curl` para descargar la versión más reciente de Istio y, a continuación, extraiga `tar` como sigue:

```bash
# Specify the Istio version that will be leveraged throughout these instructions
ISTIO_VERSION=1.1.3

# MacOS
curl -sL "https://github.com/istio/istio/releases/download/$ISTIO_VERSION/istio-$ISTIO_VERSION-osx.tar.gz" | tar xz
```

En Linux o en el subsistema de Windows para Linux, use `curl` para descargar la versión más reciente de Istio y, a continuación, extraiga `tar` como sigue:

```bash
# Specify the Istio version that will be leveraged throughout these instructions
ISTIO_VERSION=1.1.3

curl -sL "https://github.com/istio/istio/releases/download/$ISTIO_VERSION/istio-$ISTIO_VERSION-linux.tar.gz" | tar xz
```

Ahora pasar a la sección para [instalar el cliente de istioctl Istio binario](#install-the-istio-istioctl-client-binary).

### <a name="powershell"></a>PowerShell

En PowerShell, use `Invoke-WebRequest` para descargar la versión más reciente de Istio y, a continuación, extraer con `Expand-Archive` como sigue:

```powershell
# Specify the Istio version that will be leveraged throughout these instructions
$ISTIO_VERSION="1.1.3"

# Windows
$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -URI "https://github.com/istio/istio/releases/download/$ISTIO_VERSION/istio-$ISTIO_VERSION-win.zip" -OutFile "istio-$ISTIO_VERSION.zip"
Expand-Archive -Path "istio-$ISTIO_VERSION.zip" -DestinationPath .
```

Ahora pasar a la sección para [instalar el cliente de istioctl Istio binario](#install-the-istio-istioctl-client-binary).

## <a name="install-the-istio-istioctl-client-binary"></a>Instalar al cliente de istioctl Istio binario

> [!IMPORTANT]
> Asegúrese de ejecutar los pasos de esta sección, desde la carpeta de nivel superior de la versión de Istio que ha descargado y extraído.

El `istioctl` binario de cliente que se ejecuta en el equipo cliente y le permite interactuar con la malla de servicio Istio. Los pasos de instalación son un poco diferentes entre los sistemas operativos de cliente. Elija uno de los siguientes pasos de instalación que coincida con el entorno que prefiera:

* [macOS](#macos)
* [Linux o el subsistema de Windows para Linux](#linux-or-windows-subsystem-for-linux)
* [Windows](#windows)

### <a name="macos"></a>MacOS

Para instalar el binario de cliente `istioctl` de Istio en un shell basado en Bash en MacOS, use los siguientes comandos. Estos comandos copian el binario de cliente `istioctl` en la ubicación del programa de usuario estándar en su `PATH`.

```bash
cd istio-$ISTIO_VERSION
sudo cp ./bin/istioctl /usr/local/bin/istioctl
sudo chmod +x /usr/local/bin/istioctl
```

Si desea completar la línea de comandos del binario de cliente `istioctl` de Istio, realice la siguiente configuración:

```bash
# Generate the bash completion file and source it in your current shell
mkdir -p ~/completions && istioctl collateral --bash -o ~/completions
source ~/completions/istioctl.bash

# Source the bash completion file in your .bashrc so that the command-line completions
# are permanently available in your shell
echo "source ~/completions/istioctl.bash" >> ~/.bashrc
```

Avanzar a la siguiente sección para [instalar el CRD Istio en AKS](#install-the-istio-crds-on-aks).

### <a name="linux-or-windows-subsystem-for-linux"></a>Linux o Subsistema de Windows para Linux

Use los comandos siguientes para instalar el binario de cliente `istioctl` de Istio en un shell basado en Bash en Linux o en el [Subsistema de Windows para Linux][install-wsl]. Estos comandos copian el binario de cliente `istioctl` en la ubicación del programa de usuario estándar en su `PATH`.

```bash
cd istio-$ISTIO_VERSION
sudo cp ./bin/istioctl /usr/local/bin/istioctl
sudo chmod +x /usr/local/bin/istioctl
```

Si desea completar la línea de comandos del binario de cliente `istioctl` de Istio, realice la siguiente configuración:

```bash
# Generate the bash completion file and source it in your current shell
mkdir -p ~/completions && istioctl collateral --bash -o ~/completions
source ~/completions/istioctl.bash

# Source the bash completion file in your .bashrc so that the command-line completions
# are permanently available in your shell
echo "source ~/completions/istioctl.bash" >> ~/.bashrc
```

Avanzar a la siguiente sección para [instalar el CRD Istio en AKS](#install-the-istio-crds-on-aks).

### <a name="windows"></a> Windows

Para instalar el Istio `istioctl` cliente binario en un **Powershell**-basada en shell de Windows, use los siguientes comandos. Estos comandos copian el `istioctl` cliente binario a una carpeta Istio y ponerlo a disposición de forma permanente a través de su `PATH`. No necesita privilegios elevados de (Admin) para ejecutar estos comandos.

```powershell
cd istio-$ISTIO_VERSION
New-Item -ItemType Directory -Force -Path "C:\Istio"
Copy-Item -Path .\bin\istioctl.exe -Destination "C:\Istio\"
$PATH = [environment]::GetEnvironmentVariable("PATH", "User")
[environment]::SetEnvironmentVariable("PATH", $PATH + "; C:\Istio\", "User")
```

Avanzar a la siguiente sección para [instalar el CRD Istio en AKS](#install-the-istio-crds-on-aks).

## <a name="install-the-istio-crds-on-aks"></a>Instale el CRD Istio en AKS

> [!IMPORTANT]
> Asegúrese de ejecutar los pasos de esta sección, desde la carpeta de nivel superior de la versión de Istio que ha descargado y extraído.

Usa Istio [definiciones del recurso personalizado (crd)] [ kubernetes-crd] para administrar su configuración en tiempo de ejecución. Es necesario instalar el CRD Istio en primer lugar, puesto que los componentes de Istio tienen una dependencia en ellos. Uso de Helm y `istio-init` gráfico para instalar el CRD Istio en el `istio-system` espacio de nombres en el clúster de AKS:

```azurecli
helm install install/kubernetes/helm/istio-init --name istio-init --namespace istio-system
```

[Trabajos] [ kubernetes-jobs] se implementan como parte de la `istio-init` gráfico de Helm para instalar el CRD. Estos trabajos tardará entre 1 y 2 minutos en completarse, según el entorno de clúster. Puede comprobar que los trabajos han completado correctamente como sigue:

```azurecli
kubectl get jobs -n istio-system
```

La salida de ejemplo siguiente muestra los trabajos completados correctamente.

```console
NAME                COMPLETIONS   DURATION   AGE
istio-init-crd-10   1/1           16s        18s
istio-init-crd-11   1/1           15s        18s
```

Ahora que hemos confirmado la finalización correcta de los trabajos, vamos a comprobar que tenemos el número correcto de CRD Istio instalado. Puede comprobar que se han instalado todos los 53 CRD de Istio ejecutando el comando apropiado para su entorno. El comando debe devolver el número `53`.

Bash

```bash
kubectl get crds | grep 'istio.io' | wc -l
```

Powershell

```powershell
(kubectl get crds | Select-String -Pattern 'istio.io').Count
```

Si tienes a este punto, significa que se han instalado correctamente el CRD Istio. Avanzar a la siguiente sección para [instalar los componentes de Istio en AKS](#install-the-istio-components-on-aks).

## <a name="install-the-istio-components-on-aks"></a>Instale los componentes de Istio en AKS

> [!IMPORTANT]
> Asegúrese de ejecutar los pasos de esta sección, desde la carpeta de nivel superior de la versión de Istio que ha descargado y extraído.

Se va a instalar [Grafana] [ grafana] y [Kiali] [ kiali] como parte de nuestra instalación Istio. Grafana proporciona análisis y paneles de supervisión y Kiali proporciona un panel de observación de malla de servicio. En nuestra configuración, cada uno de estos componentes requiere credenciales que se deben proporcionar como un [secreto][kubernetes-secrets].

Antes de que se puede instalar los componentes de Istio, debemos crear los secretos de Grafana y Kiali. Cree estos secretos ejecutando los comandos adecuados para su entorno.

### <a name="add-grafana-secret"></a>Adición del secreto de Grafana

Reemplace el `REPLACE_WITH_YOUR_SECURE_PASSWORD` el token con la contraseña y ejecute los siguientes comandos:

#### <a name="macos-linux"></a>MacOS, Linux

```bash
GRAFANA_USERNAME=$(echo -n "grafana" | base64)
GRAFANA_PASSPHRASE=$(echo -n "REPLACE_WITH_YOUR_SECURE_PASSWORD" | base64)

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: grafana
  namespace: istio-system
  labels:
    app: grafana
type: Opaque
data:
  username: $GRAFANA_USERNAME
  passphrase: $GRAFANA_PASSPHRASE
EOF
```

#### <a name="windows"></a> Windows

```powershell
$GRAFANA_USERNAME=[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("grafana"))
$GRAFANA_PASSPHRASE=[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("REPLACE_WITH_YOUR_SECURE_PASSWORD"))

"apiVersion: v1
kind: Secret
metadata:
  name: grafana
  namespace: istio-system
  labels:
    app: grafana
type: Opaque
data:
  username: $GRAFANA_USERNAME
  passphrase: $GRAFANA_PASSPHRASE" | kubectl apply -f -
```

### <a name="add-kiali-secret"></a>Agregar secreto Kiali

Reemplace el `REPLACE_WITH_YOUR_SECURE_PASSWORD` el token con la contraseña y ejecute los siguientes comandos:

#### <a name="macos-linux"></a>MacOS, Linux

```bash
KIALI_USERNAME=$(echo -n "kiali" | base64)
KIALI_PASSPHRASE=$(echo -n "REPLACE_WITH_YOUR_SECURE_PASSWORD" | base64)

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: istio-system
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE
EOF
```

#### <a name="windows"></a> Windows

```powershell
$KIALI_USERNAME=[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("kiali"))
$KIALI_PASSPHRASE=[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("REPLACE_WITH_YOUR_SECURE_PASSWORD"))

"apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: istio-system
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE" | kubectl apply -f -
```

### <a name="install-istio-components"></a>Instalar componentes de Istio

Ahora que hemos creado correctamente los secretos de Grafana y Kiali en el clúster AKS, es momento de instalar los componentes de Istio. Uso de Helm y `istio` gráfico para instalar los componentes de Istio en el `istio-system` espacio de nombres en el clúster de AKS. Utilice los comandos adecuados para su entorno.

> [!NOTE]
> Estamos usando como parte de la instalación de las siguientes opciones:
> - `global.controlPlaneSecurityEnabled=true` -TLS mutua habilitada para el plano de control
> - `mixer.adapters.useAdapterCRDs=false` -quitar inspecciones en el adaptador de mixer CRD, ya que en desuso y esto mejorará el rendimiento
> - `grafana.enabled=true` -habilitar la implementación de Grafana para análisis y paneles de supervisión
> - `grafana.security.enabled=true` -habilitar la autenticación de Grafana
> - `tracing.enabled=true` -habilitar la implementación de Jaeger para el seguimiento
> - `kiali.enabled=true` -habilitar la implementación de Kiali para un panel de observación de malla de servicio

Bash

```bash
helm install install/kubernetes/helm/istio --name istio --namespace istio-system \
  --set global.controlPlaneSecurityEnabled=true \
  --set mixer.adapters.useAdapterCRDs=false \
  --set grafana.enabled=true --set grafana.security.enabled=true \
  --set tracing.enabled=true \
  --set kiali.enabled=true
```

Powershell

```powershell
helm install install/kubernetes/helm/istio --name istio --namespace istio-system `
  --set global.controlPlaneSecurityEnabled=true `
  --set mixer.adapters.useAdapterCRDs=false `
  --set grafana.enabled=true --set grafana.security.enabled=true `
  --set tracing.enabled=true `
  --set kiali.enabled=true
```

El `istio` gráfico de Helm implementa un gran número de objetos. Puede ver la lista de la salida de su `helm install` comando anterior. La implementación de los componentes de Istio puede tardar 4 y 5 minutos en completarse, según el entorno de clúster.

En este momento, ha implementado Istio a su clúster AKS. Para asegurarse de que tenemos una implementación correcta de Istio, pasemos a la siguiente sección para [validar la instalación de Istio](#validate-the-istio-installation).

## <a name="validate-the-istio-installation"></a>Validar la instalación de Istio

En primer lugar, confirme que se han creado los servicios esperados. Use el comando [kubectl get svc][kubectl-get] para ver los servicios en ejecución. Consulta el `istio-system` espacio de nombres, que se han instalado los componentes Istio y un complemento mediante el `istio` gráfico de Helm:

```console
kubectl get svc --namespace istio-system --output wide
```

La salida del ejemplo siguiente muestra los servicios que ahora deben estar en ejecución:

- `istio-*` servicios
- `jaeger-*`, `tracing`, y `zipkin` servicios de seguimiento del complemento
- `prometheus` servicio de métricas de complemento
- `grafana` análisis de complemento y el servicio de panel de supervisión
- `kiali` servicio de panel de malla de servicio adicional

Si `istio-ingressgateway` muestra una dirección IP externa de `<pending>`, espere unos minutos hasta que las redes de Azure hayan asignado una dirección IP.

```console
NAME                     TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                                                                                                                                      AGE       SELECTOR
grafana                  ClusterIP      10.0.81.182    <none>          3000/TCP                                                                                                                                     119s      app=grafana
istio-citadel            ClusterIP      10.0.96.33     <none>          8060/TCP,15014/TCP                                                                                                                           119s      istio=citadel
istio-galley             ClusterIP      10.0.237.158   <none>          443/TCP,15014/TCP,9901/TCP                                                                                                                   119s      istio=galley
istio-ingressgateway     LoadBalancer   10.0.154.12    20.188.211.19   15020:30603/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:31198/TCP,15030:30610/TCP,15031:30937/TCP,15032:31344/TCP,15443:31499/TCP   119s      app=istio-ingressgateway,istio=ingressgateway,release=istio
istio-pilot              ClusterIP      10.0.178.56    <none>          15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                                       119s      istio=pilot
istio-policy             ClusterIP      10.0.116.118   <none>          9091/TCP,15004/TCP,15014/TCP                                                                                                                 119s      istio-mixer-type=policy,istio=mixer
istio-sidecar-injector   ClusterIP      10.0.31.160    <none>          443/TCP                                                                                                                                      119s      istio=sidecar-injector
istio-telemetry          ClusterIP      10.0.187.246   <none>          9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                                       119s      istio-mixer-type=telemetry,istio=mixer
jaeger-agent             ClusterIP      None           <none>          5775/UDP,6831/UDP,6832/UDP                                                                                                                   119s      app=jaeger
jaeger-collector         ClusterIP      10.0.116.63    <none>          14267/TCP,14268/TCP                                                                                                                          119s      app=jaeger
jaeger-query             ClusterIP      10.0.22.108    <none>          16686/TCP                                                                                                                                    119s      app=jaeger
kiali                    ClusterIP      10.0.142.50    <none>          20001/TCP                                                                                                                                    119s      app=kiali
prometheus               ClusterIP      10.0.138.134   <none>          9090/TCP                                                                                                                                     119s      app=prometheus
tracing                  ClusterIP      10.0.165.210   <none>          80/TCP                                                                                                                                       118s      app=jaeger
zipkin                   ClusterIP      10.0.126.211   <none>          9411/TCP                                                                                                                                     118s      app=jaeger
```

A continuación, confirme que se han creado los pods necesarios. Use la [kubectl get pods] [ kubectl-get] comando y vuelva a consultar el `istio-system` espacio de nombres:

```console
kubectl get pods --namespace istio-system
```

La salida de ejemplo siguiente muestra los pods que se ejecutan:

- el `istio-*` pods
- el `prometheus-*` pod de métricas de complemento
- el `grafana-*` pod del panel supervisión y análisis de complemento
- el `kiali` pod del panel de malla de servicio de complemento

```console
NAME                                     READY     STATUS      RESTARTS   AGE
grafana-88779954d-nzpm7                  1/1       Running     0          6m26s
istio-citadel-7f699dc8c8-n7q8g           1/1       Running     0          6m26s
istio-galley-649bc8cd97-wfjzm            1/1       Running     0          6m26s
istio-ingressgateway-65dfbd566-42wkn     1/1       Running     0          6m26s
istio-init-crd-10-tmtw5                  0/1       Completed   0          20m38s
istio-init-crd-11-ql25l                  0/1       Completed   0          20m38s
istio-pilot-958dd8cc4-4ckf9              2/2       Running     0          6m26s
istio-policy-86b4b7cf9-zf7v7             2/2       Running     4          6m26s
istio-sidecar-injector-d48786c5c-pmrj9   1/1       Running     0          6m26s
istio-telemetry-7f6996fdcc-84w94         2/2       Running     3          6m26s
istio-tracing-79db5954f-h7hmz            1/1       Running     0          6m26s
kiali-5c4cdbb869-s28dv                   1/1       Running     0          6m26s
prometheus-67599bf55b-pgxd8              1/1       Running     0          6m26s
```

Debe haber dos `istio-init-crd-*` pods con un `Completed` estado. Estos pods eran responsables de ejecutar los trabajos que creó el CRD en un paso anterior. Todos los otros pods deben mostrar el estado de `Running`. Si los pods no tienen estos estados, espere un minuto o dos hasta que los tengan. Si algún pod notifica un problema, use el comando [kubectl describe pod][kubectl-describe] para revisar su salida y su estado.

## <a name="accessing-the-add-ons"></a>Acceso a complementos

Se instalaron una serie de complementos en Istio en los pasos de instalación anteriores que ofrecen una funcionalidad adicional. Las interfaces de usuario de los complementos no se exponen de forma pública a través de una dirección IP externa. Para acceder a las interfaces de usuario de los complementos, use el comando [kubectl port-forward][kubectl-port-forward]. Este comando crea una conexión segura entre el equipo cliente y el pod relevante en el clúster de AKS.

Hemos agregado una capa adicional de seguridad de Grafana y Kiali especificando credenciales para ellos anteriormente en este artículo.

### <a name="grafana"></a>Grafana

Proporcionan el análisis y paneles de supervisión para Istio [Grafana][grafana]. Reenviar el puerto local `3000` en el equipo cliente al puerto `3000` en el pod que se está ejecutando Grafana en el clúster de AKS:

```console
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000
```

La salida del ejemplo siguiente muestra el reenvío de puerto configurado para Grafana:

```console
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```

Ahora puede acceder a Grafana en la siguiente dirección URL en el equipo cliente: [http://localhost:3000](http://localhost:3000). Recuerde que debe usar las credenciales que creó mediante la Grafana secreta anteriormente cuando se le solicite.

### <a name="prometheus"></a>Prometheus

Proporciona métricas para Istio [Prometheus][prometheus]. Reenviar el puerto local `9090` en el equipo cliente al puerto `9090` en el pod que se está ejecutando Prometheus en el clúster de AKS:

```console
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') 9090:9090
```

La salida del ejemplo siguiente muestra el reenvío de puerto configurado para Prometheus:

```console
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090
```

Ahora puede acceder al explorador de expresiones de Prometheus en la siguiente dirección URL en el equipo cliente: [http://localhost:9090](http://localhost:9090).

### <a name="jaeger"></a>Jaeger

Proporciona seguimiento dentro de Istio [Jaeger][jaeger]. Reenviar el puerto local `16686` en el equipo cliente al puerto `16686` en el pod que se está ejecutando Jaeger en el clúster de AKS:

```console
kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 16686:16686
```

La salida del ejemplo siguiente muestra el reenvío de puerto configurado para Jaeger:

```console
Forwarding from 127.0.0.1:16686 -> 16686
Forwarding from [::1]:16686 -> 16686
```

Ahora puede acceder a la interfaz de usuario de seguimiento de Jaeger en la siguiente dirección URL en el equipo cliente: [http://localhost:16686](http://localhost:16686).

### <a name="kiali"></a>Kiali

[Kiali][kiali] proporciona un panel de observación de malla de servicio. Reenviar el puerto local `20001` en el equipo cliente al puerto `20001` en el pod que se está ejecutando Kiali en el clúster de AKS:

```console
kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001
```

La salida del ejemplo siguiente muestra el reenvío de puerto configurado para Kiali:

```console
Forwarding from 127.0.0.1:20001 -> 20001
Forwarding from [::1]:20001 -> 20001
```

Ahora puede acceder al panel de observación de malla de servicio de Kiali en la siguiente dirección URL en el equipo cliente: [http://localhost:20001/kiali/console/](http://localhost:20001/kiali/console/). Recuerde que debe usar las credenciales que creó mediante la Kiali secreta anteriormente cuando se le solicite.

## <a name="uninstall-istio-from-aks"></a>Desinstalar Istio de AKS

> [!WARNING]
> Eliminando Istio desde un sistema en ejecución puede producir tráfico relacionado con problemas entre los servicios. Asegúrese de que ha realizado disposiciones para que el sistema seguirá funcionando correctamente sin Istio antes de continuar.

### <a name="remove-istio-components-and-namespace"></a>Quitar espacio de nombres y componentes Istio

Para quitar el clúster de AKS Istio, use los siguientes comandos. El `helm delete` comandos quitarán el `istio` y `istio-init` gráficos y el `kubectl delete ns` comando quitará la `istio-system` espacio de nombres.

```azurecli
helm delete --purge istio
helm delete --purge istio-init
kubectl delete ns istio-system
```

### <a name="remove-istio-crds"></a>Quitar CRD Istio

Los comandos anteriores eliminan todos los componentes de Istio y espacio de nombres, pero todavía seguimos teniendo el CRD Istio. Para eliminar el CRD, puede usar uno los siguientes enfoques.

Método #1: este comando se da por supuesto que está ejecutando desde la carpeta de nivel superior de la versión descargada y extraída de Istio que usó para instalar Istio con este paso.

```azure-cli
kubectl delete -f install/kubernetes/helm/istio-init/files
```

Método #2: Use uno de estos comandos si ya no tiene acceso a la versión descargado y extraído de Istio que usó para instalar Istio con. Este comando tardará un poco más - tardará unos minutos en completarse.

Bash
```bash
kubectl get crds -o name | grep 'istio.io' | xargs -n1 kubectl delete
```

Powershell
```powershell
kubectl get crds -o name | Select-String -Pattern 'istio.io' |% { kubectl delete $_ }
```

## <a name="next-steps"></a>Pasos siguientes

La siguiente documentación describe cómo puede usar Istio para proporcionar enrutamiento inteligente para implantar un lanzamiento controlado:

> [!div class="nextstepaction"]
> [Escenario de enrutamiento inteligente con Istio en AKS][istio-scenario-routing]

Para explorar más opciones de instalación y configuración para Istio, consulte los siguientes artículos oficiales de Istio:

- [Istio: guía de instalación de Helm][istio-install-helm]
- [Istio: opciones de instalación de Helm][istio-install-helm-options]

Puede consultar otros escenarios con el [ejemplo de la aplicación Bookinfo en Istio][istio-bookinfo-example].

Para obtener información sobre cómo supervisar la aplicación de AKS con Application Insights y Istio, consulte la siguiente documentación de Azure Monitor:
- [Cero supervisión de aplicaciones de instrumentación para Kubernetes las aplicaciones hospedadas][app-insights]

<!-- LINKS - external -->
[istio]: https://istio.io
[helm]: https://helm.sh

[istio-docs-concepts]: https://istio.io/docs/concepts/what-is-istio/
[istio-github]: https://github.com/istio/istio
[istio-github-releases]: https://github.com/istio/istio/releases
[istio-release-notes]: https://istio.io/about/notes/
[istio-install-download]: https://istio.io/docs/setup/kubernetes/download-release/
[istio-install-helm]: https://istio.io/docs/setup/kubernetes/install/helm/
[istio-install-helm-options]: https://istio.io/docs/reference/config/installation-options/
[istio-bookinfo-example]: https://istio.io/docs/examples/bookinfo/
[install-wsl]: https://docs.microsoft.com/windows/wsl/install-win10

[kubernetes-crd]: https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions
[kubernetes-jobs]: https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/
[kubernetes-secrets]: https://kubernetes.io/docs/concepts/configuration/secret/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-port-forward]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward

[grafana]: https://grafana.com/
[prometheus]: https://prometheus.io/
[jaeger]: https://www.jaegertracing.io/
[kiali]: https://www.kiali.io/

[app-insights]: https://docs.microsoft.com/azure/azure-monitor/app/kubernetes

<!-- LINKS - internal -->
[aks-quickstart]: ./kubernetes-walkthrough.md
[istio-scenario-routing]: ./istio-scenario-routing.md
[helm-install]: ./kubernetes-helm.md