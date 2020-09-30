---
title: Archivo de novedades de Azure Security Center
description: Una descripción de las novedades y los cambios en Azure Security Center desde hace seis meses y versiones anteriores.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/11/2020
ms.author: memildin
ms.openlocfilehash: 2012f590aa833bea1b49955958095917a6479665
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89438575"
---
# <a name="archive-for-whats-new-in-azure-security-center"></a>Archivo de novedades de Azure Security Center

La página principal de notas de la versión de [novedades de Azure Security Center](release-notes.md) contiene actualizaciones de los últimos seis meses, mientras que esta página contiene elementos más antiguos.

En este página se proporciona información acerca de lo siguiente:

- Nuevas características
- Corrección de errores
- Funciones obsoletas


## <a name="march-2020"></a>Marzo de 2020

Las actualizaciones de marzo incluyen:

- [La automatización del flujo de trabajo ya está disponible con carácter general](#workflow-automation-is-now-generally-available)
- [Integración de Azure Security Center con Windows Admin Center](#integration-of-azure-security-center-with-windows-admin-center)
- [Protección para Azure Kubernetes Service](#protection-for-azure-kubernetes-service)
- [Experiencia Just-in-Time mejorada](#improved-just-in-time-experience)
- [Dos recomendaciones de seguridad para las aplicaciones web en desuso](#two-security-recommendations-for-web-applications-deprecated)


### <a name="workflow-automation-is-now-generally-available"></a>La automatización del flujo de trabajo ya está disponible con carácter general

La característica de automatización del flujo de trabajo de Azure Security Center ahora está disponible con carácter general. Úsela para desencadenar automáticamente una instancia de Logic Apps sobre alertas y recomendaciones de seguridad. Además, los desencadenadores manuales están disponibles para las alertas y todas las recomendaciones que tienen la opción corrección rápida disponible.

Cada programa de seguridad incluye varios flujos de trabajo para la respuesta a incidentes. Estos procesos pueden incluir notificaciones para las partes interesadas competentes, iniciar un proceso de administración de cambios y aplicar pasos de corrección específicos. Los expertos en seguridad recomiendan automatizar tantos pasos de esos procedimientos como sea posible. La automatización reduce la sobrecarga y puede mejorar la seguridad asegurándose de que los pasos del proceso se realizan de forma rápida, coherente y según sus requisitos predefinidos.

Para más información acerca de las capacidades de Security Center automática y manual para ejecutar los flujos de trabajo, consulte [ automatización de flujos de trabajo](workflow-automation.md).

Más información acerca de la [creación de Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-overview).


### <a name="integration-of-azure-security-center-with-windows-admin-center"></a>Integración de Azure Security Center con Windows Admin Center

Ahora es posible migrar los servidores de Windows locales desde Windows Admin Center directamente al Azure Security Center. Security Center, a continuación, se convierte en el único panel para ver la información de seguridad de todos los recursos de Windows Admin Center, incluidos los servidores locales, las máquinas virtuales y las cargas de trabajo PaaS adicionales.

Después de mover un servidor de Windows Admin Center a Azure Security Center, podrá realizar lo siguiente:

- Ver alertas y recomendaciones de seguridad en la extensión de Security Center de Windows Admin Center.
- Ver la postura de seguridad y recuperar información detallada adicional de los servidores administrados de Windows Admin Center en Security Center, mediante Azure Portal (o a través de una API).

Más información sobre [cómo integrar Azure Security Center con Windows Admin Center](windows-admin-center-integration.md).


### <a name="protection-for-azure-kubernetes-service"></a>Protección para Azure Kubernetes Service

Azure Security Center está expandiendo sus características de seguridad de contenedor para proteger Azure Kubernetes Service (AKS).

La popular plataforma de código abierto Kubernetes se ha adoptado tan ampliamente que ahora es un estándar del sector para la orquestación de contenedores. A pesar de esta implementación generalizada, todavía hay una falta de comprensión sobre cómo proteger un entorno de Kubernetes. Defender las superficies de ataque de una aplicación en contenedores requiere conocimientos para asegurarse de que la infraestructura esté configurada de forma segura y se supervise constantemente para detectar posibles amenazas.

La defensa de Security Center incluye:

- **Detección y visibilidad**: detección continua de instancias de AKS administradas dentro de las suscripciones registradas de Security Center.
- **Recomendaciones de seguridad**: recomendaciones útiles para ayudarle a cumplir los procedimientos recomendados de seguridad para AKS. Estas recomendaciones se incluyen en su puntuación segura para asegurarse de que se ven como parte de la postura de seguridad de su organización. Un ejemplo de una recomendación relacionada con AKS que podría ver es que "se debe usar el control de acceso basado en roles para restringir el acceso a un clúster de servicio de Kubernetes".
- **Protección contra amenazas**: gracias al análisis continuo de la implementación de AKS, Security Center le alerta de las amenazas y actividades malintencionadas que se detectan en el host y el nivel de clúster de AKS.

Más información acerca de [integración de Azure Kubernetes Services con Security Center](azure-kubernetes-service-integration.md).

Más información sobre [las características de seguridad de los contenedores en Security Center](container-security.md).


### <a name="improved-just-in-time-experience"></a>Experiencia Just-in-Time mejorada

Las características, la operación y la interfaz de usuario de las herramientas Just-in-Time de Azure Security Center que protegen los puertos de administración se han mejorado de la manera siguiente: 

- **Campo de justificación**: al solicitar acceso a una máquina virtual (VM) a través de la página Just-in-Time del Azure Portal, hay un nuevo campo opcional disponible para especificar una justificación para la solicitud. Se puede realizar un seguimiento de la información especificada en este campo en el registro de actividad. 
- **Limpieza automática de reglas Just-In-Time (JIT) redundantes**: siempre que se actualiza una directiva JIT, se ejecuta automáticamente una herramienta de limpieza para comprobar la validez de todo el conjunto de reglas. La herramienta busca discrepancias entre las reglas de la directiva y las reglas del grupo de seguridad de red. Si la herramienta de limpieza encuentra una discrepancia, determina la causa y, cuando es seguro hacerlo, quita las reglas integradas que ya no son necesarias. El limpiador no elimina nunca las reglas que ha creado. 

Más información acerca de [la característica de acceso JIT](security-center-just-in-time.md).


### <a name="two-security-recommendations-for-web-applications-deprecated"></a>Dos recomendaciones de seguridad para las aplicaciones web en desuso

Dos recomendaciones de seguridad relacionadas con las aplicaciones web están en desuso: 

- Se deben proteger las reglas de las aplicaciones web en los grupos de seguridad de red de IaaS.
    (Directiva relacionada: Se deben proteger las reglas de NSG para las aplicaciones web en IaaS).

- El acceso a App Services debe estar restringido.
    (Directiva relacionada: El acceso a App Services debe estar restringido [versión preliminar])

Estas recomendaciones ya no aparecerán en la lista de Security Center de recomendaciones. Las directivas relacionadas ya no se incluirán en la iniciativa denominada "valor predeterminado de Security Center".

Más información sobre las [recomendaciones de seguridad](recommendations-reference.md).




## <a name="february-2020"></a>Febrero de 2020

### <a name="fileless-attack-detection-for-linux-preview"></a>Detección de ataques sin archivos para Linux (versión preliminar)

A medida que los atacantes aumentan el empleo de métodos stealthier para evitar la detección, Azure Security Center está ampliando la detección de ataques no archivados para Linux, además de Windows. Los ataques sin archivos no aprovechan las vulnerabilidades del software, insertan cargas malintencionadas en procesos benignos del sistema y se ocultan en la memoria. Estas técnicas:

- minimiza o eliminan seguimientos de malware en disco
- reducen considerablemente las posibilidades de detección mediante soluciones de análisis de malware basadas en disco

Para contrarrestar esta amenaza, Azure Security Center publicó la detección de ataques sin archivos para Windows en octubre de 2018 y ahora ha extendido la detección de ataques sin archivos en Linux. 



## <a name="january-2020"></a>Enero de 2020

### <a name="enhanced-secure-score-preview"></a>Puntuación de seguridad mejorada (versión preliminar)

Ahora hay disponible una versión mejorada de la característica de puntuación segura de Azure Security Center en versión preliminar. En esta versión, se agrupan varias recomendaciones en controles de seguridad que reflejan mejor las superficies de ataque vulnerables (por ejemplo, restringir el acceso a los puertos de administración).

Familiarícese con los cambios de puntuación segura durante la fase de versión preliminar y determine otras correcciones que le ayudarán a proteger más su entorno.

Obtenga más información sobe la [puntuación de seguridad mejorada (versión preliminar)](secure-score-security-controls.md).



## <a name="november-2019"></a>Noviembre de 2019

Las actualizaciones de noviembre incluyen:
 - [Protección contra amenazas para Azure Key Vault en regiones de Norteamérica (versión preliminar)](#threat-protection-for-azure-key-vault-in-north-america-regions-preview)
 - [La protección contra amenazas para Azure Storage incluye el filtrado de reputación de malware](#threat-protection-for-azure-storage-includes-malware-reputation-screening)
 - [Automatización del flujo de trabajo con Logic Apps (versión preliminar)](#workflow-automation-with-logic-apps-preview)
 - [Corrección rápida de los recursos en masa disponibles con carácter general](#quick-fix-for-bulk-resources-generally-available)
 - [Examen de imágenes de contenedor para detectar vulnerabilidades (versión preliminar)](#scan-container-images-for-vulnerabilities-preview)
 - [Estándares de cumplimiento normativo adicionales (versión preliminar)](#additional-regulatory-compliance-standards-preview)
 - [Protección contra amenazas para Azure Kubernetes Service (versión preliminar)](#threat-protection-for-azure-kubernetes-service-preview)
 - [Evaluación de vulnerabilidades de máquinas virtuales (versión preliminar)](#virtual-machine-vulnerability-assessment-preview)
 - [Advanced Data Security para servidores SQL Server en Azure Virtual Machines (versión preliminar)](#advanced-data-security-for-sql-servers-on-azure-virtual-machines-preview)
 - [Compatibilidad con directivas personalizadas (versión preliminar)](#support-for-custom-policies-preview)
 - [Ampliación de la cobertura de Azure Security Center con la plataforma para la comunidad y los asociados](#extending-azure-security-center-coverage-with-platform-for-community-and-partners)
 - [Integraciones avanzadas con la exportación de recomendaciones y alertas (versión preliminar)](#advanced-integrations-with-export-of-recommendations-and-alerts-preview)
 - [Incorporación de servidores locales a Security Center desde Windows Admin Center (versión preliminar)](#onboard-on-prem-servers-to-security-center-from-windows-admin-center-preview)

### <a name="threat-protection-for-azure-key-vault-in-north-america-regions-preview"></a>Protección contra amenazas para Azure Key Vault en regiones de Norteamérica (versión preliminar)

Azure Key Vault es un servicio esencial para proteger los datos y mejorar el rendimiento de las aplicaciones en la nube, ya que ofrece la posibilidad de administrar de forma centralizada claves, secretos, claves criptográficas y directivas en la nube. Como Azure Key Vault almacena datos confidenciales y críticos para la empresa, requiere la máxima seguridad para los almacenes de claves y los datos almacenados en ellos.

El soporte de Azure Security Center para Threat Protection para Azure Key Vault proporciona un nivel adicional de inteligencia de seguridad que detecta intentos poco habituales y potencialmente peligrosos de acceder a los almacén de claves o vulnerarlos. Esta nueva capa de protección permite a los clientes afrontar las amenazas contra sus almacenes sin necesidad de ser un experto en seguridad ni administrar sistemas de supervisión de la seguridad. La característica está en versión preliminar pública en regiones Norteamérica.


### <a name="threat-protection-for-azure-storage-includes-malware-reputation-screening"></a>La protección contra amenazas para Azure Storage incluye el filtrado de reputación de malware

La protección contra amenazas para Azure Storage ofrece nuevas detecciones basadas en la inteligencia sobre amenazas de Microsoft para detectar cargas de malware en Azure Storage mediante el análisis de reputación de hash y accesos sospechosos desde un nodo de salida de Tor activo (un proxy de anonimización). Ahora puede ver el malware detectado en todas las cuentas de almacenamiento mediante Azure Security Center.


### <a name="workflow-automation-with-logic-apps-preview"></a>Automatización del flujo de trabajo con Logic Apps (versión preliminar)

Organizaciones con seguridad administrada centralmente y de TI/operaciones que implementan procesos de flujo de trabajo internos para impulsar la acción necesaria dentro de la organización cuando se detectan discrepancias en sus entornos. En muchos casos, estos flujos de trabajo son procesos repetibles y la automatización puede optimizar considerablemente los procesos dentro de la organización.

Hoy estamos presentando una nueva funcionalidad en Security Center que permite a los clientes crear configuraciones de automatización que aprovechan Azure Logic Apps y crear directivas que las desencadenarán automáticamente en función de determinados resultados de ASC, como recomendaciones o alertas. La aplicación lógica de Azure se puede configurar para realizar cualquier acción personalizada compatible con la amplia comunidad de conectores de aplicaciones lógicas, o bien usar una de las plantillas proporcionadas por Security Center como enviar un correo electrónico o abrir un vale de ServiceNow™.

Para más información acerca de las capacidades de Security Center automática y manual para ejecutar los flujos de trabajo, consulte [ automatización de flujos de trabajo](workflow-automation.md).

Para información acerca de la creación de Logic Apps, consulte [Azure Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-overview).


### <a name="quick-fix-for-bulk-resources-generally-available"></a>Corrección rápida de los recursos en masa disponibles con carácter general

Con las muchas tareas que se proporcionan a un usuario como parte de la puntuación segura, la capacidad de corregir los problemas de forma eficaz en una flota de gran tamaño puede resultar complicada.

Para simplificar la corrección de las configuraciones erróneas de seguridad y poder corregir rápidamente las recomendaciones en una gran parte de los recursos y mejorar la puntuación segura, use la corrección rápida.

Esta operación le permitirá seleccionar los recursos a los que desea aplicar la corrección e iniciar una acción correctiva que configurará la configuración en su nombre.

La corrección rápida está disponible con carácter general para los clientes de hoy en día como parte de la página de recomendaciones de Security Center.

Vea qué recomendaciones tiene habilitada la corrección rápida en la [guía de referencia para recomendaciones de seguridad](recommendations-reference.md).


### <a name="scan-container-images-for-vulnerabilities-preview"></a>Examinar imágenes de contenedor para detectar vulnerabilidades (versión preliminar)

Ahora Azure Security Center puede examinar imágenes de contenedor en Azure Container Registry para detectar vulnerabilidades.

El análisis de imágenes funciona mediante el análisis del archivo de imagen del contenedor y la comprobación de la existencia de alguna vulnerabilidad conocida (con la tecnología Qualys).

El propio análisis se desencadena automáticamente al insertar nuevas imágenes de contenedor en Azure Container Registry. Las vulnerabilidades detectadas se verán como recomendaciones de Security Center y se incluirán en la puntuación segura de Azure junto con información sobre cómo aplicarles revisiones para reducir la superficie expuesta a los ataques que permitían.


### <a name="additional-regulatory-compliance-standards-preview"></a>Estándares de cumplimiento normativo adicionales (versión preliminar)

En el panel de cumplimiento normativo se ofrece información sobre la postura de cumplimiento basado en las evaluaciones de Security Center. En el panel se muestra cómo el entorno cumple los controles y requisitos designados por estándares normativos específicos y pruebas comparativas del sector, y proporciona recomendaciones prescriptivas para abordar estos requisitos.

Por lo tanto, el panel de cumplimiento normativo ha admitido cuatro estándares integrados:  Azure CIS 1.1.0, PCI-DSS, ISO 27001, y SOC-TSP. Ahora anunciamos la versión preliminar pública de estándares admitidos adicionales: NIST SP 800-53 R4, SWIFT CSP CSCF v2020, Canada Federal PBMM y UK Official junto con UK NHS. También lanzamos una versión actualizada de Azure CIS 1.1.0, que cubre más controles del estándar y la mejora de la extensibilidad.

[Obtenga más información sobre cómo personalizar el conjunto de estándares en el panel de cumplimiento normativo](update-regulatory-compliance-packages.md).


### <a name="threat-protection-for-azure-kubernetes-service-preview"></a>Protección contra amenazas para Azure Kubernetes Service (versión preliminar)

Kubernetes se convierte rápidamente en el nuevo estándar para implementar y administrar software en la nube. Pocas personas tienen una amplia experiencia con Kubernetes y muchos solo se centran en la ingeniería y administración generales y pasan por alto el aspecto de seguridad. El entorno de Kubernetes debe configurarse cuidadosamente para que sea seguro, asegurándose de que no se deja abierta ninguna puerta expuesta a ataques de contenedor para los atacantes. Security Center está expandiendo su compatibilidad en el espacio del contenedor a uno de los servicios en crecimiento más rápido de Azure: Azure Kubernetes Service (AKS).

Las nuevas capacidades de esta versión preliminar pública incluyen:

- **Detección y visibilidad**: detección continua de instancias de AKS administradas dentro de las suscripciones registradas de Security Center.
- **Recomendaciones de puntuación segura**: elementos procesables para ayudar a los clientes a cumplir los procedimientos recomendados de seguridad de AKS como parte de la puntuación segura del cliente, como "El control de acceso basado en roles debe usarse para restringir el acceso a un clúster de servicio de Kubernetes".
- **Detección de amenazas**: análisis basados en host y en clúster, como "Se ha detectado un contenedor con privilegios".


### <a name="virtual-machine-vulnerability-assessment-preview"></a>Evaluación de vulnerabilidades de máquinas virtuales (versión preliminar)

Las aplicaciones que se instalan en máquinas virtuales a menudo pueden tener vulnerabilidades que podrían provocar una infracción de la máquina virtual. Estamos anunciando que el nivel estándar de Security Center incluye una evaluación de vulnerabilidades integrada para máquinas virtuales sin costo adicional. La evaluación de vulnerabilidades, con la tecnología de Qualys en la versión preliminar pública, le permitirá examinar continuamente todas las aplicaciones instaladas en una máquina virtual para encontrar aplicaciones vulnerables y presentar las conclusiones en la experiencia del portal de Security Center. Security Center se encarga de todas las operaciones de implementación para que no sea necesario ningún trabajo adicional por parte del usuario. En adelante, tenemos previsto proporcionar opciones de evaluación de vulnerabilidades para satisfacer las necesidades empresariales únicas de nuestros clientes.

[Más información acerca de evaluaciones de vulnerabilidades para Azure Virtual Machines](deploy-vulnerability-assessment-vm.md).


### <a name="advanced-data-security-for-sql-servers-on-azure-virtual-machines-preview"></a>Advanced Data Security para servidores SQL Server en Azure Virtual Machines (versión preliminar)

La compatibilidad de Azure Security Center para la protección contra amenazas y la evaluación de vulnerabilidades en las bases de SQL que se ejecutan en máquinas virtuales IaaS está ahora en versión preliminar.

[Evaluación de vulnerabilidades](https://docs.microsoft.com/azure/sql-database/sql-vulnerability-assessment) es un servicio fácil de configurar que puede detectar, realizar un seguimiento y ayudarle a corregir posibles puntos vulnerables de una base de datos. Permite ver la postura de seguridad como parte de la puntuación segura de Azure e incluye los pasos para resolver problemas de seguridad y mejorar las defensas de su base de datos.

[Advanced Threat Protection](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview) detecta actividades anómalas que indican intentos inusuales y potencialmente perjudiciales de acceder a su servidor SQL Server o de aprovechar sus vulnerabilidades. Supervisa de forma constante la base de datos en busca de actividad sospechosa y proporciona alertas de seguridad sobre patrones de acceso a la base de datos anómalos para que pueda tomar medidas. Las alertas proporcionan detalles de la actividad sospechosa y recomiendan acciones para investigar y mitigar la amenaza.


### <a name="support-for-custom-policies-preview"></a>Compatibilidad con directivas personalizadas (versión preliminar)

Azure Security Center admite ahora directivas personalizadas (en versión preliminar).

Nuestros clientes han estado intentando ampliar su cobertura de evaluaciones de seguridad actual en Security Center con sus propias evaluaciones de seguridad basadas en las directivas que crean en Azure Policy. Gracias a la compatibilidad con las directivas personalizadas, ahora es posible.

Estas nuevas directivas formarán parte de la experiencia de recomendaciones de Security Center, la puntuación segura y el panel de normas de cumplimiento normativo. Con la compatibilidad con las directivas personalizadas, ahora puede crear una iniciativa personalizada en Azure Policy y, a continuación, agregarla como una directiva en Security Center y visualizarla como una recomendación.


### <a name="extending-azure-security-center-coverage-with-platform-for-community-and-partners"></a>Ampliación de la cobertura de Azure Security Center con la plataforma para la comunidad y los asociados

Use Security Center para recibir recomendaciones no solo de Microsoft, sino también de soluciones existentes de asociados como Check Point, Tenable y CyberArk con muchas más integraciones.  El flujo de incorporación simple de Security Center puede conectar las soluciones existentes a Security Center, lo que permite ver las recomendaciones de seguridad en un único lugar, ejecutar informes unificados y aprovechar todas las funcionalidades de Security Center con las recomendaciones integradas y de asociados. También puede exportar las recomendaciones de Security Center a productos de asociados.

[Más información sobre Intelligent Security Association.](https://www.microsoft.com/security/partnerships/intelligent-security-association)



### <a name="advanced-integrations-with-export-of-recommendations-and-alerts-preview"></a>Integraciones avanzadas con la exportación de recomendaciones y alertas (versión preliminar)

Con el fin de habilitar escenarios de nivel empresarial sobre Security Center, ahora es posible consumir alertas y recomendaciones de Security Center en otros lugares, excepto el Azure Portal o la API. Estos se pueden exportar directamente a un centro de eventos y áreas de trabajo de Log Analytics. Estos son algunos de los flujos de trabajo que puede crear en torno a estas nuevas capacidades:

- Con la exportación al área de trabajo de Log Analytics, puede crear paneles personalizados con Power BI.
- Con la exportación al centro de eventos, podrá exportar alertas y recomendaciones de Security Center a los SIEM de terceros, a una solución de terceros en tiempo real o a Azure Data Explorer.


### <a name="onboard-on-prem-servers-to-security-center-from-windows-admin-center-preview"></a>Incorporación de servidores locales a Security Center desde Windows Admin Center (versión preliminar)

Windows Admin Center es un portal de administración para los servidores de Windows que no están implementados en Azure y que ofrecen varias funcionalidades de administración de Azure, como copias de seguridad y actualizaciones del sistema. Recientemente hemos agregado una capacidad para incorporar estos servidores que no son de Azure para que ASC los proteja directamente de la experiencia de Windows Admin Center.

Con esta nueva experiencia, los usuarios van a incorporar un servidor de WAC a Azure Security Center y habilitar la visualización de las alertas y recomendaciones de seguridad directamente en la experiencia de Windows Admin Center.


## <a name="september-2019"></a>Septiembre de 2019

Las actualizaciones de septiembre incluyen:

 - [Administración de reglas con mejoras de controles de aplicación adaptables](#managing-rules-with-adaptive-application-controls-improvements)
 - [Control de la recomendación de seguridad del contenedor mediante Azure Policy](#control-container-security-recommendation-using-azure-policy)

### <a name="managing-rules-with-adaptive-application-controls-improvements"></a>Administrar reglas con mejoras de controles de aplicación adaptables

Se ha mejorado la experiencia de administración de reglas para máquinas virtuales mediante controles de aplicación adaptables. Los controles de aplicación adaptables de Azure Security Center ayudan a controlar qué aplicaciones se pueden ejecutar en las máquinas virtuales. Además de una mejora general en la administración de reglas, una nueva ventaja le permite controlar qué tipos de archivo se protegerán cuando agregue una nueva regla.

[Más información acerca de los controles de aplicación adaptables](security-center-adaptive-application.md).


### <a name="control-container-security-recommendation-using-azure-policy"></a>Controlar la recomendación de seguridad del contenedor mediante Azure Policy

La recomendación de Azure Security Center para corregir vulnerabilidades en la seguridad de los contenedores ahora puede habilitarse o deshabilitarse a través de Azure Policy.

Para ver las directivas de seguridad habilitadas, en Security Center abra la página Directiva de seguridad.


## <a name="august-2019"></a>Agosto de 2019

Las actualizaciones de agosto incluyen:

 - [Acceso a máquina virtual Just-in-Time (JIT) para Azure Firewall](#just-in-time-jit-vm-access-for-azure-firewall)
 - [Corrección de un solo clic para impulsar la postura de seguridad (versión preliminar)](#single-click-remediation-to-boost-your-security-posture-preview)
 - [Administración entre inquilinos](#cross-tenant-management)

### <a name="just-in-time-jit-vm-access-for-azure-firewall"></a>Acceso a máquina virtual Just-in-Time (JIT) para Azure Firewall 

El acceso a máquina virtual Just-in-Time (JIT) para Azure Firewall ya está disponible con carácter general. Úselo para proteger sus entornos protegidos con Azure Firewall además de los entornos protegidos con NSG.

El acceso de máquina virtual JIT reduce la exposición a los ataques volumétricos de red al proporcionar acceso controlado a las máquinas virtuales solo cuando sea necesario, mediante el uso de las reglas de NSG y Azure Firewall.

Al habilitar JIT para las máquinas virtuales, se crea una directiva que determina los puertos que se van a proteger, cuánto tiempo deben permanecer abiertos los puertos y las direcciones IP aprobadas desde donde se puede tener acceso a estos puertos. Esta directiva le ayuda a mantener el control de lo que los usuarios pueden hacer cuando solicitan acceso.

Las solicitudes se registran en el registro de actividad de Azure para que pueda supervisar y auditar fácilmente el acceso. La página Just-in-Time también le ayuda a identificar rápidamente las máquinas virtuales existentes que tienen habilitado JIT y las máquinas virtuales en las que se recomienda JIT.

[Más información acerca de Azure Firewall](https://docs.microsoft.com/azure/firewall/overview).


### <a name="single-click-remediation-to-boost-your-security-posture-preview"></a>Corrección de un solo clic para impulsar la postura de seguridad (versión preliminar)

La puntuación segura es una herramienta que le ayuda a evaluar la postura de seguridad de las cargas de trabajo. Revisa las recomendaciones de seguridad y les asigna una prioridad, para que usted sepa qué recomendaciones realizar primero. De esta manera, puede encontrar las vulnerabilidades de seguridad más graves para dar prioridad a la investigación.

Con el fin de simplificar la corrección de configuraciones incompletas de seguridad y ayudarle a mejorar rápidamente su puntuación segura, hemos agregado una nueva funcionalidad que le permite corregir una recomendación en una gran parte de los recursos con un solo clic.

Esta operación le permitirá seleccionar los recursos a los que desea aplicar la corrección e iniciar una acción correctiva que configurará la configuración en su nombre.

Vea qué recomendaciones tiene habilitada la corrección rápida en la [guía de referencia para recomendaciones de seguridad](recommendations-reference.md).


### <a name="cross-tenant-management"></a>Administración entre inquilinos

Security Center admite ahora escenarios de administración entre inquilinos como parte de Azure Lighthouse. Esto le permite obtener visibilidad y administrar la postura de seguridad de varios inquilinos en Security Center. 

[Más información sobre las experiencias de administración entre inquilinos](security-center-cross-tenant-management.md).


## <a name="july-2019"></a>Julio de 2019

### <a name="updates-to-network-recommendations"></a>Actualizaciones de recomendaciones sobre redes

Azure Security Center (ASC) ha lanzado nuevas recomendaciones de redes y ha mejorado algunas existentes. Ahora, el uso de Security Center garantiza una mayor protección de la red para los recursos. 

[Más información sobre las recomendaciones de redes](recommendations-reference.md#recs-network).


## <a name="june-2019"></a>Junio de 2019

### <a name="adaptive-network-hardening---generally-available"></a>Protección de redes adaptable: disponible con carácter general

Una de las superficies de ataque más importantes para las cargas de trabajo que se ejecutan en la nube pública son las conexiones hacia y desde la red pública de Internet. Nuestros clientes saben que es difícil saber qué reglas del grupo de seguridad de red (NSG) deben estar en vigor para asegurarse de que las cargas de trabajo de Azure solo están disponibles para los intervalos de origen necesarios. Con esta característica, Security Center aprende los patrones de conectividad y el tráfico de red de las cargas de trabajo de Azure y proporciona recomendaciones de reglas de NSG, para las máquinas virtuales orientadas a Internet. Esto ayuda a nuestro cliente a configurar mejor las directivas de acceso a la red y limitar su exposición a los ataques. 

[Más información sobre la protección de red adaptable](security-center-adaptive-network-hardening.md).
