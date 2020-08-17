---
title: Extend U-SQL scripts with R in Azure Data Lake Analytics (Extensión de los scripts de U-SQL con R en Azure Data Lake Analytics)
description: Aprenda a ejecutar código R en scripts de U-SQL mediante Azure Data Lake Analytics. Inserte código R en línea o haga referencia a él desde archivos.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/20/2017
ms.openlocfilehash: 1c054dcfe12ad1d2098fc5352183839704e39330
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87127407"
---
# <a name="extend-u-sql-scripts-with-r-code-in-azure-data-lake-analytics"></a>Extend U-SQL scripts with R code in Azure Data Lake Analytics (Extensión de los scripts de U-SQL con código R en Azure Data Lake Analytics)

En el ejemplo siguiente se muestran los pasos básicos para implementar código R:

* Uso de la instrucción `REFERENCE ASSEMBLY` para habilitar las extensiones de R para el script de U-SQL.
* Use la operación `REDUCE` para la partición de datos de entrada de una clave.
* Las extensiones de R para U-SQL incluyen un reductor integrado (`Extension.R.Reducer`) que ejecuta código R en cada vértice asignado al reductor.
* Utilización de marcos de datos con nombre dedicados llamados `inputFromUSQL` y `outputToUSQL`, respectivamente, para pasar datos entre U-SQL y R. Los nombres del identificador DataFrame de entrada y salida son fijos (es decir, los usuarios no pueden cambiar estos nombres predefinidos de los identificadores DataFrame de entrada y salida).

## <a name="embedding-r-code-in-the-u-sql-script"></a>Incrustación de código R en el script de U-SQL

Mediante el parámetro de comando `Extension.R.Reducer`, puede insertar el código R de su script U-SQL. Por ejemplo, puede declarar el script de R como una variable de cadena y pasarla como un parámetro al reductor.

```usql
REFERENCE ASSEMBLY [ExtR];

DECLARE @myRScript = @"
inputFromUSQL$Species = as.factor(inputFromUSQL$Species)
lm.fit=lm(unclass(Species)~.-Par, data=inputFromUSQL)
#do not return readonly columns and make sure that the column names are the same in usql and r cripts,
outputToUSQL=data.frame(summary(lm.fit)$coefficients)
colnames(outputToUSQL) <- c(""Estimate"", ""StdError"", ""tValue"", ""Pr"")
outputToUSQL
";

@RScriptOutput = REDUCE … USING new Extension.R.Reducer(command:@myRScript, ReturnType:"dataframe");
```

## <a name="keep-the-r-code-in-a-separate-file-and-reference-it--the-u-sql-script"></a>Mantenga el código R en un archivo aparte y haga referencia a él mediante el script U-SQL.

En el ejemplo siguiente se ilustra un uso más complejo. En este caso, el código R se implementa como RESOURCE que es el script de U-SQL.

Guarde este código R como un archivo aparte.

```usql
load("my_model_LM_Iris.rda")
outputToUSQL=data.frame(predict(lm.fit, inputFromUSQL, interval="confidence"))
```

Use un script U-SQL para implementar ese script R con la instrucción DEPLOY RESOURCE.

```usql
REFERENCE ASSEMBLY [ExtR];
DEPLOY RESOURCE @"/usqlext/samples/R/RinUSQL_PredictUsingLinearModelasDF.R";
DEPLOY RESOURCE @"/usqlext/samples/R/my_model_LM_Iris.rda";
DECLARE @IrisData string = @"/usqlext/samples/R/iris.csv";
DECLARE @OutputFilePredictions string = @"/my/R/Output/LMPredictionsIris.txt";
DECLARE @PartitionCount int = 10;
@InputData =
    EXTRACT
        SepalLength double,
        SepalWidth double,
        PetalLength double,
        PetalWidth double,
        Species string
    FROM @IrisData
    USING Extractors.Csv();
@ExtendedData =
    SELECT
        Extension.R.RandomNumberGenerator.GetRandomNumber(@PartitionCount) AS Par,
        SepalLength,
        SepalWidth,
        PetalLength,
        PetalWidth
    FROM @InputData;
// Predict Species
@RScriptOutput = REDUCE @ExtendedData ON Par
    PRODUCE Par, fit double, lwr double, upr double
    READONLY Par
    USING new Extension.R.Reducer(scriptFile:"RinUSQL_PredictUsingLinearModelasDF.R", rReturnType:"dataframe", stringsAsFactors:false);
    OUTPUT @RScriptOutput TO @OutputFilePredictions USING Outputters.Tsv();
```

## <a name="how-r-integrates-with-u-sql"></a>Cómo se integra R con U-SQL

### <a name="datatypes"></a>Tipos de datos

* Las columnas de cadena y numéricas de U-SQL se convierten tal cual entre R DataFrame y U-SQL [tipos admitidos: `double`, `string`, `bool`, `integer` y `byte`].
* El tipo de datos `Factor` no se admite en U-SQL.
* `byte[]` debe serializarse como `string` codificada en base64.
* Las cadenas de U-SQL se pueden convertir en factores en el código R, una vez que U-SQL cree una trama de datos de entrada de R o mediante el establecimiento del parámetro reductor `stringsAsFactors: true`.

### <a name="schemas"></a>Esquemas

* Los conjuntos de datos de U-SQL no pueden tener nombres de columna duplicados.
* Los nombres de columna de conjuntos de datos de U-SQL deben ser cadenas.
* Los nombres de columna deben ser los mismos en los scripts de U-SQL y de R.
* La columna de solo lectura no puede formar parte de la trama de datos de salida, ya que las columnas de solo lectura vuelven a insertarse automáticamente en la tabla de U-SQL si forma parte del esquema de salida de UDO.

### <a name="functional-limitations"></a>Limitaciones funcionales

* No se puede crear una instancia dos veces del motor de R en el mismo proceso.
* En la actualidad, U-SQL no admite UDO de combinación para la predicción mediante modelos particionados generados con UDO de reducción. Los usuarios pueden declarar los modelos particionados como recurso y usarlos en su script de R (consulte el código de ejemplo `ExtR_PredictUsingLMRawStringReducer.usql`)

### <a name="r-versions"></a>Versiones de R

Solo se admite R 3.2.2.

### <a name="standard-r-modules"></a>Módulos de R estándar

```usql
base
boot
Class
Cluster
codetools
compiler
datasets
doParallel
doRSR
foreach
foreign
Graphics
grDevices
grid
iterators
KernSmooth
lattice
MASS
Matrix
Methods
mgcv
nlme
Nnet
Parallel
pkgXMLBuilder
RevoIOQ
revoIpe
RevoMods
RevoPemaR
RevoRpeConnector
RevoRsrConnector
RevoScaleR
RevoTreeView
RevoUtils
RevoUtilsMath
Rpart
RUnit
spatial
splines
Stats
stats4
survival
Tcltk
Tools
translations
utils
XML
```

### <a name="input-and-output-size-limitations"></a>Limitaciones de tamaño de entrada y salida

Cada vértice tiene una cantidad limitada de memoria asignada a él. Dado que las tramas de datos de entrada y salida deben existir en memoria en el código R, el tamaño total de la entrada y la salida no puede ser superior a 500 MB.

### <a name="sample-code"></a>Código de ejemplo

Puede encontrar más código de ejemplo en su cuenta Data Lake Store después de instalar las extensiones U-SQL Advanced Analytics. Ruta de acceso a más código de ejemplo: `<your_account_address>/usqlext/samples/R`.

## <a name="deploying-custom-r-modules-with-u-sql"></a>Implementación de módulos de R personalizados con U-SQL

En primer lugar, cree un módulo personalizado de R, comprímalo y luego cargue este archivo comprimido en su almacén de ADL. En el ejemplo, cargaremos magittr_1.5.zip en la raíz de la cuenta de ADLS predeterminada para la cuenta de ADLA que vamos a usar. Una vez que cargue el módulo en el almacén de ADL, declárelo como que usa DEPLOY RESOURCE para ponerlo a disposición en su script U-SQL y llame a `install.packages` para instalarlo.

```usql
REFERENCE ASSEMBLY [ExtR];
DEPLOY RESOURCE @"/magrittr_1.5.zip";
DECLARE @IrisData string =  @"/usqlext/samples/R/iris.csv";
DECLARE @OutputFileModelSummary string = @"/R/Output/CustomPackages.txt";
// R script to run
DECLARE @myRScript = @"
# install the magrittr package,
install.packages('magrittr_1.5.zip', repos = NULL),
# load the magrittr package,
require(magrittr),
# demonstrate use of the magrittr package,
2 %>% sqrt
";
@InputData =
EXTRACT SepalLength double,
SepalWidth double,
PetalLength double,
PetalWidth double,
Species string
FROM @IrisData
USING Extractors.Csv();
@ExtendedData =
SELECT 0 AS Par,
*
FROM @InputData;
@RScriptOutput = REDUCE @ExtendedData ON Par
PRODUCE Par, RowId int, ROutput string
READONLY Par
USING new Extension.R.Reducer(command:@myRScript, rReturnType:"charactermatrix");
OUTPUT @RScriptOutput TO @OutputFileModelSummary USING Outputters.Tsv();
```

## <a name="next-steps"></a>Pasos siguientes

* [Información general de Análisis de Microsoft Azure Data Lake](data-lake-analytics-overview.md)
* [Desarrollo de scripts U-SQL mediante Data Lake Tools for Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
* [Uso de funciones de ventana de U-SQL para trabajos de Análisis de Azure Data Lake](data-lake-analytics-use-window-functions.md)
