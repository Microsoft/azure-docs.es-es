---
title: Configuración de la replicación en Azure SQL Edge
description: Obtenga información sobre la configuración de la replicación en Azure SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: a748bf977e76357c710518e608c12ad19a8cd0be
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90888414"
---
# <a name="configure-replication-to-azure-sql-edge"></a>Configuración de la replicación en Azure SQL Edge 

Una instancia de Azure SQL Edge puede configurarse como el suscriptor de inserción para una replicación transaccional unidireccional o una replicación de instantáneas. Esta instancia no puede actuar como el publicador o el distribuidor para una configuración de replicación transaccional. Tenga en cuenta que Azure SQL Edge no admite la replicación de mezcla, la replicación punto a punto ni la publicación de Oracle.

## <a name="supported-configurations"></a>Configuraciones admitidas
  
- La instancia de Azure SQL Edge debe ser un suscriptor de inserción para un publicador.
- El publicador y el distribuidor pueden ser:
   - Una instancia de SQL Server en ejecución en el entorno local o una instancia de SQL Server en ejecución en una máquina virtual de Azure. Para obtener más información, consulte [Introducción a SQL Server en máquinas virtuales de Azure](https://docs.microsoft.com/azure/azure-sql/virtual-machines/). Las instancias de SQL Server deben usar una versión posterior a SQL Server 2016.
   - Una instancia de Instancia administrada de Azure SQL. SQL Managed Instance puede hospedar bases de datos del publicador, distribuidor y suscriptor. Para más información, consulte [Replication with SQL Database Managed Instance](https://docs.microsoft.com/azure/sql-database/replication-with-sql-database-managed-instance/) (Replicación con Instancia administrada de Azure SQL Database).

- La base de datos de distribución y los agentes de replicación no pueden colocarse en una instancia de Azure SQL Edge.  

> [!NOTE]
> Si intenta configurar la replicación con una versión incompatible, puede recibir los dos errores siguientes: MSSQL_REPL20084 ("(El proceso no pudo conectarse al suscriptor") y MSSQL_REPL40532 ("No se puede abrir el servidor \<name> solicitado por el inicio de sesión. Error de inicio de sesión").  

## <a name="remarks"></a>Observaciones

Los siguientes requisitos y procedimientos recomendados son importantes para comprender la configuración de la replicación:

- Puede configurar la replicación mediante [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms). También puede hacerlo mediante la ejecución de instrucciones Transact-SQL en el publicador, con SQL Server Management Studio o [Azure Data Studio](https://docs.microsoft.com/sql/azure-data-studio/download-azure-data-studio).
- Para replicar en una instancia de Azure SQL Edge, debe usar la autenticación de SQL Server para iniciar sesión.
- Las tablas replicadas deben tener una clave principal.
- Una sola publicación en SQL Server puede admitir los suscriptores de SQL Server (entorno local y SQL Server en una máquina virtual de Azure) y Azure SQL Edge.  
- La administración, la supervisión y la solución de problemas de la replicación deben realizarse desde la instancia de SQL Server.  
- Azure SQL Edge solo admite las suscripciones de inserción.  
- Solo se admite `@subscriber_type = 0` en el procedimiento almacenado `sp_addsubscription` para Azure SQL Edge.  
- Azure SQL Edge no admite replicaciones bidireccionales, inmediatas, actualizables ni de punto a punto.
- Azure SQL Edge solo admite un subconjunto de características disponibles en SQL Server o SQL Managed Instance. Si intenta replicar una base de datos (o los objetos de la base de datos) que contenga una o varias características no admitidas, se producirá un error. Por ejemplo, si intenta replicar una base de datos que contenga objetos con tipos de datos espaciales, se producirá un error. Para más información, vea [Características admitidas de Azure SQL Edge](features.md).

## <a name="initialize-reference-data-on-an-instance-of-azure-sql-edge"></a>Inicialización de los datos de referencia en una instancia de Azure SQL Edge

Es posible que quiera inicializar la instancia con datos de referencia que cambian con el tiempo. Por ejemplo, puede que desee actualizar modelos de Machine Learning en la instancia de Azure SQL Edge una vez que se hayan entrenado en una instancia de SQL Server. Aquí se muestra cómo inicializar la instancia en este escenario:

1. Cree una publicación de replicación transaccional en una base de datos de SQL Server.  
2. En la instancia de SQL Server, use el **Asistente para nueva suscripción** o las instrucciones Transact-SQL para crear una inserción en la suscripción de Azure SQL Edge.  
3. Puede inicializar la base de datos replicada en Azure SQL Edge con una instantánea generada por el agente de instantáneas y distribuida y entregada por el agente de distribución. También puede hacerlo mediante una copia de seguridad de la base de datos desde el publicador. Recuerde que si la copia de seguridad de la base de datos contiene objetos o características que no sean compatibles con Azure SQL Edge, se producirá un error en la operación de restauración.

## <a name="limitations"></a>Limitaciones

Las siguientes opciones no se admiten para las suscripciones de Azure SQL Edge:

- Copiar asociaciones de grupos de archivos  
- Copiar esquemas de partición de tabla  
- Copiar esquemas de partición de índice  
- Copiar estadísticas definidas por el usuario  
- Copiar enlaces predeterminados  
- Copiar enlaces de reglas  
- Copiar índices de texto completo  
- Copiar XML XSD  
- Copiar índices XML  
- Copiar permisos  
- Copiar índices espaciales  
- Copiar índices filtrados  
- Copiar atributos de compresión de datos  
- Copiar atributos de columna dispersa  
- Copiar tipos de datos espaciales, FileStream o `hierarchyid`
- Convertir `hierarchyid` en tipos de datos MAX  
- Convertir una instancia espacial en tipos de datos MAX  
- Copiar propiedades extendidas  
- Copiar permisos  

## <a name="examples"></a>Ejemplos

Cree una publicación y una suscripción de inserción. Para más información, consulte:
  
- [Creación de una publicación](https://docs.microsoft.com/sql/relational-databases/replication/publish/create-a-publication)
- [Cree una suscripción de inserción](https://docs.microsoft.com/sql/relational-databases/replication/create-a-push-subscription/) mediante el nombre del servidor y la dirección IP de Azure SQL Edge como suscriptor (por ejemplo, **myEdgeinstance,1433**) y el nombre de la base de datos en la instancia de Azure SQL Edge como base de datos de destino (por ejemplo, **AdventureWorks**).  

## <a name="next-steps"></a>Pasos siguientes  

- [Creación de una publicación](https://docs.microsoft.com/sql/relational-databases/replication/publish/create-a-publication)
- [Crear una suscripción de inserción](https://docs.microsoft.com/sql/relational-databases/replication/create-a-push-subscription/)
- [Tipos de replicación](https://docs.microsoft.com/sql/relational-databases/replication/types-of-replication)
- [Supervisión (replicación)](https://docs.microsoft.com/sql/relational-databases/replication/monitor/monitoring-replication)
- [Inicialización de una suscripción](https://docs.microsoft.com/sql/relational-databases/replication/initialize-a-subscription)  


