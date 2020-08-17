---
title: Base de referencia de seguridad de Azure para Cosmos DB
description: Base de referencia de seguridad de Azure para Cosmos DB
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 03/16/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 3431ad88a27da9d83b75ed3661d6129f157b9b1f
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87531513"
---
# <a name="azure-security-baseline-for-cosmos-db"></a>Base de referencia de seguridad de Azure para Cosmos DB

La base de referencia de seguridad de Azure para Cosmos DB contiene recomendaciones que lo ayudarán a mejorar la posición de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](https://docs.microsoft.com/azure/security/benchmarks/overview), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para obtener más información, consulte [Introducción a las líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).

## <a name="network-security"></a>Seguridad de redes

*Para obtener más información, consulte [Control de seguridad: seguridad de red](https://docs.microsoft.com/azure/security/benchmarks/security-control-network-security).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1: Proteja los recursos mediante grupos de seguridad de red o Azure Firewall en su red virtual

**Guía**: Al usar Azure Private Link, puede conectarse a una cuenta de Azure Cosmos a través de un punto de conexión privado. El tráfico entre la red virtual y el servicio atraviesa la red troncal de Microsoft, eliminando la exposición a la red pública de Internet. También puede reducir el riesgo de la exfiltración de datos mediante la configuración de un conjunto estricto de reglas de salida en un grupo de seguridad de red (NSG) y la Asociación de ese NSG con la subred.

También puede usar puntos de conexión de servicio para proteger su cuenta de Azure Cosmos. Al habilitar un punto de conexión de servicio, puede configurar las cuentas de Azure Cosmos DB para permitir el acceso solo desde una subred específica de una red virtual de Azure. Una vez habilitado el punto de conexión de servicio de Azure Cosmos DB, puede limitar el acceso a la cuenta de Azure Cosmos con conexiones de una subred en una red virtual.

Puede proteger los datos almacenados en su cuenta de Azure Cosmos mediante el uso de firewalls por IP. Azure Cosmos DB admite controles de acceso basado en IP para la compatibilidad con el firewall de entrada. Puede establecer firewall por IP en la cuenta de Azure Cosmos mediante Azure Portal, plantillas de Azure Resource Manager, o a través de la CLI de Azure o Azure PowerShell.

Información general de Azure Private Link: https://docs.microsoft.com/azure/private-link/private-link-overview

Configuración de un punto de conexión privado de Azure Private Link para Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/how-to-configure-private-endpoints 

Cómo crear un grupo de seguridad de red con una configuración de seguridad: https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic

Configuración del firewall de IP en Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/how-to-configure-firewall

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2: Supervise y registre la configuración y el tráfico de redes virtuales, subredes y NIC

**Instrucciones**: Use Azure Security Center y siga las recomendaciones de protección de redes para ayudar a proteger sus recursos de red relacionados a la cuenta de Azure Cosmos.

Cuando se implementan máquinas virtuales en la misma red virtual que la cuenta de Azure Cosmos, puede usar los grupos de seguridad de red (NSG) para reducir el riesgo de exfiltración de datos. Habilite los registros de flujo de grupos de seguridad de red y envíe registros a una cuenta de Azure Storage para la auditoría del tráfico. También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Descripción de la seguridad de red proporcionada por Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

Cómo habilitar los registros de flujo de NSG: https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Habilitación y uso del Análisis de tráfico: https://docs.microsoft.com/azure/network-watcher/traffic-analytics

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Guía**: El uso compartido de recursos entre orígenes (CORS) es una característica de HTTP que permite que una aplicación web que se ejecuta en un dominio tenga acceso a recursos de otro dominio. Los exploradores web implementan una restricción de seguridad que se conoce como directiva de mismo origen que impide que una página web realice llamadas API en un dominio diferente. Sin embargo, CORS aporta un modo seguro de permitir que el dominio de origen llame a las API de otro dominio. Después de habilitar la compatibilidad con CORS para la cuenta de Azure Cosmos, solo se evalúan las solicitudes autenticadas para determinar si se admiten según las reglas especificadas.

Configuración del uso compartido de recursos entre orígenes (CORS): https://docs.microsoft.com/azure/cosmos-db/how-to-configure-cross-origin-resource-sharing

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: Use Advanced Threat Protection (ATP) para Azure Cosmos DB. Advanced Threat Protection para Azure Cosmos DB proporciona un nivel adicional de inteligencia de seguridad que detecta intentos poco habituales y potencialmente peligrosos de acceder a las cuentas de Azure Cosmos DB o vulnerarlas. Esta capa de protección le permite afrontar las amenazas e integrarlas con sistemas centrales de supervisión de seguridad.

Habilite DDoS Protection estándar en las redes virtuales asociadas a las instancias de Azure Cosmos DB a fin de protegerse de los ataques DDoS. Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

Cómo configurar Advanced Threat Protection para Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/cosmos-db-advanced-threat-protection

Cómo configurar la protección contra DDoS: https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection

Descripción de la inteligencia sobre amenazas integrada de Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5: Registre los paquetes de red y registros de flujo

**Guía**: Habilite los registros de flujo del grupo de seguridad de red (NSG) y envíelos a una cuenta de Storage para auditar el tráfico. También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar conclusiones cognitivas sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Cómo habilitar los registros de flujo de NSG: https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Habilitación y uso del Análisis de tráfico: https://docs.microsoft.com/azure/network-watcher/traffic-analytics

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implemente sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Instrucciones**: Use Advanced Threat Protection (ATP) para Azure Cosmos DB. Advanced Threat Protection para Azure Cosmos DB proporciona un nivel adicional de inteligencia de seguridad que detecta intentos poco habituales y potencialmente peligrosos de acceder a las cuentas de Azure Cosmos DB o vulnerarlas. Esta capa de protección le permite afrontar las amenazas e integrarlas con sistemas centrales de supervisión de seguridad. 

Cómo configurar Advanced Threat Protection para Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/cosmos-db-advanced-threat-protection

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Guía**: No aplicable; la recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Instrucciones**: En el caso de los recursos que necesitan acceso a la cuenta de Azure Cosmos, use las etiquetas de servicio de Virtual Network con el fin de definir controles de acceso a la red en grupos de seguridad de red o Azure Firewall. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, AzureCosmosDB) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

Para más información sobre el uso de etiquetas de servicio, consulte: https://docs.microsoft.com/azure/virtual-network/service-tags-overview

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: Defina e implemente configuraciones de seguridad estándar para los recursos de red con Azure Policy. Use alias de Azure Policy en los espacios de nombres "Microsoft.DocumentDB" y "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de las instancias de Azure Cosmos DB. También puede usar definiciones de directivas integradas para Azure Cosmos DB, como:

- Implementar Advanced Threat Protection para cuentas de Cosmos DB

- Cosmos DB debe usar un punto de conexión de servicio de red virtual

También puede utilizar Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como plantillas de Azure Resource Manager (ARM), control de acceso basado en rol (RBAC) y directivas, en una única definición de plano técnico. Aplique fácilmente el plano técnico a nuevas suscripciones y entornos, y ajuste el control y la administración mediante el control de versiones.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Creación de un plano técnico de Azure: https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Instrucciones**: Use etiquetas para los recursos de red asociados a la implementación de Azure Cosmos DB a fin de organizarlos lógicamente en una taxonomía.

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Instrucciones**: Use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en los recursos de red relacionados con las instancias de Azure Cosmos DB. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos. 

Visualización y recuperación de eventos del registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view

Creación de alertas en Azure Monitor: https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para obtener más información, consulte [Control de seguridad: registro y supervisión](https://docs.microsoft.com/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: Microsoft mantiene el origen de la hora usado en los recursos de Azure, como Azure Cosmos DB, para las marcas de tiempo de los registros.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Ingiera registros a través de Azure Monitor para agregar datos de seguridad generados por Azure Cosmos DB. En Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y utilice cuentas para el almacenamiento de archivos a largo plazo. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros. 

Cómo habilitar los registros de diagnóstico para Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/logging

Cómo incorporar Azure Sentinel: https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Habilite la configuración de diagnóstico para Azure Cosmos DB y envíe los registros a la misma área de trabajo de Log Analytics o cuenta de almacenamiento. La configuración de diagnóstico de Azure Cosmos DB se usa para recopilar los registros de los recursos. Los registros se capturan por solicitud y también se denominan "registros de plano de datos". Algunos ejemplos de las operaciones del plano de datos son Delete, Insert y Read. También puede habilitar la configuración de diagnóstico del registro de actividad de Azure y enviar los registros a la misma área de trabajo de Log Analytics.

Cómo habilitar la configuración de diagnóstico para Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/logging

Cómo habilitar la configuración de diagnóstico para el registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención de registro para las áreas de trabajo de Log Analytics asociadas a las instancias de Azure Cosmos DB según las normativas de cumplimiento de su organización.

Cómo establecer parámetros de retención de registros: https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Guía**: Realice consultas en el área de trabajo de Log Analytics para buscar términos, identificar tendencias, analizar patrones y proporcionar muchas otras conclusiones basadas en los datos del registro de actividad que se pueden recopilar para Azure Cosmos DB.

Cómo realizar consultas en áreas de trabajo de Log Analytics para Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/monitor-cosmos-db

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: En Azure Security Center, habilite protección contra amenazas avanzada para Azure Cosmos DB para supervisar actividades anómalas en registros y eventos de seguridad. Habilite la configuración de diagnóstico en Azure Cosmos DB y envíe registros a un área de trabajo de Log Analytics.

 

También puede incorporar el área de trabajo de Log Analytics a Azure Sentinel, ya que proporciona una solución de respuesta automatizada de orquestación de seguridad (SOAR). Esto permite crear cuadernos de estrategias (soluciones automatizadas) y usarlos para corregir problemas de seguridad. Además, puede crear alertas de registro personalizadas en el área de trabajo Log Analytics mediante Azure Monitor.

Lista de alertas de protección contra amenazas para Azure Cosmos DB: https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-azurecosmos

Cómo incorporar Azure Sentinel: https://docs.microsoft.com/azure/sentinel/quickstart-onboard

Creación, visualización y administración de alertas de registro mediante Azure Monitor: https://docs.microsoft.com/azure/azure-monitor/platform/alerts-log

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: No aplicable; Azure Cosmos DB no procesa ni genera registros relacionados con antimalware.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: No aplicable; Azure Cosmos DB no procesa ni genera registros relacionados con DNS.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para obtener más información, consulte [Control de seguridad: identidad y control de acceso](https://docs.microsoft.com/azure/security/benchmarks/security-control-identity-access-control).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Instrucciones**: Puede usar el panel de identidades y control de acceso (IAM) en el Azure Portal para configurar el control de acceso basado en rol (RBAC) y mantener el inventario en los recursos de Azure Cosmos DB. Los roles se aplican a usuarios, grupos, entidades de servicio e identidades administradas en Active Directory. Puede usar roles integrados o personalizados para usuarios y grupos.

Azure Cosmos DB proporciona un control de acceso basado en role (RBAC) integrado para escenarios de administración comunes en Azure Cosmos DB. Un usuario que tiene un perfil en Azure Active Directory puede asignar estos roles de Azure a usuarios, grupos, entidades de servicio o identidades administradas para conceder o denegar el acceso a recursos y operaciones en los recursos de Azure Cosmos DB.

Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc a fin de detectar cuentas que son miembros de grupos administrativos. 

Además, algunas acciones de Azure Cosmos DB se pueden controlar con Azure Active Directory y claves maestras específicas de la cuenta.  Use la configuración de la cuenta "disableKeyBasedMetadataWriteAccess" para controlar el acceso a las claves.

Información acerca del control de acceso basado en roles de Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/role-based-access-control

Cree sus propios roles personalizados con acciones de Azure Cosmos DB (espacio de nombres Microsoft.DocumentDB): https://docs.microsoft.com/azure/role-based-access-control/resource-provider-operations#microsoftdocumentdb

Creación de un rol en Azure Active Directory: https://docs.microsoft.com/azure/role-based-access-control/custom-roles

Cómo obtener un rol de directorio en Azure Active Directory con PowerShell: https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0

Cómo obtener los miembros de un rol de directorio en Azure Active Directory con PowerShell: https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0

Restricción del acceso de los usuarios solo a las operaciones de datos: https://docs.microsoft.com/azure/cosmos-db/how-to-restrict-user-data

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: El concepto de contraseñas en blanco o predeterminadas no existe en relación con Azure AD o Azure Cosmos DB. En su lugar, Azure Cosmos DB usa dos tipos de claves para autenticar usuarios y proporcionar acceso a sus datos y recursos; claves maestras y token de recurso. Las claves se pueden regenerar en cualquier momento.

Información sobre la protección del acceso a los datos en Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/secure-access-to-data

Cómo regenerar las claves de Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/manage-with-powershell#regenerate-keys

Cómo obtener acceso a las claves mediante programación con Azure Active Directory: https://docs.microsoft.com/azure/cosmos-db/certificate-based-authentication

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Instrucciones**: No aplicable; Azure Cosmos DB no es compatible con las cuentas de administrador.  Todo el acceso se integra con Azure Active Directory y el control de acceso basado en roles (Azure RBAC) de Azure.



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Guía**: Azure Cosmos DB usa dos tipos de claves para autenticar a los usuarios y no admite el inicio de sesión único (SSO) en el nivel del plano de datos. El acceso al plano de control de Azure Cosmos DB está disponible mediante la API REST y admite SSO. Para autenticarse, establezca el encabezado de autorización de las solicitudes en JSON Web Token, que se obtiene de Azure Active Directory.

Información sobre la API REST de Azure Database para Cosmos DB: https://docs.microsoft.com/rest/api/cosmos-db/

Información sobre SSO con Azure Active Directory: https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Habilite Multi-Factor Authentication (MFA) de Azure Active Directory y siga las recomendaciones de administración de identidad y acceso de Azure Security Center.

Cómo habilitar la MFA en Azure: https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Cómo supervisar la identidad y el acceso en Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: Utilice estaciones de trabajo de acceso con privilegios (PAW), con Multi-Factor Authentication (MFA) configurado para iniciar sesión en recursos de Azure y configurarlos.

Más información sobre las estaciones de trabajo con privilegios de acceso: https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations

Cómo habilitar la MFA en Azure: https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7: Registre y alerte de la actividad sospechosa desde cuentas administrativas

**Instrucciones**: Use Advanced Threat Protection (ATP) para Azure Cosmos DB. Advanced Threat Protection para Azure Cosmos DB proporciona un nivel adicional de inteligencia de seguridad que detecta intentos poco habituales y potencialmente peligrosos de acceder a las cuentas de Azure Cosmos DB o vulnerarlas. Esta capa de protección le permite afrontar las amenazas e integrarlas con sistemas centrales de supervisión de seguridad. 

Así mismo, puede usar Azure Active Directory (AD) Privileged Identity Management (PIM) para la generación de registros y alertas cuando se produzca actividad sospechosa o insegura en el entorno.

Use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

Cómo implementar Privileged Identity Management (PIM): https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan

Información sobre las detecciones de riesgo de Azure AD: https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Instrucciones**: Configure la condición de ubicación de una directiva de acceso condicional y administre las ubicaciones con nombre. Gracias a las ubicaciones con nombre, puede crear agrupaciones lógicas de rangos de direcciones IP o de países y regiones. Puede restringir el acceso a recursos confidenciales, como las instancias de Azure Cosmos DB, a las ubicaciones con nombre configuradas.

Configuración de ubicaciones con nombre en Azure: https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

Creación y configuración de una nueva instancia de Azure Active Directory: https://docs.microsoft.com/azure/active-directory-domain-services/tutorial-create-instance

Configuración y administración de la autenticación de Azure Active Directory con Azure SQL: https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure Active Directory proporciona registros para ayudar a descubrir cuentas obsoletas. Además, puede utilizar las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

Cómo usar las revisiones de acceso de identidad de Azure: https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11: Supervise los intentos de acceso a cuentas desactivadas

**Guía**: Puede simplificar este proceso creando una configuración de diagnóstico para las cuentas de usuario de Azure Active Directory y enviando los registros de auditoría y de inicio de sesión a un área de trabajo de Log Analytics.

Integración de los registros de actividad de Azure en Azure Monitor: https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Use Advanced Threat Protection (ATP) para Azure Cosmos DB. Advanced Threat Protection para Azure Cosmos DB proporciona un nivel adicional de inteligencia de seguridad que detecta intentos poco habituales y potencialmente peligrosos de acceder a las cuentas de Azure Cosmos DB o vulnerarlas. Esta capa de protección le permite afrontar las amenazas e integrarlas con sistemas centrales de supervisión de seguridad. 

También puede usar las características de Azure AD Identity Protection y detección de riesgo para configurar respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. Además, puede ingerir datos en Azure Sentinel para investigarlos más a fondo.

Cómo ver inicios de sesión peligrosos en Azure Active Directory: https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

Configuración y habilitación de las directivas de riesgo de protección de identidad: https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

Incorporación de Azure Sentinel: https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: Actualmente, no está disponible; la Caja de seguridad del cliente todavía no se admite en Azure Database para Cosmos DB.

Lista de servicios admitidos por la Caja de seguridad del cliente: https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: No aplicable

## <a name="data-protection"></a>Protección de datos

*Para obtener más información, consulte [Control de seguridad: protección de datos](https://docs.microsoft.com/azure/security/benchmarks/security-control-data-protection).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Guía**: Use etiquetas para ayudar a realizar el seguimiento de las instancias de Azure Cosmos DB que almacenan o procesan información confidencial.

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Las instancias de Azure Cosmos DB deben separarse mediante una red virtual o subred, etiquetarse adecuadamente y protegerse con grupos de seguridad de red (NSG) o Azure Firewall. Las instancias de Azure Cosmos DB que almacenan datos confidenciales deben estar aisladas. Al usar Azure Private Link, puede conectarse a una cuenta de instancia de Azure Cosmos DB a través de un punto de conexión privado. El punto de conexión privado es un conjunto de direcciones IP privadas en una subred dentro de la red virtual. Puede limitar el acceso a las direcciones IP privadas seleccionadas. 

Creación de suscripciones adicionales de Azure: https://docs.microsoft.com/azure/billing/billing-create-subscription

Procedimiento para crear grupos de administración: https://docs.microsoft.com/azure/governance/management-groups/create

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Cómo configurar un punto de conexión privado para Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/how-to-configure-private-endpoints

Cómo crear un grupo de seguridad de red con una configuración de seguridad: https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Puede implementar Advanced Threat Protection para Azure Cosmos DB que detecta actividades anómalas que indican intentos inusuales y potencialmente maliciosos de acceso o ataque a las bases de datos. Actualmente, puede desencadenar las siguientes alertas:

- Acceso desde ubicaciones inusuales

- Extracción de datos inusual

De manera adicional, al usar máquinas virtuales para tener acceso a las instancias de Azure Cosmos DB, aproveche Private Link, el firewall, los grupos de seguridad de red y las etiquetas de servicio para mitigar la posibilidad de que se produzca una filtración de datos. Microsoft administra la infraestructura subyacente para Azure Cosmos DB y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

Cómo configurar Advanced Threat Protection para Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/cosmos-db-advanced-threat-protection

Descripción de la protección de datos de los clientes en Azure: https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: Todas las conexiones a Azure Cosmos DB admiten HTTPS. Azure Cosmos DB también admite TLS1.2. Es posible aplicar una versión de TLS mínima en el servidor. Para ello, póngase en contacto con [azurecosmosdbtls@service.microsoft.com](mailto:azurecosmosdbtls@service.microsoft.com).

Introducción a la seguridad de Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/database-security

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: Los datos automáticos de identificación, clasificación y prevención de pérdida de datos no están disponibles aún en Azure Cosmos DB. Sin embargo, puede usar la integración de Azure Cognitive Search para la clasificación y el análisis de datos. También puede implementar una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

Indexación de datos Azure Cosmos DB con Azure Cognitive Search: https://docs.microsoft.com/azure/search/search-howto-index-cosmosdb?toc=/azure/cosmos-db/toc.json&amp;bc=/azure/cosmos-db/breadcrumb/toc.json

Descripción de la protección de datos de los clientes en Azure: https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos

**Guía**: Azure Cosmos DB proporciona un control de acceso basado en role (RBAC) integrado para escenarios de administración comunes en Azure Cosmos DB. Los usuarios con un perfil en Azure Active Directory pueden asignar estos roles de Azure a usuarios, grupos, entidades de servicio o identidades administradas para conceder o denegar el acceso a recursos y operaciones en los recursos de Azure Cosmos DB. Las asignaciones de roles están dirigidas únicamente al acceso al plano de control, que incluye el acceso a las cuentas, bases de datos, contenedores y ofertas de Azure Cosmos (rendimiento).

Cómo implementar RBAC en Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/role-based-access-control

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

Microsoft administra la infraestructura subyacente para Cosmos DB y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

Descripción de la protección de datos de los clientes en Azure: https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Todos los datos almacenados en Azure Cosmos DB se cifran en reposo. No hay ningún control para activarlo o desactivarlo. Azure Cosmos DB usa el cifrado de AES-256 en todas las regiones donde se ejecuta la cuenta.

de manera predeterminada, Microsoft administra las claves que se usan para cifrar los datos de la cuenta de Azure Cosmos. también puede optar por agregar una segunda capa de cifrado con sus propias claves.

Información sobre el cifrado en reposo con Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/database-encryption-at-rest

Información sobre la administración de claves para el cifrado en reposo con Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/cosmos-db-security-controls

Cómo configurar las claves administradas por el cliente para la cuenta de Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/how-to-setup-cmk

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Instrucciones**: Use Azure Monitor con el registro de actividad de Azure para crear alertas para cuando se produzcan cambios en las instancias de Azure Cosmos DB.

Cómo crear alertas para eventos de registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

Cómo crear alertas para eventos de registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para obtener más información, consulte [Control de seguridad: administración de vulnerabilidades](https://docs.microsoft.com/azure/security/benchmarks/security-control-vulnerability-management).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Instrucciones**: Siga las recomendaciones de Azure Security Center para las instancias de Azure Cosmos DB. 

Microsoft realiza revisiones del sistema y administración de vulnerabilidades en los hosts subyacentes que admiten las instancias de Azure Cosmos DB. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

Características compatibles disponibles en Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-services?tabs=features-windows

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3: Implemente una solución automatizada de administración de revisiones de software de terceros

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para obtener más información, consulte [Control de seguridad: inventario y administración de recursos](https://docs.microsoft.com/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-use-azure-asset-discovery"></a>6.1: Uso de la detección de recursos de Azure

**Guía**: Use el Azure Portal o el Azure Resource Graph para detectar todos los recursos (no limitados a Azure Cosmos DB, sino también recursos como proceso, otro almacenamiento, red, puertos y protocolos, etc.) dentro de sus suscripciones.  Asegúrese de que tiene los permisos adecuados en el suscriptor y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager que figuran a continuación.

Creación de consultas con Azure Resource Graph: https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Visualización de las suscripciones de Azure: https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0

Información sobre el control de acceso basado en rol de Azure: https://docs.microsoft.com/azure/role-based-access-control/overview

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: Aplique etiquetas a las instancias de Azure Cosmos DB y recursos relacionados que proporcionen metadatos para organizarlos lógicamente en una taxonomía.

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Qué recursos de Azure Cosmos DB admiten etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/management/tag-support#microsoftdocumentdb

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento, incluido pero no limitado a los recursos de Azure Cosmos DB. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

Creación de suscripciones adicionales de Azure: https://docs.microsoft.com/azure/billing/billing-create-subscription

Creación de grupos de administración: https://docs.microsoft.com/azure/governance/management-groups/create

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4: Mantenimiento de un inventario de los recursos de Azure aprobados y títulos de software

**Guía**: No aplicable; esta directriz está destinada a los recursos de proceso y a Azure en su conjunto.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Instrucciones**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

Además, use Azure Resource Graph para consultar o detectar recursos dentro de sus suscripciones.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Creación de consultas con Azure Graph: https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Guía**: No aplicable; esta base de referencia está pensada para recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: No aplicable; esta directriz está destinada a los recursos de proceso y a Azure en su conjunto.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos 

- Tipos de recursos permitidos

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Denegación de un tipo de recurso específico con Azure Policy: https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="610-implement-approved-application-list"></a>6.10: Implementación de la lista de aplicaciones aprobadas

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="611-limit-users-ability-to-interact-with-azureresources-manager-via-scripts"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager mediante scripts

**Instrucciones**: Use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management. Esto puede impedir la creación y los cambios en los recursos de un entorno de alta seguridad.

Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager: https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso

**Instrucciones**: No aplicable; esta guía está pensada para recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Guía**: No aplicable; esta directriz está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="secure-configuration"></a>Configuración segura

*Para obtener más información, consulte [Control de seguridad: configuración segura](https://docs.microsoft.com/azure/security/benchmarks/security-control-secure-configuration).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Instrucciones**: Defina e implemente configuraciones de seguridad estándar para las instancias de Azure Cosmos DB con Azure Policy. Use los alias de Azure Policy en el espacio de nombres "Microsoft.DocumentDB" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de las instancias de Cosmos DB. También puede usar definiciones de directivas integradas para Azure Cosmos DB, como:

- Implementar Advanced Threat Protection para cuentas de Cosmos DB

- Cosmos DB debe usar un punto de conexión de servicio de red virtual

Visualización de los alias de Azure Policy disponibles: https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Utilice las directivas [deny] y [deploy if not exist] de Azure Policy para aplicar una configuración segura en los recursos de Azure.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Descripción de los efectos de Azure Policy: https://docs.microsoft.com/azure/governance/policy/concepts/effects

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Si usa definiciones personalizadas de Azure Policy para los recursos de Cosmos DB u otros relacionados, use Azure Repos para almacenar y administrar el código de forma segura.

Documentación de Azure Repos: https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7: Implemente herramientas de administración de configuración del sistema

**Guía**: Use los alias de Azure Policy en el espacio de nombres "Microsoft.DocumentDB" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8: Implemente herramientas de administración de configuración del sistema para sistemas operativos

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9: Implemente la supervisión de configuración automatizada para servicios de Azure

**Guía**: Use los alias de Azure Policy en el espacio de nombres "Microsoft.DocumentDB" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Use [audit], [deny] y [deploy if not exist] de Azure Policy para aplicar automáticamente las configuraciones de las instancias de Azure Cosmos DB y recursos relacionados. 

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: En el caso de las máquinas virtuales de Azure o las aplicaciones web que se ejecutan en Azure App Service y se usan para tener acceso a las instancias de Azure Cosmos DB, utilice Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de secretos de Azure Cosmos DB. Asegúrese de que la eliminación temporal de Key Vault esté habilitada.

Cómo integrar las identidades administradas de Azure: https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

Cómo crear un almacén de claves: https://docs.microsoft.com/azure/key-vault/quick-create-portal

Cómo proporcionar la autenticación de Key Vault con una identidad administrada: https://docs.microsoft.com/azure/key-vault/managed-identity

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: En el caso de las máquinas virtuales de Azure o las aplicaciones web que se ejecutan en Azure App Service y se usan para tener acceso a las instancias de Azure Cosmos DB, utilice Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de secretos de Azure Cosmos DB.

Use las identidades administradas para proporcionar a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory (AD). Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluyendo Key Vault, sin necesidad de credenciales en el código.

Cómo configurar las identidades administradas: https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm

Cómo integrar las identidades administradas de Azure: https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

Cómo configurar el escáner de credenciales: https://secdevtools.azurewebsites.net/helpcredscan.html

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para obtener más información, consulte [Control de seguridad: defensa contra malware](https://docs.microsoft.com/azure/security/benchmarks/security-control-malware-defense).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Use software antimalware administrado centralmente

**Guía**: No aplicable; esta guía está pensada para recursos de proceso. Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure App Service), pero no se ejecuta en el contenido del cliente.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Instrucciones**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure App Service), pero no se ejecuta en el contenido del cliente.

Es su responsabilidad realizar un análisis previo de los archivos que se cargan en recursos de Azure que no son de proceso, incluido Azure Cosmos DB. Microsoft no puede acceder a los datos del cliente y, por lo tanto, no puede realizar exámenes antimalware del contenido del cliente en su nombre.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Guía**: No aplicable; la prueba comparativa está pensada para recursos de proceso. Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure, pero no se ejecuta en el contenido del cliente.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-recovery"></a>Recuperación de datos

*Para obtener más información, consulte [Control de seguridad: recuperación de datos](https://docs.microsoft.com/azure/security/benchmarks/security-control-data-recovery).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: Azure Cosmos DB captura instantáneas de los datos cada cuatro horas. Todas las copias de seguridad se almacenan por separado en un servicio de almacenamiento y se replican globalmente para lograr resistencia frente a desastres regionales. En un momento dado del tiempo, solo las dos últimas instantáneas se conservan. Pero si se elimina el contenedor o la base de datos, Azure Cosmos DB conserva las instantáneas existentes de un contenedor o base de datos dado durante 30 días. Póngase en contacto con el soporte técnico de Azure para hacer una restauración desde una copia de seguridad.

Reconocimiento de las copias de seguridad automatizadas de Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/online-backup-and-restore

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Azure Cosmos DB crea automáticamente copias de seguridad de los datos a intervalos regulares. En caso de que elimine accidentalmente la base de datos o un contenedor, puede presentar una incidencia de soporte técnico o llamar al servicio de soporte técnico de Azure para restaurar los datos a partir de copias de seguridad en línea automáticas. El soporte técnico de Azure está disponible solo para algunos planes, por ejemplo, Estándar o Desarrollador, y planes superiores. Para restaurar una instantánea específica de la copia de seguridad, Azure Cosmos DB requiere que los datos estén disponibles durante el ciclo de copia de seguridad de esa instantánea. 

Si usa Key Vault para almacenar las credenciales de las instancias de Cosmos DB, asegúrese de realizar copias de seguridad automatizadas de las claves.

Información sobre las copias de seguridad automatizadas de Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/online-backup-and-restore

Cómo restaurar datos en Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/how-to-backup-and-restore

Realización de copias de seguridad de claves de Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: En caso de que elimine accidentalmente la base de datos o un contenedor, puede presentar una incidencia de soporte técnico o llamar al servicio de soporte técnico de Azure para restaurar los datos a partir de copias de seguridad en línea automáticas. El soporte técnico de Azure está disponible solo para algunos planes, por ejemplo, Estándar o Desarrollador, y planes superiores. Para restaurar una instantánea específica de la copia de seguridad, Azure Cosmos DB requiere que los datos estén disponibles durante el ciclo de copia de seguridad de esa instantánea.

Pruebe la restauración de los secretos almacenados en Azure Key Vault con PowerShell. El cmdlet restore-AzureKeyVaultKey crea una clave en el almacén de claves especificado. Esta clave es una réplica de la clave de copia de seguridad en el archivo de entrada y tiene el mismo nombre que la clave original.

Información sobre las copias de seguridad automatizadas de Azure Cosmos DB:

https://docs.microsoft.com/azure/cosmos-db/online-backup-and-restore

Cómo restaurar datos en Azure Cosmos DB:

https://docs.microsoft.com/azure/cosmos-db/how-to-backup-and-restore

Cómo restaurar secretos de Azure Key Vault:

https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas del cliente

**Instrucciones**: Dato que todos los datos de usuario almacenados en Cosmos DB están cifrados en reposo y en tránsito, no es necesario hacer nada. Otra forma de decirlo es que el cifrado en reposo está "activado" de forma predeterminada. No hay ningún mando para activarlo o desactivarlo. Azure Cosmos DB usa el cifrado de AES-256 en todas las regiones donde se ejecuta la cuenta.

Habilite la eliminación temporal en Key Vault para proteger las claves contra la eliminación accidental o malintencionada.

Información sobre el cifrado de datos en Azure Cosmos DB: https://docs.microsoft.com/azure/cosmos-db/database-encryption-at-rest

Habilitación de la eliminación temporal en Key Vault: https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

## <a name="incident-response"></a>Respuesta a los incidentes

*Para obtener más información, consulte [Control de seguridad: respuesta ante incidentes](https://docs.microsoft.com/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

También puede utilizar la guía de control de incidentes de seguridad de equipos de NIST como referencia para la creación de su propio plan de respuesta a incidentes: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf

Configuración de las automatizaciones de flujos de trabajo en Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide

Guía para crear su propio proceso de respuesta a incidentes de seguridad: https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/

Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft: https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Guía**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

Consulte la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y capacidades de TI: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos del cliente.  Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

Cómo establecer el contacto de seguridad de Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: Exporte sus alertas y recomendaciones de Azure Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Sentinel.

Configuración de la exportación continua: https://docs.microsoft.com/azure/security-center/continuous-export

Transmisión de alertas a Azure Sentinel: https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Instrucciones**: use la característica de automatización del flujo de trabajo de Azure Security Center para desencadenar automáticamente respuestas mediante "Logic Apps" en las alertas y recomendaciones de seguridad.

Configuración de la automatización de flujo de trabajo y Logic Apps: https://docs.microsoft.com/azure/security-center/workflow-automation

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para obtener más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](https://docs.microsoft.com/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos en un plazo de 60 días

**Guía**: Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft: https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

Puede encontrar más información sobre la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft aquí: https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).
