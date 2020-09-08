---
title: Línea de base de seguridad de Azure para Key Vault
description: Línea de base de seguridad de Azure para Key Vault
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/10/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 34036388ce9243c082cf79eb4be9251957eafc01
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400473"
---
# <a name="azure-security-baseline-for-key-vault"></a>Línea de base de seguridad de Azure para Key Vault

La línea de base de seguridad de Azure para Key Vault contiene recomendaciones que lo ayudarán a mejorar la posición de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](https://docs.microsoft.com/azure/security/benchmarks/overview), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para obtener más información, consulte [Introducción a las líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).

## <a name="network-security"></a>Seguridad de redes

*Para obtener más información, consulte [Control de seguridad: seguridad de red](https://docs.microsoft.com/azure/security/benchmarks/security-control-network-security).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1: Proteja los recursos mediante grupos de seguridad de red o Azure Firewall en su red virtual

**Instrucciones**: Integre Azure Key Vault con Azure Private Link. 

El servicio Azure Private Link permite acceder a los servicios de Azure (por ejemplo, Azure Key Vault) y a los servicios de asociados o clientes hospedados de Azure mediante un punto de conexión privado de la red virtual.

Un punto de conexión privado de Azure es una interfaz de red que le conecta de forma privada y segura a un servicio con la tecnología de Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual para incorporar el servicio de manera eficaz a su red virtual. Todo el tráfico dirigido al servicio se puede enrutar mediante el punto de conexión privado, por lo que no se necesita ninguna puerta de enlace, dispositivos NAT, conexiones de ExpressRoute o VPN ni direcciones IP públicas. El tráfico entre la red virtual y el servicio atraviesa la red troncal de Microsoft, eliminando la exposición a la red pública de Internet. Puede conectarse a una instancia de un recurso de Azure, lo que le otorga el nivel más alto de granularidad en el control de acceso.

Integración de Key Vault en Azure Private Link:

https://docs.microsoft.com/azure/key-vault/private-link-service


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2: Supervise y registre la configuración y el tráfico de redes virtuales, subredes y NIC

**Instrucciones**: Use Azure Security Center y siga las recomendaciones de protección de redes para ayudar a proteger sus recursos configurados de Key Vault en Azure. 

Para obtener más información sobre la seguridad de red proporcionada por Azure Security Center, consulte los siguientes recursos: 

https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Instrucciones**: Habilite el estándar de Azure DDoS Protection en las redes virtuales de Azure asociadas a las instancias de Key Vault para protegerse de los ataques por denegación de servicio distribuidos. Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

 
Administración de Azure DDoS Protection estándar mediante Azure Portal: https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection

Detección de amenazas para la capa de servicios de Azure en Azure Security Center: https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5: Registre los paquetes de red y registros de flujo

**Instrucciones**: Azure Key Vault no usa grupos de seguridad de red (NSG) y los registros de flujo de Azure Key Vault no se capturan. En su lugar, use Azure Private Link para proteger las instancias de Azure Key Vault y habilitar la configuración de diagnóstico para registrar las métricas y los eventos de auditoría.

Integración de Key Vault en Azure Private Link:

https://docs.microsoft.com/azure/key-vault/private-link-service

Registro de Azure Key Vault: https://docs.microsoft.com/azure/key-vault/key-vault-logging



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implemente sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Instrucciones**: Este requisito se puede cumplir configurando la protección contra amenazas avanzada (ATP) para Azure Key Vault. ATP proporciona una capa adicional de inteligencia de seguridad. Esta herramienta permite detectar intentos potencialmente perjudiciales de aprovechar las vulnerabilidades de sus cuentas de Azure Key Vault o de acceder a ellas.

Cuando Azure Security Center detecta una actividad anómala muestra alertas. También envía por correo electrónico al administrador de la suscripción los detalles de la actividad sospechosa y recomendaciones sobre cómo investigar y corregir las amenazas identificadas.

Configuración de la protección contra amenazas avanzada para Azure Key Vault:

https://docs.microsoft.com/azure/security-center/advanced-threat-protection-key-vault



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Instrucciones**: En el caso de los recursos que necesitan acceso a las instancias de Azure Key Vault, use las etiquetas de servicio de Azure para la instancia de Azure Key Vault con el fin de definir controles de acceso a la red en grupos de seguridad de red o Azure Firewall. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, ApiManagement) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

Introducción a las etiquetas de servicio de Azure: https://docs.microsoft.com/azure/virtual-network/service-tags-overview


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Instrucciones**: Defina e implemente configuraciones de seguridad estándar para los recursos de red asociados a las instancias de Azure Key Vault con Azure Policy. Use alias de Azure Policy en los espacios de nombres "Microsoft.KeyVault" y "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de las instancias de Azure Key Vault. También puede usar definiciones de directivas integradas relacionadas con Azure Key Vault, como:

Key Vault debe usar un punto de conexión del servicio de red virtual

Tutorial: Creación y administración de directivas para aplicar el cumplimiento:

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Ejemplos de Azure Policy:

https://docs.microsoft.com/azure/governance/policy/samples/#networ

Inicio rápido: Definición y asignación de un plano técnico en el portal:

https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Instrucciones**: Use etiquetas para los recursos relacionados con la seguridad de red y el flujo de tráfico de las instancias de Azure Key Vault para proporcionar metadatos y organización lógica.

Use cualquiera de las definiciones de Azure Policy integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor", para asegurarse de que todos los recursos se crean con etiquetas y para notificarle los recursos no etiquetados existentes.

Puede usar Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas.

Uso de etiquetas para organizar los recursos de Azure:

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Instrucciones**: Use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en los recursos de red relacionados con las instancias de Azure Key Vault. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.

Visualización y recuperación de eventos del registro de actividad de Azure:

https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view

Crear, ver y administrar las alertas del registro de actividad mediante Azure Monitor:

https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para obtener más información, consulte [Control de seguridad: registro y supervisión](https://docs.microsoft.com/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Instrucciones**: No aplicable; Microsoft mantiene el origen de la hora usado en los recursos de Azure, como Azure Key Vault, para las marcas de tiempo de los registros.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Instrucciones**: Ingiera registros a través de Azure Monitor para agregar datos de seguridad generados por Azure Key Vault. En Azure Monitor, use áreas de trabajo de Azure Log Analytics para realizar consultas y análisis, y utilice cuentas de Azure Storage para el almacenamiento de archivos a largo plazo. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros. 

Registro de Azure Key Vault:

https://docs.microsoft.com/azure/key-vault/key-vault-logging

Inicio rápido: Incorporación de Azure Sentinel:

https://docs.microsoft.com/azure/sentinel/quickstart-onboard


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Instrucciones**: Habilite Configuración de diagnóstico en las instancias de Azure Key Vault para poder acceder a los registros de auditoría, seguridad y diagnóstico. Los registros de actividad, que están disponibles automáticamente, incluyen el origen del evento, la fecha, el usuario, la marca de tiempo, las direcciones de origen y de destino, y otros elementos útiles.

Registro de Azure Key Vault:

https://docs.microsoft.com/azure/key-vault/key-vault-logging


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Instrucciones**: En Azure Monitor, en el área de trabajo de Log Analytics que se usa para almacenar los registros de Azure Key Vault, establezca el período de retención de acuerdo con las normas de cumplimiento de su organización. Use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

Cambio del período de retención de datos: https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Instrucciones**: Analice y supervise los registros en busca de un comportamiento anómalo y revise periódicamente los resultados de los recursos protegidos mediante Azure Key Vault. Use el área de trabajo de Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos del registro. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros. 

Inicio rápido: Incorporación de Azure Sentinel:

https://docs.microsoft.com/azure/sentinel/quickstart-onboard

Introducción a Log Analytics en Azure Monitor:

https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal

Introducción a las consultas de registro en Azure Monitor:

https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7: Habilitación de alertas para actividades anómalas

**Instrucciones**: En Azure Security Center, habilite la protección contra amenazas avanzada (ATP) para Key Vault. Habilite la configuración de diagnóstico en Azure Key Vault y envíe registros a un área de trabajo de Log Analytics. Incorpore el área de trabajo de Log Analytics a Azure Sentinel, ya que proporciona una solución de respuesta automatizada de orquestación de seguridad (SOAR). Esto permite crear cuadernos de estrategias (soluciones automatizadas) y usarlos para corregir problemas de seguridad.

Inicio rápido: Incorporación de Azure Sentinel:

https://docs.microsoft.com/azure/sentinel/quickstart-onboard

Administración y respuesta a alertas de seguridad en Azure Security Center:

https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts

Respuesta a eventos con las alertas de Azure Monitor:

https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-response


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Instrucciones**: No aplicable; Azure Key Vault no procesa ni genera registros relacionados con antimalware.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Instrucciones**: No aplicable; Azure Key Vault no procesa ni genera registros relacionados con DNS.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para obtener más información, consulte [Control de seguridad: identidad y control de acceso](https://docs.microsoft.com/azure/security/benchmarks/security-control-identity-access-control).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Instrucciones**: Mantenga un inventario de las aplicaciones registradas en Azure Active Directory, así como las cuentas de usuario que tengan acceso a las claves, los secretos y los certificados de Azure Key Vault. Puede usar Azure Portal o PowerShell para consultar y reconciliar el acceso a Key Vault. Para ver el acceso en PowerShell, use el siguiente comando:

(Get-AzResource -ResourceId [KeyVaultResourceID]).Properties.AccessPolicies

Registrar una aplicación con Azure Active Directory:

https://docs.microsoft.com/azure/key-vault/key-vault-manage-with-cli2#registering-an-application-with-azure-active-directory

Protección del acceso a un almacén de claves:

https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Instrucciones**: No aplicable; Azure Key Vault no tiene el concepto de contraseñas predeterminadas, ya que la autenticación la proporciona Active Directory y se protege con el control de acceso basado en roles.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Instrucciones**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas que tengan acceso a las instancias de Azure Key Vault. Use la administración de identidad y acceso de Azure Security Center (actualmente en fase de versión preliminar) para supervisar el número de cuentas administrativas activas.

Supervisión de la identidad y el acceso (versión preliminar):

https://docs.microsoft.com/azure/security-center/security-center-identity-access


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Instrucciones**: Use una entidad de servicio de Azure junto con AppId, TenantID y ClientSecret para autenticar la aplicación sin problemas y recuperar el token que se usará para acceder a los secretos de Azure Key Vault.

Autenticación entre servicios en Azure Key Vault mediante .NET:

https://docs.microsoft.com/azure/key-vault/service-to-service-authentication



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Instrucciones**: Habilite Multi-Factor Authentication de Azure Active Directory y siga las recomendaciones de administración de identidades y acceso de Azure Security Center (actualmente en fase de versión preliminar) para ayudar a proteger los recursos habilitados para Event Hubs.

Planeamiento de una implementación de Azure Multi-Factor Authentication basada en la nube:

https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Supervisión de la identidad y el acceso (versión preliminar):

https://docs.microsoft.com/azure/security-center/security-center-identity-access


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Instrucciones**: Utilice estaciones de acceso con privilegios (PAW) con Azure Multi-Factor Authentication (MFA) configurada para iniciar sesión en recursos habilitados para Azure Key Vault y configurarlos. 

Uso de estaciones de trabajo con privilegios de acceso: https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations 

Planeamiento de una implementación de Azure Multi-Factor Authentication basada en la nube: https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7: Registre y alerte de la actividad sospechosa desde cuentas administrativas

**Guía**: use Privileged Identity Management (PIM) de Azure Active Directory (Azure AD) para la generación de registros y alertas cuando se produzca una actividad sospechosa o insegura en el entorno. Use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo. Para obtener más registros, envíe alertas de detección de Azure Security Center riesgo en Azure Monitor y configure alertas o notificaciones personalizadas con grupos de acciones.

Habilite la protección contra amenazas avanzada (ATP) para Azure Key Vault con el fin de generar alertas para actividades sospechosas.

Implementación de Azure AD Privileged Identity Management (PIM): https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan

Configuración de la protección contra amenazas avanzada para Azure Key Vault (versión preliminar): https://docs.microsoft.com/azure/security-center/advanced-threat-protection-key-vault

Alertas de Azure Key Vault (versión preliminar): https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-azurekv

Detecciones de riesgos de Azure Active Directory: https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events

Creación y administración de grupos de acciones en Azure Portal: https://docs.microsoft.com/azure/azure-monitor/platform/action-groups



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Instrucciones**: Configure la condición de ubicación de una directiva de acceso condicional y administre las ubicaciones con nombre. Gracias a las ubicaciones con nombre, puede crear agrupaciones lógicas de rangos de direcciones IP o de países y regiones. Puede restringir el acceso a recursos confidenciales, como los secretos de Key Vault, a las ubicaciones con nombre configuradas.

¿Qué es la condición de ubicación del acceso condicional de Azure Active Directory?: https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations



**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Instrucciones**: Use Azure Active Directory (Azure AD) como sistema central de autenticación y autorización para recursos de Azure como Key Vault. Esto permite el control de acceso basado en roles (RBAC) para administrar recursos confidenciales.

 

Inicio rápido: Creación de un inquilino en Azure Active Directory:

https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Instrucciones**: Revise los registros de Azure Active Directory (Azure AD) para ayudar a detectar cuentas obsoletas con roles administrativos de Azure Key Vault. Además, use las revisiones de acceso de Azure AD para administrar de forma eficaz la pertenencia a grupos, el acceso a las aplicaciones empresariales que se pueden usar para tener acceso a Azure Key Vault y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica cada 90 días para asegurarse de que solo las personas adecuadas tengan acceso continuado.

Documentación sobre informes y supervisión de Azure Active Directory:

https://docs.microsoft.com/azure/active-directory/reports-monitoring/

¿Qué son las revisiones de acceso de Azure AD?:

https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11: Supervise los intentos de acceso a cuentas desactivadas

**Instrucciones**: Habilite la configuración de diagnóstico para Azure Key Vault y Azure Active Directory, y envíe todos los registros a un área de trabajo de Log Analytics. Configure las alertas que desee (como intentos de acceso a secretos deshabilitados) en Log Analytics.

Integración de registros de Azure AD con registros de Azure Monitor: https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

Migración desde la solución Key Vault antigua: https://docs.microsoft.com/azure/azure-monitor/insights/azure-key-vault#migrating-from-the-old-key-vault-solution



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Instrucciones**: Use las características de la protección de identidades y la detección de riesgos de Azure Active Directory para configurar las respuestas automatizadas a acciones sospechosas detectadas relacionadas con los recursos protegidos de Azure Key Vault. Debe habilitar las respuestas automatizadas a través de Azure Sentinel para implementar las respuestas de seguridad de su organización. 

Informe de inicios de sesión poco seguros del portal de Azure Active Directory: https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins 

Instrucciones: Configuración y habilitación de directivas de riesgo: https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

Incorporación de Azure Sentinel: https://docs.microsoft.com/azure/sentinel/quickstart-onboard


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Instrucciones**: No aplicable; Caja de seguridad del cliente no es compatible con Azure Key Vault.

Servicios y escenarios admitidos con disponibilidad general: https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-protection"></a>Protección de datos

*Para obtener más información, consulte [Control de seguridad: protección de datos](https://docs.microsoft.com/azure/security/benchmarks/security-control-data-protection).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: Use etiquetas para ayudar a realizar el seguimiento de los recursos de Azure que almacenan o procesan información confidencial en los recursos habilitados para Azure Key Vault. 

Uso de etiquetas para organizar los recursos de Azure: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Puede proteger el acceso a Azure Key Vault mediante el uso de puntos de conexión de servicio de red virtual configurados para restringir el acceso a subredes específicas.

Una vez que las reglas del firewall están en vigor, los usuarios solo pueden realizar operaciones del plano de datos de Azure Key Vault cuando las solicitudes se originan desde redes virtuales o rangos de direcciones IP permitidos. Esto también se aplica a Azure Key Vault acceso en Azure Portal. Aunque puede ir a un almacén de claves desde Azure Portal, es posible que no pueda enumerar las claves, los secretos o los certificados si su equipo cliente no está en la lista de dispositivos permitidos. Esto también afecta al selector de Azure Key Vault de otros servicios de Azure. Los usuarios pueden ver la lista de almacenes de claves, pero no pueden enumerar las claves si las reglas del firewall limitan su equipo cliente.

Configuración de firewalls y redes virtuales de Azure Key Vault: https://docs.microsoft.com/azure/key-vault/key-vault-network-security

Puntos de conexión de servicio de red virtual para Azure Key Vault: https://docs.microsoft.com/azure/key-vault/key-vault-overview-vnet-service-endpoints



**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Instrucciones**: Todos los datos almacenados en Azure Key Vault se consideran confidenciales. Utilice los controles de acceso al plano de datos de Azure Key Vault para controlar el acceso a los secretos de Azure Key Vault. También puede usar el firewall integrado de Key Vault para controlar el acceso en el nivel de red. Para supervisar el acceso a Azure Key Vault, habilite la configuración de diagnóstico de Key Vault y envíe registros a una cuenta de Azure Storage o un área de trabajo de Log Analytics.

Protección del acceso a un almacén de claves: https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault

Configuración de firewalls y redes virtuales de Azure Key Vault: https://docs.microsoft.com/azure/key-vault/key-vault-network-security

Registro de Azure Key Vault: https://docs.microsoft.com/azure/key-vault/key-vault-logging



**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Instrucciones**: Todo el tráfico dirigido a Azure Key Vault para la autenticación, la administración y el acceso al plano de datos se cifra y pasa a través de HTTPS: puerto 443. Sin embargo, ocasionalmente habrá tráfico HTTP (puerto 80) para CRL. 

Acceso a Azure Key Vault desde detrás de un firewall: https://docs.microsoft.com/azure/key-vault/key-vault-access-behind-firewall



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Instrucciones**: No aplicable; todos los datos de Azure Key Vault (secretos, claves y certificados) se consideran confidenciales.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos

**Guía**: Proteja el acceso a la administración y al plano de datos de las instancias de Azure Key Vault.

Protección del acceso a un almacén de claves:

https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Instrucciones**: Microsoft administra la infraestructura subyacente para Azure Key Vault y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

¿Qué es el Almacén de claves de Azure?

https://docs.microsoft.com/azure/key-vault/key-vault-overview

Protección de datos de cliente de Azure:

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Instrucciones**: Todos los objetos administrados (clave, certificados y secretos) se cifran en reposo en Azure Key Vault.

Documentación complementaria:

- [Tabla de modelo de cifrado y administración de claves](https://docs.microsoft.com/azure/security/fundamentals/encryption-atrest#encryption-model-and-key-management-table)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Instrucciones**: Utilice la solución Azure Key Vault Analytics en Azure Monitor para revisar los registros de eventos de auditoría de Azure Key Vault.

Solución Azure Key Vault Analytics en Azure Monitor:

https://docs.microsoft.com/azure/azure-monitor/insights/azure-key-vault



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para obtener más información, consulte [Control de seguridad: administración de vulnerabilidades](https://docs.microsoft.com/azure/security/benchmarks/security-control-vulnerability-management).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Instrucciones**: Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Azure Key Vault.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Instrucciones**: N/A; Microsoft realiza la administración de revisiones en los sistemas subyacentes que admiten Key Vault.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3: Implemente una solución automatizada de administración de revisiones de software de terceros

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Instrucciones**: Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Key Vault.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Instrucciones**: Use la clasificación de riesgo predeterminada (puntuación de seguridad) proporcionada por Azure Security Center.

Mejora de la puntuación de seguridad de Azure Security Center:

https://docs.microsoft.com/azure/security-center/security-center-secure-score


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para obtener más información, consulte [Control de seguridad: inventario y administración de recursos](https://docs.microsoft.com/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-use-azure-asset-discovery"></a>6.1: Uso de la detección de recursos de Azure

**Instrucciones**: Use Azure Resource Graph para consultar y detectar todos los recursos (incluidas las instancias de Azure Key Vault) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

Inicio rápido: Ejecución de la primera consulta de Resource Graph mediante Azure Resource Graph Explorer:

https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Obtención de las suscripciones a las que puede tener acceso la cuenta actual:

https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0

¿Qué es el control de acceso basado en rol de Azure (RBAC)?

https://docs.microsoft.com/azure/role-based-access-control/overview

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Instrucciones**: Aplique etiquetas a los recursos de Azure Key Vault que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

Cómo crear y usar etiquetas:

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Instrucciones**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de las instancias de Azure Key Vault y los recursos relacionados. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

Creación de una suscripción de Azure adicional:

https://docs.microsoft.com/azure/billing/billing-create-subscription

Creación de grupos de administración para la organización y la administración de recursos:

https://docs.microsoft.com/azure/governance/management-groups/create

Uso de etiquetas para organizar los recursos de Azure: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4: Mantenimiento de un inventario de los recursos de Azure aprobados y títulos de software

**Instrucciones**: Defina una lista de recursos de Azure aprobados y software aprobado para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Instrucciones**: Use directivas de Azure para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

Además, use Azure Resource Graph para consultar o detectar recursos dentro de sus suscripciones.

Tutorial: Creación y administración de directivas para aplicar el cumplimiento: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Inicio rápido: Ejecución de la primera consulta de Resource Graph mediante Azure Resource Graph Explorer: https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Instrucciones**: No aplicable; esta recomendación está destinada a Azure en su conjunto, así como a los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Instrucciones**: Use directivas de Azure para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

Tutorial: Creación y administración de directivas para aplicar el cumplimiento: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Ejemplos de Azure Policy: https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="610-implement-approved-application-list"></a>6.10: Implementación de la lista de aplicaciones aprobadas

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="611-limit-users-ability-to-interact-with-azureresources-manager-via-scripts"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager mediante scripts

**Guía**: use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager (ARM) al configurar "Bloquear acceso" para la aplicación "Microsoft Azure Management". Esto puede impedir la creación y los cambios en los recursos de un entorno de alta seguridad, como los que tienen la configuración de Key Vault.

Administración del acceso a la administración de Azure con acceso condicional:

https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management


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

**Instrucciones**: Use alias de Azure Policy en el espacio de nombres "Microsoft.KeyVault" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de la instancia de Azure Key Vault. También puede usar las definiciones de Azure Policy integradas para Azure Key Vault como:

Los objetos del almacén de claves deben poder recuperarse

Implementar la configuración de diagnóstico para Key Vault en un área de trabajo de Log Analytics

Los registros de diagnóstico en Key Vault deben estar habilitados

Key Vault debe usar un punto de conexión del servicio de red virtual

Implementar la configuración de diagnóstico para Key Vault en el Centro de eventos

Use las recomendaciones de Azure Security Center como línea de base de configuración segura para las instancias de Azure Key Vault.

Visualización de los alias de Azure Policy disponibles:

https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0

Tutorial: Creación y administración de directivas para aplicar el cumplimiento:

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Utilice las directivas [deny] y [deploy if not exist] de Azure Policy para aplicar una configuración segura en los recursos habilitados para Azure Key Vault. 

Tutorial: Creación y administración de directivas para aplicar el cumplimiento:

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage 

  
Descripción de los efectos de Azure Policy: 

https://docs.microsoft.com/azure/governance/policy/concepts/effects


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: si usa definiciones de Azure Policy personalizadas para los recursos habilitados para Azure Key Vault, use Azure Repos para almacenar y administrar el código de forma segura.

Cómo almacenar código en Azure DevOps: 

https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops 

Documentación de Azure Repos: 

https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7: Implemente herramientas de administración de configuración del sistema

**Instrucciones**: Use alias de Azure Policy en el espacio de nombres "Microsoft.KeyVault" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

Configuración y administración de Azure Policy:

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8: Implemente herramientas de administración de configuración del sistema para sistemas operativos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9: Implemente la supervisión de configuración automatizada para servicios de Azure

**Instrucciones**: Use Azure Security Center para realizar análisis de base de referencia para los recursos protegidos de Azure Key Vault. 

  

Cómo corregir las recomendaciones en Azure Security Center: 

https://docs.microsoft.com/azure/security-center/security-center-remediate-recommendations

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Instrucciones**: No aplicable; esta prueba comparativa está pensada para recursos de proceso.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Instrucciones**: Use Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de secretos para las aplicaciones en la nube. Asegúrese de que la eliminación temporal de Azure Key Vault esté habilitada.

Cómo integrar las identidades administradas de Azure:

https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

Cómo crear un almacén de claves:

https://docs.microsoft.com/azure/key-vault/general/quick-create-portal

Autenticación en Key Vault:

https://docs.microsoft.com/azure/key-vault/general/authentication

Asignación de una directiva de acceso de Key Vault:

https://docs.microsoft.com/azure/key-vault/general/assign-access-policy-portal

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Instrucciones**: Use Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de secretos para las aplicaciones en la nube. 

  

* [Integración con identidades administradas de Azure](/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity)

* [Creación de un almacén de claves](quick-create-portal.md)

* [Autenticación en Key Vault](authentication.md)

* [Asignación de una directiva de acceso de Key Vault](assign-access-policy-portal.md)

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

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso. Microsoft controla el antimalware de la plataforma subyacente.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Instrucciones**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Key Vault), pero no se ejecuta en el contenido del cliente.

Analice previamente cualquier contenido que se cargue o envíe a recursos de Azure que no son de proceso como Azure Key Vault. Microsoft no tiene acceso a los datos de estas instancias.

Descripción de Microsoft Antimalware para Azure Cloud Services y Virtual Machines: https://docs.microsoft.com/azure/security/fundamentals/antimalware

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso. Microsoft controla el antimalware de la plataforma subyacente.


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-recovery"></a>Recuperación de datos

*Para obtener más información, consulte [Control de seguridad: recuperación de datos](https://docs.microsoft.com/azure/security/benchmarks/security-control-data-recovery).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Instrucciones**: Asegúrese de que se realicen copias de seguridad automatizadas periódicas de sus certificados, claves, cuentas de almacenamiento administradas y secretos de Key Vault con los siguientes comandos de PowerShell:

- Backup-AzKeyVaultCertificate

- Backup-AzKeyVaultKey

- Backup-AzKeyVaultManagedStorageAccount

- Backup-AzKeyVaultSecret

Opcionalmente, puede almacenar las copias de seguridad de Key Vault en Azure Backup.

Realización de copias de seguridad de certificados de Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate

Realización de copias de seguridad de claves de Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey

Realización de copias de seguridad de cuentas de almacenamiento de Key Vault: https://docs.microsoft.com/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount

Realización de copias de seguridad de secretos de Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret

Habilitación de Azure Backup: https://docs.microsoft.com/azure/backup



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Instrucciones**: Realice copias de seguridad automatizadas periódicas de sus certificados, claves, cuentas de almacenamiento administradas y secretos de Key Vault con los siguientes comandos de PowerShell:

- Backup-AzKeyVaultCertificate

- Backup-AzKeyVaultKey

- Backup-AzKeyVaultManagedStorageAccount

- Backup-AzKeyVaultSecret

Opcionalmente, puede almacenar las copias de seguridad de Key Vault en Azure Backup.

Realización de copias de seguridad de certificados de Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate

Realización de copias de seguridad de claves de Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey

Realización de copias de seguridad de cuentas de almacenamiento de Key Vault: https://docs.microsoft.com/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount

Realización de copias de seguridad de secretos de Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret

Habilitación de Azure Backup: https://docs.microsoft.com/azure/backup



**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Instrucciones**: Realice restauraciones de datos automatizadas periódicas de sus certificados, claves, cuentas de almacenamiento administradas y secretos de Key Vault con los siguientes comandos de PowerShell:

- Restore-AzKeyVaultCertificate

- Restore-AzKeyVaultKey

- Restore-AzKeyVaultManagedStorageAccount

- Restore-AzKeyVaultSecret

Restauración de certificados de Key Vault:  https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultcertificate?view=azurermps-6.13.0

Restauración de claves de Azure Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0 

Restauración de cuentas de almacenamiento de Key Vault: https://docs.microsoft.com/powershell/module/az.keyvault/backup-azkeyvaultmanagedstorageaccount

Restauración de secretos de Azure Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultsecret?view=azurermps-6.13.0


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas del cliente

**Guía**: Asegúrese de que la eliminación temporal esté habilitada para Azure Key Vault. La eliminación temporal permite la recuperación de almacenes de claves y objetos de almacén eliminados, como claves, secretos y certificados. 

Uso de la eliminación temporal de Azure Key Vault: 

https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-powershell

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para obtener más información, consulte [Control de seguridad: respuesta ante incidentes](https://docs.microsoft.com/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia. Estos procesos deben centrarse en la protección de los sistemas confidenciales, como los que usan los secretos de Key Vault.

Configuración de las automatizaciones del flujo de trabajo en Azure Security Center: 

https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide   

Instrucciones para crear su propio proceso de respuesta a incidentes de seguridad:  

https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/

Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft:   

https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process   

El cliente también puede utilizar la guía de control de incidentes de seguridad de equipos de NIST como referencia para la creación de su propio plan de respuesta a incidentes: 

https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta. Además, marque claramente las suscripciones (por ejemplo, producción, no producción) y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure, especialmente los que procesan datos confidenciales, como los secretos de Azure Key Vault.


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Lleve a cabo ejercicios para probar las funcionalidades de respuesta a los incidentes de los sistemas con regularidad para ayudar a proteger las instancias de Azure Key Vault y los recursos relacionados. Identifique puntos débiles y brechas y revise el plan según sea necesario.

Consulte la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y capacidades de TI: 

https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Instrucciones**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos.  Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

Establecimiento del contacto de seguridad de Azure Security Center:

https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: Exporte sus alertas y recomendaciones de Azure Security Center y mediante la característica de exportación continua para ayudar a identificar los riesgos para los recursos habilitados para Azure Key Vault. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua.  Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Azure Sentinel. 

 

Configuración de la exportación continua: 

https://docs.microsoft.com/azure/security-center/continuous-export 

  

Transmisión de alertas a Azure Sentinel: 

https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Instrucciones**: Use la característica de automatización del flujo de trabajo de Azure Security Center para desencadenar automáticamente las respuestas a través de Logic Apps en las alertas y recomendaciones de seguridad para proteger los recursos protegidos mediante Azure Key Vault. 

 

Configuración de la automatización de flujo de trabajo y Logic Apps: 

https://docs.microsoft.com/azure/security-center/workflow-automation

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para obtener más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](https://docs.microsoft.com/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos en un plazo de 60 días

**Instrucciones**: No realice pruebas de penetración en el servicio Azure Key Vault directamente; se recomienda probar los recursos de Azure que usan Key Vault para garantizar la seguridad de los secretos.

Debe seguir las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft:

https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

Puede encontrar más información sobre la estrategia de Microsoft y la ejecución de las pruebas de penetración de equipo rojo y sitios activos en la nube, los servicios y las aplicaciones administradas por Microsoft aquí: 

https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).
