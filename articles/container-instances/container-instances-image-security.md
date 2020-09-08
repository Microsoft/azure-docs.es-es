---
title: Consideraciones de seguridad para instancias de contenedor
description: Recomendaciones para proteger las imágenes y los secretos de Azure Container Instances, y consideraciones generales de seguridad para cualquier plataforma de contenedores
ms.topic: article
ms.date: 01/10/2020
ms.custom: ''
ms.openlocfilehash: f49f115e10326887cf4d23406437467256b7df2e
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87922241"
---
# <a name="security-considerations-for-azure-container-instances"></a>Consideraciones de seguridad para Azure Container Instances

En este artículo se presentan las consideraciones de seguridad para usar Azure Container Instances para ejecutar aplicaciones de contenedor. Contenido de los temas:

> [!div class="checklist"]
> * **Recomendaciones de seguridad** para administrar imágenes y secretos de Azure Container Instances
> * **Consideraciones para el ecosistema de contenedores** durante el ciclo de vida de un contenedor, para cualquier plataforma de contenedores

Para consultar recomendaciones completas que le sirvan de ayuda a mejorar la posición de seguridad de una implementación, consulte la [Base de referencia de seguridad de Azure para Container Instances](security-baseline.md).


## <a name="security-recommendations-for-azure-container-instances"></a>Recomendaciones de seguridad para Azure Container Instances

### <a name="use-a-private-registry"></a>Uso de un registro privado

Los contenedores se crean a partir de imágenes que están almacenadas en uno o varios repositorios. Estos repositorios pueden pertenecer a un registro público, como [Docker Hub](https://hub.docker.com), o en un registro privado. Un ejemplo de un registro privado es el [Registro de confianza de Docker](https://docs.docker.com/datacenter/dtr/), que puede instalarse de forma local o en una nube privada virtual. También puede usar servicios de registro privado de contenedores basados en la nube, incluido [Azure Container Registry](../container-registry/container-registry-intro.md). 

Una imagen de contenedor disponible públicamente no garantiza la seguridad. Las imágenes de contenedor constan de varias capas de software, y cada capa de software podría tener vulnerabilidades. Para ayudar a reducir la amenaza de ataques, debe almacenar y recuperar las imágenes de un registro privado, como Azure Container Registry o Docker Trusted Registry. Además de proporcionar un registro privado administrado, Azure Container Registry admite [autenticación basada en la entidad de servicio](../container-registry/container-registry-authentication.md) a través de Azure Active Directory para los flujos de autenticación básica. Esta autenticación incluye el acceso basado en roles para permisos de solo lectura (extracción), escritura (inserción) y otros.

### <a name="monitor-and-scan-container-images"></a>Supervisar y analizar las imágenes de contenedor

Aproveche las soluciones para analizar imágenes de contenedor en un registro privado e identificar posibles puntos vulnerables. Es importante comprender el nivel de detalles de la detección de amenazas que proporcionan las distintas soluciones.

Por ejemplo, Azure Container Registry ofrece la opción de [integrarlo con Azure Security Center](../security-center/azure-container-registry-integration.md) para analizar automáticamente todas las imágenes de Linux insertadas en un registro. El analizador Qualys integrado de Azure Security Center detecta puntos vulnerables en las imágenes, las clasifica y proporciona una guía de corrección.

En Azure Marketplace hay disponibles soluciones de supervisión de seguridad y análisis de imágenes, como [Twistlock](https://azuremarketplace.microsoft.com/marketplace/apps/twistlock.twistlock?tab=Overview) y [Aqua Security](https://azuremarketplace.microsoft.com/marketplace/apps/aqua-security.aqua-security?tab=Overview).  

### <a name="protect-credentials"></a>Proteger las credenciales

Los contenedores pueden abarcar varios clústeres y regiones de Azure. Por lo tanto, debe proteger las credenciales necesarias para los inicios de sesión o el acceso a las API, como contraseñas o tokens. Asegúrese de que solo los usuarios con privilegios puedan acceder a esos contenedores en tránsito y en reposo. Haga un inventario de todos los secretos de credenciales y, luego, pida a los desarrolladores que usen las nuevas herramientas de administración de secretos que están diseñadas para plataformas de contenedores.  Asegúrese de que la solución incluya bases de datos cifradas, cifrado TLS para datos de los datos de secretos en tránsito y [control de acceso basado en roles de Azure (RBAC de Azure)](../role-based-access-control/overview.md) con privilegios mínimos. [Azure Key Vault](../key-vault/general/secure-your-key-vault.md) es un servicio en la nube que protege las claves de cifrado y los secretos (como certificados, cadenas de conexión y contraseñas) de las aplicaciones en contenedores. Dado que estos datos son confidenciales y críticos para la empresa, proteja el acceso a los almacenes de claves, de modo que solo las aplicaciones y los usuarios autorizados puedan acceder a ellos.

## <a name="considerations-for-the-container-ecosystem"></a>Consideraciones para el ecosistema de contenedores

Las siguientes medidas de seguridad, implementas correctamente y administradas de forma eficaz, pueden ayudar a asegurar y proteger su ecosistema de contenedores. Estas medidas se aplican en todo el ciclo de vida del contenedor, desde el desarrollo hasta la implementación en producción, y en una gama de plataformas, hosts y orquestadores de contenedores. 

### <a name="use-vulnerability-management-as-part-of-your-container-development-lifecycle"></a>Usar la administración de vulnerabilidades como parte de su ciclo de vida de desarrollo de contenedores 

Al usar una administración de vulnerabilidades eficaz a lo largo del ciclo de vida de desarrollo de los contenedores, mejora las probabilidades de detectar y resolver problemas de seguridad antes de que sean un problema más grave. 

### <a name="scan-for-vulnerabilities"></a>Análisis de vulnerabilidades 

Todo el tiempo se descubren nuevas vulnerabilidades, por lo que el análisis y la identificación de vulnerabilidades es un proceso continuo. Incorpore el análisis de vulnerabilidades a lo largo del ciclo de vida de los contenedores:

* Como comprobación final en la canalización de desarrollo, debe realizar un análisis de vulnerabilidades en los contenedores antes de insertar las imágenes en un registro público o privado. 
* Continúe analizando imágenes de contenedor en el registro tanto para identificar cualquier error que pueda haberse pasado durante el desarrollo como para abordar las vulnerabilidades descubiertas recientemente que puedan existir en el código usado en las imágenes de contenedores.  

### <a name="map-image-vulnerabilities-to-running-containers"></a>Asignar vulnerabilidades de imágenes a contenedores en ejecución 

Debe tener un medio para asignar las vulnerabilidades identificadas en las imágenes de contenedor a los contenedores en ejecución, de modo que los problemas de seguridad puedan mitigarse.  

### <a name="ensure-that-only-approved-images-are-used-in-your-environment"></a>Asegurarse de que solo se usen imágenes aprobadas en su entorno 

Existen suficientes cambios y volatilidad en un ecosistema de contenedores sin que se permitan, además, contenedores desconocidos. Permita solo imágenes de contenedor aprobadas. Ponga en práctica herramientas y procesos para supervisar y evitar el uso de imágenes de contenedor no aprobadas. 

Una forma eficaz de reducir la superficie de ataque e impedir que los desarrolladores cometan errores de seguridad críticos consiste en controlar el flujo de imágenes de contenedor en el entorno de desarrollo. Por ejemplo, podría autorizar una única distribución de Linux como imagen de base, preferiblemente una que sea eficiente (Alpine o CoreOS, en lugar de Ubuntu), para minimizar la superficie de ataques potenciales. 

La firma o creación de huellas digitales para imágenes puede generar una cadena de custodia que le permita verificar la integridad de los contenedores. Por ejemplo, Azure Container Registry es compatible con el modelo de [contenido confianza](https://docs.docker.com/engine/security/trust/content_trust) de Docker, que permite a los editores de imágenes firmar las imágenes que se insertan en un registro, y a los consumidores de imágenes extraer solo imágenes firmadas.

### <a name="permit-only-approved-registries"></a>Permitir solo registros aprobados 

Una extensión para garantizar que su entorno use solo las imágenes aprobadas radica en permitir únicamente el uso de los registros de contenedor aprobados. Requerir el uso de registros de contenedor aprobados reduce la exposición a riesgos, ya que se limita la posibilidad de que se introduzcan vulnerabilidades o problemas de seguridad desconocidos. 

### <a name="ensure-the-integrity-of-images-throughout-the-lifecycle"></a>Garantizar la integridad de las imágenes a lo largo del ciclo de vida 

Parte de la administración de la seguridad a lo largo del ciclo de vida del contenedor es garantizar la integridad de las imágenes de contenedor en el registro, así como en la medida en que se modifican o implementan en producción. 

* No debe permitirse que imágenes con vulnerabilidades, aunque sean menores, se ejecuten en un entorno de producción. Lo ideal es que todas las imágenes implementadas en producción se guarden en un registro privado donde solo algunas personas tengan acceso. Mantenga un número pequeño de imágenes de producción para asegurarse de que se puedan administrar eficazmente.

* Dado que es difícil identificar el origen del software desde una imagen de contenedor disponible públicamente, cree imágenes desde la fuente para garantizar el conocimiento del origen de la capa. Cuando surge una vulnerabilidad en una imagen de contenedor generada automáticamente, los clientes pueden encontrar una ruta más rápida a una resolución. Con una imagen pública, los clientes tendrían que encontrar la raíz de una imagen pública para corregirla u obtener otra imagen segura del publicador. 

* No se garantiza que una imagen digitalizada exhaustivamente implementada en producción esté actualizada durante la vigencia de la aplicación. Se puede informar de las vulnerabilidades de seguridad para las capas de la imagen que no se conocían anteriormente o que se introdujeron después de la implementación de producción. 

  Haga auditorías periódicas de imágenes implementadas en producción para identificar las imágenes que no están al día o que no se han actualizado en un tiempo. Puede usar metodologías de implementación de verde-azul y activar mecanismos de actualización para actualizar las imágenes de contenedor sin tiempo de inactividad. Puede analizar las imágenes con las herramientas descritas en la sección anterior. 

* Use una canalización de integración continua (CI) con análisis de seguridad integrada para crear imágenes seguras e insertarlas en su registro privado. El análisis de vulnerabilidad creado en la solución de CI garantiza que las imágenes que pasan todas las pruebas se incluyan en el registro privado desde el que se implementan las cargas de trabajo de producción. 

  Un error de canalización de CI garantiza que las imágenes vulnerables no se incluyan en el registro privado usado para las implementaciones de cargas de trabajo de producción. También automatiza el análisis de seguridad de imágenes si hay un número significativo de imágenes. De lo contrario, auditar las imágenes manualmente para buscar vulnerabilidades de seguridad puede ser bastante lento y estar propenso a errores. 

### <a name="enforce-least-privileges-in-runtime"></a>Exigir privilegios mínimos en tiempo de ejecución 

El concepto de privilegios mínimos es una práctica recomendada de seguridad básica que también se aplica a los contenedores. Cuando se aprovecha una vulnerabilidad, en general da al atacante acceso y privilegios iguales a los de la aplicación o proceso en peligro. Asegurarse de que los contenedores funcionen con privilegios mínimos y acceso requerido para realizar el trabajo reduce la exposición a riesgos. 

### <a name="reduce-the-container-attack-surface-by-removing-unneeded-privileges"></a>Reducir la superficie de ataque a contenedores mediante la eliminación de privilegios innecesarios 

También puede minimizar la superficie potencial de ataque al eliminar del tiempo de ejecución del contenedor los procesos o privilegios innecesarios o sin usar. Los contenedores con privilegios se ejecutan como raíz. Si un usuario o una carga de trabajo malintencionados se cuela en un contenedor con privilegios, el contenedor se ejecutará como raíz en el sistema.

### <a name="preapprove-files-and-executables-that-the-container-is-allowed-to-access-or-run"></a>Preaprobación de archivos y ejecutables a los que el contenedor puede acceder o que este puede ejecutar 

Reducir el número de variables o valores desconocidos le ayuda a mantener un entorno estable y confiable. Limitar los contenedores para que puedan solo acceder a archivos o ejecutables incluidos en una lista segura o para que puedan ejecutarlos es un método demostrado de limitar la exposición a riesgos.  

Es mucho más fácil administrar una lista segura cuando se implementa desde un principio. Una lista segura proporciona una medida de control y capacidad de administración a medida que aprende qué archivos y ejecutables son necesarios para que la aplicación funcione correctamente. 

Una lista segura no solo reduce la superficie expuesta a ataques, sino que puede también proporcionar una línea de base para encontrar anomalías y evitar los casos de uso de los escenarios de "vecino ruidoso" y división de contenedores. 

### <a name="enforce-network-segmentation-on-running-containers"></a>Aplicar la segmentación de la red en contenedores en ejecución  

Para ayudar a proteger los contenedores en una subred frente a los riesgos de seguridad de otra subred, mantenga la segmentación de la red (o nanosegmentación) o segregación entre contenedores en ejecución. También puede ser necesario mantener la segmentación de red para usar contenedores en sectores que deben cumplir exigencias de cumplimiento.  

Por ejemplo, la herramienta de partner [Aqua](https://azuremarketplace.microsoft.com/marketplace/apps/aqua-security.aqua-security?tab=Overview) proporciona un enfoque automatizado para la nanosegmentación. Aqua supervisa las actividades de red de los contenedores en tiempo de ejecución. Identifica todas las conexiones de red entrantes y salientes hacia y desde otros contenedores, servicios, direcciones IP e Internet pública. La nanosegmentación se crea automáticamente en función del tráfico supervisado. 

### <a name="monitor-container-activity-and-user-access"></a>Supervisar la actividad de los contenedores y el acceso de los usuarios 

Al igual que con cualquier entorno de TI, debe supervisar constantemente la actividad y el acceso de los usuario al ecosistema de contenedores para identificar rápidamente cualquier actividad sospechosa o malintencionada. Azure proporciona soluciones de supervisión de contenedores, entre otras:

* [Azure Monitor para contenedores](../azure-monitor/insights/container-insights-overview.md) para supervisar el rendimiento de las cargas de trabajo implementadas en entornos de Kubernetes hospedados en Azure Kubernetes Service (AKS). Azure Monitor para contenedores le brinda la posibilidad de visibilizar el rendimiento mediante la recopilación de métricas del procesador y de la memoria de los controladores, nodos y contenedores disponibles en Kubernetes mediante la API de métricas. 

* La [solución de supervisión de contenedores en Azure](../azure-monitor/insights/containers.md) le ayuda a ver y administrar otros hosts de contenedores de Docker y Windows en una sola ubicación. Por ejemplo:

  * Ver información de auditoría detallada que muestre los comandos usados con los contenedores. 
  * Solucionar los problemas de los contenedores al ver y buscar registros centralizados sin tener que ver los hosts de Docker o Windows de forma remota.  
  * Buscar los contenedores que puedan causar ruido o consuman un exceso de recursos en un host.
  * Ver la información centralizada acerca de la CPU, la memoria, el almacenamiento, y el uso y el rendimiento de la red en relación con los contenedores.  

  La solución admite orquestadores de contenedores como Docker Swarm, DC/OS, Kubernetes no administrado, Service Fabric y Red Hat OpenShift. 

### <a name="monitor-container-resource-activity"></a>Supervisar la actividad de recursos de contenedor 

Supervise la actividad de los recursos, como archivos, red y otros recursos, a los que acceden los contenedores. Supervisar el consumo y la actividad de los recursos resulta útil para la supervisión de rendimiento y como medida de seguridad. 

[Azure Monitor](../azure-monitor/overview.md) habilita la supervisión básica del servicio de Azure al permitir la recopilación de métricas, registros de actividad y registros de diagnóstico. Por ejemplo, el registro de actividad le indica cuándo se crean o modifican los recursos. 

  Se encuentran disponibles métricas que ofrecen estadísticas de rendimiento de diferentes recursos e incluso del sistema operativo de una máquina virtual. Puede ver estos datos con uno de los exploradores en Azure Portal y crear alertas basadas en estas métricas. Azure Monitor ofrece la canalización de métricas más rápida (desde 5 minutos hasta 1 minuto), por lo que debe usarse para las notificaciones y alertas donde el tiempo es fundamental. 

### <a name="log-all-container-administrative-user-access-for-auditing"></a>Registrar todos los accesos administrativos de usuarios al contenedor para auditoría 

Mantenga un registro de auditoría preciso del acceso administrativo a su ecosistema de contenedores, incluido el clúster de Kubernetes, el registro de contenedores y las imágenes de contenedor. Estos registros podrían ser necesarios para fines de auditoría y serán útiles como prueba forense después de un incidente de seguridad. Las soluciones de Azure incluyen:

* [Integración de Azure Kubernetes Service con Azure Security Center](../security-center/azure-kubernetes-service-integration.md) para supervisar la configuración de seguridad del entorno del clúster y generar recomendaciones de seguridad
* [Solución de supervisión de contenedores de Azure](../azure-monitor/insights/containers.md)
* Registros de recurso para [Azure Container Instances](container-instances-log-analytics.md) y [Azure Container Registry](../container-registry/container-registry-diagnostics-audit-logs.md)

## <a name="next-steps"></a>Pasos siguientes

* Consulte en la [Base de referencia de seguridad de Azure para Container Instances](security-baseline.md) recomendaciones completas que le servirán para mejorar la posición de seguridad de su implementación.

* Más información sobre el uso de [Azure Security Center](../security-center/container-security.md) para la detección de amenazas en tiempo real en los entornos en contenedores.

* Más información sobre cómo administrar las vulnerabilidades de contenedor con las soluciones [Twistlock](https://www.twistlock.com/solutions/microsoft-azure-container-security/) y [Aqua Security](https://www.aquasec.com/solutions/azure-container-security/).
