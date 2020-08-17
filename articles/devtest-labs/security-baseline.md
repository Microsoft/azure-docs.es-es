---
title: Base de referencia de seguridad de Azure para Azure DevTest Labs
description: Base de referencia de seguridad de Azure para Azure DevTest Labs
ms.topic: conceptual
ms.date: 07/23/2020
ms.openlocfilehash: 158ead7531b0b3da2e495e36e40e761961bea498
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87761015"
---
# <a name="azure-security-baseline-for-azure-devtest-labs"></a>Base de referencia de seguridad de Azure para Azure DevTest Labs

La base de referencia de seguridad de Azure para Azure DevTest Labs contiene recomendaciones que lo ayudarán a mejorar la situación de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview.md), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure.

Para obtener más información, consulte [Introducción a las líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para obtener más información, consulte [Control de seguridad: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados
**Guía:** Microsoft mantiene los orígenes de hora de los recursos de Azure. Sin embargo, puede administrar la configuración de sincronización de hora de los recursos de proceso.

Consulte el artículo siguiente para obtener información sobre la configuración de la sincronización de hora de los recursos de proceso de Azure: [Sincronizar la hora en las máquinas virtuales de Windows en Azure](../virtual-machines/windows/time-sync.md). 

**Supervisión de Azure Security Center:** no disponible actualmente

**Responsabilidad:** Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad
**Guía:** Habilite la configuración de diagnóstico del registro de actividad de Azure y envíela a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para su archivo. Los registros de actividad proporcionan información sobre las operaciones que se realizaron en las instancias de Azure DevTest Labs en el nivel de plano de administración. Con los datos del registro de actividad de Azure, puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) realizadas en el nivel del plano de administración para las instancias de DevTest Labs.

Para más información, consulte [Creación de una configuración de diagnóstico para enviar registros de plataforma y métricas a diferentes destinos](../azure-monitor/platform/diagnostic-settings.md).

**Supervisión de Azure Security Center:** no disponible actualmente

**Responsabilidad:** Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure
**Guía:** Habilite la configuración de diagnóstico del registro de actividad de Azure y envíela a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para su archivo. Los registros de actividad proporcionan información sobre las operaciones que se realizaron en las instancias de Azure DevTest Labs en el nivel de plano de administración. Con los datos del registro de actividad de Azure, puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) realizadas en el nivel del plano de administración para las instancias de DevTest Labs.

Para más información, consulte [Creación de una configuración de diagnóstico para enviar registros de plataforma y métricas a diferentes destinos](../azure-monitor/platform/diagnostic-settings.md).

**Supervisión de Azure Security Center:** no disponible actualmente

**Responsabilidad:** Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos
**Guía:** el cliente crea y es propietario de las máquinas virtuales (VM) de Azure DevTest Labs. Por lo tanto, su supervisión es responsabilidad de la organización. Puede usar Azure Security Center para supervisar el sistema operativo del proceso. Entre los datos que recopila Security Center del sistema operativo se incluyen: tipo y versión del sistema operativo, sistema operativo (registros de eventos de Windows), procesos en ejecución, nombre de la máquina, direcciones IP y usuario conectado. El agente de Log Analytics también recopila los archivos de volcado de memoria.

Para más información, consulte los siguientes artículos. 

- [Recopilación de registros de host internos de máquina virtual de Azure con Azure Monitor](../azure-monitor/learn/quick-collect-azurevm.md)
- [Descripción de la recopilación de datos de Azure Security Center](../security-center/security-center-enable-data-collection.md)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad
***Guía:** en Azure Monitor, establezca el período de retención de registro para las áreas de trabajo de Log Analytics asociadas a las instancias de Azure DevTest Labs según las normativas de cumplimiento de su organización.

Para más información, consulte el artículo siguiente: [Establecimiento de parámetros de retención de registros](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión
**Guía:** Habilite la configuración de diagnóstico en el registro de actividad de Azure y envíe los registros a un área de trabajo de Log Analytics. Ejecute consultas en Log Analytics para buscar términos, identificar tendencias, analizar patrones y proporcionar mucha otra información detallada de los datos del registro de actividad que se hayan recopilado para Azure DevTest Labs.

Para más información, consulte los siguientes artículos.

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/platform/diagnostic-settings.md)
- [Recopilación y análisis de registros de actividad de Azure en un área de trabajo de Log Analytics en Azure Monitor](../azure-monitor/platform/activity-log.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7: Habilitación de alertas para actividades anómalas
**Guía:** use el área de trabajo de Azure Log Analytics para supervisar los registros de seguridad y eventos relacionados con Azure DevTest Labs y generar alertas sobre actividades anómalas en él.

Para más información, consulte el artículo siguiente: [Alertas sobre datos de registro de Log Analytics](../azure-monitor/learn/tutorial-response.md)

**Supervisión de Azure Security Center:** no disponible actualmente

**Responsabilidad:** Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware
**Guía:** No aplicable. Azure DevTest Labs no procesa ni genera registros relacionados con antimalware.

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS
**Guía:** No aplicable. Azure DevTest Labs no procesa ni genera registros relacionados con DNS.

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos
**Guía:** Azure DevTest Labs crea máquinas de Azure Compute que son propiedad del cliente y administradas por este. Use Microsoft Monitoring Agent en todas las máquinas virtuales Windows de Azure compatibles para registrar el evento de creación de procesos y el campo `CommandLine`. En el caso de las máquinas virtuales Linux de Azure compatibles, puede configurar manualmente el registro de la consola en cada nodo y usar Syslog para almacenar los datos. Además, use el área de trabajo de Log Analytics de Azure Monitor para revisar los registros y ejecutar consultas en los datos registrados de las máquinas virtuales de Azure.

- [Recopilación de datos en Azure Security Center](../security-center/security-center-enable-data-collection.md#data-collection-tier)
- [Procedimiento para ejecutar consultas personalizadas en Azure Monitor](../azure-monitor/log-query/get-started-queries.md)
- [Orígenes de datos de Syslog en Azure Monitor](../azure-monitor/platform/data-sources-syslog.md)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

## <a name="identity-and-access-control"></a>Identidad y Access Control
*Para obtener más información, consulte [Control de seguridad: identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas
**Guía:** Azure Active Directory (Azure AD) tiene roles integrados que se deben asignar explícitamente y son consultables. Use el módulo de PowerShell de Azure AD para ejecutar consultas ad hoc a fin de detectar cuentas que son miembros de grupos administrativos.

- [Obtención de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)
- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)
- [Roles de Azure DevTest Labs](devtest-lab-add-devtest-user.md)  

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda
**Guía:** Azure Active Directory (Azure AD) no tiene el concepto de contraseñas predeterminadas. Otros recursos de Azure que requieren una contraseña obligan a crear una contraseña con requisitos de complejidad y una longitud mínima, que cambia en función del servicio. Es responsable de las aplicaciones de terceros y de los servicios de Marketplace que pueden usar contraseñas predeterminadas.

DevTest Labs no tiene el concepto de contraseñas predeterminadas. 

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas
**Guía:** Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

Además, para ayudarle a realizar un seguimiento de las cuentas administrativas dedicadas, puede seguir las recomendaciones de Azure Security Center o las directivas integradas de Azure Policy, por ejemplo:

- Debe haber más de un propietario asignado a su suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción

- [Uso de Azure Security Center para supervisar la identidad y el acceso (versión preliminar)](../security-center/security-center-identity-access.md)  
- [Uso de Azure Policy](../governance/policy/tutorials/create-and-manage.md)
- [Roles de Azure DevTest Labs](devtest-lab-add-devtest-user.md)  

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory
**Guía:** DevTest Labs usa el servicio de Azure AD para la administración de identidad. Cuando conceda a los usuarios acceso a un entorno basado en DevTest Labs, tenga en cuenta estos dos aspectos clave:

- **Administración de recursos:** Proporciona acceso a Azure Portal para administrar recursos (crear máquinas virtuales; crear entornos; iniciar, detener, reiniciar, eliminar y aplicar artefactos, etc.). La administración de recursos se lleva a cabo en Azure mediante el control de acceso basado en rol (RBAC). Usted asigna roles a los usuarios y establece permisos de nivel de acceso y recursos.
- **Máquinas virtuales (nivel de red)** : En la configuración predeterminada, las máquinas virtuales usan una cuenta de administrador local. Si hay un dominio disponible (Azure AD Domain Services, un dominio local o un dominio basado en la nube), las máquinas pueden unirse al dominio. Los usuarios pueden usar sus identidades basadas en dominio mediante el artefacto de unión a un dominio para conectarse a las máquinas. 

- [Arquitectura de referencia para DevTest Labs](devtest-lab-reference-architecture.md#architecture)
- [Descripción del SSO con Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory
**Guía:** Habilite Multi-Factor Authentication (MFA) de Azure Active Directory (AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

- [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)  
- [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Supervisión de Azure Security Center:** * sí

**Responsabilidad:** Customer


### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas
**Guía:** utilice estaciones de trabajo de acceso privilegiado (PAW) con la MFA configurada para iniciar la sesión y configurar recursos de Azure.

- [Más información sobre las estaciones de trabajo con privilegios de acceso](/windows-server/identity/securing-privileged-access/privileged-access-workstations)  
- [Procedimiento para habilitar la MFA en Azure](../active-directory/authentication/howto-mfa-getstarted.md)  

**Supervisión de Azure Security Center:** N/D

**Responsabilidad:** Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7: Registre y alerte de la actividad sospechosa desde cuentas administrativas
**Guía:** Use los informes de seguridad de Azure Active Directory (Azure AD) para la generación de registros y alertas cuando se produzcan actividades sospechosas o no seguras en el entorno. Use Azure Security Center para supervisar la actividad de identidad y acceso.

- [Procedimiento para identificar usuarios de Azure AD marcados por una actividad de riesgo](../active-directory/identity-protection/overview-identity-protection.md)  
- [Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center](../security-center/security-center-identity-access.md)  

**Supervisión de Azure Security Center:** no disponible actualmente

**Responsabilidad:** Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas
**Guía:** Use ubicaciones con nombre de acceso condicional para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

- [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)  

**Supervisión de Azure Security Center:** no disponible actualmente

**Responsabilidad:** Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory
**Guía:** Use Azure Active Directory (AD) como sistema central de autenticación y autorización. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

- [Procedimiento para crear y configurar una instancia de Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)  

**Supervisión de Azure Security Center:** no disponible actualmente

**Responsabilidad:** Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios
**Guía:** Azure Active Directory (Azure AD) proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

- [Descripción de los informes de Azure AD](../active-directory/reports-monitoring/overview-reports.md)  
- [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)  

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11: Supervise los intentos de acceso a cuentas desactivadas
**Guía:** puede usar orígenes de registro de actividad de inicio de sesión de Azure Active Directory (Azure AD), auditoría y riesgos para la supervisión, lo que permite realizar la integración con cualquier herramienta de administración de eventos e información de seguridad (SIEM) y de supervisión.

Puede simplificar este proceso creando una configuración de diagnóstico para las cuentas de usuario de Azure Active Directory y enviando los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas en el área de trabajo de Log Analytics.

- [Integración de los registros de actividad de Azure en Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)  

**Supervisión de Azure Security Center:** no disponible actualmente

**Responsabilidad:** Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas
**Guía:** Use las características de protección de identidad y detección de riesgo de Azure Active Directory (Azure AD) para configurar respuestas automatizadas a las acciones sospechosas que se detecten relacionadas con las identidades de los usuarios.

- [Visualización de los inicios de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)  
- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)  

**Supervisión de Azure Security Center:** no disponible actualmente

**Responsabilidad:** Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico
**Guía:** Caja de seguridad del cliente no es compatible actualmente con Azure DevTest Labs.

- [Lista de servicios admitidos por la Caja de seguridad del cliente](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability) 

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

## <a name="data-protection"></a>Protección de datos
*Para obtener más información, consulte [Control de seguridad: protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial
**Guía:** use etiquetas para ayudar a realizar el seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

- [Creación y uso de etiquetas](../azure-resource-manager/resource-group-using-tags.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial
**Guía:** implemente suscripciones o grupos de administración independientes para los entornos de desarrollo, pruebas y producción. Las instancias de Azure DevTest Labs deben estar separadas por la red o subred virtual y estar correctamente etiquetadas. 

- [Creación de suscripciones adicionales de Azure](../billing/billing-create-subscription.md)
- [Creación de grupos de administración](../governance/management-groups/create.md)
- [Configuración de una red virtual en Azure DevTest Labs](devtest-lab-configure-vnet.md)
- [Creación y uso de etiquetas](../azure-resource-manager/resource-group-using-tags.md)
- [Creación y uso de etiquetas de DevTest Labs](devtest-lab-add-tag.md)

**Supervisión de Azure Security Center:** no disponible actualmente

**Responsabilidad:** Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial
**Guía:** Aún no disponible; las características de identificación, clasificación y prevención de pérdida de datos aún no están disponibles en Azure DevTest Labs.

Microsoft administra la infraestructura subyacente para Azure DevTest Labs y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center:** no disponible actualmente

**Responsabilidad:** Compartido

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito
**Guía:** Azure DevTest Labs requiere comunicaciones cifradas TLS de manera predeterminada. Actualmente se admiten las versiones de TLS 1.2. Si la biblioteca o la herramienta cliente no admiten TLS, se pueden habilitar las conexiones no cifradas mediante Azure Portal o las API de administración. En casos en los que no es posible establecer conexiones cifradas, se recomienda colocar el laboratorio y la aplicación cliente en una red virtual.

[Descripción del escenario de cifrado en tránsito para DevTest Labs](https://techcommunity.microsoft.com/t5/azure-developer-community-blog/azure-devtest-labs-enforcing-tls-1-2-starting-may-01-2020/ba-p/1236279)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Compartido

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales
**Guía:** Las características de identificación, clasificación y prevención de pérdida de datos aún no están disponibles en Azure DevTest Labs. Etiquete las instancias que contienen información confidencial como tal e implemente una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center:** no disponible actualmente

**Responsabilidad:** Customer

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos
**Guía:** Use el control de acceso basado en rol de Azure Active Directory (Azure AD) para controlar el acceso a los laboratorios en Azure DevTest Labs.

- [Configuración de RBAC en Azure](../role-based-access-control/role-assignments-portal.md)
- [Descripción de los roles de DevTest Labs](devtest-lab-add-devtest-user.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso
**Guía:** Si es necesario para el cumplimiento de los recursos de proceso como parte de DevTest Labs, implemente una herramienta de terceros, como una solución de prevención de pérdida de datos basada en host automatizada, para aplicar controles de acceso a los datos incluso cuando se copian datos de un sistema.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** No aplicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo
**Guía:** Azure DevTest Labs almacena los siguientes datos de cliente:

- [Resultados de artefacto](add-artifact-vm.md) que incluyen registros de implementación y extensión generados al aplicar artefactos.
- [Documentos de fórmula](devtest-lab-manage-formulas.md) que se utilizan para crear máquinas virtuales a partir de fórmulas.
- Discos de datos y del sistema operativo para máquinas virtuales de laboratorio. 

Los resultados de artefacto y los documentos de fórmula se envían a una cuenta de Azure Storage que se crea como parte de cada implementación de laboratorio. Los datos de Azure Storage se cifran y descifran de forma transparente mediante el cifrado AES de 256 bits, uno de los cifrados de bloques más sólidos que hay disponibles, y son compatibles con FIPS 140-2. El cifrado de Azure Storage no se puede deshabilitar. Puede confiar en las claves administradas por Microsoft para el cifrado de la cuenta de almacenamiento, o puede administrar el cifrado con sus propias claves. Para obtener más información, consulte [Cifrado para una cuenta de almacenamiento de laboratorio](encrypt-storage.md).

De manera predeterminada, todos los discos de datos y del sistema operativo de laboratorio están cifrados con una clave administrada por la plataforma. Todos los discos administrados, instantáneas e imágenes, así como los datos que se escriban en discos administrados existentes, se cifran automáticamente en reposo con claves administradas por la plataforma. Como propietario de un laboratorio, puede configurar los discos del sistema operativo de laboratorio para que se cifren con una clave administrada por el cliente. El cifrado que usa una clave administrada por el cliente para los discos de datos de laboratorio no es configurable actualmente a través del propio laboratorio. Sin embargo, un administrador de suscripciones puede configurar esta opción para los discos de laboratorio dentro de una suscripción por ahora. Para obtener más información, consulte [Cifrado de discos del sistema operativo de laboratorio de DevTest Labs mediante claves administradas por el cliente](encrypt-disks-customer-managed-keys.md).

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Compartido

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure
**Guía:** Use Azure Monitor con el registro de actividad de Azure para crear alertas cuando se produzcan cambios en instancias de DevTest Labs y en otros recursos críticos o relacionados.

- [Creación de alertas para los eventos del registro de actividad de Azure](../azure-monitor/platform/alerts-activity-log.md)
- [Creación de alertas para los eventos del registro de actividad de DevTest Labs](create-alerts.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer



## <a name="vulnerability-management"></a>Administración de vulnerabilidades
*Para obtener más información, consulte [Control de seguridad: administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado
**Guía:** siga las recomendaciones de Azure Security Center sobre la protección de las instancias de Azure DevTest Labs y recursos relacionados.

Microsoft realiza la administración de vulnerabilidades en los recursos subyacentes que admiten Azure DevTest Labs.

- [Recomendaciones de Azure Security Center](../security-center/recommendations-reference.md) 

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Compartido

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada
**Guía:** use Azure Update Management para asegurarse de que las actualizaciones de seguridad más recientes se instalan en sus máquinas virtuales con Windows y Linux hospedadas dentro de DevTest Labs. En el caso de las máquinas virtuales con Windows, asegúrese de que Windows Update se ha habilitado y configurado para actualizarse automáticamente. Esta configuración no está disponible actualmente a través de DevTest Labs. Sin embargo, el administrador del laboratorio o el administrador de la suscripción puede configurar esta opción en las máquinas virtuales de proceso subyacentes de su suscripción. 

- [Configuración de Update Management para máquinas virtuales en Azure](../automation/update-management/update-mgmt-overview.md)
- [Descripción de las directivas de seguridad de Azure supervisadas por Security Center](../security-center/security-center-policy-definitions.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3: Implemente una solución automatizada de administración de revisiones de software de terceros
***Guía:*** como administrador del laboratorio, puede usar [artefactos de DevTest Labs](add-artifact-vm.md) para automatizar las actualizaciones de las imágenes personalizadas de laboratorio, incluidas las revisiones de seguridad y otras actualizaciones. 

Obtenga más información sobre un [generador de imágenes de DevTest Labs](image-factory-create.md) es una solución de configuración como código que crea y distribuye imágenes de forma automática y periódica con todas las configuraciones deseadas. 

Como administrador de la suscripción, también puede usar la solución Azure Update Management para administrar las actualizaciones y revisiones para las máquinas virtuales de DevTest Labs. Update Management se basa en el repositorio de actualización configurado localmente para aplicar revisiones a sistemas de Windows compatibles. Herramientas como System Center Updates Publisher (Updates Publisher) le permiten publicar actualizaciones personalizadas en Windows Server Update Services (WSUS). Este escenario permite que Update Management aplique revisiones a las máquinas que usan Configuration Manager como repositorio de actualizaciones con software de terceros.

- [Solución Update Management de Azure](../automation/update-management/update-mgmt-overview.md)
- [Administración de actualizaciones y revisiones para las máquinas virtuales](../automation/update-management/update-mgmt-overview.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos
**Guía:** Exporte los resultados de análisis en intervalos coherentes y compare los resultados para comprobar que se han corregido las vulnerabilidades. Al usar una recomendación de administración de vulnerabilidades sugerida por Azure Security Center, el cliente puede ir al portal de la solución seleccionada para ver los datos de análisis históricos.

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas
**Guía:** use la clasificación de riesgo predeterminada (puntuación de seguridad) proporcionada por Azure Security Center.

- [Descripción de la puntuación de seguridad de Azure Security Center](../security-center/secure-score-security-controls.md)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos
*Para más información, consulte [Control de seguridad: Administración de recursos y del inventario](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada
**Guía:** use Azure Resource Graph para consultar y detectar todos los recursos (incluidos los recursos de DevTest Labs) de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure y los recursos de las suscripciones.

- [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)
- [Visualización de las suscripciones de Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)
- [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos
**Guía:** Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica según una taxonomía.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)
- [Configuración de etiquetas de DevTest Labs](devtest-lab-add-tag.md)

**Supervisión de Azure Security Center:** No disponible

**Responsabilidad:** Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados
**Guía:** use el etiquetado, los grupos de administración y las suscripciones independientes (y, si procede, laboratorios independientes) para organizar y realizar un seguimiento de los laboratorios y los recursos relacionados con ellos. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan rápidamente de la suscripción.

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)
- [Creación de grupos de administración](../governance/management-groups/create.md)
- [Creación de un laboratorio con DevTest Labs](devtest-lab-create-lab.md)
- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)
- [Configuración de etiquetas para un laboratorio](devtest-lab-add-tag.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados
**Guía:** cree un inventario de los recursos de Azure aprobados y el software aprobado para los recursos de proceso según las necesidades de la organización. Como administrador de la suscripción, también podría usar los controles de aplicaciones adaptables, una característica de Azure Security Center para definir un conjunto de aplicaciones que se pueden ejecutar en los grupos configurados de máquinas de laboratorio. Esta característica está disponible tanto para máquinas virtuales y servidores de Azure como Windows (todas las versiones, clásica o Azure Resource Manager) y Linux que no son de Azure.

- [Habilitación de los controles de aplicaciones adaptables](../security-center/security-center-adaptive-application.md)
 
**Supervisión de Azure Security Center:** Responsabilidad **no aplicable:** Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados
**Guía:** use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Además, use Azure Resource Graph para consultar o detectar recursos dentro de sus suscripciones. Esto puede ayudar en entornos de alta seguridad, como aquellos con cuentas de almacenamiento.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)
- [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso
**Guía:** Azure Automation proporciona un control completo durante la implementación, las operaciones y la retirada de las cargas de trabajo y recursos. Como administrador de la suscripción, puede usar el inventario de Azure Virtual Machine para automatizar la recopilación de información sobre todo el software de las máquinas virtuales de DevTest Labs de la suscripción. Las propiedades de nombre del software, versión, publicador y hora de actualización están disponibles en Azure Portal. Para obtener acceso a la fecha de instalación y otra información, se requiere que el cliente habilite los diagnósticos de nivel de invitado y transfiera los registros de eventos de Windows a un área de trabajo de Log Analytics.

Además de usar Change Tracking para la supervisión de las aplicaciones de software, los controles de aplicaciones adaptables de Azure Security Center usan el aprendizaje automático para analizar las aplicaciones que se ejecutan en los equipos y crear una lista de permitidos a partir de esta inteligencia. Esta funcionalidad simplifica enormemente el proceso de configuración y mantenimiento de directivas de listas de aplicaciones permitidas, lo que le permite evitar el uso de software no deseado en su entorno. Puede configurar el modo de auditoría o de aplicación. El modo de auditoría solo audita la actividad en las máquinas virtuales protegidas. El modo de aplicación exige el cumplimiento de las reglas y se asegura de bloquear las aplicaciones cuya ejecución no está permitida. 

- [Introducción a Azure Automation](../automation/automation-intro.md)
- [Habilitación del inventario de máquinas virtuales de Azure](../automation/automation-tutorial-installed-software.md)
- [Información sobre los controles de aplicaciones adaptables](../security-center/security-center-adaptive-application.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer


### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas
**Guía:** Azure Automation proporciona un control completo durante la implementación, las operaciones y la retirada de las cargas de trabajo y recursos. Como administrador de la suscripción, puede usar Change Tracking para identificar todo el software instalado en las máquinas virtuales hospedadas en DevTest Labs. Puede implementar su propio proceso o usar State Configuration de Azure Automation para quitar software no autorizado.

- [Introducción a Azure Automation](../automation/automation-intro.md)
- [Seguimiento de cambios en el entorno con la solución Change Tracking](../automation/change-tracking.md)
- [Introducción a Azure Automation State Configuration](../automation/automation-dsc-overview.md)

**Supervisión de Azure Security Center:** No disponible

**Responsabilidad:** Customer

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas
**Guía:** como administrador de la suscripción, puede usar los controles de aplicaciones adaptables de Azure Security Center para asegurarse de que solo se ejecute software autorizado y de que todo el software no autorizado esté bloqueado para que no se ejecute en máquinas virtuales de Azure hospedadas en DevTest Labs.

- [Uso de los controles de aplicaciones adaptables de Azure Security Center](../security-center/security-center-adaptive-application.md)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados
**Guía:** use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas: 

- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Vea los artículos siguientes: 
- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)
- [Denegación de un tipo de recurso específico con Azure Policy](../governance/policy/samples/not-allowed-resource-types.md)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer


### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10: Mantenimiento de un inventario de títulos de software aprobados
**Guía:** el control de aplicaciones adaptables es una solución integral, inteligente y automatizada de Azure Security Center que ayuda a controlar qué aplicaciones se pueden ejecutar en las máquinas virtuales de Azure y que no son de Azure (Windows y Linux), hospedadas en DevTest Labs. Tenga en cuenta que debe ser administrador de la suscripción para configurar esta opción para los recursos de proceso subyacentes hospedados en DevTest Labs. Implemente una solución de terceros si esta configuración no cumple con los requisitos de la organización.

- [Uso de los controles de aplicaciones adaptables de Azure Security Center](../security-center/security-center-adaptive-application.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager
**Guía:** use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager mediante la configuración de la opción **Bloquear acceso**"** en la aplicación **Administración de Microsoft Azure**.

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso
**Guía:** según el tipo de scripts, puede usar configuraciones específicas del sistema operativo o recursos de terceros para limitar la capacidad de los usuarios de ejecutar scripts en las máquinas virtuales hospedadas en DevTest Labs. También puede usar los controles de aplicaciones adaptables de Azure Security Center para asegurarse de que solo se ejecute software autorizado y de que todo el software no autorizado no se ejecute en las máquinas virtuales de Azure subyacentes.

- [Control de la ejecución de scripts de PowerShell en entornos Windows](/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-6)
- [Uso de los controles de aplicaciones adaptables de Azure Security Center](../security-center/security-center-adaptive-application.md)

**Supervisión de Azure Security Center:** No disponible

**Responsabilidad:** Customer


### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo
**Guía:** las aplicaciones de alto riesgo implementadas en el entorno de Azure se pueden aislar mediante la red virtual, la subred, las suscripciones, los grupos de administración, etc. y suficientemente protegidos con una instancia de Azure Firewall, un firewall de aplicaciones web (WAF) o un grupo de seguridad de red (NSG).

- [Configuración de una red virtual en Azure DevTest Labs](devtest-lab-configure-vnet.md)
- [Información general de Azure Firewall](../firewall/overview.md)
- [Introducción al firewall de aplicaciones web](../web-application-firewall/overview.md)
- [Introducción a la seguridad de red](../virtual-network/security-overview.md)
- [Introducción a Azure Virtual Network]()
- [Organización de los recursos con grupos de administración de Azure](../governance/management-groups/overview.md)
- [Guía de decisiones de suscripción](/azure/cloud-adoption-framework/decision-guides/subscriptions/)

**Supervisión de Azure Security Center:** No disponible

**Responsabilidad:** Customer

## <a name="secure-configuration"></a>Configuración segura
**Para más información, consulte Control de seguridad: Configuración segura.**

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure
**Guía:** Use alias de Azure Policy para crear directivas personalizadas con el fin de auditar o aplicar la configuración de los recursos de Azure creados como parte de DevTest Labs. También puede usar definiciones de Azure Policy integradas.

Además, Azure Resource Manager tiene la capacidad de exportar la plantilla en notación de objetos JavaScript (JSON), que se debe revisar para asegurarse de que las configuraciones cumplan o superen los requisitos de seguridad de la organización.

También puede usar las recomendaciones de Azure Security Center como línea de base de configuración segura para los recursos de Azure.

- [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)
- [Tutorial: Creación y administración de directivas para aplicar el cumplimiento](../governance/policy/tutorials/create-and-manage.md)
- [Exportación de uno y varios recursos a una plantilla en Azure Portal](../azure-resource-manager/templates/export-template-portal.md)
- [Guía de referencia sobre las recomendaciones de seguridad](../security-center/recommendations-reference.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras
**Guía:** Use las recomendaciones de Azure Security Center para mantener las configuraciones de seguridad de todos los recursos de proceso subyacentes creados como parte de DevTest Labs. Además, puede usar imágenes de sistema operativo personalizado, la configuración de estado de Azure Automation o artefactos de DevTest Labs para establecer la configuración de seguridad del sistema operativo que requiere su organización.

- [Cómo supervisar las recomendaciones de Azure Security Center](../security-center/security-center-recommendations.md)
- [Guía de referencia sobre las recomendaciones de seguridad](../security-center/recommendations-reference.md)
- [Introducción a Azure Automation State Configuration](../automation/automation-dsc-overview.md)
- [Carga de un VHD y su uso para crear máquinas virtuales Windows nuevas en Azure](../virtual-machines/windows/upload-generalized-managed.md)
- [Creación de una VM Linux desde un disco personalizado con la CLI de Azure](../virtual-machines/linux/upload-vhd.md)
- [Creación y distribución de imágenes personalizadas en varias instancias de DevTest Labs](image-factory-save-distribute-custom-images.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** No aplicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras
**Guía:** Utilice las reglas **denegar** e **implementar si no existe** de Azure Policy para aplicar una configuración segura en los recursos de Azure creados como parte de DevTest Labs. Además, puede usar plantillas de Azure Resource Manager para mantener la configuración de seguridad de los recursos de Azure que requiere su organización.

- [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)
- [Creación y administración de directivas para aplicar el cumplimiento](../governance/policy/tutorials/create-and-manage.md)
- [Información general sobre plantillas de Azure Resource Manager](../azure-resource-manager/templates/overview.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras
**Guía:** Siga las recomendaciones de Azure Security Center sobre cómo realizar evaluaciones de vulnerabilidades en los recursos de proceso de Azure subyacentes creados como parte de un laboratorio. Además, puede usar plantillas de Azure Resource Manager, imágenes de sistema operativo personalizado o la configuración de estado de Azure Automation para mantener la configuración de seguridad del sistema operativo que requiere su organización. También puede usar una solución de generador de imágenes, que es una solución de configuración como código que crea y distribuye imágenes de forma automática y periódica con todas las configuraciones deseadas.

Además, las imágenes de máquinas virtuales de Azure Marketplace que publica Microsoft las administra y mantiene Microsoft mismo.

- [Implementación de las recomendaciones de evaluación de vulnerabilidades de Azure Security Center](../security-center/security-center-vulnerability-assessment-recommendations.md)
- [Introducción a Azure Automation State Configuration](../automation/automation-dsc-overview.md)
- [Script de ejemplo para cargar un disco duro virtual en Azure y crear una máquina virtual nueva](../virtual-machines/scripts/virtual-machines-windows-powershell-upload-generalized-script.md)
- [Creación de un generador de imágenes personalizadas en Azure DevTest Labs](image-factory-create.md)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Compartido

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure
**Guía:** Use Azure DevOps para almacenar y administrar de forma segura el código, como directivas de Azure personalizadas, plantillas de Azure Resource Manager y scripts de Desired State Configuration. Para acceder a los recursos que administra en Azure DevOps, puede conceder o denegar permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integran con Azure DevOps.

- [Tutorial de Git de Azure Repos](/devops/repos/git/gitworkflow?view=azure-devops)
- [Acerca de los permisos y los grupos](/devops/organizations/security/about-permissions?view=azure-devops&tabs=preview-page)
- [Integración entre Azure DevTest Labs y el flujo de trabajo de Azure DevOps](devtest-lab-dev-ops.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura
**Guía:** Si usa imágenes personalizadas, use el control de acceso basado en rol (RBAC) para asegurarse de que solo los usuarios autorizados pueden acceder a las imágenes. Con Shared Image Gallery, puede compartir sus imágenes con laboratorios específicos que las necesiten. En el caso de las imágenes de contenedor, almacénelas en Azure Container Registry y use RBAC para asegurarse de que solo los usuarios autorizados puedan obtener acceso a las imágenes.

- [Descripción de RBAC en Azure](../role-based-access-control/rbac-and-directory-admin-roles.md)
- [Configuración de RBAC en Azure](../role-based-access-control/quickstart-assign-role-user-portal.md)
- [Configuración de Shared Image Gallery para Azure DevTest Labs](configure-shared-image-gallery.md)
- [Descripción de RBAC para Container Registry](../container-registry/container-registry-roles.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="77-deploy-system-configuration-management-tools"></a>7.7: Implemente herramientas de administración de configuración del sistema
**Guía:** defina e implemente configuraciones de seguridad estándar para los recursos de Azure con Azure Policy. Use alias de Azure Policy para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de los recursos de Azure creados en DevTest Labs. También puede usar definiciones de directivas integradas relacionadas con recursos concretos. Además, puede usar Azure Automation para implementar los cambios de configuración.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)
- [Uso de alias](../governance/policy/concepts/definition-structure.md#aliases)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8: Implemente herramientas de administración de configuración del sistema para sistemas operativos
**Guía:** Azure Automation State Configuration es un servicio de administración de configuración para los nodos de Desired State Configuration (DSC) de cualquier centro de datos local o en la nube. Puede incorporar fácilmente máquinas, asignarles configuraciones declarativas y ver informes que muestran el cumplimiento por parte de cada máquina del estado deseado especificado. También puede escribir un artefacto personalizado que pueda instalarse en todas las máquinas de laboratorio para asegurarse de que siguen las directivas de la organización. 

- [Incorporación de máquinas para su administración mediante Azure Automation State Configuration](../automation/automation-dsc-onboarding.md)
- [Creación de artefactos personalizados para máquinas virtuales de DevTest Labs](devtest-lab-artifact-author.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9: Implemente la supervisión de configuración automatizada para servicios de Azure
**Guía:** Use Azure Security Center para realizar análisis de base de referencia para los recursos de Azure creados en DevTest Labs. Además, use Azure Policy para alertar y auditar las configuraciones de recursos de Azure.

- [Corrección de recomendaciones en Azure Security Center](../security-center/security-center-remediate-recommendations.md)
 
**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos
**Guía:** Use Azure Security Center para realizar exámenes de base de referencia para la configuración del sistema operativo y de Docker para los contenedores.

- [Descripción de las recomendaciones sobre contenedores de Azure Security Center](../security-center/security-center-container-recommendations.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura
**Guía:** Use Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de secretos para las aplicaciones en la nube.

- [Configuración de identidades administradas para implementar entornos de Azure Resource Manager en DevTest Labs](use-managed-identities-environments.md)
- [Configuración de identidades administradas para implementar máquinas virtuales en DevTest Labs](enable-managed-identities-lab-vms.md)
- [Creación de un almacén de claves](../key-vault/quick-create-portal.md)
- [Cómo proporcionar la autenticación de Key Vault con una identidad administrada](../key-vault/managed-identity.md)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática
**Guía:** Use las identidades administradas para proporcionar a los servicios de Azure una identidad administrada automáticamente en Azure AD. Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluyendo Key Vault, sin necesidad de credenciales en el código.

- [Configuración de identidades administradas para implementar entornos de Azure Resource Manager en DevTest Labs](use-managed-identities-environments.md)
- [Configuración de identidades administradas para implementar máquinas virtuales en DevTest Labs](enable-managed-identities-lab-vms.md)
 
**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada
**Guía:** Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

- Cómo configurar el escáner de credenciales

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer


## <a name="malware-defense"></a>Defensa contra malware
*Para más información, consulte Control de seguridad: defensa contra malware.*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Use software antimalware administrado centralmente
**Guía:** Use Microsoft Antimalware para Azure Cloud Services y Virtual Machines para supervisar y defender continuamente sus recursos. Para Linux, use una solución antimalware de terceros. Use también la detección de amenazas de Azure Security Center para los servicios de datos para detectar malware cargado en cuentas de almacenamiento.

- Configuración de Microsoft Antimalware para Cloud Services y Virtual Machines
- Protección contra amenazas en Azure Security Center

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso
**Guía:** Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure App Service hospedado en un laboratorio), pero no se ejecuta en el contenido.
Examine previamente los archivos que se cargan en recursos de Azure que no son de proceso, como App Service, Data Lake Storage, Blob Storage, etc.

Use la detección de amenazas de Azure Security Center para los servicios de datos para detectar malware cargado en cuentas de almacenamiento.

- Descripción de Microsoft Antimalware para Azure Cloud Services y Virtual Machines
- Descripción de la detección de amenazas de Azure Security Center para servicios de datos

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** No aplicable


### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware
**Guía:** Cuando se implementa, Microsoft Antimalware para Azure instalará automáticamente y de forma predeterminada las actualizaciones más recientes de firma, plataforma y motor. Siga las recomendaciones en Azure Security Center: "Compute & apps" (Proceso y aplicaciones) para asegurarse de que todos los puntos de conexión de los recursos de proceso subyacentes de DevTest Labs están actualizados con las firmas más recientes. El sistema operativo Windows se puede proteger aún más con seguridad adicional para limitar el riesgo de ataques basados en malware y virus con el servicio Microsoft Defender Advanced Threat Protection que se integra con Azure Security Center.

- Implementación de Microsoft Antimalware para Azure Cloud Services y Virtual Machines
- Protección contra amenazas avanzada de Microsoft Defender

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

## <a name="data-recovery"></a>Recuperación de datos
*Para más información, consulte [Control de seguridad: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas
**Guía:** actualmente, Azure DevTest Labs no admite copias de seguridad de máquinas virtuales ni instantáneas. Sin embargo, puede habilitar y configurar Azure Backup en las máquinas virtuales de Azure subyacentes hospedadas en DevTest Labs. Además, también puede configurar la frecuencia y el período de retención deseados para las copias de seguridad automáticas, siempre que tenga acceso adecuado a los recursos de proceso subyacentes. 

- [Información general sobre la copia de seguridad de máquinas virtuales de Azure](../backup/backup-azure-vms-introduction.md)
- [Copia de seguridad de una máquina virtual de Azure desde la configuración de esta](../backup/backup-azure-vms-first-look-arm.md)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente
**Guía:** actualmente, Azure DevTest Labs no admite copias de seguridad de máquinas virtuales ni instantáneas. Sin embargo, puede crear instantáneas de las máquinas virtuales de Azure subyacentes hospedadas en DevTest Labs o en los discos administrados conectados a esas instancias mediante PowerShell o las API REST, siempre y cuando tenga acceso adecuado a los recursos de proceso subyacentes. También puede hacer una copia de seguridad de cualquier clave administrada por el cliente en Azure Key Vault.

Habilite Azure Backup en máquinas virtuales de Azure de destino y los períodos de retención y frecuencia deseados. Esto incluye una copia de seguridad completa del estado del sistema. Si usa Azure Disk Encryption, la copia de seguridad de la máquina virtual de Azure administra automáticamente la copia de seguridad de las claves administradas por el cliente.

- [Copia de seguridad en máquinas virtuales de Azure cifradas](../backup/backup-azure-vms-encryption.md)
- [Información general sobre la copia de seguridad de máquinas virtuales de Azure](../backup/backup-azure-vms-introduction.md)
- [Información general sobre la copia de seguridad de máquinas virtuales de Azure](../backup/backup-azure-vms-introduction.md)
- [Creación de una copia de seguridad de las claves del almacén de claves en Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente
**Guía:** Garantice la capacidad de realizar la restauración de datos de contenido en Azure Backup de forma periódica. Si es necesario, pruebe la restauración del contenido en una red virtual o suscripción aislada. Además, pruebe la restauración de la copia de seguridad de las claves administradas por el cliente.

Si usa Azure Disk Encryption, puede restaurar la máquina virtual de Azure con las claves de cifrado del disco. Cuando use Disk Encryption, podrá restaurar la máquina virtual de Azure con las claves de cifrado de discos.

- [Copia de seguridad en máquinas virtuales de Azure cifradas](../backup/backup-azure-vms-encryption.md)
- [Recuperación de archivos de la copia de seguridad de máquinas virtuales de Azure](../backup/backup-azure-restore-files-from-vm.md)
- [Restauración de las claves del almacén de claves en Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)
- [Copia de seguridad y restauración de una máquina virtual cifrada](../backup/backup-azure-vms-encryption.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente
**Guía:** Al realizar la copia de seguridad de discos administrados con Azure Backup, las máquinas virtuales se cifran en reposo con Storage Service Encryption (SSE). Azure Backup también puede hacer una copia de seguridad de las VM de Azure cifradas mediante Azure Disk Encryption (ADE). Azure Disk Encryption se integra con las claves de cifrado de BitLocker (BEK), que se protegen en un almacén de claves como secretos. Azure Disk Encryption también se integra con las claves de cifrado de las claves de Azure Key Vault (KEK). Habilite la eliminación temporal en Key Vault para proteger las claves contra la eliminación accidental o malintencionada.

- [Eliminación temporal para máquinas virtuales](../backup/soft-delete-virtual-machines.md)
- [Información general sobre la eliminación temporal de Azure Key Vault](../key-vault/general/soft-delete-overview.md)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

## <a name="incident-response"></a>Respuesta a los incidentes
*Para obtener más información, consulte [Control de seguridad: respuesta ante incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes
**Guía:** Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal y las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

- [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)
- [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)
- [Uso de la guía de control de incidentes de seguridad de equipos de NIST para la creación de su propio plan de respuesta a incidentes](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes
**Guía:** Azure Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) con etiquetas y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure, especialmente los que procesan datos confidenciales. Es su responsabilidad asignar prioridades a la corrección de las alertas en función de la importancia de los recursos y el entorno de Azure donde se produjo el incidente.

- [Alertas de seguridad en el Centro de seguridad de Azure](../security-center/security-center-alerts-overview.md)
- [Uso de etiquetas para organizar los recursos de Azure](../azure-resource-manager/management/tag-resources.md)

**Supervisión de Azure Security Center:** Sí

**Responsabilidad:** Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad
**Guía:** Realice ejercicios para probar las funcionalidades de respuesta a los incidentes de los sistemas periódicamente para ayudar a proteger los recursos de Azure. Identifique puntos débiles y brechas y revise el plan según sea necesario.

- [Publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad
**Guía:** La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

- [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes
**Guía:** Exporte las alertas y recomendaciones de Azure Security Center y mediante la característica de exportación continua para ayudar a identificar los riesgos para los recursos de Azure. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Azure Sentinel.

- [Configuración de la exportación continua](../security-center/continuous-export.md)
- [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Customer

#### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad
**Guía:** Use la característica Automatización de flujo de trabajo de Azure Security Center para desencadenar automáticamente las respuestas a través de Logic Apps en las alertas y recomendaciones de seguridad para proteger los recursos de Azure.

- [Configuración de la automatización de flujo de trabajo y Logic Apps](../security-center/workflow-automation.md)
 
****Supervisión de Azure Security Center:**** no aplicable

**Responsabilidad:** Customer


## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo
*Para más información, consulte Control de seguridad: [Pruebas de penetración y ejercicios del equipo rojo](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*


### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos en un plazo de 60 días
**Guía:** siga las reglas de involucración de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft.

- [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)
- [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervisión de Azure Security Center:** No aplicable

**Responsabilidad:** Compartido

## <a name="next-steps"></a>Pasos siguientes
Consulte el artículo siguiente:

- [Alertas de seguridad de entornos en Azure DevTest Labs](environment-security-alerts.md)
