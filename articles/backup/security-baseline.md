---
title: Línea de base de seguridad de Azure para Backup
description: Línea de base de seguridad de Azure para Backup
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/23/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: d0a6b6561b5d0995d7794993608a4991b1da2718
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87810146"
---
# <a name="azure-security-baseline-for-backup"></a>Línea de base de seguridad de Azure para Backup

La línea de base de seguridad de Azure para Backup contiene recomendaciones que le ayudarán a mejorar la postura de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview.md), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para obtener más información, consulte [Introducción a las líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Seguridad de redes

*Para obtener más información, consulte [Control de seguridad: seguridad de red](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1: Proteja los recursos mediante grupos de seguridad de red o Azure Firewall en su red virtual

**Guía**: No aplicable; no se puede asociar una red virtual, una subred o un grupo de seguridad de red a un almacén de Recovery Services. Al realizar una copia de seguridad de una máquina virtual de Azure, los datos se transfieren a través de la red troncal de Azure. Cuando se realiza una copia de seguridad desde un equipo local, se crea un túnel cifrado con un punto de conexión específico en Azure y se usan las credenciales para cifrar previamente los datos y enviarlos luego a través del túnel cifrado.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2: Supervise y registre la configuración y el tráfico de redes virtuales, subredes y NIC

**Guía**: No aplicable; no se puede asociar una red virtual, una subred o un grupo de seguridad de red a un almacén de Recovery Services. Al realizar una copia de seguridad de una máquina virtual de Azure, los datos se transfieren a través de la red troncal de Azure. Cuando se realiza una copia de seguridad desde un equipo local, se crea un túnel cifrado con un punto de conexión específico en Azure y se usan las credenciales para cifrar previamente los datos y enviarlos luego a través del túnel cifrado.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: Microsoft administra todos los puntos de conexión utilizados por Azure Backup (incluido el agente de Microsoft Azure Recovery Services). El usuario es el responsable de los controles adicionales que desee implementar en sus sistemas locales.

- [Descripción de la compatibilidad de redes y acceso para el agente de MARS](./backup-support-matrix-mars-agent.md#networking-and-access-support)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="15-record-network-packets-and-flow-logs"></a>1.5: Registre los paquetes de red y registros de flujo

**Guía**: No aplicable; no se puede asociar una red virtual, una subred o un grupo de seguridad de red a un almacén de Recovery Services. Al realizar una copia de seguridad de una máquina virtual de Azure, los datos se transfieren a través de la red troncal de Azure. Cuando se realiza una copia de seguridad desde un equipo local, se crea un túnel cifrado con un punto de conexión específico en Azure y se usan las credenciales para cifrar previamente los datos y enviarlos luego a través del túnel cifrado.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implemente sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Guía**: Microsoft administra todos los puntos de conexión utilizados por Azure Backup (incluido el agente de Microsoft Azure Recovery Services). El usuario es el responsable de los controles adicionales que desee implementar en sus sistemas locales.

- [Descripción de la compatibilidad de redes y acceso para el agente de MARS](./backup-support-matrix-mars-agent.md#networking-and-access-support)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: Si usa el agente de MARS en una máquina virtual de Azure, use la etiqueta de servicio AzureBackup en su grupo de seguridad de red o Azure Firewall para permitir el acceso de salida a Azure Backup.

- [Copia de seguridad de bases de datos de SQL Server en máquinas virtuales de Azure](./backup-sql-server-database-azure-vms.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: No aplicable; Microsoft administra todos los puntos de conexión utilizados por Azure Backup (incluido el agente de Microsoft Azure Recovery Services).

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Instrucciones**: Si usa el agente de MARS en una máquina virtual de Azure, asocie esa máquina virtual a un grupo de seguridad de red y use la descripción para especificar la necesidad empresarial de la regla.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: Si usa el agente de MARS en una máquina virtual de Azure que se protege mediante un grupo de seguridad de red o Azure Firewall, use el registro de actividad de Azure para supervisar la configuración del grupo de seguridad de red o el firewall. Puede crear alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos críticos.

- [Visualización y recuperación de eventos del registro de actividad de Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Crear, ver y administrar las alertas del registro de actividad mediante Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para obtener más información, consulte [Control de seguridad: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: No aplicable; Microsoft mantiene el origen de la hora usado en los recursos de Azure, como Azure Backup, para las marcas de tiempo de los registros.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Para el registro de la auditoria del plano de control, habilite la configuración de diagnóstico del registro de actividad de Azure y envíela a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para su archivo. Con los datos del registro de actividad de Azure, puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en el nivel del plano de control para los recursos de Azure.

Además, ingiera registros a través de Azure Monitor para agregar datos de seguridad generados por Azure Backup. En Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y utilice cuentas para el almacenamiento de archivos a largo plazo. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

- [Cómo habilitar la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/activity-log.md)

- [Uso de la configuración de diagnósticos para almacenes de Recovery Services](./backup-azure-diagnostic-events.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Para el registro de la auditoria del plano de control, habilite la configuración de diagnóstico del registro de actividad de Azure y envíela a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para su archivo. Con los datos del registro de actividad de Azure, puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en el nivel del plano de control para los recursos de Azure.

Además, Azure Backup envía eventos de diagnóstico que se pueden recopilar y usar para elaborar análisis, alertas e informes. Puede usar Azure Portal para configurar los valores de diagnóstico de los almacenes de Recovery Services. Puede enviar uno o varios eventos de diagnóstico a una cuenta de almacenamiento, un centro de eventos o un área de trabajo de Log Analytics.

- [Cómo habilitar la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/activity-log.md)

- [Uso de la configuración de diagnósticos para almacenes de Recovery Services](./backup-azure-diagnostic-events.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención de registro para las áreas de trabajo de Log Analytics asociadas a los almacenes de Azure Recovery Services según las normativas de cumplimiento de su organización.

- [Cómo establecer parámetros de retención de registros](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Guía**: Azure Backup proporciona funcionalidades de supervisión y alerta integradas en un almacén de Recovery Services. Estas funcionalidades están disponibles sin ninguna infraestructura de administración adicional. También puede aumentar la escala de la supervisión y los informes mediante Azure Monitor.

Habilite la configuración de diagnóstico en el registro de actividad de Azure y envíe los registros a un área de trabajo de Log Analytics. Realice consultas en Log Analytics para buscar términos, identificar tendencias, analizar patrones y proporcionar muchas otras conclusiones basadas en los datos del registro de actividad que se pueden recopilar para almacenes de Recovery Services.

- [Supervisión de cargas de trabajo de Azure Backup](./backup-azure-monitoring-built-in-monitor.md)

- [Cómo habilitar la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/activity-log.md)

- [Recopilación y análisis de registros de actividad de Azure en un área de trabajo de Log Analytics en Azure Monitor](../azure-monitor/platform/activity-log.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: Azure Backup proporciona funcionalidades de supervisión y alerta integradas en un almacén de Recovery Services. Estas funcionalidades están disponibles sin ninguna infraestructura de administración adicional. También puede aumentar la escala de la supervisión y los informes mediante Azure Monitor.

Las alertas son, fundamentalmente, los escenarios donde los usuarios reciben notificaciones para que puedan tomar las medidas oportunas. La sección Alertas de copia de seguridad muestra las alertas generadas por el servicio de Azure Backup. Estas alertas las define el servicio; es decir, el usuario no puede crear ninguna alerta personalizada.

También puede incorporar el área de trabajo de Log Analytics a Azure Sentinel, ya que proporciona una solución de respuesta automatizada de orquestación de seguridad (SOAR). Esto permite crear cuadernos de estrategias (soluciones automatizadas) y usarlos para corregir problemas de seguridad. Además, puede crear alertas de registro personalizadas en el área de trabajo Log Analytics mediante Azure Monitor.

- [Supervisión de cargas de trabajo de Azure Backup](./backup-azure-monitoring-built-in-monitor.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Creación, visualización y administración de alertas de registro mediante Azure Monitor](../azure-monitor/platform/alerts-log.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: No aplicable; Azure Backup no procesa ni genera registros relacionados con antimalware.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: No aplicable; Azure Backup no procesa ni genera registros relacionados con DNS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para obtener más información, consulte [Control de seguridad: identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Instrucciones**: Azure Active Directory (AD) tiene roles integrados que se deben asignar explícitamente y son consultables. Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc para detectar cuentas que son miembros de grupos administrativos.

Documentación complementaria:

- [Obtención de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: Azure AD no tiene el concepto de contraseñas predeterminadas. Otros recursos de Azure que requieren una contraseña obligan a crear una contraseña con requisitos de complejidad y una longitud mínima, que cambia en función del servicio. Usted es responsable de las aplicaciones de terceros y de los servicios de Marketplace que pueden usar contraseñas predeterminadas.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

Además, para ayudarle a realizar un seguimiento de las cuentas administrativas dedicadas, puede seguir las recomendaciones de Azure Security Center o las directivas integradas de Azure Policy, por ejemplo: Debería haber más de un propietario asignado a la suscripción. Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción. Las cuentas externas con permisos de propietario deben quitarse de la suscripción.

- [Uso de Azure Security Center para supervisar la identidad y el acceso (versión preliminar)](../security-center/security-center-identity-access.md)

- [Uso de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Guía**: Use un registro de aplicaciones de Azure (entidad de servicio) para recuperar un token que se pueda usar para interactuar con los almacenes de Recovery Services a través de llamadas API.

- [Llamada a las API REST de Azure](/rest/api/azure/#how-to-call-azure-rest-apis-with-postman)

- [Registro de la aplicación cliente (entidad de servicio) con Azure AD](/rest/api/azure/#register-your-client-application-with-azure-ad)

- [Información de API de Azure Recovery Services](/rest/api/recoveryservices/)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Instrucciones**: Al realizar operaciones críticas en Azure Backup, debe especificar un PIN de seguridad, disponible en Azure Portal. Al habilitar Azure Multi-Factor Authentication, se agrega una capa de seguridad. Solo los usuarios autorizados con credenciales de Azure válidas y autenticados desde un segundo dispositivo pueden tener acceso a Azure Portal.

- [Multi-Factor Authentication en Azure Backup](./backup-azure-security-feature.md)

- [Planeamiento de una implementación de Azure Multi-Factor Authentication basada en la nube](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Instrucciones**: Utilice estaciones de acceso con privilegios (PAW) con Azure Multi-Factor Authentication (MFA) configurada para iniciar sesión en recursos habilitados para Azure Backup y configurarlos.

- [Uso de estaciones de trabajo con privilegios de acceso](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Planeamiento de una implementación de Azure Multi-Factor Authentication basada en la nube](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7: Registre y alerte de la actividad sospechosa desde cuentas administrativas

**Guía**: Use Privileged Identity Management (PIM) de Azure Active Directory (AD) para la generación de registros y alertas cuando se produzca una actividad sospechosa o insegura en el entorno.

Además, use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

- [Cómo implementar Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

- [Información sobre las detecciones de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: El cliente debe usar ubicaciones con nombre de acceso condicional para permitir el acceso a Azure Portal solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

- [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización para instancias de Azure Backup. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

- [Configuración de Azure Backup para usar el inicio de sesión de Azure AD](../app-service/configure-authentication-provider-aad.md)

- [Creación y configuración de una instancia de AAD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure Active Directory (AD) proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

- [Descripción de los informes de Azure AD](../active-directory/reports-monitoring/index.yml)

- [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11: Supervise los intentos de acceso a cuentas desactivadas

**Guía**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización para instancias de Azure Backup. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

Tiene acceso a los orígenes de registro de eventos de actividad de inicio de sesión, auditoría y riesgo de Azure AD, que permiten la integración en Azure Sentinel o SIEM de terceros.

Para simplificar este proceso, cree una configuración de diagnóstico para las cuentas de usuario de Azure AD y envíe los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas de registro deseadas en Log Analytics.

- [Integración de los registros de actividad de Azure en Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización para los almacenes de Recovery Services. Para la desviación de comportamiento de inicio de sesión de cuenta en el plan de control (Azure Portal), use las características de detección de riesgos y de Azure AD Identity Protection para configurar respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. También puede hacer que Azure Sentinel ingiera los datos para investigarlos más.

- [Configuración de Azure Backup para usar el inicio de sesión de Azure AD](../app-service/configure-authentication-provider-aad.md)

- [Visualización del inicio de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: Actualmente no disponible; la Caja de seguridad del cliente todavía no se admite en Azure Backup.

- [Lista de servicios admitidos por la Caja de seguridad del cliente](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="data-protection"></a>Protección de datos

*Para obtener más información, consulte [Control de seguridad: protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: use etiquetas para ayudar a realizar el seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Guía**: Al hacer una copia de seguridad de máquinas virtuales IaaS de Azure, Azure Backup proporciona copias de seguridad independientes y aisladas para evitar la destrucción accidental de datos originales. Las copias de seguridad se almacenan en un almacén de Recovery Services con administración integrada de puntos de recuperación.

Implemente suscripciones o grupos de administración independientes para el desarrollo, la prueba y la producción de almacenes de Recovery Services. Los recursos deben separarse por red virtual o subred, etiquetarse adecuadamente y estar protegidos por un grupo de seguridad de red o Azure Firewall. Los recursos que almacenan o procesan datos confidenciales deben estar suficientemente aislados. Para las máquinas virtuales que almacenan o procesan datos confidenciales, implemente la directiva y los procedimientos necesarios para desactivarlos cuando no estén en uso.

Documentación complementaria:

- [Información general de Azure Backup](./backup-overview.md)

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Actualmente no disponible; las características de identificación, clasificación y prevención de pérdida de datos no están disponibles aún en Azure Backup.

Microsoft administra la infraestructura subyacente para Azure Backup y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: El tráfico de copia de seguridad de los servidores al almacén de Recovery Services se transfiere a través de un vínculo HTTPS seguro y se cifra mediante Estándar de cifrado avanzado (AES) 256 cuando se almacena en el almacén.

- [Descripción del cifrado en reposo en Azure Backup](./backup-encryption.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: Actualmente no disponible; las características de identificación, clasificación y prevención de pérdida de datos no están disponibles aún en Azure Backup.

Microsoft administra la infraestructura subyacente para Azure Backup y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: no disponible actualmente

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos

**Guía**: el control de acceso basado en rol de Azure (Azure RBAC) permite una administración del acceso pormenorizada para Azure. También podrá repartir las tareas entre el equipo y conceder a los usuarios únicamente el nivel de acceso que necesitan para realizar su trabajo.

Azure Backup proporciona tres roles integrados para controlar las operaciones de administración de copia de seguridad: Colaborador de copias de seguridad, operador de copias de seguridad y lector de copias de seguridad. Puede asignar roles integrados de Backup a las diferentes acciones de administración de copia de seguridad.

- [Configuración de RBAC en Azure](../role-based-access-control/role-assignments-portal.md)

- [Uso del control de acceso basado en roles para administrar puntos de recuperación de Azure Backup](./backup-rbac-rs-vault.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso. Microsoft administra la infraestructura subyacente para Azure Backup y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

- [Protección de datos de cliente de Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Azure Backup admite el cifrado para datos en reposo. Para la copia de seguridad local, el cifrado en reposo se proporciona con la frase de contraseña que proporcione durante la copia de seguridad en Azure. Para cargas de trabajo en la nube, los datos están cifrados en reposo con Storage Service Encryption (SSE). Microsoft no descifra los datos de copia de seguridad en ningún momento.

Cuando se realiza una copia de seguridad con el agente de MARS o mediante un almacén de Recovery Services cifrado con una clave administrada por el cliente, solo se tiene acceso a la clave de cifrado. Microsoft nunca mantiene una copia y no tiene acceso a la clave. Si la clave se pierde, Microsoft no puede recuperar los datos de copia de seguridad.

- [Descripción del cifrado en reposo para Azure Backup](./backup-encryption.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure a fin de crear alertas para cuando se produzcan cambios en los almacenes de Azure Recovery Services y otros recursos críticos o relacionados.

- [Creación de alertas para los eventos del registro de actividad de Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para obtener más información, consulte [Control de seguridad: administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: Todavía no está disponible; la evaluación de vulnerabilidades en Azure Security Center todavía no se admite para Azure Backup.

Plataforma subyacente examinada y revisada por Microsoft. Revise los controles de seguridad disponibles para Azure Backup para reducir las vulnerabilidades relacionadas con la configuración del servicio.

- [Descripción de los controles de seguridad disponibles para Azure Backup](./backup-security-controls.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3: Implemente una solución automatizada de administración de revisiones de software de terceros

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Instrucciones**: Actualmente no está disponible; todavía no se admiten las configuraciones de seguridad para Azure Backup en Azure Security Center.

- [Lista de servicios PaaS compatibles con Azure Security Center](../security-center/features-paas.md)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para obtener más información, consulte [Control de seguridad: inventario y administración de recursos](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1: Uso de la detección de recursos de Azure

**Guía**: Use Azure Resource Graph para consultar o detectar todos los recursos (por ejemplo, proceso, almacenamiento, red, puertos y protocolos, etc.) dentro de las suscripciones.  Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y enumere todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager que figuran a continuación.

- [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Visualización de las suscripciones de Azure](/powershell/module/az.accounts/get-azsubscription)

- [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Instrucciones**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos de Azure. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

Además, use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas: Tipos de recursos no permitidos Tipos de recursos permitidos

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4: Mantenimiento de un inventario de los recursos de Azure aprobados y títulos de software

**Guía**: Defina los recursos de Azure y el software aprobados para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en sus suscripciones.

Use Azure Resource Graph para consultar o detectar recursos dentro de sus suscripciones.  Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervisión de Azure Security Center**: No aplicable

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

**Guía**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas: Tipos de recursos no permitidos Tipos de recursos permitidos

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](../governance/policy/samples/index.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="610-implement-approved-application-list"></a>6.10: Implementación de la lista de aplicaciones aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="611-divlimit-users-ability-to-interact-with-azure-resource-manager-via-scriptsdiv"></a>6.11: <div>Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager a través de scripts</div>

**Guía**: Configure el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management.

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="secure-configuration"></a>Configuración segura

*Para obtener más información, consulte [Control de seguridad: configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Instrucciones**: Defina e implemente configuraciones de seguridad estándar para el almacén de Recovery Services con Azure Policy. Use alias de Azure Policy en el espacio de nombres "Microsoft.RecoveryServices" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de los almacenes de Recovery Services.

- [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Use la directiva de Azure Policy [deny] y [deploy if not exist] para aplicar la configuración segura en los recursos de Azure.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Si usa definiciones de personalizadas de Azure Policy, use Azure DevOps o Azure Repos para almacenar y administrar el código de forma segura.

- [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Documentación de Azure Repos](/azure/devops/repos/index?view=azure-devops)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7: Implemente herramientas de administración de configuración del sistema

**Guía**: Use las definiciones de Azure Policy integradas junto con los alias de Azure Policy en el espacio de nombres "Microsoft.RecoveryServices" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8: Implemente herramientas de administración de configuración del sistema para sistemas operativos

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9: Implemente la supervisión de configuración automatizada para servicios de Azure

**Guía**: Use las definiciones de Azure Policy integradas junto con los alias de Azure Policy en el espacio de nombres "Microsoft.RecoveryServices" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Use la directiva de Azure Policy [audit], [deny] y [deploy if not exist] para aplicar automáticamente las configuraciones en los recursos de Azure.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: Al configurar el agente de MARS, almacene la frase de contraseña de cifrado en Azure Key Vault.

- [Creación de un almacén de claves](../key-vault/secrets/quick-create-portal.md)

- [Cómo proporcionar la autenticación de Key Vault con una identidad administrada](../key-vault/general/managed-identity.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: No aplicable; no se admiten identidades administradas para Azure Backup.

- [Servicios que admiten identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

- [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para obtener más información, consulte [Control de seguridad: defensa contra malware](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Use software antimalware administrado centralmente

**Guía**: No aplicable; esta recomendación está dirigida a recursos de proceso. Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Backup), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Backup), pero no se ejecuta en el contenido.

Examine previamente los archivos que se cargan en recursos de Azure que no son de proceso, como App Service, Data Lake Storage, Blob Storage, etc.

Use la detección de amenazas de Azure Security Center para los servicios de datos para detectar malware cargado en cuentas de almacenamiento.

- [Descripción de Microsoft Antimalware para Azure Cloud Services y Virtual Machines](../security/fundamentals/antimalware.md)

- [Descripción de la detección de amenazas de Azure Security Center para servicios de datos](../security-center/threat-protection.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-recovery"></a>Recuperación de datos

*Para obtener más información, consulte [Control de seguridad: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: No aplicable; esta recomendación está destinada a los recursos de los que se hace una copia de seguridad y no a Azure Backup.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Instrucciones**: El almacenamiento con redundancia local (LRS) replica los datos tres veces (crea tres copias de los datos) en una unidad de escalado de almacenamiento de un centro de datos. Todas las copias de los datos se encuentran en la misma región. LRS es una opción de bajo costo para proteger los datos frente a errores de hardware locales. El almacenamiento con redundancia geográfica (GRS) es la opción de replicación predeterminada y recomendada. GRS replica los datos en una región secundaria (a cientos de kilómetros de la ubicación principal de los datos de origen). GRS cuesta más que LRS, pero proporciona un mayor nivel de durabilidad de los datos, aunque se produzca una interrupción regional.

Realice una copia de seguridad de las claves administradas del cliente con Azure Key Vault.

- [Información general de Azure Backup](./backup-overview.md)

- [Creación de una copia de seguridad de las claves del almacén de claves en Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

- [Descripción del cifrado en Azure Backup](./backup-encryption.md#encryption-of-backup-data-using-customer-managed-keys)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: Pruebe la restauración de las claves administradas por el cliente de la copia de seguridad.

- [Restauración de las claves del almacén de claves en Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas del cliente

**Guía**: Para la copia de seguridad local, el cifrado en reposo se proporciona con la frase de contraseña que proporcione durante la copia de seguridad en Azure. Para máquinas virtuales de Azure, los datos están cifrados en reposo con Storage Service Encryption (SSE). Puede habilitar la eliminación temporal en Key Vault para proteger las claves contra la eliminación accidental o malintencionada.

- [Habilitación de la eliminación temporal en Key Vault](../storage/blobs/soft-delete-overview.md?tabs=azure-portal)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para obtener más información, consulte [Control de seguridad: respuesta ante incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

- [Configuración de las automatizaciones de flujos de trabajo en Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

- [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [También puede utilizar la guía de control de incidentes de seguridad de equipos de NIST como referencia para la creación de su propio plan de respuesta a incidentes](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure.

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

- [Consulte la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos del cliente.  Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

- [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: Exporte sus alertas y recomendaciones de Azure Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Sentinel.

- [Configuración de la exportación continua](../security-center/continuous-export.md)

- [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Instrucciones**: use la característica de automatización del flujo de trabajo de Azure Security Center para desencadenar automáticamente respuestas mediante "Logic Apps" en las alertas y recomendaciones de seguridad.

- [Configuración de la automatización de flujo de trabajo y Logic Apps](../security-center/workflow-automation.md)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para obtener más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos en un plazo de 60 días

**Guía**: [Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Puede encontrar más información sobre la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft aquí.](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](../security/benchmarks/overview.md).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).