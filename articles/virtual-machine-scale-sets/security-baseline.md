---
title: Línea de base de seguridad de Azure para Virtual Machine Scale Sets
description: La línea de base de seguridad de Virtual Machine Scale Sets proporciona instrucciones de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: virtual-machine-scale-sets
ms.topic: conceptual
ms.date: 07/22/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: d55e53ba455a4b91b4f57ea08b250320a5467c2b
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89398451"
---
# <a name="azure-security-baseline-for-virtual-machine-scale-sets"></a>Línea de base de seguridad de Azure para Virtual Machine Scale Sets

La línea de base de seguridad de Azure para Virtual Machine Scale Sets contiene recomendaciones que le ayudarán a mejorar la posición de seguridad de su implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](https://docs.microsoft.com/azure/security/benchmarks/overview), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para obtener más información, consulte [Introducción a las líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).

## <a name="network-security"></a>Seguridad de las redes

*Para más información, consulte [Control de seguridad: Seguridad de redes](/azure/security/benchmarks/security-control-network-security).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Guía**: cuando se crea una máquina virtual (VM) de Azure, es preciso crear una red virtual o usar una existente y configurar la máquina virtual con una subred. Asegúrese de que todas las subredes implementadas tienen un grupo de seguridad de red aplicado con controles de acceso a la red específicos para los puertos y orígenes de confianza de sus aplicaciones.

Por otro lado, si tiene un caso de uso específico para un firewall centralizado, Azure Firewall también puede usarse para cumplir esos requisitos.

* [Redes para conjuntos de escalado de máquinas virtuales de Azure](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-networking)

* [Creación de una red virtual](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

* [Implementación y configuración de Azure Firewall](https://docs.microsoft.com/azure/firewall/tutorial-firewall-deploy-portal)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes e interfaces de red

**Instrucciones**: use Azure Security Center para identificar y seguir las recomendaciones de protección de redes para ayudarle a proteger sus recursos de máquinas virtuales de Azure en Azure. Habilite los registros de flujo de NSG y envíe registros a una cuenta de almacenamiento para la auditoría del tráfico de las máquinas virtuales por la actividad inusual.

* [Habilitación de los registros de flujo de NSG](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal)

* [Descripción de la seguridad de red proporcionada por Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-network-recommendations)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Guía**: si usa el conjunto de escalado de máquinas virtuales (VMSS) para hospedar aplicaciones web, use un grupo de seguridad de red (NSG) en la subred del conjunto de escalado de máquinas virtuales para limitar el tráfico de red, los puertos y los protocolos que pueden comunicarse. Siga un enfoque de red con privilegios mínimos al configurar el grupo de seguridad de red para permitir solo el tráfico necesario a la aplicación.

También puede implementar el firewall de aplicaciones web (WAF) de Azure delante de las aplicaciones web críticas para la inspección adicional del tráfico entrante. Habilite la configuración de diagnóstico para WAF e ingiera los registros en una cuenta de almacenamiento, un centro de eventos o un área de trabajo de análisis de registros.

* [Redes para conjuntos de escalado de máquinas virtuales de Azure](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-networking)

* [Creación de una puerta de enlace de aplicaciones con un firewall de aplicaciones web mediante Azure Portal](https://docs.microsoft.com/azure/web-application-firewall/ag/application-gateway-web-application-firewall-portal)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Denegación de las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: habilite la protección contra denegación de servicio distribuida (DDoS) estándar en las redes virtuales para protegerse frente a ataques DDoS. Con la inteligencia sobre amenazas integrada de Azure Security Center puede supervisar las comunicaciones con direcciones IP malintencionadas conocidas. Configure Azure Firewall en cada uno de los segmentos de la red virtual, con la inteligencia sobre amenazas habilitada y configurada para "Alertar y denegar" el tráfico de red malintencionado.

Puede usar el acceso de red Just-in-Time de Azure Security Center para limitar la exposición de Windows Virtual Machines a las direcciones IP aprobadas durante un período limitado. Use también la protección de redes adaptativa de Azure Security Center para recomendar configuraciones de NSG que limiten los puertos y las direcciones IP basadas en el tráfico real y la inteligencia sobre amenazas.

* [Configuración de la protección contra DDoS](https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection)

* [Implementación de Azure Firewall](https://docs.microsoft.com/azure/firewall/tutorial-firewall-deploy-portal)

* [Descripción de la inteligencia sobre amenazas integrada de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer)

* [Descripción de la protección de red adaptable de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-adaptive-network-hardening)

* [Descripción del control de acceso de red Just-in-Time de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-just-in-time)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Guía**: puede anotar los registros de flujo del grupo de seguridad de red en una cuenta de almacenamiento para generar los registros de flujo de las máquinas virtuales de Azure. Al investigar actividades anómalas, puede habilitar la captura de paquetes de Network Watcher para que el tráfico de red pueda revisarse en busca de actividad inusual e inesperada.

* [Habilitación de los registros de flujo de NSG](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal)

* [Habilitación de Network Watcher](https://docs.microsoft.com/azure/network-watcher/network-watcher-create)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implementación de sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Guía**: al combinar las capturas de paquetes proporcionadas por Network Watcher con una herramientas IDS de código abierto, puede realizar la detección de intrusiones para una amplia variedad de amenazas. Además, puede implementar Azure Firewall en segmentos de la red virtual, según corresponda, con la inteligencia sobre amenazas habilitada y configurada para "Alertar y denegar" el tráfico de red malintencionado.

* [Realización de la detección de intrusiones en la red con Network Watcher y herramientas de código abierto](https://docs.microsoft.com/azure/network-watcher/network-watcher-intrusion-detection-open-source-tools)

* [Implementación de Azure Firewall](https://docs.microsoft.com/azure/firewall/tutorial-firewall-deploy-portal)

* [Configuración de alertas con Azure Firewall](https://docs.microsoft.com/azure/firewall/threat-intel)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Guía**: si usa el conjunto de escalado de máquinas virtuales (VMSS) para hospedar aplicaciones web, puede implementar Azure Application Gateway para las aplicaciones web con HTTPS/SSL habilitado para los certificados de confianza. Con Azure Application Gateway, puede dirigir el tráfico web de la aplicación a recursos específicos mediante la asignación de agentes de escucha a los puertos, la creación de reglas y la adición de recursos a un grupo de back-end como VMSS, entre otros.

* [Implementación de Application Gateway](https://docs.microsoft.com/azure/application-gateway/quick-create-portal)

* [Configuración de Application Gateway para usar HTTPS](https://docs.microsoft.com/azure/application-gateway/create-ssl-portal)

* [Creación de un conjunto de escalado que hace referencia a una instancia de Application Gateway](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-networking#create-a-scale-set-that-references-an-application-gateway)

* [Descripción del equilibrio de carga de capa 7 con las puertas de enlace de aplicaciones web de Azure](https://docs.microsoft.com/azure/application-gateway/overview)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: use etiquetas de servicio de Virtual Network para definir controles de acceso a la red en los grupos de seguridad de red o las instancias de Azure Firewall configuradas para las máquinas virtuales de Azure. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, ApiManagement) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

* [Descripción y uso de etiquetas de servicio](https://docs.microsoft.com/azure/virtual-network/service-tags-overview)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: defina e implemente configuraciones de seguridad estándar para Azure Virtual Machine Scale Sets con Azure Policy. También puede usar Azure Blueprints para simplificar las implementaciones de VM de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como plantillas de Azure Resource Manager, asignaciones de roles y asignaciones de Azure Policy, en una única definición de un plano técnico. Puede aplicar el plano técnico a las suscripciones y habilitar la administración de recursos a través del control de versiones de planos técnicos.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Más información sobre las plantillas de conjuntos de escalado de máquinas virtuales](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-mvss-start)

* [Ejemplos de Azure Policy para redes](https://docs.microsoft.com/azure/governance/policy/samples/#network)

* [Creación de un plano técnico de Azure](https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: puede usar etiquetas para grupos de seguridad de red (NSG) y otros recursos relacionados con la seguridad de red y el flujo de tráfico que están configurados para las máquinas virtuales Windows. En el caso de las reglas de NSG individuales, use el campo "Descripción" para especificar las necesidades empresariales o la duración de las reglas que permiten que entre o salga el tráfico en una red.

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

* [Creación de una red virtual](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: use el registro de actividad de Azure para supervisar los cambios a las configuraciones de recursos de red relacionados con su conjunto de escalado de máquinas virtuales de Azure. Cree alertas en Azure Monitor que se desencadenen cuando se produzcan cambios en la configuraciones o recursos de red críticos.

Use Azure Policy para validar o corregir las configuraciones de los recursos de red relacionados con el conjunto de escalado de máquinas virtuales.

* [Visualización y recuperación de eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view)

* [Creación de alertas en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log)

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Ejemplos de Azure Policy para redes](https://docs.microsoft.com/azure/governance/policy/samples/#network)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Control de seguridad: registro y supervisión](/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Instrucciones**: Microsoft mantiene los orígenes de hora de los recursos de Azure; sin embargo, tiene la opción de administrar la configuración de la sincronización de la hora de las máquinas virtuales.

* [Configuración de la sincronización de hora de los recursos de proceso de Windows de Azure](https://docs.microsoft.com/azure/virtual-machines/windows/time-sync)

* [Configuración de la sincronización de hora de los recursos de proceso de Linux de Azure](https://docs.microsoft.com/azure/virtual-machines/linux/time-sync)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Instrucciones**: los registros de actividad se pueden usar para auditar las operaciones y las acciones realizadas en los recursos del conjunto de escalado de máquinas virtuales. El registro de actividad contiene todas las operaciones de escritura (PUT, POST, DELETE) para los recursos, excepto las de lectura (GET). Los registros de actividad se pueden usar para encontrar errores al solucionar problemas o para supervisar cómo un usuario de su organización modificó un recurso.

Puede habilitar los datos de registro generados a partir de los recursos de máquina virtual o los registros de actividad de Azure e incorporarlos a Azure Sentinel o a un SIEM de terceros para la administración central de los registros de seguridad.

use Azure Security Center para ofrecer supervisión del registro de eventos de seguridad para Azure Virtual Machines. Dado el volumen de datos que genera el registro de eventos de seguridad, no se almacena de forma predeterminada.

Si su organización quiere conservar los datos del registro de eventos de seguridad de la máquina virtual, puede almacenarse en un área de trabajo de Log Analytics en el nivel de recopilación de datos deseado que configuró en Azure Security Center.

* [Recopilación de registros y métricas de plataforma con Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings)

* [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

* [Introducción a Azure Monitor e integración con herramienta SIEM de terceros](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools)

* [Recopilación de datos en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-enable-data-collection#data-collection-tier)

* [Supervisión de máquinas virtuales en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/monitor)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: los registros de actividad se pueden usar para auditar las operaciones y las acciones realizadas en los recursos del conjunto de escalado de máquinas virtuales. El registro de actividad contiene todas las operaciones de escritura (PUT, POST, DELETE) para los recursos, excepto las de lectura (GET). Los registros de actividad se pueden usar para encontrar errores al solucionar problemas o para supervisar cómo un usuario de su organización modificó un recurso.

Habilite la recopilación de datos de diagnóstico del sistema operativo invitado al implementar la extensión de diagnóstico en sus máquinas virtuales (VM). Puede usar la extensión de diagnósticos para recopilar datos de diagnóstico como registros de aplicaciones o contadores de rendimiento de una máquina virtual de Azure.

Para obtener una visibilidad avanzada de las aplicaciones y los servicios que son compatibles con el conjunto de escalado de máquinas virtuales, puede habilitar Azure Monitor para VM y Application Insights. Con Application Insights, puede supervisar la aplicación y capturar datos de telemetría, tales como solicitudes HTTP o excepciones, para que pueda correlacionar los problemas entre las máquinas virtuales y la aplicación.

* [Recopilación de registros y métricas de plataforma con Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings)

* [Visualización y recuperación de eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view)

* [Supervisión de máquinas virtuales en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/monitor)

* [Introducción a Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Guía**: use Azure Security Center para ofrecer supervisión del registro de eventos de seguridad para Azure Virtual Machines. Dado el volumen de datos que genera el registro de eventos de seguridad, no se almacena de forma predeterminada.

Si su organización quiere conservar los datos del registro de eventos de seguridad de la máquina virtual, puede almacenarse en un área de trabajo de Log Analytics en el nivel de recopilación de datos deseado que configuró en Azure Security Center.

* [Recopilación de datos en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-enable-data-collection#data-collection-tier)

* [Supervisión de máquinas virtuales en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/monitor)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: asegúrese de que todas las cuentas de almacenamiento o áreas de trabajo de Log Analytics usadas para almacenar registros de máquina virtual tengan el período de retención de registros configurado de acuerdo con la normativa de cumplimiento de la organización.

* [Supervisión de máquinas virtuales en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/monitor)

* [Configuración del período de retención del área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Guía**: Analice y supervise los registros en busca de comportamientos anómalos y revise los resultados con regularidad. Use Azure Monitor para revisar los registros y realizar consultas en los datos de registro.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros para supervisar y revisar los registros.

* [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

* [Descripción del área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal)

* [Procedimiento para realizar consultas personalizadas en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: use Azure Security Center configurado con un área de trabajo de Log Analytics para la supervisión y alertas sobre las actividades anómalas que se encuentran en los registros y eventos de seguridad para su instancia de Azure Virtual Machines.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros para configurar alertas sobre la actividad anómala.

* [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

* [Administración de alertas de seguridad en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts)

* [Alertas sobre datos de registro de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-response)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: puede usar Microsoft Antimalware para Azure Cloud Services y Virtual Machines y configurar las máquinas virtuales Windows para registrar eventos en una cuenta de Azure Storage. Configure un área de trabajo de Log Analytics para ingerir los eventos de las cuentas de almacenamiento y crear alertas cuando corresponda. Siga las recomendaciones en Azure Security Center: "Proceso y aplicaciones". Para máquinas virtuales Linux, necesitará una herramienta de terceros para la detección de vulnerabilidades antimalware.

* [Configuración de Microsoft Antimalware para Cloud Services y Virtual Machines](https://docs.microsoft.com/azure/security/fundamentals/antimalware)

* [Habilitación de la supervisión de nivel de invitado para Virtual Machines](https://docs.microsoft.com/azure/cost-management/azure-vm-extended-metrics)

* [Instrucciones para incorporar servidores Linux a Azure Security Center](https://docs.microsoft.com/azure/security-center/quick-onboard-linux-computer)

* [En el siguiente vínculo se proporcionan las directrices de seguridad recomendadas de Microsoft, que pueden servir como lista de criterios para el software de vulnerabilidad seleccionado](https://docs.microsoft.com/azure/virtual-machines/linux/security-recommendations)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: Implemente una solución de terceros de Azure Marketplace para la solución de registro DNS según las necesidades de su organización.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Guía**: Azure Security Center proporciona supervisión del registro de eventos de seguridad para Azure Virtual Machines (VM). Security Center aprovisiona Microsoft Monitoring Agent en todas las máquinas virtuales de Azure compatibles y en las que se crean si el aprovisionamiento automático está habilitado O puede instalar el agente manualmente. El agente habilita el evento 4688 de creación de procesos y el campo CommandLine dentro del evento 4688. El registro de eventos registra los nuevos procesos creados en la VM y los servicios de detección de Security Center los supervisan.

Para las máquinas virtuales Linux, puede configurar manualmente el registro de la consola en cada nodo y usar Syslog para almacenar los datos. Además, use el área de trabajo de Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos de Syslog de las máquinas virtuales de Azure.

* [Recopilación de datos en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-enable-data-collection#data-collection-tier)

* [Procedimiento para realizar consultas personalizadas en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries)

* [Orígenes de datos de Syslog en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/data-sources-syslog)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para más información, consulte [Control de seguridad: Identidad y control de acceso](/azure/security/benchmarks/security-control-identity-access-control).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: aunque Azure Active Directory es el método recomendado para administrar el acceso de los usuarios, Azure Virtual Machines puede disponer de cuentas locales. Las cuentas locales y de dominio deben revisarse y administrarse, normalmente con una superficie de memoria mínima. Además, aproveche Azure Privileged Identity Management para las cuentas administrativas usadas para tener acceso a los recursos de las máquinas virtuales.

* [Hay información disponible sobre las cuentas locales aquí](https://docs.microsoft.com/azure/active-directory/devices/assign-local-admin#manage-the-device-administrator-role)

* [Información sobre Privileged Identity Manager](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: El conjunto de escalado de máquinas virtuales de Azure y Azure Active Directory no tienen el concepto de contraseñas predeterminadas. El cliente es responsable de las aplicaciones de terceros y de los servicios de Marketplace que pueden usar contraseñas predeterminadas.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas que tengan acceso a las máquinas virtuales. Use la administración de identidades y acceso de Azure Security Center para supervisar el número de cuentas administrativas. Las cuentas de administrador que se usan para acceder a los recursos de máquinas virtuales de Azure también se pueden administrar a través de Azure Privileged Identity Management (PIM). Azure Privileged Identity Management proporciona varias opciones, como la elevación Just-in-Time, la aplicación de Multi-Factor Authentication antes de asumir un rol y las opciones de delegación para que los permisos solo estén disponibles durante intervalos de tiempo específicos y requieran un aprobador.

* [Descripción de identidad y acceso en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

* [Información sobre Privileged Identity Manager](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4: Uso del inicio de sesión único (SSO) de Azure Active Directory

**Guía**: Siempre que sea posible, use el SSO con Azure Active Directory en lugar de configurar credenciales independientes individuales por servicio. Use las recomendaciones de administración de identidades y acceso de Azure Security Center.

* [Inicio de sesión único en aplicaciones en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

* [Supervisión de la identidad y el acceso en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Uso de la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Instrucciones**: Habilite la MFA de Azure AD y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

* [Procedimiento para habilitar la MFA en Azure](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted)

* [Supervisión de la identidad y el acceso en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3.6: Uso de estaciones de trabajo seguras y administradas por Azure para realizar tareas administrativas

**Guía**: Utilice PAW (estaciones de trabajo de acceso privilegiado) con la MFA configurada para iniciar la sesión y configurar recursos de Azure.

* [Más información sobre las estaciones de trabajo con privilegios de acceso](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Procedimiento para habilitar la MFA en Azure](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: use Azure AD Privileged Identity Management (PIM) para la generación de registros y alertas cuando se produzca actividad sospechosa o insegura en el entorno. Use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo. De manera opcional, puede ingerir las alertas de detección de riesgo de Azure Security Center a Azure Monitor y configurar alertas o notificaciones personalizadas con grupos de acciones.

* [Cómo implementar Privileged Identity Management (PIM)](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan)

* [Descripción de las detecciones de riesgo de Azure Security Center (actividad sospechosa)](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events)

* [Integración de los registros de actividad de Azure en Azure Monitor](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

* [Configuración de grupos de acciones para alertas y notificaciones personalizadas](https://docs.microsoft.com/azure/azure-monitor/platform/action-groups)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: use las directivas y ubicaciones con nombre del acceso condicional de Azure Active Directory para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP, o países o regiones concretos.

* [Configuración de ubicaciones con nombre en Azure](https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Instrucciones**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios. Puede usar las identidades administradas para autenticarse en cualquier servicio que admita la autenticación de Azure AD, como Key Vault, sin necesidad de credenciales en el código. El código que se ejecuta en una máquina virtual puede usar la identidad administrada para solicitar tokens de acceso para los servicios que admiten la autenticación de Azure AD.

* [Procedimiento para crear y configurar una instancia de Azure AD](https://docs.microsoft.com/azure/active-directory-domain-services/tutorial-create-instance)

* [Información general sobre las identidades administradas de recursos de Azure](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure AD proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure Active Directory para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado. Cuando use máquinas virtuales de Azure, tendrá que revisar los grupos de seguridad locales y los usuarios para asegurarse de que no hay cuentas inesperadas que puedan poner en peligro el sistema.

* [Procedimiento para usar las revisiones de acceso de identidad de Azure](https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: configure opciones de diagnóstico para que Azure Active Directory envíe registros de auditoría y de inicio de sesión a un área de trabajo de Log Analytics. Además, use Azure Monitor para revisar los registros y realizar consultas en los datos de registro de las máquinas virtuales de Azure.

* [Descripción del área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal)

* [Integración de los registros de actividad de Azure en Azure Monitor](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

* [Procedimiento para realizar consultas personalizadas en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries)

* [Supervisión de máquinas virtuales en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/monitor)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12: Alerta de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: use las características de protección de identidad y detección de riesgo de Azure Active Directory para configurar respuestas automatizadas a las acciones sospechosas que se detecten en relación con los recursos de las cuentas de almacenamiento. Debe habilitar las respuestas automatizadas a través de Azure Sentinel para implementar las respuestas de seguridad de su organización.

* [Visualización de los inicios de sesión de riesgo de Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins)

* [Configuración y habilitación de las directivas de riesgo de protección de identidad](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies)

* [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: en escenarios de soporte técnico en los que Microsoft necesita acceder a los datos del cliente (por ejemplo, durante una solicitud de soporte técnico), use la Caja de seguridad del cliente para Azure Virtual Machines a fin de revisar y aprobar o rechazar las solicitudes de acceso a los datos del cliente.

* [Descripción de la Caja de seguridad del cliente](https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="data-protection"></a>Protección de los datos

*Para más información, consulte [Control de seguridad: Protección de datos](/azure/security/benchmarks/security-control-data-protection).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: use etiquetas para ayudar a realizar el seguimiento de las máquinas virtuales de Azure que almacenan o procesan información confidencial.

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Los recursos deben separarse por red virtual o subred, etiquetarse adecuadamente y protegerse en un grupo de seguridad de red (NSG) o una instancia de Azure Firewall. Para las máquinas virtuales que almacenan o procesan datos confidenciales, implemente la directiva y los procedimientos necesarios para desactivarlos cuando no estén en uso.

* [Creación de suscripciones adicionales de Azure](https://docs.microsoft.com/azure/billing/billing-create-subscription)

* [Creación de grupos de administración](https://docs.microsoft.com/azure/governance/management-groups/create)

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

* [Creación de una red virtual](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

* [Implementación de Azure Firewall](https://docs.microsoft.com/azure/firewall/tutorial-firewall-deploy-portal)

* [Configuración de alertas o alertas y denegación con Azure Firewall](https://docs.microsoft.com/azure/firewall/threat-intel)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: implemente una solución de terceros en los perímetros de red que supervisan la transferencia no autorizada de información confidencial y bloquean dichas transferencias mientras alerta a los profesionales de seguridad de la información.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

* [Descripción de la protección de datos de los clientes en Azure](https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: los datos en tránsito hacia, desde y entre máquinas virtuales (VM) que ejecutan Windows se cifran de varias maneras en función de la naturaleza de la conexión, como cuando se conecta a una máquina virtual en una sesión de RDP o SSH.

Microsoft usa el protocolo Seguridad de la capa de transporte (TLS) para proteger los datos cuando se transmiten entre los servicios en la nube y los clientes.

* [Cifrado en tránsito en VM](https://docs.microsoft.com/azure/security/fundamentals/encryption-overview#in-transit-encryption-in-vms)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Instrucciones**: Use una herramienta de detección activa de terceros para identificar toda la información confidencial almacenada, procesada o transmitida por los sistemas tecnológicos de la organización, incluidos los del ubicación local o los que se encuentran en un proveedor de servicios remoto, y actualizar el inventario de información confidencial de la organización.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos

**Instrucciones**: Mediante el control de acceso basado en roles de Azure (RBAC de Azure), puede repartir las tareas entre el equipo y conceder a los usuarios únicamente el nivel de acceso que necesitan en la máquina virtual (VM) para realizar su trabajo. En lugar de proporcionar a todos los empleados permisos no restringidos en la máquina virtual, puede permitir solo determinadas acciones. Puede configurar el control de acceso para la máquina virtual en Azure Portal, mediante la CLI de Azure o Azure PowerShell.

* [Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview)

* [Roles integrados en los recursos de Azure](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#virtual-machine-contributor)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: implemente una herramienta de terceros, como una solución de prevención de pérdida de datos basada en host automatizada, para aplicar controles de acceso que mitiguen el riesgo de las infracciones de datos.

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: los discos virtuales de Virtual Machines (VM) se cifran en reposo mediante el cifrado del lado servidor o el cifrado de discos de Azure (ADE). Azure Disk Encryption aprovecha la característica DM-Crypt de Linux para cifrar los discos administrados con claves administradas por el cliente dentro de la máquina virtual invitada. El cifrado del lado servidor con claves administradas por el cliente mejora en ADE al permitir el uso de cualquier tipo de sistema operativo y de imágenes para las máquinas virtuales mediante el cifrado de datos en el servicio Storage.

* [Azure Disk Encryption para Virtual Machine Scale Sets](https://docs.microsoft.com/azure/virtual-machine-scale-sets/disk-encryption-overview)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: use Azure Monitor con el registro de actividad de Azure para crear alertas para cuando se produzcan cambios en los conjuntos de escalado de máquinas virtuales y los recursos relacionados.

* [Creación de alertas para los eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log)

* [Registro de Azure Storage Analytics](https://docs.microsoft.com/azure/storage/common/storage-analytics-logging)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Control de seguridad: Administración de vulnerabilidades](/azure/security/benchmarks/security-control-vulnerability-management).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: siga las recomendaciones de Azure Security Center sobre cómo realizar evaluaciones de vulnerabilidades en Azure Virtual Machines. Use las recomendaciones de Azure Security o una solución de terceros para realizar evaluaciones de vulnerabilidades para las máquinas virtuales.

* [Implementación de las recomendaciones de evaluación de vulnerabilidades de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-vulnerability-assessment-recommendations)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Guía**: habilite las actualizaciones automáticas del sistema operativo para las versiones de sistemas operativos compatibles o para las imágenes personalizadas almacenadas en Shared Image Gallery.

* [Actualizaciones automáticas del sistema operativo en conjuntos de escalado de máquinas virtuales de Azure](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3: Implementación de una solución de administración de revisiones automatizada de títulos de software de terceros

**Instrucciones**: Azure Virtual Machine Scale Sets (VMSS) puede usar las actualizaciones automáticas de las imágenes del sistema operativo. Puede usar la extensión de Desired State Configuration (DSC) de Azure para las máquinas virtuales subyacentes en VMSS. DSC se utiliza para configurar las máquinas virtuales a medida que se conectan, ya que ejecutan el software deseado.

* [Uso de Virtual Machine Scale Sets con la extensión de DSC de Azure](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-dsc)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Guía**: Exporte los resultados de análisis en intervalos coherentes y compare los resultados para comprobar que se han corregido las vulnerabilidades. Al usar una recomendación de administración de vulnerabilidades sugerida por Azure Security Center, el cliente puede ir al portal de la solución seleccionada para ver los datos de análisis históricos.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Instrucciones**: Use la clasificación de riesgo predeterminada (puntuación de seguridad) proporcionada por Azure Security Center.

* [Descripción de la puntuación de seguridad de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-secure-score)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para más información, consulte [Control de seguridad: Administración de recursos y del inventario](/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Instrucciones**: use Azure Resource Graph para consultar y detectar todos los recursos (incluidas las máquinas virtuales) de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

* [Creación de consultas con Azure Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)

* [Visualización de las suscripciones de Azure](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

* [Descripción de Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica según una taxonomía.

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de Virtual Machines Scale Sets y los recursos relacionados. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

* [Creación de suscripciones adicionales de Azure](https://docs.microsoft.com/azure/billing/billing-create-subscription)

* [Creación de grupos de administración](https://docs.microsoft.com/azure/governance/management-groups/create)

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: Cree un inventario de los recursos de Azure aprobados y el software aprobado para los recursos de proceso según las necesidades de la organización.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Instrucciones**: use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Además, use Azure Resource Graph para consultar o detectar recursos dentro de sus suscripciones. Esto puede ayudar en entornos de alta seguridad, como aquellos con cuentas de Storage.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Creación de consultas con Azure Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Guía**: Azure Automation proporciona un control completo durante la implementación, las operaciones y la retirada de las cargas de trabajo y recursos. Use el inventario de máquinas virtuales de Azure para automatizar la recopilación de información sobre todo el software en Virtual Machines. Nota: El nombre del software, la versión, el publicador y la hora de actualización están disponibles en Azure Portal. Para obtener acceso a la fecha de instalación y otra información, se requiere que el cliente habilite los diagnósticos de nivel de invitado y transfiera los registros de eventos de Windows a un área de trabajo de Log Analytics.

Actualmente, los controles de aplicaciones adaptables no están disponibles para Virtual Machine Scale Sets.

* [Introducción a Azure Automation](https://docs.microsoft.com/azure/automation/automation-intro)

* [Habilitación del inventario de máquinas virtuales de Azure](https://docs.microsoft.com/azure/automation/automation-tutorial-installed-software)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: Azure Automation proporciona un control completo durante la implementación, las operaciones y la retirada de las cargas de trabajo y recursos. Puede usar Change Tracking para identificar todo el software instalado en Virtual Machines. Puede implementar su propio proceso o usar State Configuration de Azure Automation para quitar software no autorizado.

* [Introducción a Azure Automation](https://docs.microsoft.com/azure/automation/automation-intro)

* [Seguimiento de cambios en el entorno con la solución Change Tracking](https://docs.microsoft.com/azure/automation/change-tracking)

* [Introducción a State Configuration de Azure Automation](https://docs.microsoft.com/azure/automation/automation-dsc-overview)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Guía**: Actualmente, los controles de aplicaciones adaptables no están disponibles para Virtual Machine Scale Sets. Use software de terceros para controlar el uso de las aplicaciones aprobadas solamente.

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Instrucciones**: use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Denegación de un tipo de recurso específico con Azure Policy](https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10: Mantenimiento de un inventario de títulos de software aprobados

**Guía**: Actualmente, los controles de aplicaciones adaptables no están disponibles para Virtual Machine Scale Sets. Implemente una solución de terceros si esto no cumple los requisitos de la organización.

* [Uso de los controles de aplicaciones adaptables de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-adaptive-application)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager mediante la configuración de la opción "Bloquear acceso" en la aplicación "Administración de Microsoft Azure".

* [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso

**Guía**: Según el tipo de scripts, puede usar configuraciones específicas del sistema operativo o recursos de terceros para limitar la capacidad de los usuarios de ejecutar scripts en los recursos de proceso de Azure.

* [Control de la ejecución de scripts de PowerShell en entornos Windows](https://docs.microsoft.com/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-6)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Instrucciones**: Las aplicaciones de alto riesgo implementadas en el entorno de Azure se pueden aislar mediante una red virtual, subred, suscripciones, grupos de administración, etc. y proteger suficientemente con una instancia de Azure Firewall, un firewall de aplicaciones web (WAF) o un grupo de seguridad de red (NSG).

* [Redes virtuales y máquinas virtuales en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/network-overview)

* [Información general de Azure Firewall](https://docs.microsoft.com/azure/firewall/overview)

* [Introducción al firewall de aplicaciones web](https://docs.microsoft.com/azure/web-application-firewall/overview)

* [Introducción a la seguridad de red](https://docs.microsoft.com/azure/virtual-network/security-overview)

* [Introducción a Azure Virtual Network](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)

* [Organización de los recursos con grupos de administración de Azure](https://docs.microsoft.com/azure/governance/management-groups/overview)

* [Guía de decisiones de suscripción](https://docs.microsoft.com/azure/cloud-adoption-framework/decision-guides/subscriptions/)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Control de seguridad: Configuración segura](/azure/security/benchmarks/security-control-secure-configuration).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: use Azure Policy o Azure Security Center para mantener las configuraciones de seguridad para todos los recursos de Azure. Además, Azure Resource Manager tiene la capacidad de exportar la plantilla en notación de objetos JavaScript (JSON), que se debe revisar para asegurarse de que las configuraciones cumplan o superen los requisitos de seguridad de la empresa.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Información sobre cómo descargar la plantilla de máquina virtual](https://docs.microsoft.com/azure/virtual-machines/windows/download-template)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Instrucciones**: use la recomendación de Azure Security Center [Remediate Vulnerabilities in Security Configurations on your Virtual Machines] (Corregir las vulnerabilidades en las configuraciones de seguridad de sus máquinas virtuales) para mantener las configuraciones de seguridad en todos los recursos de proceso.

* [Cómo supervisar las recomendaciones de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-recommendations)

* [Cómo corregir las recomendaciones de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-remediate-recommendations)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Instrucciones**: use las plantillas de Azure Resource Manager y las directivas de Azure para configurar de forma segura los recursos de Azure asociados con Virtual Machine Scale Sets. Las plantillas de Azure Resource Manager son archivos basados en JSON que se usan para implementar la máquina virtual junto con los recursos de Azure, y las plantillas personalizadas deberán mantenerse. Microsoft realiza el mantenimiento en las plantillas base. Use la directiva de Azure [denegar] e [implementar si no existe] para aplicar la configuración segura en los recursos de Azure.

* [Información sobre la creación de plantillas de Azure Resource Manager](https://docs.microsoft.com/azure/virtual-machines/windows/ps-template)

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Descripción de los efectos de Azure Policy](https://docs.microsoft.com/azure/governance/policy/concepts/effects)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Guía**: existen varias opciones para mantener una configuración segura de Azure Virtual Machines (VM) para implementación:

1. Plantillas de Azure Resource Manager: Son archivos basados en JSON que se usan para implementar una máquina virtual desde Azure Portal. Las plantillas personalizadas deberán mantenerse. Microsoft realiza el mantenimiento en las plantillas base.

2. Disco duro virtual (VHD) personalizado: En algunas circunstancias, puede ser necesario usar archivos VHD personalizados, como cuando se trabaja con entornos complejos que no se pueden administrar a través de otros medios.

3. State Configuration de Azure Automation: Una vez implementado el sistema operativo base, se puede usar para un control más pormenorizado de la configuración y para aplicarla a través del marco de automatización.

En la mayoría de los escenarios, las plantillas de máquina virtual base de Microsoft combinadas con Desired State Configuration de Azure Automation pueden ayudar a cumplir y mantener los requisitos de seguridad.

* [Información sobre cómo descargar la plantilla de máquina virtual](https://docs.microsoft.com/azure/virtual-machines/windows/download-template)

* [Información sobre la creación de plantillas ARM](https://docs.microsoft.com/azure/virtual-machines/windows/ps-template)

* [Carga de un VHD de máquina virtual personalizado en Azure](https://docs.microsoft.com/azure-stack/operator/azure-stack-add-vm-image?view=azs-1910)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: use Azure DevOps para almacenar y administrar de forma segura el código, como directivas de Azure personalizadas, plantillas de Azure Resource Manager, scripts de Desired State Configuration, etc.  Para acceder a los recursos que administra en Azure DevOps, como el código, las compilaciones y el seguimiento del trabajo, debe tener los permisos para esos recursos específicos. La mayoría de los permisos se conceden a través de grupos de seguridad integrados, tal como se describe en Permisos y acceso. Puede conceder o denegar permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integra con Azure DevOps, o en Active Directory si se integra con TFS.

* [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops)

* [Acerca de los permisos y los grupos en Azure DevOps](https://docs.microsoft.com/azure/devops/organizations/security/about-permissions)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Instrucciones**: si usa imágenes personalizadas (p. ej., un disco duro virtual), use el control de acceso basado en roles de Azure (RBAC de Azure) para asegurarse de que solo los usuarios autorizados pueden acceder a las imágenes.

* [Descripción de Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/rbac-and-directory-admin-roles)

* [Configuración de Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/quickstart-assign-role-user-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: aproveche Azure Policy para alertar, auditar y aplicar las configuraciones del sistema para las máquinas virtuales. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8: Implementación de herramientas de administración de la configuración para sistemas operativos

**Guía**: Azure Automation State Configuration es un servicio de administración de configuración para los nodos de Desired State Configuration (DSC) de cualquier centro de datos local o en la nube. Permite la escalabilidad en miles de máquinas de forma rápida y sencilla desde una ubicación central y segura. Puede incorporar fácilmente máquinas, asignarles configuraciones declarativas y ver informes que muestran el cumplimiento por parte de cada máquina del estado deseado especificado.

* [Incorporación de máquinas para su administración mediante Azure Automation State Configuration](https://docs.microsoft.com/azure/automation/automation-dsc-onboarding)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: aproveche Azure Security Center para realizar exámenes de línea de base de las máquinas virtuales de Azure. Los métodos adicionales para la configuración automatizada incluyen el uso de State Configuration de Azure Automation.

* [Corrección de recomendaciones en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-remediate-recommendations)

* [Introducción a State Configuration de Azure Automation State Configuration](https://docs.microsoft.com/azure/automation/automation-dsc-getting-started)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Instrucciones**: Azure Automation State Configuration es un servicio de administración de configuración para los nodos de Desired State Configuration (DSC) de cualquier centro de datos local o en la nube. Permite la escalabilidad en miles de máquinas de forma rápida y sencilla desde una ubicación central y segura. Puede incorporar fácilmente máquinas, asignarles configuraciones declarativas y ver informes que muestran el cumplimiento por parte de cada máquina del estado deseado especificado.

* [Incorporación de máquinas para su administración mediante Azure Automation State Configuration](https://docs.microsoft.com/azure/automation/automation-dsc-onboarding)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Instrucciones**: Use Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de secretos para las aplicaciones en la nube.

* [Integración con identidades administradas de Azure](https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity)

* [Creación de un almacén de claves](https://docs.microsoft.com/azure/key-vault/quick-create-portal)

* [Autenticación en Key Vault](../key-vault/general/authentication.md)

* [Asignación de una directiva de acceso de Key Vault](../key-vault/general/assign-access-policy-portal.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Instrucciones**: Use las identidades administradas para proporcionar a los servicios de Azure una identidad administrada automáticamente en Azure AD. Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluyendo Key Vault, sin necesidad de credenciales en el código.

* [Configuración de las identidades administradas](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

* [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Control de seguridad: defensa contra malware](/azure/security/benchmarks/security-control-malware-defense).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Uso de software antimalware administrado centralmente

**Instrucciones**: use Microsoft Antimalware para Windows Virtual Machines en Azure, a fin de supervisar y proteger sus recursos continuamente. necesitará una herramienta de terceros para la protección contra malware en la máquina virtual con Linux de Azure.

* [Configuración de Microsoft Antimalware para Cloud Services y Virtual Machines](https://docs.microsoft.com/azure/security/fundamentals/antimalware)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: no se aplica a las máquinas virtuales de Azure, ya que es un recurso de proceso.

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: No aplicable

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Guía**: cuando se implementa en Windows Virtual Machines, Microsoft Antimalware para Azure instalará automáticamente y de forma predeterminada las actualizaciones más recientes de la firma, la plataforma y el motor. Siga las recomendaciones en Azure Security Center: "Proceso y aplicaciones" para asegurarse de que todos los puntos de conexión estén actualizados con las firmas más recientes. El sistema operativo Windows se puede proteger aún más con seguridad adicional para limitar el riesgo de ataques basados en malware y virus con el servicio Microsoft Defender Advanced Threat Protection que se integra con Azure Security Center.

necesitará una herramienta de terceros para la protección contra malware en la máquina virtual con Linux de Azure.

* [Implementación de Microsoft Antimalware para Azure Cloud Services y Virtual Machines](https://docs.microsoft.com/azure/security/fundamentals/antimalware)

* [Protección contra amenazas avanzada de Microsoft Defender](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/onboard-configure)

* [Configuración de Microsoft Antimalware para Cloud Services y Virtual Machines](https://docs.microsoft.com/azure/virtual-machines/linux/security-recommendations)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Control de seguridad: recuperación de datos](/azure/security/benchmarks/security-control-data-recovery).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: cree una instantánea de la instancia del conjunto de escalado de máquinas virtuales de Azure o del disco administrado conectado a la instancia mediante PowerShell o las API de REST. También puede usar Azure Automation para ejecutar los scripts de copia de seguridad a intervalos regulares.

* [¿Cómo se puede crear una instantánea de una instancia de un conjunto de escalado de máquinas virtuales y de un disco administrado?](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-faq#how-do-i-take-a-snapshot-of-a-virtual-machine-scale-set-instance)

* [Introducción a Azure Automation](https://docs.microsoft.com/azure/automation/automation-intro)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Instrucciones**: cree instantáneas de las máquinas virtuales de Azure o de los discos administrados asociados a esas instancias mediante PowerShell o las API REST. Realice una copia de seguridad de cualquier clave administrada por el cliente en Azure Key Vault.

Habilite Azure Backup y las máquinas virtuales (VM) de destino de Azure, así como la frecuencia y los períodos de retención deseados. Esto incluye una copia de seguridad completa del estado del sistema. Si usa Azure Disk Encryption, la copia de seguridad de la máquina virtual de Azure administra automáticamente la copia de seguridad de las claves administradas por el cliente.

* [Copia de seguridad en máquinas virtuales de Azure cifradas](https://docs.microsoft.com/azure/backup/backup-azure-vms-encryption)

* [Información general sobre la copia de seguridad de máquinas virtuales de Azure](https://docs.microsoft.com/azure/backup/backup-azure-vms-introduction)

* [¿Cómo se puede crear una instantánea de una instancia de un conjunto de escalado de máquinas virtuales y de un disco administrado?](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-faq#how-do-i-take-a-snapshot-of-a-virtual-machine-scale-set-instance)

* [Creación de una copia de seguridad de las claves del almacén de claves en Azure](https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: garantice la capacidad de realizar la restauración de datos de un disco administrado en Azure Backup de forma periódica. Si es necesario, pruebe la restauración del contenido en una red virtual o suscripción aislada. El cliente debe probar la restauración de la copia de seguridad de las claves administradas por el cliente.

Si usa Azure Disk Encryption, puede restaurar los conjuntos de escalado de máquinas virtuales con las claves de cifrado del disco. Cuando use Disk Encryption, podrá restaurar la máquina virtual de Azure con las claves de cifrado de discos.

* [Copia de seguridad en máquinas virtuales de Azure cifradas](https://docs.microsoft.com/azure/backup/backup-azure-vms-encryption)

* [Restauración de un disco y creación de una máquina virtual recuperada en Azure](https://docs.microsoft.com/azure/backup/tutorial-restore-disk)

* [Restauración de las claves del almacén de claves en Azure](https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

* [Habilitar el cifrado de disco para Azure Virtual Machine Scale Sets](https://docs.microsoft.com/azure/virtual-machine-scale-sets/disk-encryption-overview)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: habilite la protección de eliminación para el disco administrado mediante bloqueos. Habilite la eliminación temporal y la protección de purga en Key Vault para proteger las claves contra la eliminación accidental o malintencionada.

* [Bloqueo de recursos para impedir cambios inesperados](https://docs.microsoft.com/azure/azure-resource-manager/management/lock-resources)

* [Introducción a la protección de purga y eliminación temporal en Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-ovw-soft-delete)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Control de seguridad: Respuesta a los incidentes](/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

* [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

* [Uso de la guía de control de incidentes de seguridad de equipos de NIST para la creación de su propio plan de respuesta a incidentes](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) con etiquetas y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure, especialmente los que procesan datos confidenciales. Es su responsabilidad asignar prioridades a la corrección de las alertas en función de la importancia de los recursos y el entorno de Azure donde se produjo el incidente.

* [Alertas de seguridad en el Centro de seguridad de Azure](https://docs.microsoft.com/azure/security-center/security-center-alerts-overview)

* [Uso de etiquetas para organizar los recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Guía**: Realice ejercicios para probar las funcionalidades de respuesta a los incidentes de los sistemas periódicamente para ayudar a proteger los recursos de Azure. Identifique puntos débiles y brechas y revise el plan según sea necesario.

* [Publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Instrucciones**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

* [Establecimiento del contacto de seguridad de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Guía**: Exporte las alertas y recomendaciones de Azure Security Center y mediante la característica de exportación continua para ayudar a identificar los riesgos para los recursos de Azure. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Azure Sentinel.

* [Configuración de la exportación continua](https://docs.microsoft.com/azure/security-center/continuous-export)

* [Transmisión de alertas a Azure Sentinel](https://docs.microsoft.com/azure/sentinel/connect-azure-security-center)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Guía**: Use la característica Automatización de flujo de trabajo de Azure Security Center para desencadenar automáticamente las respuestas a través de Logic Apps en las alertas y recomendaciones de seguridad para proteger los recursos de Azure.

* [Configuración de la automatización de flujo de trabajo y Logic Apps](https://docs.microsoft.com/azure/security-center/workflow-automation)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de pruebas de penetración de sitios activos y ataques simulados en la infraestructura en la nube, los servicios y las aplicaciones administrados por Microsoft.

* [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

* [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](/azure/security/benchmarks/security-baselines-overview).
