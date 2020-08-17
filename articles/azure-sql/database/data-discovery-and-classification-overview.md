---
title: Clasificación y detección de datos
description: Clasificación y detección de datos para Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse Analytics
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: sqldbrb=1
titleSuffix: Azure SQL Database, Azure SQL Managed Instance, and Azure Synapse
ms.devlang: ''
ms.topic: conceptual
author: DavidTrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 04/28/2020
tags: azure-synapse
ms.openlocfilehash: 3f1f284255d1c0e77779c175951eaf33d3e56067
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87004110"
---
# <a name="data-discovery--classification"></a>Clasificación y detección de datos
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

La clasificación y detección de datos está integrada en Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse Analytics. Ofrece funcionalidades avanzadas para detectar, clasificar, etiquetar e informar de los datos confidenciales de las bases de datos.

Los datos más confidenciales pueden incluir información empresarial, financiera, sanitaria o personal. La detección y clasificación de estos datos puede representar un rol fundamental en el enfoque de la protección de la información de la organización. Puede servir como infraestructura para lo siguiente:

- Ayudar a satisfacer los estándares de privacidad de datos y los requisitos de cumplimiento normativo.
- Varios escenarios de seguridad, como la supervisión (auditoría) y las alertas relacionadas con accesos anómalos a información confidencial.
- Controlar el acceso y mejorar la seguridad de las bases de datos que contienen información altamente confidencial.

La clasificación y detección de datos forma parte de la oferta de [Advanced Data Security](advanced-data-security.md) un paquete unificado para funcionalidades avanzadas de seguridad de Azure SQL. Puede obtener acceso y administrar la clasificación y detección de datos a través de la sección central de **Advanced Data Security de SQL** de Azure Portal.

> [!NOTE]
> Para información sobre SQL Server local, consulte [Clasificación y detección de datos de SQL](https://go.microsoft.com/fwlink/?linkid=866999).

## <a name="what-is-data-discovery--classification"></a><a id="what-is-dc"></a>¿Qué es la clasificación y detección de datos?

La clasificación y detección de datos presenta un conjunto de servicios avanzados y nuevas capacidades de Azure. Constituye un nuevo paradigma de protección de la información para SQL Database, Instancia administrada de SQL y Azure Synapse, destinado no solo a proteger la base de datos, sino también los datos. El paradigma incluye:

- **Detección y recomendaciones:** el motor de clasificación examina la base de datos e identifica las columnas que contienen datos potencialmente confidenciales. A continuación, le proporciona una manera sencilla de revisar y aplicar la clasificación recomendada a través de Azure Portal.

- **Etiquetado:** puede aplicar etiquetas de clasificación de confidencialidad de forma persistente a las columnas, usando los nuevos atributos de metadatos que se han agregado al motor de base de datos de SQL Server. A continuación, estos metadatos se pueden utilizar para escenarios avanzados de auditoría y protección basados en la confidencialidad.

- **Confidencialidad del conjunto de resultados de consulta:** la confidencialidad del conjunto de resultados de consulta se calcula en tiempo real con fines de auditoría.

- **Visibilidad**: puede ver el estado de clasificación de la base de datos en un panel detallado en Azure Portal. Además, puede descargar un informe en formato Excel para usarlo con fines de auditoría y cumplimiento de normas, así como para otras necesidades.

## <a name="discover-classify-and-label-sensitive-columns"></a><a id="discover-classify-columns"></a>Detección, clasificación y etiquetado de columnas confidenciales

En esta sección se describen los pasos para:

- La detección, clasificación y etiquetado de columnas que contienen información confidencial en la base de datos.
- La visualización del estado de clasificación actual de la base de datos y exportación de informes.

La clasificación incluye dos atributos de metadatos:

- **Etiquetas**: los atributos principales de clasificación, que se usan para definir el nivel de confidencialidad de los datos almacenados en la columna.  
- **Tipos de información**: atributos que proporcionan información más detallada sobre el tipo de datos almacenados en la columna.

### <a name="define-and-customize-your-classification-taxonomy"></a>Definir y personalizar la taxonomía de clasificación

La función de clasificación y detección de datos incluye un conjunto integrado de etiquetas de confidencialidad y un conjunto integrado de tipos de información y lógica de detección. Ahora puede personalizar esta taxonomía y definir un conjunto y la categoría de construcciones de clasificación específicamente para su entorno.

Se define y personaliza la taxonomía de clasificación en una ubicación central para toda la organización de Azure. Esa ubicación se encuentra en [Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-intro), como parte de la directiva de seguridad. Solo un usuario con derechos administrativos en el grupo de administración raíz de la organización puede realizar esta tarea.

Como parte de la administración de directivas de protección de la información, puede definir etiquetas personalizadas, clasificarlas y asociarlas con un conjunto seleccionado de tipos de información. También puede agregar sus propios tipos de información personalizados y configurarlos con patrones de cadena. Los patrones se agregan a la lógica de detección para identificar este tipo de datos en las bases de datos.

Para más información, consulte [Personalización de la directiva de SQL Information Protection en Azure Security Center (versión preliminar)](https://go.microsoft.com/fwlink/?linkid=2009845&clcid=0x409).

Una vez definida la directiva de toda la organización, puede continuar con la clasificación de bases de datos individuales mediante la directiva personalizada.

### <a name="classify-your-database"></a>Clasificación de la base de datos

> [!NOTE]
> En el ejemplo siguiente se usa Azure SQL Database, pero debe seleccionar el producto adecuado para el que quiera configurar la clasificación y detección de datos.

1. Vaya a [Azure Portal](https://portal.azure.com).

2. Vaya a **Advanced Data Security** en el encabezado **Seguridad** del panel de Azure SQL Database. Seleccione **Advanced Data Security** y luego seleccione la tarjeta **Clasificación y detección de datos**.

   ![Panel Advanced Data Security de Azure Portal](./media/data-discovery-and-classification-overview/data_classification.png)

3. En la página **Clasificación y detección de datos**, en la pestaña **Información general** se incluye un resumen del estado de clasificación actual de la base de datos. El resumen incluye una lista detallada de todas las columnas clasificadas, que también se pueden filtrar para mostrar solo partes específicas de esquema, tipos de información y etiquetas. Si aún no ha clasificado ninguna columna, [vaya al paso 5](#step-5).

   ![Resumen del estado de clasificación actual](./media/data-discovery-and-classification-overview/2_data_classification_overview_dashboard.png)

4. Para descargar un informe en formato de Excel, seleccione **Exportar** en el menú superior del panel.

5. <a id="step-5"></a>Para empezar a clasificar los datos, seleccione la pestaña **Clasificación** en la página **Clasificación y detección de datos**.

    El motor de clasificación examina la base de datos en busca de columnas que contengan datos potencialmente confidenciales y proporciona una lista de clasificaciones de columna recomendadas.

6. Visualice y aplique las recomendaciones de clasificación:

   - Para ver la lista de clasificaciones de columnas recomendadas, seleccione el panel de recomendaciones en la parte inferior del panel.

   - Para aceptar una recomendación para una columna específica, seleccione la casilla en la columna izquierda de la fila correspondiente. Para marcar todas las recomendaciones como aceptadas, seleccione la casilla más a la izquierda del encabezado de la tabla de recomendaciones.

       ![Revisión y selección en la lista de recomendaciones de clasificación](./media/data-discovery-and-classification-overview/6_data_classification_recommendations_list.png)

   - Para aplicar las recomendaciones seleccionadas, seleccione **Aceptar las recomendaciones seleccionadas**.

7. También puede clasificar las columnas manualmente, como alternativa a la clasificación basada en recomendaciones, o además de ella:

   1. Seleccione **Agregar clasificación** en el menú superior del panel.

   1. En la ventana contextual que se abre, seleccione el esquema, la tabla y la columna que quiera clasificar, así como el tipo de información y la etiqueta de confidencialidad.

   1. Seleccione **Agregar clasificación** en la parte inferior de la ventana contextual.

      ![Selección de una columna para clasificar](./media/data-discovery-and-classification-overview/9_data_classification_manual_classification.png)

8. Para completar la clasificación y etiquetar de forma persistente las columnas de la base de datos con los nuevos metadatos de clasificación, seleccione **Guardar** en el menú superior de la ventana.

## <a name="audit-access-to-sensitive-data"></a><a id="audit-sensitive-data"></a>Auditoría del acceso a datos confidenciales

Un aspecto importante del paradigma de protección de la información es la capacidad de supervisar el acceso a información confidencial. [Auditoría de Azure SQL](../../azure-sql/database/auditing-overview.md) se ha mejorado para incluir un nuevo campo en el registro de auditoría denominado `data_sensitivity_information`. Este campo registra las clasificaciones de confidencialidad (etiquetas) de los datos devueltos por una consulta. Este es un ejemplo:

![Registro de auditoría](./media/data-discovery-and-classification-overview/11_data_classification_audit_log.png)

## <a name="permissions"></a><a id="permissions"></a>Permisos

Estos roles integrados pueden leer la clasificación de datos de una base de datos:

- Propietario
- Lector
- Colaborador
- Administrador de seguridad SQL
- Administrador de acceso de usuario

Estos roles integrados pueden modificar la clasificación de datos de una base de datos:

- Propietario
- Colaborador
- Administrador de seguridad SQL

Obtenga información sobre los permisos basados en roles en [RBAC de Azure](https://docs.microsoft.com/azure/role-based-access-control/overview).

## <a name="manage-classifications"></a><a id="manage-classification"></a>Administración de clasificaciones

Puede usar T-SQL, una API de REST o PowerShell para administrar las clasificaciones.

### <a name="use-t-sql"></a>Uso de T-SQL

Puede utilizar T-SQL para agregar o quitar las clasificaciones de columna, y para recuperar todas las clasificaciones para toda la base de datos.

> [!NOTE]
> Al usar T-SQL para administrar las etiquetas, no se valida que las etiquetas que se agregan a una columna existan en la directiva de protección de información de la organización (el conjunto de etiquetas que aparecen en las recomendaciones del portal). Por lo tanto, validarlas es su decisión.

Para obtener información sobre el uso de T-SQL para las clasificaciones, vea las siguientes referencias:

- Para agregar o actualizar la clasificación de una o varias columnas: [ADD SENSITIVITY CLASSIFICATION](https://docs.microsoft.com/sql/t-sql/statements/add-sensitivity-classification-transact-sql) (Agregar clasificación de confidencialidad)
- Para quitar la clasificación de una o varias columnas: [DROP SENSITIVITY CLASSIFICATION](https://docs.microsoft.com/sql/t-sql/statements/drop-sensitivity-classification-transact-sql) (Eliminar clasificación de la confidencialidad)
- Para ver todas las clasificaciones de la base de datos: [sys.sensitivity_classifications](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-sensitivity-classifications-transact-sql)

### <a name="use-powershell-cmdlets"></a>Uso de cmdlets de PowerShell
Administre clasificaciones y las recomendaciones de Azure SQL Database e Instancia administrada de Azure SQL mediante PowerShell.

#### <a name="powershell-cmdlets-for-azure-sql-database"></a>Cmdlets de PowerShell para Azure SQL Database

- [Get-AzSqlDatabaseSensitivityClassification](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabasesensitivityclassification)
- [Set-AzSqlDatabaseSensitivityClassification](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabasesensitivityclassification)
- [Remove-AzSqlDatabaseSensitivityClassification](https://docs.microsoft.com/powershell/module/az.sql/remove-azsqldatabasesensitivityclassification)
- [Get-AzSqlDatabaseSensitivityRecommendation](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabasesensitivityrecommendation)
- [Enable-AzSqlDatabaSesensitivityRecommendation](https://docs.microsoft.com/powershell/module/az.sql/enable-azsqldatabasesensitivityrecommendation)
- [Disable-AzSqlDatabaseSensitivityRecommendation](https://docs.microsoft.com/powershell/module/az.sql/disable-azsqldatabasesensitivityrecommendation)

#### <a name="powershell-cmdlets-for-azure-sql-managed-instance"></a>Cmdlets de PowerShell para Instancia administrada de Azure SQL

- [Get-AzSqlInstanceDatabaseSensitivityClassification](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlinstancedatabasesensitivityclassification)
- [Set-AzSqlInstanceDatabaseSensitivityClassification](https://docs.microsoft.com/powershell/module/az.sql/set-azsqlinstancedatabasesensitivityclassification)
- [Remove-AzSqlInstanceDatabaseSensitivityClassification](https://docs.microsoft.com/powershell/module/az.sql/remove-azsqlinstancedatabasesensitivityclassification)
- [Get-AzSqlInstanceDatabaseSensitivityRecommendation](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlinstancedatabasesensitivityrecommendation)
- [Enable-AzSqlInstanceDatabaseSensitivityRecommendation](https://docs.microsoft.com/powershell/module/az.sql/enable-azsqlinstancedatabasesensitivityrecommendation)
- [Disable-AzSqlInstanceDatabaseSensitivityRecommendation](https://docs.microsoft.com/powershell/module/az.sql/disable-azsqlinstancedatabasesensitivityrecommendation)

### <a name="use-the-rest-api"></a>Uso de la API de REST

Puede usar las API REST para administrar las clasificaciones y recomendaciones mediante programación. Las API de REST publicadas admiten las siguientes operaciones:

- [Crear o actualizar](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/createorupdate): crea o actualiza la etiqueta de confidencialidad de la columna especificada.
- [Eliminar](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/delete): elimina la etiqueta de confidencialidad de la columna especificada.
- [Deshabilitar la recomendación](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/disablerecommendation): deshabilita las recomendaciones de confidencialidad en la columna especificada.
- [Habilitar la recomendación](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/enablerecommendation): habilita las recomendaciones de confidencialidad en la columna especificada. (Las recomendaciones están habilitadas de forma predeterminada en todas las columnas).
- [Obtener](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/get): obtiene la etiqueta de confidencialidad de la columna especificada.
- [Enumerar las actuales por base de datos](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/listcurrentbydatabase): obtiene las etiquetas de confidencialidad actuales de la base de datos especificada.
- [Enumerar las recomendadas por base de datos](https://docs.microsoft.com/rest/api/sql/sensitivitylabels/listrecommendedbydatabase): obtiene las etiquetas de confidencialidad recomendadas de la base de datos especificada.

## <a name="next-steps"></a><a id="next-steps"></a>Pasos siguientes

- Más información sobre [Advanced Data Security](advanced-data-security.md)
- Considere la posibilidad de configurar [Auditoría de Azure SQL](../../azure-sql/database/auditing-overview.md) para supervisar y auditar el acceso a los datos confidenciales clasificados.
- Para ver una presentación que incluye la detección y clasificación de datos, consulte [Detección, clasificación, etiquetado y protección de datos de SQL | Datos expuestos](https://www.youtube.com/watch?v=itVi9bkJUNc).
