---
title: Directivas de soporte técnico para Azure Kubernetes Service (AKS)
description: Aprenda sobre las directivas de soporte técnico de Azure Kubernetes Service (AKS), la responsabilidad compartida y las características en versión preliminar (o alfa o beta).
services: container-service
author: jnoller
ms.topic: article
ms.date: 01/24/2020
ms.author: jenoller
ms.openlocfilehash: c51b5c4d9cd7362aeda0a0998d8031d0ba358ce1
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89012354"
---
# <a name="support-policies-for-azure-kubernetes-service"></a>Directivas de soporte técnico para Azure Kubernetes Service

En este artículo se proporcionan detalles sobre las directivas de soporte técnico y las limitaciones de Azure Kubernetes Service (AKS). En el artículo también se describen la administración de los nodos de trabajo, los componentes del plano de control administrado, los componentes de código abierto de terceros y la administración de la seguridad o las revisiones.

## <a name="service-updates-and-releases"></a>Actualizaciones de servicio y versiones

* Para información sobre la versión, consulte las [notas de la versión de AKS](https://github.com/Azure/AKS/releases).
* Para información sobre las características en versión preliminar, consulte [AKS preview features and related projects](https://awesomeopensource.com/projects/aks?categoryPage=11) (Características en versión preliminar y proyectos relacionados de AKS).

## <a name="managed-features-in-aks"></a>Características administradas en AKS

Los componentes base de la nube de infraestructura como servicio (IaaS), como los de proceso o red, proporcionan a los usuarios acceso a opciones de personalización y controles de nivel bajo. Por el contrario, AKS proporciona una implementación de Kubernetes llave en mano que ofrece a los clientes el conjunto habitual de configuraciones y funcionalidades que necesitan. Los clientes de AKS tienen limitada la personalización, la implementación y otras opciones. Estos clientes no tienen que preocuparse de administrar directamente los clústeres de Kubernetes.

Con AKS, el cliente obtiene un *plano de control* totalmente administrado. El plano de control contiene todos los componentes y servicios que el cliente debe ofrecer y proporciona clústeres de Kubernetes para los usuarios finales. Todos los componentes de Kubernetes los mantiene y opera Microsoft.

Microsoft administra y supervisa los siguientes componentes mediante el panel de control:

* Servidores de la API de Kubernetes o Kubelet
* Etcd o un almacén de pares clave-valor compatible, que proporciona calidad de servicio (QoS), escalabilidad y tiempo de ejecución
* Servicios DNS (por ejemplo, kube-dns o CoreDNS)
* Proxy o red de Kubernetes

AKS no es una solución de clúster totalmente administrado. Algunos componentes, como los nodos de trabajo, tienen *responsabilidad compartida* con los usuarios para ayudar a mantener el clúster de AKS. Se requiere la participación del usuario, por ejemplo, para aplicar una revisión de seguridad del sistema operativo en el nodo de trabajo.

Los servicios son *administrados* en el sentido en que Microsoft y el equipo de AKS implementan y operan el servicio, y son responsables de su disponibilidad y funcionalidad. Los clientes no pueden modificar estos componentes administrados. Microsoft limita la personalización para garantizar una experiencia de usuario coherente y escalable. Para una solución totalmente personalizable, consulte el [motor de AKS](https://github.com/Azure/aks-engine).

## <a name="shared-responsibility"></a>Responsabilidad compartida

Al crear un clúster, el cliente define los nodos de trabajo de Kubernetes que AKS crea. Las cargas de trabajo de cliente se ejecutan en esos nodos. Los clientes poseen y pueden ver o modificar los nodos de trabajo.

Dado que los nodos del clúster de cliente ejecutan código privado y almacenan información confidencial, el Soporte técnico de Microsoft solo puede acceder a ellos con restricciones. El Soporte técnico de Microsoft no puede iniciar sesión en estos nodos, ejecutar comandos en ellos ni ver sus registros sin el consentimiento expreso o la asistencia del cliente.

Dado que los nodos de trabajo son confidenciales, Microsoft presta mucha atención para limitar su administración en segundo plano. En muchos casos, la carga de trabajo seguirá ejecutándose aunque fallen los nodos maestros de Kubernetes, etcd u otros componentes administrados por Microsoft. Modificar los nodos de trabajo sin prestar atención puede causar la pérdida de datos y de cargas de trabajo y que el clúster deje de ser admitido.

## <a name="aks-support-coverage"></a>Cobertura de soporte de AKS

Microsoft proporciona soporte técnico para lo siguiente:

> [!NOTE]
> Todas las acciones de clúster realizadas por Microsoft/AKS se realizan con el consentimiento del usuario en un rol de Kubernetes integrado `aks-service` y el enlace de roles integrado `aks-service-rolebinding`. Este rol permite a AKS solucionar problemas de clústeres, pero no puede modificar permisos ni crear roles, enlaces de roles ni otras acciones de privilegios elevados. El acceso a roles solo se habilita en incidencias de soporte técnico activos con acceso Just-in-Time (JIT).

* Conectividad a todos los componentes de Kubernetes que el servicio Kubernetes proporciona y admite, como el servidor de API.
* La administración, el tiempo de actividad, la calidad de servicio y las operaciones de los servicios del plano de control de Kubernetes (por ejemplo, los nodos maestros de Kubernetes, el servidor de API, etcd y kube-dns).
* Etcd. La compatibilidad incluye las copias de seguridad automatizadas y transparentes de todos los datos de etcd cada 30 minutos para la restauración del estado del clúster y la planificación ante desastres. Estas copias de seguridad no están directamente disponibles para los clientes o los usuarios. Garantizan la coherencia y confiabilidad de los datos. Etcd. la reversión a petición o la restauración no se admiten como una característica.
* Los puntos de integración en el controlador del proveedor de nube de Azure para Kubernetes. Estos incluyen las integraciones en otros servicios de Azure, como los equilibradores de carga, los volúmenes persistentes o las redes (Kubernetes y Azure CNI).
* Preguntas o problemas sobre la personalización de los componentes del plano de control, como el servidor de API de Kubernetes, etcd y kube-dns.
* Problemas relacionados con las redes, como Azure CNI, kubenet, u otros problemas de acceso de red y funcionalidades. Podrían incluir los de resolución de DNS, pérdida de paquetes, enrutamiento, etc. Microsoft admite diversos escenarios de redes:
  * Kubenet (básico) y redes avanzadas (Azure CNI) dentro del clúster y componentes asociados
  * Conectividad con otros servicios y aplicaciones de Azure
  * Controladores de entrada y las configuraciones del equilibrador de carga o de entrada
  * Rendimiento y latencia de la red

Microsoft no proporciona soporte técnico para lo siguiente:

* Preguntas acerca de cómo usar Kubernetes. Por ejemplo, el Soporte técnico de Microsoft no aconseja sobre la creación controladores de entrada personalizados, el uso de cargas de trabajo de aplicación o la aplicación de herramientas o paquetes de software de código abierto de terceros.
  > [!NOTE]
  > El Soporte técnico de Microsoft puede aconsejar sobre la funcionalidad del clúster de AKS, la personalización y los ajustes (por ejemplo, los problemas y los procedimientos de las operaciones de Kubernetes).
* Proyectos de código abierto de terceros que no se proporcionan como parte del plano de control de Kubernetes o se implementan con clústeres de AKS. Estos proyectos pueden incluir Istio, Helm, Envoy u otros.
  > [!NOTE]
  > Microsoft puede hacer lo posible por ayudar con proyectos de código abierto de terceros como Helm y Kured. Cuando la herramienta de código abierto de terceros integre el proveedor de nube de Azure de Kubernetes u otros errores específicos de AKS, Microsoft es compatible con ejemplos y aplicaciones de la documentación de Microsoft.
* Software de código cerrado de terceros. Este software puede incluir herramientas de análisis de seguridad y dispositivos o software de red.
* Problemas relacionados con la nube múltiple o con construcciones de varios proveedores. Por ejemplo, Microsoft no admite los problemas relacionados con la ejecución de una solución de proveedor en la nube federado para distintas audiencias.
* Personalizaciones de red distintas de las que se enumeran en la [documentación de AKS](./index.yml).
  > [!NOTE]
  > Microsoft sí que admite problemas y errores relacionados con los grupos de seguridad de red (NSG). Por ejemplo, el Soporte técnico de Microsoft puede responder preguntas sobre errores de actualización de los grupos de seguridad de red o un comportamiento imprevisto de estos o del equilibrador de carga.

## <a name="aks-support-coverage-for-worker-nodes"></a>Cobertura de soporte técnico de AKS para los nodos de trabajo

### <a name="microsoft-responsibilities-for-aks-worker-nodes"></a>Responsabilidades de Microsoft sobre los nodos de trabajo de AKS

Microsoft y los clientes comparten la responsabilidad sobre los nodos de trabajo de Kubernetes donde:

* La imagen del sistema operativo base ha requerido adiciones (por ejemplo, agentes de red y de supervisión).
* Los nodos de trabajo reciban automáticamente revisiones del sistema operativo.
* Los problemas con los componentes del plano de control de Kubernetes que se ejecuten en los nodos de trabajo se remedian automáticamente. Los componentes incluyen los siguientes:
  * El proxy de Kube
  * Túneles de red que proporcionan rutas de acceso de comunicación para los componentes maestros de Kubernetes
  * Kubelet
  * Demonio de Docker o Moby

> [!NOTE]
> En un nodo de trabajo, si un componente del plano de control no está operativo, el equipo de AKS podría necesitar reiniciar componentes individuales o el nodo de trabajo por completo. Estas operaciones de reinicio se automatizan y proporcionan corrección automática de los problemas comunes. Estos reinicios solo se producen a nivel de _nodo_ y no en el clúster, a menos que se produzca una interrupción o un mantenimiento de emergencia.

### <a name="customer-responsibilities-for-aks-worker-nodes"></a>Responsabilidades del cliente sobre los nodos de trabajo de AKS

Microsoft no reinicia automáticamente los nodos de trabajo para aplicar revisiones de nivel de sistema operativo. Aunque las revisiones del sistema operativo se entregan a los nodos de trabajo, el *cliente* es responsable de reiniciar los nodos de trabajo para aplicar los cambios. Las revisiones a las bibliotecas compartidas, los demonios como la unidad de estado sólido híbrida (SSHD) y otros componentes del nivel del sistema o del sistema operativo se realizan automáticamente.

Los clientes son responsables de ejecutar las actualizaciones de Kubernetes. Pueden ejecutar las actualizaciones mediante el panel de control de Azure o la CLI de Azure. Esto se aplica a las actualizaciones que contienen mejoras de seguridad o funcionalidad para Kubernetes.

#### <a name="user-customization-of-worker-nodes"></a>Personalización de usuario de los nodos de trabajo
> [!NOTE]
> Los nodos de trabajo de AKS aparecen en Azure Portal como recursos de Azure IaaS normales. No obstante, estas máquinas virtuales se implementan en un grupo de recursos de Azure personalizado (con el prefijo MC\\\*). Es posible aumentar los nodos de trabajo de AKS desde sus configuraciones base. Por ejemplo, puede usar Secure Shell (SSH) para cambiar los nodos de trabajo de AKS de la misma forma que cambia las máquinas virtuales normales. Sin embargo, no puede cambiar la imagen base del sistema operativo. Es posible que no se conserven los cambios personalizados si realiza una actualización, un escalado o un reinicio. **Sin embargo**, si hace cambios *fuera de banda y fuera del ámbito de la API de AKS* el clúster de AKS dejará de ser compatible. Evite cambiar los nodos de trabajo a menos que el Soporte técnico de Microsoft se lo solicite.

La emisión de operaciones no compatibles, como se definió anteriormente, como la desasignación de todos los nodos de agente, hace que el clúster deje de ser compatible. AKS se reserva el derecho de archivar los planos de control que se han configurado fuera de las directrices de compatibilidad durante períodos prolongados iguales o superiores a 30 días. AKS mantiene copias de seguridad de los metadatos de etcd del clúster y puede reasignar fácilmente el clúster. Esta reasignación puede iniciarse con cualquier operación PUT haciendo que el clúster vuelva a ser compatible, como una actualización o un escalado a los nodos de agente activos.

AKS administra el ciclo de vida y las operaciones de los nodos de trabajo en nombre de los clientes, por lo que **no admite** la modificación de los recursos de IaaS asociados a los nodos de trabajo. Un ejemplo de una operación no admitida es la personalización de un conjunto de escalado de máquinas virtuales del grupo de nodos mediante el cambio manual de las configuraciones en VMSS a través del portal o la API de VMSS.
 
En el caso de las configuraciones o paquetes específicos de la carga de trabajo, AKS recomienda usar [controladores DaemonSet de Kubernetes](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

El uso de contenedores de tipo init y controladores DaemonSet con privilegios de Kubernetes permite a los clientes ajustar, modificar o instalar software de terceros en los nodos de trabajo del clúster. Algunos ejemplos de estas personalizaciones incluyen agregar software personalizado de examen de seguridad o actualizar la configuración de sysctl.

Aunque se trata de una ruta de acceso recomendada si se cumplen los requisitos anteriores, los servicios de ingeniería y soporte técnico de AKS no pueden ayudarle en la solución de problemas, en el diagnóstico de modificaciones interrumpidas o que no funcionen, ni en aquellas modificaciones que indiquen que el nodo no está disponible debido a un controlador de DaemonSet que haya implementado un cliente.

> [!NOTE]
> Dado que AKS es un *servicio administrado*, sus objetivos finales incluyen eximir de responsabilidad sobre las revisiones, las actualizaciones y la recopilación de registros para que la administración de servicios sea más completa y discreta. A medida que aumenta la capacidad del servicio para la administración de un extremo a otro, las versiones futuras podrían omitir algunas funciones (por ejemplo, el reinicio del nodo y aplicación de revisiones automática).

### <a name="security-issues-and-patching"></a>Problemas de seguridad y aplicación de revisiones

Si se encuentra un error de seguridad en uno o más componentes de AKS, el equipo de AKS revisará todos los clústeres afectados para mitigar el problema. Como alternativa, el equipo proporciona a los usuarios instrucciones de actualización.

Para los nodos de trabajo afectados por un error de seguridad, si hay una revisión sin interrupción de la actividad disponible, el equipo de AKS la aplica y notifica a los usuarios del cambio.

Cuando una revisión de seguridad requiera el reinicio del nodo de trabajo, Microsoft envía una notificación a los clientes de este requisito. El cliente es responsable del reinicio o la actualización para obtener la revisión del clúster. Si el usuario no aplica las revisiones de acuerdo con las instrucciones de AKS, el clúster seguirá siendo vulnerable frente al problema de seguridad.

### <a name="node-maintenance-and-access"></a>Acceso a los nodos y mantenimiento

Los nodos de trabajo son una responsabilidad compartida y pertenecen a los clientes. Por este motivo, los clientes tienen la capacidad de iniciar sesión en sus nodos de trabajo y de realizar cambios potencialmente dañinos, como las actualizaciones del kernel, o la instalación o eliminación de paquetes.

Si los clientes realizan cambios destructivos o desconectan los componentes del plano de control o impiden su funcionamiento, AKS detecta el error y restaura automáticamente el nodo de trabajo al estado de funcionamiento anterior.

Aunque los clientes pueden iniciar sesión en los nodos de trabajo y cambiarlos, esto no es recomendable, ya que los cambios pueden hacer que un clúster deje de ser compatible.

## <a name="network-ports-access-and-nsgs"></a>Puertos de red, acceso y grupos de seguridad de red

Como servicio administrado, AKS tiene requisitos específicos de red y de conectividad. Estos requisitos son menos flexibles que los requisitos para los componentes normales de IaaS. En AKS, las operaciones como la personalización de las reglas de los grupos de seguridad de red, el bloqueo de un puerto específico (por ejemplo, mediante reglas de firewall que bloqueen el puerto de salida 443) y la adición de direcciones URL a una lista de permitidos pueden hacer que el clúster deje de ser compatible.

> [!NOTE]
> Actualmente, AKS no permite bloquear completamente el tráfico de salida del clúster. Para controlar la lista de direcciones URL y puertos que puede usar el clúster para el tráfico saliente, vea [Limitar el tráfico de salida](limit-egress-traffic.md).

## <a name="unsupported-alpha-and-beta-kubernetes-features"></a>Características de Kubernetes alfa y beta no admitidas

Dentro del proyecto de Kubernetes ascendente, AKS solo admite características estables. A menos que se documente lo contrario, AKS no admite características alfa y beta que estén disponibles en el proyecto de Kubernetes ascendente.

En dos escenarios, las características alfa o beta se podrían introducir antes de que tengan disponibilidad general:

* Los clientes hayan conocido a los equipos de producto de AKS, soporte técnico o ingeniería, y que les hayan pedido que prueben las nuevas características.
* Estas características han sido [habilitadas por una marca de característica](https://awesomeopensource.com/projects/aks?categoryPage=11). Los clientes deben participar explícitamente para usar estas características.

## <a name="preview-features-or-feature-flags"></a>Características en versión preliminar o marcas de característica

Para las características y la funcionalidad que requieren pruebas ampliadas y comentarios de los usuarios, Microsoft publica nuevas características en versión preliminar o características con una marca de característica. Considere estas características como versión preliminar o características beta.

Las características en versión preliminar o con marca de característica no son para producción. Los cambios continuos en las API y el comportamiento, las soluciones de errores y otros cambios pueden dar lugar a inestabilidad en los clústeres y tiempo de inactividad.

Las características en versión preliminar pública suponen el mejor esfuerzo de soporte, por estar en versión preliminar y no estar previstas para producción; se ocupan de ellas los equipos de soporte técnico de AKS únicamente durante el horario comercial. Para información adicional, consulte:

* [Preguntas más frecuentes de soporte técnico de Azure](https://azure.microsoft.com/support/faq/)

> [!NOTE]
> Las características en versión preliminar surtirán efecto a nivel de *suscripción* de Azure. No instale las características en versión preliminar en una suscripción de producción. Aquí, las característica en versión preliminar pueden cambiar el comportamiento predeterminado de API y afectar a las operaciones normales.

## <a name="upstream-bugs-and-issues"></a>Problemas y errores ascendentes

Dada la velocidad de desarrollo del proyecto ascendente de Kubernetes, siempre se producen errores. Algunos de estos errores no se pueden revisar o solucionar dentro del sistema de AKS. En su lugar, las correcciones de errores requieren mayores revisiones en los proyectos ascendentes (como Kubernetes, los sistemas operativos del nodo o de trabajo y kernels). Para los componentes que pertenecen a Microsoft (por ejemplo, el proveedor de nube de Azure), el personal de AKS y Azure se comprometen a solucionar los problemas ascendentes en la comunidad.

Cuando un problema de soporte técnico lo causa originalmente uno o más errores ascendentes, los equipos de soporte técnico e ingeniería de AKS:

* Identificarán y vincularán los errores ascendentes con detalles que lo justifiquen para intentar explicar por qué afecta al clúster o a la carga de trabajo. Los clientes recibirán vínculos a los repositorios necesarios para que puedan consultar los problemas y cuándo una nueva versión proporciona correcciones.
* Proporcionarán posibles soluciones alternativas o mitigaciones. Si se puede mitigar el problema, se archivará un [problema conocido](https://github.com/Azure/AKS/issues?q=is%3Aissue+is%3Aopen+label%3Aknown-issue) en el repositorio de AKS. En el archivo de problemas conocidos se explica:
  * El problema, con vínculos a los errores ascendentes.
  * La solución y los detalles sobre una actualización u otra manera de que la solución persista.
  * Escalas de tiempo aproximadas de la inclusión del problema en función de la cadencia de la versión ascendente.
