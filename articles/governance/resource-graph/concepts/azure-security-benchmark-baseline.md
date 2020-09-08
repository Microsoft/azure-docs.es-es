---
title: Línea de base de seguridad de Azure Resource Graph para Azure Security Benchmark
description: La base de referencia de seguridad de Azure Resource Graph proporciona instrucciones de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: resource-graph
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: d2ef76a054642807f0d72a758ae084a19557caf8
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009008"
---
# <a name="azure-resource-graph-security-baseline-for-azure-security-benchmark"></a>Línea de base de seguridad de Azure Resource Graph para Azure Security Benchmark

Esta línea de base de seguridad aplica la guía de [Azure Security Benchmark](../../../security/benchmarks/overview.md) a Azure Resource Graph. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. El contenido se agrupa por los **controles de seguridad** definidos por Azure Security Benchmark y las instrucciones relacionadas aplicables a Azure Resource Graph. Se han excluido los **controles** no aplicables a Azure Resource Graph. Para ver cómo Azure Resource Graph se asigna por completo a Azure Security Benchmark, consulte el [archivo de asignación de base de referencia de seguridad de Azure Virtual Network completo](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).



## <a name="identity-and-access-control"></a>Control de identidades y acceso

*Para más información, consulte [Control de seguridad: Identidad y control de acceso](../../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure Resource Graph proporciona acceso a tipos de recursos y propiedades según el control de acceso basado en rol de Azure (RBAC de Azure). Audite y revise el acceso concedido a las entidades de seguridad (usuarios, grupos y cuentas de servicio) periódicamente para asegurarse de que las consultas devuelven resultados para los recursos adecuados.

* [Permisos en Azure Resource Graph](../overview.md#permissions-in-azure-resource-graph)

* [Procedimiento para usar las revisiones de acceso de identidad de Azure](../../../active-directory/governance/access-reviews-overview.md)


**Supervisión de Azure Security Center**: Sí

**Responsabilidad**: Customer

## <a name="data-protection"></a>Protección de los datos

*Para más información, consulte [Control de seguridad: Protección de datos](../../../security/benchmarks/security-control-data-protection.md).*

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6: Uso del control de acceso basado en rol para controlar el acceso a los recursos

**Guía**: Use el control de acceso basado en rol (RBAC) para controlar el acceso a datos y recursos. Para usar Azure Resource Graph, también debe tener el acceso adecuado a los recursos que desea consultar. El ámbito de este acceso debe ser de solo lectura y solo se puede conceder al personal requerido.

* [Permisos en Azure Resource Graph](../overview.md#permissions-in-azure-resource-graph)

* [Configuración de RBAC en Azure](../../../role-based-access-control/role-assignments-rest.md)


**Supervisión de Azure Security Center**: no disponible actualmente

**Responsabilidad**: Customer

## <a name="inventory-and-asset-management"></a>Inventario y administración de recursos

*Para más información, consulte [Control de seguridad: Administración de recursos y del inventario](../../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: Use Azure Resource Graph para consultar y detectar todos los recursos admitidos en las suscripciones, los grupos de administración y los inquilinos. Asegúrese de que tiene los permisos adecuados en el suscriptor y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

* [Creación de consultas con Azure Resource Graph](../first-query-portal.md)

* [Descripción de Azure RBAC](../../../role-based-access-control/overview.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: Cree un inventario de los recursos de Azure aprobados y el software aprobado para los recursos de proceso según las necesidades de la organización. Use Azure Resource Graph para consultar los recursos de Azure aprobados y el historial de cambios (versión preliminar) para revisar las instantáneas y ver lo que ha cambiado.

* [Consulta de recursos de Azure con Azure Resource Graph](../first-query-portal.md)

* [Obtención de los cambios del recurso](../how-to/get-resource-changes.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Instrucciones**: Use Azure Resource Graph para consultar y detectar los recursos en las suscripciones, los grupos de administración y los inquilinos. Asegúrese de que todos los recursos de Azure del entorno estén aprobados.

* [Consulta de recursos de Azure con Azure Resource Graph](../first-query-portal.md)

* [Ejemplos: Consultas de inicio de Azure Resource Graph](../samples/starter.md)


**Supervisión de Azure Security Center**: No aplicable

**Responsabilidad**: Customer

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [prueba comparativa de seguridad de Azure](../../../security/benchmarks/overview.md).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](../../../security/benchmarks/security-baselines-overview.md).
