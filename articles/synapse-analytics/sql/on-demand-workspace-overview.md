---
title: SQL a petición (versión preliminar)
description: Más información sobre Synapse SQL a petición en Azure Synapse Analytics.
services: synapse analytics
author: filippopovic
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: fc306dbca3191f04a85f2c5cc88d41336c13e09c
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496396"
---
# <a name="sql-on-demand-preview-in-azure-synapse-analytics"></a>SQL a petición (versión preliminar) en Azure Synapse Analytics 

Todas las áreas de trabajo de Azure Synapse Analytics (versión preliminar) incluyen puntos de conexión de SQL a petición (versión preliminar) que puede usar para consultar datos en el lago.

SQL a petición es un servicio de consulta de los datos del lago de datos. Le permite acceder a los datos mediante las siguientes funcionalidades:
 
- Una sintaxis T-SQL conocida para consultar los datos activos sin necesidad de copiarlos ni cargarlos en un almacén especializado. 
- Conectividad integrada mediante la interfaz de T-SQL que ofrece una amplia gama de herramientas de inteligencia empresarial y consulta ad hoc, entre las que se incluyen los controladores más populares. 

SQL a petición es un sistema de procesamiento de datos distribuido, creado para datos y procesos a gran escala. SQL a petición le permite analizar macrodatos en segundos, según la carga de trabajo. Gracias a la tolerancia a errores integrada en la ejecución de consultas, el sistema proporciona una alta confiabilidad y un alto índice de éxito incluso para consultas de ejecución prolongada que implican grandes conjuntos de datos.

SQL a petición funciona sin servidor, por lo que no hay que instalar ninguna infraestructura ni mantener ningún clúster. Se proporciona un punto de conexión predeterminado para este servicio en cada área de trabajo de Azure Synapse, por lo que puede empezar a consultar datos tan pronto como se crea el área de trabajo. No se cobra por los recursos reservados, solo se cobra por los datos examinados por las consultas que se ejecutan, por lo que este modelo es un modelo de pago por uso auténtico.  

Si usa Apache Spark para Azure Synapse en la canalización de datos, o para la preparación, limpieza o enriquecimiento de datos, puede [consultar las tablas externas de Spark](develop-storage-files-spark-tables.md) que haya creado en el proceso, directamente desde SQL On-Demand. Use [Private Link](../security/how-to-connect-to-workspace-with-private-links.md) para llevar el punto de conexión de SQL a petición a la [red virtual del área de trabajo administrada](../security/synapse-workspace-managed-vnet.md).  

## <a name="sql-on-demand-benefits"></a>Ventajas de SQL a petición

Si necesita explorar los datos de un lago de datos, obtener información de este u optimizar la canalización de transformación de datos existente, puede beneficiarse del uso de SQL a petición. Es adecuado para los escenarios siguientes:

- Detección y exploración básicos: analice rápidamente datos de diversos formatos (Parquet, CSV y JSON) en el lago de datos, para que pueda planear cómo extraer información de estos.
- Almacenamiento de datos lógico: proporcione una abstracción relacional sobre datos sin procesar o dispares sin tener que reubicar y transformar los datos, lo que permite una vista siempre actualizada de estos.
- Transformación de datos: una forma sencilla, escalable y eficaz de transformar los datos del lago mediante T-SQL, para que estos puedan alimentar a las herramientas de inteligencia empresarial o de cualquier otro tipo, o se carguen en un almacén de datos relacional (bases de datos de Synapse SQL, Azure SQL Database, etc.).

Diferentes puestos profesionales se pueden beneficiar de SQL a petición:

- Los ingenieros de datos pueden explorar el lago, transformar y preparar los datos mediante este servicio, y simplificar las canalizaciones de transformación de datos. Para más información, consulte este [tutorial](tutorial-data-analyst.md).
- Los científicos de datos pueden analizar rápidamente el contenido y la estructura de los datos del lago gracias a características como OPENROWSET y la inferencia de esquemas automática.
- Los analistas de datos pueden [explorar los datos y tablas externas de Spark](develop-storage-files-spark-tables.md) que crean los científicos o ingenieros de datos mediante el conocido lenguaje T-SQL o con sus herramientas favoritas, las cuales pueden conectarse a SQL On-Demand.
- Los profesionales de inteligencia empresarial pueden [crear rápidamente informes de Power BI a partir de los datos del lago](tutorial-connect-power-bi-desktop.md) y las tablas de Spark.

## <a name="how-to-start-using-sql-on-demand"></a>Primeros pasos con SQL a petición

El punto de conexión de SQL a petición se proporciona en cada área de trabajo de Azure Synapse. Puede crear un área de trabajo e iniciar la consulta de datos al instante mediante herramientas con las que está familiarizado.

## <a name="client-tools"></a>Herramientas de cliente

SQL a petición permite que las herramientas ya existentes de inteligencia empresarial y de consultas ad-hoc de SQL accedan al lago de datos. Como proporciona una sintaxis T-SQL familiar, cualquier herramienta capaz de establecer ofertas de SQL de conexión TDS puede [conectarse y realizar consultas a Synapse SQL a petición](connect-overview.md). Puede conectarse con Azure Data Studio y ejecutar consultas ad-hoc o conectarse con Power BI para obtener información en cuestión de minutos.

## <a name="t-sql-support"></a>Compatibilidad con T-SQL

SQL a petición ofrece un área expuesta de consultas de T-SQL, que se ha mejorado o ampliado ligeramente en algunos aspectos para dar cabida a experiencias relacionadas con la consulta de datos semiestructurados y datos no estructurados. Además, algunos aspectos del lenguaje T-SQL no son compatibles debido al diseño de SQL a petición como, por ejemplo, la funcionalidad DML en la actualidad.

- La carga de trabajo se puede organizar mediante conceptos conocidos:
- Bases de datos: el punto de conexión de SQL a petición puede tener varias bases de datos.
- Esquemas: en una base de datos, puede haber uno o varios grupos de propiedad de objetos denominados esquemas.
- Vistas
- Recursos externos: orígenes de datos, formatos de archivo y tablas

La seguridad se puede reforzar mediante:

- Inicios de sesión y usuarios
- Credenciales para controlar el acceso a las cuentas de almacenamiento
- Concesión, denegación y revocación de permisos por nivel de objeto
- Integración de Azure Active Directory

T-SQL compatible:

- Se admite el área expuesta [SELECT](/sql/t-sql/queries/select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) completa, que incluye la mayoría de las funciones de SQL.
- CETAS - CREATE EXTERNAL TABLE AS SELECT
- Instrucciones de DDL relacionadas solo con vistas y seguridad

SQL a petición no tiene almacenamiento local, solo los objetos de metadatos se almacenan en las bases de datos. Por lo tanto, no se admite T-SQL relacionado con los siguientes conceptos:

- Tablas
- Desencadenadores
- Vistas materializadas
- Instrucciones DDL distintas de las relacionadas con las vistas y la seguridad
- Instrucciones DML

### <a name="extensions"></a>Extensiones

Con el fin de permitir una experiencia fluida para la consulta en contexto de los datos que residen en el lago de datos, SQL a petición amplía la función [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) existente con la incorporación de las siguientes funcionalidades:

[Consulta de varios archivos o carpetas](query-data-storage.md#query-multiple-files-or-folders)

[Formato de archivo PARQUET](query-data-storage.md#query-parquet-files)

[Otras opciones para trabajar con texto delimitado (terminador de campo, terminador de fila, carácter de escape)](query-data-storage.md#query-csv-files)

[Lectura de un subconjunto de columnas elegido](query-data-storage.md#read-a-chosen-subset-of-columns)

[Inferencia de esquemas](query-data-storage.md#schema-inference)

[Función filename](query-data-storage.md#filename-function)

[Función filepath](query-data-storage.md#filepath-function)

[Uso con tipos complejos y estructuras de datos anidadas o repetidas](query-data-storage.md#work-with-complex-types-and-nested-or-repeated-data-structures)

## <a name="security"></a>Seguridad

SQL a petición ofrece mecanismos para proteger el acceso a los datos.

### <a name="azure-active-directory-integration-and-multi-factor-authentication"></a>Integración de Azure Active Directory y autenticación multifactor

SQL a petición permite administrar centralmente las identidades de usuario de base de datos y otros servicios de Microsoft con la [integración de Azure Active Directory](../../azure-sql/database/authentication-aad-configure.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json). Esta funcionalidad simplifica la administración de permisos y mejora la seguridad. Azure Active Directory (Azure AD) admite la [autenticación multifactor](../../azure-sql/database/authentication-mfa-ssms-configure.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) (MFA) para aumentar la seguridad tanto de los datos como de las aplicaciones y admite un proceso de inicio de sesión único.

#### <a name="authentication"></a>Authentication

La autenticación de SQL a petición indica cómo los usuarios prueban su identidad al conectarse al punto de conexión. Se admiten dos tipos de autenticación:

- **Autenticación de SQL**

  Este método de autenticación utiliza un nombre de usuario y una contraseña.

- **Autenticación con Azure Active Directory**:

  Este método de autenticación usa identidades administradas por Azure Active Directory. Se puede habilitar la autenticación multifactor para los usuarios de Azure AD. Use la autenticación de Active Directory (seguridad integrada) [siempre que sea posible](/sql/relational-databases/security/choose-an-authentication-mode?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).

#### <a name="authorization"></a>Authorization

La autorización indica las acciones que pueden llevar a cabo los usuarios en una base de datos de SQL a petición, algo que controlan los permisos de nivel de objeto y las pertenencias a roles de bases de datos de la cuenta de usuario.

Si se usa la autenticación de SQL, el usuario de SQL solo existe en SQL a petición y los permisos se limitan a los objetos de SQL a petición. El acceso a los objetos protegibles de otros servicios (como Azure Storage) no se puede conceder al usuario de SQL directamente, ya que solo existe en el ámbito de SQL a petición. El usuario de SQL debe usar uno de los [tipos de autorización admitidos](develop-storage-files-storage-access-control.md#supported-storage-authorization-types) para acceder a los archivos.

Si se usa la autenticación de Azure AD, un usuario puede iniciar sesión en SQL a petición y en otros servicios, como Azure Storage, y puede conceder permisos al usuario de Azure AD.

### <a name="access-to-storage-accounts"></a>Acceso a las cuentas de almacenamiento

Un usuario que haya iniciado sesión en un servicio de SQL a petición debe estar autorizado para acceder a los archivos de Azure Storage y realizar consultas en ellos. SQL a petición admite los siguientes métodos de autorización:

- La **firma de acceso compartido (SAS)** ofrece acceso delegado a los recursos de una cuenta de almacenamiento. Con una SAS, puede conceder a los clientes acceso a los recursos de su cuenta de almacenamiento sin compartir las claves de la cuenta. Una SAS le ofrece un control detallado sobre el tipo de acceso que concede a los clientes que tienen una SAS: intervalo de validez, permisos concedidos, intervalo de direcciones IP aceptable y protocolo aceptable (https/http).

- La **identidad de usuario**, también conocida como "paso a través", es un tipo de autorización en el que la identidad del usuario de Azure AD que inició sesión en SQL a petición se usa para autorizar el acceso a los datos. Antes de acceder a los datos, el administrador de Azure Storage debe conceder permisos al usuario de Azure AD para acceder a los datos. Este tipo de autorización utiliza el usuario de Azure AD que inició sesión en SQL a petición, por lo que no se admite para los tipos de usuario de SQL.

## <a name="next-steps"></a>Pasos siguientes
Puede encontrar información adicional sobre la conexión de punto de conexión y los archivos de consulta en los siguientes artículos: 
- [Conexión al punto de conexión](connect-overview.md)
- [Consulta de los archivos](develop-storage-files-overview.md)
