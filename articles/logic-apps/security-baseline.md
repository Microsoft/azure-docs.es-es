---
title: Base de referencia de seguridad de Azure para Logic Apps
description: Base de referencia de seguridad de Azure para Logic Apps
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 06/22/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 7dc0913ab206c6b9152d7d6155642c28b430d6f2
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836333"
---
# <a name="azure-security-baseline-for-logic-apps"></a>Base de referencia de seguridad de Azure para Logic Apps

La base de referencia de seguridad de Azure para Logic Apps contiene recomendaciones que le ayudarán a mejorar la posición de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview.md), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para más información, consulte [Introducción a la línea de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Seguridad de las redes

*Para más información, consulte [Control de seguridad: Seguridad de redes](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Guía**: Microsoft implementa y administra los conectores que se ejecutan en el servicio multiinquilino "global" de Logic Apps. Estos conectores proporcionan desencadenadores y acciones para acceder a servicios en la nube, sistemas locales o ambos, incluidos Office 365, Azure Blob Storage, SQL Server, Dynamics, Salesforce, SharePoint y muchos más. Puede usar la etiqueta de servicio AzureConnectors al especificar reglas en el grupo de seguridad de red o en Azure Firewall para permitir el acceso a los recursos relacionados.

En el caso de las aplicaciones lógicas que necesitan acceso directo a los recursos de una red virtual de Azure, puede crear un entorno del servicio de integración (ISE) en el que puede compilar, implementar y ejecutar aplicaciones lógicas en recursos dedicados. Algunas redes virtuales de Azure usan puntos de conexión privados (Azure Private Link) para proporcionar acceso a los servicios de PaaS de Azure, como Azure Storage, Azure Cosmos DB o Azure SQL Database, servicios de asociados o servicios de clientes que se hospedan en Azure. Si las aplicaciones lógicas necesitan acceder a redes virtuales que usan puntos de conexión privados, debe crear, implementar y ejecutar esas aplicaciones lógicas dentro de un ISE.

Al crear el ISE, puede optar por usar puntos de conexión de acceso internos o externos. La selección determina si los desencadenadores de solicitudes o de webhooks de las aplicaciones lógicas del ISE pueden recibir llamadas desde fuera de la red virtual o no. Los puntos de conexión de acceso interno y externo también afectan a la posibilidad de ver o no el historial de ejecución de la aplicación lógica, incluidas las entradas y salidas de una ejecución, desde dentro o fuera de la red virtual.

Asegúrese de que todas las implementaciones de subred de la red virtual que estén relacionadas con el ISE tengan un grupo de seguridad de red aplicado, con controles de acceso a la red específicos para los puertos y orígenes de confianza de su aplicación. Cuando implemente las aplicaciones lógicas en un ISE, use Private Link. Azure Private Link le permite acceder a los servicios PaaS de Azure y a los servicios hospedados en Azure que son propiedad de los clientes, o a los servicios de asociados, a través de un punto de conexión privado de la red virtual. Como alternativa, si tiene un caso de uso específico, puede cumplir el requisito con la implementación de Azure Firewall. Para ayudar a reducir la complejidad que conlleva la configuración de reglas de seguridad, use etiquetas de servicio que representen grupos de prefijos de direcciones IP de un servicio de Azure concreto.

* [Información sobre los conectores para Logic Apps](../connectors/apis-list.md)

* [Información sobre las etiquetas de servicio en Azure](../virtual-network/service-tags-overview.md)

* [Información sobre el acceso a recursos de Azure Virtual Network desde Azure Logic Apps mediante entornos de servicio de integración (ISE)](./connect-virtual-network-vnet-isolated-environment-overview.md)

* [Introducción a los puntos de conexión de servicio de red virtual](../virtual-network/virtual-network-service-endpoints-overview.md)

* [¿Qué es Azure Private Link?](../private-link/private-link-overview.md)

* [Información sobre el acceso al punto de conexión del ISE](./connect-virtual-network-vnet-isolated-environment-overview.md#ise-endpoint-access)

* [Creación de una red virtual](../virtual-network/quick-create-portal.md)

* [Creación de un NSG con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

* [Implementación y configuración de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)

* [Habilitación del acceso para el ISE](./connect-virtual-network-vnet-isolated-environment.md#enable-access-for-ise)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Compartido

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes y NIC

**Instrucciones**: Si ejecuta Logic Apps en un entorno de servicio de integración (ISE) que usa un punto de acceso externo, puede utilizar un grupo de seguridad de red (NSG) para reducir el riesgo de filtración de datos. Habilite los registros de flujo del grupo de seguridad de red y envíe registros a una cuenta de Azure Storage para auditar el tráfico. También puede enviar registros de flujo del grupo de seguridad de red a un área de trabajo de Log Analytics y usar Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

* [Información sobre el acceso al punto de conexión del ISE](./connect-virtual-network-vnet-isolated-environment-overview.md#ise-endpoint-access)

* [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: No aplicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: Si la aplicación lógica usa un desencadenador basado en solicitudes que recibe llamadas o solicitudes entrantes, como el desencadenador Solicitud o Webhook, puede limitar el acceso para que solo los clientes autorizados puedan llamar a la aplicación lógica.

Si está ejecutando Logic Apps en un entorno de servicio de integración (ISE), habilite DDoS Protection estándar en la red virtual asociada a su ISE para protegerse frente a ataques de DDoS. Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

Implemente Azure Firewall en cada uno de los límites de red de la organización con la inteligencia de amenazas habilitada y configurada para alertar y denegar el tráfico de red malintencionado.

Use el acceso de red Just-in-Time de Azure Security Center para configurar los NSG para limitar la exposición de los puntos de conexión a las direcciones IP aprobadas durante un período limitado.

Use la protección de redes adaptativa de Azure Security Center para recomendar configuraciones de NSG que limiten los puertos y las direcciones IP de origen según el tráfico real y la inteligencia sobre amenazas.

* [Protección de las llamadas entrantes a Logic Apps](./logic-apps-securing-a-logic-app.md#access-to-request-based-triggers)

* [Restricción de direcciones IP entrantes](./logic-apps-securing-a-logic-app.md#restrict-inbound-ip-addresses)

* [Configuración de la protección contra DDoS](../virtual-network/manage-ddos-protection.md)

* [Implementación de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)

* [Descripción de la inteligencia sobre amenazas integrada de Azure Security Center](../security-center/threat-protection.md)

* [Descripción de la protección de red adaptable de Azure Security Center](../security-center/security-center-adaptive-network-hardening.md)

* [Descripción del control de acceso de red Just-in-Time de Azure Security Center](../security-center/security-center-just-in-time.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Instrucciones**: Si ejecuta Logic Apps en un entorno de servicio de integración (ISE) que usa un punto de acceso externo, puede utilizar un grupo de seguridad de red (NSG) para reducir el riesgo de filtración de datos. Habilite los registros de flujo del grupo de seguridad de red y envíe registros a una cuenta de Azure Storage para auditar el tráfico. También puede enviar registros de flujo del grupo de seguridad de red a un área de trabajo de Log Analytics y usar Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Para proporcionar más protección e información sobre el tráfico de red, puede consultar los registros de acceso, que se generan solo si los ha habilitado en cada una de las instancias de Application Gateway. Puede usar este registro para ver los patrones de acceso de Application Gateway y analizar información importante. Esto incluye la dirección IP del autor de la llamada, la dirección URL solicitada, la latencia de la respuesta, el código de devolución y los bytes de entrada y salida.

De lo contrario, puede aprovechar una solución de terceros de Marketplace para satisfacer este requisito.

* [Información sobre el acceso al punto de conexión del ISE](./connect-virtual-network-vnet-isolated-environment-overview.md#ise-endpoint-access)

* [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

* [Integración de API Management en una red virtual interna con Application Gateway](../api-management/api-management-howto-integrate-internal-vnet-appgateway.md)

* [Descripción de los registros de acceso de WAF](../web-application-firewall/ag/web-application-firewall-logs.md#access-log)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implementación de sistemas de prevención de intrusiones y de detección de intrusiones (IDS/IPS) basados en la red.

**Guía**: Seleccione una oferta de Azure Marketplace que admita la funcionalidad de IDS/IPS con funcionalidades de inspección de carga. Si la detección y/o la prevención de intrusiones basadas en la inspección de carga no es un requisito, se puede usar Azure Firewall con la inteligencia sobre amenazas. El filtrado basado en inteligencia sobre amenazas de Azure Firewall puede alertar y denegar el tráfico desde y hacia los dominios y las direcciones IP malintencionados conocidos. La direcciones IP y los dominios proceden de la fuente Inteligencia sobre amenazas de Microsoft.

Implemente la solución de firewall que prefiera en cada uno de los límites de red de su organización para detectar y/o denegar el tráfico malintencionado.

* [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

* [Implementación de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)

* [Configuración de alertas con Azure Firewall](../firewall/threat-intel.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Guía**: Si está ejecutando Logic Apps en un entorno de servicio de integración (ISE), implemente Azure Application Gateway.

* [Integración de API Management en una red virtual interna con Application Gateway](../api-management/api-management-howto-integrate-internal-vnet-appgateway.md)

* [Configuración de Application Gateway para usar HTTPS](../application-gateway/create-ssl-portal.md)

* [Descripción del equilibrio de carga de capa 7 con las puertas de enlace de aplicaciones web de Azure](../application-gateway/overview.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: En el caso de los recursos que necesitan acceso a las instancias de Azure Logic Apps, use las etiquetas de servicio de red virtual con el fin de definir controles de acceso a la red en grupos de seguridad de red o Azure Firewall. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, LogicApps, LogicAppsManagement) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

* [Más información sobre el uso de etiquetas de servicio](../virtual-network/service-tags-overview.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: Defina e implemente configuraciones de seguridad estándar para los recursos de red relacionados con las instancias de Azure Logic Apps con Azure Policy. Use alias de Azure Policy en los espacios de nombres "Microsoft.Logic" y "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de las instancias de Azure Logic Apps. También puede utilizar definiciones de directivas integradas como:

Los registros de diagnóstico de Logic Apps deben estar habilitados

Se debe habilitar DDoS Protection estándar

También puede utilizar Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como plantillas de Azure Resource Manager, control de acceso basado en rol (RBAC) y directivas, en una única definición de un plano técnico. Aplique fácilmente el plano técnico a nuevas suscripciones y entornos, y ajuste el control y la administración mediante el control de versiones.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Creación de un plano técnico de Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: En el caso de las reglas de NSG individuales, use el campo "Descripción" para especificar las necesidades empresariales o la duración (etc.) de las reglas que permiten que entre o salga el tráfico en una red.

Use cualquiera de las definiciones de Azure Policy integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor", para asegurarse de que todos los recursos se creen con etiquetas y para notificarle los recursos no etiquetados existentes.

Puede usar Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas.

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

* [Creación de una red virtual](../virtual-network/quick-create-portal.md)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

* [Lista de definiciones de Azure Policy para Logic Apps](./policy-samples.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: Use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en los recursos de red relacionados con las instancias de Azure Logic Apps. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.

* [Visualización y recuperación de eventos del registro de actividad de Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

* [Creación de alertas en Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Control de seguridad: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: Microsoft utiliza el mismo origen de la hora que los recursos de Azure, como Azure Logic Apps, para establecer las marcas de tiempo en los registros.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Si necesita información de depuración más completa sobre las aplicaciones lógicas durante el tiempo de ejecución, puede configurar y usar registros de Azure Monitor para registrar y almacenar información sobre los eventos y datos en tiempo de ejecución; por ejemplo, eventos de desencadenador, eventos de ejecución y eventos de acción en un área de trabajo de Log Analytics. Azure Monitor le ayuda a supervisar los entornos locales y en la nube para que pueda mantener más fácilmente su disponibilidad y rendimiento. Con los registros de Azure Monitor, puede crear consultas de registro que le ayuden a recopilar y revisar esta información. También puede usar estos datos de diagnóstico con otros servicios de Azure, como Azure Storage y Azure Event Hubs.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

* [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/activity-log.md)

* [Configuración de registros de Azure Monitor y recopilación de datos de diagnóstico para Azure Logic Apps](./monitor-logic-apps-log-analytics.md)

* [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Instrucciones**: Si necesita información de depuración más completa sobre las aplicaciones lógicas durante el tiempo de ejecución, puede configurar y usar registros de Azure Monitor para registrar y almacenar información sobre los eventos y datos en tiempo de ejecución; por ejemplo, eventos de desencadenador, eventos de ejecución y eventos de acción en un área de trabajo de Log Analytics. Azure Monitor le ayuda a supervisar los entornos locales y en la nube para que pueda mantener más fácilmente su disponibilidad y rendimiento. Con los registros de Azure Monitor, puede crear consultas de registro que le ayuden a recopilar y revisar esta información. También puede usar estos datos de diagnóstico con otros servicios de Azure, como Azure Storage y Azure Event Hubs.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

* [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/activity-log.md)

* [Configuración de registros de Azure Monitor y recopilación de datos de diagnóstico para Azure Logic Apps](./monitor-logic-apps-log-analytics.md)

* [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: No aplicable

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Instrucciones**: Después de crear y ejecutar una aplicación lógica, puede comprobar su estado de ejecución, el historial de ejecuciones, el historial de los desencadenadores y el rendimiento. Para la supervisión de eventos en tiempo real y una depuración más completa, configure el registro de diagnóstico de la aplicación lógica mediante los registros de Azure Monitor. Este servicio de Azure le ayuda a supervisar los entornos locales y en la nube para que pueda mantener más fácilmente su disponibilidad y rendimiento. De este modo, puede buscar y ver eventos, como los de desencadenador, de ejecución y de acción. Al almacenar esta información en registros de Azure Monitor, puede crear consultas de registro que le ayuden a buscar y analizar dicha información. También puede usar estos datos de diagnóstico con otros servicios de Azure, como Azure Storage y Azure Event Hubs.

En Azure Monitor, establezca el período de retención de los registros asociados a las instancias de Azure Logic Apps, de acuerdo con las normativas de cumplimiento de su organización.

* [Supervisión del estado de ejecución, revisión del historial de los desencadenadores y configuración de alertas en Azure Logic Apps](./monitor-logic-apps.md)

* [Establecimiento de parámetros de retención de registros](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Guía**: Para configurar el registro de la aplicación lógica, puede habilitar Log Analytics al crear la aplicación lógica, o bien instalar la solución de administración de Logic Apps en el área de trabajo de Log Analytics para las aplicaciones lógicas existentes. Esta solución proporciona información agregada para las ejecuciones de aplicaciones lógicas e incluye detalles específicos, como el estado, el tiempo de ejecución, el estado de reenvío y los identificadores de correlación. Luego, para habilitar el registro y la creación de consultas para esta información, configure los registros de Azure Monitor.

También puede habilitar la configuración de diagnóstico en el registro de actividad de Azure y enviar los registros a un área de trabajo de Log Analytics. Realice consultas en Log Analytics para buscar términos, identificar tendencias, analizar patrones y extraer otra información detallada de los datos del registro de actividad que se hayan recopilado para Azure Logic Apps.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

* [Configuración de registros de Azure Monitor y recopilación de datos de diagnóstico para Azure Logic Apps](./monitor-logic-apps-log-analytics.md)

* [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/activity-log.md)

* [Recopilación y análisis de registros de actividad de Azure en un área de trabajo de Log Analytics en Azure Monitor](../azure-monitor/platform/activity-log.md)

* [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Instrucciones**: Use Azure Security Center con Log Analytics para la supervisión y alerta de actividades anómalas que se encuentren en los registros y eventos de seguridad.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel.

* [Incoporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

* [Administración de alertas de seguridad en Azure Security Center](../security-center/security-center-managing-and-responding-alerts.md)

* [Alertas sobre datos de registro de Log Analytics](../azure-monitor/learn/tutorial-response.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Instrucciones**: No aplicable; Azure Logic Apps no procesa ni genera registros relacionados con antimalware.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: No aplicable

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: No aplicable; Azure Logic Apps no procesa ni genera registros relacionados con DNS.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: No aplicable

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: No aplicable

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para más información, consulte [Control de seguridad: Identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Instrucciones**: Azure Active Directory (AD) tiene roles integrados que se deben asignar explícitamente y son consultables. Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc para detectar cuentas que son miembros de grupos administrativos.

Para acceder fácilmente a otros recursos protegidos por Azure Active Directory (Azure AD) y autenticar la identidad sin iniciar sesión, la aplicación lógica puede usar una identidad administrada (anteriormente conocida como Managed Service Identity o MSI), en lugar de credenciales o secretos. Azure administra esta identidad y le ayuda a proteger las credenciales porque, de esta forma, no tiene que proporcionar secretos o cambiarlos.

Cada de punto de conexión de solicitud de una aplicación lógica incluye una Firma de acceso compartido (SAS) en la dirección URL del punto de conexión. Si comparte la dirección URL del punto de conexión de un desencadenador basado en solicitudes con otras partes, puede generar direcciones URL de devolución de llamada con claves específicas y fechas de expiración. De esta forma puede acumular claves fácilmente o restringir el acceso para desencadenar la aplicación lógica en función de un determinado intervalo de tiempo.

* [Obtención de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

* [Autenticación del acceso a los recursos de Azure mediante identidades administradas en Azure Logic Apps](./create-managed-service-identity.md)

* [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

* [Protección del acceso y los datos en Azure Logic Apps mediante SAS](./logic-apps-securing-a-logic-app.md#access-to-request-based-triggers)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: Azure Active Directory y Azure Logic Apps no tienen el concepto de contraseñas predeterminadas.

Si se utiliza la autenticación básica, tendrá que especificar un nombre de usuario y una contraseña. Al crear estas credenciales, asegúrese de configurar una contraseña segura para la autenticación.

Si usa la infraestructura como código, evite almacenar las contraseñas en el código y, en su lugar, use Azure Key Vault para almacenar y recuperar las credenciales.

* [Protección del acceso y los datos en Logic Apps](./logic-apps-securing-a-logic-app.md)

* [Establecimiento y recuperación de un secreto desde Azure Key Vault](../key-vault/secrets/quick-create-portal.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

Además, para ayudarle a realizar un seguimiento de las cuentas administrativas dedicadas, puede seguir las recomendaciones de Azure Security Center o las directivas integradas de Azure Policy, por ejemplo:
- Debe haber más de un propietario asignado a su suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción

* [Uso de Azure Security Center para supervisar la identidad y el acceso (versión preliminar)](../security-center/security-center-identity-access.md)

* [Uso de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Instrucciones**: Use un registro de aplicaciones de Azure (entidad de servicio) para recuperar un token que se pueda usar para interactuar con los almacenes de Recovery Services a través de llamadas API.

Muchos conectores también requieren que primero se cree una conexión al servicio o sistema de destino, y que proporcione credenciales de autenticación u otros detalles de configuración antes de poder usar un desencadenador o una acción en la aplicación lógica. Por ejemplo, tendrá que autorizar una conexión a una cuenta de Twitter para acceder a los datos o para publicar en su nombre.]

En el caso de los conectores que usan la autenticación OAuth de Azure Active Directory (Azure AD), crear una conexión significa iniciar sesión en el servicio (por ejemplo, Office 365, Salesforce o GitHub), donde el token de acceso se cifra y se almacena de forma segura en un almacén de secretos de Azure. Otros conectores (como FTP y SQL) requieren una conexión con detalles de configuración como la dirección del servidor, el nombre de usuario y la contraseña. Estos detalles sobre la configuración de la conexión también se cifran y se almacenan de forma segura.

* [Llamada a las API REST de Azure](/rest/api/azure/#how-to-call-azure-rest-apis-with-postman)

* [Registro de la aplicación cliente (entidad de servicio) con Azure AD](/rest/api/azure/#register-your-client-application-with-azure-ad)

* [Información de API de desencadenadores de flujo de trabajo](/rest/api/logic/workflowtriggers)

* [Información sobre la configuración del conector](../connectors/apis-list.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Habilite Multi-Factor Authentication (MFA) de Azure Active Directory (AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

* [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

* [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Instrucciones**: Utilice estaciones de trabajo de acceso con privilegios (PAW), con Multi-Factor Authentication (MFA) configurado para iniciar sesión en recursos de Azure y configurarlos.

* [Más información sobre las estaciones de trabajo con privilegios de acceso](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: Use Privileged Identity Management (PIM) de Azure Active Directory (AD) para la generación de registros y alertas cuando se produzca una actividad sospechosa o insegura en el entorno.

Además, use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

* [Cómo implementar Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

* [Información sobre las detecciones de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: El cliente debe usar ubicaciones con nombre de acceso condicional para permitir el acceso a Azure Portal solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

Además, cada punto de conexión de solicitud de una aplicación lógica incluye una Firma de acceso compartido (SAS) en la dirección URL del punto de conexión. Puede restringir la aplicación lógica para que acepte solo solicitudes de ciertas direcciones IP.

* [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

* [Información para restringir las direcciones IP entrantes en Logic Apps](./logic-apps-securing-a-logic-app.md#restrict-inbound-ip-addresses)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización para las instancias de Azure Logic Apps. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

Cuando Logic Apps lo permita, use una identidad administrada para acceder fácilmente a otros recursos protegidos por Azure Active Directory (Azure AD) y autenticar la identidad sin iniciar sesión, en lugar de usar credenciales o secretos. Azure administra esta identidad y le ayuda a proteger las credenciales porque, de esta forma, no tiene que proporcionar secretos o cambiarlos.

Azure Logic Apps admite identidades administradas tanto asignadas por el sistema como por el usuario. Su aplicación lógica puede usar la identidad asignada por el sistema o una sola identidad asignada por el usuario, que puede compartir en un grupo de aplicaciones lógicas, pero no ambas. Actualmente, solo las acciones y los desencadenadores integrados específicos admiten identidades administradas y conexiones o conectores no administrados, por ejemplo:
- HTTP
- Azure Functions
- Azure API Management
- Azure App Services

* [Procedimiento para crear y configurar una instancia de Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

* [Autenticación del acceso a los recursos de Azure mediante identidades administradas en Azure Logic Apps](./create-managed-service-identity.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure Active Directory (AD) proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

* [Descripción de los informes de Azure AD](../active-directory/reports-monitoring/index.yml)

* [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización para las instancias de Azure Logic Apps. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

Tiene acceso a los orígenes de registro de eventos de actividad de inicio de sesión, auditoría y riesgo de Azure AD, que permiten la integración en Azure Sentinel o SIEM de terceros.

Para simplificar este proceso, cree una configuración de diagnóstico para las cuentas de usuario de Azure AD y envíe los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas de registro deseadas en Log Analytics.

* [Integración de los registros de actividad de Azure en Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

* [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Instrucciones**: Use las características de protección de riesgos e identidad de Azure AD para configurar las respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. También puede hacer que Azure Sentinel ingiera los datos para investigarlos más.

* [Visualización de los inicios de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

* [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

* [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: Actualmente no disponible; la Caja de seguridad del cliente todavía no se admite en Azure Logic Apps.

* [Lista de servicios admitidos por la Caja de seguridad del cliente](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

## <a name="data-protection"></a>Protección de los datos

*Para más información, consulte [Control de seguridad: Protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: use etiquetas para ayudar a realizar el seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Microsoft implementa y administra los conectores que se ejecutan en el servicio multiinquilino "global" de Logic Apps. Estos conectores proporcionan desencadenadores y acciones para acceder a servicios en la nube, sistemas locales o ambos, incluidos Office 365, Azure Blob Storage, SQL Server, Dynamics, Salesforce, SharePoint y muchos más.

En el caso de las aplicaciones lógicas que necesitan acceso directo a los recursos de una red virtual de Azure, puede crear un entorno del servicio de integración (ISE) en el que puede compilar, implementar y ejecutar aplicaciones lógicas en recursos dedicados. Algunas redes virtuales de Azure usan puntos de conexión privados (Azure Private Link) para proporcionar acceso a los servicios de PaaS de Azure, como Azure Storage, Azure Cosmos DB o Azure SQL Database, servicios de asociados o servicios de clientes que se hospedan en Azure. Si las aplicaciones lógicas necesitan acceder a redes virtuales que usan puntos de conexión privados, debe crear, implementar y ejecutar esas aplicaciones lógicas dentro de un ISE.

Al crear el ISE, puede optar por usar puntos de conexión de acceso internos o externos. La selección determina si los desencadenadores de solicitudes o de webhooks de las aplicaciones lógicas del ISE pueden recibir llamadas desde fuera de la red virtual o no.

Implemente también el aislamiento mediante suscripciones independientes y grupos de administración para dominios de seguridad individuales, como el tipo de entorno y el nivel de confidencialidad de los datos. Puede restringir el nivel de acceso a los recursos de Azure que necesitan sus aplicaciones y entornos empresariales. Puede controlar el acceso a los recursos de Azure a través del control de acceso basado en rol (Azure RBAC).

* [Información sobre los conectores para Logic Apps](../connectors/apis-list.md)

* [Acceso a recursos de Azure Virtual Network desde Azure Logic Apps mediante entornos de servicio de integración (ISE)](./connect-virtual-network-vnet-isolated-environment-overview.md)

* [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

* [Creación de grupos de administración](../governance/management-groups/create.md)

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Actualmente no disponible; las características de identificación, clasificación y prevención de pérdida de datos no están disponibles aún en Azure Logic Apps.

Use una solución de terceros de Azure Marketplace en los perímetros de red que supervise la transferencia no autorizada de información confidencial y bloquee dichas transferencias al tiempo que alerta a los profesionales de seguridad de la información.

Microsoft administra la infraestructura subyacente de Azure Logic Apps y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

* [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Compartido

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Instrucciones**: Cifre toda la información confidencial en tránsito. En Azure Logic Apps, durante la ejecución de una aplicación lógica, todos los datos se cifran en tránsito mediante Seguridad de la capa de transporte (TLS) y en reposo. Cuando visualiza el historial de ejecución de la aplicación lógica, Logic Apps autentica su acceso y proporciona vínculos a las entradas y salidas para las solicitudes y respuestas de cada ejecución. Sin embargo, para las acciones que controlan contraseñas, secretos, claves u otra información confidencial, es recomendable evitar que otros usuarios vean los datos y accedan a ellos. Por ejemplo, si la aplicación lógica obtiene un secreto de Azure Key Vault para usarlo al autenticar una acción HTTP, puede ocultar ese secreto.

El desencadenador de solicitud admite solo la seguridad de la capa de transporte (TLS) 1.2 para las solicitudes entrantes. Asegúrese de que los clientes que se conectan a los recursos de Azure puedan negociar TLS 1.2 o superior. Las llamadas salientes que usan el conector HTTP admiten la seguridad de la capa de transporte (TLS) 1.0, 1.1 y 1.2.

Siga las recomendaciones de Azure Security Center para el cifrado en reposo y el cifrado en tránsito, si procede.

* [Proteger el acceso y los datos en Azure Logic Apps](./logic-apps-securing-a-logic-app.md)

* [Recepción y respuesta de solicitudes HTTPS entrantes en Azure Logic Apps](../connectors/connectors-native-reqres.md#tls-support)

* [Llamada a puntos de conexión de servicio mediante HTTP o HTTPS desde Azure Logic Apps](../connectors/connectors-native-http.md#tls-support)

* [Descripción del cifrado en tránsito con Azure](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit)

* [Información sobre el cifrado de datos en reposo con Azure](../security/fundamentals/encryption-atrest.md)

* [Configuración de las claves administradas por el cliente para cifrar los datos en reposo para los entornos de servicio de integración (ISE) en Azure Logic Apps](./customer-managed-keys-integration-service-environment.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Compartido

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: En Azure Logic Apps, muchos desencadenadores y acciones tienen valores de configuración que se pueden habilitar para proteger las entradas, las salidas o ambas mediante la ocultación de los datos del historial de ejecución de una aplicación lógica.

Microsoft administra la infraestructura subyacente de Azure Logic Apps y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

* [Protección del acceso a los datos del historial de ejecución](./logic-apps-securing-a-logic-app.md#access-to-run-history-data)

* [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Compartido

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6: Uso del control de acceso basado en rol para controlar el acceso a los recursos

**Guía**: Puede permitir que solo determinados usuarios o grupos ejecuten tareas específicas, como administrar, editar y ver aplicaciones lógicas. Para controlar sus permisos, use el control de acceso basado en rol (RBAC) de Azure de forma que se puedan asignar roles personalizados o integrados a los miembros de la suscripción a Azure:
- Colaborador de aplicaciones lógicas: Le permite administrar aplicaciones lógicas, pero no puede cambiar el acceso a ellas.
- Operador de aplicaciones lógicas: Le permite leer, habilitar y deshabilitar aplicaciones lógicas, pero no puede editarlas ni actualizarlas.

Para evitar que otros usuarios cambien o eliminen la aplicación lógica, puede usar el bloqueo de recursos de Azure. Esta funcionalidad evita que otros usuarios cambien o eliminen los recursos de producción.

* [Protección del acceso a las operaciones de Azure Logic Apps](./logic-apps-securing-a-logic-app.md#access-to-logic-app-operations)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso. Microsoft administra la infraestructura subyacente de Azure Logic Apps y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

* [Protección de datos de cliente de Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Microsoft

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Azure Logic Apps se basa en Azure Storage para almacenar y cifrar automáticamente los datos en reposo. Este cifrado protege los datos y le ayuda a cumplir los compromisos de cumplimiento y seguridad de la organización. De forma predeterminada, Azure Storage usa claves que administra Microsoft para cifrar sus datos.

Cuando crea un entorno de servicio de integración (ISE) para hospedar las aplicaciones lógicas y desea tener un mayor control sobre las claves de cifrado que usa Azure Storage, puede configurar, usar y administrar su propia clave mediante Azure Key Vault. Esta funcionalidad también se conoce como "Bring Your Own Key" (BYOK) y la clave se denomina "clave administrada por el cliente".

* [Cifrado de los datos en reposo para los entornos de servicio de integración en Azure Logic Apps](./customer-managed-keys-integration-service-environment.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure para crear alertas cuando se produzcan cambios en Azure Logic Apps y en otros recursos críticos o relacionados.

* [Creación de alertas para los eventos del registro de actividad de Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Control de seguridad: Administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: [No aplicable; Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Azure Logic Apps].

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3: Implementación de una solución de administración de revisiones automatizada de títulos de software de terceros

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Guía**: No aplicable; Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Azure Logic Apps.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Guía**: No aplicable; Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Azure Logic Apps.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para más información, consulte [Control de seguridad: Administración de recursos y del inventario](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: Use Azure Resource Graph para consultar o detectar todos los recursos (por ejemplo, proceso, almacenamiento, red, puertos y protocolos, etc.) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y enumere todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, a partir de ahora es muy recomendable crear y usar los recursos con Azure Resource Manager.

* [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

* [Visualización de las suscripciones de Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

* [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Instrucciones**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos de Azure. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

Además, use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

* [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

* [Creación de grupos de administración](../governance/management-groups/create.md)

* [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: Cree un inventario de los recursos de Azure aprobados (como los conectores) y el software aprobado para los recursos de proceso según las necesidades de la organización.

Nota: Debido a las directivas de privacidad y datos de Google, puede usar el conector de Gmail solo con los servicios aprobados de Google. Esta situación está en evolución y puede afectar a otros conectores de Google en el futuro.

* [Lista de todos los conectores de Logic Apps](/connectors/connector-reference/connector-reference-logicapps-connectors)

* [Información sobre los problemas y las limitaciones de los conectores de Gmail](/connectors/gmail/#known-issues-and-limitations)

* [Más información sobre la Directiva de privacidad de Google](../connectors/connectors-google-data-security-privacy-policy.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en sus suscripciones.

Use Azure Resource Graph para consultar o detectar recursos dentro de sus suscripciones. Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Denegación de un tipo de recurso específico con Azure Policy](../governance/policy/samples/index.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10: Mantenimiento de un inventario de títulos de software aprobados

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: Configure el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management.

* [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Guía**: Los recursos relacionados con la instancia de Logic Apps que son necesarios para las operaciones empresariales pero que pueden suponer un riesgo mayor para la organización, deben aislarse en su propia máquina virtual o red virtual, y mantenerse lo suficientemente protegidos con Azure Firewall o un grupo de seguridad de red.

Las instancias de Logic Apps que son necesarias para las operaciones empresariales, pero pueden suponer un riesgo mayor para la organización, deben aislarse siempre que sea posible en grupos de recursos independientes con permisos específicos y límites de RBAC.

* [Creación de una red virtual](../virtual-network/quick-create-portal.md)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

* [Creación de grupos de administración](../governance/management-groups/create.md)

* [Protección del acceso a Logic Apps mediante RBAC](./logic-apps-securing-a-logic-app.md#access-to-logic-app-operations)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Control de seguridad: Configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Defina e implemente configuraciones de seguridad estándar para las instancias de Azure Logic Apps con Azure Policy. Use alias de Azure Policy en el espacio de nombres "Microsoft.Logic" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de las instancias de Logic Apps. Por ejemplo, puede impedir que otros creen o usen conexiones a los recursos a los que desea restringir el acceso.

Además, Azure Resource Manager tiene la capacidad de exportar la plantilla en notación de objetos JavaScript (JSON), que se debe revisar para asegurarse de que las configuraciones cumplan o superen los requisitos de seguridad de la organización.

Use también parámetros seguros para proteger datos confidenciales y secretos.

* [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Bloqueo de conexiones creadas por conectores en Azure Logic Apps](./block-connections-connectors.md)

* [Exportación de uno y varios recursos a una plantilla en Azure Portal](../azure-resource-manager/templates/export-template-portal.md)

* [Implementación de plantillas de Azure Resource Manager para Azure Logic Apps](./logic-apps-deploy-azure-resource-manager-templates.md)

* [Información sobre los parámetros de acción segura](./logic-apps-securing-a-logic-app.md#secure-action-parameters)

* [Recomendaciones de seguridad para parámetros](../azure-resource-manager/templates/template-best-practices.md#security-recommendations-for-parameters)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Utilice las directivas [deny] y [deploy if not exist] de Azure Policy para aplicar una configuración segura en los recursos de Azure.

Defina e implemente configuraciones de seguridad estándar para las instancias de Azure Logic Apps con Azure Policy. Use alias de Azure Policy en el espacio de nombres "Microsoft.Logic" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de las instancias de Logic Apps. Por ejemplo, puede impedir que otros creen o usen conexiones a los recursos a los que desea restringir el acceso.

Además, Azure Resource Manager tiene la capacidad de exportar la plantilla en notación de objetos JavaScript (JSON), que se debe revisar para asegurarse de que las configuraciones cumplan o superen los requisitos de seguridad de la organización.

Además, asegúrese de proteger los datos del historial de ejecución mediante ofuscación.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

* [Bloqueo de conexiones creadas por conectores en Azure Logic Apps](./block-connections-connectors.md)

* [Exportación de uno y varios recursos a una plantilla en Azure Portal](../azure-resource-manager/templates/export-template-portal.md)

* [Implementación de plantillas de Azure Resource Manager para Azure Logic Apps](./logic-apps-deploy-azure-resource-manager-templates.md)

* [Protección del acceso a las entradas y salidas del historial de ejecución](./logic-apps-securing-a-logic-app.md#obfuscate)

* [Protección del acceso a las entradas de parámetros](./logic-apps-securing-a-logic-app.md#secure-action-parameters)

* [Recomendaciones de seguridad para parámetros](../azure-resource-manager/templates/template-best-practices.md#security-recommendations-for-parameters)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Compartido

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Si usa definiciones personalizadas de Azure Policy, use Azure DevOps o Azure Repos para almacenar y administrar el código de forma segura.

Además, Azure Resource Manager tiene la capacidad de exportar la plantilla en notación de objetos JavaScript (JSON), que se debe revisar para asegurarse de que las configuraciones cumplan o superen los requisitos de seguridad de la organización.

* [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

* [Documentación de Azure Repos](/azure/devops/repos/index?view=azure-devops)

* [Exportación de uno y varios recursos a una plantilla en Azure Portal](../azure-resource-manager/templates/export-template-portal.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: Use las definiciones integradas en Azure Policy junto con los alias de esta misma aplicación en el espacio de nombres "Microsoft.Logic" para crear directivas personalizadas que permitan auditar y aplicar las configuraciones, así como enviar alertas sobre ellas. Use alias de Azure Policy para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de los recursos de Azure. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8: Implementación de herramientas de administración de la configuración para sistemas operativos

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: Use las definiciones integradas en Azure Policy junto con los alias de esta misma aplicación en el espacio de nombres "Microsoft.Logic" para crear directivas personalizadas que permitan auditar y aplicar las configuraciones, así como enviar alertas sobre ellas. Use la directiva de Azure Policy [audit], [deny] y [deploy if not exist] para aplicar automáticamente las configuraciones en los recursos de Azure.

* [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: Proteja las entradas y salidas del historial de ejecución de la aplicación lógica mediante ofuscación. Si realiza la implementación en entornos diferentes, considere la posibilidad de parametrizar los valores de la definición de flujo de trabajo de la aplicación lógica, que varía en función de esos entornos. De este modo, puede evitar los datos codificados de forma rígida mediante una plantilla de Azure Resource Manager para implementar la aplicación lógica, proteger los datos confidenciales definiendo parámetros seguros y pasar dichos datos como entradas separadas mediante los parámetros de la plantilla con un archivo de parámetros. Puede usar Key Vault para almacenar datos confidenciales y usar parámetros de plantilla protegidos que recuperen esos valores de Key Vault en la implementación. A continuación, puede hacer referencia a Key Vault y a los secretos en el archivo de parámetros.

Cuando crea un entorno de servicio de integración (ISE) para hospedar las aplicaciones lógicas y desea tener un mayor control sobre las claves de cifrado que usa Azure Storage, puede configurar, usar y administrar su propia clave mediante Azure Key Vault. Esta funcionalidad también se conoce como "Bring Your Own Key" (BYOK) y la clave se denomina "clave administrada por el cliente".

* [Protección de entradas y salidas en el historial de ejecución en Azure Logic Apps](./logic-apps-securing-a-logic-app.md#obfuscate)

* [Recomendaciones de seguridad para parámetros](../azure-resource-manager/templates/template-best-practices.md#security-recommendations-for-parameters)

* [Protección del acceso a las entradas de parámetros en Azure Logic Apps](./logic-apps-securing-a-logic-app.md#access-to-parameter-inputs)

* [Paso de los valores de parámetro seguros durante la implementación mediante Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md)

* [Configuración de las claves administradas por el cliente para cifrar los datos en reposo para los entornos de servicio de integración (ISE) en Azure Logic Apps](./customer-managed-keys-integration-service-environment.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: Para acceder fácilmente a otros recursos protegidos por Azure Active Directory (Azure AD) y autenticar la identidad sin iniciar sesión, la aplicación lógica puede usar una identidad administrada (anteriormente conocida como Managed Service Identity o MSI), en lugar de credenciales o secretos. Azure administra esta identidad y le ayuda a proteger las credenciales porque, de esta forma, no tiene que proporcionar secretos o cambiarlos.

Actualmente, solo determinadas acciones y desencadenadores integrados admiten identidades administradas y conexiones o conectores no administrados, por ejemplo:
- HTTP
- Azure Functions
- Azure API Management
- Azure App Services

* [Autenticación del acceso a los recursos de Azure mediante identidades administradas en Azure Logic Apps](./create-managed-service-identity.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Guía**: Proteja las entradas y salidas del historial de ejecución de la aplicación lógica mediante ofuscación. Si realiza la implementación en entornos diferentes, considere la posibilidad de parametrizar los valores de la definición de flujo de trabajo de la aplicación lógica, que varía en función de esos entornos. De este modo, puede evitar los datos codificados de forma rígida mediante una plantilla de Azure Resource Manager para implementar la aplicación lógica, proteger los datos confidenciales definiendo parámetros seguros y pasar dichos datos como entradas separadas mediante los parámetros de la plantilla con un archivo de parámetros. Puede usar Key Vault para almacenar datos confidenciales y usar parámetros de plantilla protegidos que recuperen esos valores de Key Vault en la implementación. A continuación, puede hacer referencia a Key Vault y a los secretos en el archivo de parámetros.

También puede implementar el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

* [Protección de entradas y salidas en el historial de ejecución en Azure Logic Apps](./logic-apps-securing-a-logic-app.md#obfuscate)

* [Recomendaciones de seguridad para parámetros](../azure-resource-manager/templates/template-best-practices.md#security-recommendations-for-parameters)

* [Protección del acceso a las entradas de parámetros en Azure Logic Apps](./logic-apps-securing-a-logic-app.md#access-to-parameter-inputs)

* [Paso de los valores de parámetro seguros durante la implementación mediante Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md)

* [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Control de seguridad: defensa contra malware](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Use software antimalware administrado centralmente

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso. Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (como Azure Logic Apps), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Backup), pero no se ejecuta en el contenido.

Examine previamente los archivos que se cargan en recursos de Azure que no son de proceso, como App Service, Data Lake Storage, Blob Storage, etc.

Use la detección de amenazas de Azure Security Center para los servicios de datos para detectar malware cargado en cuentas de almacenamiento.

* [Información sobre Microsoft Antimalware para Azure Cloud Services y Virtual Machines](../security/fundamentals/antimalware.md)

* [Descripción de la detección de amenazas de Azure Security Center para servicios de datos](../security-center/threat-protection.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Control de seguridad: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: Implemente una solución de recuperación ante desastres para poder proteger los datos, restaurar rápidamente los recursos que sirven de apoyo a funciones empresariales críticas y seguir ejecutando las operaciones para mantener la continuidad empresarial.

El objetivo de esta estrategia de recuperación ante desastres es configurar la aplicación lógica principal para que realice una conmutación por error en una aplicación lógica de reserva o en espera, en una ubicación alternativa donde también esté disponible Azure Logic Apps. De este modo, si la aplicación lógica principal sufre pérdidas, interrupciones o errores, la secundaria puede continuar el trabajo. En esta estrategia es necesario que la aplicación lógica secundaria y los recursos dependientes ya estén implementados y listos en la ubicación alternativa.

Además, debe expandir la definición del flujo de trabajo subyacente de la aplicación lógica en una plantilla de Azure Resource Manager. Esta plantilla define la infraestructura, los recursos, los parámetros y otra información para aprovisionar e implementar la aplicación lógica.

* [Más información sobre continuidad empresarial y recuperación ante desastres para Azure Logic Apps](./business-continuity-disaster-recovery-guidance.md)

* [Automatización de la implementación para Azure Logic Apps mediante plantillas de Azure Resource Manager](./logic-apps-azure-resource-manager-templates-overview.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Implemente una solución de recuperación ante desastres para poder proteger los datos, restaurar rápidamente los recursos que sirven de apoyo a funciones empresariales críticas y seguir ejecutando las operaciones para mantener la continuidad empresarial.

El objetivo de esta estrategia de recuperación ante desastres es configurar la aplicación lógica principal para que realice una conmutación por error en una aplicación lógica de reserva o en espera, en una ubicación alternativa donde también esté disponible Azure Logic Apps. De este modo, si la aplicación lógica principal sufre pérdidas, interrupciones o errores, la secundaria puede continuar el trabajo. En esta estrategia es necesario que la aplicación lógica secundaria y los recursos dependientes ya estén implementados y listos en la ubicación alternativa.

Además, debe expandir la definición del flujo de trabajo subyacente de la aplicación lógica en una plantilla de Azure Resource Manager. Esta plantilla define la infraestructura, los recursos, los parámetros y otra información para aprovisionar e implementar la aplicación lógica.

Cada de punto de conexión de solicitud de una aplicación lógica incluye una Firma de acceso compartido (SAS) en la dirección URL del punto de conexión. Si usa Azure Key Vault para almacenar los secretos, asegúrese de que se realizan con regularidad copias de seguridad automatizadas de las claves y las direcciones URL.

* [Más información sobre continuidad empresarial y recuperación ante desastres para Azure Logic Apps](./business-continuity-disaster-recovery-guidance.md)

* [Automatización de la implementación para Azure Logic Apps mediante plantillas de Azure Resource Manager](./logic-apps-azure-resource-manager-templates-overview.md)

* [Protección del acceso y los datos en Azure Logic Apps mediante SAS](./logic-apps-securing-a-logic-app.md#access-to-request-based-triggers)

* [Copia de seguridad de claves de Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: Su estrategia de recuperación ante desastres se debe centrarse en configurar la aplicación lógica principal para que realice una conmutación por error en una aplicación lógica en espera o de reserva, en una ubicación alternativa donde también esté disponible Azure Logic Apps. De este modo, si la aplicación lógica principal sufre pérdidas, interrupciones o errores, la secundaria puede continuar el trabajo. En esta estrategia es necesario que la aplicación lógica secundaria y los recursos dependientes ya estén implementados y listos en la ubicación alternativa.

Pruebe la restauración de la copia de seguridad de las claves administradas por el cliente. Tenga en cuenta que esto solo se aplica a las instancias de Logic Apps que se ejecutan en entornos de servicio de integración (ISE).

* [Más información sobre continuidad empresarial y recuperación ante desastres para Azure Logic Apps](./business-continuity-disaster-recovery-guidance.md)

* [Configuración de las claves administradas por el cliente para cifrar los datos en reposo para los entornos de servicio de integración (ISE) en Azure Logic Apps](./customer-managed-keys-integration-service-environment.md)

* [Restauración de las claves del almacén de claves en Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas del cliente

**Guía**: Su estrategia de recuperación ante desastres se debe centrarse en configurar la aplicación lógica principal para que realice una conmutación por error en una aplicación lógica en espera o de reserva, en una ubicación alternativa donde también esté disponible Azure Logic Apps. De este modo, si la aplicación lógica principal sufre pérdidas, interrupciones o errores, la secundaria puede continuar el trabajo. En esta estrategia es necesario que la aplicación lógica secundaria y los recursos dependientes ya estén implementados y listos en la ubicación alternativa.

Proteja las claves administradas por el cliente. Tenga en cuenta que esto solo se aplica a las instancias de Logic Apps que se ejecutan en entornos de servicio de integración (ISE).

Habilite la eliminación temporal y la protección de purga en Key Vault para proteger las claves contra la eliminación accidental o malintencionada.

* [Más información sobre continuidad empresarial y recuperación ante desastres para Azure Logic Apps](./business-continuity-disaster-recovery-guidance.md)

* [Configuración de las claves administradas por el cliente para cifrar los datos en reposo para los entornos de servicio de integración (ISE) en Azure Logic Apps](./customer-managed-keys-integration-service-environment.md)

* [Procedimiento para habilitar la eliminación temporal y la protección de purga en Key Vault](../storage/blobs/soft-delete-overview.md?tabs=azure-portal)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Control de seguridad: Respuesta a los incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

* [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

* [Uso de la guía de control de incidentes de seguridad de equipos de NIST para la creación de su propio plan de respuesta a incidentes](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) con etiquetas y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure, especialmente los que procesan datos confidenciales. Es su responsabilidad asignar prioridades a la corrección de las alertas en función de la importancia de los recursos y el entorno de Azure donde se produjo el incidente.

* [Alertas de seguridad en el Centro de seguridad de Azure](../security-center/security-center-alerts-overview.md)

* [Uso de etiquetas para organizar los recursos de Azure](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las funcionalidades de respuesta a los incidentes de los sistemas periódicamente; así, ayudará a proteger los recursos de Azure. Identifique puntos débiles y brechas y revise el plan según sea necesario.

* [Publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Instrucciones**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

* [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Guía**: Exporte las alertas y recomendaciones de Azure Security Center y mediante la característica de exportación continua para ayudar a identificar los riesgos para los recursos de Azure. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Azure Sentinel.

* [Configuración de la exportación continua](../security-center/continuous-export.md)

* [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Guía**: Use la característica Automatización de flujo de trabajo de Azure Security Center para desencadenar automáticamente las respuestas a través de Logic Apps en las alertas y recomendaciones de seguridad para proteger los recursos de Azure.

* [Configuración de la automatización de flujo de trabajo y Logic Apps](../security-center/workflow-automation.md)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft.

* [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

* [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](../security/benchmarks/overview.md).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).