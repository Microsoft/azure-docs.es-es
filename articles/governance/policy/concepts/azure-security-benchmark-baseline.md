---
title: Línea de base de seguridad de Azure Policy para Azure Security Benchmark
description: La base de referencia de seguridad de Azure Policy proporciona instrucciones de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: azure-policy
ms.topic: conceptual
ms.date: 07/02/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 6c6bd3ab87013aba04f43c95b3a986a64f2bc394
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89231476"
---
# <a name="azure-policy-security-baseline-for-azure-security-benchmark"></a>Línea de base de seguridad de Azure Policy para Azure Security Benchmark

Esta línea de base de seguridad aplica la guía de [Azure Security Benchmark](../../../security/benchmarks/overview.md) a Azure Policy. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. El contenido se agrupa por los **dominios de cumplimiento** y los **controles de seguridad** definidos por Azure Security Benchmark y las instrucciones relacionadas aplicables a Azure Policy. Se han excluido los **controles** no aplicables a Azure Policy. Para ver cómo Azure Policy se asigna por completo a Azure Security Benchmark, consulte el [archivo de asignación de base de referencia de seguridad de Azure Policy completo](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Para la asignación de los controles de Azure Security Benchmark a las definiciones de directiva integradas a través de la iniciativa integrada, vea [Cumplimiento normativo: Azure Security Benchmark](../samples/azure-security-benchmark.md).

Azure Policy usa el término _Propiedad_ en lugar de _Responsabilidad_. Para obtener detalles sobre el concepto de _Propiedad_, consulte [Definiciones de directiva de Azure Policy](./definition-structure.md#type) y [Responsabilidad compartida en la nube](../../../security/fundamentals/shared-responsibility.md).


## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Control de seguridad: registro y supervisión](../../../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Azure Policy utiliza los registros de actividad, que están habilitados automáticamente, para incluir el origen del evento, la fecha, el usuario, la marca de tiempo, las direcciones de origen y de destino, y otros elementos útiles.

* [Recopilación de registros y métricas de plataforma con Azure Monitor](../../../azure-monitor/platform/diagnostic-settings.md)

* [Descripción del registro y de los distintos tipos de registro de Azure](../../../azure-monitor/platform/platform-logs-overview.md)


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para más información, consulte [Control de seguridad: Identidad y control de acceso](../../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas. 

También puede habilitar una solución de acceso Just-in-Time/Just-Enough usando roles de [Azure AD Privileged Identity Management](../../../active-directory/privileged-identity-management/pim-configure.md) con privilegios o [Azure Resource Manager](../../../azure-resource-manager/management/overview.md).


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: Utilice PAW (estaciones de trabajo de acceso privilegiado) con la MFA configurada para iniciar la sesión y configurar recursos de Azure.

* [Más información sobre las estaciones de trabajo con privilegios de acceso](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Procedimiento para habilitar la MFA en Azure](../../../active-directory/authentication/howto-mfa-getstarted.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="data-protection"></a>Protección de los datos

*Para más información, consulte [Control de seguridad: Protección de datos](../../../security/benchmarks/security-control-data-protection.md).*

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos

**Guía**: Use el control de acceso basado en roles de Azure (RBAC de Azure) para controlar el acceso a Azure Policy.

* [Permisos de RBAC de Azure en Azure Policy](../overview.md#rbac-permissions-in-azure-policy)

* [Configuración de Azure RBAC](../../../role-based-access-control/role-assignments-portal.md)


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Instrucciones**: Use Azure Monitor con los registros de actividad para crear alertas para cuando se produzcan cambios en Azure Policy.

* [Creación de alertas para los eventos del registro de actividad de Azure](../../../azure-monitor/platform/alerts-activity-log.md)


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para más información, consulte [Control de seguridad: Administración de recursos y del inventario](../../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Instrucciones**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica en una taxonomía. Use el efecto _modificar_ de Azure Policy para notificar y aplicar el cumplimiento y la gobernanza de etiquetas coherentes.

* [Tutorial: Creación y administración de directivas](../tutorials/create-and-manage.md)

* [Tutorial: Administración de gobernanza de etiquetas](../tutorials/govern-tags.md)


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: Cree un inventario de definiciones de directivas y asignaciones de directivas aprobadas según las necesidades de su organización.

**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en sus suscripciones.

* [Configuración y administración de Azure Policy](../tutorials/create-and-manage.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](../../../security/benchmarks/overview.md).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](../../../security/benchmarks/security-baselines-overview.md).
