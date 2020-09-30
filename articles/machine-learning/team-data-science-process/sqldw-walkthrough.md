---
title: 'Compilación e implementación de un modelo con Azure Synapse Analytics: Proceso de ciencia de datos en equipo'
description: Compile e implemente un modelo de aprendizaje automático mediante Azure Synapse Analytics con un conjunto de datos disponible públicamente.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, devx-track-python, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 59935d2659d27059617059c021fef9b6a2b552e0
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89440208"
---
# <a name="the-team-data-science-process-in-action-using-azure-synapse-analytics"></a>Proceso de ciencia de datos en equipos en acción: uso de Azure Synapse Analytics
En este tutorial le guiaremos a través de la creación e implementación de un modelo de aprendizaje automático mediante Azure Synapse Analytics para un conjunto de datos disponible públicamente: el conjunto de datos [NYC Taxi Trips](https://www.andresmh.com/nyctaxitrips/). El modelo de clasificación binaria construido predice si se dará propina por la carrera o no.  Los modelos incluyen la clasificación multiclase (si hay una propina o si no la hay) y la regresión (la distribución de los importes de propinas pagadas).

El procedimiento sigue el flujo de trabajo del [proceso de ciencia de datos en equipos (TDSP)](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/) . Se muestra cómo configurar un entorno de ciencia de datos, cómo cargar los datos en Azure Synapse Analytics y cómo usar Azure Synapse Analytics o un IPython Notebook para explorar las características de datos y de diseño para modelar. Luego se muestra cómo compilar e implementar un modelo con Azure Machine Learning.

## <a name="the-nyc-taxi-trips-dataset"></a><a name="dataset"></a>Conjunto de datos NYC Taxi Trips
El conjunto de datos NYC Taxi Trip consta de aproximadamente 20 GB de archivos CSV comprimidos (aproximadamente, 48 GB sin comprimir), que registran más de 173 millones de carreras individuales y las tarifas pagadas por cada carrera. Cada registro de carrera incluye la hora y la ubicación de recogida y llegada, el número de licencia del taxista anonimizado y el número de placa (id. único del taxi). Los datos cubren todos los viajes del año 2013 y se proporcionan en los dos conjuntos de datos siguientes para cada mes:

1. El archivo **trip_data.csv** contiene información detallada de las carreras, como el número de pasajeros, los puntos de recogida y destino, la duración de las carreras y la longitud del recorrido. Estos son algunos registros de ejemplo:

`medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude`

`89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171`

`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066`

`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002`

`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388`

`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868`

2. El archivo **trip_fare.csv** contiene información detallada de la tarifa que se paga en cada carrera, como el tipo de pago, el importe de la tarifa, los suplementos e impuestos, las propinas y los peajes, y el importe total pagado. Estos son algunos registros de ejemplo:

`medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount`

`89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7`

`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7`

`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7`

`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6`

`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5`

La **clave única** para unir trip\_data y trip\_fare se compone de los tres campos siguientes:

* medallion,
* hack\_license y
* pickup\_datetime.

## <a name="address-three-types-of-prediction-tasks"></a><a name="mltasks"></a>Realicemos tres tipos de tareas de predicción
Se formulan tres problemas de predicción basados en el valor *tip\_amount* para mostrar tres tipos de tareas de modelado:

1. **Clasificación binaria**: Permite predecir si se pagó o no una propina tras una carrera; es decir, un valor de *tip\_amount* mayor que 0 $ es un ejemplo positivo, mientras que un valor de *tip\_amount* de 0 $ es un ejemplo negativo.
2. **Clasificación multiclase**: permite predecir el intervalo de la propina de la carrera. Dividimos *tip\_amount* en cinco ubicaciones o clases:

`Class 0 : tip_amount = $0`

`Class 1 : tip_amount > $0 and tip_amount <= $5`

`Class 2 : tip_amount > $5 and tip_amount <= $10`

`Class 3 : tip_amount > $10 and tip_amount <= $20`

`Class 4 : tip_amount > $20`

3. **Tarea de regresión**: Permite predecir el importe de la propina pagada por una carrera.

## <a name="set-up-the-azure-data-science-environment-for-advanced-analytics"></a><a name="setup"></a>Configuración del entorno de ciencia de datos de Azure para análisis avanzado
Para configurar el entorno de ciencia de datos de Azure, siga estos pasos.

**Cree su propia cuenta de Almacenamiento de blobs de Azure.**

* Cuando aprovisione su propia instancia de Azure Blob Storage, elija una ubicación geográfica para esta en la región **centro-sur de EE.UU.** , o lo más cerca posible de esta región, ya que es donde se almacenan los datos de NYC Taxi. Los datos se copian con AzCopy desde el contenedor de Almacenamiento de blobs público a un contenedor de su propia cuenta de almacenamiento. Cuanto más se acerque el Almacenamiento de blobs de Azure a la región centro-sur de EE. UU., más rápida se completará esta tarea (paso 4).
* Para crear una cuenta propia de Azure Storage, siga los pasos descritos en [Acerca de las cuentas de Azure Storage](../../storage/common/storage-create-storage-account.md). Asegúrese de hacer anotaciones en los valores de las credenciales de la cuenta de almacenamiento siguientes, que necesitará más adelante en este tutorial.

  * **Nombre de cuenta de almacenamiento**
  * **Clave de cuenta de almacenamiento**
  * **Nombre de contenedor** (en donde los datos se van a almacenar en el Almacenamiento de blobs de Azure)

**Aprovisione su instancia de Azure Synapse Analytics.**
Siga la documentación de [Creación de una instancia de Azure Synapse Analytics en Azure Portal y realización de consultas](../../synapse-analytics/sql-data-warehouse/create-data-warehouse-portal.md) para aprovisionar una instancia de Azure Synapse Analytics. Asegúrese de que hacer anotaciones en las credenciales de Azure Synapse Analytics siguientes que se usarán en los pasos posteriores.

* **Nombre del servidor**: \<server Name>.database.windows.net
* **Nombre de SQLDW (base de datos)**
* **Nombre de usuario**
* **Contraseña**

**Instale Visual Studio y SQL Server Data Tools.** Para obtener instrucciones, vea [Introducción a Visual Studio 2019 para Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-install-visual-studio.md).

**Conéctese a Azure Synapse Analytics con Visual Studio.** Para obtener instrucciones, consulte los pasos 1 y 2 en [Conexión a SQL Analytics en Azure Synapse Analytics](../../synapse-analytics/sql/connect-overview.md).

> [!NOTE]
> Ejecute la siguiente consulta SQL en la base de datos que creó en Azure Synapse Analytics (en lugar de la consulta proporcionada en el paso 3 del tema sobre la conexión) para **crear una clave maestra**.
>
>

```sql
BEGIN TRY
       --Try to create the master key
    CREATE MASTER KEY
END TRY
BEGIN CATCH
       --If the master key exists, do nothing
END CATCH;
```

**Cree un área de trabajo de Azure Machine Learning en su suscripción de Azure.** Para ver instrucciones, consulte [Creación de un área de trabajo de Azure Machine Learning](../studio/create-workspace.md).

## <a name="load-the-data-into-azure-synapse-analytics"></a><a name="getdata"></a>Carga de datos en Azure Synapse Analytics
Abra una consola de comandos de Windows PowerShell. Ejecute los comandos de PowerShell siguientes para descargar los archivos de script SQL de ejemplo que compartimos en GitHub en un directorio local especificado con el parámetro *-DestDir*. Puede cambiar el valor del parámetro *-DestDir* en cualquier directorio local. Si *-DestDir* no existe, lo creará el script de PowerShell.

> [!NOTE]
> Es posible que necesite **ejecutar como administrador** el siguiente script de PowerShell si su directorio *DestDir* necesita privilegios de administrador para crearlo o escribir en él.
>
>

```azurepowershell
$source = "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/SQLDW/Download_Scripts_SQLDW_Walkthrough.ps1"
$ps1_dest = "$pwd\Download_Scripts_SQLDW_Walkthrough.ps1"
$wc = New-Object System.Net.WebClient
$wc.DownloadFile($source, $ps1_dest)
.\Download_Scripts_SQLDW_Walkthrough.ps1 –DestDir 'C:\tempSQLDW'
```

Cuando se haya ejecutado correctamente, el directorio de trabajo actual cambia a *-DestDir*. Debe ver una pantalla similar a la siguiente:

![El directorio de trabajo actual cambia][19]

En su *-DestDir*, ejecute el siguiente script de PowerShell en modo de administrador:

```azurepowershell
./SQLDW_Data_Import.ps1
```

Cuando se ejecuta el script de PowerShell por primera vez, se le pedirá que introduzca información desde Azure Synapse Analytics y la cuenta de Almacenamiento de blobs de Azure. Cuando este script de PowerShell termine de ejecutarse por primera vez, las credenciales indicadas se habrán escrito en un archivo de configuración SQLDW.conf del directorio de trabajo actual. La ejecución futura de este archivo de script de PowerShell puede leer todos los parámetros necesarios de este archivo de configuración. Si necesita cambiar algunos parámetros, puede elegir escribir los parámetros en la pantalla después del aviso mediante la eliminación de este archivo de configuración y de escribir los valores de parámetros cuando se le solicite o cambiar los valores de parámetros mediante la edición del archivo SQLDW.conf en el directorio *-DestDir* .

> [!NOTE]
> Para evitar conflictos de nombres de esquema con los que ya existen en Azure Synapse Analytics, al leer los parámetros directamente desde el archivo SQLDW.conf, se agrega un número aleatorio de tres dígitos al nombre del esquema desde el archivo SQLDW.conf como el nombre de esquema predeterminado para cada ejecución. El script de PowerShell puede pedirle un nombre de esquema: se puede especificar el nombre a discreción del usuario.
>
>

Este archivo de **script de PowerShell** realiza las tareas siguientes:

* **Descarga e instala AzCopy**, si AzCopy no se ha instalado aún

  ```azurepowershell
  $AzCopy_path = SearchAzCopy
  if ($AzCopy_path -eq $null){
         Write-Host "AzCopy.exe is not found in C:\Program Files*. Now, start installing AzCopy..." -ForegroundColor "Yellow"
      InstallAzCopy
      $AzCopy_path = SearchAzCopy
  }
      $env_path = $env:Path
      for ($i=0; $i -lt $AzCopy_path.count; $i++){
          if ($AzCopy_path.count -eq 1){
              $AzCopy_path_i = $AzCopy_path
          } else {
              $AzCopy_path_i = $AzCopy_path[$i]
          }
          if ($env_path -notlike '*' +$AzCopy_path_i+'*'){
              Write-Host $AzCopy_path_i 'not in system path, add it...'
              [Environment]::SetEnvironmentVariable("Path", "$AzCopy_path_i;$env_path", "Machine")
              $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine")
              $env_path = $env:Path
          }
  ```

* **Copia datos en la cuenta de Almacenamiento de blobs privada** desde el blob público con AzCopy

  ```azurepowershell
  Write-Host "AzCopy is copying data from public blob to yo storage account. It may take a while..." -ForegroundColor "Yellow"
  $start_time = Get-Date
  AzCopy.exe /Source:$Source /Dest:$DestURL /DestKey:$StorageAccountKey /S
  $end_time = Get-Date
  $time_span = $end_time - $start_time
  $total_seconds = [math]::Round($time_span.TotalSeconds,2)
  Write-Host "AzCopy finished copying data. Please check your storage account to verify." -ForegroundColor "Yellow"
  Write-Host "This step (copying data from public blob to your storage account) takes $total_seconds seconds." -ForegroundColor "Green"
  ```

* **Carga datos mediante Polybase (mediante la ejecución de LoadDataToSQLDW.sql) para Azure Synapse Analytics** de la cuenta de almacenamiento de blobs privada con los siguientes comandos.

  * Creación de un esquema

    ```sql
    EXEC (''CREATE SCHEMA {schemaname};'');
    ```

  * Creación de una credencial con ámbito de base de datos

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL {KeyAlias}
    WITH IDENTITY = ''asbkey'' ,
    Secret = ''{StorageAccountKey}''
    ```

  * Creación de un origen de datos externo para un blob de Azure Storage

    ```sql
    CREATE EXTERNAL DATA SOURCE {nyctaxi_trip_storage}
    WITH
    (
        TYPE = HADOOP,
        LOCATION =''wasbs://{ContainerName}@{StorageAccountName}.blob.core.windows.net'',
        CREDENTIAL = {KeyAlias}
    )
    ;

    CREATE EXTERNAL DATA SOURCE {nyctaxi_fare_storage}
    WITH
    (
        TYPE = HADOOP,
        LOCATION =''wasbs://{ContainerName}@{StorageAccountName}.blob.core.windows.net'',
        CREDENTIAL = {KeyAlias}
    )
    ;
    ```

  * Cree un formato de archivo externo para un archivo .csv. Los datos se descomprimen y los campos se separan con el carácter de barra vertical.

    ```sql
    CREATE EXTERNAL FILE FORMAT {csv_file_format}
    WITH
    (
        FORMAT_TYPE = DELIMITEDTEXT,
        FORMAT_OPTIONS
        (
            FIELD_TERMINATOR ='','',
            USE_TYPE_DEFAULT = TRUE
        )
    )
    ;
    ```

  * Cree tablas externas de tarifas y propinas para el conjunto de datos de NYC Taxi en el Almacenamiento de blobs de Azure.

    ```sql
    CREATE EXTERNAL TABLE {external_nyctaxi_fare}
    (
        medallion varchar(50) not null,
        hack_license varchar(50) not null,
        vendor_id char(3),
        pickup_datetime datetime not null,
        payment_type char(3),
        fare_amount float,
        surcharge float,
        mta_tax float,
        tip_amount float,
        tolls_amount float,
        total_amount float
    )
    with (
        LOCATION    = ''/nyctaxifare/'',
        DATA_SOURCE = {nyctaxi_fare_storage},
        FILE_FORMAT = {csv_file_format},
        REJECT_TYPE = VALUE,
        REJECT_VALUE = 12
    )

      CREATE EXTERNAL TABLE {external_nyctaxi_trip}
      (
             medallion varchar(50) not null,
             hack_license varchar(50)  not null,
             vendor_id char(3),
             rate_code char(3),
             store_and_fwd_flag char(3),
             pickup_datetime datetime  not null,
             dropoff_datetime datetime,
             passenger_count int,
             trip_time_in_secs bigint,
             trip_distance float,
             pickup_longitude varchar(30),
             pickup_latitude varchar(30),
             dropoff_longitude varchar(30),
             dropoff_latitude varchar(30)
      )
      with (
          LOCATION    = ''/nyctaxitrip/'',
          DATA_SOURCE = {nyctaxi_trip_storage},
          FILE_FORMAT = {csv_file_format},
          REJECT_TYPE = VALUE,
          REJECT_VALUE = 12
      )
    ```

    - Carga de datos de tablas externas de Azure Blob Storage en Azure Synapse Analytics

      ```sql
      CREATE TABLE {schemaname}.{nyctaxi_fare}
      WITH
      (
          CLUSTERED COLUMNSTORE INDEX,
          DISTRIBUTION = HASH(medallion)
      )
      AS
      SELECT *
      FROM   {external_nyctaxi_fare}
      ;

      CREATE TABLE {schemaname}.{nyctaxi_trip}
      WITH
      (
          CLUSTERED COLUMNSTORE INDEX,
          DISTRIBUTION = HASH(medallion)
      )
      AS
      SELECT *
      FROM   {external_nyctaxi_trip}
      ;
      ```

    - Cree una tabla de datos de ejemplo (NYCTaxi_Sample) e inserte datos en ella mediante la selección de consultas SQL en las tablas de carreras y tarifas. (Algunos pasos de este tutorial necesitan usar esta tabla de ejemplo).

      ```sql
      CREATE TABLE {schemaname}.{nyctaxi_sample}
      WITH
      (
          CLUSTERED COLUMNSTORE INDEX,
          DISTRIBUTION = HASH(medallion)
      )
      AS
      (
          SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount, f.total_amount, f.tip_amount,
          tipped = CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END,
          tip_class = CASE
                  WHEN (tip_amount = 0) THEN 0
                  WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
                  WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
                  WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
                  ELSE 4
              END
          FROM {schemaname}.{nyctaxi_trip} t, {schemaname}.{nyctaxi_fare} f
          WHERE datepart("mi",t.pickup_datetime) = 1
          AND t.medallion = f.medallion
          AND   t.hack_license = f.hack_license
          AND   t.pickup_datetime = f.pickup_datetime
          AND   pickup_longitude <> ''0''
          AND   dropoff_longitude <> ''0''
      )
      ;
      ```

La ubicación geográfica de las cuentas de almacenamiento afecta a los tiempos de carga.

> [!NOTE]
> Según la ubicación geográfica de la cuenta de almacenamiento de blobs privada, el proceso de copiar datos de un blob público a su cuenta de almacenamiento privada puede tardar unos 15 minutos o incluso más tiempo, y el proceso de carga de datos desde la cuenta de almacenamiento a Azure Synapse Analytics podría tardar 20 minutos o más tiempo.
>
>

Tendrá que decidir qué hacer si tiene archivos de origen y de destino duplicados.

> [!NOTE]
> Si los archivos .csv que se van a copiar desde el almacenamiento de blobs público a la cuenta de almacenamiento de blobs privada ya existen en la cuenta de almacenamiento de blobs privada, AzCopy le preguntará si desea sobrescribirlos. Si no desea sobrescribirlos, escriba **n** cuando se le solicite. Si desea sobrescribir **todos** ellos, escriba **a** cuando se le solicite. También puede escribir **y** para sobrescribir los archivos .csv individualmente.
>
>

![Salida de AzCopy][21]

Puede usar sus propios datos. Si los datos están en la máquina local en la aplicación de la vida real, todavía puede usar AzCopy para cargar los datos locales a su área privada de Azure Blob Storage. Solo tiene que cambiar la ubicación de **origen**, `$Source = "http://getgoing.blob.core.windows.net/public/nyctaxidataset"`, en el comando de AzCopy del archivo de scripts de PowerShell por un directorio local que contiene los datos.

> [!TIP]
> Si los datos ya están en el Almacenamiento de blobs de Azure privado en la aplicación de la vida real, puede omitir el paso de AzCopy en el script de PowerShell y cargar directamente los datos en Azure Synapse Analytics. Esto requerirá modificaciones adicionales del script para adaptarlo al formato de los datos.
>
>

Este script de Powershell también conecta la información de Azure Synapse Analytics en los archivos de ejemplo de exploración de datos SQLDW_Explorations.sql, SQLDW_Explorations.ipynb y SQLDW_Explorations_Scripts.py para que estos tres archivos están listos para ser probados de inmediato cuando finalice el script de PowerShell.

Después de una ejecución correcta, verá una pantalla similar a la siguiente:

![Salida de una ejecución correcta del script][20]

## <a name="data-exploration-and-feature-engineering-in-azure-synapse-analytics"></a><a name="dbexplore"></a>Exploración de datos y diseño de características en Azure Synapse Analytics
En esta sección, realizamos la generación de características y la exploración de datos mediante la ejecución de consultas SQL en Azure Synapse Analytics directamente mediante **Visual Studio Data Tools**. Todas las consultas SQL que se usan en esta sección se pueden encontrar en el script de ejemplo llamado *SQLDW_Explorations.sql*. Este archivo ya lo ha descargado en el directorio local el script de PowerShell. También puede recuperarlo desde [GitHub](https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/SQLDW/SQLDW_Explorations.sql). Pero el archivo de GitHub no tiene la información de Azure Synapse Analytics conectada.

Conéctese con Azure Synapse Analytics utilizando Visual Studio con el nombre de inicio de sesión de Azure Synapse Analytics y la contraseña y abra el **Explorador de objetos SQL** para confirmar que la base de datos y las tablas se han importado. Recupere el archivo *SQLDW_Explorations.sql*.

> [!NOTE]
> Para abrir un editor de consultas de Almacenamiento de datos paralelos (PDW), utilice el comando **Nueva consulta** mientras el PDW está seleccionado en el **Explorador de objetos SQL**. El editor de consultas estándar de SQL no es compatible con PDW.
>
>

A continuación se muestra el tipo de tareas de exploración de datos y de generación de características realizado en esta sección:

* Explorar distribuciones de datos de algunos campos en diferentes ventanas de tiempo.
* Investigar la calidad de los datos de los campos de longitud y latitud.
* Generar etiquetas de clasificación binaria y multiclase según **tip\_amount**.
* Generar características y calcular o comparar distancias de carreras.
* Combinar las dos tablas y extraer una muestra aleatoria que se usará para generar modelos.

### <a name="data-import-verification"></a>Comprobación de la importación de datos
Estas consultas proporcionan una comprobación rápida del número de filas y columnas en las tablas que se rellenaron anteriormente mediante la importación masiva paralela de Polybase.

-- Informe con el número de filas de la tabla <nyctaxi_trip> sin recorrido de tabla

   ```sql
   SELECT SUM(rows) FROM sys.partitions WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_trip>')
   ```

-- Informe con el número de columnas de la tabla <nyctaxi_trip>

   ```sql
   SELECT COUNT(*) FROM information_schema.columns WHERE table_name = '<nyctaxi_trip>' AND table_schema = '<schemaname>'
   ```

**Salida:** debe obtener 173 179 759 filas y 14 columnas.

### <a name="exploration-trip-distribution-by-medallion"></a>Exploración: Distribución de carreras por medallion
Esta consulta de ejemplo identifica las licencias (números de taxi) que han completado más de 100 carreras dentro de un período de tiempo especificado. La consulta se beneficiaría del acceso de la tabla con particiones, ya que está condicionada por el esquema de partición de **pickup\_datetime**. La consulta el conjunto de datos completo también hará uso de la tabla con particiones o del recorrido de índice.

```sql
SELECT medallion, COUNT(*)
FROM <schemaname>.<nyctaxi_fare>
WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
GROUP BY medallion
HAVING COUNT(*) > 100
```

**Salida:** la consulta debe devolver una tabla con filas en las que se especifiquen las 13 369 licencias (taxis) y el número de carreras completadas en 2013. La última columna contiene el recuento del número de carreras realizadas.

### <a name="exploration-trip-distribution-by-medallion-and-hack_license"></a>Exploración: distribución de carreras por medallion y hack_license
Este ejemplo identifica las licencias (números de taxi) y los números de hack_license (conductores) que han completado más de 100 carreras dentro de un período de tiempo.

```sql
SELECT medallion, hack_license, COUNT(*)
FROM <schemaname>.<nyctaxi_fare>
WHERE pickup_datetime BETWEEN '20130101' AND '20130131'
GROUP BY medallion, hack_license
HAVING COUNT(*) > 100
```

**Salida:** la consulta debe devolver una tabla con 13 369 filas en las que se especifiquen los 13 369 identificadores de vehículo/conductor que han realizado más de 100 carreras en 2013. La última columna contiene el recuento del número de carreras realizadas.

### <a name="data-quality-assessment-verify-records-with-incorrect-longitude-andor-latitude"></a>Evaluación de la calidad de los datos: compruebe los registros con longitud o latitud incorrectas
En este ejemplo se investiga si alguno de los campos de longitud y latitud contiene un valor no válido (los grados radianes deben encontrarse entre -90 y 90) o tienen coordenadas (0, 0).

```sql
SELECT COUNT(*) FROM <schemaname>.<nyctaxi_trip>
WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
AND  (CAST(pickup_longitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(pickup_latitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(dropoff_longitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(dropoff_latitude AS float) NOT BETWEEN -90 AND 90
OR    (pickup_longitude = '0' AND pickup_latitude = '0')
OR    (dropoff_longitude = '0' AND dropoff_latitude = '0'))
```

**Salida:** la consulta devuelve 837 467 carreras con campos de latitud o longitud no válidos.

### <a name="exploration-tipped-vs-not-tipped-trips-distribution"></a>Exploración: distribución de carreras con propinas frente a sin propinas
Este ejemplo busca el número de carreras en las que se han dado propinas frente a aquellas en las que no se han dado en un período de tiempo especificado (o en el conjunto de datos completo si se abarca todo el año, como se establece aquí). Esta distribución refleja la distribución de etiquetas binarias que se usará más adelante para el modelado de clasificación binaria.

```sql
SELECT tipped, COUNT(*) AS tip_freq FROM (
  SELECT CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped, tip_amount
  FROM <schemaname>.<nyctaxi_fare>
  WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
GROUP BY tipped
```

**Salida:** La consulta debe devolver las siguientes frecuencias de propinas para el año 2013: 90 447 622 con propina y 82 264 709 sin propina.

### <a name="exploration-tip-classrange-distribution"></a>Exploración: Distribución por intervalos y clases de propinas
Este ejemplo calcula la distribución de los intervalos de propinas de un período de tiempo determinado (o en el conjunto de datos completo si abarca todo el año). Esta distribución de las clases de etiquetas se usará posteriormente para el modelado de clasificación multiclase.

```sql
SELECT tip_class, COUNT(*) AS tip_freq FROM (
    SELECT CASE
        WHEN (tip_amount = 0) THEN 0
        WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
        WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
        WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
        ELSE 4
    END AS tip_class
FROM <schemaname>.<nyctaxi_fare>
WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
GROUP BY tip_class
```

**Salida:**

| tip_class | tip_freq |
| --- | --- |
| 1 |82 230 915 |
| 2 |6 198 803 |
| 3 |1 932 223 |
| 0 |82 264 625 |
| 4 |85 765 |

### <a name="exploration-compute-and-compare-trip-distance"></a>Exploración: proceso y comparación de la distancia de la carrera
En este ejemplo se convierte la longitud y latitud de los puntos de recogida y destino a puntos geográficos de SQL, se calcula la distancia de la carrera mediante la diferencia de puntos geográficos de SQL y se devuelve una muestra aleatoria de los resultados de la comparación. En el ejemplo se limitan los resultados a coordenadas válidas usando solo la consulta de evaluación de calidad de datos tratada anteriormente.

```sql
/****** Object:  UserDefinedFunction [dbo].[fnCalculateDistance] ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

IF EXISTS (SELECT * FROM sys.objects WHERE type IN ('FN', 'IF') AND name = 'fnCalculateDistance')
  DROP FUNCTION fnCalculateDistance
GO

-- User-defined function to calculate the direct distance  in mile between two geographical coordinates.
CREATE FUNCTION [dbo].[fnCalculateDistance] (@Lat1 float, @Long1 float, @Lat2 float, @Long2 float)

RETURNS float
AS
BEGIN
      DECLARE @distance decimal(28, 10)
      -- Convert to radians
      SET @Lat1 = @Lat1 / 57.2958
      SET @Long1 = @Long1 / 57.2958
      SET @Lat2 = @Lat2 / 57.2958
      SET @Long2 = @Long2 / 57.2958
      -- Calculate distance
      SET @distance = (SIN(@Lat1) * SIN(@Lat2)) + (COS(@Lat1) * COS(@Lat2) * COS(@Long2 - @Long1))
      --Convert to miles
      IF @distance <> 0
      BEGIN
        SET @distance = 3958.75 * ATAN(SQRT(1 - POWER(@distance, 2)) / @distance);
      END
      RETURN @distance
END
GO

SELECT pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude,
dbo.fnCalculateDistance(pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude) AS DirectDistance
FROM <schemaname>.<nyctaxi_trip>
WHERE datepart("mi",pickup_datetime)=1
AND CAST(pickup_latitude AS float) BETWEEN -90 AND 90
AND CAST(dropoff_latitude AS float) BETWEEN -90 AND 90
AND pickup_longitude != '0' AND dropoff_longitude != '0'
```

### <a name="feature-engineering-using-sql-functions"></a>Diseño de características con las funciones de SQL
A veces, las funciones de SQL pueden ser una opción eficiente de diseño de características. En este tutorial, hemos definido una función SQL para calcular la distancia directa entre las ubicaciones de recogida y entrega. Puede ejecutar los scripts SQL siguientes en **Visual Studio Data Tools**.

Este es el script SQL que define la función de distancia.

```sql
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

IF EXISTS (SELECT * FROM sys.objects WHERE type IN ('FN', 'IF') AND name = 'fnCalculateDistance')
  DROP FUNCTION fnCalculateDistance
GO

-- User-defined function calculate the direct distance between two geographical coordinates.
CREATE FUNCTION [dbo].[fnCalculateDistance] (@Lat1 float, @Long1 float, @Lat2 float, @Long2 float)

RETURNS float
AS
BEGIN
      DECLARE @distance decimal(28, 10)
      -- Convert to radians
      SET @Lat1 = @Lat1 / 57.2958
      SET @Long1 = @Long1 / 57.2958
      SET @Lat2 = @Lat2 / 57.2958
      SET @Long2 = @Long2 / 57.2958
      -- Calculate distance
      SET @distance = (SIN(@Lat1) * SIN(@Lat2)) + (COS(@Lat1) * COS(@Lat2) * COS(@Long2 - @Long1))
      --Convert to miles
      IF @distance <> 0
      BEGIN
        SET @distance = 3958.75 * ATAN(SQRT(1 - POWER(@distance, 2)) / @distance);
      END
      RETURN @distance
END
GO
```

Este es un ejemplo para llamar a esta función para generar características en la consulta SQL:

-- Consulta de ejemplo para llamar a la función para crear características

   ```sql
SELECT pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude,
dbo.fnCalculateDistance(pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude) AS DirectDistance
FROM <schemaname>.<nyctaxi_trip>
WHERE datepart("mi",pickup_datetime)=1
AND CAST(pickup_latitude AS float) BETWEEN -90 AND 90
AND CAST(dropoff_latitude AS float) BETWEEN -90 AND 90
AND pickup_longitude != '0' AND dropoff_longitude != '0'
   ```

**Salida:** esta consulta genera una tabla (con 2 803 538 filas) con las latitudes y longitud de los puntos de origen y destino y las distancias directas correspondientes en millas. Estos son los resultados de las tres primeras filas:

| (Número de fila) | pickup_latitude | pickup_longitude | dropoff_latitude | dropoff_longitude | DirectDistance |
| --- | --- | --- | --- | --- | --- |
| 1 |40,731804 |-74,001083 |40,736622 |-73,988953 |0,7169601222 |
| 2 |40,715794 |-74,010635 |40,725338 |-74,00399 |0,7448343721 |
| 3 |40,761456 |-73,999886 |40,766544 |-73,988228 |0,7037227967 |

### <a name="prepare-data-for-model-building"></a>Preparar datos para la generación de modelos
La siguiente consulta combina las tablas **nyctaxi\_trip** y **nyctaxi\_fare**, genera una etiqueta de clasificación binaria **tipped**, una etiqueta de clasificación multiclase **tip\_class** y extrae una muestra aleatoria del conjunto de datos combinado completo. El muestreo se realiza mediante la recuperación de un subconjunto de los viajes en función de la hora de recogida.  Esta consulta se puede copiar y pegar directamente en el módulo [Import Data](https://studio.azureml.net) (Importar datos) [import-data] de [Azure Machine Learning Studio (clásico)] para la ingesta directa de datos desde la instancia de SQL Database de Azure. La consulta excluye los registros con coordenadas (0, 0) incorrectas.

```sql
SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount,     f.total_amount, f.tip_amount,
    CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped,
    CASE WHEN (tip_amount = 0) THEN 0
        WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
        WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
        WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
        ELSE 4
    END AS tip_class
FROM <schemaname>.<nyctaxi_trip> t, <schemaname>.<nyctaxi_fare> f
WHERE datepart("mi",t.pickup_datetime) = 1
AND   t.medallion = f.medallion
AND   t.hack_license = f.hack_license
AND   t.pickup_datetime = f.pickup_datetime
AND   pickup_longitude != '0' AND dropoff_longitude != '0'
```

Cuando esté listo para continuar con Azure Machine Learning, puede:

1. Guardar la consulta SQL final para extraer y muestrear los datos, y copiar y pegar la consulta directamente en un módulo para importar datos [import-data] de Azure Machine Learning.
2. Conservar los datos muestreados y de ingeniería que planea usar para la generación de modelos en una nueva tabla de Azure Synapse Analytics y usar la nueva tabla en el módulo [Importar datos][import-data] de Azure Machine Learning. El script de PowerShell del paso anterior se ha encargado de hacerlo. Puede leer directamente de esta tabla en el módulo Importar datos.

## <a name="data-exploration-and-feature-engineering-in-ipython-notebook"></a><a name="ipnb"></a>Exploración de datos e ingeniería de características en IPython Notebook
En esta sección, se llevará a cabo la exploración de datos y la generación de características con consultas Python y SQL en la instancia de Azure Synapse Analytics creada anteriormente. Se han descargado un cuaderno de IPython Notebook de ejemplo denominado **SQLDW_Explorations.ipynb** y un archivo de script de Python **SQLDW_Explorations_Scripts.py** en el directorio local. También están disponibles en [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/SQLDW). Estos dos archivos son idénticos en los scripts de Python. El archivo de script de Python se proporciona en caso de que no tenga un servidor de IPython Notebook. Estos dos archivos de Python de muestra están diseñados en **Python 2.7**.

La información necesaria de Azure Synapse Analytics en el cuaderno de IPython Notebook de ejemplo y el archivo de script de Python descargados en el equipo local está conectada por el script PowerShell. Son ejecutables sin ninguna modificación.

Si ya ha configurado un área de trabajo de Azure Machine Learning, puede cargar directamente la instancia de ejemplo de IPython Notebook en el servicio IPython Notebook de Azure ML y comenzar a ejecutarla. Estos son los pasos para cargarla en el servicio IPython Notebook de Azure ML:

1. Inicie sesión en el área de trabajo de Azure Machine Learning, haga clic en **Studio"** en la parte superior y, luego, en **NOTEBOOKS** (CUADERNOS) en el lado izquierdo de la página web.

    ![Haga clic en Studio, a continuación, en CUADERNOS.][22]
2. Haga clic en **NEW** (NUEVO) en la esquina inferior izquierda de la página web y seleccione **Python 2**. A continuación, proporcione un nombre para el cuaderno y haga clic en la marca de verificación para crear el nuevo cuaderno de IPython Notebook en blanco.

    ![Haga clic en NUEVO y, a continuación, seleccione Python 2.][23]
3. Haga clic en el símbolo **Jupyter** en la esquina superior izquierda de la nueva instancia de IPython Notebook.

    ![Haga clic en el símbolo de Jupyter.][24]
4. Arrastre y coloque el cuaderno de IPython Notebook de ejemplo en la página **árbol** del servicio IPython Notebook de Azure Machine Learning y haga clic en **Cargar**. A continuación, se cargará el cuaderno de IPython Notebook de ejemplo en el servicio IPython Notebook de Aprendizaje automático de Azure.

    ![Haga clic en Cargar][25]

Para ejecutar el cuaderno de IPython Notebook de ejemplo o el archivo de script de Python, son necesarios los siguientes paquetes Python. Si usa el servicio IPython Notebook de Aprendizaje automático de Azure, estos paquetes ya están preinstalados.

- Pandas
- numpy
- matplotlib
- pyodbc
- PyTables

La secuencia recomendada al crear soluciones de análisis avanzado en Azure Machine Learning con datos de gran tamaño es la siguiente:

* Leer una pequeña muestra de los datos en una trama de datos en memoria.
* Realizar algunas visualizaciones y exploraciones con los datos de muestreo.
* Experimentar con el diseño de características con los datos de muestreo.
* Para el diseño de características, la exploración y la manipulación de datos más grandes, usar Python para emitir consultas SQL directamente en Azure Synapse Analytics.
* Decidir el tamaño de muestra que se usará para la creación del modelo de Azure Machine Learning.

A continuación, se muestran algunas exploraciones de datos, visualizaciones de datos y ejemplos de diseño de características. Se pueden encontrar más exploraciones de datos en el cuaderno de IPython Notebook de ejemplo y en el archivo de script de Python de ejemplo.

### <a name="initialize-database-credentials"></a>Inicialización de las credenciales de la base de datos
Inicialice la configuración de conexión de base de datos en las variables siguientes:

```sql
SERVER_NAME=<server name>
DATABASE_NAME=<database name>
USERID=<user name>
PASSWORD=<password>
DB_DRIVER = <database driver>
```

### <a name="create-database-connection"></a>Creación de conexiones de base de datos
Esta es la cadena de conexión que crea la conexión a la base de datos.

```sql
CONNECTION_STRING = 'DRIVER={'+DRIVER+'};SERVER='+SERVER_NAME+';DATABASE='+DATABASE_NAME+';UID='+USERID+';PWD='+PASSWORD
conn = pyodbc.connect(CONNECTION_STRING)
```

### <a name="report-number-of-rows-and-columns-in-table-nyctaxi_trip"></a>Informe con el número de filas y columnas de la tabla <nyctaxi_trip>

```sql
nrows = pd.read_sql('''
    SELECT SUM(rows) FROM sys.partitions
    WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_trip>')
''', conn)

print 'Total number of rows = %d' % nrows.iloc[0,0]

ncols = pd.read_sql('''
    SELECT COUNT(*) FROM information_schema.columns
    WHERE table_name = ('<nyctaxi_trip>') AND table_schema = ('<schemaname>')
''', conn)

print 'Total number of columns = %d' % ncols.iloc[0,0]
```

* Número total de filas = 173179759
* Número total de columnas = 14

### <a name="report-number-of-rows-and-columns-in-table-nyctaxi_fare"></a>Informe con el número de filas y columnas de la tabla <nyctaxi_fare>

```sql
nrows = pd.read_sql('''
    SELECT SUM(rows) FROM sys.partitions
    WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_fare>')
''', conn)

print 'Total number of rows = %d' % nrows.iloc[0,0]

ncols = pd.read_sql('''
    SELECT COUNT(*) FROM information_schema.columns
    WHERE table_name = ('<nyctaxi_fare>') AND table_schema = ('<schemaname>')
''', conn)

    print 'Total number of columns = %d' % ncols.iloc[0,0]
```

* Número total de filas = 173179759
* Número total de columnas = 11

### <a name="read-in-a-small-data-sample-from-the-azure-synapse-analytics-database"></a>Lectura de una muestra de datos pequeña de la base de datos de Azure Synapse Analytics

```sql
t0 = time.time()

query = '''
    SELECT TOP 10000 t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax,
        f.tolls_amount, f.total_amount, f.tip_amount
    FROM <schemaname>.<nyctaxi_trip> t, <schemaname>.<nyctaxi_fare> f
    WHERE datepart("mi",t.pickup_datetime) = 1
    AND   t.medallion = f.medallion
    AND   t.hack_license = f.hack_license
    AND   t.pickup_datetime = f.pickup_datetime
'''

df1 = pd.read_sql(query, conn)

t1 = time.time()
print 'Time to read the sample table is %f seconds' % (t1-t0)

print 'Number of rows and columns retrieved = (%d, %d)' % (df1.shape[0], df1.shape[1])
```

El tiempo empleado en leer la tabla de ejemplo es 14,096495 segundos
Número de filas y columnas recuperadas = (1000, 21)

### <a name="descriptive-statistics"></a>Estadísticas descriptivas
Ya puede explorar los datos de muestreo. Comenzamos echando un vistazo a algunas estadísticas descriptivas del campo **trip\_distance** (o de cualquier otro que elija).

```sql
df1['trip_distance'].describe()
```

### <a name="visualization-box-plot-example"></a>Visualización: Ejemplo de diagrama de caja
A continuación, observaremos el diagrama de caja de la distancia de la carrera para ver los cuantiles.

```sql
df1.boxplot(column='trip_distance',return_type='dict')
```

![Salida del diagrama de caja][1]

### <a name="visualization-distribution-plot-example"></a>Visualización: Ejemplo de diagrama de distribución
Diagramas que visualizan la distribución y un histograma de las distancias de las carreras muestreadas.

```sql
fig = plt.figure()
ax1 = fig.add_subplot(1,2,1)
ax2 = fig.add_subplot(1,2,2)
df1['trip_distance'].plot(ax=ax1,kind='kde', style='b-')
df1['trip_distance'].hist(ax=ax2, bins=100, color='k')
```

![Salida del diagrama de distribución][2]

### <a name="visualization-bar-and-line-plots"></a>Visualización: Diagramas de barras y líneas
En este ejemplo, se discretiza la distancia de la carrera en cinco discretizaciones y se visualizan los resultados de la discretización.

```sql
trip_dist_bins = [0, 1, 2, 4, 10, 1000]
df1['trip_distance']
trip_dist_bin_id = pd.cut(df1['trip_distance'], trip_dist_bins)
trip_dist_bin_id
```

Podemos trazar la distribución de discretización anterior en un gráfico de barras o líneas con:

```sql
pd.Series(trip_dist_bin_id).value_counts().plot(kind='bar')
```

![Salida del diagrama de barras][3]

y

```sql
pd.Series(trip_dist_bin_id).value_counts().plot(kind='line')
```

![Salida del diagrama de líneas][4]

### <a name="visualization-scatterplot-examples"></a>Visualización: Ejemplo de gráfico de dispersión
Se muestra el gráfico de dispersión entre **trip\_time\_in\_secs** y **trip\_distance** para ver si existe algún tipo de correlación

```sql
plt.scatter(df1['trip_time_in_secs'], df1['trip_distance'])
```

![Salida del gráfico de dispersión de la relación entre el tiempo y la distancia][6]

También podemos comprobar la relación entre **rate\_code** y **trip\_distance**.

```sql
plt.scatter(df1['passenger_count'], df1['trip_distance'])
```

![Salida del gráfico de dispersión de la relación entre el código y la distancia][8]

### <a name="data-exploration-on-sampled-data-using-sql-queries-in-ipython-notebook"></a>Exploración de datos en datos de muestreo mediante consultas SQL en IPython Notebook
En esta sección, se exploran las distribuciones de datos con los datos de los datos de muestreo que se conservan en la nueva tabla que se creó anteriormente. Se pueden realizar exploraciones similares con las tablas originales.

#### <a name="exploration-report-number-of-rows-and-columns-in-the-sampled-table"></a>Exploración: notificación del número de filas y columnas de la tabla de muestreo

```sql
nrows = pd.read_sql('''SELECT SUM(rows) FROM sys.partitions WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_sample>')''', conn)
print 'Number of rows in sample = %d' % nrows.iloc[0,0]

ncols = pd.read_sql('''SELECT count(*) FROM information_schema.columns WHERE table_name = ('<nyctaxi_sample>') AND table_schema = '<schemaname>'''', conn)
print 'Number of columns in sample = %d' % ncols.iloc[0,0]
```

#### <a name="exploration-tippednot-tripped-distribution"></a>Exploración: distribución con propinas y sin propinas

```sql
query = '''
SELECT tipped, count(*) AS tip_freq
    FROM <schemaname>.<nyctaxi_sample>
    GROUP BY tipped
    '''

    pd.read_sql(query, conn)
```

#### <a name="exploration-tip-class-distribution"></a>Exploración: distribución de la clase de prueba

```sql
query = '''
    SELECT tip_class, count(*) AS tip_freq
    FROM <schemaname>.<nyctaxi_sample>
    GROUP BY tip_class
'''

tip_class_dist = pd.read_sql(query, conn)
```

#### <a name="exploration-plot-the-tip-distribution-by-class"></a>Exploración: trazado de la distribución de propinas por clase

```sql
tip_class_dist['tip_freq'].plot(kind='bar')
```

![Diagrama 26][26]

#### <a name="exploration-daily-distribution-of-trips"></a>Exploración: distribución diaria de carreras

```sql
query = '''
    SELECT CONVERT(date, dropoff_datetime) AS date, COUNT(*) AS c
    FROM <schemaname>.<nyctaxi_sample>
    GROUP BY CONVERT(date, dropoff_datetime)
'''

pd.read_sql(query,conn)
```

#### <a name="exploration-trip-distribution-per-medallion"></a>Exploración: distribución de carreras por placa

```sql
query = '''
    SELECT medallion,count(*) AS c
    FROM <schemaname>.<nyctaxi_sample>
    GROUP BY medallion
'''

pd.read_sql(query,conn)
```

#### <a name="exploration-trip-distribution-by-medallion-and-hack-license"></a>Exploración: Distribución de carreras por medallion y hack_license

```sql
query = '''select medallion, hack_license,count(*) from <schemaname>.<nyctaxi_sample> group by medallion, hack_license'''
pd.read_sql(query,conn)
```

#### <a name="exploration-trip-time-distribution"></a>Exploración: distribución del tiempo de la carrera

```sql
query = '''select trip_time_in_secs, count(*) from <schemaname>.<nyctaxi_sample> group by trip_time_in_secs order by count(*) desc'''
pd.read_sql(query,conn)
```

#### <a name="exploration-trip-distance-distribution"></a>Exploración: distribución de la distancia de la carrera

```sql
query = '''select floor(trip_distance/5)*5 as tripbin, count(*) from <schemaname>.<nyctaxi_sample> group by floor(trip_distance/5)*5 order by count(*) desc'''
pd.read_sql(query,conn)
```

#### <a name="exploration-payment-type-distribution"></a>Exploración: distribución del tipo de pago

```sql
query = '''select payment_type,count(*) from <schemaname>.<nyctaxi_sample> group by payment_type'''
pd.read_sql(query,conn)
```

#### <a name="verify-the-final-form-of-the-featurized-table"></a>Comprobar el formulario final de la tabla con características

```sql
query = '''SELECT TOP 100 * FROM <schemaname>.<nyctaxi_sample>'''
pd.read_sql(query,conn)
```

## <a name="build-models-in-azure-machine-learning"></a><a name="mlmodel"></a>Creación de modelos en Azure Machine Learning
Ya está todo listo para pasar a la creación del modelo y la implementación del mismo en [Azure Machine Learning](https://studio.azureml.net). Los datos están listos para usarse en cualquiera de los problemas de predicción identificados anteriormente, a saber:

1. **Clasificación binaria**: permite predecir si se dio propina en una carrera o no.
2. **Clasificación multiclase**: permite predecir el rango de la propina dada, según las clases definidas anteriormente.
3. **Tarea de regresión**: Permite predecir el importe de la propina pagada por una carrera.

Para comenzar el ejercicio de modelado, inicie sesión en el área de trabajo de **Azure Machine Learning (clásico)** . Si aún no ha creado un área de trabajo de aprendizaje automático, consulte [Creación de un área de trabajo de Azure Machine Learning Studio (clásico)](../studio/create-workspace.md).

1. Para empezar a usar Azure Machine Learning, consulte [¿Qué es Microsoft Azure Machine Learning Studio (clásico)?](../studio/what-is-ml-studio.md)
2. Inicie sesión en [Azure Machine Learning Studio (clásico)](https://studio.azureml.net).
3. La página principal de Machine Learning Studio (clásico) ofrece una gran cantidad de información, así como vídeos, tutoriales, vínculos a referencias de módulos y otros recursos. Para obtener más información sobre Azure Machine Learning, visite el [Centro de documentación de Azure Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/).

Un experimento de entrenamiento típico consta de los pasos siguientes:

1. Crear un experimento **+NUEVO** .
2. Lleve los datos a Azure Machine Learning Studio (clásico).
3. Preprocesar, transformar y manipular los datos según sea necesario.
4. Generar características según sea necesario.
5. Dividir los datos en conjuntos de datos de entrenamiento, validación y pruebas (o disponer de conjuntos de datos independientes para cada uno).
6. Seleccionar uno o varios algoritmos de aprendizaje automático, según el problema de aprendizaje que se quiera resolver. Por ejemplo: clasificación binaria, clasificación multiclase, regresión.
7. Entrenar uno o varios modelos utilizando el conjunto de datos de entrenamiento.
8. Puntuar el conjunto de datos de validación con los modelos entrenados.
9. Evaluar los modelos para calcular las métricas relevantes para el problema de aprendizaje.
10. Ajustar los modelos y seleccionar el mejor para su implementación.

En este ejercicio, ya se han explorado y diseñado los datos en Azure Synapse Analytics, y también se ha decidido el tamaño de la muestra para la ingesta en Azure Machine Learning Studio (clásico). Este es el procedimiento para crear uno o varios de los modelos de predicción:

1. Proporcione los datos a Azure Machine Learning Studio (clásico) con el módulo de [importar datos][import-data], disponible en la sección **Entrada y salida de datos**. Para obtener más información, consulte la página de referencia sobre el módulo [Importar datos][import-data].

    ![Datos de importación de Aprendizaje automático de Azure][17]
2. Seleccionar **Azure SQL Database** como **Origen de datos** en el panel **Propiedades**.
3. Escribir el nombre DNS de la base de datos en el campo **Nombre del servidor de la base de datos** . Formato: `tcp:<your_virtual_machine_DNS_name>,1433`
4. Escribir el **nombre de la base de datos** en el campo correspondiente.
5. Escribir el *nombre de usuario de SQL* en **Nombre de la cuenta de usuario del servidor** y la *contraseña* en **Contraseña de la cuenta de usuario del servidor**.
7. En el área de texto editable **Consulta de base de datos**, pegar la consulta que extrae los campos de la base de datos necesarios (incluidos los campos calculados, como las etiquetas) y reducir la muestra al tamaño de muestra deseado.

En la ilustración siguiente se muestra un ejemplo de un experimento de clasificación binaria que lee datos directamente desde la base de datos de Azure Synapse Analytics (no olvide reemplazar los nombres de tabla nyctaxi_trip y nyctaxi_fare por el nombre de esquema y los nombres de tabla que utilizó en el tutorial). Se pueden construir experimentos similares para problemas de clasificación multiclase y de regresión.

![Entrenamiento de Aprendizaje automático de Azure][10]

> [!IMPORTANT]
> En los ejemplos de consultas de extracción y muestreo de datos de modelado de las secciones anteriores, **las etiquetas de los tres ejercicios de modelado se incluyen en la consulta**. Un paso importante (requerido) en cada uno de los ejercicios de modelado consiste en **excluir** las etiquetas innecesarias de los otros dos problemas y cualquier otra **fuga de destino**. Por ejemplo, cuando use clasificación binaria, utilice la etiqueta **tipped** y excluya los campos **tip\_class**, **tip\_amount** y **total\_amount**. Estos últimos son fugas de destino ya que implican que se pagó propina.
>
> Para excluir cualquier columna innecesaria o fugas de destino, puede usar el módulo [Seleccionar columnas de conjunto de datos][select-columns] o [Editar metadatos][edit-metadata]. Para obtener más información, consulte las páginas de referencia de [Seleccionar columnas de conjunto de datos][select-columns] y [Editar metadatos][edit-metadata].
>
>

## <a name="deploy-models-in-azure-machine-learning"></a><a name="mldeploy"></a>Implementación de modelos en Azure Machine Learning
Cuando el modelo esté listo, podrá implementarlo fácilmente como un servicio web directamente desde el experimento. Para obtener más información sobre la implementación de servicios web de Azure Machine Learning, vea [Implementación de un servicio web de Azure Machine Learning](../studio/deploy-a-machine-learning-web-service.md).

Para implementar un nuevo servicio web, deberá:

1. Crear un experimento de puntuación.
2. Implemente el servicio web.

Para crear un experimento de puntuación a partir de un experimento de entrenamiento **Finalizado**, haga clic en **CREAR EXPERIMENTO DE PUNTUACIÓN** en la barra de acciones inferior.

![Puntuación de Azure][18]

Azure Machine Learning intentará crear un experimento de puntuación en función de los componentes del experimento de entrenamiento. En concreto, hará lo siguiente:

1. Guardar el modelo entrenado y quitar los módulos de entrenamiento del modelo.
2. Identificar un **puerto de entrada** lógico que represente el esquema de datos de entrada esperado.
3. Identificar un **puerto de salida** lógico que represente el esquema de salida del servicio web.

Cuando se crea el experimento de puntuación, revise los resultados y ajústelo según sea necesario. Un ajuste común consiste en reemplazar la consulta o el conjunto de datos de entrada por uno que excluya los campos de etiqueta, ya que estos campos de etiqueta no se asignarán al esquema cuando se llame al servicio. También es una procedimiento recomendable reducir el tamaño de la consulta o del conjunto de datos de entrada a unos pocos registros, los suficientes para indicar el esquema de entrada. En el caso del puerto de salida, es habitual excluir todos los campos de entrada e incluir solo las **etiquetas puntuadas** y las **probabilidades puntuadas** en la salida mediante el módulo [Seleccionar columnas de conjunto de datos][select-columns].

En la ilustración siguiente se muestra un ejemplo de experimento de puntuación. Cuando todo esté listo para implementar, haga clic en el botón **PUBLICAR SERVICIO WEB** de la barra de acciones inferior.

![Publicación de Aprendizaje automático de Azure][11]

## <a name="summary"></a>Resumen
A modo de recapitulación, en este tutorial paso a paso se ha creado un entorno de ciencia de datos de Azure, se ha trabajado con un conjunto de datos público grande de principio a fin, llevándolo a través del proceso de ciencia de datos en equipos, desde la adquisición de los datos al entrenamiento del modelo, para finalizar con la implementación de un servicio web de Azure Machine Learning.

### <a name="license-information"></a>Información de licencia
Microsoft comparte este tutorial de ejemplo y sus scripts adjuntos y Blocs de notas de IPython bajo la licencia MIT. Vea el archivo LICENSE.txt que se encuentra en el directorio del código de ejemplo en GitHub para obtener más detalles.

## <a name="references"></a>Referencias
- [Página de descarga de NYC Taxi Trips de Andrés Monroy](https://www.andresmh.com/nyctaxitrips/)
- [FOILing NYC's Taxi Trip Data de Chris Whong](https://chriswhong.com/open-data/foil_nyc_taxi/)
- [Estadísticas e investigación de la Comisión de taxis y limusinas de la Ciudad de Nueva York](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

[1]: ./media/sqldw-walkthrough/sql-walkthrough_26_1.png
[2]: ./media/sqldw-walkthrough/sql-walkthrough_28_1.png
[3]: ./media/sqldw-walkthrough/sql-walkthrough_35_1.png
[4]: ./media/sqldw-walkthrough/sql-walkthrough_36_1.png
[5]: ./media/sqldw-walkthrough/sql-walkthrough_39_1.png
[6]: ./media/sqldw-walkthrough/sql-walkthrough_42_1.png
[7]: ./media/sqldw-walkthrough/sql-walkthrough_44_1.png
[8]: ./media/sqldw-walkthrough/sql-walkthrough_46_1.png
[9]: ./media/sqldw-walkthrough/sql-walkthrough_71_1.png
[10]: ./media/sqldw-walkthrough/azuremltrain.png
[11]: ./media/sqldw-walkthrough/azuremlpublish.png
[12]: ./media/sqldw-walkthrough/ssmsconnect.png
[13]: ./media/sqldw-walkthrough/executescript.png
[14]: ./media/sqldw-walkthrough/sqlserverproperties.png
[15]: ./media/sqldw-walkthrough/sqldefaultdirs.png
[16]: ./media/sqldw-walkthrough/bulkimport.png
[17]: ./media/sqldw-walkthrough/amlreader.png
[18]: ./media/sqldw-walkthrough/amlscoring.png
[19]: ./media/sqldw-walkthrough/ps_download_scripts.png
[20]: ./media/sqldw-walkthrough/ps_load_data.png
[21]: ./media/sqldw-walkthrough/azcopy-overwrite.png
[22]: ./media/sqldw-walkthrough/ipnb-service-aml-1.png
[23]: ./media/sqldw-walkthrough/ipnb-service-aml-2.png
[24]: ./media/sqldw-walkthrough/ipnb-service-aml-3.png
[25]: ./media/sqldw-walkthrough/ipnb-service-aml-4.png
[26]: ./media/sqldw-walkthrough/tip_class_hist_1.png


<!-- Module References -->
[edit-metadata]: https://msdn.microsoft.com/library/azure/370b6676-c11c-486f-bf73-35349f842a66/
[select-columns]: https://msdn.microsoft.com/library/azure/1ec722fa-b623-4e26-a44e-a50c6d726223/
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
