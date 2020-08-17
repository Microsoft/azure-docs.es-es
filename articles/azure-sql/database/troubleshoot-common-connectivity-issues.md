---
title: Cómo trabajar con errores transitorios
description: Obtenga información sobre cómo solucionar problemas, diagnosticar y evitar un error de conexión de SQL o un error transitorio al conectarse a Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse Analytics.
keywords: conexión de sql, cadena de conexión, problemas de conectividad, error transitorio, error de conexión
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: dalechen
ms.author: ninarn
ms.reviewer: carlrab, vanto
ms.date: 01/14/2020
ms.openlocfilehash: 0b28fa788e7b35e94482104d807c228db21f49b4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87003923"
---
# <a name="troubleshoot-transient-connection-errors-in-sql-database-and-sql-managed-instance"></a>Solución de problemas de errores de conexión transitorios en SQL Database e Instancia administrada de SQL

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

En este artículo se describe cómo evitar, solucionar, diagnosticar y mitigar los errores de conexión y los errores transitorios que la aplicación cliente encuentra cuando interactúa con Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse Analytics. Aprenda a configurar la lógica de reintento, generar la cadena de conexión y ajustar otros valores de conexión.

<a id="i-transient-faults" name="i-transient-faults"></a>

## <a name="transient-errors-transient-faults"></a>Errores transitorios

Un error transitorio tiene una causa subyacente que pronto se solucionará automáticamente. Una causa ocasional de errores transitorios se produce cuando el sistema de Azure rápidamente desplaza recursos de hardware para equilibrar mejor la carga de varias cargas de trabajo. La mayoría de estos eventos de reconfiguración se completan en menos de 60 segundos. Durante este período de tiempo de reconfiguración, es posible que tenga problemas de conexión con la base de datos de SQL Database. Las aplicaciones que se conectan a la base de datos deberían crearse de modo que contemplen esos errores transitorios. Para controlarlos, implemente una lógica de reintento en el código en lugar de mostrarlas a los usuarios como errores de aplicación.

Si su programa cliente utiliza ADO.NET, se notifican al programa los errores transitorios a través del inicio de una excepción **SqlException**.

<a id="connection-versus-command" name="connection-versus-command"></a>

### <a name="connection-vs-command"></a>Conexión frente a comandos

Vuelva a intentar la conexión de SQL Database e Instancia administrada de SQL o establézcala de nuevo, dependiendo de lo siguiente:

- **Se produce un error transitorio durante un intento de conexión**

Después de un retraso de varios segundos, vuelva a intentar la conexión.

- **Se produce un error transitorio durante un comando de consulta de SQL Database Instancia administrada de SQL**

No vuelva a intentarlo inmediatamente. Por el contrario, después de un retraso, establezca de nuevo la conexión. A continuación, vuelva a intentar el comando.

<a id="j-retry-logic-transient-faults" name="j-retry-logic-transient-faults"></a>

## <a name="retry-logic-for-transient-errors"></a>Lógica de reintento para errores transitorios.

Los programas cliente que encuentran ocasionalmente un error transitorio son más sólidos cuando contienen lógica de reintento. Si el programa se comunica con la base de datos de SQL Database a través de un middleware de otros fabricantes, consulte con el proveedor si el middleware contiene lógica de reintento para errores transitorios.

<a id="principles-for-retry" name="principles-for-retry"></a>

### <a name="principles-for-retry"></a>Principios de reintento

- Si el error es transitorio, intente volver a abrir una conexión.
- No vuelva a intentar directamente una instrucción `SELECT` de SQL Database o Instancia administrada de SQL que produjo un error transitorio. En su lugar, establezca una conexión nueva y, después, vuelva a intentar la instrucción `SELECT`.
- Cuando una instrucción `UPDATE` de SQL Database o Instancia administrada de SQL presenta un error transitorio, establezca una conexión nueva antes de volver a intentar dicha instrucción. La lógica de reintento debe asegurar que se completó toda la transacción de base de datos o que toda la transacción se revirtió.

### <a name="other-considerations-for-retry"></a>Otras consideraciones para el reintento

- Un programa por lotes que se inicia automáticamente después del horario de trabajo y que se completará antes de mañana puede permitirse el transcurso de largos intervalos de tiempo entre los reintentos.
- Un programa de la interfaz de usuario debe tener en cuenta la tendencia humana de abandonar tras una espera demasiado larga. La solución no debe ser repetir el reintento cada pocos segundos porque dicha directiva puede saturar el sistema con solicitudes.

### <a name="interval-increase-between-retries"></a>Aumento del intervalo entre reintentos

Se recomienda que espere 5 segundos antes del primer reintento. Si se vuelve a intentar después de un retraso menor de 5 segundos, se correrá el riesgo de sobrecargar el servicio en la nube. Para cada intento siguiente, el retraso debería aumentar exponencialmente, hasta un máximo de 60 segundos.

Para obtener una explicación del período de bloqueo para clientes que usan ADO.NET, vea [Agrupación de conexiones de SQL Server (ADO.NET)](https://msdn.microsoft.com/library/8xx3tyca.aspx).

También puede establecer un número máximo de reintentos antes de que el programa se cierre automáticamente.

### <a name="code-samples-with-retry-logic"></a>Ejemplos de código con lógica de reintento

Algunos ejemplos de código con la lógica de reintento se encuentran disponibles en:

- [Conexión resistente a Azure SQL con ADO.NET][step-4-connect-resiliently-to-sql-with-ado-net-a78n]
- [Conexión resistente a Azure SQL con PHP][step-4-connect-resiliently-to-sql-with-php-p42h]

<a id="k-test-retry-logic" name="k-test-retry-logic"></a>

### <a name="test-your-retry-logic"></a>Probar su lógica de reintento

Para probar la lógica de reintento, debe simular o producir un error que se pueda corregir mientras aún se ejecuta el programa.

#### <a name="test-by-disconnecting-from-the-network"></a>Prueba mediante la desconexión de la red

Una forma de probar la lógica de reintento es desconectar el equipo cliente de la red mientras se ejecuta el programa. El error es:

- **SqlException.Number** = 11001
- Mensaje: "Host desconocido"

Como parte del primer reintento, puede volver a conectar el equipo cliente a la red y, después, intentar conectarlo.

Para llevar a cabo esta prueba, desconecte el equipo de la red antes de iniciar el programa. El programa reconoce entonces un parámetro de tiempo de ejecución que hace que el programa:

- Agregue temporalmente 11001 a la lista de errores para considerarlo como transitorio.
- Intente su primera conexión como de costumbre.
- Una vez capturado el error, quite 11001 de la lista.
- Muestre un mensaje que indica al usuario que conecte el equipo a la red.
- Pause la ejecución mediante el método **Console.ReadLine** o un cuadro de diálogo con un botón Aceptar. El usuario presiona la tecla Entrar después de que se conecte el equipo a la red.
- Nuevo intento de conexión, se espera una realización correcta.

#### <a name="test-by-misspelling-the-user-name-when-connecting"></a>Realización de la prueba escribiendo mal el nombre del usuario al conectarse

El programa deliberadamente escribe mal el nombre de usuario antes del primer intento de conexión. El error es:

- **SqlException.Number** = 18456
- Mensaje: "Error de inicio de sesión para el usuario 'WRONG_MyUserName'."

Como parte del primer reintento, el programa puede corregir el error ortográfico y, a continuación, intentar conectarse.

Para llevar a cabo esta prueba, el programa reconocerá un parámetro de tiempo de ejecución que hará que el programa:

- Agregue temporalmente 18456 a la lista de errores para considerarlo como transitorio.
- Agregue deliberadamente 'WRONG_' al nombre de usuario.
- Una vez capturado el error, quite 18456 de la lista.
- Quite 'WRONG_' del nombre de usuario.
- Nuevo intento de conexión, se espera una realización correcta.

<a id="net-sqlconnection-parameters-for-connection-retry" name="net-sqlconnection-parameters-for-connection-retry"></a>

## <a name="net-sqlconnection-parameters-for-connection-retry"></a>Parámetros .NET SqlConnection para reintento de conexión

Si el programa cliente se conecta a la base de datos de SQL Database mediante la clase **System.Data.SqlClient.SqlConnection** de .NET Framework, use .NET 4.6.1 o una versión posterior, o .NET Core, para poder utilizar la característica de reintento de conexión. Para obtener más información sobre esta característica, consulte [esta página web](https://docs.microsoft.com/dotnet/api/system.data.sqlclient.sqlconnection).

<!--
2015-11-30, FwLink 393996 points to dn632678.aspx, which links to a downloadable .docx related to SqlClient and SQL Server 2014.
-->

Cuando cree la [cadena de conexión](https://msdn.microsoft.com/library/System.Data.SqlClient.SqlConnection.connectionstring.aspx) para su objeto **SqlConnection**, coordine los valores entre los parámetros siguientes:

- **ConnectRetryCount**:&nbsp;&nbsp;El valor predeterminado es 1. El intervalo es de 0 a 255.
- **ConnectRetryInterval**:&nbsp;&nbsp;El valor predeterminado es 10 segundos. El intervalo es de 1 a 60.
- **Connection Timeout**:&nbsp;&nbsp;El valor predeterminado es 15 segundos. El intervalo es de 0 a 2147483647.

Específicamente, los valores elegidos deben cumplir la siguiente igualdad: Connection Timeout = ConnectRetryCount * ConnectionRetryInterval

Por ejemplo, si el recuento es igual a 3 y el intervalo es igual a 10 segundos, un tiempo de espera de solo 29 segundos no proporcionará al sistema tiempo suficiente para su tercer y último reintento de conexión: 29 < 3 * 10.

<a id="connection-versus-command" name="connection-versus-command"></a>

## <a name="connection-vs-command"></a>Conexión frente a comandos

Los parámetros **ConnectRetryCount** y **ConnectRetryInterval** permiten al objeto **SqlConnection** volver a intentar la operación de conexión sin indicarlo al programa o sin alterarlo, como una devolución de control al programa. Los reintentos pueden producirse en las situaciones siguientes:

- Llamada al método SqlConnection.Open
- Llamada al método SqlConnection.Execute

Hay algo muy sutil que tener en cuenta. Si se produce un error transitorio mientras su *consulta* se está ejecutando, el objeto **SqlConnection** no vuelve a intentar la operación de conexión. Sin duda, no vuelve a intentar la consulta. Sin embargo, **SqlConnection** comprueba muy rápidamente la conexión antes de enviar la consulta para su ejecución. Si la comprobación rápida detecta un problema de conexión, **SqlConnection** vuelve a intentar la operación de conexión. Si el reintento se realiza correctamente, se envía la consulta para su ejecución.

### <a name="should-connectretrycount-be-combined-with-application-retry-logic"></a>¿Se debe combinar ConnectRetryCount con lógica de reintento de la aplicación?

Supongamos que su aplicación tiene una lógica de reintento personalizada. Puede reintentar la operación de conexión cuatro veces. Si agrega **ConnectRetryInterval** y **ConnectRetryCount** = 3 a la cadena de conexión, aumentará el número de reintentos a 4 * 3 = 12 reintentos. Es posible que no desee un gran número de reintentos.

<a id="a-connection-connection-string" name="a-connection-connection-string"></a>

## <a name="connections-to-your-database-in-sql-database"></a>Conexiones a la base de datos en SQL Database

<a id="c-connection-string" name="c-connection-string"></a>

### <a name="connection-connection-string"></a>Conexión: Cadena de conexión

La cadena de conexión necesaria para conectarse a la base de datos es ligeramente diferente de la cadena que se utiliza para conectarse a SQL Server. Puede copiar la cadena de conexión para la base de datos en [Azure Portal](https://portal.azure.com/).

[!INCLUDE [sql-database-include-connection-string-20-portalshots](../../../includes/sql-database-include-connection-string-20-portalshots.md)]

<a id="b-connection-ip-address" name="b-connection-ip-address"></a>

### <a name="connection-ip-address"></a>Conexión: Dirección IP

Debe configurar SQL Database para que acepte la comunicación desde la dirección IP del equipo que hospeda el programa cliente. Para establecer esta configuración, modifique la configuración del firewall a través de [Azure Portal](https://portal.azure.com/).

Si olvida configurar la dirección IP, el programa fallará con un mensaje de error que indica la dirección IP necesaria.

[!INCLUDE [sql-database-include-ip-address-22-portal](../../../includes/sql-database-include-ip-address-22-v12portal.md)]

Para más información, vea [Configuración del firewall en SQL Database](firewall-configure.md).
<a id="c-connection-ports" name="c-connection-ports"></a>

### <a name="connection-ports"></a>Conexión: Puertos

Por lo general, debe asegurarse de que solo el puerto 1433 está abierto para la comunicación saliente en el equipo que hospeda el programa cliente.

Por ejemplo, cuando el programa cliente está hospedado en un equipo con Windows, puede usar Firewall de Windows en el host para abrir el puerto 1433.

1. Abra el Panel de control.
2. Seleccione **Todos los elementos de Panel de control** > **Firewall de Windows** > **Configuración avanzada** > **Reglas de salida** > **Acción** > **Nueva regla**.

Si el programa cliente se hospeda en una máquina virtual (VM) de Azure, lea [Puertos más allá del 1433 para ADO.NET 4.5 y SQL Database](adonet-v12-develop-direct-route-ports.md).

Para obtener información general sobre la configuración de puertos y direcciones IP en la base de datos, vea [Firewall de Azure SQL Database](firewall-configure.md).

<a id="d-connection-ado-net-4-5" name="d-connection-ado-net-4-5"></a>

### <a name="connection-adonet-462-or-later"></a>Conexión: ADO.NET 4.6.2 o versiones posteriores

Si el programa usa clases ADO.NET como **System.Data.SqlClient.SqlConnection** para conectarse a SQL Database, le recomendamos que use .NET Framework 4.6.2, o cualquier versión posterior.

#### <a name="starting-with-adonet-462"></a>A partir de ADO.NET 4.6.2

- El intento de apertura de la conexión que se va a reintentar de inmediato para Azure SQL, lo que mejora el rendimiento de aplicaciones habilitadas para la nube.

#### <a name="starting-with-adonet-461"></a>A partir de ADO.NET 4.6.1

- Con SQL Database, se mejora la confiabilidad si abre una conexión con el método **SqlConnection.Open**. Ahora, el método **Open** incorpora los mejores mecanismos de reintento en respuesta a errores transitorios para determinados errores dentro del período de tiempo de espera de conexión.
- Se admite a agrupación de conexiones, que incluye una comprobación eficaz de que el objeto de conexión que ofrece el programa está funcionando.

Cuando se usa un objeto de conexión desde un grupo de conexiones, se recomienda que el programa cierre temporalmente la conexión cuando no se vaya a usar de inmediato. Volver a abrir una conexión no tiene un costo alto, pero sí lo tiene crear una nueva.

Si usa ADO.NET 4.0 o versiones anteriores, se recomienda que lo actualice a la versión de ADO.NET más reciente. Desde agosto de 2018 se puede [descargar ADO.NET 4.6.2](https://blogs.msdn.microsoft.com/dotnet/20../../announcing-the-net-framework-4-7-2/).

<a id="e-diagnostics-test-utilities-connect" name="e-diagnostics-test-utilities-connect"></a>

## <a name="diagnostics"></a>Diagnóstico

<a id="d-test-whether-utilities-can-connect" name="d-test-whether-utilities-can-connect"></a>

### <a name="diagnostics-test-whether-utilities-can-connect"></a>Diagnóstico: Probar si las utilidades pueden conectarse

Si el programa no puede conectarse a la base de datos en SQL Database, una opción de diagnóstico es intentar conectarse con una utilidad. Idealmente, la utilidad se conecta mediante la misma biblioteca que usa el programa.

En cualquier equipo con Windows, puede probar estas utilidades:

- SQL Server Management Studio (ssms.exe), que se conecta mediante ADO.NET
- `sqlcmd.exe`, que se conecta mediante [ODBC](https://msdn.microsoft.com/library/jj730308.aspx)

Una el programa se conecte, compruebe que funciona una breve consulta SELECT de SQL.

<a id="f-diagnostics-check-open-ports" name="f-diagnostics-check-open-ports"></a>

### <a name="diagnostics-check-the-open-ports"></a>Diagnóstico: Comprobar los puertos abiertos

Si cree que los intentos de conexión no pueden realizarse debido a problemas en los puertos, puede ejecutar una utilidad en el equipo que le informa sobre las configuraciones de los puertos.

En Linux, las utilidades siguientes pueden resultar útiles:

- `netstat -nap`
- `nmap -sS -O 127.0.0.1`: Cambie el valor de ejemplo para que sea su dirección IP.

En Windows, la utilidad [PortQry.exe](https://www.microsoft.com/download/details.aspx?id=17148) puede resultar útil. Esta es una ejecución de ejemplo que consultó la situación del puerto en una base de datos de SQL Database y que se ejecutó en un equipo portátil:

```cmd
[C:\Users\johndoe\]
>> portqry.exe -n johndoesvr9.database.windows.net -p tcp -e 1433

Querying target system called: johndoesvr9.database.windows.net

Attempting to resolve name to IP address...
Name resolved to 23.100.117.95

querying...
TCP port 1433 (ms-sql-s service): LISTENING

[C:\Users\johndoe\]
>>
```

<a id="g-diagnostics-log-your-errors" name="g-diagnostics-log-your-errors"></a>

### <a name="diagnostics-log-your-errors"></a>Diagnóstico: Registrar los errores

A veces un problema intermitente se diagnostica mejor mediante la detección de un patrón general durante varios días o semanas.

El cliente puede ayudar al diagnóstico mediante el registro de todos los errores que encuentra. Puede correlacionar las entradas del registro con los datos de error que SQL Database registra internamente.

Enterprise Library 6 (EntLib60) ofrece clases administradas de .NET para ayudar con el registro. Para más información, vea [5 - El procedimiento más sencillo: uso del bloque de aplicación de registro](https://msdn.microsoft.com/library/dn440731.aspx).

<a id="h-diagnostics-examine-logs-errors" name="h-diagnostics-examine-logs-errors"></a>

### <a name="diagnostics-examine-system-logs-for-errors"></a>Diagnóstico: Examinar los registros del sistema en busca de errores

Presentamos algunas instrucciones SELECT de Transact-SQL que consultan los registros de error y otra información.

| Consulta de un registro | Descripción |
|:--- |:--- |
| `SELECT e.*`<br/>`FROM sys.event_log AS e`<br/>`WHERE e.database_name = 'myDbName'`<br/>`AND e.event_category = 'connectivity'`<br/>`AND 2 >= DateDiff`<br/>&nbsp;&nbsp;`(hour, e.end_time, GetUtcDate())`<br/>`ORDER BY e.event_category,`<br/>&nbsp;&nbsp;`e.event_type, e.end_time;` |La vista [sys.event_log](https://msdn.microsoft.com/library/dn270018.aspx) proporciona información acerca de eventos individuales, que incluye algunos que pueden causar errores transitorios o errores de conectividad.<br/><br/>Idealmente, puede poner en correlación los valores **start_time** o **end_time** con información acerca de cuándo ha tenido problemas su programa cliente.<br/><br/>Tiene que conectarse a la base de datos *maestra* para ejecutar esta consulta. |
| `SELECT c.*`<br/>`FROM sys.database_connection_stats AS c`<br/>`WHERE c.database_name = 'myDbName'`<br/>`AND 24 >= DateDiff`<br/>&nbsp;&nbsp;`(hour, c.end_time, GetUtcDate())`<br/>`ORDER BY c.end_time;` |La vista [sys.database_connection_stats](https://msdn.microsoft.com/library/dn269986.aspx) ofrece recuentos agregados de los tipos de eventos para realizar diagnósticos adicionales.<br/><br/>Tiene que conectarse a la base de datos *maestra* para ejecutar esta consulta. |

<a id="d-search-for-problem-events-in-the-sql-database-log" name="d-search-for-problem-events-in-the-sql-database-log"></a>

### <a name="diagnostics-search-for-problem-events-in-the-sql-database-log"></a>Diagnóstico: Buscar eventos de problema en el registro de SQL Database

Puede buscar entradas sobre los eventos de problema en el registro de SQL Database. Pruebe la siguiente instrucción SELECT de Transact-SQL en la base de datos *principal* :

```sql
SELECT
   object_name
  ,CAST(f.event_data as XML).value
      ('(/event/@timestamp)[1]', 'datetime2')                      AS [timestamp]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="error"]/value)[1]', 'int')             AS [error]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="state"]/value)[1]', 'int')             AS [state]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="is_success"]/value)[1]', 'bit')        AS [is_success]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="database_name"]/value)[1]', 'sysname') AS [database_name]
FROM
  sys.fn_xe_telemetry_blob_target_read_file('el', null, null, null) AS f
WHERE
  object_name != 'login_event'  -- Login events are numerous.
  and
  '2015-06-21' < CAST(f.event_data as XML).value
        ('(/event/@timestamp)[1]', 'datetime2')
ORDER BY
  [timestamp] DESC
;
```

#### <a name="a-few-returned-rows-from-sysfn_xe_telemetry_blob_target_read_file"></a>Algunas filas devueltas de sys.fn_xe_telemetry_blob_target_read_file

En el ejemplo siguiente se muestra el aspecto que podría tener una fila devuelta. Los valores nulos que se muestran no suelen ser nulos en las demás filas.

```
object_name                   timestamp                    error  state  is_success  database_name

database_xml_deadlock_report  2015-10-16 20:28:01.0090000  NULL   NULL   NULL        AdventureWorks
```

<a id="l-enterprise-library-6" name="l-enterprise-library-6"></a>

## <a name="enterprise-library-6"></a>Enterprise Library 6

Enterprise Library 6 (EntLib60) es un marco de clases de .NET que ayuda a implementar a clientes sólidos de servicios en la nube, uno de los cuales es SQL Database. Para buscar temas dedicados a cada área en la que puede ayudar EntLib60 en, vea [Enterprise Library 6 - April 2013](https://msdn.microsoft.com/library/dn169621%28v=pandp.60%29.aspx) (Enterprise Library 6: abril de 2013).

La lógica de reintento para controlar los errores transitorios es un área en la que puede ayudar EntLib60. Para más información, vea [4: Perseverancia, el secreto de todos los triunfos: uso del bloque de aplicación de control de errores transitorios](https://msdn.microsoft.com/library/dn440719%28v=pandp.60%29.aspx).

> [!NOTE]
> El código fuente de EntLib60 está disponible de forma pública para descargarlo desde el [Centro de descarga](https://go.microsoft.com/fwlink/p/?LinkID=290898). Microsoft no tiene previsto realizar más actualizaciones de mantenimiento o de característica en EntLib.

<a id="entlib60-classes-for-transient-errors-and-retry" name="entlib60-classes-for-transient-errors-and-retry"></a>

### <a name="entlib60-classes-for-transient-errors-and-retry"></a>Clases de EntLib60 para errores transitorios y reintentos

Las siguientes clases de EntLib60 son especialmente útiles para la lógica de reintento. Todas esas clases se encuentran en el espacio de nombres **Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling**.

En el espacio de nombres **Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling**:

- **RetryPolicy**
  - **ExecuteAction**
- **ExponentialBackoff**
- **SqlDatabaseTransientErrorDetectionStrategy**
- **ReliableSqlConnection**
  - **ExecuteCommand**

En el espacio de nombres **Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling.TestSupport**:

- **AlwaysTransientErrorDetectionStrategy**
- **NeverTransientErrorDetectionStrategy**

Estos son algunos vínculos a información sobre EntLib60:

- Descarga gratis del libro: [Developer's Guide to Microsoft Enterprise Library, 2nd Edition](https://www.microsoft.com/download/details.aspx?id=41145) (Guía del desarrollador para Microsoft Enterprise Library, segunda edición).
- Procedimientos recomendados: [Orientación general sobre reintentos](/azure/architecture/best-practices/transient-faults) tiene una excelente explicación detallada de la lógica de reintento.
- Descarga de NuGet: [Enterprise Library: Bloque de aplicación de control de errores transitorios 6.0](https://www.nuget.org/packages/EnterpriseLibrary.TransientFaultHandling/).

<a id="entlib60-the-logging-block" name="entlib60-the-logging-block"></a>

### <a name="entlib60-the-logging-block"></a>EntLib60: El bloque de registro

- El bloque de registro es una solución muy flexible y configurable que puede usar para:
  - Crear y almacenar los mensajes de registro en una amplia variedad de ubicaciones.
  - Clasificar y filtrar los mensajes.
  - Recopilar la información contextual es útil para la depuración y el seguimiento, así como para los requisitos de registro generales y de auditoría.
- El bloque de registro abstrae la funcionalidad de registro desde el destino de registro para que el código de la aplicación sea coherente, con independencia de la ubicación y del tipo de almacén de registro de destino.

Para más información, vea [5 - El procedimiento más sencillo: uso del bloque de aplicación de registro](https://msdn.microsoft.com/library/dn440731%28v=pandp.60%29.aspx).

<a id="entlib60-istransient-method-source-code" name="entlib60-istransient-method-source-code"></a>

### <a name="entlib60-istransient-method-source-code"></a>Código fuente del método IsTransient de EntLib60

A continuación, en la clase **SqlDatabaseTransientErrorDetectionStrategy**, se encuentra el código fuente de C# para el método **IsTransient**. Desde abril de 2013, el código fuente explica qué errores se consideran transitorios y dignos de reintento.

```csharp
public bool IsTransient(Exception ex)
{
  if (ex != null)
  {
    SqlException sqlException;
    if ((sqlException = ex as SqlException) != null)
    {
      // Enumerate through all errors found in the exception.
      foreach (SqlError err in sqlException.Errors)
      {
        switch (err.Number)
        {
            // SQL Error Code: 40501
            // The service is currently busy. Retry the request after 10 seconds.
            // Code: (reason code to be decoded).
          case ThrottlingCondition.ThrottlingErrorNumber:
            // Decode the reason code from the error message to
            // determine the grounds for throttling.
            var condition = ThrottlingCondition.FromError(err);

            // Attach the decoded values as additional attributes to
            // the original SQL exception.
            sqlException.Data[condition.ThrottlingMode.GetType().Name] =
              condition.ThrottlingMode.ToString();
            sqlException.Data[condition.GetType().Name] = condition;

            return true;

          case 10928:
          case 10929:
          case 10053:
          case 10054:
          case 10060:
          case 40197:
          case 40540:
          case 40613:
          case 40143:
          case 233:
          case 64:
            // DBNETLIB Error Code: 20
            // The instance of SQL Server you attempted to connect to
            // does not support encryption.
          case (int)ProcessNetLibErrorCode.EncryptionNotSupported:
            return true;
        }
      }
    }
    else if (ex is TimeoutException)
    {
      return true;
    }
    else
    {
      EntityException entityException;
      if ((entityException = ex as EntityException) != null)
      {
        return this.IsTransient(entityException.InnerException);
      }
    }
  }

  return false;
}
```

## <a name="next-steps"></a>Pasos siguientes

- [Bibliotecas de conexiones para SQL Database y SQL Server](connect-query-content-reference-guide.md#libraries)
- [Agrupación de conexiones (ADO.NET)](https://docs.microsoft.com/dotnet/framework/data/adonet/sql-server-connection-pooling)
- [*Retrying* es una biblioteca de reintentos de uso general con licencia de Apache 2.0, escrita en Python,](https://pypi.python.org/pypi/retrying) para simplificar la tarea de agregar comportamiento de reintento a prácticamente todo.

<!-- Link references. -->

[step-4-connect-resiliently-to-sql-with-ado-net-a78n]: https://docs.microsoft.com/sql/connect/ado-net/step-4-connect-resiliently-sql-ado-net

[step-4-connect-resiliently-to-sql-with-php-p42h]: https://docs.microsoft.com/sql/connect/php/step-4-connect-resiliently-to-sql-with-php
