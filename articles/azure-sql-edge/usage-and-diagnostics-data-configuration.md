---
title: Configuración de datos de diagnóstico y uso de Azure SQL Edge
description: Aprenda a configurar datos de uso y diagnóstico en Azure SQL Edge.
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 08/04/2020
ms.openlocfilehash: 130e23c290ce493d3fb92f6dd0be4cd7c61a86fd
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90888052"
---
# <a name="azure-sql-edge-usage-and-diagnostics-data-configuration"></a>Configuración de datos de diagnóstico y uso de Azure SQL Edge

De manera predeterminada, Azure SQL Edge recopila información sobre cómo sus clientes usan la aplicación. En concreto, Azure SQL Edge recopila información sobre la experiencia de implementación, el uso y el rendimiento. Esta información ayuda a Microsoft a mejorar el producto para satisfacer mejor las necesidades del cliente. Por ejemplo, Microsoft recopila información sobre los tipos de códigos de error que encuentran los clientes para que podamos corregir errores relacionados, mejorar nuestra documentación sobre cómo usar Azure SQL Edge y determinar si deben agregarse características al producto para ofrecer un mejor servicio a los clientes.

En concreto, Microsoft no envía ninguno de los tipos de información siguientes a través de este mecanismo:

- Valores de dentro de las tablas de usuario.
- Credenciales de inicio de sesión u otra información de autenticación.
- Datos personales o de clientes.

En el escenario de ejemplo siguiente se incluye información de uso de características que ayuda a mejorar el producto.

A continuación, se proporciona una consulta de ejemplo de las consultas que se usan para la recopilación de datos de uso y diagnóstico. La consulta identifica el recuento y los tipos de distintos orígenes de datos de streaming que se usan en Azure SQL Edge. Estos datos ayudan a Microsoft a identificar qué orígenes de datos de streaming se usan con frecuencia, por lo que Microsoft puede mejorar el rendimiento y la experiencia del usuario asociados con estos orígenes de datos. 

```sql
select 
count(*) as [count], sum(inputs) as inputs, sum(outputs) as outputs, sum(linked_to_job) 
as linked_to_job, data_source_type  
from ( 
select isnull(value,'unknown') as data_source_type, inputs, outputs, linked_to_job  
from 
    ( 
        select 
        convert(sysname, lower(substring(ds.location, 0, charindex('://', ds.location))), 1) as data_source_type, 
        isnull(inputs, 0) as inputs, isnull(outputs, 0) as outputs, isnull(js.stream_id/js.stream_id, 0) as linked_to_job 
        from sys.external_streams es              
        join sys.external_data_sources ds 
             on es.data_source_id = ds.data_source_id             
        left join 
            ( 
            select stream_id, max(cast(is_input as int)) inputs, max(cast(is_output as int)) outputs 
            from sys.external_job_streams group by stream_id 
            ) js                
             on js.stream_id = es.object_id 
    ) ds            
left join 
    (
        select value from string_split('edgehub,sqlserver,kafka', ',')) as known_ep on data_source_type = value 
    ) known_ds        
group by data_source_type
```

## <a name="disable-usage-and-diagnostic-data-collection"></a>Deshabilitación de la recopilación de datos de uso y diagnóstico

La recopilación de datos de uso y diagnóstico en Azure SQL Edge se puede deshabilitar mediante cualquiera de los métodos siguientes.

> [!NOTE]
> Los datos de utilización y diagnóstico no se pueden deshabilitar para la versión del Desarrollador.

### <a name="disable-usage-and-diagnostics-using-environment-variables"></a>Deshabilitación del uso y los diagnósticos mediante variables de entorno

Para deshabilitar la recopilación de datos de uso y diagnóstico en Azure SQL Edge, agregue la siguiente variable de entorno y establezca su valor en `*False*`. Para más información sobre la configuración de Azure SQL Edge mediante variables de entorno, consulte [Configuración con variables de entorno](configure.md#configure-by-using-environment-variables).

`MSSQL_TELEMETRY_ENABLED = TRUE | FALSE`

- TRUE: habilita la recopilación de datos de uso y diagnóstico. Esta es la configuración predeterminada.
- FALSE: deshabilita la recopilación de datos de utilización y diagnóstico.

### <a name="disable-usage-and-diagnostics-using-mssqlconf-file"></a>Deshabilitación del uso y los diagnósticos mediante un archivo mssql.conf

Para deshabilitar la recopilación de datos de utilización y diagnóstico en Azure SQL Edge, agregue las líneas siguiente en el archivo mssql.conf en la unidad de almacenamiento persistente que está asignada a la carpeta /var/opt/mssql/ en el módulo SQL Edge. Para más información sobre cómo configurar Azure SQL Edge mediante el archivo mssql.conf, consulte [Configuración mediante un archivo mssql.conf](configure.md#configure-by-using-an-mssqlconf-file).

```ini
[telemetry]
customerfeedback = false
```

## <a name="local-audit-of-usage-and-diagnostic-data-collection"></a>Auditoría local de la recopilación de datos de uso y diagnóstico

El componente Auditoría local de recopilación de datos de uso y diagnóstico de Azure SQL Edge puede escribir los datos recopilados por el servicio en una carpeta designada, lo que representa los datos (registros) que se van a enviar a Microsoft. El propósito de la Auditoría local es permitir que los clientes vean todos los datos que Microsoft recopila con esta característica, por motivos de cumplimiento, reglamentarios o por validación de privacidad.

### <a name="enable-local-audit-of-usage-and-diagnostics-data"></a>Habilitación de auditoría local de la recopilación de datos de uso y diagnóstico

Para habilitar los datos de uso y diagnóstico de Auditoría local en Azure SQL Edge:

1. Cree un directorio de destino para el nuevo almacenamiento de registros de Auditoría local. Este directorio de destino puede estar en el host o en el contenedor. En el ejemplo siguiente, el directorio de destino se crea en el mismo volumen de montaje que está asignado a la ruta de acceso /var/opt/mssql/ en SQL Edge.

   ```bash
   sudo mkdir <host mount path>/audit
   ```

2. Configure la auditoría de datos de uso y diagnóstico mediante variables de entorno o el archivo mssql.conf.

   - Mediante el uso de variables de entorno: agregue la siguiente variable de entorno a la implementación de SQL Edge y especifique el directorio de destino para los archivos de auditoría.
   
     `*MSSQL_TELEMETRY_DIR = <host mount path>/audit*`
   
   - Mediante el uso del archivo mssql.conf: agregue las líneas siguientes en el archivo mssql.conf y especifique el directorio de destino para los archivos de auditoría.
       ```ini
       [telemetry]
       userrequestedlocalauditdirectory  = <host mount path>/audit
       ```  

## <a name="next-steps"></a>Pasos siguientes

- [Conexión a Azure SQL Edge](connect.md)
- [Creación de una solución de IoT global con SQL Edge](tutorial-deploy-azure-resources.md)
