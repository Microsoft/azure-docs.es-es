---
title: Guía de programación de U-SQL para Azure Data Lake
description: Obtenga más información sobre el conjunto de servicios de Azure Data Lake Analytics que le permiten crear una plataforma de macrodatos basada en la nube.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: 63be271e-7c44-4d19-9897-c2913ee9599d
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: 1c22aa9fb91b0a86704b95586afc1779023e85b6
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288951"
---
# <a name="u-sql-programmability-guide"></a>Guía de programación de U-SQL

U-SQL es un lenguaje de consulta diseñado para tipo de macrodatos de cargas de trabajo. Una de las características exclusivas de U-SQL es la combinación del lenguaje declarativo similar a SQL con la extensibilidad y capacidad de programación que proporciona C#. En esta guía, nos centramos en la extensibilidad y capacidad de programación del lenguaje U-SQL que proporciona C#.

## <a name="requirements"></a>Requisitos

Descarga e instalación de [Herramientas de Azure Data Lake para Visual Studio](https://www.microsoft.com/download/details.aspx?id=49504).

## <a name="get-started-with-u-sql"></a>Introducción a U-SQL  

Observe el siguiente script U-SQL:

```usql
@a  = 
  SELECT * FROM 
    (VALUES
       ("Contoso",   1500.0, "2017-03-39"),
       ("Woodgrove", 2700.0, "2017-04-10")
    ) AS D( customer, amount, date );

@results =
  SELECT
    customer,
    amount,
    date
  FROM @a;    
```

Este script define dos conjuntos de filas: `@a` y `@results`. El conjunto de filas `@results` se define desde `@a`.

## <a name="c-types-and-expressions-in-u-sql-script"></a>Expresiones y tipos de C# en un script U-SQL

Una expresión U-SQL es una expresión de C# combinada con operaciones lógicas de U-SQL como `AND`, `OR` y `NOT`. Las expresiones U-SQL pueden utilizarse con SELECT, EXTRACT, WHERE, HAVING, GROUP BY y DECLARE. Por ejemplo, el siguiente script analiza una cadena como un valor de DateTime.

```usql
@results =
  SELECT
    customer,
    amount,
    DateTime.Parse(date) AS date
  FROM @a;    
```

El siguiente fragmento de código analiza una cadena de un valor de DateTime en la cláusula DECLARE.

```usql
DECLARE @d = DateTime.Parse("2016/01/01");
```

### <a name="use-c-expressions-for-data-type-conversions"></a>Uso de expresiones de C# para conversiones de tipo de datos

En el ejemplo siguiente se muestra cómo se puede hacer una conversión de datos de fecha y hora mediante el uso de expresiones de C#. En este escenario en particular, los datos de fecha y hora de cadena se convierten en la fecha y hora estándar con la notación de hora de medianoche 00:00:00.

```usql
DECLARE @dt = "2016-07-06 10:23:15";

@rs1 =
  SELECT 
    Convert.ToDateTime(Convert.ToDateTime(@dt).ToString("yyyy-MM-dd")) AS dt,
    dt AS olddt
  FROM @rs0;

OUTPUT @rs1 
  TO @output_file 
  USING Outputters.Text();
```

### <a name="use-c-expressions-for-todays-date"></a>Uso de expresiones de C# para la fecha de hoy

Para extraer la fecha de hoy, podemos usar la siguiente expresión de C#: `DateTime.Now.ToString("M/d/yyyy")`

Este es un ejemplo de cómo utilizar esta expresión en un script:

```usql
@rs1 =
  SELECT
    MAX(guid) AS start_id,
    MIN(dt) AS start_time,
    MIN(Convert.ToDateTime(Convert.ToDateTime(dt<@default_dt?@default_dt:dt).ToString("yyyy-MM-dd"))) AS start_zero_time,
    MIN(USQL_Programmability.CustomFunctions.GetFiscalPeriod(dt)) AS start_fiscalperiod,
    DateTime.Now.ToString("M/d/yyyy") AS Nowdate,
    user,
    des
  FROM @rs0
  GROUP BY user, des;
```
## <a name="using-net-assemblies"></a>Uso de los ensamblados .NET

El modelo de extensibilidad de U-SQL depende en gran medida de la capacidad de agregar código personalizado a partir de ensamblados .NET. 

### <a name="register-a-net-assembly"></a>Registrar un ensamblado de .NET

Utilice la instrucción `CREATE ASSEMBLY` para colocar un ensamblado .NET en una base de datos U-SQL. Posteriormente, los scripts de U-SQL pueden usar esos ensamblados mediante la instrucción `REFERENCE ASSEMBLY`. 

El código siguiente muestra cómo registrar un ensamblado:

```usql
CREATE ASSEMBLY MyDB.[MyAssembly]
   FROM "/myassembly.dll";
```

El código siguiente muestra cómo hacer referencia a un ensamblado:

```usql
REFERENCE ASSEMBLY MyDB.[MyAssembly];
```

Consulte las [instrucciones de registro de un ensamblado](https://blogs.msdn.microsoft.com/azuredatalake/2016/08/26/how-to-register-u-sql-assemblies-in-your-u-sql-catalog/) que tratan este tema con mayor detalle.


### <a name="use-assembly-versioning"></a>Uso del control de versiones de ensamblado
Actualmente, U-SQL usa la versión 4.7.2 de .NET°Framework. Por lo tanto, asegúrese de que sus propios ensamblados son compatibles con esa versión del tiempo de ejecución.

Como se mencionó anteriormente, U-SQL ejecuta código en un formato de 64 bits (x64). Por lo tanto, asegúrese de que el código se compila para ejecutarse en x64. De lo contrario, se produce el error de formato incorrecto mostrado anteriormente.

El archivo DLL y el de recursos de cada ensamblado cargado como, por ejemplo, un tiempo de ejecución diferente, un ensamblado nativo o un archivo de configuración pueden tener 400 MB como máximo. El tamaño total de recursos implementados, ya sea mediante DEPLOY RESOURCE o mediante referencias a ensamblados y a sus archivos adicionales, no puede superar los 3 GB.

Por último, tenga en cuenta que cada base de datos de U-SQL solo puede contener una versión de cualquier ensamblado dado. Por ejemplo, si necesita tanto la versión 7 como la versión 8 de la biblioteca NewtonSoft Json.NET, debe registrarlos en dos bases de datos diferentes. Además, cada script solo puede hacer referencia a una versión de un archivo DLL de ensamblado dado. En este sentido, U-SQL sigue la semántica de control de versiones y la administración de ensamblados de C#.

## <a name="use-user-defined-functions-udf"></a>Uso de funciones definidas por el usuario: UDF
Las funciones definidas por el usuario de U-SQL o UDF son rutinas programadas que aceptan parámetros, realizan una acción (como un cálculo complejo) y devuelven el resultado de esa acción como un valor. El valor devuelto de UDF solo puede ser un escalar único. Se puede llamar a UDF de U-SQL en un script base U-SQL como a cualquier otra función escalar de C#.

Es recomendable que inicialice las funciones definidas por el usuario de U-SQL como **públicas** y **estáticas**.

```usql
public static string MyFunction(string param1)
{
    return "my result";
}
```

Primero veamos el ejemplo sencillo de creación de una UDF.

En este escenario de caso de uso, es necesario determinar el período fiscal, incluido el trimestre fiscal y el mes fiscal del primer inicio de sesión del usuario específico. El primer mes fiscal del año en nuestro escenario es junio.

Para calcular el período fiscal, se introduce la siguiente función de C#:

```usql
public static string GetFiscalPeriod(DateTime dt)
{
    int FiscalMonth=0;
    if (dt.Month < 7)
    {
    FiscalMonth = dt.Month + 6;
    }
    else
    {
    FiscalMonth = dt.Month - 6;
    }

    int FiscalQuarter=0;
    if (FiscalMonth >=1 && FiscalMonth<=3)
    {
    FiscalQuarter = 1;
    }
    if (FiscalMonth >= 4 && FiscalMonth <= 6)
    {
    FiscalQuarter = 2;
    }
    if (FiscalMonth >= 7 && FiscalMonth <= 9)
    {
    FiscalQuarter = 3;
    }
    if (FiscalMonth >= 10 && FiscalMonth <= 12)
    {
    FiscalQuarter = 4;
    }

    return "Q" + FiscalQuarter.ToString() + ":P" + FiscalMonth.ToString();
}
```

Simplemente calcula el trimestre y mes fiscal y se devuelve un valor de cadena. Para junio (primer mes del primer trimestre fiscal) se usa "Q1:P1". Para julio: "Q1:P2" y así sucesivamente.

Se trata de una función de C# que vamos a usar en nuestro proyecto U-SQL.

Este es el aspecto de la sección de código subyacente en este escenario:

```usql
using Microsoft.Analytics.Interfaces;
using Microsoft.Analytics.Types.Sql;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace USQL_Programmability
{
    public class CustomFunctions
    {
        public static string GetFiscalPeriod(DateTime dt)
        {
            int FiscalMonth=0;
            if (dt.Month < 7)
            {
                FiscalMonth = dt.Month + 6;
            }
            else
            {
                FiscalMonth = dt.Month - 6;
            }

            int FiscalQuarter=0;
            if (FiscalMonth >=1 && FiscalMonth<=3)
            {
                FiscalQuarter = 1;
            }
            if (FiscalMonth >= 4 && FiscalMonth <= 6)
            {
                FiscalQuarter = 2;
            }
            if (FiscalMonth >= 7 && FiscalMonth <= 9)
            {
                FiscalQuarter = 3;
            }
            if (FiscalMonth >= 10 && FiscalMonth <= 12)
            {
                FiscalQuarter = 4;
            }

            return "Q" + FiscalQuarter.ToString() + ":" + FiscalMonth.ToString();
        }
    }
}
```

Ahora vamos a llamar a esta función desde el script base U-SQL. Para ello, se debe proporcionar un nombre completo de la función, incluido el espacio de nombres que, en este caso, es EspacioDeNombres.Clase.Función(parámetro).
```usql
USQL_Programmability.CustomFunctions.GetFiscalPeriod(dt)
```

El siguiente es el script base U-SQL real:
```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid Guid,
        dt DateTime,
            user String,
            des String
    FROM @input_file USING Extractors.Tsv();

DECLARE @default_dt DateTime = Convert.ToDateTime("06/01/2016");

@rs1 =
    SELECT
        MAX(guid) AS start_id,
    MIN(dt) AS start_time,
        MIN(Convert.ToDateTime(Convert.ToDateTime(dt<@default_dt?@default_dt:dt).ToString("yyyy-MM-dd"))) AS start_zero_time,
        MIN(USQL_Programmability.CustomFunctions.GetFiscalPeriod(dt)) AS start_fiscalperiod,
        user,
        des
    FROM @rs0
    GROUP BY user, des;

OUTPUT @rs1 
    TO @output_file 
    USING Outputters.Text();
```

El siguiente es el archivo de salida de la ejecución del script:

```output
0d8b9630-d5ca-11e5-8329-251efa3a2941,2016-02-11T07:04:17.2630000-08:00,2016-06-01T00:00:00.0000000,"Q3:8","User1",""

20843640-d771-11e5-b87b-8b7265c75a44,2016-02-11T07:04:17.2630000-08:00,2016-06-01T00:00:00.0000000,"Q3:8","User2",""

301f23d2-d690-11e5-9a98-4b4f60a1836f,2016-02-11T09:01:33.9720000-08:00,2016-06-01T00:00:00.0000000,"Q3:8","User3",""
```

Este ejemplo muestra un uso simple de UDF insertada en U-SQL.

### <a name="keep-state-between-udf-invocations"></a>Mantenimiento del estado entre las invocaciones de UDF
Los objetos de programación de C# de U-SQL pueden ser más sofisticados utilizando interactividad a través de variables globales de código subyacente. Veamos el siguiente escenario de caso de uso empresarial.

En organizaciones grandes, los usuarios pueden cambiar entre variedades de aplicaciones internas. Entre ellas, se encuentran Microsoft Dynamics CRM, Power BI, etc. Es posible que los clientes deseen aplicar un análisis de telemetría de cómo los usuarios cambian entre distintas aplicaciones, cuáles son las tendencias de uso, etc. El objetivo para la empresa es optimizar el uso de las aplicaciones. Es posible que también quieran combinar aplicaciones diferentes o rutinas de inicio de sesión específicas.

Para lograr este objetivo, tenemos que determinar identificadores de sesión y retardo de tiempo entre las últimas sesiones que han tenido lugar.

Es necesario encontrar un inicio de sesión anterior y luego asignar este inicio de sesión a todas las sesiones que se generan en la misma aplicación. El primer desafío es que el script base U-SQL no nos permitirá aplicar cálculos sobre una columna ya calculada con la función LAG. El segundo reto es que se debe mantener la sesión específica para todas las sesiones dentro del mismo período de tiempo.

Para solucionar este problema, vamos a usar una variable global dentro de una sección de código subyacente: `static public string globalSession;`.

Esta variable global se aplica a todo el conjunto de filas durante nuestra ejecución de script.

Esta es la sección de código subyacente de nuestro programa U-SQL:

```csharp
using Microsoft.Analytics.Interfaces;
using Microsoft.Analytics.Types.Sql;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace USQLApplication21
{
    public class UserSession
    {
        static public string globalSession;
        static public string StampUserSession(string eventTime, string PreviousRow, string Session)
        {

            if (!string.IsNullOrEmpty(PreviousRow))
            {
                double timeGap = Convert.ToDateTime(eventTime).Subtract(Convert.ToDateTime(PreviousRow)).TotalMinutes;
                if (timeGap <= 60) {return Session;}
                else {return Guid.NewGuid().ToString();}
            }
            else {return Guid.NewGuid().ToString();}

        }

        static public string getStampUserSession(string Session)
        {
            if (Session != globalSession && !string.IsNullOrEmpty(Session)) { globalSession = Session; }
            return globalSession;
        }

    }
}
```

Este ejemplo muestra la variable global `static public string globalSession;` que se usa dentro de la función `getStampUserSession` y se reinicializa cada vez que cambia el parámetro Session.

El script base de U-SQL es como sigue:

```usql
DECLARE @in string = @"\UserSession\test1.tsv";
DECLARE @out1 string = @"\UserSession\Out1.csv";
DECLARE @out2 string = @"\UserSession\Out2.csv";
DECLARE @out3 string = @"\UserSession\Out3.csv";

@records =
    EXTRACT DataId string,
            EventDateTime string,           
            UserName string,
            UserSessionTimestamp string

    FROM @in
    USING Extractors.Tsv();

@rs1 =
    SELECT 
        EventDateTime,
        UserName,
    LAG(EventDateTime, 1) 
        OVER(PARTITION BY UserName ORDER BY EventDateTime ASC) AS prevDateTime,          
        string.IsNullOrEmpty(LAG(EventDateTime, 1) 
        OVER(PARTITION BY UserName ORDER BY EventDateTime ASC)) AS Flag,           
        USQLApplication21.UserSession.StampUserSession
           (
            EventDateTime,
            LAG(EventDateTime, 1) OVER(PARTITION BY UserName ORDER BY EventDateTime ASC),
            LAG(UserSessionTimestamp, 1) OVER(PARTITION BY UserName ORDER BY EventDateTime ASC)
           ) AS UserSessionTimestamp
    FROM @records;

@rs2 =
    SELECT 
        EventDateTime,
        UserName,
        LAG(EventDateTime, 1) 
        OVER(PARTITION BY UserName ORDER BY EventDateTime ASC) AS prevDateTime,
        string.IsNullOrEmpty( LAG(EventDateTime, 1) OVER(PARTITION BY UserName ORDER BY EventDateTime ASC)) AS Flag,
        USQLApplication21.UserSession.getStampUserSession(UserSessionTimestamp) AS UserSessionTimestamp
    FROM @rs1
    WHERE UserName != "UserName";

OUTPUT @rs2
    TO @out2
    ORDER BY UserName, EventDateTime ASC
    USING Outputters.Csv();
```

Se llama a la función `USQLApplication21.UserSession.getStampUserSession(UserSessionTimestamp)` aquí durante el segundo cálculo del conjunto de filas de memoria. Pasa la columna `UserSessionTimestamp` y devuelve el valor hasta que `UserSessionTimestamp` se cambia.

El archivo de salida tiene el siguiente aspecto:

```output
"2016-02-19T07:32:36.8420000-08:00","User1",,True,"72a0660e-22df-428e-b672-e0977007177f"
"2016-02-17T11:52:43.6350000-08:00","User2",,True,"4a0cd19a-6e67-4d95-a119-4eda590226ba"
"2016-02-17T11:59:08.8320000-08:00","User2","2016-02-17T11:52:43.6350000-08:00",False,"4a0cd19a-6e67-4d95-a119-4eda590226ba"
"2016-02-11T07:04:17.2630000-08:00","User3",,True,"51860a7a-1610-4f74-a9ea-69d5eef7cd9c"
"2016-02-11T07:10:33.9720000-08:00","User3","2016-02-11T07:04:17.2630000-08:00",False,"51860a7a-1610-4f74-a9ea-69d5eef7cd9c"
"2016-02-15T21:27:41.8210000-08:00","User3","2016-02-11T07:10:33.9720000-08:00",False,"4d2bc48d-bdf3-4591-a9c1-7b15ceb8e074"
"2016-02-16T05:48:49.6360000-08:00","User3","2016-02-15T21:27:41.8210000-08:00",False,"dd3006d0-2dcd-42d0-b3a2-bc03dd77c8b9"
"2016-02-16T06:22:43.6390000-08:00","User3","2016-02-16T05:48:49.6360000-08:00",False,"dd3006d0-2dcd-42d0-b3a2-bc03dd77c8b9"
"2016-02-17T16:29:53.2280000-08:00","User3","2016-02-16T06:22:43.6390000-08:00",False,"2fa899c7-eecf-4b1b-a8cd-30c5357b4f3a"
"2016-02-17T16:39:07.2430000-08:00","User3","2016-02-17T16:29:53.2280000-08:00",False,"2fa899c7-eecf-4b1b-a8cd-30c5357b4f3a"
"2016-02-17T17:20:39.3220000-08:00","User3","2016-02-17T16:39:07.2430000-08:00",False,"2fa899c7-eecf-4b1b-a8cd-30c5357b4f3a"
"2016-02-19T05:23:54.5710000-08:00","User3","2016-02-17T17:20:39.3220000-08:00",False,"6ca7ed80-c149-4c22-b24b-94ff5b0d824d"
"2016-02-19T05:48:37.7510000-08:00","User3","2016-02-19T05:23:54.5710000-08:00",False,"6ca7ed80-c149-4c22-b24b-94ff5b0d824d"
"2016-02-19T06:40:27.4830000-08:00","User3","2016-02-19T05:48:37.7510000-08:00",False,"6ca7ed80-c149-4c22-b24b-94ff5b0d824d"
"2016-02-19T07:27:37.7550000-08:00","User3","2016-02-19T06:40:27.4830000-08:00",False,"6ca7ed80-c149-4c22-b24b-94ff5b0d824d"
"2016-02-19T19:35:40.9450000-08:00","User3","2016-02-19T07:27:37.7550000-08:00",False,"3f385f0b-3e68-4456-ac74-ff6cef093674"
"2016-02-20T00:07:37.8250000-08:00","User3","2016-02-19T19:35:40.9450000-08:00",False,"685f76d5-ca48-4c58-b77d-bd3a9ddb33da"
"2016-02-11T09:01:33.9720000-08:00","User4",,True,"9f0cf696-c8ba-449a-8d5f-1ca6ed8f2ee8"
"2016-02-17T06:30:38.6210000-08:00","User4","2016-02-11T09:01:33.9720000-08:00",False,"8b11fd2a-01bf-4a5e-a9af-3c92c4e4382a"
"2016-02-17T22:15:26.4020000-08:00","User4","2016-02-17T06:30:38.6210000-08:00",False,"4e1cb707-3b5f-49c1-90c7-9b33b86ca1f4"
"2016-02-18T14:37:27.6560000-08:00","User4","2016-02-17T22:15:26.4020000-08:00",False,"f4e44400-e837-40ed-8dfd-2ea264d4e338"
"2016-02-19T01:20:31.4800000-08:00","User4","2016-02-18T14:37:27.6560000-08:00",False,"2136f4cf-7c7d-43c1-8ae2-08f4ad6a6e08"
```

Este ejemplo muestra un escenario de caso de uso más complicado en el que se usa una variable global dentro de la sección de código subyacente que se aplica al conjunto de filas de memoria completo.

## <a name="use-user-defined-types-udt"></a>Uso de tipos definidos por el usuario: UDT
Los tipos de definidos por el usuario o UDT constituyen otra característica de programación de U-SQL. UDT de U-SQL actúa como un tipo de definido por el usuario de C# normal. C# es un lenguaje fuertemente tipado que permite el uso de tipos integrados y definidos por el usuario personalizados.

U-SQL no puede serializar ni deserializar de forma implícita UDT arbitrarios mientras el UDT se pasa entre vértices de conjuntos de filas. Esto implica que el usuario debe proporcionar un formateador explícito mediante la interfaz de IFormatter. Esto proporcionará a U-SQL los métodos parar serializar o deserializar el UDT.

> [!NOTE]
> En la actualidad, los extractores y outputters integrados de U-SQL no pueden serializar ni deserializar los datos de UDT en archivos ni desde estos ni siquiera con el conjunto IFormatter. Por ello, cuando esté escribiendo datos UDT en un archivo con la instrucción OUTPUT o leyéndolos con un extractor, tiene que pasarlos como una cadena o matriz de bytes. A continuación, llama al código de serialización y deserialización explícitamente (es decir, al método ToString() del UDT). Por otra parte, los extractores y outputters definidos por el usuario pueden leer y escribir UDT.

Si se intenta utilizar UDT en EXTRACTOR o OUTPUTTER (fuera de SELECT anterior), como se muestra a continuación:

```usql
@rs1 =
    SELECT 
        MyNameSpace.Myfunction_Returning_UDT(filed1) AS myfield
    FROM @rs0;

OUTPUT @rs1 
    TO @output_file 
    USING Outputters.Text();
```

Recibirá el error siguiente:

```output
Error   1   E_CSC_USER_INVALIDTYPEINOUTPUTTER: Outputters.Text was used to output column myfield of type
MyNameSpace.Myfunction_Returning_UDT.

Description:

Outputters.Text only supports built-in types.

Resolution:

Implement a custom outputter that knows how to serialize this type, or call a serialization method on the type in
the preceding SELECT.   C:\Users\sergeypu\Documents\Visual Studio 2013\Projects\USQL-Programmability\
USQL-Programmability\Types.usql 52  1   USQL-Programmability
```

Para trabajar con UDT en outptutter tenemos que serializar en una cadena con el método ToString() o crear un outputter personalizado.

Actualmente los UDT no se pueden usar en GROUP BY. Si UDT se usa en GROUP BY, se produce el siguiente error:

```output
Error   1   E_CSC_USER_INVALIDTYPEINCLAUSE: GROUP BY doesn't support type MyNameSpace.Myfunction_Returning_UDT
for column myfield

Description:

GROUP BY doesn't support UDT or Complex types.

Resolution:

Add a SELECT statement where you can project a scalar column that you want to use with GROUP BY.
C:\Users\sergeypu\Documents\Visual Studio 2013\Projects\USQL-Programmability\USQL-Programmability\Types.usql
62  5   USQL-Programmability
```

Para definir un UDT, tenemos que:

* Agregue los siguientes espacios de nombres:

```csharp
using Microsoft.Analytics.Interfaces
using System.IO;
```

* Agregue `Microsoft.Analytics.Interfaces`, necesario para las interfaces UDT. También se podría necesitar `System.IO` para definir la interfaz de IFormatter.

* Defina un tipo definido por el usuario con el atributo SqlUserDefinedType.

**SqlUserDefinedType** se utiliza para marcar una definición de tipo en un ensamblado como un tipo definido por el usuario (UDT) en U-SQL. Las propiedades del atributo reflejan las características físicas del UDT. No se puede heredar esta clase.

SqlUserDefinedType es un atributo necesario para la definición de UDT.

El constructor de la clase:  

* SqlUserDefinedTypeAttribute (formateador de tipos)

* Formateador de tipos: parámetro necesario para definir un formateador UDT. En concreto, el tipo de interfaz `IFormatter` que se debe pasar aquí.

```csharp
[SqlUserDefinedType(typeof(MyTypeFormatter))]
public class MyType
{ … }
```

* El UDT típico también requerirá la definición de la interfaz IFormatter, tal como se muestra en el ejemplo siguiente:

```csharp
public class MyTypeFormatter : IFormatter<MyType>
{
    public void Serialize(MyType instance, IColumnWriter writer, ISerializationContext context)
    { … }

    public MyType Deserialize(IColumnReader reader, ISerializationContext context)
    { … }
}
```

La interfaz `IFormatter` serializa y deserializa un grafo de objetos con el tipo de raíz de \<typeparamref name="T">.

\<typeparam name="T">El tipo de raíz del grafo de objetos para serializar y deserializar.

* **Deserialize**: deserializa los datos en el flujo proporcionado y reconstituye el grafo de objetos.

* **Serialize**: serializa un objeto o un grafo de objetos con el directorio raíz especificado en el flujo proporcionado.

Instancia de `MyType`: instancia del tipo.  
Escritor `IColumnWriter`/lector `IColumnReader`: flujo de la columna subyacente.  
Contexto `ISerializationContext`: enumeración que define un conjunto de marcas que especifica el contexto de origen o destino para el flujo durante la serialización.

* **Intermediate**: especifica que el contexto de origen o destino no es un almacén persistente.

* **Persistence**: especifica que el contexto de origen o destino es un almacén persistente.

Como un tipo de C# normal, la definición de UDT de U-SQL puede incluir invalidaciones para los operadores como +/==/!=. Puede incluir también métodos estáticos. Por ejemplo, si vamos a usar este UDT como un parámetro para la función de agregado MIN de U-SQL, tenemos que definir la invalidación del operador <.

Anteriormente en esta guía se mostró un ejemplo para la identificación del período fiscal a partir de la fecha específica con el formato `Qn:Pn (Q1:P10)`. En el ejemplo siguiente se muestra cómo definir un tipo personalizado para los valores del período fiscal.

A continuación aparece un ejemplo de sección de código subyacente con interfaz IFormatter y UDT personalizados:

```csharp
[SqlUserDefinedType(typeof(FiscalPeriodFormatter))]
public struct FiscalPeriod
{
    public int Quarter { get; private set; }

    public int Month { get; private set; }

    public FiscalPeriod(int quarter, int month):this()
    {
    this.Quarter = quarter;
    this.Month = month;
    }

    public override bool Equals(object obj)
    {
    if (ReferenceEquals(null, obj))
    {
        return false;
    }

    return obj is FiscalPeriod && Equals((FiscalPeriod)obj);
    }

    public bool Equals(FiscalPeriod other)
    {
return this.Quarter.Equals(other.Quarter) && this.Month.Equals(other.Month);
    }

    public bool GreaterThan(FiscalPeriod other)
    {
return this.Quarter.CompareTo(other.Quarter) > 0 || this.Month.CompareTo(other.Month) > 0;
    }

    public bool LessThan(FiscalPeriod other)
    {
return this.Quarter.CompareTo(other.Quarter) < 0 || this.Month.CompareTo(other.Month) < 0;
    }

    public override int GetHashCode()
    {
    unchecked
    {
        return (this.Quarter.GetHashCode() * 397) ^ this.Month.GetHashCode();
    }
    }

    public static FiscalPeriod operator +(FiscalPeriod c1, FiscalPeriod c2)
    {
return new FiscalPeriod((c1.Quarter + c2.Quarter) > 4 ? (c1.Quarter + c2.Quarter)-4 : (c1.Quarter + c2.Quarter), (c1.Month + c2.Month) > 12 ? (c1.Month + c2.Month) - 12 : (c1.Month + c2.Month));
    }

    public static bool operator ==(FiscalPeriod c1, FiscalPeriod c2)
    {
    return c1.Equals(c2);
    }

    public static bool operator !=(FiscalPeriod c1, FiscalPeriod c2)
    {
    return !c1.Equals(c2);
    }
    public static bool operator >(FiscalPeriod c1, FiscalPeriod c2)
    {
    return c1.GreaterThan(c2);
    }
    public static bool operator <(FiscalPeriod c1, FiscalPeriod c2)
    {
    return c1.LessThan(c2);
    }
    public override string ToString()
    {
    return (String.Format("Q{0}:P{1}", this.Quarter, this.Month));
    }

}

public class FiscalPeriodFormatter : IFormatter<FiscalPeriod>
{
    public void Serialize(FiscalPeriod instance, IColumnWriter writer, ISerializationContext context)
    {
    using (var binaryWriter = new BinaryWriter(writer.BaseStream))
    {
        binaryWriter.Write(instance.Quarter);
        binaryWriter.Write(instance.Month);
        binaryWriter.Flush();
    }
    }

    public FiscalPeriod Deserialize(IColumnReader reader, ISerializationContext context)
    {
    using (var binaryReader = new BinaryReader(reader.BaseStream))
    {
var result = new FiscalPeriod(binaryReader.ReadInt16(), binaryReader.ReadInt16());
        return result;
    }
    }
}
```

El tipo definido incluye dos números: trimestre y mes. Los operadores `==/!=/>/<` y el método estático `ToString()` se definen aquí.

Como se mencionó anteriormente, UDT se puede utilizar en expresiones SELECT, pero no se puede usar en OUTPUTTER/EXTRACTOR sin serialización personalizada. Debe serializarse como una cadena con `ToString()` o usarse con OUTPUTTER/EXTRACTOR personalizado.

Ahora vamos a analizar el uso de UDT. En una sección de código subyacente, cambiamos nuestra función GetFiscalPeriod a la siguiente:

```csharp
public static FiscalPeriod GetFiscalPeriodWithCustomType(DateTime dt)
{
    int FiscalMonth = 0;
    if (dt.Month < 7)
    {
    FiscalMonth = dt.Month + 6;
    }
    else
    {
    FiscalMonth = dt.Month - 6;
    }

    int FiscalQuarter = 0;
    if (FiscalMonth >= 1 && FiscalMonth <= 3)
    {
    FiscalQuarter = 1;
    }
    if (FiscalMonth >= 4 && FiscalMonth <= 6)
    {
    FiscalQuarter = 2;
    }
    if (FiscalMonth >= 7 && FiscalMonth <= 9)
    {
    FiscalQuarter = 3;
    }
    if (FiscalMonth >= 10 && FiscalMonth <= 12)
    {
    FiscalQuarter = 4;
    }

    return new FiscalPeriod(FiscalQuarter, FiscalMonth);
}
```

Como puede ver, devuelve el valor de nuestro tipo FiscalPeriod.

Vea este ejemplo de cómo se usa más en el script base de U-SQL. En este ejemplo se muestran distintas maneras de invocación de UDT a partir del script U-SQL.

```usql
DECLARE @input_file string = @"c:\work\cosmos\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"c:\work\cosmos\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
        guid string,
        dt DateTime,
        user String,
        des String
    FROM @input_file USING Extractors.Tsv();

@rs1 =
    SELECT 
        guid AS start_id,
        dt,
        DateTime.Now.ToString("M/d/yyyy") AS Nowdate,
        USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt).Quarter AS fiscalquarter,
        USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt).Month AS fiscalmonth,
        USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt) + new USQL_Programmability.CustomFunctions.FiscalPeriod(1,7) AS fiscalperiod_adjusted,
        user,
        des
    FROM @rs0;

@rs2 =
    SELECT 
           start_id,
           dt,
           DateTime.Now.ToString("M/d/yyyy") AS Nowdate,
           fiscalquarter,
           fiscalmonth,
           USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt).ToString() AS fiscalperiod,

       // This user-defined type was created in the prior SELECT.  Passing the UDT to this subsequent SELECT would have failed if the UDT was not annotated with an IFormatter.
           fiscalperiod_adjusted.ToString() AS fiscalperiod_adjusted,
           user,
           des
    FROM @rs1;

OUTPUT @rs2 
    TO @output_file 
    USING Outputters.Text();
```

Este es un ejemplo de una sección de código subyacente completa:

```csharp
using Microsoft.Analytics.Interfaces;
using Microsoft.Analytics.Types.Sql;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;

namespace USQL_Programmability
{
    public class CustomFunctions
    {
        static public DateTime? ToDateTime(string dt)
        {
            DateTime dtValue;

            if (!DateTime.TryParse(dt, out dtValue))
                return Convert.ToDateTime(dt);
            else
                return null;
        }

        public static FiscalPeriod GetFiscalPeriodWithCustomType(DateTime dt)
        {
            int FiscalMonth = 0;
            if (dt.Month < 7)
            {
                FiscalMonth = dt.Month + 6;
            }
            else
            {
                FiscalMonth = dt.Month - 6;
            }

            int FiscalQuarter = 0;
            if (FiscalMonth >= 1 && FiscalMonth <= 3)
            {
                FiscalQuarter = 1;
            }
            if (FiscalMonth >= 4 && FiscalMonth <= 6)
            {
                FiscalQuarter = 2;
            }
            if (FiscalMonth >= 7 && FiscalMonth <= 9)
            {
                FiscalQuarter = 3;
            }
            if (FiscalMonth >= 10 && FiscalMonth <= 12)
            {
                FiscalQuarter = 4;
            }

            return new FiscalPeriod(FiscalQuarter, FiscalMonth);
        }        [SqlUserDefinedType(typeof(FiscalPeriodFormatter))]
        public struct FiscalPeriod
        {
            public int Quarter { get; private set; }

            public int Month { get; private set; }

            public FiscalPeriod(int quarter, int month):this()
            {
                this.Quarter = quarter;
                this.Month = month;
            }

            public override bool Equals(object obj)
            {
                if (ReferenceEquals(null, obj))
                {
                    return false;
                }

                return obj is FiscalPeriod && Equals((FiscalPeriod)obj);
            }

            public bool Equals(FiscalPeriod other)
            {
return this.Quarter.Equals(other.Quarter) &&    this.Month.Equals(other.Month);
            }

            public bool GreaterThan(FiscalPeriod other)
            {
return this.Quarter.CompareTo(other.Quarter) > 0 || this.Month.CompareTo(other.Month) > 0;
            }

            public bool LessThan(FiscalPeriod other)
            {
return this.Quarter.CompareTo(other.Quarter) < 0 || this.Month.CompareTo(other.Month) < 0;
            }

            public override int GetHashCode()
            {
                unchecked
                {
                    return (this.Quarter.GetHashCode() * 397) ^ this.Month.GetHashCode();
                }
            }

            public static FiscalPeriod operator +(FiscalPeriod c1, FiscalPeriod c2)
            {
return new FiscalPeriod((c1.Quarter + c2.Quarter) > 4 ? (c1.Quarter + c2.Quarter)-4 : (c1.Quarter + c2.Quarter), (c1.Month + c2.Month) > 12 ? (c1.Month + c2.Month) - 12 : (c1.Month + c2.Month));
            }

            public static bool operator ==(FiscalPeriod c1, FiscalPeriod c2)
            {
                return c1.Equals(c2);
            }

            public static bool operator !=(FiscalPeriod c1, FiscalPeriod c2)
            {
                return !c1.Equals(c2);
            }
            public static bool operator >(FiscalPeriod c1, FiscalPeriod c2)
            {
                return c1.GreaterThan(c2);
            }
            public static bool operator <(FiscalPeriod c1, FiscalPeriod c2)
            {
                return c1.LessThan(c2);
            }
            public override string ToString()
            {
                return (String.Format("Q{0}:P{1}", this.Quarter, this.Month));
            }

        }

        public class FiscalPeriodFormatter : IFormatter<FiscalPeriod>
        {
public void Serialize(FiscalPeriod instance, IColumnWriter writer, ISerializationContext context)
            {
                using (var binaryWriter = new BinaryWriter(writer.BaseStream))
                {
                    binaryWriter.Write(instance.Quarter);
                    binaryWriter.Write(instance.Month);
                    binaryWriter.Flush();
                }
            }

public FiscalPeriod Deserialize(IColumnReader reader, ISerializationContext context)
            {
                using (var binaryReader = new BinaryReader(reader.BaseStream))
                {
var result = new FiscalPeriod(binaryReader.ReadInt16(), binaryReader.ReadInt16());
                    return result;
                }
            }
        }
    }
}
```

## <a name="use-user-defined-aggregates-udagg"></a>Uso de agregados definidos por el usuario: UDAGG
Los agregados definidos por el usuario son todas las funciones relacionadas con la agregación que no se distribuyen de fábrica con U-SQL. El ejemplo puede ser un agregado para realizar un cálculo matemático personalizado, realizar concatenaciones de cadenas, manipulaciones con cadenas, etc.

La definición de clase base agregada definida por el usuario es la siguiente:

```csharp
    [SqlUserDefinedAggregate]
    public abstract class IAggregate<T1, T2, TResult> : IAggregate
    {
        protected IAggregate();

        public abstract void Accumulate(T1 t1, T2 t2);
        public abstract void Init();
        public abstract TResult Terminate();
    }
```

**SqlUserDefinedAggregate** indica que el tipo se debe registrar como un agregado definido por el usuario. No se puede heredar esta clase.

El atributo SqlUserDefinedType es **opcional** para la definición de UDAGG.


La clase base le permite pasar tres parámetros abstractos: dos de entrada y uno de resultado. Los tipos de datos son variables y se deben definirse durante la herencia de clases.

```csharp
public class GuidAggregate : IAggregate<string, string, string>
{
    string guid_agg;

    public override void Init()
    { … }

    public override void Accumulate(string guid, string user)
    { … }

    public override string Terminate()
    { … }
}
```

* **Init** se invoca una vez para cada grupo durante el cálculo. Proporciona la rutina de inicialización para cada grupo de agregación.  
* **Accumulate** se ejecuta una vez para cada valor. Proporciona la funcionalidad principal para el algoritmo de agregación. Puede utilizarse para agregar valores con varios tipos de datos definidos durante la herencia de clases. Puede aceptar dos parámetros de tipos de datos de variable.
* **Terminate** se ejecuta una vez por cada grupo de agregación al final del procesamiento para generar el resultado para cada grupo.

Para declarar tipos de datos de entrada y salida correctos, utilice la definición de clase de la siguiente manera:

```csharp
public abstract class IAggregate<T1, T2, TResult> : IAggregate
```

* T1: segundo parámetro para acumular.
* T2: Segundo parámetro para acumular.
* TResult: tipo de valor de terminación devuelto.

Por ejemplo:

```csharp
public class GuidAggregate : IAggregate<string, int, int>
```

or

```csharp
public class GuidAggregate : IAggregate<string, string, string>
```

### <a name="use-udagg-in-u-sql"></a>Uso de UDAGG en U-SQL
Para usar UDAGG, defínalo primero en código subyacente o haga referencia a él desde el archivo DLL de programación existente como se indicó anteriormente.

Después use la sintaxis siguiente:

```csharp
AGG<UDAGG_functionname>(param1,param2)
```

A continuación se muestra un ejemplo de UDAGG:

```csharp
public class GuidAggregate : IAggregate<string, string, string>
{
    string guid_agg;

    public override void Init()
    {
        guid_agg = "";
    }

    public override void Accumulate(string guid, string user)
    {
        if (user.ToUpper()== "USER1")
        {
        guid_agg += "{" + guid + "}";
        }
    }

    public override string Terminate()
    {
        return guid_agg;
    }

}
```

Y el script base U-SQL:

```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @" \usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid string,
        dt DateTime,
            user String,
            des String
    FROM @input_file 
    USING Extractors.Tsv();

@rs1 =
    SELECT
        user,
        AGG<USQL_Programmability.GuidAggregate>(guid,user) AS guid_list
    FROM @rs0
    GROUP BY user;

OUTPUT @rs1 TO @output_file USING Outputters.Text();
```

En este escenario de caso de uso, se concatenan los GUID de clase para los usuarios específicos.

## <a name="use-user-defined-objects-udo"></a>Uso de objetos definidos por el usuario: UDO
U-SQL le permite definir objetos de programación personalizados, que se denominan objetos definidos por el usuario o UDO.

A continuación aparece una lista de UDO en U-SQL:

* Extractores definidos por el usuario
    * Extraer fila por fila
    * Se usa para implementar la extracción de datos de archivos estructurados personalizados

* Outputters definidos por el usuario
    * Dar salida fila por fila
    * Se usa para la salida de tipos de datos personalizados o para formatos de archivo personalizados

* Procesadores definidos por el usuario
    * Tomar una fila y producir una fila
    * Se usa para reducir el número de columnas o producir nuevas columnas con valores derivados de un conjunto de columnas existente

* Aplicadores definidos por el usuario
    * Tomar una fila y producir de 0 a n filas
    * Se usa con CROSS/OUTER APPLY

* Combinadores definidos por el usuario
    * Combina conjuntos de filas (JOIN definidas por el usuario)

* Reductores definidos por el usuario
    * Tomar n filas y producir una fila
    * Se usa para reducir el número de filas

UDO normalmente se llama explícitamente en el script U-SQL como parte de las siguientes instrucciones U-SQL:

* EXTRACT
* OUTPUT
* PROCESS
* COMBINE
* REDUCE

> [!NOTE]  
> Los UDO están limitados a un consumo de 0,5 GB de memoria.  Esta limitación de memoria no se aplica a las ejecuciones locales.

## <a name="use-user-defined-extractors"></a>Uso de extractores definidos por el usuario
U-SQL permite importar datos externos utilizando una instrucción EXTRACT. La instrucción EXTRACT puede utilizar extractores UDO integrados:  

* *Extractors.Text()* : proporciona extracción de archivos de texto delimitado con diferentes codificaciones.

* *Extractors.Csv()* : proporciona extracción de archivos de valores separados por comas (CSV) con diferentes codificaciones.

* *Extractors.Tsv()* : proporciona extracción de archivos de valores separados por tabulaciones (TSV) con diferentes codificaciones.

Puede ser útil desarrollar un extractor personalizado. Esto puede resultar útil durante la importación de datos si se desea realizar alguna de las siguientes tareas:

* Modificar datos de entrada mediante la división de columnas y modificar valores individuales. La funcionalidad PROCESSOR es mejor para combinar las columnas.
* Analizar datos no estructurados, como páginas web o correos electrónicos o datos parcialmente no estructurados, como XML o JSON.
* Analizar datos en codificación no compatible.

Para definir un extractor definido por el usuario o UDE, necesitamos crear una interfaz `IExtractor`. Todos los parámetros de entrada al extractor, como delimitadores de columna o fila, codificación, etc., deben definirse en el constructor de la clase. La interfaz `IExtractor` también debe contener la definición de la invalidación `IEnumerable<IRow>` como se indica a continuación:

```csharp
[SqlUserDefinedExtractor]
public class SampleExtractor : IExtractor
{
     public SampleExtractor(string row_delimiter, char col_delimiter)
     { … }

     public override IEnumerable<IRow> Extract(IUnstructuredReader input, IUpdatableRow output)
     { … }
}
```

El atributo **SqlUserDefinedExtractor** indica que el tipo se debe registrar como un extractor definido por el usuario. No se puede heredar esta clase.

SqlUserDefinedExtractor es un atributo opcional para la definición de UDE. Se utiliza para definir la propiedad AtomicFileProcessing para el objeto UDE.

* bool     AtomicFileProcessing   

* **true** = Indica que este extractor requiere archivos de entrada atómicos (JSON, XML...)
* **false** = Indica que este extractor puede tratar con archivos divididos o distribuidos (CSV, SEQ...)

Los objetos de programación UDE principales son **input** y **output**. El objeto de entrada se utiliza para enumerar los datos de entrada como `IUnstructuredReader`. El objeto de salida se utiliza para establecer datos de salida como resultado de la actividad del extractor.

Se tiene acceso a los datos de entrada a través de `System.IO.Stream` y `System.IO.StreamReader`.

Para la enumeración de columnas de entrada, primero dividiremos el flujo de entrada mediante un delimitador de filas.

```csharp
foreach (Stream current in input.Split(my_row_delimiter))
{
…
}
```

Después, dividiremos aún más la fila de entrada en partes de la columna.

```csharp
foreach (Stream current in input.Split(my_row_delimiter))
{
…
    string[] parts = line.Split(my_column_delimiter);
    foreach (string part in parts)
    { … }
}
```

Para establecer datos de salida, usamos el método `output.Set`.

Es importante comprender que el extractor personalizado solo genera columnas y valores que se definen con la salida. Establezca la llamada del método.

```csharp
output.Set<string>(count, part);
```

La salida del extractor real se desencadena mediante una llamada a `yield return output.AsReadOnly();`.

Este es el ejemplo de extractor:

```csharp
[SqlUserDefinedExtractor(AtomicFileProcessing = true)]
public class FullDescriptionExtractor : IExtractor
{
     private Encoding _encoding;
     private byte[] _row_delim;
     private char _col_delim;

    public FullDescriptionExtractor(Encoding encoding, string row_delim = "\r\n", char col_delim = '\t')
    {
         this._encoding = ((encoding == null) ? Encoding.UTF8 : encoding);
         this._row_delim = this._encoding.GetBytes(row_delim);
         this._col_delim = col_delim;

    }

    public override IEnumerable<IRow> Extract(IUnstructuredReader input, IUpdatableRow output)
    {
         string line;
         //Read the input line by line
         foreach (Stream current in input.Split(_encoding.GetBytes("\r\n")))
         {
        using (System.IO.StreamReader streamReader = new StreamReader(current, this._encoding))
         {
             line = streamReader.ReadToEnd().Trim();
             //Split the input by the column delimiter
             string[] parts = line.Split(this._col_delim);
             int count = 0; // start with first column
             foreach (string part in parts)
             {
    if (count == 0)
             {  // for column “guid”, re-generated guid
                 Guid new_guid = Guid.NewGuid();
                 output.Set<Guid>(count, new_guid);
             }
             else if (count == 2)
             {
                 // for column “user”, convert to UPPER case
                 output.Set<string>(count, part.ToUpper());

             }
             else
             {
                 // keep the rest of the columns as-is
                 output.Set<string>(count, part);
             }
             count += 1;
             }

         }
         yield return output.AsReadOnly();
         }
         yield break;
     }
}
```

En este escenario de caso de uso, el extractor vuelve a generar un GUID para la columna “guid” y convierte los valores de la columna “user” en mayúsculas. Los extractores personalizados pueden producir resultados más complicados mediante el análisis de datos de entrada y la manipulación de estos.

Este es un script base U-SQL que utiliza un extractor personalizado:

```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid Guid,
        dt String,
            user String,
            des String
    FROM @input_file
        USING new USQL_Programmability.FullDescriptionExtractor(Encoding.UTF8);

OUTPUT @rs0 TO @output_file USING Outputters.Text();
```

## <a name="use-user-defined-outputters"></a>Uso de outputters definidos por el usuario
Outputter definido por el usuario es otro UDO de U-SQL que permite extender una funcionalidad integrada de U-SQL. Al igual que el extractor, hay varios outputters integrados.

* *Outputters.Text()* : escribe datos en archivos de texto delimitado con diferentes codificaciones.
* *Outputters.Csv()* : escribe datos en archivos de valores separados por comas (CSV) con diferentes codificaciones.
* *Outputters.Tsv()* : escribe datos en archivos de valores separados por tabulaciones (TSV) con diferentes codificaciones.

El outputter personalizado le permite escribir datos en un formato definido personalizado. Esto puede resultar útil para las siguientes tareas:

* Escribir datos en archivos semiestructurados o no estructurados.
* Escribir datos en codificaciones no admitidas.
* Modificar datos de salida o agregar atributos personalizados.

Para definir el outputter definido por el usuario, necesitamos crear la interfaz `IOutputter`.

Esta es la implementación de la clase base `IOutputter`:

```csharp
public abstract class IOutputter : IUserDefinedOperator
{
    protected IOutputter();

    public virtual void Close();
    public abstract void Output(IRow input, IUnstructuredWriter output);
}
```

Todos los parámetros de entrada al outputter, como delimitadores de columna o fila, codificación, etc., deben definirse en el constructor de la clase. La interfaz `IOutputter` también debe contener una definición de la invalidación `void Output`. El atributo `[SqlUserDefinedOutputter(AtomicFileProcessing = true)`, opcionalmente, se puede establecer para el procesamiento de archivos atómico. Para más información, consulte los siguientes detalles.

```csharp
[SqlUserDefinedOutputter(AtomicFileProcessing = true)]
public class MyOutputter : IOutputter
{

    public MyOutputter(myparam1, myparam2)
    {
      …
    }

    public override void Close()
    {
      …
    }

    public override void Output(IRow row, IUnstructuredWriter output)
    {
      …
    }
}
```

* `Output` se llama para cada fila de entrada. Devuelve el conjunto de filas `IUnstructuredWriter output`.
* La clase Constructor se utiliza para pasar parámetros al outputter definido por el usuario.
* `Close` se usa para invalidar opcionalmente para liberar el estado costoso o saber cuándo se ha escrito la última fila.

El atributo **SqlUserDefinedOutputter** indica que el tipo se debe registrar como un outputter definido por el usuario. No se puede heredar esta clase.

SqlUserDefinedOutputter es un atributo opcional para la definición del outputter definido por el usuario. Se utiliza para definir la propiedad AtomicFileProcessing.

* bool     AtomicFileProcessing   

* **true** = Indica que este outputter requiere archivos de salida atómicos (JSON, XML...)
* **false** = Indica que este outputter puede tratar con archivos divididos o distribuidos (CSV, SEQ...)

Los objetos de programación principales son **row** y **output**. El objeto **fila** se usa para enumerar los datos de salida como la interfaz `IRow`. **Salida** se usa para establecer los datos de salida al archivo de destino.

Se tiene acceso a los datos de entrada a través de la interfaz `IRow`. Los datos de salida pasan una fila en cada momento.

Los valores individuales se enumeran mediante una llamada al método Get de la interfaz IRow:

```csharp
row.Get<string>("column_name")
```

Los nombres de columna individuales se pueden determinar mediante una llamada a `row.Schema`:

```csharp
ISchema schema = row.Schema;
var col = schema[i];
string val = row.Get<string>(col.Name)
```

Este enfoque permite crear un outputter flexible para cualquier esquema de metadatos.

Los datos de salida se escriben en el archivo mediante `System.IO.StreamWriter`. El parámetro de flujo se establece en `output.BaseStream` como parte de `IUnstructuredWriter output`.

Tenga en cuenta que esto es importante para vaciar el búfer de datos en el archivo después de cada iteración de fila. Además, el objeto `StreamWriter` debe utilizarse con el atributo Disposable habilitado (valor predeterminado) y con la palabra clave **using**:

```csharp
using (StreamWriter streamWriter = new StreamWriter(output.BaseStream, this._encoding))
{
…
}
```

De lo contrario, llame al método Flush() explícitamente después de cada iteración. Esto se muestra en el ejemplo siguiente.

### <a name="set-headers-and-footers-for-user-defined-outputter"></a>Establecimiento de encabezados y pies de página para el outputter definido por el usuario
Para establecer un encabezado, use el flujo de ejecución de iteración único.

```csharp
public override void Output(IRow row, IUnstructuredWriter output)
{
 …
if (isHeaderRow)
{
    …                
}

 …
if (isHeaderRow)
{
    isHeaderRow = false;
}
 …
}
}
```

El código del primer bloque `if (isHeaderRow)` se ejecuta solo una vez.

Para el pie de página, use la referencia a la instancia del objeto `System.IO.Stream` (`output.BaseStream`). Escriba el pie de página en el método Close() de la interfaz `IOutputter`.  Para más información, consulte el siguiente ejemplo.

Este es un ejemplo de un outputter definido por el usuario:

```csharp
[SqlUserDefinedOutputter(AtomicFileProcessing = true)]
public class HTMLOutputter : IOutputter
{
    // Local variables initialization
    private string row_delimiter;
    private char col_delimiter;
    private bool isHeaderRow;
    private Encoding encoding;
    private bool IsTableHeader = true;
    private Stream g_writer;

    // Parameters definition            
    public HTMLOutputter(bool isHeader = false, Encoding encoding = null)
    {
    this.isHeaderRow = isHeader;
    this.encoding = ((encoding == null) ? Encoding.UTF8 : encoding);
    }

    // The Close method is used to write the footer to the file. It's executed only once, after all rows
    public override void Close()
    {
    //Reference to IO.Stream object - g_writer
    StreamWriter streamWriter = new StreamWriter(g_writer, this.encoding);
    streamWriter.Write("</table>");
    streamWriter.Flush();
    streamWriter.Close();
    }

    public override void Output(IRow row, IUnstructuredWriter output)
    {
    System.IO.StreamWriter streamWriter = new StreamWriter(output.BaseStream, this.encoding);

    // Metadata schema initialization to enumerate column names
    ISchema schema = row.Schema;

    // This is a data-independent header--HTML table definition
    if (IsTableHeader)
    {
        streamWriter.Write("<table border=1>");
        IsTableHeader = false;
    }

    // HTML table attributes
    string header_wrapper_on = "<th>";
    string header_wrapper_off = "</th>";
    string data_wrapper_on = "<td>";
    string data_wrapper_off = "</td>";

    // Header row output--runs only once
    if (isHeaderRow)
    {
        streamWriter.Write("<tr>");
        for (int i = 0; i < schema.Count(); i++)
        {
        var col = schema[i];
        streamWriter.Write(header_wrapper_on + col.Name + header_wrapper_off);
        }
        streamWriter.Write("</tr>");
    }

    // Data row output
    streamWriter.Write("<tr>");                
    for (int i = 0; i < schema.Count(); i++)
    {
        var col = schema[i];
        string val = "";
        try
        {
        // Data type enumeration--required to match the distinct list of types from OUTPUT statement
        switch (col.Type.Name.ToString().ToLower())
        {
            case "string": val = row.Get<string>(col.Name).ToString(); break;
            case "guid": val = row.Get<Guid>(col.Name).ToString(); break;
            default: break;
        }
        }
        // Handling NULL values--keeping them empty
        catch (System.NullReferenceException)
        {
        }
        streamWriter.Write(data_wrapper_on + val + data_wrapper_off);
    }
    streamWriter.Write("</tr>");

    if (isHeaderRow)
    {
        isHeaderRow = false;
    }
    // Reference to the instance of the IO.Stream object for footer generation
    g_writer = output.BaseStream;
    streamWriter.Flush();
    }
}

// Define the factory classes
public static class Factory
{
    public static HTMLOutputter HTMLOutputter(bool isHeader = false, Encoding encoding = null)
    {
    return new HTMLOutputter(isHeader, encoding);
    }
}
```

Y el script base U-SQL:

```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.html";

@rs0 =
    EXTRACT
            guid Guid,
        dt String,
            user String,
            des String
         FROM @input_file
         USING new USQL_Programmability.FullDescriptionExtractor(Encoding.UTF8);

OUTPUT @rs0 
    TO @output_file 
    USING new USQL_Programmability.HTMLOutputter(isHeader: true);
```

Se trata de un outputter HTML que crea un archivo HTML con los datos de la tabla.

### <a name="call-outputter-from-u-sql-base-script"></a>Llamada a un outputter desde el script base U-SQL
Para llamar a un outputter personalizado desde el script base U-SQL, se debe crear la nueva instancia del objeto outputter.

```sql
OUTPUT @rs0 TO @output_file USING new USQL_Programmability.HTMLOutputter(isHeader: true);
```

Para evitar la creación de una instancia del objeto en el script base, podemos crear un contenedor de función como se indicó en el ejemplo anterior:

```csharp
        // Define the factory classes
        public static class Factory
        {
            public static HTMLOutputter HTMLOutputter(bool isHeader = false, Encoding encoding = null)
            {
                return new HTMLOutputter(isHeader, encoding);
            }
        }
```

En este caso, la llamada original es similar a la siguiente:

```usql
OUTPUT @rs0 
TO @output_file 
USING USQL_Programmability.Factory.HTMLOutputter(isHeader: true);
```

## <a name="use-user-defined-processors"></a>Uso de procesadores definidos por el usuario
El procesador definido por el usuario o UDP es un tipo de UDO de U-SQL que permite procesar las filas entrantes mediante la aplicación de características de programación. UDP permite combinar columnas, modificar valores y agregar nuevas columnas si es necesario. Básicamente le ayuda a procesar un conjunto de filas para generar elementos de datos necesarios.

Para definir un UDP, necesitamos crear una interfaz `IProcessor` con el atributo `SqlUserDefinedProcessor`, que es opcional para UDP.

Esta interfaz debe contener la definición de la invalidación del conjunto de filas de interfaz `IRow`, como se muestra en el ejemplo siguiente:

```csharp
[SqlUserDefinedProcessor]
public class MyProcessor: IProcessor
{
public override IRow Process(IRow input, IUpdatableRow output)
 {
        …
 }
}
```

**SqlUserDefinedProcessor** indica que el tipo se debe registrar como un procesador definido por el usuario. No se puede heredar esta clase.

El atributo SqlUserDefinedProcessor es **opcional** para la definición de UDP.

Los objetos de programación principales son **input** y **output**. El objeto Input se utiliza para enumerar las columnas de entrada y Output para establecer datos de salida como resultado de la actividad del procesador.

Para la enumeración de las columnas de entrada, usamos el método `input.Get`.

```csharp
string column_name = input.Get<string>("column_name");
```

El parámetro para el método `input.Get` es una columna que se pasa como parte de la cláusula `PRODUCE` de la instrucción `PROCESS` del script base U-SQL. Es necesario usar aquí el tipo de datos correcto.

Para la salida, use el método `output.Set`.

Es importante comprender que el productor personalizado solo genera columnas y valores que se definen con la llamada al método `output.Set`.

```csharp
output.Set<string>("mycolumn", mycolumn);
```

La salida del procesador real se desencadena mediante una llamada a `return output.AsReadOnly();`.

Este es un ejemplo de procesador:

```csharp
[SqlUserDefinedProcessor]
public class FullDescriptionProcessor : IProcessor
{
public override IRow Process(IRow input, IUpdatableRow output)
 {
     string user = input.Get<string>("user");
     string des = input.Get<string>("des");
     string full_description = user.ToUpper() + "=>" + des;
     output.Set<string>("dt", input.Get<string>("dt"));
     output.Set<string>("full_description", full_description);
     output.Set<Guid>("new_guid", Guid.NewGuid());
     output.Set<Guid>("guid", input.Get<Guid>("guid"));
     return output.AsReadOnly();
 }
}
```

En este escenario de caso de uso, el procesador genera una nueva columna denominada "full_description" combinando las existentes (en este caso, “user” en mayúsculas y “des”). También vuelve a generar un GUID y devuelve los valores de GUID originales y nuevos.

Como puede ver en el ejemplo anterior, puede llamar a métodos de C# durante la llamada al método `output.Set`.

Este es un ejemplo de script U-SQL base que utiliza un procesador personalizado:

```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid Guid,
        dt String,
            user String,
            des String
    FROM @input_file USING Extractors.Tsv();

@rs1 =
     PROCESS @rs0
     PRODUCE dt String,
             full_description String,
             guid Guid,
             new_guid Guid
     USING new USQL_Programmability.FullDescriptionProcessor();

OUTPUT @rs1 TO @output_file USING Outputters.Text();
```

## <a name="use-user-defined-appliers"></a>Uso de aplicadores definidos por el usuario
Un aplicador definido por el usuario U-SQL permite invocar una función de C# personalizada para cada fila devuelta por la expresión de tabla externa de una consulta. La entrada de la derecha se evalúa para cada fila de la entrada izquierda y las filas producidas se combinan en la salida final. La lista de las columnas producidas por el operador APPLY es la combinación del conjunto de columnas de la entrada izquierda y derecha.

El aplicador definido por el usuario se invoca como parte de la expresión USQL SELECT.

La llamada típica al aplicador definido por el usuario tiene un aspecto parecido al siguiente:

```usql
SELECT …
FROM …
CROSS APPLYis used to pass parameters
new MyScript.MyApplier(param1, param2) AS alias(output_param1 string, …);
```

Consulte [U-SQL SELECT Selecting from CROSS APPLY and OUTER APPLY](/u-sql/statements-and-expressions/select/from/select-selecting-from-cross-apply-and-outer-apply) (U-SQL SELECT, selección de CROSS APPLY y OUTER APPLY) para más información sobre cómo utilizar aplicadores en una expresión SELECT.

La definición de clase base de aplicador definida por el usuario es la siguiente:

```csharp
public abstract class IApplier : IUserDefinedOperator
{
protected IApplier();

public abstract IEnumerable<IRow> Apply(IRow input, IUpdatableRow output);
}
```

Para definir un aplicador definido por el usuario, es necesario crear la interfaz `IApplier` con el atributo [`SqlUserDefinedApplier`], que es opcional para la definición de este aplicador.

```csharp
[SqlUserDefinedApplier]
public class ParserApplier : IApplier
{
    public ParserApplier()
    {
        …
    }

    public override IEnumerable<IRow> Apply(IRow input, IUpdatableRow output)
    {
        …
    }
}
```

* Se llama a Apply para cada fila de la tabla externa. Devuelve el conjunto de filas de salida `IUpdatableRow`.
* La clase Constructor se utiliza para pasar parámetros al aplicador definido por el usuario.

**SqlUserDefinedApplier** indica que el tipo se debe registrar como un aplicador definido por el usuario. No se puede heredar esta clase.

**SqlUserDefinedApplier** es **opcional** para la definición del aplicador definido por el usuario.


Los objetos de programación principales son los siguientes:

```csharp
public override IEnumerable<IRow> Apply(IRow input, IUpdatableRow output)
```

Los conjuntos de filas de entrada se pasan como entrada `IRow`. Las filas de salida se generan como interfaz de salida `IUpdatableRow`.

Los nombres de columna individuales se pueden determinar mediante una llamada al método Schema `IRow`.

```csharp
ISchema schema = row.Schema;
var col = schema[i];
string val = row.Get<string>(col.Name)
```

Para obtener los valores de datos reales a partir de `IRow` de entrada, utilizamos el método Get() de la interfaz `IRow`.

```csharp
mycolumn = row.Get<int>("mycolumn")
```

O bien, se utiliza el nombre de columna de esquema:

```csharp
row.Get<int>(row.Schema[0].Name)
```

Los valores de salida se deben establecer con la salida `IUpdatableRow`:

```csharp
output.Set<int>("mycolumn", mycolumn)
```

Es importante comprender que los aplicadores personalizados solo darán como resultado columnas y valores definidos con la llamada al método `output.Set`.

La salida real se desencadena mediante una llamada a `yield return output.AsReadOnly();`.

Los parámetros del aplicador definido por el usuario se pueden pasar al constructor. El aplicador puede devolver un número variable de columnas que debe definirse durante la llamada al aplicador en el script base U-SQL.

```csharp
new USQL_Programmability.ParserApplier ("all") AS properties(make string, model string, year string, type string, millage int);
```

Este es el ejemplo del aplicador definido por el usuario:

```csharp
[SqlUserDefinedApplier]
public class ParserApplier : IApplier
{
private string parsingPart;

public ParserApplier(string parsingPart)
{
    if (parsingPart.ToUpper().Contains("ALL")
    || parsingPart.ToUpper().Contains("MAKE")
    || parsingPart.ToUpper().Contains("MODEL")
    || parsingPart.ToUpper().Contains("YEAR")
    || parsingPart.ToUpper().Contains("TYPE")
    || parsingPart.ToUpper().Contains("MILLAGE")
    )
    {
    this.parsingPart = parsingPart;
    }
    else
    {
    throw new ArgumentException("Incorrect parameter. Please use: 'ALL[MAKE|MODEL|TYPE|MILLAGE]'");
    }
}

public override IEnumerable<IRow> Apply(IRow input, IUpdatableRow output)
{

    string[] properties = input.Get<string>("properties").Split(',');

    //  only process with correct number of properties
    if (properties.Count() == 5)
    {

    string make = properties[0];
    string model = properties[1];
    string year = properties[2];
    string type = properties[3];
    int millage = -1;

    // Only return millage if it is number, otherwise, -1
    if (!int.TryParse(properties[4], out millage))
    {
        millage = -1;
    }

    if (parsingPart.ToUpper().Contains("MAKE") || parsingPart.ToUpper().Contains("ALL")) output.Set<string>("make", make);
    if (parsingPart.ToUpper().Contains("MODEL") || parsingPart.ToUpper().Contains("ALL")) output.Set<string>("model", model);
    if (parsingPart.ToUpper().Contains("YEAR") || parsingPart.ToUpper().Contains("ALL")) output.Set<string>("year", year);
    if (parsingPart.ToUpper().Contains("TYPE") || parsingPart.ToUpper().Contains("ALL")) output.Set<string>("type", type);
    if (parsingPart.ToUpper().Contains("MILLAGE") || parsingPart.ToUpper().Contains("ALL")) output.Set<int>("millage", millage);
    }
    yield return output.AsReadOnly();            
}
}
```

Este es el script base U-SQL para este aplicador definido por el usuario:

```usql
DECLARE @input_file string = @"c:\usql-programmability\car_fleet.tsv";
DECLARE @output_file string = @"c:\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
        stocknumber int,
        vin String,
        properties String
    FROM @input_file USING Extractors.Tsv();

@rs1 =
    SELECT
        r.stocknumber,
        r.vin,
        properties.make,
        properties.model,
        properties.year,
        properties.type,
        properties.millage
    FROM @rs0 AS r
    CROSS APPLY
    new USQL_Programmability.ParserApplier ("all") AS properties(make string, model string, year string, type string, millage int);

OUTPUT @rs1 TO @output_file USING Outputters.Text();
```

En este escenario de caso de uso, el aplicador definido por el usuario actúa como un analizador de valores delimitados por comas para las propiedades de flota de automóviles. Las filas del archivo de entrada tienen el siguiente aspecto:

```text
103 Z1AB2CD123XY45889   Ford,Explorer,2005,SUV,152345
303 Y0AB2CD34XY458890   Chevrolet,Cruise,2010,4Dr,32455
210 X5AB2CD45XY458893   Nissan,Altima,2011,4Dr,74000
```

Es un archivo TSV delimitado por tabulaciones típico con la columna de propiedades que contiene las propiedades de los automóviles, como la marca y el modelo. Esas propiedades deben analizarse con las columnas de tabla. El aplicador proporcionado también permite generar un número dinámico de propiedades en el conjunto de filas de resultados, según el parámetro que se pasa. Puede generar todas las propiedades o solo un conjunto específico de propiedades.

```text
...USQL_Programmability.ParserApplier ("all")
...USQL_Programmability.ParserApplier ("make")
...USQL_Programmability.ParserApplier ("make&model")
```

Se puede llamar al aplicador definido por el usuario como una nueva instancia del objeto applier:

```usql
CROSS APPLY new MyNameSpace.MyApplier (parameter: "value") AS alias([columns types]…);
```

O con la invocación de un método de fábrica de contenedor:

```csharp
    CROSS APPLY MyNameSpace.MyApplier (parameter: "value") AS alias([columns types]…);
```

## <a name="use-user-defined-combiners"></a>Uso de combinadores definidos por el usuario
El combinador definido por el usuario o UDC permite combinar filas de conjuntos de filas izquierdos y derechos, en función de una lógica personalizada. El combinador definido por el usuario se utiliza con la expresión de COMBINE.

Se invoca a un combinador con la expresión COMBINE que proporciona la información necesaria sobre los conjuntos de filas de entrada, las columnas de agrupamiento, el esquema de resultados esperado e información adicional.

Para llamar a un combinador en un script base U-SQL, se usa la sintaxis siguiente:

```usql
Combine_Expression :=
    'COMBINE' Combine_Input
    'WITH' Combine_Input
    Join_On_Clause
    Produce_Clause
    [Readonly_Clause]
    [Required_Clause]
    USING_Clause.
```

Para más información, vea [COMBINE Expression (U-SQL)](/u-sql/statements-and-expressions/combine-expression) (Expresión COMBINE (U-SQL)).

Para definir un combinador definido por el usuario, es necesario crear la interfaz `ICombiner` con el atributo [`SqlUserDefinedCombiner`], que es opcional para la definición de dicho combinador.

Definición de clase base `ICombiner`:

```csharp
public abstract class ICombiner : IUserDefinedOperator
{
protected ICombiner();
public virtual IEnumerable<IRow> Combine(List<IRowset> inputs,
       IUpdatableRow output);
public abstract IEnumerable<IRow> Combine(IRowset left, IRowset right,
       IUpdatableRow output);
}
```

La implementación personalizada de una interfaz `ICombiner` debe contener la definición de una invalidación Combine `IEnumerable<IRow>`.

```csharp
[SqlUserDefinedCombiner]
public class MyCombiner : ICombiner
{

public override IEnumerable<IRow> Combine(IRowset left, IRowset right,
    IUpdatableRow output)
{
    …
}
}
```

El atributo **SqlUserDefinedCombiner** indica que el tipo se debe registrar como un combinador definido por el usuario. No se puede heredar esta clase.

**SqlUserDefinedCombiner** se utiliza para definir la propiedad del modo de combinador. Es un atributo opcional para la definición de un combinador definido por el usuario.

Modo CombinerMode

La enumeración de CombinerMode puede tomar los siguientes valores:

* Full (0) Cada fila de salida depende potencialmente de todas las filas de entrada desde la izquierda y derecha con el mismo valor de clave.

* Left (1) Cada fila de salida depende de una sola fila de entrada de la izquierda (y potencialmente todas las filas de la derecha con el mismo valor de clave).

* Right (2) Cada fila de salida depende de una sola fila de entrada de la derecha (y potencialmente todas las filas de la izquierda con el mismo valor de clave).

* Inner (3) Cada fila de salida depende de una sola fila de entrada de la izquierda y derecha con el mismo valor.

Ejemplo:    [`SqlUserDefinedCombiner(Mode=CombinerMode.Left)`]


Los objetos de programación principales son:

```csharp
    public override IEnumerable<IRow> Combine(IRowset left, IRowset right,
        IUpdatableRow output
```

Los conjuntos de filas de entrada se pasan como el tipo de interfaz `IRowset` **left** y **right**. Ambos conjuntos de filas deben enumerarse para su procesamiento. Solo puede enumerar una vez cada interfaz, por lo que se debe enumerar y almacenar en caché si es necesario.

Para el almacenamiento en caché, se puede crear un tipo de estructura de memoria List\<T\> como resultado de la ejecución de la consulta LINQ y, más concretamente, List<`IRow`>. El tipo de datos anónimo también se puede utilizar durante la enumeración.

Consulte [Introducción a las consultas LINQ (C#)](/dotnet/csharp/programming-guide/concepts/linq/introduction-to-linq-queries) para más información sobre las consultas LINQ e [Interfaz IEnumerable\<T\>](/dotnet/api/system.collections.generic.ienumerable-1) para más información sobre la interfaz IEnumerable\<T\>.

Para obtener los valores de datos reales a partir de `IRowset` de entrada, utilizamos el método Get() de la interfaz `IRow`.

```csharp
mycolumn = row.Get<int>("mycolumn")
```

Los nombres de columna individuales se pueden determinar mediante una llamada al método Schema `IRow`.

```csharp
ISchema schema = row.Schema;
var col = schema[i];
string val = row.Get<string>(col.Name)
```

O bien, se utiliza el nombre de columna de esquema:

```csharp
c# row.Get<int>(row.Schema[0].Name)
```

La enumeración general con LINQ tiene el siguiente aspecto:

```csharp
var myRowset =
            (from row in left.Rows
                          select new
                          {
                              Mycolumn = row.Get<int>("mycolumn"),
                          }).ToList();
```

Después de enumerar ambos conjuntos de filas, vamos a crear un bucle a través de todas las filas. Para cada fila del conjunto de filas izquierdo, vamos a buscar todas las filas que satisfagan la condición de nuestro combinador.

Los valores de salida se deben establecer con la salida `IUpdatableRow`.

```csharp
output.Set<int>("mycolumn", mycolumn)
```

La salida real se desencadena mediante una llamada a `yield return output.AsReadOnly();`.

Este es un ejemplo de combinador:

```csharp
[SqlUserDefinedCombiner]
public class CombineSales : ICombiner
{

public override IEnumerable<IRow> Combine(IRowset left, IRowset right,
    IUpdatableRow output)
{
    var internetSales =
    (from row in left.Rows
          select new
          {
              ProductKey = row.Get<int>("ProductKey"),
              OrderDateKey = row.Get<int>("OrderDateKey"),
              SalesAmount = row.Get<decimal>("SalesAmount"),
              TaxAmt = row.Get<decimal>("TaxAmt")
          }).ToList();

    var resellerSales =
    (from row in right.Rows
     select new
     {
     ProductKey = row.Get<int>("ProductKey"),
     OrderDateKey = row.Get<int>("OrderDateKey"),
     SalesAmount = row.Get<decimal>("SalesAmount"),
     TaxAmt = row.Get<decimal>("TaxAmt")
     }).ToList();

    foreach (var row_i in internetSales)
    {
    foreach (var row_r in resellerSales)
    {

        if (
        row_i.OrderDateKey > 0
        && row_i.OrderDateKey < row_r.OrderDateKey
        && row_i.OrderDateKey == 20010701
        && (row_r.SalesAmount + row_r.TaxAmt) > 20000)
        {
        output.Set<int>("OrderDateKey", row_i.OrderDateKey);
        output.Set<int>("ProductKey", row_i.ProductKey);
        output.Set<decimal>("Internet_Sales_Amount", row_i.SalesAmount + row_i.TaxAmt);
        output.Set<decimal>("Reseller_Sales_Amount", row_r.SalesAmount + row_r.TaxAmt);
        }

    }
    }
    yield return output.AsReadOnly();
}
}
```

En este escenario de caso de uso, vamos a crear un informe de análisis para cada distribuidor. El objetivo es encontrar todos los productos cuyo costo es superior a 20 000 USD y que se venden a través del sitio web de Internet más rápido que mediante el distribuidor convencional dentro de un determinado período de tiempo.

Aquí está el script base U-SQL: Puede comparar la lógica entre JOIN convencional y un combinador:

```sql
DECLARE @LocalURI string = @"\usql-programmability\";

DECLARE @input_file_internet_sales string = @LocalURI+"FactInternetSales.txt";
DECLARE @input_file_reseller_sales string = @LocalURI+"FactResellerSales.txt";
DECLARE @output_file1 string = @LocalURI+"output_file1.tsv";
DECLARE @output_file2 string = @LocalURI+"output_file2.tsv";

@fact_internet_sales =
EXTRACT
    ProductKey int ,
    OrderDateKey int ,
    DueDateKey int ,
    ShipDateKey int ,
    CustomerKey int ,
    PromotionKey int ,
    CurrencyKey int ,
    SalesTerritoryKey int ,
    SalesOrderNumber String ,
    SalesOrderLineNumber  int ,
    RevisionNumber int ,
    OrderQuantity int ,
    UnitPrice decimal ,
    ExtendedAmount decimal,
    UnitPriceDiscountPct float ,
    DiscountAmount float ,
    ProductStandardCost decimal ,
    TotalProductCost decimal ,
    SalesAmount decimal ,
    TaxAmt decimal ,
    Freight decimal ,
    CarrierTrackingNumber String,
    CustomerPONumber String
FROM @input_file_internet_sales
USING Extractors.Text(delimiter:'|', encoding: Encoding.Unicode);

@fact_reseller_sales =
EXTRACT
    ProductKey int ,
    OrderDateKey int ,
    DueDateKey int ,
    ShipDateKey int ,
    ResellerKey int ,
    EmployeeKey int ,
    PromotionKey int ,
    CurrencyKey int ,
    SalesTerritoryKey int ,
    SalesOrderNumber String ,
    SalesOrderLineNumber  int ,
    RevisionNumber int ,
    OrderQuantity int ,
    UnitPrice decimal ,
    ExtendedAmount decimal,
    UnitPriceDiscountPct float ,
    DiscountAmount float ,
    ProductStandardCost decimal ,
    TotalProductCost decimal ,
    SalesAmount decimal ,
    TaxAmt decimal ,
    Freight decimal ,
    CarrierTrackingNumber String,
    CustomerPONumber String
FROM @input_file_reseller_sales
USING Extractors.Text(delimiter:'|', encoding: Encoding.Unicode);

@rs1 =
SELECT
    fis.OrderDateKey,
    fis.ProductKey,
    fis.SalesAmount+fis.TaxAmt AS Internet_Sales_Amount,
    frs.SalesAmount+frs.TaxAmt AS Reseller_Sales_Amount
FROM @fact_internet_sales AS fis
     INNER JOIN @fact_reseller_sales AS frs
     ON fis.ProductKey == frs.ProductKey
WHERE
    fis.OrderDateKey < frs.OrderDateKey
    AND fis.OrderDateKey == 20010701
    AND frs.SalesAmount+frs.TaxAmt > 20000;

@rs2 =
COMBINE @fact_internet_sales AS fis
WITH @fact_reseller_sales AS frs
ON fis.ProductKey == frs.ProductKey
PRODUCE OrderDateKey int,
        ProductKey int,
        Internet_Sales_Amount decimal,
        Reseller_Sales_Amount decimal
USING new USQL_Programmability.CombineSales();

OUTPUT @rs1 TO @output_file1 USING Outputters.Tsv();
OUTPUT @rs2 TO @output_file2 USING Outputters.Tsv();
```

Se puede llamar al combinador definido por el usuario como una nueva instancia del objeto applier:

```csharp
USING new MyNameSpace.MyCombiner();
```


O con la invocación de un método de fábrica de contenedor:

```csharp
USING MyNameSpace.MyCombiner();
```

## <a name="use-user-defined-reducers"></a>Uso de reductores definidos por el usuario

U-SQL le permite escribir reductores de conjuntos de filas personalizados en C# mediante el marco de extensibilidad de operadores definidos por el usuario mediante la implementación de una interfaz IReducer.

El reductor definido por el usuario o UDR puede usarse para eliminar las filas innecesarias durante la extracción de datos (importar). También se puede utilizar para manipular y evaluar las filas y columnas. En función de la lógica de programación, puede definir también qué filas se deben extraer.

Para definir una clase UDR, necesitamos crear una interfaz `IReducer` con el atributo `SqlUserDefinedReducer` opcional.

Esta interfaz de clase debe contener una definición de la invalidación del conjunto de filas de interfaz `IEnumerable`.

```csharp
[SqlUserDefinedReducer]
public class EmptyUserReducer : IReducer
{

    public override IEnumerable<IRow> Reduce(IRowset input, IUpdatableRow output)
    {
        …
    }

}
```

El atributo **SqlUserDefinedReducer** indica que el tipo se debe registrar como un reductor definido por el usuario. No se puede heredar esta clase.
**SqlUserDefinedReducer** es un atributo opcional para la definición del reductor definido por el usuario. Se utiliza para definir la propiedad IsRecursive.

* bool     IsRecursive    
* **true** = indica si esta reductor es asociativo y conmutativo

Los objetos de programación principales son **input** y **output**. El objeto input se utiliza para enumerar las filas de entrada. Output se usa para establecer filas de salida como resultado de la actividad de reducción.

Para la enumeración de filas de entrada, usamos el método `Row.Get`.

```csharp
foreach (IRow row in input.Rows)
{
    row.Get<string>("mycolumn");
}
```

El parámetro para el método `Row.Get` es una columna que se pasa como parte de la clase `PRODUCE` de la instrucción `REDUCE` del script base U-SQL. También es necesario usar aquí el tipo de datos correcto.

Para la salida, use el método `output.Set`.

Es importante comprender que el reductor personalizado solo dará como resultado valores definidos con la llamada al método `output.Set`.

```csharp
output.Set<string>("mycolumn", guid);
```

La salida del reductor real se desencadena mediante una llamada a `yield return output.AsReadOnly();`.

Este es un ejemplo de reductor:

```csharp
[SqlUserDefinedReducer]
public class EmptyUserReducer : IReducer
{

    public override IEnumerable<IRow> Reduce(IRowset input, IUpdatableRow output)
    {
    string guid;
    DateTime dt;
    string user;
    string des;

    foreach (IRow row in input.Rows)
    {
        guid = row.Get<string>("guid");
        dt = row.Get<DateTime>("dt");
        user = row.Get<string>("user");
        des = row.Get<string>("des");

        if (user.Length > 0)
        {
        output.Set<string>("guid", guid);
        output.Set<DateTime>("dt", dt);
        output.Set<string>("user", user);
        output.Set<string>("des", des);

        yield return output.AsReadOnly();
        }
    }
    }

}
```

En este escenario de caso de uso, el reductor omite filas con un nombre de usuario vacío. Para cada fila del conjunto de filas lee cada columna necesaria, después evalúa la longitud del nombre de usuario. Y genera la fila real solo si la longitud del valor del nombre de usuario es mayor que 0.

Este es un script base U-SQL que utiliza un reductor personalizado:

```usql
DECLARE @input_file string = @"\usql-programmability\input_file_reducer.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid string,
        dt DateTime,
            user String,
            des String
    FROM @input_file 
    USING Extractors.Tsv();

@rs1 =
    REDUCE @rs0 PRESORT guid
    ON guid  
    PRODUCE guid string, dt DateTime, user String, des String
    USING new USQL_Programmability.EmptyUserReducer();

@rs2 =
    SELECT guid AS start_id,
           dt AS start_time,
           DateTime.Now.ToString("M/d/yyyy") AS Nowdate,
           USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt).ToString() AS start_fiscalperiod,
           user,
           des
    FROM @rs1;

OUTPUT @rs2 
    TO @output_file 
    USING Outputters.Text();
```
