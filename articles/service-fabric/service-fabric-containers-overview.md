---
title: Información general de Service Fabric y contenedores
description: Información general sobre Service Fabric y el uso de contenedores para implementar aplicaciones de microservicios. Este artículo le proporciona información general de cómo se pueden utilizar los contenedores y las funcionalidades disponibles en Service Fabric.
ms.topic: conceptual
ms.date: 7/9/2020
ms.openlocfilehash: cd0ec7dd2247fdd791df362fa34542178c17df4d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091664"
---
# <a name="service-fabric-and-containers"></a>Service Fabric y contenedores

## <a name="introduction"></a>Introducción

Azure Service Fabric es una plataforma de sistemas distribuidos que facilita el empaquetado, la implementación y la administración de microservicios y contenedores escalables y confiables.

Service Fabric es el [orquestador de contenedores](service-fabric-cluster-resource-manager-introduction.md) de Microsoft que implementa microservicios en un clúster de máquinas. Service Fabric se beneficia de las lecciones aprendidas durante sus años de ejecución de servicios en Microsoft a gran escala.

Se pueden desarrollar microservicios de muchas maneras, desde usando los [modelos de programación de Service Fabric](service-fabric-choose-framework.md) o [ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md) hasta implementando [cualquier código que prefiera](service-fabric-guest-executables-introduction.md). O, si simplemente quiere [implementar y administrar contenedores](service-fabric-containers-overview.md), Service Fabric es también una buena opción.

De forma predeterminada Service Fabric permite implementar y activar estos servicios como procesos. Los procesos proporcionan la activación más rápida y el uso de densidad más alto de los recursos del clúster. Service Fabric puede implementar también servicios en imágenes de contenedor. También puede mezclar servicios en procesos y servicios en contenedores en la misma aplicación.

Para comenzar de inmediato y probar contenedores en Service Fabric, pruebe una guía de inicio rápido, tutorial o ejemplo:  

[Inicio rápido: Implementación de una aplicación contenedora de Linux en Service Fabric](service-fabric-quickstart-containers-linux.md)  
[Inicio rápido: Implementación de una aplicación contenedora de Windows en Service Fabric](service-fabric-quickstart-containers.md)  
[Inclusión de una aplicación .NET existente en un contenedor](service-fabric-host-app-in-a-container.md)  
[Ejemplos de contenedores de Service Fabric](https://azure.microsoft.com/resources/samples/service-fabric-containers/)  

## <a name="what-are-containers"></a>¿Qué son los contenedores?

Los contenedores vienen a solucionar el problema de ejecutar aplicaciones de forma fiable en distintos entornos de proceso proporcionando un entorno inmutable para que la aplicación se ejecute en él. Los contenedores encapsulan una aplicación y todas sus dependencias, como bibliotecas y archivos de configuración, en su propio "cuadro" aislado que contiene todo lo necesario para ejecutar el software en el contenedor. Siempre que se ejecuta el contenedor, la aplicación que está dentro de él dispone de todo lo necesario para ejecutarse como, por ejemplo, las versiones adecuadas de sus bibliotecas dependientes, todos los archivos de configuración y cualquier otra cosa que necesite para ejecutarse.

Los contenedores se ejecutan directamente en el kernel y tienen una vista aislada del sistema de archivos y de otros recursos. Una aplicación en un contenedor no tiene relación con ninguna otra aplicación o proceso fuera de su contenedor. Cada aplicación y su entorno en tiempo de ejecución, las dependencias y las bibliotecas del sistema se ejecutan dentro de un contenedor con acceso completo y privado a su propia vista aislada del sistema operativo. Además de proporcionar con facilidad todas las dependencias que la aplicación necesita para ejecutarse en diferentes entornos de proceso, la seguridad y el aislamiento de los recursos son ventajas importantes del uso de contenedores con Service Fabric que, de lo contrario, ejecutaría los servicios en un proceso.

En comparación con las máquinas virtuales, los contenedores ofrecen las siguientes ventajas:

* **Pequeño**: usan un único espacio de almacenamiento y actualizaciones y versiones de capa para aumentar la eficacia.
* **Rápidos**: no tienen que arrancar un sistema operativo entero, así que pueden iniciarse con mayor rapidez, normalmente en unos segundos.
* **Portabilidad**: se puede portar una imagen de aplicación en contenedor para que se ejecute en la nube o localmente, en máquinas virtuales o directamente en máquinas físicas.
* **Gobernanza de recursos**: un contenedor puede limitar los recursos físicos que puede consumir en el host.

### <a name="container-types-and-supported-environments"></a>Tipos de contenedor y entornos compatibles

Service Fabric admite contenedores en Linux y Windows, y admite el modo de aislamiento de Hyper-V en Windows.

#### <a name="docker-containers-on-linux"></a>Contenedores de Docker en Linux

Docker ofrece API para crear y administrar contenedores sobre contenedores de kernel de Linux. Docker Hub proporciona un repositorio central para almacenar y recuperar imágenes de contenedor.
Para ver un tutorial basado en Linux, consulte [Cree la primera aplicación de contenedor de Service Fabric en Linux](service-fabric-get-started-containers-linux.md).

#### <a name="windows-server-containers"></a>Contenedores de Windows Server

Windows Server 2016 y versiones posteriores proporcionan dos tipos distintos de contenedores que difieren en el nivel de aislamiento que ofrecen. Los contenedores de Windows Server y los de Docker se parecen en que tienen un aislamiento del sistema de archivos y un espacio de nombres, pero comparten el kernel con el host en que se ejecutan. En Linux, este aislamiento tradicionalmente lo han proporcionado los cgroups y los espacios de nombres, y los contenedores de Windows Server se comportan de forma parecida.

Los contenedores de Windows compatibles con Hyper-V proporcionan mayor seguridad y aislamiento, ya que los contenedores no comparten el kernel del sistema operativo entre ellos ni con el host. Con este mayor nivel de aislamiento de seguridad, los contenedores habilitados para Hyper-V están diseñados para escenarios potencialmente hostiles y multiinquilino.
Para ver un tutorial basado en Windows, consulte [Cree la primera aplicación de contenedor de Service Fabric en Windows](service-fabric-get-started-containers.md).

La ilustración siguiente muestra los diferentes tipos de niveles de aislamiento y virtualización disponibles.
![Plataforma de Service Fabric][Image1]

## <a name="scenarios-for-using-containers"></a>Escenarios de uso de contenedores

Ejemplos típicos de buena elección de contenedor:

* **Migración mediante lift-and-shift de IIS**: puede colocar una aplicación existente de [ASP.NET MVC](https://www.asp.net/mvc) en un contenedor en lugar de migrarla a ASP.NET Core. Estas aplicaciones de ASP.NET MVC dependen de Internet Information Services (IIS). Puede empaquetarlas en imágenes de contenedor a partir de la imagen de IIS creada previamente e implementarlas con Service Fabric. Consulte [Imágenes de contenedores en Windows Server](/virtualization/windowscontainers/quick-start/quick-start-windows-server) para obtener información sobre los contenedores de Windows.

* **Mezcla de los contenedores y los microservicios de Service Fabric**: Use una imagen de contenedor existente para parte de la aplicación. Por ejemplo, podría usar el [contenedor NGINX](https://hub.docker.com/_/nginx/) para el front-end web de su aplicación y los servicios con estado para la computación de back-end más intensa.

* **Reducción del impacto de los servicios de "vecinos ruidosos"** : puede usar la capacidad de gobernanza de recursos de los contenedores para restringir los recursos que utiliza un servicio en un host. Si hay servicios que consuman un gran número de recursos y afecten al rendimiento de otros (por ejemplo, una consulta de larga ejecución como operación), considere la posibilidad de poner estos servicios en contenedores con gobernanza de recursos.

## <a name="service-fabric-support-for-containers"></a>Compatibilidad de los contenedores con Service Fabric

Service Fabric admite la implementación de contenedores de Docker en Linux y de contenedores de Windows Server en Windows Server 2016 y versiones posteriores, además de el modo de aislamiento de Hyper-V.

> [!NOTE]
> Los contenedores no se admiten en los clústeres de Service Fabric de nodo único local (ni en los clústeres de Linux en OneBox, ni en los clústeres de Windows en instalaciones de Service Fabric locales).

Service Fabric proporciona un [modelo de aplicación](service-fabric-application-model.md)en el que un contenedor representa un host de la aplicación en el que se colocan varias réplicas de servicio. Service Fabric admite también un [escenario de archivo ejecutable invitado](service-fabric-guest-executables-introduction.md) en el que el usuario no usa los modelos de programación de Service Fabric integrados y, en su lugar, empaqueta una aplicación existente, escrita con cualquier lenguaje o plataforma, dentro de un contenedor. Este escenario es el caso habitual de los contenedores.

También puede ejecutar [servicios de Service Fabric dentro de un contenedor](service-fabric-services-inside-containers.md). La compatibilidad con la ejecución de servicios de Service Fabric dentro de contenedores está limitada actualmente.

Service Fabric ofrece varias funcionalidades de contenedor que le ayudarán a crear aplicaciones que se componen de microservicios en contenedores, como:

* Activación e implementación de la imagen de contenedor.
* Gobernanza de recursos, incluida la configuración de valores de recursos predeterminados en clústeres de Azure.
* La autenticación de repositorios.
* La asignación de puerto a host de contenedor.
* La detección y comunicación entre contenedores.
* La capacidad de configurar y establecer variables de entorno.
* La capacidad de configurar credenciales de seguridad en el contenedor.
* Selección de diferentes modos de red para contenedores.

Para obtener una introducción completa sobre la compatibilidad de contenedores en Azure para, por ejemplo, crear un clúster de Kubernetes con Azure Kubernetes Service, crear un registro privado de Docker en Azure Container Registry, etc, consulte [Azure for Containers](../containers/index.yml).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido sobre la compatibilidad que ofrece Service Fabric para la ejecución de contenedores. A continuación, analizaremos algunos ejemplos de cada una de las características para mostrarle cómo utilizarlas.

[Cree la primera aplicación de contenedor de Service Fabric en Linux](service-fabric-get-started-containers-linux.md)  
[Cree la primera aplicación de contenedor en Service Fabric en Windows](service-fabric-get-started-containers.md)  
[Más información acerca de los contenedores de Windows](/virtualization/windowscontainers/about/)

[Image1]: media/service-fabric-containers/Service-Fabric-Types-of-Isolation.png
