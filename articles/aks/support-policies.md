---
title: Directivas de soporte técnico para Azure Kubernetes Service (AKS)
description: Obtenga información sobre las directivas de soporte técnico de Azure Kubernetes Service (AKS), responsabilidad compartida y las características que se encuentran en versión preliminar (o alfa o beta).
services: container-service
author: jnoller
ms.service: container-service
ms.topic: article
ms.date: 04/01/2019
ms.author: jenoller
ms.openlocfilehash: 0d2c080be727d2ae13d6d9e5274f17cadffbe640
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65786458"
---
# <a name="support-policies-for-azure-kubernetes-service"></a>Directivas de soporte técnico para Azure Kubernetes Service

Este artículo proporciona detalles sobre las directivas de soporte técnico y limitaciones de Azure Kubernetes Service (AKS). El artículo también describe la administración de nodos de trabajo, los componentes del plano de control administrado, los componentes de código abierto de terceros y administración de seguridad o revisiones.

## <a name="service-updates-and-releases"></a>Las versiones y actualizaciones de servicio

* Para obtener información de versión, consulte [AKS notas de la versión](https://github.com/Azure/AKS/releases).
* Para obtener información sobre las características en versión preliminar, consulte [AKS obtener una vista previa de las características y los proyectos relacionados](https://github.com/Azure/AKS/blob/master/previews.md).

## <a name="managed-features-in-aks"></a>Administra las características en AKS

La infraestructura base como componentes de nube de servicio (IaaS), como proceso o los componentes de red, proporcionar a los usuarios acceso a controles de bajo nivel y opciones de personalización. Por el contrario, AKS proporciona una implementación de Kubernetes llave en mano que ofrece a los clientes el conjunto común de configuraciones y capacidades que necesitan. Los clientes AKS tienen limitada la personalización, implementación y otras opciones. Estos clientes no es necesario preocuparse ni administrar clústeres de Kubernetes directamente.

Con AKS, el cliente obtiene totalmente administrado *plano de control*. El plano de control contiene todos los componentes y servicios, que el cliente debe ofrecer y clústeres de Kubernetes para los usuarios finales. Todos los componentes de Kubernetes se mantienen y operados por Microsoft.

Microsoft administra y supervisa los siguientes componentes a través del panel de control:

* Servidores de Kubelet o API de Kubernetes
* Etcd o un almacén de pares clave-valor compatible, proporcionar calidad de servicio (QoS), la escalabilidad y el tiempo de ejecución
* Servicios DNS (por ejemplo, kube-dns o CoreDNS)
* Proxy de Kubernetes o conexión de red

AKS no es una solución de clúster totalmente administrado. Algunos componentes, como nodos de trabajo tienen *responsabilidad compartida*, donde los usuarios deben ayudar a mantener el clúster de AKS. Se requiere proporcionados por el usuario, por ejemplo, para aplicar una revisión de seguridad de sistema operativo (SO) de nodo de trabajo.

Los servicios son *administrada* en el sentido de que Microsoft y el equipo AKS se implementa, funciona y es responsables de la disponibilidad del servicio y la funcionalidad. Los clientes no pueden modificar estos componentes administrados. Microsoft limita la personalización para garantizar una experiencia de usuario coherente y escalable. Para una solución totalmente personalizable, consulte [AKS motor](https://github.com/Azure/aks-engine).

> [!NOTE]
> Nodos de AKS trabajo aparecen en el portal de Azure como los recursos de IaaS de Azure habituales. Pero estas máquinas virtuales se implementan en un grupo de recursos de Azure personalizada (prefijo MC\\*). Es posible cambiar los nodos de trabajo AKS. Por ejemplo, puede usar Secure Shell (SSH) para cambiar la forma de modificar normal de las máquinas virtuales de nodos de trabajo AKS (sin embargo, no es posible, cambie la imagen de sistema operativo base, y los cambios no pueden persistir a través de una actualización o reiniciar), y otros recursos de Azure se pueden adjuntar a AKS nodos de trabajo. Pero si realiza cambios *fuera de la administración de banda y la personalización,* el clúster de AKS puede ser incompatible. Evite cambiar los nodos de trabajo a menos que Microsoft Support le solicite realizar cambios.

## <a name="shared-responsibility"></a>Responsabilidad compartida

Cuando se crea un clúster, el cliente define los nodos de trabajo de Kubernetes que AKS crea. Las cargas de trabajo se ejecutan en esos nodos. Los clientes poseen y pueden ver o modificar los nodos de trabajo.

Dado que los nodos de clúster de cliente ejecutan código privado y almacenan información confidencial, Microsoft Support puede tener acceso a ellos en sólo una manera limitada. Microsoft Support no pueden iniciar sesión, ejecute los comandos de o ver los registros de estos nodos sin permiso expreso del cliente o asistencia.

Dado que los nodos de trabajo son confidenciales, Microsoft toma mucho cuidado para limitar su administración en segundo plano. En muchos casos, la carga de trabajo seguirá ejecutándose aunque el Kubernetes maestro nodos, etcd y otros errores de componentes administrados por Microsoft. Nodos de trabajo modificada infiltrar pueden ocasionar pérdidas de datos y las cargas de trabajo y pueden representar el clúster de incompatibilidad.

## <a name="aks-support-coverage"></a>Cobertura de soporte de AKS

Microsoft proporciona soporte técnico para lo siguiente:

* Conectividad a todos los componentes de Kubernetes que el servicio de Kubernetes proporciona y admite, como el servidor de API.
* Administración, tiempo de actividad, QoS y las operaciones de Kubernetes controlan servicios de plano (nodos maestros de Kubernetes, server API, etcd y kube-dns, por ejemplo).
* Etcd. La compatibilidad incluye las copias de seguridad automatizadas y transparentes de todos los datos de etcd cada 30 minutos para la restauración del estado del clúster y planificación ante desastres. Estas copias de seguridad no están directamente disponibles para los clientes o usuarios. Garantizan la coherencia y confiabilidad de los datos.
* Los puntos de integración en el controlador del proveedor de nube de Azure para Kubernetes. Estos incluyen integraciones en otros servicios de Azure, como los equilibradores de carga, los volúmenes persistentes o redes (Kubernetes y Azure CNI).
* Preguntas o problemas acerca de la personalización de los componentes del plano de control como el servidor de API de Kubernetes, etcd y kube-dns.
* Problemas relacionados con problemas de red, como Azure CNI, kubenet, u otro acceso de red y funcionalidad. Podrían incluir problemas DNS resolución, pérdida de paquetes, enrutamiento y así sucesivamente. Microsoft admite diversos escenarios de redes:
  * Kubenet (basic) y advanced networking (Azure CNI) dentro del clúster y componentes asociados
  * Conectividad a otras aplicaciones y servicios de Azure
  * Los controladores de entrada y las configuraciones del equilibrador de carga o de entrada
  * Rendimiento de la red y latencia

Microsoft no ofrece soporte técnico para lo siguiente:

* Preguntas acerca de cómo usar Kubernetes. Por ejemplo, Microsoft Support no proporciona consejos sobre cómo crear controladores de entrada personalizado, use cargas de trabajo de aplicación o aplicar paquetes de software de terceros o de código abierto o de herramientas.
  > [!NOTE]
  > Microsoft Support puede aconsejarle sobre la funcionalidad de clúster AKS, personalización y optimización (por ejemplo, problemas de operaciones de Kubernetes y procedimientos).
* Proyectos de código abierto de terceros que no se proporcionan como parte de la Kubernetes plano de control o implementan con los clústeres de AKS. Estos proyectos pueden incluir Istio, Helm, Envoy u otros.
  > [!NOTE]
  > Microsoft puede proporcionar soporte técnico de mejor esfuerzo para proyectos de código abierto de terceros, como Helm y Kured. Cuando la herramienta de código abierto de terceros se integra con el proveedor de nube de Kubernetes Azure o de otros errores específicos de AKS, Microsoft es compatible con ejemplos y las aplicaciones de la documentación de Microsoft.
* Software de código cerrado de terceros. Este software puede incluir herramientas de análisis de seguridad y redes dispositivos o software.
* Problemas relacionados con la nube múltiple o múltiples proveedores salidas de compilación. Por ejemplo, Microsoft no admite los problemas relacionados con la ejecución de una solución de proveedor en la nube multipublic federado.
* Las personalizaciones que no aparecen en la red el [documentación de AKS](https://docs.microsoft.com/azure/aks/).
  > [!NOTE]
  > Microsoft es compatible con problemas y los errores relacionados con grupos de seguridad de red (NSG). Por ejemplo, Microsoft Support puede responder preguntas sobre un error de NSG para actualizar o un comportamiento inesperado de equilibrador de carga o NSG.

## <a name="aks-support-coverage-for-worker-nodes"></a>Cobertura de soporte de AKS para nodos de trabajo

### <a name="microsoft-responsibilities-for-aks-worker-nodes"></a>Responsabilidades de Microsoft para los nodos de trabajo AKS

Microsoft y los clientes comparten la responsabilidad de los nodos de trabajo de Kubernetes donde:

* La imagen de sistema operativo base ha requerido adiciones (por ejemplo, supervisión y los agentes de red).
* Los nodos de trabajo reciban automáticamente revisiones de sistema operativo.
* Problemas con el Kubernetes controlan plano se corrigen automáticamente los componentes que se ejecutan en los nodos de trabajo. Los componentes son los siguientes:
  * kube-proxy
  * componentes del maestro de túneles de red que proporcionan rutas de comunicación para Kubernetes
  * kubelet
  * Demonio de docker o Moby

> [!NOTE]
> En un nodo de trabajo, si un componente de plano de control no está operativo, el equipo AKS es posible que deba reiniciar el nodo de trabajo completo. Debido a su acceso restringido a los datos y cargas de trabajo activo del cliente, el equipo de AKS reinicia un nodo de trabajo solo si el cliente realiza una escalada el problema. Siempre que sea posible, el equipo AKS trabaja para evitar el reinicio necesario afecte a la aplicación.

### <a name="customer-responsibilities-for-aks-worker-nodes"></a>Responsabilidades del cliente para los nodos de trabajo AKS

Microsoft no reinicia automáticamente los nodos de trabajo para aplicar las revisiones de nivel de sistema operativo. Aunque las revisiones del sistema operativo se entregan a los nodos de trabajo, el *cliente* es responsable de tener que reiniciar los nodos de trabajo para aplicar los cambios. Bibliotecas compartidas, demonios como unidad de estado sólido híbrida (SSHD) y otros componentes en el nivel del sistema o del sistema operativo se aplica automáticamente las revisiones.

Los clientes son responsables de ejecutar las actualizaciones de Kubernetes. Pueden ejecutar las actualizaciones a través del panel de control de Azure o la CLI de Azure. Esto se aplica a las actualizaciones que contienen la seguridad o mejoras de funcionalidad en Kubernetes.

> [!NOTE]
> Dado que AKS es un *servicio administrado*, sus objetivos finales incluyen la eliminación responsable de revisiones, actualizaciones e inicie la colección para realizar la administración de servicios más completa y sin intervención humana. A medida que aumenta la capacidad del servicio para la administración de extremo a otro, las futuras versiones podrían omitir algunas funciones (por ejemplo, reinicio del nodo y aplicación de revisiones automática).

### <a name="security-issues-and-patching"></a>Problemas de seguridad y aplicación de revisiones

Si se encuentra un error de seguridad en uno o más componentes de AKS, el equipo de AKS revisará todos los clústeres afectados para mitigar el problema. Como alternativa, el equipo proporcionará a los usuarios instrucciones de actualización.

Para nodos de trabajo que está disponible un afecta a un error de seguridad, si una revisión sin tiempo de inactividad, el equipo AKS se aplica esa revisión y notificar a los usuarios del cambio.

Cuando una revisión de seguridad requiere el reinicio del nodo de trabajo, Microsoft enviará una notificación a los clientes de este requisito. El cliente es responsable de tener que reiniciar o actualizar para obtener la revisión del clúster. Si los usuarios no aplican las revisiones de acuerdo con las instrucciones de AKS, su clúster seguirá siendo vulnerables frente al problema de seguridad.

### <a name="node-maintenance-and-access"></a>Acceso y el mantenimiento de nodo

Nodos de trabajo son una responsabilidad compartida y pertenecen a los clientes. Por este motivo, los clientes tienen la capacidad de iniciar sesión en sus nodos de trabajo y realizar cambios potencialmente dañinos, como actualizaciones del kernel y para instalar o desinstalar paquetes.

Si los clientes realizar cambios destructivos o que queden sin conexión o dejar de funcionar los componentes de plano de control, AKS detectará este error y restaurar automáticamente el nodo de trabajo en el estado de funcionamiento anterior.

Aunque los clientes pueden iniciar sesión en y cambiar los nodos de trabajo, esto no es recomendable porque los cambios pueden hacer que un clúster de incompatibilidad.

## <a name="network-ports-access-and-nsgs"></a>Puertos de red, acceso y NSG

Como un servicio administrado, AKS tiene requisitos específicos de red y conectividad. Estos requisitos son menos flexibles que los requisitos para los componentes normales de IaaS. En AKS, las operaciones como la personalización de las reglas NSG, bloqueo de un puerto específico (por ejemplo, mediante reglas de firewall que bloquee el puerto de salida 443) y direcciones URL permitidas pueden hacer que el clúster de incompatibilidad.

> [!NOTE]
> Actualmente, AKS no permite bloquear completamente la salida desde el clúster (por ejemplo, dominio explícita o creación de listas blancas de puerto). La lista de puertos y direcciones URL está sujeta a cambios sin previo aviso. Puede obtener la lista actualizada mediante la creación de una incidencia de soporte técnico de Azure. La lista es solo para los clientes que están dispuestos a aceptar que podría verse afectada la disponibilidad del clúster *en cualquier momento.*

## <a name="unsupported-alpha-and-beta-kubernetes-features"></a>Características no admitidas de Kubernetes alfa y beta

AKS admite solo estables características dentro del proyecto ascendente de Kubernetes. A menos que se documentan en caso contrario, AKS no es compatible con alfa y beta de las características que están disponibles en el proyecto ascendente de Kubernetes.

En dos escenarios, podrían lanzarse características alfa o beta antes de estar disponibles con carácter general:

* Los clientes han cumplido con el producto AKS, soporte técnico o los equipos de ingeniería y ha pedido que pruebe estas nuevas características.
* Estas características han sido [habilitadas por una marca de características](https://github.com/Azure/AKS/blob/master/previews.md). Los clientes deben participar explícitamente para usar estas características.

## <a name="preview-features-or-feature-flags"></a>Características de vista previa o las marcas de características

Para las características y funcionalidad que requieren pruebas extendidas y comentarios de los usuarios, Microsoft publica nuevas características de vista previa o características detrás de una marca de características. Tenga en cuenta estas características como las características de versión preliminar o beta.

Características de vista previa o marca de características no están diseñadas para producción. Pueden dar lugar a los cambios continuos en las API y comportamiento, correcciones de errores y otros cambios en tiempo de inactividad y clústeres inestables.

Características en versión preliminar pública son otoño de "mejor esfuerzo" support como estas características están en versión preliminar y no destinada a producción y son compatibles con los equipos de soporte técnico de AKS durante el horario comercial solo. Para obtener más información consulte:

* [Preguntas más frecuentes de soporte técnico de Azure](https://azure.microsoft.com/support/faq/)

> [!NOTE]
> Características de vista previa tendrán efecto al Azure *suscripción* nivel. No instale las características de vista previa en una suscripción de producción. En una suscripción de producción, características de vista previa pueden cambiar el comportamiento de la API de forma predeterminada y afectan a las operaciones normales.

## <a name="upstream-bugs-and-issues"></a>Problemas y errores de nivel superior

Dada la velocidad de desarrollo en el proyecto ascendente de Kubernetes, invariablemente se producen errores. Algunos de estos errores no se pueden revisar o solucionar dentro del sistema AKS. En su lugar, las correcciones de errores requiere mayor revisiones a los proyectos de nivel superior (como Kubernetes, sistemas operativos de nodo o de trabajo y núcleos). Para los componentes que Microsoft posee (por ejemplo, el proveedor de nube de Azure), se confirman a corregir problemas de nivel superior de la Comunidad de AKS y el personal de Azure.

Cuando un problema de soporte técnico es la causa raíz por uno o varios errores de nivel superior, los equipos de soporte técnico e ingeniería de AKS hará lo siguiente:

* Identifique y vincular los errores de nivel superior con los detalles necesarios para ayudar a explicar por qué este problema afecta a su clúster o una carga de trabajo. Los clientes reciben vínculos a los repositorios necesarios para que puedan ver los problemas y ver cuándo una nueva versión proporcionará correcciones.
* Se proporcionan posibles soluciones alternativas o mitigaciones. Si se puede mitigar el problema, un [problema conocido](https://github.com/Azure/AKS/issues?q=is%3Aissue+is%3Aopen+label%3Aknown-issue) se almacenará en el repositorio AKS. Explica la presentación de los problemas conocidos:
  * El problema, lo que incluye vínculos a los errores de nivel superior.
  * La solución y los detalles sobre una actualización u otra persistencia de la solución.
  * Aproximado escalas de tiempo para la inclusión del problema, en función de la cadencia de versión ascendente.
