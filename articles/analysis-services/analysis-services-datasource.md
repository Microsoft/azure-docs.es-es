---
title: Orígenes de datos admitidos en Azure Analysis Services | Microsoft Docs
description: Describe los orígenes de datos y los conectores admitidos para los modelos de datos tabulares de 1200 y superiores en Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/31/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 72a1a37bf240355e6bc87cbfd62b0dc2d25ce68b
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503606"
---
# <a name="data-sources-supported-in-azure-analysis-services"></a>Orígenes de datos admitidos en Azure Analysis Services

Los orígenes de datos y los conectores que se muestran en Obtener datos o en el Asistente para la importación de tablas de Visual Studio con los proyectos de Analysis Services corresponden tanto a Azure Analysis Services como a SQL Server Analysis Services. Sin embargo, no todos los orígenes de datos y los conectores que aparecen son compatibles con Azure Analysis Services. Los tipos de orígenes de datos con los que se puede establecer conexión dependen de muchos factores, como el nivel de compatibilidad del modelo, los conectores de datos disponibles, el tipo de autenticación y la compatibilidad con la puerta de enlace de datos local. En las tablas siguientes se describen los orígenes de datos que admite Azure Analysis Services.

## <a name="azure-data-sources"></a>Orígenes de datos de Azure

|Origen de datos  |En memoria  |DirectQuery  |Notas |
|---------|---------|---------|---------|
|Azure SQL Database      |   Sí      |    Sí      |<sup>[2](#azprovider)</sup>, <sup>[3](#azsqlmanaged)</sup>|
|Azure Synapse Analytics (SQL DW)      |   Sí      |   Sí       |<sup>[2](#azprovider)</sup>|
|Azure Blob Storage      |   Sí       |    No      | <sup>[1](#tab1400a)</sup> |
|Azure Table Storage     |   Sí       |    No      | <sup>[1](#tab1400a)</sup>|
|Azure Cosmos DB     |  Sí        |  No        |<sup>[1](#tab1400a)</sup> |
|Azure Data Lake Store Gen1      |   Sí       |    No      |<sup>[1](#tab1400a)</sup> |
|Azure Data Lake Store Gen2       |   Sí       |    No      |<sup>[1](#tab1400a)</sup>, <sup>[5](#gen2)</sup>|
|Azure HDInsight HDFS    |     Sí     |   No       |<sup>[1](#tab1400a)</sup> |
|Azure HDInsight Spark     |   Sí       |   No       |<sup>[1](#tab1400a)</sup>, <sup>[4](#databricks)</sup>|
||||

**Notas:**

<a name="tab1400a">1</a>: Tabular 1400 y modelos posteriores solamente.  
<a name="azprovider">2</a>: Cuando se especifica como un origen de datos de *proveedor* en los modelos tabulares 1200 y posteriores, los modelos en memoria y de DirectQuery requieren Microsoft OLE DB Driver for SQL Server MSOLEDBSQL (recomendado), SQL Server Native Client 11.0 o Proveedor de datos .NET Framework para SQL Server.  
<a name="azsqlmanaged">3</a>: Compatible con Instancia administrada de Azure SQL. Dado que SQL Managed Instance se ejecuta dentro de una red virtual de Azure con una dirección IP privada, el punto de conexión público debe estar habilitado en la instancia. Si no está habilitado, se requiere una [puerta de enlace de datos local](analysis-services-gateway.md).  
<a name="databricks">4</a>: Actualmente no se admite Azure Databricks con el conector de Spark.  
<a name="gen2">5</a>: el conector ADLS Gen2 no se admite actualmente; no obstante, se puede usar el conector de Azure Blob Storage con un origen de datos ADLS Gen2.

## <a name="other-data-sources"></a>Otros orígenes de datos

|Origen de datos | En memoria | DirectQuery |Notas   |
|  --- | --- | --- | --- |
|Base de datos de Access     |  Sí | No |  |
|Active Directory     |  Sí | No | <sup>[6](#tab1400b)</sup>  |
|Analysis Services     |  Sí | No |  |
|Sistema de la plataforma de análisis     |  Sí | No |  |
|Archivo CSV  |Sí | No |  |
|Dynamics 365     |  Sí | No | <sup>[6](#tab1400b)</sup> |
|Libro de Excel     |  Sí | No |  |
|Exchange      |  Sí | No | <sup>[6](#tab1400b)</sup> |
|Carpeta      |Sí | No | <sup>[6](#tab1400b)</sup> |
|IBM Informix  |Sí | No |  |
|Documento JSON      |  Sí | No | <sup>[6](#tab1400b)</sup> |
|Líneas de archivo binario      | Sí | No | <sup>[6](#tab1400b)</sup> |
|Base de datos de MySQL     | Sí | No |  |
|Fuente OData      |  Sí | No | <sup>[6](#tab1400b)</sup> |
|Consulta ODBC     | Sí | No |  |
|OLE DB     |   Sí | No |  |
|Oracle  | Sí  |Sí  | <sup>[9](#oracle)</sup> |
|Base de datos de PostgreSQL   | Sí | No | <sup>[6](#tab1400b)</sup> |
|Objetos de Salesforce|  Sí | No | <sup>[6](#tab1400b)</sup> |
|Informes de Salesforce |Sí | No | <sup>[6](#tab1400b)</sup> |
|SAP HANA     |  Sí | No |  |
|SAP Business Warehouse    |  Sí | No | <sup>[6](#tab1400b)</sup> |
|Lista de SharePoint      |   Sí | No | <sup>[6](#tab1400b)</sup>, <sup>[11](#filesSP)</sup> |
|SQL Server |Sí   | Sí  | <sup>[7](#sqlim)</sup>, <sup>[8](#instgw)</sup> |
|SQL Server Data Warehouse |Sí   | Sí  | <sup>[7](#sqlim)</sup>, <sup>[8](#instgw)</sup> |
|Base de datos de Sybase     |  Sí | No |  |
|Teradata | Sí  | Sí  | <sup>[10](#teradata)</sup> |
|Archivo TXT  |Sí | No |  |
|Tabla XML    |  Sí | No | <sup>[6](#tab1400b)</sup> |
| | | |

**Notas:**  
<a name="tab1400b">6</a>: Tabular 1400 y modelos posteriores solamente.  
<a name="sqlim">7</a>: Cuando se especifica como un origen de datos de *proveedor* en los modelos tabulares 1200 y posteriores, especifique Microsoft OLE DB Driver for SQL Server MSOLEDBSQL (recomendado), SQL Server Native Client 11.0 o Proveedor de datos .NET Framework para SQL Server.  
<a name="instgw">8</a>: Si especifica MSOLEDBSQL como proveedor de datos, puede que sea necesario descargar e instalar [Microsoft OLE DB Driver for SQL Server](https://docs.microsoft.com/sql/connect/oledb/oledb-driver-for-sql-server) en el mismo equipo que la puerta de enlace de datos local.  
<a name="oracle">9</a>: Para los modelos tabulares 1200, o como origen de datos de *proveedor* en los modelos tabulares 1400 o posteriores, especifique el proveedor de datos de Oracle para .NET. Si se especifica como un origen de datos estructurado, no olvide [habilitar el proveedor administrado de Oracle](#enable-oracle-managed-provider).   
<a name="teradata">10</a>: Para los modelos tabulares 1200, o como origen de datos de *proveedor* en los modelos tabulares 1400 o posteriores, especifique el proveedor de datos de Teradata para .NET.  
<a name="filesSP">11</a>: No se admiten archivos en SharePoint en el entorno local.

Para conectarse a los orígenes de datos locales desde un servidor de Azure Analysis Services, se necesita una [puerta de enlace local](analysis-services-gateway.md). Cuando se usa una puerta de enlace, se requieren proveedores de 64 bits.

## <a name="understanding-providers"></a>Descripción de los proveedores

Cuando crea proyectos de modelos tabulares 1400 y superiores en Visual Studio, no se especifica un proveedor de datos de forma predeterminada al conectarse a un origen de datos mediante **Get Data**. Los modelos tabulares 1400 y posteriores usan conectores de [Power Query](/power-query/power-query-what-is-power-query) para administrar conexiones, consultas de datos y mashups entre el origen de datos y Analysis Services. A veces se hace referencia a ellos como conexiones de origen de datos *estructuradas* en los que la configuración de las propiedades de conexión se establece automáticamente. Sin embargo, puede habilitar orígenes de datos heredados para un proyecto de modelo en Visual Studio. Cuando está habilitado, puede usar el **Asistente para la importación de tablas** para conectarse a ciertos orígenes de datos admitidos tradicionalmente en modelos tabulares 1200 y modelos inferiores como orígenes de datos *heredados* o del *proveedor*. Cuando se especifica como un origen de datos del proveedor, puede especificar un proveedor de datos determinado y otras propiedades de conexión avanzadas. Por ejemplo, puede conectarse a una instancia de Almacenamiento de datos de SQL Server o incluso a una instancia de Azure SQL Database como un origen de datos heredado. Después, puede seleccionar el proveedor de datos MSOLEDBSQL de OLE DB Driver for SQL Server. En este caso, la selección de un proveedor de datos de OLE DB puede proporcionar un rendimiento mejorado en el conector de Power Query. 

Cuando se usa el Asistente para la importación de tablas en Visual Studio, las conexiones a cualquier origen de datos requieren un proveedor de datos. Se selecciona un proveedor de datos predeterminado. Puede cambiarlo si es necesario. El tipo de proveedor que elija puede depender del rendimiento, de si el modelo usa el almacenamiento en memoria o DirectQuery, y de en qué plataforma de Analysis Services implementa el modelo.

### <a name="specify-provider-data-sources-in-tabular-1400-and-higher-model-projects"></a>Especificar orígenes de datos del proveedor en proyectos de modelos tabulares 1400 y posteriores

Para habilitar orígenes de datos del proveedor, en Visual Studio, haga clic en **Herramientas** > **Opciones** > **Tabular de Analysis Services** > **Importación de datos** y seleccione **Habilitar orígenes de datos heredados**.

![Habilitar orígenes de datos heredados](media/analysis-services-datasource/aas-enable-legacy-datasources.png)

Con los orígenes de datos heredados habilitados, en **Explorador de modelos tabulares**, haga clic con el botón derecho en **Orígenes de datos** > **Importar desde origen de datos (heredado)** .

![Orígenes de datos heredados en el Explorador de modelos tabulares](media/analysis-services-datasource/aas-import-legacy-datasources.png)

Al igual que con los proyectos de modelos tabulares 1200, use el **Asistente para la importación de tablas** para conectarse a un origen de datos. En la página de conexión, haga clic en **Avanzado**. Especifique el proveedor de datos y los demás valores de conexión en **Establecer propiedades avanzadas**.

![Propiedades avanzadas de orígenes de datos heredados](media/analysis-services-datasource/aas-import-legacy-advanced.png)

## <a name="impersonation"></a>Suplantación
En algunos casos, puede ser necesario especificar otra cuenta de suplantación. La cuenta de suplantación se puede especificar en Visual Studio o SQL Server Management Studio (SSMS).

Para orígenes de datos locales:

* Si se utiliza la autenticación de SQL, la suplantación debe ser la Cuenta de servicio.
* Si se utiliza la autenticación de Windows, establezca la contraseña y el usuario de Windows. Para SQL Server, se admite la autenticación de Windows con una cuenta de suplantación específica solo para los modelos de datos en memoria.

Para orígenes de datos en la nube:

* Si se utiliza la autenticación de SQL, la suplantación debe ser la Cuenta de servicio.

## <a name="oauth-credentials"></a>Credenciales de OAuth

En el caso de los modelos tabulares con el nivel de compatibilidad 1400 y superior que emplean el modo en memoria, Azure SQL Database, Azure Synapse (anteriormente, SQL Data Warehouse), Dynamics 365 y la lista de SharePoint admiten las credenciales de OAuth. Azure Analysis Services administra la actualización de tokens para los orígenes de datos de OAuth para evitar los tiempos de espera con las operaciones de actualización de ejecución prolongadas. Para generar tokens válidos, establezca las credenciales mediante SSMS.

El modo de consulta directa no es compatible con las credenciales de OAuth.

## <a name="enable-oracle-managed-provider"></a>Habilitación del proveedor administrado de Oracle

En algunos casos, las consultas DAX que se realizan en un origen de datos de Oracle pueden devolver resultados inesperados. Esto puede deberse al proveedor que se utiliza para establecer la conexión del origen de datos.

Tal y como se indica en la sección [Descripción de los proveedores](#understanding-providers), los modelos tabulares se conectan a orígenes de datos como un origen de datos *estructurado* o como un origen de datos de *proveedor*. En el caso de los modelos con un origen de datos de Oracle especificado como un origen de datos de proveedor, asegúrese de que el proveedor especificado es Oracle Data Provider for .NET (Oracle.DataAccess.Client). 

Si el origen de datos de Oracle se especifica como un origen de datos estructurado, habilite la propiedad **MDataEngine\UseManagedOracleProvider** del servidor. Al configurar esta propiedad, se asegura de que el modelo se conecta al origen de datos de Oracle utilizando el proveedor administrado de Oracle Data Provider for .NET recomendado.
 
Para habilitar el proveedor administrado de Oracle:

1. En SQL Server Management Studio, conéctese al servidor.
2. Cree una consulta XMLA con el siguiente script. Sustituya **ServerName** por el nombre completo del servidor y ejecute la consulta.

    ```xml
    <Alter AllowCreate="true" ObjectExpansion="ObjectProperties" xmlns="http://schemas.microsoft.com/analysisservices/2003/engine">
        <Object />
        <ObjectDefinition>
            <Server xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:ddl2="http://schemas.microsoft.com/analysisservices/2003/engine/2" xmlns:ddl2_2="http://schemas.microsoft.com/analysisservices/2003/engine/2/2" 
    xmlns:ddl100_100="http://schemas.microsoft.com/analysisservices/2008/engine/100/100" xmlns:ddl200="http://schemas.microsoft.com/analysisservices/2010/engine/200" xmlns:ddl200_200="http://schemas.microsoft.com/analysisservices/2010/engine/200/200" 
    xmlns:ddl300="http://schemas.microsoft.com/analysisservices/2011/engine/300" xmlns:ddl300_300="http://schemas.microsoft.com/analysisservices/2011/engine/300/300" xmlns:ddl400="http://schemas.microsoft.com/analysisservices/2012/engine/400" 
    xmlns:ddl400_400="http://schemas.microsoft.com/analysisservices/2012/engine/400/400" xmlns:ddl500="http://schemas.microsoft.com/analysisservices/2013/engine/500" xmlns:ddl500_500="http://schemas.microsoft.com/analysisservices/2013/engine/500/500">
                <ID>ServerName</ID>
                <Name>ServerName</Name>
                <ServerProperties>
                    <ServerProperty>
                        <Name>MDataEngine\UseManagedOracleProvider</Name>
                        <Value>1</Value>
                    </ServerProperty>
                </ServerProperties>
            </Server>
        </ObjectDefinition>
    </Alter>
    ```

3. Reinicie el servidor.


## <a name="next-steps"></a>Pasos siguientes

* [Puerta de enlace local](analysis-services-gateway.md)
* [Administración del servidor](analysis-services-manage.md)
