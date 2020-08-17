---
title: Base de referencia de seguridad de Azure para Azure Data Box
description: Base de referencia de seguridad de Azure para Azure Data Box
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 06/22/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: a13a8b2eef9fa1bd9c7281ac6b7c7dba40105428
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497331"
---
# <a name="azure-security-baseline-for-azure-data-box"></a>Base de referencia de seguridad de Azure para Azure Data Box

La línea de base de seguridad de Azure para Azure Data Box contiene recomendaciones que le ayudarán a mejorar la situación de seguridad de la implementación.

La base de referencia de este servicio se extrae de la [versión 1.0 de Azure Security Benchmark](https://docs.microsoft.com/azure/security/benchmarks/overview), que proporciona recomendaciones sobre cómo puede proteger las soluciones en la nube en Azure con nuestra guía de procedimientos recomendados.

Para más información, consulte [Introducción a la línea de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).

## <a name="network-security"></a>Seguridad de las redes

*Para más información, consulte [Control de seguridad: Seguridad de redes](https://docs.microsoft.com/azure/security/benchmarks/security-control-network-security).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Instrucciones**: no es aplicable, Azure Data Box no se puede asociar a una red virtual. Puede controlar el tráfico desde Data Box al almacenamiento hospedado en Azure a través de Azure Portal. Con Data Box, los datos se transfieren a través de la red troncal de Azure.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes y NIC

**Guía**: no es aplicable, Azure Data Box no se puede asociar a una red virtual. Puede controlar el tráfico desde Data Box al almacenamiento hospedado en Azure a través de Azure Portal. Con Data Box, los datos se transfieren a través de la red troncal de Azure.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Guía**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Instrucciones**: no es aplicable, Azure Data Box no se puede asociar a una red virtual. Puede controlar el tráfico desde Data Box al almacenamiento hospedado en Azure a través de Azure Portal. Con Data Box, los datos se transfieren a través de la red troncal de Azure.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Guía**: no es aplicable, Azure Data Box no se puede asociar a una red virtual. Puede controlar el tráfico desde Data Box al almacenamiento hospedado en Azure a través de Azure Portal. Con Data Box, los datos se transfieren a través de la red troncal de Azure.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implementación de sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Guía**: instrucciones: Microsoft administra los puntos de conexión que Azure Data Box usa. El usuario es el responsable de los controles adicionales que desee implementar en sus sistemas locales.

* [Descripción de la seguridad de Azure Data Box](https://docs.microsoft.com/azure/databox/data-box-security)

* [Información de puerto para Azure Data Box](https://docs.microsoft.com/azure/databox/data-box-system-requirements#port-requirements)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Instrucciones**: No aplicable; esta recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: no es aplicable, Azure Data Box no se puede asociar a una red virtual.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: no es aplicable, Azure Data Box no se puede asociar a una red virtual.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Instrucciones**: no es aplicable, Azure Data Box no se puede asociar a una red virtual.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: no es aplicable, Azure Data Box no se puede asociar a una red virtual.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Control de seguridad: registro y supervisión](https://docs.microsoft.com/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: Microsoft mantiene el mismo origen de tiempo que los recursos de Azure, por ejemplo, para establecer las marcas de tiempo en los registros. La hora de Azure Data Box podría desplazarse si no está conectado a la red que pueda acceder al servidor NTP en el sitio del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: en función de cada paso del pedido de Data Box, puede realizar varias acciones para controlar el acceso al pedido, auditar los eventos, hacer un seguimiento del pedido e interpretar los distintos registros que se generen.

* [Seguimiento y registro de eventos para Azure Data Box y Azure Data Box Heavy](https://docs.microsoft.com/azure/databox/data-box-logs)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: en función de cada paso del pedido de Data Box, puede realizar varias acciones para controlar el acceso al pedido, auditar los eventos, hacer un seguimiento del pedido e interpretar los distintos registros que se generen.

* [Seguimiento y registro de eventos para Azure Data Box y Azure Data Box Heavy](https://docs.microsoft.com/azure/databox/data-box-logs)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Instrucciones**: esta recomendación está pensada para los recursos de proceso. Data Box tiene registros de auditoría y un paquete de soporte que contiene registros de seguridad.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Instrucciones**: No aplicable

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Guía**: en función de cada paso del pedido de Data Box, puede realizar varias acciones para controlar el acceso al pedido, auditar los eventos, hacer un seguimiento del pedido e interpretar los distintos registros que se generen.

* [Seguimiento y registro de eventos para Azure Data Box y Azure Data Box Heavy](https://docs.microsoft.com/azure/databox/data-box-logs)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: en función de cada paso del pedido de Data Box, puede realizar varias acciones para controlar el acceso al pedido, auditar los eventos, hacer un seguimiento del pedido e interpretar los distintos registros que se generen.

* [Seguimiento y registro de eventos para Azure Data Box y Azure Data Box Heavy](https://docs.microsoft.com/azure/databox/data-box-logs)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: No aplicable; Azure Data Box no procesa ni genera registros relacionados con antimalware.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: No aplicable; Azure Data Box no procesa ni genera registros relacionados con DNS.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para más información, consulte [Control de seguridad: Identidad y control de acceso](https://docs.microsoft.com/azure/security/benchmarks/security-control-identity-access-control).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: mantenga un inventario de las cuentas de usuario que tienen acceso administrativo a las solicitudes de Azure Data Box. Puede usar el panel Administración de identidad y acceso (IAM) de Azure Portal para la suscripción a fin de configurar el control de acceso basado en rol (RBAC). Los roles se aplican a los usuarios, grupos, entidades de servicio e identidades administradas en Active Directory. Puede controlar quién puede acceder a su pedido la primera vez que se cree. Configure los roles de Azure en los distintos ámbitos para controlar el acceso al pedido de Data Box. Un rol de Azure determina el tipo de acceso: lectura y escritura, solo lectura, o lectura y escritura para un subconjunto de operaciones.

* [Descripción de los roles personalizados](https://docs.microsoft.com/azure/role-based-access-control/custom-roles)

* [Procedimiento para configurar RBAC en libros](https://docs.microsoft.com/azure/sentinel/quickstart-get-visibility)

* [Procedimiento para configurar el control de acceso en el pedido](https://docs.microsoft.com/azure/databox/data-box-logs#set-up-access-control-on-the-order)

**Supervisión de Azure Security Center**: No

**Responsabilidad**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: Azure AD no tiene el concepto de contraseñas predeterminadas. Otros recursos de Azure que requieren una contraseña obligan a crear una contraseña con requisitos de complejidad y una longitud mínima, que cambia en función del servicio. Usted es responsable de las aplicaciones de terceros y de los servicios de Marketplace que pueden usar contraseñas predeterminadas.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

Además, para ayudarle a realizar un seguimiento de las cuentas administrativas dedicadas, puede seguir las recomendaciones de Azure Security Center o las directivas integradas de Azure Policy, por ejemplo:
- Debe haber más de un propietario asignado a su suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción

* [Uso de Azure Security Center para supervisar la identidad y el acceso (versión preliminar)](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

* [Uso de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Guía**: No aplicable; el acceso al pedido de Data Box se realiza con Azure Portal y se reserva para las cuentas con los roles de inquilino Propietario o colaborador.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Instrucciones**: No aplicable

**Supervisión de Azure Security Center**: No

**Responsabilidad**: No aplicable

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: utilice Estación de trabajo de acceso con privilegios (PAW) con Azure Multi-Factor Authentication (MFA) habilitada para iniciar sesión en los pedidos de Azure Data Box y para configurarlos.

* [Uso de estaciones de trabajo con privilegios de acceso](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Planeamiento de una implementación de Azure Multi-Factor Authentication basada en la nube](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Instrucciones**: no es aplicable, Azure Data Box no tiene su propia cuenta administrativa. Puede acceder a ella con Azure Portal. Sin embargo, puede configurar la suscripción a Azure para usar Privileged Identity Management (PIM) de Azure Active Directory (AD) para la generación de registros y alertas cuando se produzca una actividad sospechosa o insegura en el entorno.

Además, use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

* [Cómo implementar Privileged Identity Management (PIM)](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan)

* [Información sobre las detecciones de riesgo de Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: El cliente debe usar ubicaciones con nombre de acceso condicional para permitir el acceso a Azure Portal solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

* [Configuración de ubicaciones con nombre en Azure](https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: use Azure Active Directory (AD) como sistema central de autenticación y autorización cuando proceda. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

* [Procedimiento para crear y configurar una instancia de Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure Active Directory (AD) proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

En el caso de la aplicación Data Box, esto no se admite en tiempo real. Puede revisar los registros al final del trabajo.

* [Descripción de los informes de Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/)

* [Procedimiento para usar las revisiones de acceso de identidad de Azure](https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: use Azure Active Directory (AD) como sistema central de autenticación y autorización cuando proceda. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

Tiene acceso a los orígenes de registro de eventos de actividad de inicio de sesión, auditoría y riesgo de Azure AD, que permiten la integración en Azure Sentinel o SIEM de terceros.

Para simplificar este proceso, cree una configuración de diagnóstico para las cuentas de usuario de Azure AD y envíe los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas de registro deseadas en Log Analytics.

Los registros de servicio de Azure Data Box no se escriben en el área de trabajo de Log Analytics.

* [Integración de los registros de actividad de Azure en Azure Monitor](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

* [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: para la desviación del comportamiento de inicio de sesión de cuenta en el plan de control (por ejemplo, Azure Portal), use las características de detección de riesgos y de Azure AD Identity Protection para configurar respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. También puede hacer que Azure Sentinel ingiera los datos para investigarlos más.

* [Visualización del inicio de sesión de riesgo de Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins)

* [Configuración y habilitación de las directivas de riesgo de protección de identidad](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies)

* [Incorporación de Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: Caja de seguridad del cliente no es compatible actualmente con Azure Data Box.

* [Lista de servicios admitidos por la Caja de seguridad del cliente](https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="data-protection"></a>Protección de los datos

*Para más información, consulte [Control de seguridad: Protección de datos](https://docs.microsoft.com/azure/security/benchmarks/security-control-data-protection).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Guía**: no se aplica a Azure Data Box.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Guía**: Azure Data Box se aprovisionará en la suscripción en la que residan los recursos a los que se va a conceder acceso. No hay ningún punto de conexión público que proteger o aislar. El acceso a Data Box está disponible para los usuarios con acceso de propietario o colaborador a la suscripción.

Durante la carga de datos en Azure, la aplicación Data Box y el servicio usado para cargar los datos están aislados.

* [Introducción a Azure Data Box](https://docs.microsoft.com/azure/databox/data-box-quickstart-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Instrucciones**: Microsoft administra la infraestructura subyacente para Azure Data Box y ha implementado controles estrictos a fin de evitar la pérdida o exposición de los datos de los clientes. Cuando Data Box esté en el sitio del cliente, siga los procedimientos recomendados para asegurarse de que la información confidencial que se transfiere está protegida.

* [Descripción de la protección de datos de los clientes en Azure](https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Compartido

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: Microsoft administra la infraestructura subyacente para Azure Data Box y ha implementado controles estrictos a fin de evitar la pérdida o exposición de los datos de los clientes. Cuando Data Box esté en el sitio del cliente, siga los procedimientos recomendados para asegurarse de que la información confidencial que se transfiere está protegida.

* [Descripción de la migración de datos en Azure Data Box](https://docs.microsoft.com/azure/databox/data-box-faq)

* [Información general sobre seguridad de Data Box](https://docs.microsoft.com/azure/databox/data-box-security)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: Actualmente no está disponible; las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Data Box. Microsoft administra la infraestructura subyacente de Azure Data Box y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

* [Descripción de la protección de datos de los clientes en Azure](https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data)

**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: No aplicable

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6: Uso del control de acceso basado en rol para controlar el acceso a los recursos

**Guía**: Asegúrese de que tiene acceso de propietario o colaborador a la suscripción para crear un pedido de Data Box. También puede definir los roles de lector y colaborador de Data Box el recurso.

* [Introducción a Azure Data Box](https://docs.microsoft.com/azure/databox/data-box-quickstart-portal)

* [Configuración del control de acceso en el pedido](https://docs.microsoft.com/azure/databox/data-box-logs#set-up-access-control-on-the-order)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: No aplicable; esta recomendación está pensada para los recursos de proceso. Microsoft administra la infraestructura subyacente para Azure Data Box y ha implementado controles estrictos a fin de evitar la pérdida o exposición de los datos de los clientes.

* [Protección de datos de cliente de Azure](https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Azure Data Box implementa el cifrado AES de 256 bits para los datos en reposo.

Azure Data Box implementa el cifrado AES de 256 bits para los datos en reposo. Además, protege la clave de desbloqueo del dispositivo (también conocida como contraseña del dispositivo) que se usa para bloquearlo a través de una clave de cifrado. De forma predeterminada, la clave de desbloqueo del dispositivo para una orden de Data Box está cifrada con una clave administrada de Microsoft. Para tener un mayor control sobre las claves de cifrado del dispositivo, puede proporcionar también claves administradas por el cliente. Las claves administradas por el cliente se deben crear y almacenar en una instancia de Azure Key Vault.

* [Protección de datos y seguridad de Azure Data Box](https://docs.microsoft.com/azure/databox/data-box-security)

* [Uso de claves administradas por el cliente en Azure Key Vault para Azure Data Box](https://docs.microsoft.com/azure/databox/data-box-customer-managed-encryption-key-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: use Azure Monitor con el registro de actividad de Azure para crear alertas cuando se produzcan cambios en Azure Data Box así como y en otros recursos críticos o relacionados.

* [Creación de alertas para los eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log)

**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Control de seguridad: Administración de vulnerabilidades](https://docs.microsoft.com/azure/security/benchmarks/security-control-vulnerability-management).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Azure Data Box.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Guía**: cuando se envía Data Box, se instala con las actualizaciones más recientes. No se realizan actualizaciones de campos.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3: Implementación de una solución de administración de revisiones automatizada de títulos de software de terceros

**Guía**: cuando se envía Data Box, se instala con las actualizaciones más recientes. No se realizan actualizaciones de campos.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Instrucciones**: Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Azure Data Box.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Instrucciones**: Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Azure Data Box.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para más información, consulte [Control de seguridad: Administración de recursos y del inventario](https://docs.microsoft.com/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Instrucciones**: no es aplicable, no hay recursos de Data Box para detectar.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Instrucciones**: no es aplicable, no hay metadatos de recursos para Data Box.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: no es aplicable, el servicio Data Box garantiza que no se use ningún recurso de Azure no autorizado.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: no es aplicable, no hay ninguno en el nivel de servicio de Data Box.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Instrucciones**: no es aplicable, no hay ninguna en el nivel de servicio Data Box.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Instrucciones**: no es aplicable, no hay ninguna en el nivel de servicio Data Box.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: no es aplicable, no hay ninguna en el nivel de servicio Data Box.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Guía**: no es aplicable, no hay ninguna en el nivel de servicio Data Box.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: no es aplicable, Data Box Services solo usa los servicios de Azure aprobados.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10: Mantenimiento de un inventario de títulos de software aprobados

**Guía**: no es aplicable, Data Box Services solo usa los títulos de software aprobados.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: Configure el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management.

* [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12: Limitación de capacidad de los usuarios para ejecutar scripts en recursos de proceso

**Guía**: no es aplicable, el servicio de Data Box no admite la ejecución de scripts.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Guía**: no es aplicable, el servicio de Data Box no tiene aplicaciones web que se ejecutan en el servicio Azure App.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Control de seguridad: Configuración segura](https://docs.microsoft.com/azure/security/benchmarks/security-control-secure-configuration).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Azure Data Box viene con la configuración de seguridad de procedimientos recomendados predeterminada.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Instrucciones**: Azure Data Box viene con la configuración de seguridad de procedimientos recomendados predeterminada.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Azure Data Box viene con la configuración de seguridad de procedimientos recomendados predeterminados para los recursos y no se puede cambiar.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Guía**: Azure Data Box viene con la configuración de seguridad de procedimientos recomendados predeterminados para los recursos y no se puede cambiar.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: todas las configuraciones de Data Box se almacenan de forma segura.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Instrucciones**: todas las imágenes del sistema operativo de Data Box se almacenan de forma segura.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Microsoft

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: no es aplicable, las configuraciones de Azure Data Box no se pueden cambiar.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8: Implementación de herramientas de administración de la configuración para sistemas operativos

**Guía**: no es aplicable, las configuraciones de Azure Data Box no se pueden cambiar.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: no es aplicable, las configuraciones de Azure Data Box no se pueden cambiar.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Guía**: no es aplicable, las configuraciones de Azure Data Box no se pueden cambiar.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: Las claves administradas por el cliente se deben crear y almacenar en una instancia de Azure Key Vault.

* [Uso de claves administradas por el cliente en Azure Key Vault para Azure Data Box](https://docs.microsoft.com/azure/databox/data-box-customer-managed-encryption-key-portal)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Instrucciones**: no es aplicable, Azure Data Box no utiliza identidades administradas.

* [Servicios de Azure que admiten identidades administradas para recursos de Azure](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Microsoft ejecuta el escáner de credenciales en el código de Data Box. Además, protege también de forma segura las credenciales. Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

* [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervisión de Azure Security Center**: N/D

**Responsabilidad**: Compartido

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Control de seguridad: defensa contra malware](https://docs.microsoft.com/azure/security/benchmarks/security-control-malware-defense).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1: Use software antimalware administrado centralmente

**Guía**: No aplicable; esta guía está pensada para recursos de proceso. Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure App Service), pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Instrucciones**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Caja de seguridad del cliente de Azure), aunque no se ejecuta en el contenido del cliente.

Es su responsabilidad realizar un análisis previo del contenido que se carga en los recursos de Azure que no son de proceso. Microsoft no puede acceder a los datos del cliente y, por lo tanto, no puede realizar exámenes antimalware del contenido del cliente en su nombre.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3: Asegúrese de que se han actualizado el software y las firmas antimalware

**Instrucciones**: No aplicable; esta recomendación está pensada para los recursos de proceso. Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure, pero no se ejecuta en el contenido del cliente.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Control de seguridad: recuperación de datos](https://docs.microsoft.com/azure/security/benchmarks/security-control-data-recovery).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: no es aplicable, el servicio de Data Box no requiere copias de seguridad.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: No aplicable

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copias de seguridad de las claves administradas por el cliente

**Instrucciones**: Asegúrese de hacer una copia de seguridad de los datos y de la clave administrada del cliente. Data Box no realiza ninguna copia de seguridad.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: debe comprobar que todos los datos estén en la cuenta de Azure Storage antes de eliminarlos de su entorno local.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: Asegúrese de que las copias de seguridad o las claves administradas por el cliente que tiene están protegidas de acuerdo con los procedimientos recomendados.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Control de seguridad: Respuesta a los incidentes](https://docs.microsoft.com/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

* [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

* [Uso de la guía de control de incidentes de seguridad de equipos de NIST para la creación de su propio plan de respuesta a incidentes](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervisión de Azure Security Center**: No aplicable

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

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Guía**: Use la característica Automatización de flujo de trabajo de Azure Security Center para desencadenar automáticamente las respuestas a través de Logic Apps en las alertas y recomendaciones de seguridad para proteger los recursos de Azure.

* [Configuración de la automatización de flujo de trabajo y Logic Apps](https://docs.microsoft.com/azure/security-center/workflow-automation)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Control de seguridad: Pruebas de penetración y ejercicios del equipo rojo](https://docs.microsoft.com/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: Microsoft realiza pruebas de penetración y análisis de vulnerabilidades en los dispositivos Data Box. Puede realizar sus propias pruebas de penetración y análisis de vulnerabilidades. Si decide hacerlo, siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de pruebas de penetración de sitios activos y ataques simulados en la infraestructura en la nube, los servicios y las aplicaciones administrados por Microsoft.

* [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

* [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Compartido

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).
