---
title: Línea de base de seguridad de Azure para Azure Database for MariaDB
description: Línea de base de seguridad de Azure para Azure Database for MariaDB
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 03/23/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: f067d4c777bb9a49ac2cc50595046249479a8f7a
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393843"
---
# <a name="azure-security-baseline-for-azure-database-for-mariadb"></a>Línea de base de seguridad de Azure para Azure Database for MariaDB

La línea de base de seguridad de Azure para Azure Database for MariaDB contiene recomendaciones que le ayudarán a mejorar la posición de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](https://docs.microsoft.com/azure/security/benchmarks/overview), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para obtener más información, consulte [Introducción a las líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).

## <a name="network-security"></a>Seguridad de redes

*Para obtener más información, consulte [Control de seguridad: seguridad de red](https://docs.microsoft.com/azure/security/benchmarks/security-control-network-security).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1: Proteja los recursos mediante grupos de seguridad de red o Azure Firewall en su red virtual

**Guía**: Configuración de Private Link para Azure Database for MariaDB con puntos de conexión privados. Private Link permite conectarse a varios servicios PaaS en Azure mediante un punto de conexión privado. En esencia, Azure Private Link incorpora los servicios de Azure dentro de su red virtual privada (VNet). El tráfico entre la red virtual y la instancia de MariaDB viaja por la red troncal de Microsoft.

Como alternativa, puede usar puntos de conexión de servicio de red virtual para proteger y limitar el acceso de red a las implementaciones de Azure Database for MariaDB. Las reglas de red virtual son una característica de seguridad de firewall que controla si la instancia de Azure Database for MariaDB acepta las comunicaciones que se envían desde subredes específicas en redes virtuales.

También puede proteger su instancia de Azure Database for MariaDB con reglas de firewall. El firewall de servidor impide todo acceso al servidor de bases de datos, excepto a aquellos equipos a los que haya concedido permiso. Para configurar su firewall, cree reglas de firewall que especifiquen intervalos de direcciones IP aceptables. Puede crear reglas de firewall a nivel de servidor.

Configuración de Private Link para Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/howto-configure-privatelink-portal

Creación y administración de reglas y puntos de conexión de servicio de red virtual de Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/howto-manage-vnet-portal

Configuración de reglas de firewall para Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-firewall-rules

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2: Supervise y registre la configuración y el tráfico de redes virtuales, subredes y NIC

**Guía**: Cuando el servidor de Azure Database for MariaDB está protegido en un punto de conexión privado, puede implementar máquinas virtuales en la misma red virtual. Puede usar un grupo de seguridad de red (NSG) para reducir el riesgo de la filtración de datos. Habilite los registros de flujo de NSG y envíe registros a una cuenta de almacenamiento para la auditoría del tráfico. También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Configuración de Private Link para Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/howto-configure-privatelink-portal

Cómo habilitar los registros de flujo de NSG: https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal Habilitación y uso del Análisis de tráfico: https://docs.microsoft.com/azure/network-watcher/traffic-analytics



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: Uso de Advanced Threat Protection para Azure Database for MariaDB. Advanced Threat Protection detecta actividades anómalas que indican intentos poco habituales y posiblemente dañinos de acceder a las bases de datos o aprovecharse de ellas.

Habilite el estándar de DDoS Protection en las redes virtuales asociadas a las instancias de Azure Database for MariaDB a fin de protegerse de los ataques DDoS. Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

Configuración de Advanced Threat Protection para Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/howto-database-threat-protection-portal

Configuración de la protección contra DDoS: https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5: Registre los paquetes de red y registros de flujo

**Guía**: Cuando el servidor de Azure Database for MariaDB está protegido en un punto de conexión privado, puede implementar máquinas virtuales en la misma red virtual. A continuación, puede configurar un grupo de seguridad de red (NSG) para reducir el riesgo de la filtración de datos. Habilite los registros de flujo de NSG y envíe registros a una cuenta de almacenamiento para la auditoría del tráfico. También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Cómo habilitar los registros de flujo de NSG: https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal Habilitación y uso del Análisis de tráfico: https://docs.microsoft.com/azure/network-watcher/traffic-analytics



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implemente sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Guía**: Uso de Advanced Threat Protection para Azure Database for MariaDB. Advanced Threat Protection detecta actividades anómalas que indican intentos poco habituales y posiblemente dañinos de acceder a las bases de datos o aprovecharse de ellas.
Configuración de Advanced Threat Protection para Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/howto-database-threat-protection-portal


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: En el caso de los recursos que necesitan acceso a las instancias de Azure Database for MariaDB, use las etiquetas de servicio de red virtual con el fin de definir controles de acceso a la red en grupos de seguridad de red o Azure Firewall. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, SQL.WestUs) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.
Nota: Azure Database for MariaDB usa la etiqueta de servicio "Microsoft.Sql".

Para obtener más información sobre el uso de etiquetas de servicio, consulte: https://docs.microsoft.com/azure/virtual-network/service-tags-overview Descripción del uso de etiquetas de servicio para Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-data-access-security-vnet#terminology-and-description



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Instrucciones**: Defina e implemente configuraciones de seguridad estándar para los recursos y la configuración de red asociados a las instancias de Azure Database for MariaDB con Azure Policy. Use alias de Azure Policy en los espacios de nombres "Microsoft.DBforMariaDB" y "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de las instancias de Azure Database for MariaDB. También puede usar las definiciones de directiva integradas relacionadas con las redes o las instancias de Azure Database for MariaDB, como:

- Se debe habilitar DDoS Protection estándar

- El punto de conexión privado debe estar habilitado para servidores MariaDB

- El servidor MariaDB debe usar un punto de conexión del servicio de red virtual

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Ejemplos de Azure Policy para redes: https://docs.microsoft.com/azure/governance/policy/samples/

Creación de un plano técnico de Azure: https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: Use etiquetas para los recursos relacionados con la seguridad de red y el flujo de tráfico de las instancias de MariaDB para proporcionar metadatos y organización lógica.

Use cualquiera de las definiciones de Azure Policy integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor", para asegurarse de que todos los recursos se crean con etiquetas y para notificarle los recursos no etiquetados existentes.

Puede usar Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas.

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: Use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en dichos recursos relacionados con las instancias de Azure Database for MariaDB. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.
Visualización y recuperación de eventos del registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view Creación de alertas en Azure Monitor: https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para obtener más información, consulte [Control de seguridad: registro y supervisión](https://docs.microsoft.com/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Instrucciones**: Microsoft mantiene el origen de la hora usado en los recursos de Azure, como Azure Database for MariaDB, para las marcas de tiempo de los registros.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Habilite la configuración de diagnóstico y los registros de servidor y de ingesta para agregar los datos de seguridad generados por las instancias de Azure Database for MariaDB. En Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y utilice cuentas de Azure Storage para el almacenamiento de archivos a largo plazo. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.
Configuración y acceso a los registros de servidor de Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-server-logs

Configuración y acceso a los registros de auditoría de Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/howto-configure-audit-logs-portal Incorporación de Azure Sentinel: https://docs.microsoft.com/azure/sentinel/quickstart-onboard



**Supervisión de Azure Security Center**: No disponible

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Instrucciones**: Habilite Configuración de diagnóstico en las instancias de Azure Database for MariaDB para poder acceder a los registros de auditoría, seguridad y diagnóstico. Asegúrese de habilitar específicamente el registro de auditoría de MariaDB. Los registros de actividad, que están disponibles automáticamente, incluyen el origen del evento, la fecha, el usuario, la marca de tiempo, las direcciones de origen y de destino, y otros elementos útiles. También puede habilitar la configuración de diagnóstico del registro de actividad de Azure y enviar los registros a la misma área de trabajo de Log Analytics o cuenta de almacenamiento.

Configuración y acceso a los registros de servidor de Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-server-logs Configuración y acceso a los registros de auditoría de Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/howto-configure-audit-logs-portal Establecimiento de la configuración de diagnóstico para el registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy



**Supervisión de Azure Security Center**: No disponible

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, en el área de trabajo de Log Analytics que se usa para almacenar los registros de Azure Database for MariaDB, establezca el período de retención de acuerdo con las normas de cumplimiento de su organización. Use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.
Configuración de parámetros de retención de registros de áreas de trabajo de Log Analytics: https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period Almacenamiento de los registros de recursos en una cuenta de Azure Storage: https://docs.microsoft.com/azure/azure-monitor/platform/resource-logs-collect-storage



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Guía**: Analice y supervise los registros de las instancias de MariaDB para comprobar si tienen comportamientos anómalos. Use el área de trabajo de Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos del registro. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

Incorporación de Azure Sentinel: https://docs.microsoft.com/azure/sentinel/quickstart-onboard

Para obtener más información acerca del área de trabajo de Log Analytics, consulte: https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal

Realización de consultas personalizadas en Azure Monitor: https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: Habilite Advanced Threat Protection para MariaDB. Advanced Threat Protection para Azure Database for MariaDB detecta actividades anómalas que indican intentos inusuales y potencialmente dañinos de acceso o ataque a las bases de datos.

Además, puede habilitar los registros de servidor y la configuración de diagnóstico para MariaDB y enviar registros a un área de trabajo de Log Analytics. Incorpore el área de trabajo de Log Analytics a Azure Sentinel, ya que proporciona una solución de respuesta automatizada de orquestación de seguridad (SOAR). Esto permite crear cuadernos de estrategias (soluciones automatizadas) y usarlos para corregir problemas de seguridad.

Habilitación de Advanced Threat Protection para MariaDB: https://docs.microsoft.com/azure/mariadb/howto-database-threat-protection-portal

Configuración y acceso a los registros de servidor para MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-server-logs

Configuración y acceso a los registros de auditoría para MariaDB: https://docs.microsoft.com/azure/mariadb/howto-configure-audit-logs-portal

Incorporación de Azure Sentinel: https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: No aplicable; MariaDB no procesa ni genera registros relacionados con antimalware.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: No aplicable; MariaDB no procesa ni genera registros relacionados con DNS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Guía**: No aplicable; la prueba comparativa está pensada para recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para obtener más información, consulte [Control de seguridad: identidad y control de acceso](https://docs.microsoft.com/azure/security/benchmarks/security-control-identity-access-control).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: Mantenga un inventario de las cuentas de usuario que tienen acceso administrativo al plano de administración (Azure Portal/Azure Resource Manager) de las instancias de MariaDB. Así mismo, mantenga un inventario de las cuentas administrativas que tienen acceso al plano de datos de las instancias de MariaDB. (Al crear el servidor de MariaDB, se deben proporcionar las credenciales de un usuario administrador. Este administrador se puede usar para crear más usuarios de MariaDB).

Descripción de la administración de acceso para MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-security#access-management

Información sobre los roles integrados de Azure para las suscripciones de Azure: https://docs.microsoft.com/azure/role-based-access-control/built-in-roles


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Instrucciones**: Azure Active Directory no tiene el concepto de contraseñas predeterminadas.

Tras la creación del recurso de MariaDB mismo, Azure fuerza la creación de un usuario administrativo con una contraseña segura. Sin embargo, una vez creada la instancia de MariaDB, puede usar la primera cuenta de administrador de servidor que creó a fin de crear usuarios adicionales y concederles acceso administrativo. Al crear estas cuentas, asegúrese de configurar una contraseña segura distinta para cada una.

Creación de cuentas adicionales para MariaDB: https://docs.microsoft.com/azure/mariadb/howto-create-users


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas que tengan acceso a las instancias de MariaDB. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

Descripción de identidad y acceso en Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Guía**: El acceso del plano de datos a MariaDB se controla mediante identidades almacenadas en la base de datos y no admite SSO. El acceso del plano de control para MariaDB está disponible a través de la API de REST y es compatible con SSO. Para autenticarse, establezca el encabezado de autorización de las solicitudes en JSON Web Token, que se obtiene de Azure Active Directory (AAD).

Descripción de la API de REST de Azure Database for MariaDB: https://docs.microsoft.com/rest/api/mariadb/

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Instrucciones**: Habilite la MFA de Azure AD y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

Cómo habilitar la MFA en Azure: https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Cómo supervisar la identidad y el acceso en Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: Utilice PAW (estaciones de trabajo de acceso privilegiado) con la MFA configurada para iniciar la sesión y configurar recursos de Azure.

Más información sobre las estaciones de trabajo con privilegios de acceso: https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations

Cómo habilitar la MFA en Azure: https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7: Registre y alerte de la actividad sospechosa desde cuentas administrativas

**Guía**: Habilite Advanced Threat Protection para MariaDB con el fin de generar alertas para actividades sospechosas.

Así mismo, puede usar Azure AD Privileged Identity Management (PIM) para la generación de registros y alertas cuando se produzca actividad sospechosa o insegura en el entorno. Use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

Configuración de Advanced Threat Protection para MariaDB: https://docs.microsoft.com/azure/mariadb/howto-database-threat-protection-portal

Implementación de Privileged Identity Management (PIM): https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan

Información sobre las detecciones de riesgo de Azure AD: https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: Use ubicaciones con nombre de acceso condicional para permitir el acceso solo desde agrupaciones lógicas de intervalos de direcciones IP o países o regiones específicos, con el fin de limitar el acceso a los recursos de Azure, como MariaDB.

Configuración de ubicaciones con nombre en Azure: https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Instrucciones**: Use Azure Active Directory (AAD) como el sistema central de autenticación y autorización. AAD protege los datos mediante un fuerte cifrado para los datos en reposo y en tránsito. AAD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

No se puede usar la autenticación de Azure AD para el acceso directo al plano de datos de MariaDB; no obstante, se pueden usar credenciales de Azure AD para la administración a nivel del plano de administración (por ejemplo, Azure Portal) para controlar las cuentas de administrador de MariaDB.

Actualización de la contraseña del administrador en MariaDB: https://docs.microsoft.com/azure/mariadb/howto-create-manage-server-portal#update-admin-password

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Revise los registros de Azure Active Directory a fin de detectar cuentas obsoletas, como aquellas con roles administrativos de MariaDB. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficaz la pertenencia a grupos, el acceso a las aplicaciones empresariales que se pueden usar para tener acceso a MariaDB y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica cada 90 días para asegurarse de que solo los usuarios adecuados tengan acceso continuado.

Descripción de los informes de Azure AD: https://docs.microsoft.com/azure/active-directory/reports-monitoring/

Cómo usar las revisiones de acceso de identidad de Azure: https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11: Supervise los intentos de acceso a cuentas desactivadas

**Guía**: Habilite la configuración de diagnóstico para MariaDB y Azure Active Directory, y envíe todos los registros a un área de trabajo de Log Analytics. Configure las alertas que desee (como intentos de autenticación con errores) en el área de trabajo de Log Analytics.

Configuración y acceso a los registros de servidor para MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-server-logs

Configuración y acceso a los registros de auditoría para MariaDB: https://docs.microsoft.com/azure/mariadb/howto-configure-audit-logs-portal

Integración de los registros de actividad de Azure en Azure Monitor: https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

**Supervisión de Azure Security Center**: No disponible

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Habilite Advanced Threat Protection para MariaDB. Advanced Threat Protection para Azure Database for MariaDB detecta actividades anómalas que indican intentos inusuales y potencialmente dañinos de acceso o ataque a las bases de datos.

Use las características de detección de riesgo y Azure Active Directory Identity Protection para configurar respuestas automatizadas a las acciones sospechosas detectadas. Puede habilitar las respuestas automatizadas a través de Azure Sentinel para implementar las respuestas de seguridad de su organización.

Habilitación de Advanced Threat Protection para MariaDB: https://docs.microsoft.com/azure/mariadb/howto-database-threat-protection-portal

Procedimiento para configurar y habilitar las directivas de riesgo de Identity Protection: https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

Visualización de los inicios de sesión de riesgo de Azure AD: https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

Incorporación de Azure Sentinel: https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervisión de Azure Security Center**: No disponible

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: No aplicable; la Caja de seguridad del cliente todavía no se admite en Azure Database for MariaDB.

Lista de servicios admitidos por la Caja de seguridad del cliente: https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-protection"></a>Protección de datos

*Para obtener más información, consulte [Control de seguridad: protección de datos](https://docs.microsoft.com/azure/security/benchmarks/security-control-data-protection).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: Use etiquetas para ayudar a realizar un seguimiento de las instancias de Azure Database for MariaDB o los recursos relacionados que almacenan o procesan información confidencial.

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Use una combinación de Private Link, puntos de conexión de servicio y reglas de firewall de MariaDB para aislar y limitar el acceso de red a las instancias de MariaDB.

Creación de suscripciones adicionales de Azure: https://docs.microsoft.com/azure/billing/billing-create-subscription

Creación de grupos de administración: https://docs.microsoft.com/azure/governance/management-groups/create

Configuración de Private Link para Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-data-access-security-private-link

Configuración de puntos de conexión de servicio para Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/howto-manage-vnet-portal

Configuración de reglas de firewall para Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-firewall-rules

**Supervisión de Azure Security Center**: No disponible

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Al usar VM de Azure para tener acceso a instancias de MariaDB, use Private Link, configuraciones de red de MariaDB, grupos de seguridad de red y etiquetas de servicio para mitigar la posibilidad de que se puedan filtrar los datos.

Microsoft administra la infraestructura subyacente para MariaDB y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

Mitigación de la filtración de datos para Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-data-access-security-private-link

Descripción de la protección de datos de los clientes en Azure: https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: Azure Database for MariaDB permite conectar el servidor de Azure Database for MariaDB con aplicaciones cliente mediante Seguridad de la capa de transporte (TLS), llamado anteriormente Capa de sockets seguros (SSL). Aplicar conexiones TLS entre el servidor de bases de datos y las aplicaciones cliente ayuda a proteger contra los ataques de tipo "Man in the middle" mediante el cifrado del flujo de datos entre el servidor y la aplicación. En Azure Portal, asegúrese de que esté habilitada la opción "Aplicar conexión SSL" para todas las instancias de MariaDB.

Configuración del cifrado en tránsito para MariaDB: https://docs.microsoft.com/azure/mariadb/howto-configure-ssl

**Supervisión de Azure Security Center**: No disponible

**Responsabilidad**: Compartido

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: Las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Database for MariaDB. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

Descripción de la protección de datos de los clientes en Azure: https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervisión de Azure Security Center**: No disponible

**Responsabilidad**: Compartido

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos

**Guía**: Use el control de acceso basado en roles de Azure para controlar el acceso a Azure Database para el plano de administración de MariaDB (Azure Portal/Azure Resource Manager). Para el acceso al plano de datos (dentro de la base de datos), use consultas SQL para crear usuarios y configurar permisos de usuario.

Configuración de Azure RBAC: https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal

Configuración del acceso de usuario con SQL para MariaDB: https://docs.microsoft.com/azure/mariadb/howto-create-users

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: No aplicable; esta guía está pensada para recursos de proceso.

Microsoft administra la infraestructura subyacente para MariaDB y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

Descripción de la protección de datos de los clientes en Azure: https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Instrucciones**: El servicio Azure Database for MariaDB usa el módulo criptográfico con validación FIPS 140-2 para el cifrado del almacenamiento de los datos en reposo. Los datos, incluidas las copias de seguridad, se cifran en el disco, a excepción de los archivos temporales creados mientras se ejecutan las consultas. El servicio usa el cifrado AES de 256 bits que se incluye en el cifrado de almacenamiento de Azure y las claves las administra el sistema. El cifrado de almacenamiento siempre está activado y no se puede deshabilitar.

Descripción del cifrado en reposo para MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-security

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure para crear alertas para cuando se produzcan cambios en las instancias de producción de Azure Database for MariaDB y otros recursos críticos o relacionados.

Creación de alertas para eventos de registro de actividad de Azure: https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para obtener más información, consulte [Control de seguridad: administración de vulnerabilidades](https://docs.microsoft.com/azure/security/benchmarks/security-control-vulnerability-management).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: Actualmente, no está disponible. Azure Security Center todavía no admite la evaluación de vulnerabilidades para servidores de Azure Database for MariaDB.


**Supervisión de Azure Security Center**: Sí

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

**Instrucciones**: Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten servidores de Azure Database for MariaDB.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para obtener más información, consulte [Control de seguridad: inventario y administración de recursos](https://docs.microsoft.com/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-use-azure-asset-discovery"></a>6.1: Uso de la detección de recursos de Azure

**Guía**: Use Azure Resource Graph para consultar y detectar todos los recursos (incluidos servidores de Azure Database for MariaDB) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

Creación de consultas con Azure Resource Graph: https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Visualización de las suscripciones de Azure: https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0

Descripción de Azure RBAC: https://docs.microsoft.com/azure/role-based-access-control/overview

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: Aplique etiquetas al servidor de Azure Database for MariaDB y otros recursos relacionados que proporcionen metadatos para organizarlos lógicamente en una taxonomía.

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: Use etiquetados, grupos de administración y suscripciones independientes, si procede, para organizar los servidores de Azure Database for MariaDB y los recursos relacionados y hacer un seguimiento de ellos. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

Creación de suscripciones adicionales de Azure: https://docs.microsoft.com/azure/billing/billing-create-subscription

Creación de grupos de administración: https://docs.microsoft.com/azure/governance/management-groups/create

Creación y uso de etiquetas: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4: Mantenimiento de un inventario de los recursos de Azure aprobados y títulos de software

**Guía**: No aplicable; esta recomendación está destinada a los recursos de proceso y a Azure en su conjunto.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

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

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: No aplicable; esta recomendación está destinada a los recursos de proceso y a Azure en su conjunto.



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Instrucciones**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Denegación de un tipo de recurso específico con Azure Policy: https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="610-implement-approved-application-list"></a>6.10: Implementación de la lista de aplicaciones aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="611-divlimit-users-ability-to-interact-with-azure-resources-manager-via-scriptsdiv"></a>6.11: <div>Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager mediante scripts</div>

**Instrucciones**: Use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management. De este modo, se puede impedir la creación y los cambios en los recursos dentro de un entorno de alta seguridad, como servidores de Azure Database for MariaDB que contienen información confidencial.

Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager: https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management



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

*Para obtener más información, consulte [Control de seguridad: configuración segura](https://docs.microsoft.com/azure/security/benchmarks/security-control-secure-configuration).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Defina e implemente configuraciones de seguridad estándar para las instancias de Azure Database for MariaDB con Azure Policy. Use alias de Azure Policy en el espacio de nombres "Microsoft.DBforMariaDB" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de las instancias de Azure Database for MariaDB. También puede usar las definiciones de directiva integradas relacionadas con los servidores de Azure Database for MariaDB, como:

- La copia de seguridad con redundancia geográfica debe estar habilitada para Azure Database for MariaDB.

Visualización de los alias de Azure Policy disponibles: https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Utilice las directivas [deny] y [deploy if not exist] de Azure Policy para aplicar una configuración segura en los recursos de Azure.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Descripción de los efectos de Azure Policy: https://docs.microsoft.com/azure/governance/policy/concepts/effects



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Si usa definiciones personalizadas de Azure Policy para los servidores de Azure Database for MariaDB y los recursos relacionados, use Azure Repos para almacenar y administrar el código de forma segura.

Cómo almacenar código en Azure DevOps: https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops

Documentación de Azure Repos: https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7: Implemente herramientas de administración de configuración del sistema

**Guía**: Use alias de Azure Policy en el espacio de nombres "Microsoft.DBforMariaDB" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8: Implemente herramientas de administración de configuración del sistema para sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9: Implemente la supervisión de configuración automatizada para servicios de Azure

**Guía**: Use alias de Azure Policy en el espacio de nombres "Microsoft.DBforMariaDB" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Use [audit], [deny] y [deployifnotexist] de Azure Policy para aplicar automáticamente las configuraciones de las instancias de Azure Database for MariaDB y los recursos relacionados.

Cómo configurar y administrar Azure Policy: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: En el caso de Azure Virtual Machines o aplicaciones web que se ejecutan en Azure App Service y se usan para tener acceso a los servidores de Azure Database for MariaDB, use Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de secretos de Azure Database for MariaDB. Asegúrese de que la eliminación temporal de Key Vault esté habilitada.

Cómo integrar las identidades administradas de Azure: https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

Cómo crear un almacén de claves: https://docs.microsoft.com/azure/key-vault/general/quick-create-portal

Autenticación en Azure Key Vault: https://docs.microsoft.com/azure/key-vault/general/authentication

Asignación de una directiva de acceso de Key Vault: https://docs.microsoft.com/azure/key-vault/general/assign-access-policy-portal

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: El servidor de Azure Database for MariaDB actualmente no admite la autenticación de Azure Active Directory para tener acceso a las bases de datos.  Al crear el servidor de Azure Database for MariaDB, se deben proporcionar las credenciales de un usuario administrador. Este administrador se puede usar para crear más usuarios de MariaDB.  

En el caso de Azure Virtual Machines o aplicaciones web que se ejecutan en Azure App Service y se usan para tener acceso al servidor de Azure Database for MariaDB, use Managed Service Identity junto con Azure Key Vault para almacenar y recuperar las credenciales del servidor de Azure Database for MariaDB.  Asegúrese de que la eliminación temporal de Key Vault esté habilitada.

Use las identidades administradas para proporcionar a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory (AD). Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluyendo Key Vault, sin necesidad de credenciales en el código. Cómo configurar las identidades administradas: https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm. Cómo integrar las identidades administradas de Azure: https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity.



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault. 

Cómo configurar el escáner de credenciales: https://secdevtools.azurewebsites.net/helpcredscan.html

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="malware-defense"></a>Defensa contra malware

*Para obtener más información, consulte [Control de seguridad: defensa contra malware](https://docs.microsoft.com/azure/security/benchmarks/security-control-malware-defense).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Use software antimalware administrado centralmente

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure App Service), pero no se ejecuta en el contenido del cliente.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Instrucciones**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, servidores de Azure Database for MariaDB), pero no se ejecuta en el contenido del cliente.

Examine previamente el contenido que se carga en recursos de Azure que no son de proceso, como App Service, Data Lake Storage, Blob Storage, servidores de Azure Database for MariaDB, etc. Microsoft no tiene acceso a los datos de estas instancias.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, servidores de Azure Database for MariaDB), pero no se ejecuta en el contenido del cliente.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-recovery"></a>Recuperación de datos

*Para obtener más información, consulte [Control de seguridad: recuperación de datos](https://docs.microsoft.com/azure/security/benchmarks/security-control-data-recovery).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: Azure Database for MariaDB realiza copias de seguridad completas, diferenciales y del registro de transacciones.  Azure Database for MariaDB crea automáticamente copias de seguridad del servidor y las almacena en el almacenamiento con redundancia local o con redundancia geográfica configurado por el usuario. Las copias de seguridad pueden utilizarse para restaurar el servidor a un momento dado. Las copias de seguridad y las restauraciones son una parte esencial de cualquier estrategia de continuidad del negocio, ya que protegen los datos frente a daños o eliminaciones accidentales.  El período de retención predeterminado es siete días. Opcionalmente, puede configurarlo hasta 35 días. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits.

Descripción de las copias de seguridad de MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-backup

Descripción de la configuración inicial de MariaDB: https://docs.microsoft.com/azure/mariadb/tutorial-design-database-using-portal



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Azure Database for MariaDB crea automáticamente copias de seguridad del servidor y las almacena en el almacenamiento con redundancia local o con redundancia geográfica configurado por el usuario. Las copias de seguridad pueden utilizarse para restaurar el servidor a un momento dado.  Las copias de seguridad y las restauraciones son una parte esencial de cualquier estrategia de continuidad del negocio, ya que protegen los datos frente a daños o eliminaciones accidentales.

Si usa Key Vault para el cifrado de datos del lado cliente para los datos almacenados en el servidor de MariaDB, asegúrese de realizar copias de seguridad automatizadas periódicas de las claves.

Descripción de las copias de seguridad de MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-backup

Realización de copias de seguridad de claves de Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Compartido

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Instrucciones**: En Azure Database for MariaDB, realice una restauración a partir de las copias de seguridad del servidor original a fin de realizar pruebas periódicas de las copias de seguridad. Hay dos tipos de restauración disponibles: Restauración a un momento dado y geográfica. La restauración a un momento dado está disponible con cualquier opción de redundancia de copia de seguridad y crea un nuevo servidor en la misma región que el servidor original. La restauración geográfica solo está disponible si ha configurado el servidor para el almacenamiento con redundancia geográfica y le permite restaurar el servidor en otra región.

El tiempo estimado de recuperación depende de varios factores, como el tamaño de la bases de datos, el tamaño del registro de transacciones, el ancho de banda de red y el número total de bases de datos que se están recuperando en la misma región al mismo tiempo. Normalmente, el tiempo de recuperación es inferior a 12 horas.

Descripción de la copia de seguridad y restauración en Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-backup#restore


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas del cliente

**Guía**: Azure Database for MariaDB realiza copias de seguridad completas, diferenciales y del registro de transacciones. Estas copias de seguridad permiten restaurar un servidor a un momento dado dentro del período de retención de copias de seguridad configurado. El período de retención predeterminado es siete días. Opcionalmente, puede configurarlo hasta 35 días. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits.

Descripción de la copia de seguridad y restauración en Azure Database for MariaDB: https://docs.microsoft.com/azure/mariadb/concepts-backup


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para obtener más información, consulte [Control de seguridad: respuesta ante incidentes](https://docs.microsoft.com/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

- Guía para crear su propio proceso de respuesta a incidentes de seguridad: https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/
- Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft: https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/
- El cliente también puede utilizar la guía de control de incidentes de seguridad de equipos de NIST como referencia para la creación de su propio plan de respuesta a incidentes: https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final 

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta. 

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) con etiquetas y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure, especialmente los que procesan datos confidenciales.  Es su responsabilidad asignar prioridades a la corrección de las alertas en función de la importancia de los recursos y el entorno de Azure donde se produjo el incidente.

- Alertas de seguridad en Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-alerts-overview

- Uso de etiquetas para organizar los recursos de Azure: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las funcionalidades de respuesta a los incidentes de los sistemas periódicamente; así, ayudará a proteger los recursos de Azure. Identifique puntos débiles y brechas y revise el plan según sea necesario.

- Consulte la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y capacidades de TI: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Instrucciones**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

- Cómo establecer el contacto de seguridad de Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Guía**: Exporte las alertas y recomendaciones de Azure Security Center y mediante la característica de exportación continua para ayudar a identificar los riesgos para los recursos de Azure. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Azure Sentinel.

- Configuración de la exportación continua: https://docs.microsoft.com/azure/security-center/continuous-export
- Transmisión de alertas a Azure Sentinel: https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Guía**: Use la característica Automatización de flujo de trabajo de Azure Security Center para desencadenar automáticamente las respuestas a través de Logic Apps en las alertas y recomendaciones de seguridad para proteger los recursos de Azure.
    

Configuración de la automatización de flujo de trabajo y Logic Apps: https://docs.microsoft.com/azure/security-center/workflow-automation

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para obtener más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](https://docs.microsoft.com/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos en un plazo de 60 días

**Guía**: Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft:

https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

Puede encontrar más información sobre la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft aquí: https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).
