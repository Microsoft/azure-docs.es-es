---
title: 'Ejecución de trabajos de U-SQL en Python, R y C#: Azure Data Lake Analytics'
description: Obtenga información sobre cómo usar código subyacente con Python, R y C# al enviar trabajos en Azure Data Lake.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 11/22/2017
ms.custom: devx-track-python
ms.openlocfilehash: 673c8db0b7c6d3d8658d2d3b6cfd7e00f3d7d986
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876213"
---
# <a name="develop-u-sql-with-python-r-and-c-for-azure-data-lake-analytics-in-visual-studio-code"></a>Desarrollo de trabajos U-SQL con Python, R y C# para Azure Data Lake Analytics en Visual Studio Code
Aprenda cómo usar Visual Studio Code (VSCode) para escribir código subyacente de Python, R y C# con U-SQL y enviar trabajos al servicio Azure Data Lake. Para más información sobre las Herramientas de Azure Data Lake para Visual Studio Code, vea [Uso de Herramientas de Azure Data Lake para Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md).

Antes de escribir código personalizado subyacente, necesita abrir una carpeta o un área de trabajo en VSCode.


## <a name="prerequisites-for-python-and-r"></a>Requisitos previos para Python y R
Registre los ensamblados de extensiones de Python y R en la cuenta de ADL. 
1. Abra la cuenta en el portal.
   - Seleccione **Información general**. 
   - Haga clic en **Script de ejemplo**.
2. Haga clic en **Más**.
3. Seleccione **Instalar las extensiones de U-SQL**. 
4. Aparece el mensaje de confirmación una vez instaladas las extensiones de U-SQL. 

   ![Configuración del entorno de Python y R](./media/data-lake-analytics-data-lake-tools-for-vscode/setup-the-enrionment-for-python-and-r.png)

   > [!Note]
   > Para optimizar la experiencia con el servicio de lenguaje Python y R, instale la extensión de Python y R para VSCode. 

## <a name="develop-python-file"></a>Desarrollo de archivos de Python
1. Haga clic en **Archivo nuevo** en el área de trabajo.
2. Escriba el código en U-SQL. A continuación, se muestra un ejemplo de código.
    ```U-SQL
    REFERENCE ASSEMBLY [ExtPython];
    @t  = 
        SELECT * FROM 
        (VALUES
            ("D1","T1","A1","@foo Hello World @bar"),
            ("D2","T2","A2","@baz Hello World @beer")
        ) AS 
            D( date, time, author, tweet );

    @m  =
        REDUCE @t ON date
        PRODUCE date string, mentions string
        USING new Extension.Python.Reducer("pythonSample.usql.py", pyVersion : "3.5.1");

    OUTPUT @m
        TO "/tweetmentions.csv"
        USING Outputters.Csv();
    ```
    
3. Haga clic con el botón derecho en un archivo de script y, luego, seleccione **ADL: Generate Python Code Behind File**. 
4. Se genera el archivo **xxx.usql.py** en la carpeta de trabajo. Escriba el código en el archivo de Python. A continuación, se muestra un ejemplo de código.

    ```Python
    def get_mentions(tweet):
        return ';'.join( ( w[1:] for w in tweet.split() if w[0]=='@' ) )

    def usqlml_main(df):
        del df['time']
        del df['author']
        df['mentions'] = df.tweet.apply(get_mentions)
        del df['tweet']
        return df
    ```
5. Haga clic con el botón derecho en el archivo **USQL**; puede hacer clic en **Compile Script** (Compilar script) o **Submit Job** (Enviar trabajo) para ejecutar el trabajo.

## <a name="develop-r-file"></a>Desarrollo de archivos de R
1. Haga clic en **Archivo nuevo** en el área de trabajo.
2. Escriba el código en el archivo U-SQL. A continuación, se muestra un ejemplo de código.
    ```U-SQL
    DEPLOY RESOURCE @"/usqlext/samples/R/my_model_LM_Iris.rda";
    DECLARE @IrisData string = @"/usqlext/samples/R/iris.csv";
    DECLARE @OutputFilePredictions string = @"/my/R/Output/LMPredictionsIris.txt";
    DECLARE @PartitionCount int = 10;

    @InputData =
        EXTRACT SepalLength double,
                SepalWidth double,
                PetalLength double,
                PetalWidth double,
                Species string
        FROM @IrisData
        USING Extractors.Csv();

    @ExtendedData =
        SELECT Extension.R.RandomNumberGenerator.GetRandomNumber(@PartitionCount) AS Par,
            SepalLength,
            SepalWidth,
            PetalLength,
            PetalWidth
        FROM @InputData;

    // Predict Species

    @RScriptOutput =
        REDUCE @ExtendedData
        ON Par
        PRODUCE Par,
                fit double,
                lwr double,
                upr double
        READONLY Par
        USING new Extension.R.Reducer(scriptFile : "RClusterRun.usql.R", rReturnType : "dataframe", stringsAsFactors : false);
    OUTPUT @RScriptOutput
    TO @OutputFilePredictions
    USING Outputters.Tsv();
    ```
3. Haga clic con el botón derecho en el archivo **USQL** y después seleccione **ADL: Generate R Code Behind File**. 
4. Se genera el archivo **xxx.usql.r** en la carpeta de trabajo. Escriba el código en el archivo de R. A continuación, se muestra un ejemplo de código.

    ```R
    load("my_model_LM_Iris.rda")
    outputToUSQL=data.frame(predict(lm.fit, inputFromUSQL, interval="confidence"))
    ```
5. Haga clic con el botón derecho en el archivo **USQL**; puede hacer clic en **Compile Script** (Compilar script) o **Submit Job** (Enviar trabajo) para ejecutar el trabajo.

## <a name="develop-c-file"></a>Desarrollo de archivos de C#
Un archivo de código subyacente es un archivo C# asociado con un solo script U-SQL. Puede definir un script dedicado a UDO, UDA, UDT o UDF en el archivo de código subyacente. UDF, UDO, UDA o UDT se puede usar directamente en el script sin tener que registrar primero el ensamblado. El archivo de código subyacente se coloca en la misma carpeta que su archivo de script U-SQL de emparejamiento. Si el script se denomina xxx.usql, el código subyacente se denomina xxx.usql.cs. Si elimina manualmente el archivo de código subyacente, la característica de código subyacente se deshabilita para su script U-SQL asociado. Para más información sobre cómo escribir código de cliente para el script U-SQL, consulte [Writing and Using Custom Code in U-SQL – User-Defined Functions]( https://blogs.msdn.microsoft.com/visualstudio/2015/10/28/writing-and-using-custom-code-in-u-sql-user-defined-functions/) (Escritura y uso de código personalizado en U-SQL: funciones definidas por el usuario).

1. Haga clic en **Archivo nuevo** en el área de trabajo.
2. Escriba el código en el archivo U-SQL. A continuación, se muestra un ejemplo de código.
    ```U-SQL
    @a = 
        EXTRACT 
            Iid int,
        Starts DateTime,
        Region string,
        Query string,
        DwellTime int,
        Results string,
        ClickedUrls string 
        FROM @"/Samples/Data/SearchLog.tsv" 
        USING Extractors.Tsv();

    @d =
        SELECT DISTINCT Region 
        FROM @a;

    @d1 = 
        PROCESS @d
        PRODUCE 
            Region string,
        Mkt string
        USING new USQLApplication_codebehind.MyProcessor();

    OUTPUT @d1 
        TO @"/output/SearchLogtest.txt" 
        USING Outputters.Tsv();
    ```
3. Haga clic con el botón derecho en el archivo **USQL** y después seleccione **ADL: Generate CS Code Behind File**. 
4. Se genera el archivo **xxx.usql.cs** en la carpeta de trabajo. Escriba el código en el archivo CS. A continuación, se muestra un ejemplo de código.

    ```CS
    namespace USQLApplication_codebehind
    {
        [SqlUserDefinedProcessor]

        public class MyProcessor : IProcessor
        {
            public override IRow Process(IRow input, IUpdatableRow output)
            {
                output.Set(0, input.Get<string>(0));
                output.Set(1, input.Get<string>(0));
                return output.AsReadOnly();
            } 
        }
    }
    ```
5. Haga clic con el botón derecho en el archivo **USQL**; puede hacer clic en **Compile Script** (Compilar script) o **Submit Job** (Enviar trabajo) para ejecutar el trabajo.

## <a name="next-steps"></a>Pasos siguientes
* [Uso de Azure Data Lake Tools para Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md)
* [Ejecución y depuración locales de U-SQL con Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md)
* [Introducción a Análisis de Data Lake mediante PowerShell](data-lake-analytics-get-started-powershell.md)
* [Introducción a Análisis de Data Lake mediante el Portal de Azure](data-lake-analytics-get-started-portal.md)
* [Uso de Data Lake Tools for Visual Studio para desarrollar aplicaciones de U-SQL](data-lake-analytics-data-lake-tools-get-started.md)
* [Uso del catálogo de Data Lake Analytics (U-SQL)](data-lake-analytics-use-u-sql-catalog.md)
