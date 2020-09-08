---
title: Administración de los costos y el uso de AWS en Azure Cost Management
description: Este artículo le ayudará a comprender cómo usar el análisis de costos y los presupuestos en Cost Management para administrar los costos y el uso de AWS.
author: bandersmsft
ms.author: banders
ms.date: 08/28/2020
ms.topic: how-to
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: matrive
ms.custom: ''
ms.openlocfilehash: 7df27a6ed288555d0f4815223fd0bb6dddff6f44
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266224"
---
# <a name="manage-aws-costs-and-usage-in-azure"></a>Administración de los costos y el uso de AWS en Azure

Una vez que haya instalado y configurado la integración de informes de uso y costos de AWS para Azure Cost Management, estará listo para empezar a administrar los costos y el uso de AWS. Este artículo le ayudará a comprender cómo usar el análisis de costos y los presupuestos en Cost Management para administrar los costos y el uso de AWS.

Si no ha configurado aún la integración, consulte [Set up and configure AWS Usage report integration](aws-integration-set-up-configure.md) (Instalación y configuración de la integración de los informes de uso de AWS).

_Antes de empezar_: Si está familiarizado con el análisis de costos, consulte [Quickstart: Explore and analyze costs with Cost analysis](quick-acm-cost-analysis.md) (Inicio rápido: exploración y análisis de los costos con Análisis de costos). Y, si no está familiarizado con los presupuestos en Azure, consulte [Tutorial: Create and manage Azure budgets](tutorial-acm-create-budgets.md) (Tutorial: creación y administración de presupuestos en Azure).

## <a name="view-aws-costs-in-cost-analysis"></a>Visualización de los costos de AWS en Análisis de costos

Los costos de AWS están disponibles en Análisis de costos en los siguientes ámbitos:

- Cuentas vinculadas de AWS en un grupo de administración
- Costos de las cuentas vinculadas de AWS
- Costos de las cuentas consolidadas de AWS

En las secciones siguientes se describe cómo utilizar los ámbitos para ver datos de uso y costo de cada uno.

### <a name="view-aws-linked-accounts-under-a-management-group"></a>Visualización de las cuentas vinculadas de AWS en un grupo de administración

Ver los costos mediante el ámbito de grupo de administración es la única manera de ver los costos agregados procedentes de distintas suscripciones a Azure y cuentas vinculadas de AWS. El uso de un grupo de administración proporciona una vista de toda la nube, para ver los costos de Azure y de AWS juntos.

En Análisis de costos, abra el selector de ámbito y seleccione el grupo de administración que contenga las cuentas vinculadas de AWS. Este es un ejemplo de imagen en Azure Portal:

:::image type="content" source="./media/aws-integration-manage/select-scope01.png" alt-text="Ejemplo de la vista de selección de ámbito con cuentas vinculadas en un grupo de administración" :::

Este es un ejemplo que muestra el costo del grupo de administración en Análisis de costos agrupado por proveedor (Azure y AWS).

:::image type="content" source="./media/aws-integration-manage/cost-analysis-aws-azure.png" alt-text="Ejemplo que muestra los costos de Azure y AWS de un trimestre en Análisis de costos" lightbox="./media/aws-integration-manage/cost-analysis-aws-azure.png" :::

> [!NOTE]
> Los grupos de administración no se admiten actualmente para clientes con Contrato de cliente de Microsoft (MCA). Los clientes MCA pueden crear el conector y ver sus datos de AWS. Sin embargo, los clientes MCA no pueden ver los costos de Azure y los costos de AWS juntos en un grupo de administración.

### <a name="view-aws-linked-account-costs"></a>Visualización de los costos de las cuentas vinculadas de AWS

Para ver los costos de una cuenta vinculada de AWS, abra el selector de ámbito y selecciónela. Tenga en cuenta que las cuentas vinculadas están asociadas a un grupo de administración, tal como se define en el conector de AWS.

Este es un ejemplo que muestra la selección de un ámbito de cuenta vinculada de AWS.

:::image type="content" source="./media/aws-integration-manage/select-scope02.png" alt-text="Ejemplo de la vista de selección de ámbito que muestra cuentas vinculadas de AWS" :::

### <a name="view-aws-consolidated-account-costs"></a>Visualización de los costos de las cuentas consolidadas de AWS

Para ver los costos de una cuenta consolidada de AWS, abra el selector de ámbito y selecciónela. Este es un ejemplo que muestra la selección de un ámbito de cuenta consolidada de AWS.

:::image type="content" source="./media/aws-integration-manage/select-scope03.png" alt-text="Ejemplo de la vista de selección de ámbito con cuentas consolidadas" :::

Este ámbito proporciona una vista agregada de todas las cuentas vinculadas de AWS asociadas con la cuenta consolidada de AWS. Este es un ejemplo que muestra los costos de una cuenta consolidada de AWS agrupados por nombre de servicio.

:::image type="content" source="./media/aws-integration-manage/cost-analysis-aws-consolidated.png" alt-text="Ejemplo que muestra los costos consolidados de AWS en Análisis de costos" lightbox="./media/aws-integration-manage/cost-analysis-aws-consolidated.png" :::

### <a name="dimensions-available-for-filtering-and-grouping"></a>Dimensiones disponibles para el filtrado y el agrupamiento

En la siguiente tabla se describen las dimensiones disponibles de agrupamiento y filtrado en Análisis de costos.

| Dimensión | Encabezado de archivo CUR de Amazon | Ámbitos | Comentarios |
| --- | --- | --- | --- |
| Zona de disponibilidad | LineItem/AvailabilityZone | All |   |
| Location | product/Region | All |   |
| Medidor |   | All |   |
| Meter category | lineItem/ProductCode | All |   |
| Subcategoría del medidor | lineitem/UsageType | All |   |
| Operación | lineItem/Operation | All |   |
| Resource | lineItem/ResourceId | All |   |
| Tipo de recurso | product/instanceType | All | Si product/instanceType es nulo, se usa lineItem/UsageType. |
| ResourceGuid | N/D | All | Identificador único global de medición de Azure. |
| Nombre del servicio | product/ProductName | All | Si product/ProductName es nulo, se usa lineItem/ProductCode. |
| Nivel de servicio |   |   |   |
| Id. de suscripción | lineItem/UsageAccountId | Cuenta consolidada y grupo de administración |   |
| Nombre de suscripción | N/D | Cuenta consolidada y grupo de administración | Los nombres de cuenta se recopilan mediante la API de AWS Organizations. |
| Etiqueta | resourceTags | All | El _usuario:_ el prefijo se quita de las etiquetas definidas por el usuario para permitir las etiquetas de toda la nube. El _aws:_ prefijo que se deja intacto. |
| Identificador de la cuenta de facturación | bill/PayerAccountId | Grupo de administración |   |
| Nombre de la cuenta de facturación | N/D | Grupo de administración | Los nombres de cuenta se recopilan mediante la API de AWS Organizations. |
| Proveedor | N/D | Grupo de administración | O AWS o Azure. |

## <a name="set-budgets-on-aws-scopes"></a>Establecimiento de presupuestos en los ámbitos de AWS

Use presupuestos para administrar los costos de manera proactiva e impulsar la responsabilidad en la organización. Los presupuestos se establecen en los ámbitos de cuenta consolidada de AWS y cuenta vinculada de AWS. Este es un ejemplo de presupuestos para una cuenta consolidada de AWS como aparece en Cost Management:

:::image type="content" source="./media/aws-integration-manage/budgets-aws-consolidated-account01.png" alt-text="Ejemplo que muestra los presupuestos para una cuenta consolidada de AWS" :::

## <a name="aws-data-collection-process"></a>Proceso de recopilación de datos de AWS

Después de configurar el conector de AWS, se inician los procesos de recopilación y detección de datos. Los datos de uso pueden tardar unas horas en recopilarse. La duración depende de lo siguiente:

- El tiempo necesario para procesar los archivos CUR que se encuentran en el cubo de AWS S3.
- El tiempo necesario para crear los ámbitos de cuenta consolidada de AWS y cuenta vinculada de AWS.
- El tiempo que AWS escribe en los archivos de los informes de uso y costos en el cubo S3, y la frecuencia con la que lo hace.

## <a name="aws-integration-pricing"></a>Precios de integración de AWS

Cada conector de AWS obtiene 90 días de prueba gratuitos.

El precio de lista es el 1 % de los costos mensuales de AWS. Cada mes se cobra en función de los costos facturados del mes anterior.

El acceso a las API de AWS puede suponer costos adicionales de AWS.

## <a name="aws-integration-limitations"></a>Limitaciones de la integración de AWS

- Los presupuestos de Cost Management no admiten grupos de administración con varias divisas. Los grupos de administración con varias divisas no verán una evaluación de presupuesto. Se muestra un mensaje de error si selecciona un grupo de administración que tiene varias divisas al crear un presupuesto.
- Los conectores de nube no admiten AWS GovCloud (US), AWS Gov ni AWS China.
- Cost Management solo muestra los _costos de uso_ de AWS. Aún no se admiten impuestos, soporte técnico, reembolsos, instancias reservadas, créditos ni otros tipos de cargos.

## <a name="troubleshooting-aws-integration"></a>Solución de problemas de integración de AWS

Use la siguiente información de solución de problemas para resolver los comunes.

### <a name="no-permission-to-aws-linked-accounts"></a>Falta de permiso a las cuentas vinculadas de AWS

**Código de error:** _Unauthorized_

Hay dos maneras de obtener permisos para acceder a los costos de las cuentas vinculadas de AWS:

- Obtener acceso al grupo de administración que tenga las cuentas vinculadas de AWS.
- Que alguien le dé permisos para la cuenta vinculada de AWS.

De forma predeterminada, el creador del conector de AWS es el propietario de todos los objetos creados por el conector, incluidas la cuenta consolidada de AWS y la cuenta vinculada de AWS.

Para poder comprobar la configuración del conector, necesita al menos un rol de colaborador; un lector no puede comprobar la configuración del conector.

### <a name="collection-failed-with-assumerole"></a>Error de recopilación con AssumeRole

**Código de error:** _FailedToAssumeRole_

Este error significa que Cost Management no puede llamar a la API de AWS AssumeRole. Esto puede ocurrir debido a un problema con la definición de roles. Compruebe que se cumplan las condiciones siguientes:

- El identificador externo es el mismo que el de la definición de roles y de la definición del conector.
- El tipo de rol está establecido en **Another AWS account Belonging to you or 3rd party** (Otra cuenta de AWS suya o de terceros).
- La opción **Require MFA** (Requerir MFA) está desactivada.
- La cuenta de AWS de confianza en el rol de AWS es _432263259397_.

### <a name="collection-failed-with-access-denied---cur-report-definitions"></a>Error de recopilación con acceso denegado: definiciones de informe CUR

**Código de error:** _AccessDeniedReportDefinitions_

Este error significa que Cost Management no puede ver las definiciones de los informes de Costo y Uso. Este permiso se usa para comprobar que el valor de CUR está definido tal como se espera en Azure Cost Management. Consulte [Crear un informe de uso y costo en AWS](aws-integration-set-up-configure.md#create-a-cost-and-usage-report-in-aws).

### <a name="collection-failed-with-access-denied---list-reports"></a>Error de recopilación con acceso denegado: enumerar informes

**Código de error:** _AccessDeniedListReports_

Este error significa que Cost Management no puede enumerar el objeto en el cubo S3 donde se encuentra CUR. La directiva IAM de AWS requiere un permiso en el cubo y en los objetos del mismo. Consulte [Crear un rol y una directiva en AWS](aws-integration-set-up-configure.md#create-a-role-and-policy-in-aws).

### <a name="collection-failed-with-access-denied---download-report"></a>Error de recopilación con acceso denegado: descargar informe

**Código de error:** _AccessDeniedDownloadReport_

Estos mensajes de error significan que Cost Management no puede acceder a los archivos CUR almacenados en el cubo S3 de Amazon ni descargarlos. Asegúrese de que la directiva JSON de AWS asociada al rol sea similar al ejemplo que se muestra en la parte inferior de la sección [Create a role and policy in AWS](aws-integration-set-up-configure.md#create-a-role-and-policy-in-aws) (Creación de un rol y de una directiva en AWS).

### <a name="collection-failed-since-we-did-not-find-the-cost-and-usage-report"></a>Error en la colección porque no encontramos el informe sobre uso y costes

**Código de error:** _FailedToFindReport_

Este error significa que Cost Management no puede encontrar el informe de uso y costos que se definió en el conector. Asegúrese de que no se haya eliminado y de que la directiva JSON de AWS asociada al rol sea similar al ejemplo que se muestra en la parte inferior de la sección [Create a role and policy in AWS](aws-integration-set-up-configure.md#create-a-role-and-policy-in-aws) (Creación de un rol y de una directiva en AWS).

### <a name="unable-to-create-or-verify-connector-due-to-cost-and-usage-report-definitions-mismatch"></a>No se puede crear ni comprobar el conector debido a la falta de coincidencia de definiciones del informe sobre uso y costes

**Código de error:** _ReportIsNotValid_

Este error está relacionado con la definición del informe sobre uso y costes de AWS y necesitamos una configuración específica para él. Consulte los requisitos en [Crear un informe de uso y costo en AWS](aws-integration-set-up-configure.md#create-a-cost-and-usage-report-in-aws).

## <a name="next-steps"></a>Pasos siguientes

- Si aún no ha configurado el entorno de Azure con grupos de administración, consulte [Initial setup of management groups](../../governance/management-groups/overview.md#initial-setup-of-management-groups) (Configuración inicial de los grupos de administración).
