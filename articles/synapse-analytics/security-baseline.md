---
title: Base de referencia de seguridad de Azure para Synapse Analytics
description: La línea de base de seguridad de Synapse Analytics proporciona instrucciones de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Bechmark.
author: msmbaldwin
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 07/22/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 2cd42d8d4ba1ecabc1b87942536e2ef9a9017b22
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501957"
---
# <a name="azure-security-baseline-for-synapse-analytics"></a>Base de referencia de seguridad de Azure para Synapse Analytics

La base de referencia de seguridad de Azure para Synapse Analytics contiene recomendaciones que le ayudarán a mejorar la estrategia de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](https://docs.microsoft.com/azure/security/benchmarks/overview), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para obtener más información, consulte [Introducción a las líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).

## <a name="network-security"></a>Seguridad de las redes

*Para más información, consulte [Control de seguridad: Seguridad de redes](/azure/security/benchmarks/security-control-network-security).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Instrucciones**: Proteja su instancia de Azure SQL Server en una red virtual a través de Private Link. Private Link permite el acceso a los servicios PaaS de Azure a través de un punto de conexión privado en la red virtual. El tráfico entre la red virtual y el servicio viaja por la red troncal de Microsoft.

Como alternativa, al conectarse al grupo de Synapse SQL, limite el ámbito de la conexión saliente a la base de datos SQL mediante un grupo de seguridad de red. Deshabilite todo el tráfico de los servicios de Azure a la base de datos SQL mediante el punto de conexión público, para lo que debe establecer Allow Azure Services (Permitir servicios de Azure) en OFF (Desactivado). Asegúrese de que no se permiten direcciones IP públicas en las reglas de firewall.

* [¿Qué es Azure Private Link?](https://docs.microsoft.com/azure/private-link/private-link-overview)

* [Descripción de Private Link para Azure Synapse SQL](https://docs.microsoft.com/azure/sql-database/sql-database-private-endpoint-overview)

* [Creación de una red virtual](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)

* [Creación de un NSG con una configuración de seguridad](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes e interfaces de red

**Guía**: Cuando se conecte al grupo de Azure Synapse SQL después de habilitar los registros de flujo del grupo de seguridad de red (NSG), envíe los registros a una cuenta de Azure Storage para la auditoría de tráfico.

También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

* [Habilitación de los registros de flujo de NSG](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal)

* [Descripción de la seguridad de red proporcionada por Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-network-recommendations)

* [Habilitación y uso del Análisis de tráfico](https://docs.microsoft.com/azure/network-watcher/traffic-analytics)

* [Descripción de la seguridad de red proporcionada por Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-network-recommendations)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Instrucciones**: no aplicable; esta recomendación está pensada para Azure App Service o recursos de proceso que hospedan aplicaciones web.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Denegación de las comunicaciones con direcciones IP malintencionadas conocidas

**Instrucciones**: Use Advanced Threat Protection (ATP) para Azure Synapse SQL. ATP detecta actividades anómalas que indican intentos inusuales y potencialmente peligrosos de acceder a bases de datos, o de vulnerar su seguridad, y puede desencadenar distintas alertas, como "Posible inyección de código SQL" y "Acceso desde una ubicación inusual". Puede acceder a la característica ATP, que forma parte de la oferta de Advanced Data Security (ADS), así como administrarla, a través del portal central de ADS de SQL.

Habilite DDoS Protection estándar en las redes virtuales asociadas a Azure Synapse SQL para protegerse de los ataques por denegación de servicio distribuidos. Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

* [Descripción de ATP para Azure Synapse SQL](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview)

* [Habilitación de Advanced Data Security para Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security)

* [Información general de ADS](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security)

* [Configuración de la protección contra DDoS](https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection)

* [Descripción de la inteligencia sobre amenazas integrada de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-alerts-data-services)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Guía**: Cuando se conecte al grupo de Azure Synapse SQL después de habilitar los registros de flujo del grupo de seguridad de red (NSG), envíe los registros a una cuenta de Azure Storage para la auditoría de tráfico. También puede enviar registros de flujo a un área de trabajo Log Analytics o transmitirlos a Event Hubs. Si es necesario para investigar actividades anómalas, habilite la captura de paquetes de Network Watcher.

* [Habilitación de los registros de flujo de NSG](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal)

* [Habilitación de Network Watcher](https://docs.microsoft.com/azure/network-watcher/network-watcher-create)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implementación de sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Instrucciones**: Use Advanced Threat Protection (ATP) para Azure Synapse SQL. ATP detecta actividades anómalas que indican intentos inusuales y potencialmente peligrosos de acceder a bases de datos, o de vulnerar su seguridad, y puede desencadenar distintas alertas, como "Posible inyección de código SQL" y "Acceso desde una ubicación inusual". Puede acceder a la característica ATP, que forma parte de la oferta de Advanced Data Security (ADS), así como administrarla, a través del portal central de ADS de SQL. ATP también integra alertas con Azure Security Center.

* [Descripción de ATP para Azure Synapse SQL](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Instrucciones**: no aplicable; esta recomendación está pensada para Azure App Service o recursos de proceso que hospedan aplicaciones web.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: Puede usar etiquetas de servicio de red virtual para definir controles de acceso a la red en los grupos de seguridad de red o Azure Firewall. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, ApiManagement) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

Cuando se usa un punto de conexión de servicio para el grupo de Azure Synapse SQL, se requiere una salida a las direcciones IP públicas de Azure SQL Database: los grupos de seguridad de red (NSG) deben estar abiertos en las direcciones IP de Azure SQL Database para permitir la conectividad. Puede hacerlo mediante el uso de etiquetas de servicio de grupos de seguridad de red para Azure SQL Database.

* [Descripción de las etiquetas de servicio con puntos de conexión de servicio para Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview#limitations)

* [Descripción y uso de etiquetas de servicio](https://docs.microsoft.com/azure/virtual-network/service-tags-overview)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Instrucciones**: Defina e implemente configuraciones de seguridad de red para recursos relacionados con su grupo de SQL con Azure Policy. Puede usar el espacio de nombres "Microsoft.Sql" para detallar definiciones de directiva personalizadas o usar cualquiera de las definiciones de directiva integradas diseñadas para la protección de red de Azure SQL Database o el servidor. Un ejemplo de una directiva de seguridad de red integrada aplicable para Azure SQL Database Server sería: "SQL Server debe usar un punto de conexión de servicio de red virtual".

Use Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como plantillas de Administración de recursos de Azure, control de acceso basado en rol (RBAC) y directivas, en una única definición de plano técnico. Aplique fácilmente el plano técnico a nuevas suscripciones y entornos, y ajuste el control y la administración mediante el control de versiones.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Creación de un plano técnico de Azure](https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Instrucciones**: use etiquetas para grupos de seguridad de red (NSG) y otros recursos relacionados con la seguridad de red y el flujo de tráfico. En el caso de las reglas de NSG individuales, use el campo "Descripción" para especificar las necesidades empresariales o la duración (etc.) de las reglas que permiten que entre o salga el tráfico en una red.

Use cualquiera de las definiciones de Azure Policy integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor", para asegurarse de que todos los recursos se creen con etiquetas y para notificarle los recursos no etiquetados existentes.

Puede usar Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas.

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Instrucciones**: Use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en los recursos de red relacionados con el grupo de Azure Synapse. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.

* [Visualización y recuperación de eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view)

* [Creación de alertas en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Control de seguridad: registro y supervisión](/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Instrucciones**: Microsoft mantiene los orígenes de hora de los recursos de Azure. Puede actualizar la sincronización de la hora para las implementaciones de proceso.

* [Configuración de la sincronización de hora de los recursos de proceso de Azure](https://docs.microsoft.com/azure/virtual-machines/windows/time-sync)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Se puede definir una directiva de auditoría para una base de datos específica o como directiva de servidor predeterminada en Azure (que hospeda Azure Synapse). Una directiva de servidor se aplica a todas las bases de datos recién creadas en el servidor.

Si la auditoría de servidor está habilitada, se aplica siempre a la base de datos. La base de datos se auditará, independientemente de la configuración de auditoría de la base de datos.

Al habilitar la auditoría, los resultados se pueden escribir en un registro de auditoría en la cuenta de Azure Storage, el área de trabajo de Log Analytics o Event Hubs.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

* [Configuración de la auditoría para los recursos de Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/auditing-overview#server-vs-database-level)

* [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Habilite la auditoría en el nivel de servidor de Azure SQL para el grupo de Synapse SQL y elija una ubicación de almacenamiento para los registros de auditoría (Azure Storage, Log Analytics o Event Hubs).

La auditoría se puede habilitar tanto en el nivel de base de datos como en el servidor. Se recomienda habilitarla solo en el nivel de servidor, a menos que sea necesario configurar un receptor de datos independiente o una retención para una base de datos específica.

* [Habilitación de la auditoría de Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-auditing)

* [Habilitación de la auditoría del servidor](https://docs.microsoft.com/azure/azure-sql/database/auditing-overview#setup-auditing)

* [Diferencias de la directiva de auditoría de nivel de servidor frente la de nivel de base de datos](https://docs.microsoft.com/azure/sql-database/sql-database-auditing#server-vs-database-level)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Instrucciones**: No aplicable; esta prueba comparativa está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: Al almacenar los registros relacionados con el grupo de Synapse SQL en una cuenta de Storage, el área de trabajo de Log Analytics o Event Hubs, establezca el período de retención de registro según la normativa de cumplimiento de su organización.

* [Administración del ciclo de vida de Azure Blob Storage](https://docs.microsoft.com/azure/storage/blobs/storage-lifecycle-management-concepts?tabs=azure-portal)

* [Configuración de parámetros de retención de registros de un área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period)

* [Captura de eventos de streaming en Event Hubs](https://docs.microsoft.com/azure/event-hubs/event-hubs-capture-overview)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Instrucciones**: analice y supervise los registros en busca de comportamientos anómalos y revise los resultados con regularidad. Use Advanced Threat Protection para Azure SQL Database junto con Azure Security Center para alertar sobre actividad inusual relacionada con su instancia de SQL Database. También puede configurar alertas basadas en valores de métricas o en entradas del registro de actividad de Azure relacionadas con las instancias de SQL Database.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

* [Descripción de Advanced Threat Protection y alertas de Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview)

* [Habilitación de Advanced Data Security para Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security)

* [Configuración de alertas personalizadas para Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-insights-alerts-portal?view=azps-1.4.0)

* [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: Use Advanced Threat Protection (ATP) para Azure SQL Database junto con de Azure Security Center para supervisar cualquier actividad anómala y alertar sobre ella. ATP forma parte de la oferta de seguridad de datos avanzada (ADS), y se puede acceder a esta y administrarla mediante la funcionalidad ADS central de SQL en el portal. ADS incluye una funcionalidad para detectar y clasificar datos confidenciales, buscar y mitigar los puntos vulnerables potenciales de una base de datos, así como para detectar actividades anómalas que puedan indicar una amenaza para su base de datos.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel.

* [Descripción de Advanced Threat Protection y alertas de Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview)

* [Habilitación de Advanced Data Security para Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security)

* [Administración de alertas de seguridad en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts)

* [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: No aplicable; en el caso de los recursos relacionados con el grupo de Synapse SQL, Microsoft administra la solución antimalware en la plataforma subyacente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: No aplicable; los recursos relacionados con el grupo de Synapse SQL no generan ningún registro DNS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Guía**: No aplicable; la auditoría de la línea de comandos no es aplicable a Azure Synapse SQL.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para más información, consulte [Control de seguridad: Identidad y control de acceso](/azure/security/benchmarks/security-control-identity-access-control).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Instrucciones**: Los usuarios se autentican con Azure Active Directory o con la autenticación de SQL.

Al crear la primera implementación de Azure SQL, hay que especificar un inicio de sesión de administrador y una contraseña asociada a ese inicio de sesión. Esta cuenta administrativa se denomina "administrador del servidor". Para saber cuáles son las cuentas de administrador de una base de datos, abra Azure Portal y vaya a la pestaña Propiedades de su servidor o instancia administrada. También puede configurar una cuenta de administrador de Azure AD con permisos administrativos completos; lo que es necesario si quiere habilitar la autenticación de Azure Active Directory.

En el caso de las operaciones de administración, use los roles integrados de Azure, que deben asignarse explícitamente. Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc a fin de detectar cuentas que son miembros de grupos administrativos.

* [Autenticación de Azure SQL Database](https://docs.microsoft.com/azure/azure-sql/database/security-overview#authentication)

* [Creación de cuentas para usuarios que no son administradores](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#create-accounts-for-non-administrator-users)

* [Uso de una cuenta de Azure Active Directory para la autenticación](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#create-additional-logins-and-users-having-administrative-permissions)

* [Obtención de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

* [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

* [Cómo administrar inicios de sesión y cuentas de administrador existentes en Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#existing-logins-and-user-accounts-after-creating-a-new-database)

* [Roles integrados en los recursos de Azure](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Instrucciones**: Azure Active Directory no tiene el concepto de contraseñas predeterminadas. Al aprovisionar un grupo de Azure Synapse SQL, se recomienda optar por integrar la autenticación con Azure Active Directory. Con este método de autenticación, el usuario envía un nombre de cuenta de usuario y solicita que el servicio use la información de credenciales que hay almacenada en Azure Active Directory (Azure AD).

* [Configuración y administración de la autenticación de Azure Active Directory con Azure SQL](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure?tabs=azure-powershell#active-directory-password-authentication)

* [Descripción de la autenticación en Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#existing-logins-and-user-accounts-after-creating-a-new-database)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Instrucciones**: cree directivas y procedimientos en torno al uso de cuentas administrativas dedicadas. Use la Administración de identidades y acceso de Azure Security Center para supervisar el número de cuentas administrativas en las que se inicia sesión a través de Azure Active Directory.

Para saber cuáles son las cuentas de administrador de una base de datos, abra Azure Portal y vaya a la pestaña Propiedades de su servidor o instancia administrada.

* [Descripción de identidad y acceso en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

* [Cómo administrar inicios de sesión y cuentas de administrador existentes en Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#existing-logins-and-user-accounts-after-creating-a-new-database)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4: Uso del inicio de sesión único (SSO) de Azure Active Directory

**Guía**: Use un registro de aplicaciones de Azure (entidad de servicio) para recuperar un token que se pueda usar para interactuar con su almacén de datos en el plano de control (Azure Portal) a través de llamadas API.

* [Llamada a las API REST de Azure](https://docs.microsoft.com/rest/api/azure/#how-to-call-azure-rest-apis-with-postman)

* [Registro de la aplicación cliente (entidad de servicio) con Azure AD](https://docs.microsoft.com/rest/api/azure/#register-your-client-application-with-azure-ad)

* [Información de la API de REST de Azure Synapse SQL](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-manage-compute-rest-api)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Uso de la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Habilite Multi-Factor Authentication (MFA) de Azure Active Directory (AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

* [Procedimiento para habilitar la MFA en Azure](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted)

* [Supervisión de la identidad y el acceso en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

* [Descripción de MFA en Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/authentication-mfa-ssms-overview)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3.6: Uso de estaciones de trabajo seguras y administradas por Azure para realizar tareas administrativas

**Instrucciones**: Utilice una estación de acceso con privilegios (PAW) con Multi-Factor Authentication (MFA) configurada para iniciar sesión en recursos de Azure y configurarlos.

* [Más información sobre las estaciones de trabajo con privilegios de acceso](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Procedimiento para habilitar la MFA en Azure](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: Use los informes de seguridad de Azure Active Directory para la generación de registros y alertas cuando se producen actividades sospechosas o no seguras en el entorno.

Use Advanced Threat Protection para Azure SQL Database junto con Azure Security Center para la detección y alerta de actividades anómalas que indican intentos inusuales y potencialmente dañinos de acceso o ataque a las bases de datos.

La auditoría de SQL Server le permite crear auditorías de servidor, que pueden contener especificaciones de auditoría de servidor para los eventos del nivel de servidor así como especificaciones de auditoría de base de datos para los eventos del nivel de base de datos. Los eventos auditados se pueden escribir en los registros de eventos o en los archivos de auditoría.

* [Procedimiento para identificar usuarios de Azure AD marcados por una actividad de riesgo](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-user-at-risk)

* [Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

* [Revisión de Advanced Threat Protection y posibles alertas](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview#advanced-threat-protection-alerts)

* [Descripción de inicios de sesión y cuentas de usuario en Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage)

* [Descripción de la auditoría de SQL Server](https://docs.microsoft.com/sql/relational-databases/security/auditing/sql-server-audit-database-engine?view=sql-server-ver15)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Instrucciones**: use ubicaciones con nombre de acceso condicional para permitir al portal y a la Administración de recursos de Azure el acceso solo desde agrupaciones lógicas de intervalos de direcciones IP o países o regiones específicos.

* [Configuración de ubicaciones con nombre en Azure](https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Instrucciones**: Cree un administrador de Azure Active Directory (AD) para el servidor de Azure SQL Database en su grupo de Synapse SQL.

* [Configuración y administración de la autenticación de Azure AD con Azure SQL](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure)

* [Procedimiento para crear y configurar una instancia de Azure AD](https://docs.microsoft.com/azure/active-directory-domain-services/tutorial-create-instance)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Instrucciones**: Azure Active Directory proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de Azure Active Directory para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo los usuarios adecuados tengan acceso continuado.

Si se usa la autenticación de SQL, cree usuarios de base de datos independientes en la base de datos. Asegúrese de asignar a uno o varios usuarios de base de datos un rol de base de datos personalizado con permisos específicos adecuados para ese grupo de usuarios.

* [Uso de las revisiones de acceso](https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview)

* [Descripción de inicios de sesión y cuentas de usuario en Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: Configure la autenticación de Azure Active Directory (AD) con Azure SQL y habilite la configuración de diagnóstico para las cuentas de usuario de Azure Active Directory, para lo cual debe enviar los registros de auditoría e inicio de sesión a un área de trabajo de Log Analytics. Configure las alertas deseadas en el área de trabajo de Log Analytics.

Si se usa la autenticación de SQL, cree usuarios de base de datos independientes en la base de datos. Asegúrese de asignar a uno o varios usuarios de base de datos un rol de base de datos personalizado con permisos específicos adecuados para ese grupo de usuarios.

* [Uso de las revisiones de acceso](https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview)

* [Configuración y administración de la autenticación de Azure AD con Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure)

* [Integración de los registros de actividad de Azure en Azure Monitor](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

* [Descripción de inicios de sesión y cuentas de usuario en Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12: Alerta de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Instrucciones**: Use las características de protección de identidad y detección de riesgo de Azure Active Directory (Azure AD) para configurar respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. Además, puede incorporar e ingerir datos en Azure Sentinel para investigarlos más a fondo.

Si se usa la autenticación de SQL, cree usuarios de base de datos independientes en la base de datos. Asegúrese de asignar a uno o varios usuarios de base de datos un rol de base de datos personalizado con permisos específicos adecuados para ese grupo de usuarios.

* [Visualización de los inicios de sesión de riesgo de Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins)

* [Configuración y habilitación de las directivas de riesgo de protección de identidad](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies)

* [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/connect-data-sources)

* [Descripción de inicios de sesión y cuentas de usuario en Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: En escenarios de soporte técnico en los que Microsoft necesita acceder a datos relacionados con Azure SQL Database de su grupo de Synapse SQL, la característica Caja de seguridad del cliente de Azure proporciona una interfaz para que pueda revisar y aprobar o rechazar las solicitudes de acceso a los datos del cliente.

* [Descripción de la Caja de seguridad del cliente](https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="data-protection"></a>Protección de los datos

*Para más información, consulte [Control de seguridad: Protección de datos](/azure/security/benchmarks/security-control-data-protection).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: use etiquetas para ayudar a realizar el seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

La clasificación y detección de datos está integrada en Azure Synapse SQL. Ofrece funcionalidades avanzadas para detectar, clasificar, etiquetar e informar de los datos confidenciales de las bases de datos.

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

* [Descripción de la clasificación y la detección de datos](https://docs.microsoft.com/azure/azure-sql/database/data-discovery-and-classification-overview)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Los recursos deben separarse por red virtual o subred, etiquetarse adecuadamente y protegerse en un grupo de seguridad de red o Azure Firewall. Los recursos que almacenan o procesan datos confidenciales deben estar aislados. Use Private Link; implemente Azure SQL Server en una red virtual y conéctese de forma segura mediante Private Link.

* [Creación de suscripciones adicionales de Azure](https://docs.microsoft.com/azure/billing/billing-create-subscription)

* [Creación de grupos de administración](https://docs.microsoft.com/azure/governance/management-groups/create)

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

* [Configuración de Private Link para Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-private-endpoint-overview#how-to-set-up-private-link-for-azure-sql-database)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: En el caso de cualquier instancia de Azure SQL Database de su grupo de Synapse SQL que almacene o procese información confidencial, marque la base de datos y los recursos relacionados como confidenciales mediante etiquetas. Configure Private Link junto con las etiquetas de servicio del grupo de seguridad de red (NSG) en las instancias de Azure SQL Database para evitar la filtración de información confidencial.

Además, Advanced Threat Protection para Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse detecta actividades anómalas que indican intentos inusuales y potencialmente peligrosos de acceder a las bases de datos o de vulnerar su seguridad.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

* [Configuración de Private Link y los grupos de seguridad de red con el fin de evitar el filtrado de datos en las instancias de Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-private-endpoint-overview)

* [Configuración de Advanced Threat Protection para Azure SQL Database](https://docs.microsoft.com/azure/azure-sql/database/threat-detection-overview)

* [Descripción de la protección de datos de los clientes en Azure](https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Instrucciones**: Azure SQL Database protege los datos mediante el cifrado de datos en movimiento con Seguridad de la capa de transporte. SQL Server aplica el cifrado (SSL/TLS) en todo momento para todas las conexiones. Esto garantiza que todos los datos se cifran "en tránsito" entre el cliente y el servidor independientemente de la configuración de Encrypt o TrustServerCertificate en la cadena de conexión.

* [Descripción del cifrado de Azure SQL en tránsito](https://docs.microsoft.com/azure/sql-database/sql-database-security-overview#information-protection-and-encryption)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: Use la característica de clasificación y detección de datos de Azure Synapse SQL. La clasificación y detección de datos proporciona funcionalidades avanzadas integradas en Azure SQL Database para detectar, clasificar, etiquetar y proteger los datos confidenciales de las bases de datos.

La clasificación y detección de datos forma parte de la oferta de Advanced Data Security. Dicha oferta es un paquete unificado de funcionalidades avanzadas de seguridad de SQL. Se puede acceder a la clasificación y detección de datos, así como administrarla, desde el portal de ADS de SQL.

Además, puede configurar una directiva de enmascaramiento dinámico de datos (DDM) en Azure Portal. El motor de recomendaciones de DDM marca determinados campos de la base de datos como campos potencialmente confidenciales, que pueden ser buenos candidatos para el enmascaramiento.

* [Uso de la detección y la clasificación de datos para Azure SQL Server](https://docs.microsoft.com/azure/sql-database/sql-database-data-discovery-and-classification)

* [Descripción del enmascaramiento dinámico de datos para Azure Synapse SQL](https://docs.microsoft.com/azure/sql-database/sql-database-dynamic-data-masking-get-started)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6: Uso del control de acceso basado en rol para controlar el acceso a los recursos

**Guía**: Use el control de acceso basado en roles (Azure RBAC) de Azure para administrar el acceso a las instancias de Azure SQL Database en el grupo de Synapse SQL.

La autorización se controla por medio de las pertenencias a roles y los permisos de nivel de objeto de la base de datos de la cuenta de usuario. Como procedimiento recomendado, debe conceder a los usuarios los privilegios mínimos necesarios.

* [Integración de Azure SQL Server con Azure Active Directory para la autenticación](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication)

* [Control del acceso en Azure SQL Server](https://docs.microsoft.com/azure/sql-database/sql-database-control-access)

* [Descripción de la autorización y la autenticación en Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: No aplicable; Microsoft administra la infraestructura subyacente de Azure Synapse SQL y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

* [Descripción de la protección de datos de los clientes en Azure](https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Instrucciones**: El Cifrado de datos transparente (TDE) ayuda a proteger Azure Synapse SQL frente a amenazas de actividad sin conexión malintencionada mediante el cifrado de datos en reposo. También realiza cifrado y descifrado de la base de datos en tiempo real, copias de seguridad asociadas y archivos de registro de transacciones en reposo sin necesidad de efectuar cambios en la aplicación. En Azure, la configuración predeterminada de TDE es que la clave de cifrado está protegida mediante un certificado de servidor integrado. Asimismo, puede usar el cifrado TDE administrado por el cliente, que también se conoce como compatibilidad de Bring Your Own Key (BYOK) con TDE. En este escenario, el protector de TDE que cifra la clave de cifrado es una clave asimétrica administrada por el cliente, que se almacena en una instancia de Azure Key Vault que es propiedad del cliente y que este administra (un sistema de administración de claves externas basado en la nube de Azure), y que nunca sale del almacén de claves.

* [Descripción del cifrado de datos transparente administrado por el servicio](https://docs.microsoft.com/azure/azure-sql/database/transparent-data-encryption-tde-overview?tabs=azure-portal)

* [Descripción del cifrado de datos transparente administrado por el cliente](https://docs.microsoft.com/azure/azure-sql/database/transparent-data-encryption-tde-overview?tabs=azure-portal#customer-managed-transparent-data-encryption---bring-your-own-key)

* [Cómo activar TDE con su propia clave](https://docs.microsoft.com/azure/azure-sql/database/transparent-data-encryption-byok-configure)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure para crear alertas para cuando se produzcan cambios en las instancias de producción de grupos de Synapse SQL y otros recursos críticos o relacionados.

Además, puede configurar alertas para las bases de datos de su grupo de Synapse SQL mediante Azure Portal. Las alertas pueden enviarle un correo electrónico o llamar a un webhook cuando alguna métrica (por ejemplo, el tamaño de la base de datos o el uso de la CPU) alcanza el umbral.

* [Creación de alertas para los eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log)

* [Creación de alertas para Azure Synapse SQL](https://docs.microsoft.com/azure/azure-sql/database/alerts-insights-configure-portal)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Control de seguridad: Administración de vulnerabilidades](/azure/security/benchmarks/security-control-vulnerability-management).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Instrucciones**: Habilite Advanced Data Security para y siga las recomendaciones de Azure Security Center sobre cómo realizar evaluaciones de vulnerabilidades en sus instancias de Azure SQL Database.

* [Ejecución de evaluaciones de vulnerabilidades en las instancias de Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-vulnerability-assessment)

* [Habilitación de Advanced Data Security](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security)

* [Implementación de las recomendaciones de evaluación de vulnerabilidades de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-vulnerability-assessment-recommendations)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3: Implementación de una solución de administración de revisiones automatizada de títulos de software de terceros

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Guía**: La evaluación de vulnerabilidades es un servicio de detección integrado en Azure Synapse SQL. Este servicio emplea una base de conocimiento de reglas que marcan las vulnerabilidades de seguridad. Se resaltan las desviaciones de los procedimientos recomendados, como configuraciones inadecuadas, permisos excesivos y datos confidenciales desprotegidos. Puede acceder a la evaluación de vulnerabilidades y administrarla a través del portal central de Advanced Data Security (ADS) de SQL.

* [Administración y exportación de exámenes de evaluación de vulnerabilidades en el portal de ADS de SQL](https://docs.microsoft.com/azure/sql-database/sql-vulnerability-assessment)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Instrucciones**: Use la clasificación de riesgo predeterminada (puntuación de seguridad) proporcionada por Azure Security Center.

La clasificación y detección de datos está integrada en Azure Synapse SQL. Ofrece funcionalidades avanzadas para detectar, clasificar, etiquetar e informar de los datos confidenciales de las bases de datos.

* [Descripción de la puntuación de seguridad de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-secure-score)

* [Descripción de la clasificación y la detección de datos](https://docs.microsoft.com/azure/azure-sql/database/data-discovery-and-classification-overview)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para más información, consulte [Control de seguridad: Administración de recursos y del inventario](/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: Use Azure Resource Graph para consultar y detectar todos los recursos relacionados con su grupo de Synapse SQL dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Azure Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager que figuran a continuación.

* [Creación de consultas con Azure Resource Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)

* [Visualización de las suscripciones de Azure](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

* [Descripción de Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Instrucciones**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Instrucciones**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

* [Creación de suscripciones adicionales de Azure](https://docs.microsoft.com/azure/billing/billing-create-subscription)

* [Creación de grupos de administración](https://docs.microsoft.com/azure/governance/management-groups/create)

* [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: Defina una lista de recursos de Azure aprobados relacionados con su grupo de Synapse SQL.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Use Azure Resource Graph para consultar o detectar recursos en las suscripciones. Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Creación de consultas con Azure Resource Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Instrucciones**: use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Use Azure Resource Graph para consultar o detectar recursos dentro de las suscripciones. Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Denegación de un tipo de recurso específico con Azure Policy](https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10: Mantenimiento de un inventario de títulos de software aprobados

**Guía**: no aplicable; esta recomendación está destinada a las aplicaciones que se ejecutan en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager mediante la configuración de la opción "Bloquear acceso" en la aplicación "Administración de Microsoft Azure".

* [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Guía**: Cualquier recurso relacionado con el grupo de Synapse SQL que se requiera para las operaciones empresariales, pero que pueda suponer un riesgo mayor para la organización, debe aislarse en su propia máquina virtual o red virtual y estar lo suficientemente protegido con Azure Firewall o un grupo de seguridad de red.

* [Creación de una red virtual](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)

* [Creación de un grupo de seguridad de red con una configuración de seguridad](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Control de seguridad: Configuración segura](/azure/security/benchmarks/security-control-secure-configuration).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Use alias de Azure Policy en el espacio de nombres "Microsoft.Sql" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de recursos relacionada con el grupo de Synapse SQL. También puede usar definiciones de directiva integradas en servidores o bases de datos de Azure, como:
- Implementación de la detección de amenazas en servidores SQL Server.
- SQL Server debe usar un punto de conexión del servicio de red virtual

* [Visualización de los alias de Azure Policy disponibles](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Use la directiva de Azure Policy [deny] y [deploy if not exist] para aplicar la configuración segura en los recursos de Azure.

* [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Descripción de los efectos de Azure Policy](https://docs.microsoft.com/azure/governance/policy/concepts/effects)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Si usa definiciones de personalizadas de Azure Policy, use Azure DevOps o Azure Repos para almacenar y administrar el código de forma segura.

* [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops)

* [Documentación de Azure Repos](https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: No aplicable; Azure Synapse SQL no tiene opciones de configuración de seguridad configurables.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8: Implementación de herramientas de administración de la configuración para sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: Use Azure Security Center para realizar exámenes de base de referencia de los recursos relacionados con su grupo de Synapse SQL.

* [Corrección de recomendaciones en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-sql-service-recommendations)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: El Cifrado de datos transparente (TDE) con claves administradas por el cliente en Azure Key Vault permite cifrar la clave de cifrado de base de datos (DEK) generada automáticamente con una clave asimétrica administrada por el cliente denominada protector de TDE. Esto también se conoce como compatibilidad de Bring Your Own Key (BYOK) para Cifrado de datos transparente. En el escenario BYOK, el protector de TDE se almacena en una instancia de Azure Key Vault administrada por el cliente, y de la que este es propietario. Asimismo, asegúrese de que la eliminación temporal está habilitada en Azure Key Vault.

* [Habilitación del cifrado TDE con una clave administrada por el cliente de Azure Key Vault](https://docs.microsoft.com/azure/azure-sql/database/transparent-data-encryption-byok-configure?tabs=azure-powershell)

* [Habilitación de la eliminación temporal en Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-powershell)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Instrucciones**: Use las identidades administradas para proporcionar a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory (AD). Identidades administradas le permite autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluido Azure Key Vault, sin necesidad de credenciales en el código.

* [Tutorial: Uso de una identidad administrada asignada por el sistema de una VM Windows para acceder a Azure SQL](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-sql)

* [Configuración de las identidades administradas](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: implemente Credential Scanner para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

* [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Control de seguridad: defensa contra malware](/azure/security/benchmarks/security-control-malware-defense).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Uso de software antimalware administrado centralmente

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso. Microsoft controla el antimalware de la plataforma subyacente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (como Azure Synapse SQL), pero no se ejecuta en el contenido del cliente.

Examine previamente el contenido que se carga en recursos de Azure que no son de proceso, como App Service, Data Lake Storage, Blob Storage, Azure SQL Server, etc. Microsoft no tiene acceso a los datos de estas instancias.

* [Descripción de Microsoft Antimalware para Azure Cloud Services y Virtual Machines](https://docs.microsoft.com/azure/security/fundamentals/antimalware)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso. Microsoft controla el antimalware de la plataforma subyacente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Control de seguridad: recuperación de datos](/azure/security/benchmarks/security-control-data-recovery).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Instrucciones**: Las instantáneas del grupo de Synapse SQL se toman a lo largo del día y crean puntos de restauración que están disponibles durante siete días. No se puede cambiar este período de retención. El grupo de SQL admite un objetivo de punto de recuperación (RPO) de ocho horas. Puede restaurar el almacenamiento de datos de la región primaria a partir de cualquiera de las instantáneas capturadas en los últimos siete días. Tenga en cuenta que también puede desencadenar manualmente las instantáneas si es necesario.

* [Copia de seguridad y restauración en el grupo de Azure Synapse SQL](/azure/synapse-analytics/sql-data-warehouse/backup-and-restore)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copias de seguridad de las claves administradas por el cliente

**Guía**: Las instantáneas del almacenamiento de datos se toman automáticamente a lo largo del día y crean puntos de restauración que están disponibles durante siete días. No se puede cambiar este período de retención. El grupo de SQL admite un objetivo de punto de recuperación (RPO) de ocho horas. Puede restaurar el almacenamiento de datos de la región primaria a partir de cualquiera de las instantáneas capturadas en los últimos siete días. Tenga en cuenta que también puede desencadenar manualmente las instantáneas si es necesario.

Si usa una clave administrada por el cliente para cifrar la clave de cifrado de base de datos, asegúrese de que se está realizando una copia de seguridad de la clave.

* [Copia de seguridad y restauración en el grupo de Azure Synapse SQL](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/backup-and-restore)

* [Realización de copias de seguridad de claves de Azure Key Vault](https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: Pruebe periódicamente los puntos de restauración para asegurarse de que las instantáneas son válidas. Para restaurar un grupo de SQL existente a partir de un punto de restauración, puede usar Azure Portal o PowerShell. Pruebe la restauración de la copia de seguridad de las claves administradas por el cliente.

* [Cómo restaurar claves de Azure Key Vault](https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

* [Copia de seguridad y restauración en el grupo de Azure Synapse SQL](https://docs.microsoft.coms/azure/synapse-analytics/sql-data-warehouse/backup-and-restore)

* [Cómo restaurar un grupo de SQL existente](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-restore-active-paused-dw)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: En Azure SQL Database, puede configurar una base de datos única o agrupada con una directiva de retención de copias de seguridad a largo plazo (LTR) para retener automáticamente las copias de seguridad de las bases de datos en contenedores de Azure Blob Storage hasta 10 años. Los datos de Azure Storage se cifran y descifran de forma transparente mediante el cifrado AES de 256 bits, uno de los cifrados de bloques más sólidos que hay disponibles, y son compatibles con FIPS 140-2.

De manera predeterminada, los datos de una cuenta de almacenamiento se cifran con claves administradas por Microsoft. Puede confiar en las claves administradas por Microsoft para el cifrado de los datos, o puede administrar el cifrado con sus propias claves. Si administra sus propias claves con Key Vault, asegúrese de que la eliminación temporal está habilitada.

* [Administración de la retención de copias de seguridad a largo plazo de Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-long-term-backup-retention-configure)

* [Cifrado de Azure Storage para datos en reposo](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)

* [Habilitación de la eliminación temporal en Key Vault](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Control de seguridad: Respuesta a los incidentes](/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: asegúrese de que haya planes de respuesta ante incidentes escritos que definan los roles del personal, así como las fases de administración y gestión de los incidentes.

* [Configuración de las automatizaciones de flujos de trabajo en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a las alertas, que le ayudan a priorizar el orden en el que asiste a cada alerta, de modo que cuando un recurso está en peligro, puede obtenerlo inmediatamente. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

* [Alertas de seguridad en el Centro de seguridad de Azure](https://docs.microsoft.com/azure/security-center/security-center-alerts-overview)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

* [Puede consultar la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Instrucciones**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos.

* [Establecimiento del contacto de seguridad de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: Exporte sus alertas y recomendaciones de Azure Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Sentinel.

* [Configuración de la exportación continua](https://docs.microsoft.com/azure/security-center/continuous-export)

* [Transmisión de alertas a Azure Sentinel](https://docs.microsoft.com/azure/sentinel/connect-azure-security-center)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Instrucciones**: use la característica de automatización del flujo de trabajo de Azure Security Center para desencadenar automáticamente respuestas mediante "Logic Apps" en las alertas y recomendaciones de seguridad.

* [Configuración de la automatización de flujo de trabajo y Logic Apps](https://docs.microsoft.com/azure/security-center/workflow-automation)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: * [Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.).

* [Aquí encontrará más información sobre la estrategia y puesta en marcha de un equipo rojo de Microsoft, y las pruebas de penetración en sitios activos de la infraestructura en la nube, las aplicaciones y los servicios administrados por Microsoft](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e).

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](/azure/security/benchmarks/security-baselines-overview).
