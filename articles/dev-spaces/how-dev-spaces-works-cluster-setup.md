---
title: Cómo funciona la configuración de un clúster para Azure Dev Spaces
services: azure-dev-spaces
ms.date: 03/24/2020
ms.topic: conceptual
description: Describe cómo funciona la configuración de un clúster de Azure Kubernetes Service para Azure Dev Spaces
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores
ms.openlocfilehash: 41b955fac7abcc700d73d8ace601f80982b2253d
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88207961"
---
# <a name="how-setting-up-a-cluster-for-azure-dev-spaces-works"></a>Cómo funciona la configuración de un clúster para Azure Dev Spaces

Azure Dev Spaces le ofrece varias formas de iterar y depurar rápidamente las aplicaciones Kubernetes y colaborar con su equipo en un clúster de Azure Kubernetes Service (AKS). Una forma de hacerlo consiste en habilitar Azure Dev Spaces en el clúster de AKS para que pueda [ejecutar servicios directamente en el clúster][how-it-works-up] y usar [funciones adicionales de red y de enrutamiento][how-it-works-routing]. En este artículo se describe lo que sucede cuando se prepara el clúster y se habilita Azure Dev Spaces.

## <a name="prepare-your-aks-cluster"></a>Preparación del clúster de AKS

Para preparar el clúster de AKS para Dev Spaces, debe comprobar que el clúster de AKS está en una región [compatible con Azure Dev Spaces][supported-regions] y que está ejecutando Kubernetes 1.10.3 o una versión posterior. Puede habilitar Azure Dev Spaces desde la CLI de Azure ejecutando `az aks use-dev-spaces`.

Cuando se habilita Azure Dev Spaces en el clúster de AKS, instala el controlador para el clúster. El controlador reside fuera del clúster de AKS. impulsa el comportamiento y la comunicación entre las herramientas de cliente y el clúster de AKS. Una vez habilitado, se puede interactuar con él usando las herramientas de cliente.

El controlador realiza las acciones siguientes:

* Administra la creación y la selección de espacios de desarrollo.
* Instala el gráfico de Helm de la aplicación y crea objetos de Kubernetes.
* Compila la imagen de contenedor de la aplicación.
* Implementa la aplicación en AKS.
* Lleva a cabo compilaciones incrementales y reinicios cuando el código fuente cambia.
* Administra los registros y los seguimientos HTTP.
* Reenvía stdout y stderr a las herramientas de cliente.
* Configura el enrutamiento para las aplicaciones dentro de un espacio, así como entre espacios primarios y secundarios.

El controlador es un recurso de Azure independiente situado fuera del clúster que hace lo siguiente a los recursos del clúster:

* Crea o designa un espacio de nombres de Kubernetes que se usará como un espacio de desarrollo.
* Quita cualquier espacio de nombres de Kubernetes que se denomine *azds*, si existe, y crea uno nuevo.
* Implementa una configuración de webhook de Kubernetes.
* Implementa un servidor de admisión de webhook.

Usa la misma entidad de servicio que emplea el clúster de AKS para realizar llamadas de servicio a otros componentes de Azure Dev Spaces.

![Preparación del clúster para Azure Dev Spaces](media/how-dev-spaces-works/prepare-cluster.svg)

Para poder usar Azure Dev Spaces, debe haber al menos un espacio de desarrollo. Azure Dev Spaces usa los espacios de nombres de Kubernetes dentro del clúster de AKS para los espacios de desarrollo. Cuando instale un controlador, se le pedirá que cree un espacio de nombres de Kubernetes o que elija uno existente para usarlo como primer espacio de desarrollo. De forma predeterminada, el controlador ofrece la opción de actualizar el espacio de nombres *predeterminado* de Kubernetes al primer espacio de desarrollo.

Cuando un espacio de nombres se designa como espacio de desarrollo, el controlador agrega la etiqueta *azds.io/space=true* a ese espacio de nombres para identificarlo como espacio de desarrollo. El espacio de desarrollo inicial que cree o designe se seleccionará de forma predeterminada una vez que haya preparado el clúster. Cuando se seleccione un espacio, Azure Dev Spaces lo usará para crear cargas de trabajo.

Puede usar las herramientas de cliente para crear espacios de desarrollo y quitar otros existentes. Debido a una limitación en Kubernetes, el espacio de desarrollo *default* no se puede quitar. El controlador también quita los espacios de nombres existentes de Kubernetes que se denominen *azds* para evitar conflictos con el comando `azds` que usan las herramientas de cliente.

El servidor de admisión de webhook de Kubernetes se usa para insertar pods con tres contenedores durante la implementación para la instrumentación: devspaces-proxy, devspaces-proxy-init y devspaces-build. **Estos tres contenedores se ejecutan con acceso a la raíz en el clúster de AKS.** Además, usan la misma entidad de servicio que emplea el clúster de AKS para realizar llamadas de servicio a otros componentes de Azure Dev Spaces.

![Servidor de admisión de webhook de Kubernetes de Azure Dev Spaces](media/how-dev-spaces-works/kubernetes-webhook-admission-server.svg)

El contenedor devspaces-proxy es un contenedor sidecar que controla todo el tráfico TCP dentro y fuera del contenedor de la aplicación y que ayuda a realizar el enrutamiento. El contenedor devspaces-proxy vuelve a enrutar los mensajes HTTP si se usan determinados espacios. Por ejemplo, puede ayudar a enrutar los mensajes HTTP entre aplicaciones en espacios primarios y secundarios. Todo el tráfico que no sea HTTP pasa a través de devspaces-proxy sin modificar. El contenedor devspaces-proxy también registra todos los mensajes HTTP entrantes y salientes y los envía a las herramientas de cliente como seguimientos. El desarrollador puede visualizar estos seguimientos para inspeccionar el comportamiento de la aplicación.

El contenedor devspaces-proxy-init es un [contenedor init](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) que agrega reglas de enrutamiento adicionales en función de la jerarquía de espacio al contenedor de la aplicación. Para agregar reglas de enrutamiento, actualiza el archivo */etc/resolv.conf* del contenedor de la aplicación y la configuración de iptables antes de comenzar. Las actualizaciones de */etc/resolv.conf* permiten la resolución DNS de los servicios en espacios primarios. Las actualizaciones de la configuración de iptables garantizan que todo el tráfico TCP dentro y fuera del contenedor de la aplicación se enrute a través de devspaces-proxy. Todas las actualizaciones de devspaces-proxy-init se producen de forma complementaria a las reglas que agrega Kubernetes.

El contenedor devspaces-build es un contenedor init que tiene el código fuente del proyecto y el socket de Docker montado. El código fuente del proyecto y el acceso a Docker permiten que el pod compile directamente el contenedor de la aplicación.

> [!NOTE]
> Azure Dev Spaces usa el mismo nodo para compilar el contenedor de la aplicación y ejecutarlo. Como resultado, Azure Dev Spaces no necesita un registro de contenedor externo para compilar y ejecutar la aplicación.

El servidor de admisión de webhook de Kubernetes escucha los nuevos pods que se creen en el clúster de AKS. Si un pod se implementa en un espacio de nombres con la etiqueta *azds.io/space=true*, inserta en dicho pod los contenedores adicionales. El contenedor devspaces-build solo se inserta si el contenedor de la aplicación se ejecuta con las herramientas de cliente.

Una vez que haya preparado el clúster de AKS, puede usar las herramientas de cliente para preparar y ejecutar el código en el espacio de desarrollo.

## <a name="client-side-tooling"></a>Herramientas del lado cliente

Las herramientas de cliente permiten al usuario:
* Generar un archivo Dockerfile, un gráfico de Helm y un archivo de configuración de Azure Dev Spaces para la aplicación.
* Crear espacios de desarrollo primarios y secundarios.
* Indicar al controlador que compile e inicie la aplicación.

Mientras se ejecuta la aplicación, las herramientas de cliente también hacen lo siguiente:
* Recibir y mostrar stdout y stderr desde la aplicación que se ejecuta en AKS.
* Usar el [reenvío de puerto](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) para permitir el acceso web a la aplicación mediante http:\//localhost.
* Asociar un depurador a la aplicación que se ejecuta en AKS.
* Sincronizar el código fuente con el espacio de desarrollo cuando se detecta un cambio para las compilaciones incrementales, lo que permite una iteración rápida.
* Le permite conectar el equipo del desarrollador directamente al clúster de AKS.

Puede usar las herramientas de cliente desde la línea de comandos como parte del comando `azds`. También puede usar las herramientas de cliente con:

* Visual Studio Code con la [extensión Azure Dev Spaces](https://marketplace.visualstudio.com/items?itemName=azuredevspaces.azds).
* Visual Studio con la carga de trabajo de desarrollo de Azure.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo usar las herramientas de cliente para preparar y ejecutar el código en el espacio de desarrollo, consulte [Cómo funciona la preparación de un proyecto para Azure Dev Spaces][how-it-works-prep].


[how-it-works-prep]: how-dev-spaces-works-prep.md
[how-it-works-routing]: how-dev-spaces-works-routing.md
[how-it-works-up]: how-dev-spaces-works-up.md
[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service