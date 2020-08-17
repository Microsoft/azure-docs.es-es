---
title: Enmascaramiento de datos dinámicos
description: El enmascaramiento dinámico de datos enmascara los datos confidenciales para los usuarios de Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse Analytics sin privilegios con el fin de limitar su exposición.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: DavidTrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 08/04/2020
tags: azure-synpase
ms.openlocfilehash: 14ae9103571d72b0a48ee8e1a9c9dc6bb008373b
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87552153"
---
# <a name="dynamic-data-masking"></a>Enmascaramiento de datos dinámicos 
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse Analytics admiten el enmascaramiento dinámico de datos. El enmascaramiento dinámico de datos limita la exposición de información confidencial ocultándolos a los usuarios sin privilegios. 

El enmascaramiento de datos dinámicos ayuda a impedir el acceso no autorizado a datos confidenciales permitiendo a los usuarios designar la cantidad de los datos confidenciales que se revelarán con un impacto mínimo en el nivel de aplicación. Se trata de una característica de protección de datos que oculta la información confidencial del conjunto de resultados de una consulta de campos designados de una base de datos, sin modificar los datos de esta última.

Por ejemplo, un representante de servicio de un centro de llamadas podría identificar a los autores de las llamadas a partir de varios dígitos del número de su tarjeta de crédito, pero esa es una información que no debería exponerse por completo al representante del servicio. Se puede definir una regla de enmascaramiento que enmascare todo excepto los cuatro últimos dígitos de un número de tarjeta de crédito en el conjunto de resultados de cualquier consulta. Otro ejemplo, una máscara de datos apropiada se puede definir para proteger los datos personales, para que un desarrollador pueda consultar los entornos de producción para solucionar problemas sin infringir las reglamentaciones de cumplimiento.

## <a name="dynamic-data-masking-basics"></a>Aspectos básicos del enmascaramiento dinámico de datos

Para configurar una directiva de enmascaramiento de datos dinámicos en Azure Portal, se selecciona la hoja **Enmascaramiento de datos dinámicos** en **Seguridad** en el panel de configuración de SQL Database. Esta característica no se puede establecer para Azure Synapse mediante el portal (use PowerShell o la API de REST) o Instancia administrada de SQL. Para obtener más información, vea [Dynamic Data Masking](/sql/relational-databases/security/dynamic-data-masking).

### <a name="dynamic-data-masking-permissions"></a>Permisos de enmascaramiento de datos dinámicos

El enmascaramiento dinámico de datos se puede configurar mediante el administrador de Azure SQL Database, el administrador del servidor o los roles de [Administrador de seguridad de SQL](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#sql-security-manager).

### <a name="dynamic-data-masking-policy"></a>Directiva de enmascaramiento de datos dinámicos

* **Usuarios de SQL excluidos del enmascaramiento**: conjunto de usuarios de SQL o identidades de Azure AD que obtendrán datos sin máscara en los resultados de consulta SQL. Los usuarios con privilegios de administrador se excluirán siempre del enmascaramiento y verán los datos originales sin ninguna máscara.
* **Reglas de enmascaramiento**: un conjunto de reglas que definen los campos designados para el enmascaramiento y la función de enmascaramiento que se va a usar. Los campos designados se pueden definir mediante un nombre de esquema de base de datos, un nombre de tabla y un nombre de columna.
* **Funciones de enmascaramiento** : un conjunto de métodos que controlan la exposición de datos para diferentes escenarios.

| Función de enmascaramiento | Lógica de enmascaramiento |
| --- | --- |
| **Valor predeterminado** |**Enmascaramiento completo según los tipos de datos de los campos designados**<br/><br/>• Use XXXX o menos X si el tamaño del campo es menor que 4 caracteres para los tipos de datos de cadena (nchar, ntext, nvarchar).<br/>• Use un valor de cero para los tipos de datos numéricos (bigint, bit, decimal, int, money, numeric, smallint, smallmoney, tinyint, float, real).<br/>• Use 01-01-1900 para los tipos de datos de fecha y hora (date, datetime2, datetime, datetimeoffset, smalldatetime, time).<br/>• Para variant SQL, se utiliza el valor predeterminado del tipo actual.<br/>• Para XML, se utiliza el documento \<masked/>.<br/>• Use un valor vacío para los tipos de datos especiales (tipos timestamp table, hierarchyid, GUID, binary, image, varbinary spatial). |
| **Tarjeta de crédito** |**Método de enmascaramiento que expone los últimos cuatro dígitos de los campos designados** y agrega una cadena constante como prefijo en el formato de una tarjeta de crédito.<br/><br/>XXXX-XXXX-XXXX-1234 |
| **Correo electrónico** |**Método de enmascaramiento que expone la primera letra y reemplaza el dominio con XXX.com** con una cadena constante como prefijo en el formato de una dirección de correo electrónico.<br/><br/>aXX@XXXX.com |
| **Número aleatorio** |**Método de enmascaramiento que genera un número aleatorio** según los límites seleccionados y los tipos de datos reales. Si los límites designados son iguales, la función de enmascaramiento será un número constante.<br/><br/>![Panel de navegación](./media/dynamic-data-masking-overview/1_DDM_Random_number.png) |
| **Texto personalizado** |**Método de enmascaramiento que expone el primero y el último carácter** y agrega una cadena de relleno personalizada en el medio. Si la cadena original es más corta que el prefijo y el sufijo expuestos, se usará únicamente la cadena de relleno. <br/>prefijo[relleno]sufijo<br/><br/>![Panel de navegación](./media/dynamic-data-masking-overview/2_DDM_Custom_text.png) |

<a name="Anchor1"></a>

### <a name="recommended-fields-to-mask"></a>Campos recomendados para enmascarar

El motor de recomendaciones de DDM marca determinados campos de la base de datos como campos potencialmente confidenciales, que pueden ser buenos candidatos para el enmascaramiento. En la hoja Enmascaramiento de datos dinámicos del portal, verá las columnas recomendadas para la base de datos. Todo lo que debe hacer es hacer clic en **Agregar máscara** para una o más columnas y, después, en **Guardar** a fin de aplicar una máscara para estos campos.

## <a name="set-up-dynamic-data-masking-for-your-database-using-powershell-cmdlets"></a>Configuración del enmascaramiento de datos dinámicos para la base de datos mediante cmdlets de PowerShell

### <a name="data-masking-policies"></a>Directivas de enmascaramiento de datos

- [Get-AzSqlDatabaseDataMaskingPolicy](https://docs.microsoft.com/powershell/module/az.sql/Get-AzSqlDatabaseDataMaskingPolicy)
- [Set-AzSqlDatabaseDataMaskingPolicy](https://docs.microsoft.com/powershell/module/az.sql/Set-AzSqlDatabaseDataMaskingPolicy)

### <a name="data-masking-rules"></a>Reglas de enmascaramiento de datos

- [Get-AzSqlDatabaseDataMaskingRule](https://docs.microsoft.com/powershell/module/az.sql/Get-AzSqlDatabaseDataMaskingRule)
- [New-AzSqlDatabaseDataMaskingRule](https://docs.microsoft.com/powershell/module/az.sql/New-AzSqlDatabaseDataMaskingRule)
- [Remove-AzSqlDatabaseDataMaskingRule](https://docs.microsoft.com/powershell/module/az.sql/Remove-AzSqlDatabaseDataMaskingRule)
- [Set-AzSqlDatabaseDataMaskingRule](https://docs.microsoft.com/powershell/module/az.sql/Set-AzSqlDatabaseDataMaskingRule)

## <a name="set-up-dynamic-data-masking-for-your-database-using-the-rest-api"></a>Configuración del enmascaramiento de datos dinámicos para la base de datos mediante la API de REST

Puede usar las API de REST para administrar las reglas y directivas de enmascaramiento de datos mediante programación. Las API de REST publicadas admiten las siguientes operaciones:

### <a name="data-masking-policies"></a>Directivas de enmascaramiento de datos

- [Crear o actualizar](https://docs.microsoft.com/rest/api/sql/datamaskingpolicies/createorupdate): crea o actualiza una directiva de enmascaramiento de datos para la base de datos.
- [Obtener](https://docs.microsoft.com/rest/api/sql/datamaskingpolicies/get): obtiene una directiva de enmascaramiento de datos para la base de datos. 

### <a name="data-masking-rules"></a>Reglas de enmascaramiento de datos

- [Crear o actualizar](https://docs.microsoft.com/rest/api/sql/datamaskingrules/createorupdate): crea o actualiza una regla de enmascaramiento de datos para la base de datos.
- [Lista por base de datos](https://docs.microsoft.com/rest/api/sql/datamaskingrules/listbydatabase): obtiene una lista de reglas de enmascaramiento de datos para la base de datos.
