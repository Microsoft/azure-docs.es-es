---
title: Restringir el tráfico de salida en Azure Kubernetes Service (AKS)
description: Obtenga información sobre qué puertos y direcciones son necesarios para controlar el tráfico de salida en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.author: jpalma
ms.date: 06/29/2020
ms.custom: fasttrack-edit
author: palma21
ms.openlocfilehash: 51b457b99afc478631ce9b39a4a7d51ffd57401c
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88003182"
---
# <a name="control-egress-traffic-for-cluster-nodes-in-azure-kubernetes-service-aks"></a>Control del tráfico de salida de los nodos de clúster en Azure Kubernetes Service (AKS)

En este artículo se proporcionan los detalles necesarios que permiten proteger el tráfico de salida desde Azure Kubernetes Service (AKS). Contiene los requisitos del clúster para una implementación de AKS base y requisitos adicionales para características y complementos opcionales. [Al final, se proporciona un ejemplo de cómo configurar estos requisitos con Azure Firewall](#restrict-egress-traffic-using-azure-firewall). Pero puede aplicar esta información a cualquier método o dispositivo de restricción de salida.

## <a name="background"></a>Información previa

Los clústeres de AKS se implementan en una red virtual. Esta red puede ser administrada (creada por AKS) o personalizada (configurada previamente por el usuario). En cualquier caso, el clúster tiene dependencias de **salida** en servicios externos a esa red virtual (el servicio no tiene dependencias de entrada).

Para fines operativos y de administración, los nodos de un clúster de AKS deben tener acceso a determinados puertos y nombres de dominio completo (FQDN). Estos puntos de conexión son obligatorios para que los nodos se comuniquen con el servidor de API, o bien para descargar e instalar actualizaciones de seguridad de nodos y componentes principales de clúster de Kubernetes. Por ejemplo, el clúster debe extraer imágenes de contenedor del sistema base desde el Registro de contenedor de Microsoft (MCR).

Las dependencias de salida de AKS se definen casi por completo mediante FQDN, que no tienen direcciones estáticas tras ellos. La falta de direcciones estáticas significa que no se pueden usar grupos de seguridad de red para bloquear el tráfico saliente desde un clúster de AKS. 

De forma predeterminada, los clústeres de AKS tienen acceso de salida a Internet ilimitado. Este nivel de acceso a la red permite que los nodos y servicios que ejecuta accedan a recursos externos según sea necesario. Si desea restringir el tráfico de salida, es necesario el acceso a un número limitado de puertos y direcciones para mantener las tareas de mantenimiento del clúster en buen estado. La solución más sencilla para proteger las direcciones de salida consiste en usar un dispositivo de firewall que pueda controlar el tráfico saliente en función de los nombres de dominio. Azure Firewall, por ejemplo, puede restringir el tráfico saliente HTTP y HTTPS en función del FQDN de destino. También puede configurar las reglas de seguridad y de firewall que prefiera para permitir estos puertos y direcciones necesarios.

> [!IMPORTANT]
> En este documento solo se explica cómo bloquear el tráfico que sale de la subred de AKS. De forma predeterminada, AKS no tiene requisitos de entrada.  No se admite el bloqueo del **tráfico de subred interno** mediante grupos de seguridad de red (NSG) y firewalls. Para controlar y bloquear el tráfico dentro del clúster, use [***directivas de red***][network-policy].

## <a name="required-outbound-network-rules-and-fqdns-for-aks-clusters"></a>Reglas de red de salida y FQDN necesarios para clústeres de AKS

Las siguientes reglas de red y FQDN o aplicación son obligatorias para un clúster de AKS; puede usarlas si quiere configurar una solución que no sea Azure Firewall.

* Las dependencias de dirección IP son para tráfico que no sea HTTP/HTTPS (tráfico TCP y UDP).
* Los puntos de conexión HTTP/HTTPS de FQDN se pueden colocar en el dispositivo de firewall.
* Los puntos de conexión HTTP/HTTPS de carácter comodín son dependencias que pueden variar con el clúster de AKS en función de una serie de calificadores.
* AKS usa un controlador de admisión para insertar el FQDN como una variable de entorno en todas las implementaciones en kube-system y gatekeeper-system, lo que garantiza que toda la comunicación del sistema entre los nodos y el servidor de API usa el FQDN del servidor de API y no su dirección IP. 
* Si tiene una aplicación o solución que necesita comunicarse con el servidor de API, debe agregar una regla de red **adicional** para permitir la *comunicación TCP con el puerto 443 de la dirección IP del servidor de API*.
* En raras ocasiones, si hay una operación de mantenimiento, es posible que cambie la dirección IP del servidor de API. Las operaciones de mantenimiento planeado que pueden cambiar la dirección IP del servidor de API siempre se comunican con antelación.


### <a name="azure-global-required-network-rules"></a>Reglas de red obligatorias globales de Azure

Las reglas de red obligatorias y las dependencias de dirección IP son las siguientes:

| Punto de conexión de destino                                                             | Protocolo | Port    | Uso  |
|----------------------------------------------------------------------------------|----------|---------|------|
| **`*:1194`** <br/> *O* <br/> [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:1194`** <br/> *O* <br/> [CIDR regionales](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:1194`** <br/> *O* <br/> **`APIServerIP:1194`** `(only known after cluster creation)`  | UDP           | 1194      | Para la comunicación segura por túnel entre los nodos y el plano de control. |
| **`*:9000`** <br/> *O* <br/> [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:9000`** <br/> *O* <br/> [CIDR regionales](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:9000`** <br/> *O* <br/> **`APIServerIP:9000`** `(only known after cluster creation)`  | TCP           | 9000      | Para la comunicación segura por túnel entre los nodos y el plano de control. |
| **`*:123`** o **`ntp.ubuntu.com:123`** (si usa reglas de red Azure Firewall)  | UDP      | 123     | Obligatorio para la sincronización de hora del protocolo de tiempo de red (NTP) en nodos de Linux.                 |
| **`CustomDNSIP:53`** `(if using custom DNS servers)`                             | UDP      | 53      | Si usa servidores DNS personalizados, debe asegurarse de que sean accesibles para los nodos de clúster. |
| **`APIServerIP:443`** `(if running pods/deployments that access the API Server)` | TCP      | 443     | Obligatorio si se ejecutan pods o implementaciones que acceden al servidor de API, en cuyo caso usarían la dirección IP de la API.  |

### <a name="azure-global-required-fqdn--application-rules"></a>Reglas de aplicación o FQDN obligatorias globales de Azure 

Se requieren las siguientes reglas de aplicación / FQDN:

| FQDN de destino                 | Port            | Uso      |
|----------------------------------|-----------------|----------|
| **`*.hcp.<location>.azmk8s.io`** | **`HTTPS:443`** | Obligatorio para la comunicación entre el nodo y el servidor de API. Reemplace *\<location\>* con la región en la que está implementado el clúster de AKS. |
| **`mcr.microsoft.com`**          | **`HTTPS:443`** | Obligatorio para acceder a las imágenes del Registro de contenedor de Microsoft (MCR). Este registro contiene imágenes o gráficos propios (por ejemplo, coreDNS, etc.) Estas imágenes son necesarias para la creación correcta y el funcionamiento del clúster, incluidas las operaciones de escalado y actualización.  |
| **`*.cdn.mscr.io`**              | **`HTTPS:443`** | Obligatorio para el almacenamiento de MCR respaldado por Azure Content Delivery Network (CDN). |
| **`*.data.mcr.microsoft.com`**   | **`HTTPS:443`** | Obligatorio para el almacenamiento de MCR respaldado por Azure Content Delivery Network (CDN). |
| **`management.azure.com`**       | **`HTTPS:443`** | Obligatorio para las operaciones de Kubernetes en la API de Azure. |
| **`login.microsoftonline.com`**  | **`HTTPS:443`** | Obligatorio para autenticación de Azure Active Directory. |
| **`packages.microsoft.com`**     | **`HTTPS:443`** | Esta dirección es el repositorio de paquetes de Microsoft que se usa para las operaciones *apt-get* almacenadas en caché.  Los paquetes de ejemplo incluyen Moby, PowerShell y la CLI de Azure. |
| **`acs-mirror.azureedge.net`**   | **`HTTPS:443`** | Esta dirección es para el repositorio necesario para descargar e instalar los archivos binarios necesarios, como kubenet y Azure CNI. |

### <a name="azure-china-21vianet-required-network-rules"></a>Reglas de red obligatorias para Azure China 21Vianet

Las reglas de red obligatorias y las dependencias de dirección IP son las siguientes:

| Punto de conexión de destino                                                             | Protocolo | Port    | Uso  |
|----------------------------------------------------------------------------------|----------|---------|------|
| **`*:1194`** <br/> *O* <br/> [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.Region:1194`** <br/> *O* <br/> [CIDR regionales](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:1194`** <br/> *O* <br/> **`APIServerIP:1194`** `(only known after cluster creation)`  | UDP           | 1194      | Para la comunicación segura por túnel entre los nodos y el plano de control. |
| **`*:9000`** <br/> *O* <br/> [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:9000`** <br/> *O* <br/> [CIDR regionales](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:9000`** <br/> *O* <br/> **`APIServerIP:9000`** `(only known after cluster creation)`  | TCP           | 9000      | Para la comunicación segura por túnel entre los nodos y el plano de control. |
| **`*:22`** <br/> *O* <br/> [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:22`** <br/> *O* <br/> [CIDR regionales](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:22`** <br/> *O* <br/> **`APIServerIP:22`** `(only known after cluster creation)`  | TCP           | 22      | Para la comunicación segura por túnel entre los nodos y el plano de control. |
| **`*:123`** o **`ntp.ubuntu.com:123`** (si usa reglas de red Azure Firewall)  | UDP      | 123     | Obligatorio para la sincronización de hora del protocolo de tiempo de red (NTP) en nodos de Linux.                 |
| **`CustomDNSIP:53`** `(if using custom DNS servers)`                             | UDP      | 53      | Si usa servidores DNS personalizados, debe asegurarse de que sean accesibles para los nodos de clúster. |
| **`APIServerIP:443`** `(if running pods/deployments that access the API Server)` | TCP      | 443     | Obligatorio si se ejecutan pods o implementaciones que acceden al servidor de API, en cuyo caso usarían la dirección IP de la API.  |

### <a name="azure-china-21vianet-required-fqdn--application-rules"></a>Reglas de aplicación o FQDN obligatorias para Azure China 21Vianet

Se requieren las siguientes reglas de aplicación / FQDN:

| FQDN de destino                               | Port            | Uso      |
|------------------------------------------------|-----------------|----------|
| **`*.hcp.<location>.cx.prod.service.azk8s.cn`**| **`HTTPS:443`** | Obligatorio para la comunicación entre el nodo y el servidor de API. Reemplace *\<location\>* con la región en la que está implementado el clúster de AKS. |
| **`*.tun.<location>.cx.prod.service.azk8s.cn`**| **`HTTPS:443`** | Obligatorio para la comunicación entre el nodo y el servidor de API. Reemplace *\<location\>* con la región en la que está implementado el clúster de AKS. |
| **`mcr.microsoft.com`**                        | **`HTTPS:443`** | Obligatorio para acceder a las imágenes del Registro de contenedor de Microsoft (MCR). Este registro contiene imágenes o gráficos propios (por ejemplo, coreDNS, etc.) Estas imágenes son necesarias para la creación correcta y el funcionamiento del clúster, incluidas las operaciones de escalado y actualización. |
| **`*.cdn.mscr.io`**                            | **`HTTPS:443`** | Obligatorio para el almacenamiento de MCR respaldado por Azure Content Delivery Network (CDN). |
| **`.data.mcr.microsoft.com`**                  | **`HTTPS:443`** | Obligatorio para el almacenamiento de MCR respaldado por Azure Content Delivery Network (CDN). |
| **`management.chinacloudapi.cn`**              | **`HTTPS:443`** | Obligatorio para las operaciones de Kubernetes en la API de Azure. |
| **`login.chinacloudapi.cn`**                   | **`HTTPS:443`** | Obligatorio para autenticación de Azure Active Directory. |
| **`packages.microsoft.com`**                   | **`HTTPS:443`** | Esta dirección es el repositorio de paquetes de Microsoft que se usa para las operaciones *apt-get* almacenadas en caché.  Los paquetes de ejemplo incluyen Moby, PowerShell y la CLI de Azure. |
| **`*.azk8s.cn`**                               | **`HTTPS:443`** | Esta dirección es para el repositorio necesario para descargar e instalar los archivos binarios necesarios, como kubenet y Azure CNI. |

### <a name="azure-us-government-required-network-rules"></a>Reglas de red obligatorias para Azure Gobierno de EE. UU.

Las reglas de red obligatorias y las dependencias de dirección IP son las siguientes:

| Punto de conexión de destino                                                             | Protocolo | Port    | Uso  |
|----------------------------------------------------------------------------------|----------|---------|------|
| **`*:1194`** <br/> *O* <br/> [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:1194`** <br/> *O* <br/> [CIDR regionales](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:1194`** <br/> *O* <br/> **`APIServerIP:1194`** `(only known after cluster creation)`  | UDP           | 1194      | Para la comunicación segura por túnel entre los nodos y el plano de control. |
| **`*:9000`** <br/> *O* <br/> [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureCloud.<Region>:9000`** <br/> *O* <br/> [CIDR regionales](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) -  **`RegionCIDRs:9000`** <br/> *O* <br/> **`APIServerIP:9000`** `(only known after cluster creation)`  | TCP           | 9000      | Para la comunicación segura por túnel entre los nodos y el plano de control. |
| **`*:123`** o **`ntp.ubuntu.com:123`** (si usa reglas de red Azure Firewall)  | UDP      | 123     | Obligatorio para la sincronización de hora del protocolo de tiempo de red (NTP) en nodos de Linux.                 |
| **`CustomDNSIP:53`** `(if using custom DNS servers)`                             | UDP      | 53      | Si usa servidores DNS personalizados, debe asegurarse de que sean accesibles para los nodos de clúster. |
| **`APIServerIP:443`** `(if running pods/deployments that access the API Server)` | TCP      | 443     | Obligatorio si se ejecutan pods o implementaciones que acceden al servidor de API, en cuyo caso usarían la dirección IP de la API.  |

### <a name="azure-us-government-required-fqdn--application-rules"></a>Reglas de aplicación o FQDN obligatorias para Azure Gobierno de EE. UU. 

Se requieren las siguientes reglas de aplicación / FQDN:

| FQDN de destino                                        | Port            | Uso      |
|---------------------------------------------------------|-----------------|----------|
| **`*.hcp.<location>.cx.aks.containerservice.azure.us`** | **`HTTPS:443`** | Obligatorio para la comunicación entre el nodo y el servidor de API. Reemplace *\<location\>* con la región en la que está implementado el clúster de AKS.|
| **`mcr.microsoft.com`**                                 | **`HTTPS:443`** | Obligatorio para acceder a las imágenes del Registro de contenedor de Microsoft (MCR). Este registro contiene imágenes o gráficos propios (por ejemplo, coreDNS, etc.) Estas imágenes son necesarias para la creación correcta y el funcionamiento del clúster, incluidas las operaciones de escalado y actualización. |
| **`*.cdn.mscr.io`**                                     | **`HTTPS:443`** | Obligatorio para el almacenamiento de MCR respaldado por Azure Content Delivery Network (CDN). |
| **`*.data.mcr.microsoft.com`**                          | **`HTTPS:443`** | Obligatorio para el almacenamiento de MCR respaldado por Azure Content Delivery Network (CDN). |
| **`management.usgovcloudapi.net`**                      | **`HTTPS:443`** | Obligatorio para las operaciones de Kubernetes en la API de Azure. |
| **`login.microsoftonline.us`**                          | **`HTTPS:443`** | Obligatorio para autenticación de Azure Active Directory. |
| **`packages.microsoft.com`**                            | **`HTTPS:443`** | Esta dirección es el repositorio de paquetes de Microsoft que se usa para las operaciones *apt-get* almacenadas en caché.  Los paquetes de ejemplo incluyen Moby, PowerShell y la CLI de Azure. |
| **`acs-mirror.azureedge.net`**                          | **`HTTPS:443`** | Esta dirección es para el repositorio necesario para instalar los archivos binarios necesarios, como kubenet y Azure CNI. |

## <a name="optional-recommended-fqdn--application-rules-for-aks-clusters"></a>Reglas opcionales de FQDN o aplicación para clústeres de AKS

Las siguientes reglas de aplicación o FQDN son opcionales, pero se recomiendan para clústeres de AKS:

| FQDN de destino                                                               | Port          | Uso      |
|--------------------------------------------------------------------------------|---------------|----------|
| **`security.ubuntu.com`, `azure.archive.ubuntu.com`, `changelogs.ubuntu.com`** | **`HTTP:80`** | Esta dirección permite a los nodos del clúster de Linux descargar las revisiones de seguridad y actualizaciones necesarias. |

Si decide bloquear o no permitir estos FQDN, los nodos solo recibirán actualizaciones del sistema operativo cuando realice una [actualización de imagen de nodo](node-image-upgrade.md) o una [actualización de clústeres](upgrade-cluster.md).

## <a name="gpu-enabled-aks-clusters"></a>Clústeres de AKS habilitados para GPU

### <a name="required-fqdn--application-rules"></a>Reglas de aplicación o FQDN obligatorias

Los clústeres de AKS que tienen GPU habilitado necesitan las reglas de aplicación / FQDN siguientes:

| FQDN de destino                        | Port      | Uso      |
|-----------------------------------------|-----------|----------|
| **`nvidia.github.io`**                  | **`HTTPS:443`** | Esta dirección se utiliza para la instalación y el funcionamiento correctos del controlador en nodos basados en GPU. |
| **`us.download.nvidia.com`**            | **`HTTPS:443`** | Esta dirección se utiliza para la instalación y el funcionamiento correctos del controlador en nodos basados en GPU. |
| **`apt.dockerproject.org`**             | **`HTTPS:443`** | Esta dirección se utiliza para la instalación y el funcionamiento correctos del controlador en nodos basados en GPU. |

## <a name="windows-server-based-node-pools"></a>Grupos de nodos basados en Windows Server 

### <a name="required-fqdn--application-rules"></a>Reglas de aplicación o FQDN obligatorias

Los grupos de nodos basados en Windows Server necesitan las reglas de aplicación/FQDN siguientes:

| FQDN de destino                                                           | Port      | Uso      |
|----------------------------------------------------------------------------|-----------|----------|
| **`onegetcdn.azureedge.net, go.microsoft.com`**                            | **`HTTPS:443`** | Para instalar archivos binarios relacionados con Windows |
| **`*.mp.microsoft.com, www.msftconnecttest.com, ctldl.windowsupdate.com`** | **`HTTP:80`**   | Para instalar archivos binarios relacionados con Windows |

## <a name="aks-addons-and-integrations"></a>Complementos e integraciones de AKS

### <a name="azure-monitor-for-containers"></a>Azure Monitor para contenedores

Hay dos opciones para proporcionar a los contenedores acceso a Azure Monitor; puede permitir [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags) de Azure Monitor, **o bien** proporcionar acceso a las reglas de aplicación y FQDN necesarias.

#### <a name="required-network-rules"></a>Reglas de red obligatorias

Se requieren las siguientes reglas de aplicación / FQDN:

| Punto de conexión de destino                                                             | Protocolo | Port    | Uso  |
|----------------------------------------------------------------------------------|----------|---------|------|
| [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureMonitor:443`**  | TCP           | 443      | Este punto de conexión se usa para enviar datos de métricas y registros a Azure Monitor y Log Analytics. |

#### <a name="required-fqdn--application-rules"></a>Reglas de aplicación o FQDN obligatorias

Los clústeres de AKS que tienen habilitado Azure Monitor para contenedores necesitan las reglas de aplicación / FQDN siguientes:

| FQDN                                    | Port      | Uso      |
|-----------------------------------------|-----------|----------|
| dc.services.visualstudio.com | **`HTTPS:443`**    | Este punto de conexión se usa para la telemetría de supervisión y las métricas mediante Azure Monitor. |
| *.ods.opinsights.azure.com    | **`HTTPS:443`**    | Azure Monitor usa este punto de conexión para la ingesta de datos de análisis de registros. |
| *.oms.opinsights.azure.com | **`HTTPS:443`** | omsagent usa este punto de conexión para autenticar el servicio de análisis de registros. |
| *.monitoring.azure.com | **`HTTPS:443`** | Este punto de conexión se usa para enviar datos de métricas a Azure Monitor. |

### <a name="azure-dev-spaces"></a>Azure Dev Spaces

Actualice el firewall o la configuración de seguridad para permitir el tráfico de red hacia y desde todos los FQDN siguientes y los [servicios de infraestructura de Azure Dev Spaces][dev-spaces-service-tags].

#### <a name="required-network-rules"></a>Reglas de red obligatorias

| Punto de conexión de destino                                                             | Protocolo | Port    | Uso  |
|----------------------------------------------------------------------------------|----------|---------|------|
| [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureDevSpaces`**  | TCP           | 443      | Este punto de conexión se usa para enviar datos de métricas y registros a Azure Monitor y Log Analytics. |

#### <a name="required-fqdn--application-rules"></a>Reglas de aplicación o FQDN obligatorias 

Los clústeres de AKS que tienen habilitado Azure Dev Spaces necesitan las reglas de aplicación/FQDN siguientes:

| FQDN                                    | Port      | Uso      |
|-----------------------------------------|-----------|----------|
| `cloudflare.docker.com` | **`HTTPS:443`** | Esta dirección se usa para extraer Linux Alpine y otras imágenes de Azure Dev Spaces |
| `gcr.io` | **`HTTPS:443`** | Esta dirección se usa para extraer imágenes de Helm/Tiller |
| `storage.googleapis.com` | **`HTTPS:443`** | Esta dirección se usa para extraer imágenes de Helm/Tiller |


### <a name="azure-policy-preview"></a>Azure Policy (versión preliminar)

> [!CAUTION]
> Algunas de las características siguientes se encuentran en versión preliminar.  Las sugerencias de este artículo están sujetas a cambio a medida que la característica avanza a la versión preliminar pública y a las fases de versiones futuras.

#### <a name="required-fqdn--application-rules"></a>Reglas de aplicación o FQDN obligatorias 

Los clústeres de AKS que tienen habilitado Azure Policy necesitan las reglas de aplicación / FQDN siguientes.

| FQDN                                          | Port      | Uso      |
|-----------------------------------------------|-----------|----------|
| **`gov-prod-policy-data.trafficmanager.net`** | **`HTTPS:443`** | Esta dirección se usa para el funcionamiento correcto de Azure Policy. (Actualmente en versión preliminar en AKS) |
| **`raw.githubusercontent.com`**               | **`HTTPS:443`** | Esta dirección se usa para extraer las directivas integradas de GitHub para garantizar el funcionamiento correcto de Azure Policy. (Actualmente en versión preliminar en AKS) |
| **`dc.services.visualstudio.com`**            | **`HTTPS:443`** | Complemento de Azure Policy que envía datos de telemetría al punto de conexión de Application Insights. |


## <a name="restrict-egress-traffic-using-azure-firewall"></a>Restricción del tráfico de salida mediante el firewall de Azure

Azure Firewall proporciona una etiqueta FQDN de Azure Kubernetes Service (`AzureKubernetesService`) para simplificar esta configuración. 

> [!NOTE]
> La etiqueta FQDN contiene todos los FQDN enumerados anteriormente y se mantiene actualizada de forma automática.
>
> Se recomienda tener un mínimo de 20 direcciones IP de front-end en Azure Firewall en escenarios de producción para evitar incurrir en problemas de agotamiento de puertos SNAT.

A continuación se muestra un ejemplo de arquitectura de la implementación:

![Detalle de la topología](media/limit-egress-traffic/aks-azure-firewall-egress.png)

* La entrada publica se inserta en el flujo mediante filtros de firewall
  * Los nodos de agente de AKS están aislados en una subred dedicada.
  * [Azure Firewall](../firewall/overview.md) se implementa en una subred propia.
  * Una regla de DNAT traduce la dirección IP pública del firewall en la dirección IP de front-end del equilibrador de carga.
* Las solicitudes de salida parten de los nodos del agente hacia la dirección IP interna de Azure Firewall mediante una [ruta definida por el usuario](egress-outboundtype.md)
  * Las solicitudes de los nodos de agente de AKS siguen una UDR que se ha colocado en la subred en la que se implementó el clúster de AKS.
  * El firewall de Azure sale de la red virtual de un front-end con una IP pública.
  * El acceso a la red pública de Internet u otros servicios de Azure fluye hacia y desde la dirección IP del servidor front-end del firewall.
  * Opcionalmente, el acceso al plano de control de AKS está protegido por [intervalos IP autorizados del servidor de API](./api-server-authorized-ip-ranges.md), que incluye la dirección IP de front-end pública del firewall.
* Tráfico interno
  * Opcionalmente, en lugar o además de un [equilibrador de carga público](load-balancer-standard.md), puede usar un [equilibrador de carga interno](internal-lb.md) para el tráfico interno, que también se podría aislar en una subred propia.


En los pasos siguientes se usa la etiqueta de FQDN `AzureKubernetesService` de Azure Firewall para restringir el tráfico de salida desde el clúster de AKS y se proporciona un ejemplo de cómo configurar el tráfico de entrada público a través del firewall.


### <a name="set-configuration-via-environment-variables"></a>Configuración mediante variables de entorno

Defina el conjunto de variables de entorno que se usarán en la creación de recursos.

```bash
PREFIX="aks-egress"
RG="${PREFIX}-rg"
LOC="eastus"
PLUGIN=azure
AKSNAME="${PREFIX}"
VNET_NAME="${PREFIX}-vnet"
AKSSUBNET_NAME="aks-subnet"
# DO NOT CHANGE FWSUBNET_NAME - This is currently a requirement for Azure Firewall.
FWSUBNET_NAME="AzureFirewallSubnet"
FWNAME="${PREFIX}-fw"
FWPUBLICIP_NAME="${PREFIX}-fwpublicip"
FWIPCONFIG_NAME="${PREFIX}-fwconfig"
FWROUTE_TABLE_NAME="${PREFIX}-fwrt"
FWROUTE_NAME="${PREFIX}-fwrn"
FWROUTE_NAME_INTERNET="${PREFIX}-fwinternet"
```

### <a name="create-a-virtual-network-with-multiple-subnets"></a>Creación de una red virtual con varias subredes

Aprovisione una red virtual con dos subredes independientes, una para el clúster y otra para el firewall. Opcionalmente, también puede crear una para la entrada del servicio interno.

![Topología de red vacía](media/limit-egress-traffic/empty-network.png)

Cree el grupo de recursos que contendrá todos los recursos.

```azure-cli
# Create Resource Group

az group create --name $RG --location $LOC
```

Cree una red virtual con dos subredes para hospedar el clúster de AKS y Azure Firewall. Cada uno tendrá su propia subred. Comencemos con la red de AKS.

```
# Dedicated virtual network with AKS subnet

az network vnet create \
    --resource-group $RG \
    --name $VNET_NAME \
    --address-prefixes 10.42.0.0/16 \
    --subnet-name $AKSSUBNET_NAME \
    --subnet-prefix 10.42.1.0/24

# Dedicated subnet for Azure Firewall (Firewall name cannot be changed)

az network vnet subnet create \
    --resource-group $RG \
    --vnet-name $VNET_NAME \
    --name $FWSUBNET_NAME \
    --address-prefix 10.42.2.0/24
```

### <a name="create-and-set-up-an-azure-firewall-with-a-udr"></a>Creación y configuración de un firewall de Azure con una UDR

Deben configurarse las reglas de entrada y salida del firewall de Azure. El propósito principal del firewall es que las organizaciones puedan configurar reglas de tráfico de entrada y salida pormenorizadas hacia y desde el clúster de AKS.

![Firewall y UDR](media/limit-egress-traffic/firewall-udr.png)


> [!IMPORTANT]
> Si el clúster o la aplicación crea un gran número de conexiones de salida dirigidas al mismo subconjunto de destinos (o a uno más reducido), es posible que necesite más direcciones IP de front-end de firewall para evitar que se agoten los puertos por IP de front-end.
> Para obtener más información sobre cómo crear un firewall de Azure con varias direcciones IP, vea [**esto**](../firewall/quick-create-multiple-ip-template.md).

Cree un recurso de SKU estándar con IP pública que se usará como dirección de front-end de Azure Firewall.

```azure-cli
az network public-ip create -g $RG -n $FWPUBLICIP_NAME -l $LOC --sku "Standard"
```

Registre la extensión de la CLI en versión preliminar para crear un firewall de Azure.
```azure-cli
# Install Azure Firewall preview CLI extension

az extension add --name azure-firewall

# Deploy Azure Firewall

az network firewall create -g $RG -n $FWNAME -l $LOC --enable-dns-proxy true
```
La dirección IP creada anteriormente puede asignarse ahora al front-end del firewall.

> [!NOTE]
> La configuración de la dirección IP pública en el firewall de Azure puede tardar unos minutos.
> Para aprovechar el FQDN en las reglas de red, es necesario habilitar el proxy DNS; cuando lo esté, el firewall escuchará en el puerto 53 y reenviará las solicitudes DNS al servidor DNS especificado anteriormente. Esto permitirá al firewall traducir ese FQDN de forma automática.

```azure-cli
# Configure Firewall IP Config

az network firewall ip-config create -g $RG -f $FWNAME -n $FWIPCONFIG_NAME --public-ip-address $FWPUBLICIP_NAME --vnet-name $VNET_NAME
```

Cuando el comando anterior se haya realizado correctamente, anote la dirección IP del servidor front-end del firewall para la configuración posterior.

```bash
# Capture Firewall IP Address for Later Use

FWPUBLIC_IP=$(az network public-ip show -g $RG -n $FWPUBLICIP_NAME --query "ipAddress" -o tsv)
FWPRIVATE_IP=$(az network firewall show -g $RG -n $FWNAME --query "ipConfigurations[0].privateIpAddress" -o tsv)
```

> [!NOTE]
> Si utiliza el acceso seguro al servidor de la API de AKS con [intervalos de direcciones IP autorizados](./api-server-authorized-ip-ranges.md), debe agregar la dirección IP pública del firewall en el intervalo de IP autorizado.

### <a name="create-a-udr-with-a-hop-to-azure-firewall"></a>Creación de una UDR con un salto al firewall de Azure

Azure enruta automáticamente el tráfico entre redes locales, las redes virtuales y las subredes de Azure. Si desea cambiar algún enrutamiento predeterminado de Azure, debe crear una tabla de rutas.

Cree una tabla de rutas vacía para asociarla a una subred determinada. La tabla de rutas definirá el próximo salto como el firewall de Azure creado anteriormente. Cada subred puede tener cero o una ruta de tablas de ruta asociada.

```azure-cli
# Create UDR and add a route for Azure Firewall

az network route-table create -g $RG -$LOC --name $FWROUTE_TABLE_NAME
az network route-table route create -g $RG --name $FWROUTE_NAME --route-table-name $FWROUTE_TABLE_NAME --address-prefix 0.0.0.0/0 --next-hop-type VirtualAppliance --next-hop-ip-address $FWPRIVATE_IP --subscription $SUBID
az network route-table route create -g $RG --name $FWROUTE_NAME_INTERNET --route-table-name $FWROUTE_TABLE_NAME --address-prefix $FWPUBLIC_IP/32 --next-hop-type Internet
```

Consulte en la [documentación sobre las tablas de rutas de red virtual](../virtual-network/virtual-networks-udr-overview.md#user-defined) cómo invalidar las rutas del sistema predeterminadas de Azure o agregar rutas adicionales a la tabla de rutas de una subred.

### <a name="adding-firewall-rules"></a>Adición de reglas de firewall

A continuación se muestran tres reglas de red que puede usar para configurar en el firewall; es posible que tenga que adaptarlas en función de la implementación. La primera regla permite el acceso al puerto 9000 a través de TCP. La segunda regla permite el acceso a los puertos 1194 y 123 a través de UDP (si va a implementar en Azure China 21Vianet, es posible que necesite [más](#azure-china-21vianet-required-network-rules)). Las dos reglas solo permitirán el tráfico destinado al CIDR de la región de Azure que se use, en este caso, Este de EE. UU. Por último, se agrega una tercera regla de red para abrir el puerto 123 al FQDN `ntp.ubuntu.com` a través de UDP (la adición de un FQDN como una regla de red es una de las características específicas de Azure Firewall y tendrá que adaptarla cuando use opciones propias).

Después de establecer las reglas de red, también se agregará una regla de aplicación mediante `AzureKubernetesService`, que abarca todos los FQDN necesarios a los que se puede acceder a través del puerto TCP 443 y el puerto 80.

```
# Add FW Network Rules

az network firewall network-rule create -g $RG -f $FWNAME --collection-name 'aksfwnr' -n 'apiudp' --protocols 'UDP' --source-addresses '*' --destination-addresses "AzureCloud.$LOC" --destination-ports 1194 --action allow --priority 100
az network firewall network-rule create -g $RG -f $FWNAME --collection-name 'aksfwnr' -n 'apitcp' --protocols 'TCP' --source-addresses '*' --destination-addresses "AzureCloud.$LOC" --destination-ports 9000
az network firewall network-rule create -g $RG -f $FWNAME --collection-name 'aksfwnr' -n 'time' --protocols 'UDP' --source-addresses '*' --destination-fqdns 'ntp.ubuntu.com' --destination-ports 123

# Add FW Application Rules

az network firewall application-rule create -g $RG -f $FWNAME --collection-name 'aksfwar' -n 'fqdn' --source-addresses '*' --protocols 'http=80' 'https=443' --fqdn-tags "AzureKubernetesService" --action allow --priority 100
```

Para más información acerca del servicio Azure Firewall, consulte la [documentación de Azure Firewall](../firewall/overview.md).

### <a name="associate-the-route-table-to-aks"></a>Asociación de la tabla de rutas a AKS

Para asociar el clúster al firewall, la subred dedicada del clúster debe hacer referencia a la tabla de rutas creada anteriormente. Para crear la asociación, se puede emitir un comando a la red virtual que contiene el clúster y el firewall para actualizar la tabla de rutas de la subred del clúster.

```azure-cli
# Associate route table with next hop to Firewall to the AKS subnet

az network vnet subnet update -g $RG --vnet-name $VNET_NAME --name $AKSSUBNET_NAME --route-table $FWROUTE_TABLE_NAME
```

### <a name="deploy-aks-with-outbound-type-of-udr-to-the-existing-network"></a>Implementación de AKS con el tipo de salida UDR a la red existente

Ahora, un clúster de AKS se puede implementar en una red virtual existente. También se usará el [tipo de salida `userDefinedRouting`](egress-outboundtype.md); esta característica garantiza que se forzará el tráfico saliente a través del firewall y no existirán otras rutas de salida (de forma predeterminada, se podría usar el tipo de salida Load Balancer).

![aks-deploy](media/limit-egress-traffic/aks-udr-fw.png)

### <a name="create-a-service-principal-with-access-to-provision-inside-the-existing-virtual-network"></a>Creación de una entidad de servicio con acceso de aprovisionamiento dentro de la red virtual existente

AKS usa una entidad de servicio para crear recursos de clúster. La entidad de servicio que se pasa en el momento de la creación se usa para crear recursos de AKS subyacentes, como los recursos de almacenamiento, las direcciones IP y los equilibradores de carga que utiliza AKS (en su lugar también se puede usar una [identidad administrada](use-managed-identity.md)). Si no se le han concedido los permisos adecuados siguientes, no podrá aprovisionar el clúster de AKS.

```azure-cli
# Create SP and Assign Permission to Virtual Network

az ad sp create-for-rbac -n "${PREFIX}sp" --skip-assignment
```

Ahora, reemplace `APPID` y `PASSWORD` a continuación por el identificador de aplicación y la contraseña de la entidad de servicio, generados automáticamente por la salida del comando anterior. Se hará referencia al identificador de recurso de red virtual para conceder los permisos a la entidad de servicio para que AKS pueda implementar recursos en ella.

```azure-cli
APPID="<SERVICE_PRINCIPAL_APPID_GOES_HERE>"
PASSWORD="<SERVICEPRINCIPAL_PASSWORD_GOES_HERE>"
VNETID=$(az network vnet show -g $RG --name $VNET_NAME --query id -o tsv)

# Assign SP Permission to VNET

az role assignment create --assignee $APPID --scope $VNETID --role "Network Contributor"
```

[Aquí](kubernetes-service-principal.md#delegate-access-to-other-azure-resources) puede comprobar los permisos detallados necesarios.

> [!NOTE]
> Si usa el complemento de red kubenet, tendrá que conceder a la entidad de servicio de AKS o a la identidad administrada permisos en la tabla de rutas creada previamente, ya que kubenet requiere una tabla de rutas para agregar las reglas de enrutamiento necesarias. 
> ```azurecli-interactive
> RTID=$(az network route-table show -g $RG -n $FWROUTE_TABLE_NAME --query id -o tsv)
> az role assignment create --assignee $APPID --scope $RTID --role "Network Contributor"
> ```

### <a name="deploy-aks"></a>Implementación de AKS

Por último, el clúster de AKS se puede implementar en la subred existente que se ha dedicado al clúster. La subred de destino en la que se va a realizar la implementación se define con la variable de entorno `$SUBNETID`. No definimos la variable `$SUBNETID` en los pasos anteriores. Para establecer el valor del id. de la subred, puede usar el comando siguiente:

```azurecli
SUBNETID=$(az network vnet subnet show -g $RG --vnet-name $VNET_NAME --name $AKSSUBNET_NAME --query id -o tsv)
```

Definirá el tipo de salida para usar el UDR que ya existe en la subred. Esta configuración permitirá a AKS omitir la configuración y el aprovisionamiento de IP para el equilibrador de carga.

> [!IMPORTANT]
> Para obtener más información sobre el tipo de salida UDR, incluidas las limitaciones, vea [**UDR de tipo de salida**](egress-outboundtype.md#limitations).


> [!TIP]
> Se pueden agregar otras características a la implementación del clúster, como [**Clúster privado**](private-clusters.md). 
>
> La característica de [**intervalos de IP autorizados del servidor de API**](api-server-authorized-ip-ranges.md) de AKS se puede agregar para limitar el acceso del servidor de API solo al punto de conexión público del firewall. La característica de intervalos IP autorizados se indica en el diagrama como opcional. Al habilitar la característica de intervalos de IP autorizados para limitar el acceso del servidor de API, las herramientas de desarrollo deben usar una JumpBox desde la red virtual del firewall, o bien debe agregar todos los puntos de conexión de desarrollador al intervalo de direcciones IP autorizado.

```azure-cli
az aks create -g $RG -n $AKSNAME -l $LOC \
  --node-count 3 --generate-ssh-keys \
  --network-plugin $PLUGIN \
  --outbound-type userDefinedRouting \
  --service-cidr 10.41.0.0/16 \
  --dns-service-ip 10.41.0.10 \
  --docker-bridge-address 172.17.0.1/16 \
  --vnet-subnet-id $SUBNETID \
  --service-principal $APPID \
  --client-secret $PASSWORD \
  --api-server-authorized-ip-ranges $FWPUBLIC_IP
```

### <a name="enable-developer-access-to-the-api-server"></a>Habilitación del acceso de desarrollador al servidor de API

Si en el paso anterior ha usado intervalos IP autorizados para el clúster, tendrá que agregar las direcciones IP de las herramientas de desarrollador a la lista de intervalos IP autorizados del clúster de AKS para poder acceder al servidor de API desde ahí. Otra opción consiste en configurar una JumpBox con las herramientas necesarias en una subred independiente de la red virtual del firewall.

Agrege otra dirección IP a los intervalos autorizados con el siguiente comando:

```bash
# Retrieve your IP address
CURRENT_IP=$(dig @resolver1.opendns.com ANY myip.opendns.com +short)

# Add to AKS approved list
az aks update -g $RG -n $AKSNAME --api-server-authorized-ip-ranges $CURRENT_IP/32

```

 Use el comando [az aks get-credentials][az-aks-get-credentials] para configurar `kubectl` a fin de conectarse al clúster de Kubernetes recién creado. 

 ```azure-cli
 az aks get-credentials -g $RG -n $AKSNAME
 ```

### <a name="deploy-a-public-service"></a>Implementación de un servicio público
Ahora puede empezar a exponer servicios e implementar aplicaciones en este clúster. En este ejemplo, se expondrá un servicio público, pero también puede optar por exponer un servicio interno a través de un [equilibrador de carga interno](internal-lb.md).

![DNAT de servicio público](media/limit-egress-traffic/aks-create-svc.png)

Para implementar aplicación de votación de Azure, copie el código YAML siguiente en un archivo llamado `example.yaml`.

```yaml
# voting-storage-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-storage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: voting-storage
  template:
    metadata:
      labels:
        app: voting-storage
    spec:
      containers:
      - name: voting-storage
        image: mcr.microsoft.com/aks/samples/voting/storage:2.0
        args: ["--ignore-db-dir=lost+found"]
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_DATABASE
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
---
# voting-storage-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: voting-storage-secret
type: Opaque
data:
  MYSQL_USER: ZGJ1c2Vy
  MYSQL_PASSWORD: UGFzc3dvcmQxMg==
  MYSQL_DATABASE: YXp1cmV2b3Rl
  MYSQL_ROOT_PASSWORD: UGFzc3dvcmQxMg==
---
# voting-storage-pv-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
# voting-storage-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: voting-storage
  labels: 
    app: voting-storage
spec:
  ports:
  - port: 3306
    name: mysql
  selector:
    app: voting-storage
---
# voting-app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: voting-app
  template:
    metadata:
      labels:
        app: voting-app
    spec:
      containers:
      - name: voting-app
        image: mcr.microsoft.com/aks/samples/voting/app:2.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
        env:
        - name: MYSQL_HOST
          value: "voting-storage"
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_DATABASE
        - name: ANALYTICS_HOST
          value: "voting-analytics"
---
# voting-app-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: voting-app
  labels: 
    app: voting-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
    name: http
  selector:
    app: voting-app
---
# voting-analytics-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-analytics
spec:
  replicas: 1
  selector:
    matchLabels:
      app: voting-analytics
      version: "2.0"
  template:
    metadata:
      labels:
        app: voting-analytics
        version: "2.0"
    spec:
      containers:
      - name: voting-analytics
        image: mcr.microsoft.com/aks/samples/voting/analytics:2.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
        env:
        - name: MYSQL_HOST
          value: "voting-storage"
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: voting-storage-secret
              key: MYSQL_DATABASE
---
# voting-analytics-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: voting-analytics
  labels: 
    app: voting-analytics
spec:
  ports:
  - port: 8080
    name: http
  selector:
    app: voting-analytics
```

Para implementar el servicio, ejecute:

```bash
kubectl apply -f example.yaml
```

### <a name="add-a-dnat-rule-to-azure-firewall"></a>Adición de una regla de DNAT al firewall de Azure

> [!IMPORTANT]
> Cuando use Azure Firewall para restringir el tráfico de salida y cree una ruta definida por el usuario (UDR) para forzar todo el tráfico de salida, asegúrese de crear una regla DNAT adecuada en el firewall para permitir correctamente el tráfico de entrada. El uso de Azure Firewall con una UDR interrumpe la configuración de entrada debido al enrutamiento asimétrico. (El problema se produce si la subred de AKS tiene una ruta predeterminada que va a la dirección IP privada del firewall, pero está usando un equilibrador de carga público, de entrada o de servicio de Kubernetes del tipo: LoadBalancer). En este caso, el tráfico entrante del equilibrador de carga se recibe a través de su dirección IP pública, pero la ruta de vuelta pasa a través de la dirección IP privada del firewall. Dado que el firewall es con estado, quita el paquete de vuelta porque el firewall no tiene conocimiento de una sesión establecida. Para aprender a integrar Azure Firewall con el equilibrador de carga de entrada o de servicio, consulte [Integración de Azure Firewall con Azure Standard Load Balancer](../firewall/integrate-lb.md).


Para configurar la conectividad de entrada, se debe escribir una regla de DNAT en el firewall de Azure. Para probar la conectividad con el clúster, se define una regla para la dirección IP pública de front-end del firewall que se va a enrutar a la dirección IP interna expuesta por el servicio interno.

La dirección de destino se puede personalizar porque es el puerto del firewall al que se va a acceder. La dirección traducida debe ser la dirección IP del equilibrador de carga interno. El puerto traducido debe ser el puerto expuesto para el servicio Kubernetes.

Tendrá que especificar la dirección IP interna asignada al equilibrador de carga creado por el servicio Kubernetes. Para recuperar la dirección, ejecute:

```bash
kubectl get services
```

La dirección IP necesaria se mostrará en la columna EXTERNAL-IP y será similar a la siguiente.

```bash
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP      10.41.0.1       <none>        443/TCP        10h
voting-analytics   ClusterIP      10.41.88.129    <none>        8080/TCP       9m
voting-app         LoadBalancer   10.41.185.82    20.39.18.6    80:32718/TCP   9m
voting-storage     ClusterIP      10.41.221.201   <none>        3306/TCP       9m
```

Para obtener la IP del servicio, ejecute lo siguiente:
```bash
SERVICE_IP=$(k get svc voting-app -o jsonpath='{.status.loadBalancer.ingress[*].ip}')
```

Para agregar la regla de NAT, ejecute lo siguiente:
```azure-cli
az network firewall nat-rule create --collection-name exampleset --destination-addresses $FWPUBLIC_IP --destination-ports 80 --firewall-name $FWNAME --name inboundrule --protocols Any --resource-group $RG --source-addresses '*' --translated-port 80 --action Dnat --priority 100 --translated-address $SERVICE_IP
```

### <a name="validate-connectivity"></a>Validar conectividad

Vaya a la dirección IP de front-end del firewall de Azure en un explorador para validar la conectividad.

Debería ver la aplicación de votación de AKS. En este ejemplo, la dirección IP pública del firewall es `52.253.228.132`.


![aks-vote](media/limit-egress-traffic/aks-vote.png)


### <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar los recursos de Azure, elimine el grupo de recursos de AKS.

```azure-cli
az group delete -g $RG
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido qué puertos y direcciones se deben permitir si se quiere restringir el tráfico de salida para el clúster. También ha visto cómo proteger el tráfico de salida mediante Azure Firewall. 

Si es necesario, puede generalizar los pasos anteriores para reenviar el tráfico a la solución de salida preferida, si sigue la [documentación del tipo de salida `userDefinedRoute`](egress-outboundtype.md).

Si quiere restringir cómo se comunican los pods entre sí y con las restricciones de tráfico horizontal de derecha a izquierda en el clúster, vea [Protección del tráfico entre pods mediante directivas de red en AKS][network-policy].

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
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[dev-spaces-service-tags]: ../dev-spaces/configure-networking.md#virtual-network-or-subnet-configurations
