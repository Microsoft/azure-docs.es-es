---
title: 'C#, Apache Hive y Apache Pig en Apache Hadoop: Azure HDInsight'
description: Vea cómo usar funciones definidas por el usuario de C# con el streaming de Apache Hive y Apache Pig en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 12/06/2019
ms.openlocfilehash: 5e32e269a36196d774bb1163a1c70c5d818ddcbb
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89000624"
---
# <a name="use-c-user-defined-functions-with-apache-hive-and-apache-pig-on-apache-hadoop-in-hdinsight"></a>Use las funciones definidas por el usuario de C# con el streaming de Apache Hive y Apache Pig en Apache Hadoop de HDInsight.

Aprenda a usar funciones definidas por el usuario (UDF) de C# con [Apache Hive](https://hive.apache.org) y [Apache Pig](https://pig.apache.org) en HDInsight.

> [!IMPORTANT]
> Los pasos que se describen en este documento funcionan con clústeres de HDInsight basado en Linux. Linux es el único sistema operativo que se usa en la versión 3.4 de HDInsight, o en las superiores. Para obtener más información, consulte el artículo relativo al [control de versiones de componentes de HDInsight](../hdinsight-component-versioning.md).

Tanto Hive como Pig pueden pasar datos a aplicaciones externas para el procesamiento. Este proceso se conoce como _streaming_. Cuando se usa una aplicación .NET, los datos se pasan a la aplicación en STDIN y la aplicación devuelve los resultados en STDOUT. Para leer y escribir en STDIN y STDOUT, puede usar `Console.ReadLine()` y `Console.WriteLine()` desde una aplicación de la consola.

## <a name="prerequisites"></a>Prerrequisitos

* Estar familiarizado con la escritura y la compilación del código C# orientado a .NET Framework 4.5.

    Use el IDE que prefiera. Recomendamos [Visual Studio](https://www.visualstudio.com/vs) o [Visual Studio Code](https://code.visualstudio.com/). En los pasos descritos en este documento se utiliza Visual Studio 2019.

* Una manera de cargar archivos .exe en el clúster y ejecutar trabajos de Pig y Hive. Es aconsejable usar las [herramientas de Data Lake para Visual Studio](../../data-lake-analytics/data-lake-analytics-data-lake-tools-install.md), [Azure PowerShell](/powershell/azure) y [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest). En los pasos descritos en este documento se emplean las herramientas Data Lake para Visual Studio para cargar los archivos y ejecutar la consulta de ejemplo de Hive.

    Para obtener información sobre otras formas de ejecutar consultas de Hive, consulte [¿Qué son Apache Hive y HiveQL en Azure HDInsight?](hdinsight-use-hive.md)

* Hadoop en un clúster de HDInsight. Para obtener más información sobre cómo crear un clúster, consulte [Creación de clústeres de HDInsight](../hdinsight-hadoop-provision-linux-clusters.md).

## <a name="net-on-hdinsight"></a>.NET en HDInsight

Los clústeres de *HDInsight basados en Linux* usan [Mono (https://mono-project.com)](https://mono-project.com) para ejecutar aplicaciones .NET. La versión 4.2.1 de Mono está incluida en la versión 3.6 de HDInsight.

Si desea conocer más detalles sobre la compatibilidad entre Mono y las versiones de .NET Framework, consulte la página en la que se trata la [compatibilidad de Mono](https://www.mono-project.com/docs/about-mono/compatibility/).

Para obtener más información sobre la versión de .NET Framework y Mono incluida en las versiones de HDInsight, consulte el artículo relativo a las [versiones de componentes de HDInsight](../hdinsight-component-versioning.md).

## <a name="create-the-c-projects"></a>Creación de proyectos de C\#

En las siguientes secciones se describe cómo crear un proyecto de C# en Visual Studio para una UDF de Apache Hive y una UDF de Apache Pig.

### <a name="apache-hive-udf"></a>UDF de Apache Hive

Para crear un proyecto de C# para una UDF de Apache Hive:

1. Inicie Visual Studio.

2. Seleccione **Crear un proyecto**.

3. En la ventana **Crear un proyecto**, seleccione la plantilla **Aplicación de consola (.NET Framework)** (la versión de C#). Luego, seleccione **Siguiente**.

4. En la ventana **Configurar el nuevo proyecto**, escriba un **nombre de proyecto** de *HiveCSharp* y vaya a una **ubicación** donde guardar el nuevo proyecto o créela. Seleccione **Crear**.

5. En el IDE de Visual Studio, reemplace el contenido de *Program.cs* por el código siguiente:

    ```csharp
    using System;
    using System.Security.Cryptography;
    using System.Text;
    using System.Threading.Tasks;

    namespace HiveCSharp
    {
        class Program
        {
            static void Main(string[] args)
            {
                string line;
                // Read stdin in a loop
                while ((line = Console.ReadLine()) != null)
                {
                    // Parse the string, trimming line feeds
                    // and splitting fields at tabs
                    line = line.TrimEnd('\n');
                    string[] field = line.Split('\t');
                    string phoneLabel = field[1] + ' ' + field[2];
                    // Emit new data to stdout, delimited by tabs
                    Console.WriteLine("{0}\t{1}\t{2}", field[0], phoneLabel, GetMD5Hash(phoneLabel));
                }
            }
            /// <summary>
            /// Returns an MD5 hash for the given string
            /// </summary>
            /// <param name="input">string value</param>
            /// <returns>an MD5 hash</returns>
            static string GetMD5Hash(string input)
            {
                // Step 1, calculate MD5 hash from input
                MD5 md5 = System.Security.Cryptography.MD5.Create();
                byte[] inputBytes = System.Text.Encoding.ASCII.GetBytes(input);
                byte[] hash = md5.ComputeHash(inputBytes);

                // Step 2, convert byte array to hex string
                StringBuilder sb = new StringBuilder();
                for (int i = 0; i < hash.Length; i++)
                {
                    sb.Append(hash[i].ToString("x2"));
                }
                return sb.ToString();
            }
        }
    }
    ```

6. En la barra de menús, seleccione **Compilar** > **Compilar solución** para compilar el proyecto.

7. Cierre la solución.

### <a name="apache-pig-udf"></a>UDF de Apache Pig

Para crear un proyecto de C# para una UDF de Apache Hive:

1. Abra Visual Studio.

2. En la ventana **Inicio**, seleccione **Crear un proyecto**.

3. En la ventana **Crear un proyecto**, seleccione la plantilla **Aplicación de consola (.NET Framework)** (la versión de C#). Luego, seleccione **Siguiente**.

4. En la ventana **Configurar el nuevo proyecto**, escriba un **nombre de proyecto** de *PigUDF* y vaya a una **ubicación** donde guardar el nuevo proyecto o créela. Seleccione **Crear**.

5. En el IDE de Visual Studio, reemplace el contenido de *Program.cs* por el código siguiente:

    ```csharp
    using System;

    namespace PigUDF
    {
        class Program
        {
            static void Main(string[] args)
            {
                string line;
                // Read stdin in a loop
                while ((line = Console.ReadLine()) != null)
                {
                    // Fix formatting on lines that begin with an exception
                    if(line.StartsWith("java.lang.Exception"))
                    {
                        // Trim the error info off the beginning and add a note to the end of the line
                        line = line.Remove(0, 21) + " - java.lang.Exception";
                    }
                    // Split the fields apart at tab characters
                    string[] field = line.Split('\t');
                    // Put fields back together for writing
                    Console.WriteLine(String.Join("\t",field));
                }
            }
        }
    }
    ```

    Este código analiza las líneas que se envían desde Pig y cambia el formato de las que comienzan por `java.lang.Exception`.

6. En la barra de menús, elija **Compilar** > **Compilar solución** para compilar el proyecto.

7. Deje abierta la solución.

## <a name="upload-to-storage"></a>Carga en el almacenamiento

A continuación, cargue las aplicaciones UDF de Hive y Pig que se van a almacenar en un clúster de HDInsight.

1. En Visual Studio, vaya a **Ver** > **Explorador de servidores**.

1. En el **Explorador de servidores**, haga clic con el botón derecho en **Azure**, seleccione **Conectar a la suscripción de Microsoft Azure** y complete el proceso de inicio de sesión.

1. Expanda el clúster de HDInsight en el que desee implementar esta aplicación. Aparecerá una entrada con el texto **(Cuenta de almacenamiento predeterminada)** en la lista.

    ![Cuenta de almacenamiento predeterminada, clúster de HDInsight, Explorador de servidores](./media/apache-hadoop-hive-pig-udf-dotnet-csharp/hdinsight-storage-account.png)

    * Si se puede expandir esta entrada, significa que está utilizando una **cuenta de Azure Storage** como almacenamiento predeterminado para el clúster. Para ver los archivos incluidos en el almacenamiento predeterminado del clúster, expanda la entrada y, a continuación, haga doble clic en **(Contenedor predeterminado)** .

    * Si esta entrada no se puede expandir, quiere decir que está usando **Azure Data Lake Storage** como almacenamiento predeterminado para el clúster. Para ver los archivos incluidos en el almacenamiento predeterminado del clúster, haga doble clic en la entrada **(Cuenta de almacenamiento predeterminada)** .

1. Para cargar los archivos .exe, siga uno de estos métodos:

    * Si está usando una **cuenta de Azure Storage**, seleccione el icono **Cargar blob**.

        ![HDInsight: icono de carga para el proyecto nuevo](./media/apache-hadoop-hive-pig-udf-dotnet-csharp/hdinsight-upload-icon.png)

        En el cuadro de diálogo **Cargar nuevo archivo**, en **Nombre de archivo**, seleccione **Examinar**. En el cuadro de diálogo **Cargar blob**, vaya a la carpeta *bin\debug* del proyecto *HiveCSharp* y, a continuación, elija el archivo *HiveCSharp.exe*. Por último, seleccione **Abrir** y, a continuación, **Aceptar** para completar la carga.

    * Si usa **Azure Data Lake Storage**, haga clic con el botón derecho en un área vacía de la lista de archivos y, después, seleccione **Cargar**. Por último, elija el archivo *HiveCSharp.exe* y seleccione **Abrir**.

    Una vez que la carga del archivo *HiveCSharp.exe* haya finalizado, repita el proceso de carga con el archivo *PigUDF.exe*.

## <a name="run-an-apache-hive-query"></a>Ejecución de una consulta de Apache Hive

Ahora puede ejecutar una consulta de Hive que use su aplicación UDF de Hive.

1. En Visual Studio, vaya a **Ver** > **Explorador de servidores**.

2. Expanda **Azure** y, después, haga lo mismo con **HDInsight**.

3. Haga clic con el botón derecho en el clúster en el que ha implementado la aplicación *HiveCSharp* y, a continuación, elija **Escribir una consulta de Hive**.

4. Use el siguiente texto para la consulta de Hive:

    ```hiveql
    -- Uncomment the following if you are using Azure Storage
    -- add file wasbs:///HiveCSharp.exe;
    -- Uncomment the following if you are using Azure Data Lake Storage Gen1
    -- add file adl:///HiveCSharp.exe;
    -- Uncomment the following if you are using Azure Data Lake Storage Gen2
    -- add file abfs:///HiveCSharp.exe;

    SELECT TRANSFORM (clientid, devicemake, devicemodel)
    USING 'HiveCSharp.exe' AS
    (clientid string, phoneLabel string, phoneHash string)
    FROM hivesampletable
    ORDER BY clientid LIMIT 50;
    ```

    > [!IMPORTANT]
    > Quite la marca de comentario de la instrucción `add file` que coincide con el tipo de almacenamiento predeterminado que se usa para su clúster.

    Esta consulta selecciona los campos `clientid`, `devicemake` y `devicemodel` de `hivesampletable` y, a continuación, los pasa a la aplicación *HiveCSharp.exe*. La consulta espera que la aplicación devuelva tres campos, que están almacenados como `clientid`, `phoneLabel` y `phoneHash`. La consulta también espera que *HiveCSharp.exe* esté en la raíz del contenedor de almacenamiento predeterminado.

5. Cambie el valor predeterminado de **Interactivo** a **Lote** y después seleccione **Enviar** para enviar el trabajo al clúster de HDInsight. Se abre la ventana **Resumen del trabajo de Hive**.

6. Seleccione **Actualizar** para actualizar el resumen hasta que el valor de **Estado del trabajo** cambie a **Completado**. Para ver la salida del trabajo, seleccione **Salida de trabajo**.

## <a name="run-an-apache-pig-job"></a>Ejecución de un trabajo de Apache Pig

También puede ejecutar un trabajo de Pig que use su aplicación UDF de Pig.

1. Use SSH para conectarse a su clúster de HDInsight. (por ejemplo, ejecute el comando `ssh sshuser@<clustername>-ssh.azurehdinsight.net`). Para obtener más información, consulte [Uso de SSH con HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

2. Utilice el siguiente comando para iniciar la línea de comandos de Pig:

    ```shell
    pig
    ```

    Se muestra un símbolo del sistema `grunt>`.

3. Escriba lo siguiente para ejecutar un trabajo de Pig que usa la aplicación de .NET Framework:

    ```pig
    DEFINE streamer `PigUDF.exe` CACHE('/PigUDF.exe');
    LOGS = LOAD '/example/data/sample.log' as (LINE:chararray);
    LOG = FILTER LOGS by LINE is not null;
    DETAILS = STREAM LOG through streamer as (col1, col2, col3, col4, col5);
    DUMP DETAILS;
    ```

    La instrucción `DEFINE` crea un alias de `streamer` para la aplicación de *PigUDF.exe* y `CACHE` lo carga desde el almacenamiento predeterminado del clúster. Más adelante, `streamer` se usa con el operador `STREAM` para procesar las líneas individuales incluidas en `LOG` y devolver los datos como una serie de columnas.

    > [!NOTE]
    > El nombre de la aplicación que se usa para el streaming debe estar encerrado entre caracteres \` (acento grave) cuando se usa como alias, y entre caracteres ' (comilla sencilla) cuando se utiliza con `SHIP`.

4. Después de escribir la última línea, debe iniciarse el trabajo. Devuelve un resultado similar al texto siguiente:

    ```output
    (2019-07-15 16:43:25 SampleClass5 [WARN] problem finding id 1358451042 - java.lang.Exception)
    (2019-07-15 16:43:25 SampleClass5 [DEBUG] detail for id 1976092771)
    (2019-07-15 16:43:25 SampleClass5 [TRACE] verbose detail for id 1317358561)
    (2019-07-15 16:43:25 SampleClass5 [TRACE] verbose detail for id 1737534798)
    (2019-07-15 16:43:25 SampleClass7 [DEBUG] detail for id 1475865947)
    ```

5. Use `exit` para salir de Pig.

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a utilizar una aplicación de .NET Framework desde Hive y Pig en HDInsight. Si quiere obtener información sobre cómo usar Python con Hive y Pig, vea [Uso de Python con Apache Hive y Apache Pig en HDInsight](python-udf-hdinsight.md).

Para conocer otras formas de usar Pig y Hive y para obtener información acerca del uso de MapReduce, consulte los siguientes artículos:

* [Uso de Apache Hive con HDInsight](hdinsight-use-hive.md)
* [Uso de MapReduce con HDInsight](hdinsight-use-mapreduce.md)
* [Aspectos básicos de Pig Latin](https://pig.apache.org/docs/latest/basic.html)
