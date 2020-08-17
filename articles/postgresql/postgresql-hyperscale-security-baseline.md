---
title: 'Base de referencia de seguridad de Azure para Azure Database for PostgreSQL: Hiperescala'
description: La base de referencia de seguridad de Azure Database for PostgreSQL Hiperescala es una guía de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: postgresql
ms.topic: conceptual
ms.date: 08/04/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: debbc47204aca2fbea1307bd4b692921725e5fd0
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87563915"
---
# <a name="azure-security-baseline-for-azure-database-for-postgresql---hyperscale"></a>Base de referencia de seguridad de Azure para Azure Database for PostgreSQL: Hiperescala

La base de referencia de seguridad de Azure Database for PostgreSQL Hiperescala contiene recomendaciones que le ayudarán a mejorar la posición de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](https://docs.microsoft.com/azure/security/benchmarks/overview), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para obtener más información, consulte [Introducción a las líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).

## <a name="network-security"></a>Seguridad de las redes

*Para más información, consulte [Control de seguridad: Seguridad de redes](/azure/security/benchmarks/security-control-network-security).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Guía**: El firewall del servidor Azure Database for PostgreSQL impide todo acceso al nodo de coordinación de Hiperescala (Citus) hasta que se especifica qué equipos tienen permiso. Asimismo, otorgan acceso al servidor según la dirección IP de origen de cada solicitud. Para configurar su firewall, cree reglas de firewall que especifiquen intervalos de direcciones IP aceptables. Puede crear reglas de firewall a nivel de servidor.

- [Reglas de firewall en Azure Database for PostgreSQL Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/concepts-hyperscale-firewall-rules)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: Defina e implemente configuraciones de seguridad estándar para los recursos y la configuración de red asociados a las instancias de Azure Database for PostgreSQL con Azure Policy. Use los alias de Azure Policy en el espacio de nombres "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de las instancias de Azure Database for PostgreSQL.

- [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

- [Ejemplos de Azure Policy para redes](https://docs.microsoft.com/azure/governance/policy/samples/#network)

- [Creación de un plano técnico de Azure](https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Control de seguridad: registro y supervisión](/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Para el registro de la auditoria del plano de control, habilite la configuración de diagnóstico del registro de actividad de Azure y envíela a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para su archivo. Con los datos del registro de actividad de Azure, puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en el nivel del plano de control para los recursos de Azure.

Además, ingiera registros a través de Azure Monitor para agregar datos de seguridad generados por Hiperescala (Citus). En Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y utilice cuentas para el almacenamiento de archivos a largo plazo. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros. 

- [Cómo habilitar la configuración de diagnóstico para el registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy)

- [Métricas en Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/concepts-hyperscale-monitoring)

- [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Hiperescala (Citus) proporciona métricas para cada nodo de un grupo de servidores. Las métricas proporcionan información detallada sobre el comportamiento de los recursos complementarios. Cada métrica se emite con una frecuencia de un minuto y tiene un historial de hasta 30 días.

Para el registro de la auditoria del plano de control, habilite la configuración de diagnóstico del registro de actividad de Azure y envíela a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para su archivo. Con los datos del registro de actividad de Azure, puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en el nivel del plano de control para los recursos de Azure.

Además, ingiera registros a través de Azure Monitor para agregar datos de seguridad generados por Hiperescala (Citus). En Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y utilice cuentas para el almacenamiento de archivos a largo plazo. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros. 

- [Métricas en Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/concepts-hyperscale-monitoring)

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy)

- [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Instrucciones**: En Azure Monitor, en el área de trabajo de Log Analytics que se usa para almacenar los registros de Hiperescala (Citus), establezca el período de retención de acuerdo con las normas de cumplimiento de su organización. Use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

- [Configuración de parámetros de retención de registros de áreas de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period)

- [Almacenamiento de registros de recursos en una cuenta de Azure Storage](https://docs.microsoft.com/azure/azure-monitor/platform/resource-logs-collect-storage)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Instrucciones**: Analice y supervise los registros de las instancias de Hiperescala (Citus) para comprobar si tienen comportamientos anómalos. Use Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos de registro. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

- [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

- [Para más información sobre Log Analytics, consulte ](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal).

- [Procedimiento para realizar consultas personalizadas en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: Puede habilitar la configuración de diagnóstico de Hiperescala (Citus) y envíe registros a un área de trabajo de Log Analytics. Puede recibir una alerta basada en las métricas de supervisión para los servicios de Azure. Use Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos de registro. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

Incorpore el área de trabajo de Log Analytics a Azure Sentinel, ya que proporciona una solución de respuesta automatizada de orquestación de seguridad (SOAR). Esto permite crear cuadernos de estrategias (soluciones automatizadas) y usarlos para corregir problemas de seguridad.

- [Métricas en Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/howto-hyperscale-alert-on-metric)

- [Configuración de las opciones de diagnóstico para el registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy)

- [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para más información, consulte [Control de seguridad: Identidad y control de acceso](/azure/security/benchmarks/security-control-identity-access-control).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: Mantenga un inventario de las cuentas de usuario que tienen acceso administrativo al plano de control (por ejemplo, Azure Portal) de las instancias de Hiperescala (Citus). Asimismo, mantenga un inventario de las cuentas administrativas que tienen acceso al plano de datos (dentro de la propia base de datos) de las instancias de Hiperescala (Citus).

Hiperescala (Citus) no admite el control de acceso basado en roles integrado, pero se pueden crear roles personalizados basados en operaciones específicas del proveedor de recursos.

Además, el motor de PostgreSQL usa roles para controlar el acceso a los objetos de base de datos, y un grupo de servidores de Hiperescala (Citus) recién creado incluye varios roles predefinidos. Para modificar los privilegios de usuario, use los comandos estándar de PostgreSQL con una herramienta como PgAdmin o psql.

- [Roles personalizados de Azure](https://docs.microsoft.com/azure/role-based-access-control/custom-roles) 

- [Operaciones de proveedores de recursos de Azure](https://docs.microsoft.com/azure/role-based-access-control/resource-provider-operations#microsoftdbforpostgresql) 

- [Seguridad en Azure Database for PostgreSQL con un único servidor](https://docs.microsoft.com/azure/postgresql/concepts-security#access-management])

- [Crear usuarios en Azure Database for PostgreSQL - Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/howto-hyperscale-create-users)

- [Conexión a la base de datos mediante psql](https://docs.microsoft.com/azure/postgresql/quickstart-create-hyperscale-portal#connect-to-the-database-using-psql)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: Azure AD no tiene el concepto de contraseñas predeterminadas. Otros recursos de Azure que requieren una contraseña obligan a crear una contraseña con requisitos de complejidad y una longitud mínima, que cambia en función del servicio. Usted es responsable de las aplicaciones de terceros y de los servicios de Marketplace que pueden usar contraseñas predeterminadas.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas que se usan para acceder a las instancias de Hiperescala (Citus). Las cuentas de administrador para administrar el recurso de Azure están asociadas a Azure Active Directory; también hay cuentas de administrador del servidor local que existen en el grupo de servidores de Hiperescala (Citus) para administrar los permisos de acceso a la base de datos. Use la administración de acceso e identidades de Azure Security Center para supervisar el número de cuentas administrativas en Azure Active Directory.

- [Descripción de identidad y acceso en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access) 

- [Crear usuarios en Azure Database for PostgreSQL - Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/howto-hyperscale-create-users)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Para obtener acceso a Azure Portal, habilite Multi-Factor Authentication (MFA) de Azure Active Directory y siga las recomendaciones de administración de identidad y acceso de Azure Security Center.

- [Procedimiento para habilitar la MFA en Azure](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted)

- [Supervisión de la identidad y el acceso en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access)


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: utilice estaciones de trabajo de acceso con privilegios (PAW) con Multi-Factor Authentication (MFA) configurado para iniciar sesión en recursos de Azure y configurarlos.

- [Más información sobre las estaciones de trabajo con privilegios de acceso](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Procedimiento para habilitar la MFA en Azure](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-alert-on-account-login-behavior-deviation"></a>3.7: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Use Privileged Identity Management (PIM) de Azure Active Directory (AD) para la generación de registros y alertas cuando se produzca una actividad sospechosa o insegura en el entorno.

Use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

- [Cómo implementar Privileged Identity Management (PIM)](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan)

- [Información sobre las detecciones de riesgo de Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: Use ubicaciones con nombre de acceso condicional para permitir al portal y a Azure Resource Manager el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

- [Configuración de ubicaciones con nombre en Azure](https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: Use Azure Active Directory (A D) como sistema central de autenticación y autorización para administrar los recursos de PostgreSQL. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

Los usuarios de un grupo de servidores de Hiperescala (Citus) no se pueden asociar directamente a cuentas de Azure Active Directory. Para modificar los privilegios de usuario para el acceso al objeto de base de datos, use los comandos estándar de PostgreSQL con herramientas como PgAdmin o psql.

- [Modificación de los privilegios para los roles de usuario](https://docs.microsoft.com/azure/postgresql/howto-hyperscale-create-users#how-to-modify-privileges-for-user-role)

- [Creación y configuración de una instancia de AAD](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Revise y reconcilie el acceso para los usuarios que tienen acceso a la base de datos local, así como mediante Azure Active Directory para administrar los recursos de PostgreSQL.

Para los usuarios con acceso para administrar los recursos de Azure de la base de datos, revise los registros de Azure Active Directory (AD) para ayudar a detectar cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficaz la pertenencia a grupos, el acceso a las aplicaciones empresariales que se pueden usar para tener acceso a Hiperescala (Citus) y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica cada 90 días para asegurarse de que solo las personas adecuadas tengan acceso continuado.

- [Revisión de los usuarios de PostgreSQL y los roles asignados](https://www.postgresql.org/docs/current/database-roles.html)

- [Descripción de los informes de Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/)

- [Procedimiento para usar las revisiones de acceso de identidad de Azure](https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: Con Azure Active Directory (AD), tiene acceso a los orígenes de registro de eventos de actividad de inicio de sesión, auditoría y riesgo de Azure Active Directory (AD), que permiten la integración en cualquier herramienta de SIEM o supervisión. 

Puede simplificar este proceso creando una configuración de diagnóstico para las cuentas de usuario de Azure Active Directory y enviando los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas deseadas en el área de trabajo de Log Analytics. 

- [Integración de los registros de actividad de Azure en Azure Monitor](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12: Alerta de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Use las características de protección de identidad y detección de riesgo de Azure Active Directory para configurar respuestas automatizadas a las acciones sospechosas detectadas en el nivel de Azure Active Directory (AD). Puede habilitar las respuestas automatizadas a través de Azure Sentinel para implementar las respuestas de seguridad de su organización.

También puede hacer que Azure Sentinel ingiera los registros para investigarlos más.

- [¿Qué es Azure Active Directory Identity Protection?](https://docs.microsoft.com/azure/active-directory/identity-protection/overview-identity-protection)

- [Visualización de los inicios de sesión de riesgo de Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events)

- [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: Actualmente no disponible; la Caja de seguridad del cliente todavía no se admite en Hiperescala (Citus).

- [Lista de servicios admitidos por la Caja de seguridad del cliente](https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: no disponible actualmente

## <a name="data-protection"></a>Protección de los datos

*Para más información, consulte [Control de seguridad: Protección de datos](/azure/security/benchmarks/security-control-data-protection).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Guía**: Use etiquetas para ayudar a realizar un seguimiento de las instancias de Hiperescala (Citus) o los recursos relacionados que almacenan o procesan información confidencial.

- [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Use una combinación de roles administrativos y reglas de firewall para aislar y limitar el acceso de red a las instancias de Azure Database for PostgreSQL.

- [Creación de suscripciones adicionales de Azure](https://docs.microsoft.com/azure/billing/billing-create-subscription)

- [Creación de grupos de administración](https://docs.microsoft.com/azure/governance/management-groups/create)

- [Reglas de firewall en Azure Database for PostgreSQL: Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/concepts-hyperscale-firewall-rules)

- [Crear usuarios en Azure Database for PostgreSQL - Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/howto-hyperscale-create-users)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: Las conexiones de la aplicación cliente al nodo de coordinación de Hiperescala (Citus) requieren la seguridad de la capa de transporte (SSL) 1.2. Aplicar conexiones TLS entre el servidor de bases de datos y las aplicaciones cliente ayuda a proteger contra los ataques de tipo "Man in the middle" mediante el cifrado del flujo de datos entre el servidor y la aplicación.

Para todos los servidores Azure Database for PostgreSQL aprovisionados en Azure Portal, la obligatoriedad de las conexiones TLS está habilitada de forma predeterminada.

En algunos casos, las aplicaciones de terceros requieren un archivo de certificado local generado a partir de un archivo de certificado (.cer) de una entidad de certificación (CA) de confianza para conectarse de forma segura.

- [Configuración de TLS en Azure Database for PostgreSQL: Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/concepts-hyperscale-ssl-connection-security)

- [Aplicaciones que requieren la verificación del certificado para la conectividad TLS](https://docs.microsoft.com/azure/postgresql/concepts-hyperscale-ssl-connection-security)



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6: Uso del control de acceso basado en rol para controlar el acceso a los recursos

**Guía**: Use el control de acceso basado en rol (RBAC) de Azure para controlar el acceso al plano de control de Hiperescala (Citus) (por ejemplo, Azure Portal). RBAC no afecta a los permisos de usuario de la base de datos.

Para modificar los privilegios de usuario a nivel de base de datos, use los comandos estándar de PostgreSQL con una herramienta como PgAdmin o psql.

- [Configuración de RBAC en Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal)

- [Crear usuarios en Azure Database for PostgreSQL - Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/howto-hyperscale-create-users)


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**:  
Al menos una vez al día, Azure Database for PostgreSQL - Hiperescala (Citus) realiza copias de seguridad de instantáneas de los archivos de datos y del registro de transacciones de la base de datos. Las copias de seguridad permiten restaurar un servidor a un momento dado dentro del período de retención. (El período de retención es actualmente de 35 días para todos los clústeres). Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits. La oferta Hiperescala (Citus) de PostgreSQL usa claves administradas por Microsoft para el cifrado.

- [Copia de seguridad y restauración en Azure Database for PostgreSQL: Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/concepts-hyperscale-backup)



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure con el fin de crear alertas para cuando se produzcan cambios en las instancias de producción de Hiperescala (Citus) y otros recursos críticos o relacionados.

- [Creación de alertas para los eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Control de seguridad: Administración de vulnerabilidades](/azure/security/benchmarks/security-control-vulnerability-management).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: Actualmente, no está disponible. Azure Security Center todavía no admite la evaluación de vulnerabilidades para Azure Database for PostgreSQL Hiperescala (Citus).

- [Cobertura de características para los servicios de PaaS de Azure en Azure Security Center](https://docs.microsoft.com/azure/security-center/features-paas)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: no disponible actualmente

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para más información, consulte [Control de seguridad: Administración de recursos y del inventario](/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: Use Azure Resource Graph para consultar y detectar todos los recursos (incluidas las instancias de Hiperescala [Citus]) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

- [Creación de consultas con Azure Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)

- [Visualización de las suscripciones de Azure](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

- [Descripción de Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: Aplique etiquetas a las instancias de Hiperescala (Citus) y otros recursos relacionados que proporcionen metadatos para organizarlos lógicamente en una taxonomía.

- [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de las instancias de Hiperescala (Citus) y los recursos relacionados. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

- [Creación de suscripciones adicionales de Azure](https://docs.microsoft.com/azure/billing/billing-create-subscription)

- [Creación de grupos de administración](https://docs.microsoft.com/azure/governance/management-groups/create)

- [Creación y uso de etiquetas](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Instrucciones**: use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

Además, use Azure Resource Graph para consultar o detectar recursos dentro de sus suscripciones.

- [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

- [Creación de consultas con Azure Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Instrucciones**: use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Además, use Azure Resource Graph para consultar o detectar recursos dentro de sus suscripciones.

- [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

- [Creación de consultas con Azure Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Instrucciones**: use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos
- Tipos de recursos permitidos

- [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

- [Denegación de un tipo de recurso específico con Azure Policy](https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: Use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management. De este modo, se puede impedir la creación y los cambios en los recursos dentro de un entorno de alta seguridad, como instancias de Hiperescala (Citus) que contienen información confidencial.

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Control de seguridad: Configuración segura](/azure/security/benchmarks/security-control-secure-configuration).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Defina e implemente configuraciones de seguridad estándar para las instancias de Hiperescala (Citus) con Azure Policy. Use Azure Policy para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de las instancias de Azure Database for PostgreSQL.

Además, Azure Resource Manager tiene la capacidad de exportar la plantilla en notación de objetos JavaScript (JSON), que se debe revisar para asegurarse de que las configuraciones cumplan o superen los requisitos de seguridad de la organización. 

- [Visualización de los alias de Azure Policy disponibles](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

- [Exportación de uno y varios recursos a una plantilla en Azure Portal](https://docs.microsoft.com/azure/azure-resource-manager/templates/export-template-portal) 



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Instrucciones**: Use la directiva de Azure [denegar] e [implementar si no existe] para aplicar la configuración segura en los recursos de Azure.  Además, puede usar plantillas de Azure Resource Manager para mantener la configuración de seguridad de los recursos de Azure que requiere su organización. 

- [Descripción de los efectos de Azure Policy](https://docs.microsoft.com/azure/governance/policy/concepts/effects)

- [Creación y administración de directivas para aplicar el cumplimiento](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

- [Información general sobre plantillas de Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/templates/overview)



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Si usa definiciones personalizadas de Azure Policy para las instancias de Hiperescala (Citus) y los recursos relacionados, use Azure Repos para almacenar y administrar el código de forma segura.

- [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Documentación de Azure Repos](https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Instrucciones**: Use la directiva de Azure [denegar] e [implementar si no existe] para aplicar la configuración segura en los recursos de Azure.  Además, puede usar plantillas de Azure Resource Manager para mantener la configuración de seguridad de los recursos de Azure que requiere su organización. 

- [Descripción de los efectos de Azure Policy](https://docs.microsoft.com/azure/governance/policy/concepts/effects)

- [Creación y administración de directivas para aplicar el cumplimiento](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

- [Información general sobre plantillas de Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/templates/overview)



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: use alias de Azure Policy en el espacio de nombres "Microsoft.DBforPostgreSQL" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Use la directiva de Azure [auditar], [denegar] e [implementar si no existe] para aplicar automáticamente las configuraciones de las instancias de Azure Database for PostgreSQL y los recursos relacionados.

- [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: Azure Database for PostgreSQL: actualmente Hiperescala (Citus) no admite directamente identidades administradas. Al crear el servidor de Azure Database for PostgreSQL, se deben proporcionar las credenciales de un usuario administrador. Puede crear roles de usuario adicionales en la interfaz de Azure Portal.

- [Creación de una instancia de Azure Database for PostgreSQL: Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/quickstart-create-hyperscale-portal#create-an-azure-database-for-postgresql---hyperscale-citus)

- [Creación de roles de usuario adicionales](https://docs.microsoft.com/azure/postgresql/howto-hyperscale-create-users#how-to-create-additional-user-roles)


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: no disponible actualmente

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

- [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Control de seguridad: defensa contra malware](/azure/security/benchmarks/security-control-malware-defense).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (como Hiperescala [Citus]), pero no se ejecuta en el contenido del cliente.

Examine previamente el contenido que se carga en recursos de Azure que no son de proceso, como App Service, Data Lake Storage, Blob Storage, Azure Database for PostgreSQL, etc. Microsoft no tiene acceso a los datos de estas instancias.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Control de seguridad: recuperación de datos](/azure/security/benchmarks/security-control-data-recovery).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: Azure Database for PostgreSQL: Hiperescala (Citus) crea automáticamente copias de seguridad de cada nodo y las almacena en un almacenamiento con redundancia local. Las copias de seguridad se pueden usar para restaurar el clúster de Hiperescala (Citus) a una hora especificada.

- [Copia de seguridad y restauración en Azure Database for PostgreSQL: Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/concepts-hyperscale-backup)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Instrucciones**: Al menos una vez al día, Azure Database for PostgreSQL realiza copias de seguridad de instantáneas de los archivos de datos y del registro de transacciones de la base de datos. Las copias de seguridad permiten restaurar un servidor a un momento dado dentro del período de retención. El período de retención es actualmente de 35 días para todos los clústeres. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits.

En las regiones de Azure que admiten zonas de disponibilidad, las instantáneas de copia de seguridad se almacenan en tres zonas de disponibilidad. Siempre que haya al menos una zona de disponibilidad en línea, el clúster de Hiperescala (Citus) se puede restaurar.

- [Copia de seguridad y restauración en Azure Database for PostgreSQL: Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/concepts-hyperscale-backup)


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: En Azure Database for PostgreSQL, la restauración de un clúster de Hiperescala (Citus) crea un clúster nuevo a partir de las copias de seguridad de los nodos originales. Puede restaurar un clúster a un momento dado cualquiera de los últimos 35 días. El proceso de restauración crea un nuevo clúster en la misma región, suscripción y grupo de recursos de Azure que el clúster original. La nueva configuración del clúster es la misma que la configuración del clúster original: el mismo número de nodos, el número de núcleos virtuales, el tamaño de almacenamiento y los roles de usuario.

La configuración del firewall y los parámetros del servidor de PostgreSQL no se conservan del grupo de servidores original sino que se restablecen a los valores predeterminados. El firewall impedirá todas las conexiones. Tendrá que ajustar manualmente estos valores después de la restauración.

- [Copia de seguridad y restauración en Azure Database for PostgreSQL: Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/concepts-hyperscale-backup)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Instrucciones**: Los clústeres eliminados de Hiperescala (Citus) no se pueden restaurar. Si elimina el clúster, todos los nodos que pertenecen a este se eliminan y no se pueden recuperar. Para proteger los recursos del clúster, después de la implementación, de eliminaciones accidentales o cambios inesperados, los administradores pueden aprovechar los bloqueos de administración.

- [Copia de seguridad y restauración en Azure Database for PostgreSQL: Hiperescala (Citus)](https://docs.microsoft.com/azure/postgresql/concepts-hyperscale-backup)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Control de seguridad: Respuesta a los incidentes](/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia. 

- [Configuración de las automatizaciones de flujos de trabajo en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide) 

- [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/) 

- [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/) 

- [El cliente también puede usar la guía de control de incidentes de seguridad de equipos de NIST como referencia para la creación de su propio plan de respuesta a incidentes](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta. 

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario. 

- [Consulte la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos del cliente.  Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas. 

- [Establecimiento del contacto de seguridad de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: Exporte sus alertas y recomendaciones de Azure Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Sentinel. 

- [Configuración de la exportación continua](https://docs.microsoft.com/azure/security-center/continuous-export) 

- [Transmisión de alertas a Azure Sentinel](https://docs.microsoft.com/azure/sentinel/connect-azure-security-center)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Instrucciones**: use la característica de automatización del flujo de trabajo de Azure Security Center para desencadenar automáticamente respuestas mediante "Logic Apps" en las alertas y recomendaciones de seguridad. 

- [Configuración de la automatización de flujo de trabajo y Logic Apps](https://docs.microsoft.com/azure/security-center/workflow-automation)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft: https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1 

- [Puede encontrar más información sobre la estrategia y ejecución de Microsoft en las pruebas de penetración del equipo rojo y los sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft aquí.](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](/azure/security/benchmarks/security-baselines-overview).
