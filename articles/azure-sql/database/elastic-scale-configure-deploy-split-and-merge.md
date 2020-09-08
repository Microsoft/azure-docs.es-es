---
title: Implemente un servicio de división y combinación
description: Use la herramienta de división y combinación para mover los datos entre bases de datos con particiones.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 12/04/2018
ms.openlocfilehash: 1cd03814e1590abebb74db490a2692d492a9207d
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88064951"
---
# <a name="deploy-a-split-merge-service-to-move-data-between-sharded-databases"></a>Implementación de un servicio de división y combinación para mover datos entre bases de datos particionadas
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

La herramienta de división y combinación permite mover los datos entre bases de datos particionadas. Consulte [Mover datos entre bases de datos en la nube escaladas horizontalmente](elastic-scale-overview-split-and-merge.md)

## <a name="download-the-split-merge-packages"></a>Descarga de los paquetes de División y combinación

1. Descargue la última versión de NuGet desde [NuGet](https://docs.nuget.org/docs/start-here/installing-nuget).

1. Abra un símbolo del sistema y vaya al directorio al que descargó nuget.exe. La descarga incluye comandos de PowerShell.

1. Descargue el paquete de División y combinación más reciente en el directorio actual con el comando que aparece a continuación:

   ```cmd
   nuget install Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge
   ```  

Los archivos están ubicados en un directorio llamado **Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge.x.x.xxx.x** , donde *x.x.xxx.x* refleja el número de la versión. Busque los archivos del servicio de división y combinación en el subdirectorio **content\splitmerge\service** y los scripts de PowerShell de división y combinación (y las .dll de cliente necesarias) en el subdirectorio **content\splitmerge\powershell**.

## <a name="prerequisites"></a>Requisitos previos

1. Cree una base de datos de Azure SQL Database que se usará como la base de datos de estado de división y combinación. Vaya a [Azure Portal](https://portal.azure.com). Cree una nueva **SQL Database**. Asigne un nombre a la base de datos y cree un nuevo administrador y una contraseña. Asegúrese de anotar el nombre y la contraseña para usarlos más adelante.

1. Asegúrese de que el servidor permita que los servicios de Azure se conecten a él. En el portal, en **Configuración de firewall**, asegúrese de que la opción **Permitir el acceso a servicios de Azure** esté establecida en **Activado**. Haga clic en el icono de "guardar".

1. Cree una cuenta de Azure Storage para la salida de diagnóstico.

1. Cree un servicio en la nube de Azure para el servicio de división y combinación.

## <a name="configure-your-split-merge-service"></a>Configuración del servicio de división y combinación

### <a name="split-merge-service-configuration"></a>Configuración del servicio División y combinación

1. En la carpeta donde ha descargado los ensamblados de división y combinación, cree una copia del archivo *ServiceConfiguration.Template.cscfg* que se suministra junto a *SplitMergeService.cspkg* y cámbiele el nombre por *ServiceConfiguration.cscfg*.

1. Abra *ServiceConfiguration.cscfg* en un editor de texto como Visual Studio que permite validar las entradas como el formato de las huellas digitales de certificados.

1. Cree una nueva base de datos o elija una base de datos existente que sirva como la base de datos de estado para las operaciones de división y combinación y recupere la cadena de conexión de esa base de datos.

   > [!IMPORTANT]
   > En este momento, la base de datos de estado debe usar la intercalación latina (SQL\_Latin1\_General\_CP1\_CI\_AS). Para más información, vea [Nombre de intercalación de Windows (Transact-SQL)](https://msdn.microsoft.com/library/ms188046.aspx).

   Con Azure SQL Database, la cadena de conexión suele tener el formato:

      `Server=<serverName>.database.windows.net; Database=<databaseName>;User ID=<userId>; Password=<password>; Encrypt=True; Connection Timeout=30`

1. Escriba esta cadena de conexión en el archivo *.cscfg* en las secciones de roles **SplitMergeWeb** y **SplitMergeWorker** en la configuración de ElasticScaleMetadata.

1. Para el rol **SplitMergeWorker**, escriba una cadena de conexión válida en Azure Storage para la configuración **WorkerRoleSynchronizationStorageAccountConnectionString**.

### <a name="configure-security"></a>Configuración de seguridad

Para obtener instrucciones detalladas para configurar la seguridad del servicio, consulte [Configuración de seguridad de división y combinación](elastic-scale-split-merge-security-configuration.md).

Para fines de una simple implementación de prueba para este tutorial, se realizará un conjunto mínimo de pasos de configuración para configurar y ejecutar el servicio. Estos pasos solo habilitan la máquina/cuenta que los ejecuta para comunicarse con el servicio.

### <a name="create-a-self-signed-certificate"></a>Creación de un certificado autofirmado

Cree un directorio nuevo y desde este directorio ejecute el siguiente comando usando una ventana de [Símbolo del sistema para desarrolladores para Visual Studio](https://msdn.microsoft.com/library/ms229859.aspx) :

   ```cmd
   makecert ^
    -n "CN=*.cloudapp.net" ^
    -r -cy end -sky exchange -eku "1.3.6.1.5.5.7.3.1,1.3.6.1.5.5.7.3.2" ^
    -a sha256 -len 2048 ^
    -sr currentuser -ss root ^
    -sv MyCert.pvk MyCert.cer
   ```

Se le pedirá que escriba una contraseña para proteger la clave privada. Escriba una contraseña segura y confírmela. Luego se le pedirá que escriba nuevamente la contraseña. Haga clic en **Sí** al final para importarla al almacén Raíz de Entidades de certificación de confianza.

### <a name="create-a-pfx-file"></a>Creación de un archivo PFX

Ejecute el siguiente comando desde la misma ventana donde se ejecutó makecert; use la misma contraseña que usó para crear el certificado:

   ```cmd
   pvk2pfx -pvk MyCert.pvk -spc MyCert.cer -pfx MyCert.pfx -pi <password>
   ```

### <a name="import-the-client-certificate-into-the-personal-store"></a>Importación del certificado de cliente al almacén Personal

1. En el Explorador de Windows, haga doble clic en *MyCert.pfx*.
2. En el **Asistente para importar certificados**, seleccione **Usuario actual** y haga clic en **Siguiente**.
3. Confirme la ruta del archivo y haga clic en **Siguiente**.
4. Escriba la contraseña, deje activada la opción **Incluir todas las propiedades extendidas** y haga clic en **Siguiente**.
5. Deje activada la opción **Seleccionar automáticamente el almacén de certificados…** y haga clic en **Siguiente**.
6. Haga clic en **Finalizar** y en **Aceptar**.

### <a name="upload-the-pfx-file-to-the-cloud-service"></a>Carga del archivo PFX al servicio en la nube

1. Vaya a [Azure Portal](https://portal.azure.com).
2. Seleccione **Cloud Services**.
3. Seleccione el servicio en la nube que creó anteriormente para el servicio División y combinación.
4. Haga clic en **Certificados** en el menú superior.
5. Haga clic en **Cargar** en la barra inferior.
6. Seleccione el archivo PFX y escriba la misma contraseña anterior.
7. Una vez realizado, copie la huella digital del certificado desde la entrada nueva en la lista.

### <a name="update-the-service-configuration-file"></a>Actualizar el archivo de configuración del servicio

Pegue la huella digital del certificado copiada anteriormente en el atributo de huella digital/valor de esta configuración.
Para el rol de trabajo:

   ```xml
    <Setting name="DataEncryptionPrimaryCertificateThumbprint" value="" />
    <Certificate name="DataEncryptionPrimary" thumbprint="" thumbprintAlgorithm="sha1" />
   ```

Para el rol web:

   ```xml
    <Setting name="AdditionalTrustedRootCertificationAuthorities" value="" />
    <Setting name="AllowedClientCertificateThumbprints" value="" />
    <Setting name="DataEncryptionPrimaryCertificateThumbprint" value="" />
    <Certificate name="SSL" thumbprint="" thumbprintAlgorithm="sha1" />
    <Certificate name="CA" thumbprint="" thumbprintAlgorithm="sha1" />
    <Certificate name="DataEncryptionPrimary" thumbprint="" thumbprintAlgorithm="sha1" />
   ```

Tenga en cuenta que para las implementaciones de producción se deben usar certificados independientes para la CA, para cifrado, el certificado de servidor y los certificados de cliente. Para obtener instrucciones detalladas al respecto, consulte [Configuración de seguridad](elastic-scale-split-merge-security-configuration.md).

## <a name="deploy-your-service"></a>Implementación del servicio

1. Vaya a [Azure Portal](https://portal.azure.com).
2. Seleccione el servicio en la nube que creó anteriormente.
3. Haga clic en **Descripción general**.
4. Elija el entorno de ensayo y, a continuación, haga clic en **Cargar**.
5. En el cuadro de diálogo, escriba una etiqueta de implementación. Para "Paquete" y "Configuración", haga clic en "Desde local" y elija el archivo *SplitMergeService.cspkg* y el archivo .cscfg que configuró anteriormente.
6. Asegúrese de que la casilla de verificación con la etiqueta **Implementar aunque uno o varios roles contengan una sola instancia esté activada** .
7. Presione el botón de verificación que aparece en la esquina inferior derecha para comenzar la implementación. Tenga en cuenta que puede tardar unos minutos en completarse.

## <a name="troubleshoot-the-deployment"></a>Solución de problemas de la implementación

Si el rol web no puede ponerse en línea, probablemente haya un problema con la configuración de seguridad. Compruebe que TLS/SSL esté configurado como se describió anteriormente.

Si el rol de trabajo no puede ponerse en línea, pero el rol web sí, probablemente se trate de un problema al conectarse a la base de datos de estado que creó anteriormente.

- Asegúrese de que la cadena de conexión en el archivo .cscfg es correcta.
- Compruebe que existan el servidor y la base de datos y de que el identificador de usuario y la contraseña estén correctos.
- En el caso de Azure SQL Database, la cadena de conexión debe tener el formato:

   `Server=<serverName>.database.windows.net; Database=<databaseName>;User ID=<user>; Password=<password>; Encrypt=True; Connection Timeout=30`

- Asegúrese de que el nombre del servidor no comience por **https://** .
- Asegúrese de que el servidor permita que los servicios de Azure se conecten a él. Para ello, abra la base de datos en el portal y asegúrese de que la opción **Permitir el acceso a servicios de Azure** esté establecida en **Activado**.

## <a name="test-the-service-deployment"></a>Prueba de la implementación del servicio

### <a name="connect-with-a-web-browser"></a>Conexión con un explorador web

Determine el extremo web de su servicio División y combinación. Para averiguar esto, en el portal, vaya a **Información general** de su servicio en la nube y busque en **Dirección URL del sitio** en el lado derecho. Reemplace **http://** por **https://** , dado que la configuración de seguridad predeterminada deshabilita el punto de conexión HTTP. Cargue la página de esta dirección URL en el explorador.

### <a name="test-with-powershell-scripts"></a>Pruebas con scripts de PowerShell

Puede probar la implementación y su entorno si ejecuta los scripts de PowerShell de ejemplo incluidos.

> [!IMPORTANT]
> Los scripts de ejemplo se ejecutan en PowerShell 5.1. No se ejecutan actualmente en PowerShell 6 o posterior.

Los archivos de script incluidos son:

1. *SetupSampleSplitMergeEnvironment.ps1* : configura una capa de datos de prueba para División y combinación (consulte la tabla que aparece a continuación para ver una descripción detallada).
2. *ExecuteSampleSplitMerge.ps1* : ejecuta operaciones de prueba en la capa de datos de prueba (consulte la tabla que aparece a continuación para ver una descripción detallada).
3. *GetMappings.ps1*: script de muestra de nivel superior que imprime el estado actual de las asignaciones de particiones.
4. *ShardManagement.psm1*: script auxiliar que incluye la API de ShardManagement.
5. *SqlDatabaseHelpers.psm1*: script auxiliar para crear y administrar bases de datos en SQL Database.

   <table style="width:100%">
     <tr>
       <th>Archivo de PowerShell</th>
       <th>Pasos</th>
     </tr>
     <tr>
       <th rowspan="5">SetupSampleSplitMergeEnvironment.ps1</th>
       <td>1. Crea una base de datos del administrador de mapa de particiones</td>
     </tr>
     <tr>
       <td>2. Crea dos bases de datos de particiones.
     </tr>
     <tr>
       <td>3. Crea un mapa de particiones para esas bases de datos (elimina todo mapa de particiones existente en esas bases de datos). </td>
     </tr>
     <tr>
       <td>4. Crea una pequeña tabla de ejemplo en las particiones y rellena la tabla en una de las particiones.</td>
     </tr>
     <tr>
       <td>5. Declara la SchemaInfo para la tabla particionada.</td>
     </tr>
   </table>
   <table style="width:100%">
     <tr>
       <th>Archivo de PowerShell</th>
       <th>Pasos</th>
     </tr>
   <tr>
       <th rowspan="4">ExecuteSampleSplitMerge.ps1 </th>
       <td>1. Envía una solicitud de división al front-end web del servicio división-combinación, que divide la mitad de los datos entre la primera partición y la segunda.</td>
     </tr>
     <tr>
       <td>2. Sondea el front-end web para ver el estado de la solicitud de división y espera hasta que se complete la solicitud.</td>
     </tr>
     <tr>
       <td>3. Envía una solicitud de combinación al front-end del servicio División y combinación, que mueve los datos desde la segunda partición de vuelta a la primera partición.</td>
     </tr>
     <tr>
       <td>4. Sondea el front-end web para ver el estado de la solicitud de combinación y espera hasta que se complete la solicitud.</td>
     </tr>
   </table>

## <a name="use-powershell-to-verify-your-deployment"></a>Uso de PowerShell para comprobar la implementación

1. Abra una nueva ventana de PowerShell y vaya al directorio al que descargó el paquete División y combinación y, a continuación, vaya al directorio "powershell".

2. Cree un servidor (o elija un servidor existente) en el que se crearán las particiones y el administrador de mapa de particiones.

   > [!NOTE]
   > El script *SetupSampleSplitMergeEnvironment.ps1* crea todas estas bases de datos en el mismo servidor de manera predeterminada para que el script sea sencillo. Esta no es una restricción del servicio División y combinación mismo.

   Se requerirá un inicio de sesión de autenticación de SQL con acceso de lectura/escritura a las bases de datos para que el servicio División y combinación mueva los datos y actualice el mapa de particiones. Debido a que el servicio División y combinación se ejecuta en la nube, actualmente no es compatible con Autenticación integrada.

   Asegúrese de que el servidor esté configurado para permitir el acceso desde la dirección IP de la máquina que ejecuta estos scripts. Puede encontrar esta configuración en SQL Server/Firewalls y redes virtuales/Direcciones IP de cliente.

3. Ejecute el script *SetupSampleSplitMergeEnvironment.ps1* para crear el entorno de ejemplo.

   Ejecutar este script eliminará toda estructura de datos de administrador del mapa de particiones existente en la base de datos del administrador de mapa de particiones y las particiones. Puede ser útil volver a ejecutar el script si desea volver a inicializar las particiones o el mapa de particiones.

   Línea de comandos de ejemplo:

   ```cmd
   .\SetupSampleSplitMergeEnvironment.ps1
    -UserName 'mysqluser' -Password 'MySqlPassw0rd' -ShardMapManagerServerName 'abcdefghij.database.windows.net'
   ```

4. Ejecute el script Getmappings.ps1 para ver las asignaciones que existen actualmente en el entorno de ejemplo.

   ```cmd
   .\GetMappings.ps1
    -UserName 'mysqluser' -Password 'MySqlPassw0rd' -ShardMapManagerServerName 'abcdefghij.database.windows.net'
   ```

5. Ejecute el script *ExecuteSampleSplitMerge.ps1* para ejecutar una operación de división (moviendo la mitad de los datos de la primera partición a la segunda partición) y luego una operación Merge (moviendo los datos de vuelta a la primera partición). Si configuró TLS y dejó deshabilitado el punto de conexión http, asegúrese de usar el punto de conexión https://.

    Línea de comandos de ejemplo:

    ```cmd
    .\ExecuteSampleSplitMerge.ps1
    -UserName 'mysqluser' -Password 'MySqlPassw0rd'
    -ShardMapManagerServerName 'abcdefghij.database.windows.net'
    -SplitMergeServiceEndpoint 'https://mysplitmergeservice.cloudapp.net'
    -CertificateThumbprint '0123456789abcdef0123456789abcdef01234567'
    ```

    Si recibe el siguiente error, es muy probable que se trate de un problema con el certificado de su extremo web. Intente conectarse al extremo web con el explorador web de su preferencia y revise si hay un error en el certificado.

    `Invoke-WebRequest : The underlying connection was closed: Could not establish trust relationship for the SSL/TLSsecure channel.`

    Si está correcto, el resultado debiera verse de la siguiente manera:

    ```output
    > .\ExecuteSampleSplitMerge.ps1 -UserName 'mysqluser' -Password 'MySqlPassw0rd' -ShardMapManagerServerName 'abcdefghij.database.windows.net' -SplitMergeServiceEndpoint 'http://mysplitmergeservice.cloudapp.net' -CertificateThumbprint 0123456789abcdef0123456789abcdef01234567
    > Sending split request
    > Began split operation with id dc68dfa0-e22b-4823-886a-9bdc903c80f3
    > Polling split-merge request status. Press Ctrl-C to end
    > Progress: 0% | Status: Queued | Details: [Informational] Queued request
    > Progress: 5% | Status: Starting | Details: [Informational] Starting split-merge state machine for request.
    > Progress: 5% | Status: Starting | Details: [Informational] Performing data consistency checks on target     shards.
    > Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Moving reference tables from     source to target shard.
    > Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Waiting for reference tables copy     completion.
    > Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Moving reference tables from     source to target shard.
    > Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Moving key range [100:110) of     Sharded tables
    > Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Successfully copied key range     [100:110) for table [dbo].[MyShardedTable]
    > ...
    > ...
    > Progress: 90% | Status: Completing | Details: [Informational] Successfully deleted shardlets in table     [dbo].[MyShardedTable].
    > Progress: 90% | Status: Completing | Details: [Informational] Deleting any temp tables that were created     while processing the request.
    > Progress: 100% | Status: Succeeded | Details: [Informational] Successfully processed request.
    > Sending merge request
    > Began merge operation with id 6ffc308f-d006-466b-b24e-857242ec5f66
    > Polling request status. Press Ctrl-C to end
    > Progress: 0% | Status: Queued | Details: [Informational] Queued request
    > Progress: 5% | Status: Starting | Details: [Informational] Starting split-merge state machine for request.
    > Progress: 5% | Status: Starting | Details: [Informational] Performing data consistency checks on target     shards.
    > Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Moving reference tables from     source to target shard.
    > Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Moving key range [100:110) of     Sharded tables
    > Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Successfully copied key range     [100:110) for table [dbo].[MyShardedTable]
    > ...
    > ...
    > Progress: 90% | Status: Completing | Details: [Informational] Successfully deleted shardlets in table     [dbo].[MyShardedTable].
    > Progress: 90% | Status: Completing | Details: [Informational] Deleting any temp tables that were created     while processing the request.
    > Progress: 100% | Status: Succeeded | Details: [Informational] Successfully processed request.
    >
    ```

6. Experimente con otros tipos de datos. Todos estos scripts tienen un parámetro -ShardKeyType opcional que le permite especificar el tipo de clave. El valor predeterminado es Int32, pero también puede especificar Int64, Guid o Binary.

## <a name="create-requests"></a>Creación de solicitudes

El servicio se puede utilizar mediante la interfaz de usuario web o importando y utilizando el módulo de PowerShell SplitMerge.psm1 que enviará sus solicitudes a través del rol web.

El servicio puede mover los datos de las tablas particionadas y de las tablas de referencia. Una tabla particionada tiene una columna de clave de particionamiento y distintos datos de fila en cada partición. Una tabla de referencia no está particionada, por lo que contiene los mismos datos de fila en cada partición. Las tablas de referencia son útiles para los datos que no cambian con frecuencia y que se usan para UNIRSE con tablas particionadas en consultas.

Para realizar una operación de división y combinación, debe declarar las tablas particionadas y las tablas de referencia que desea mover. Esto se realiza con la API **SchemaInfo** . Esta API está en el espacio de nombres **Microsoft.Azure.SqlDatabase.ElasticScale.ShardManagement.Schema** .

1. Para cada tabla particionada, cree un objeto **ShardedTableInfo** que describa el nombre del esquema principal de la tabla (opcional, se define de manera predeterminada en "dbo"), el nombre de la tabla y el nombre de la columna de esa tabla que contiene la clave de particionamiento.
2. Para cada tabla de referencia, cree un objeto **ReferenceTableInfo** que describa el nombre del esquema principal de la tabla (opcional, se define de manera predeterminada en "dbo") y el nombre de la tabla.
3. Agregue los objetos TableInfo anteriores a un nuevo objeto **SchemaInfo** .
4. Obtenga una referencia a un objeto **ShardMapManager** y llame a **GetSchemaInfoCollection**.
5. Agregue **SchemaInfo** a **SchemaInfoCollection** y proporcione el nombre del mapa de particiones.

Es posible ver un ejemplo de esto en el script SetupSampleSplitMergeEnvironment.ps1.

El servicio División y combinación no crea la base de datos de destino (o esquema para cualquier tabla de la base de datos) por usted. Deben crearse previamente antes de enviar una solicitud al servicio.

## <a name="troubleshooting"></a>Solución de problemas

Es posible que, cuando ejecute los scripts de PowerShell de ejemplo, vea el siguiente mensaje:

   `Invoke-WebRequest : The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel.`

Este error significa que el certificado TLS/SSL no está configurado correctamente. Siga las instrucciones que aparecen en la sección "Conexión con un explorador web".

Si no se pueden enviar solicitudes, verá lo siguiente:

   `[Exception] System.Data.SqlClient.SqlException (0x80131904): Could not find stored procedure 'dbo.InsertRequest'.`

En este caso, compruebe el archivo de configuración, en particular la configuración de **WorkerRoleSynchronizationStorageAccountConnectionString**. Este error normalmente indica que el rol de trabajo no pudo inicializar correctamente la base de datos de metadatos la primera vez.

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/allowed-services.png
[2]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/manage.png
[3]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/staging.png
[4]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/upload.png
[5]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/storage.png
