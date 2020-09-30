---
title: Protección de las credenciales de acceso con Servicios vinculados en Apache Spark para Azure Synapse Analytics
description: En este artículo se proporcionan conceptos necesarios para la integración de forma segura de Apache Spark para Synapse Analytics con otros servicios mediante servicios vinculados y la biblioteca de tokens
services: synapse-analytics
author: mlee3gsd
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: spark
ms.date: 08/26/2020
ms.author: martinle
ms.reviewer: euang
ms.openlocfilehash: 21b571c859ec8ecc66c1c9a222e0648dc7f28f4f
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89422131"
---
# <a name="securing-your-credentials-through-linked-services-with-the-tokenlibrary"></a>Protección de las credenciales a través de servicios vinculados con TokenLibrary
El acceso a los datos desde orígenes externos es un patrón común. Salvo que el origen de datos externo permita el acceso anónimo, lo más probable es que tenga que proteger la conexión con una credencial, un secreto o una cadena de conexión.  Azure Synapse Analytics proporciona servicios vinculados que simplifican el proceso de integración mediante el almacenamiento de los detalles de conexión en un servicio vinculado o en Azure Key Vault. Una vez que haya creado un servicio vinculado, Apache Spark puede hacer referencia a él para aplicar la información de conexión en el código. Para más información, consulte la información sobre los [servicios vinculados](../../data-factory/concepts-linked-services.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).
> [!NOTE]
> El acceso a los archivos desde Azure Data Lake Storage en su área de trabajo usa el acceso directo de AAD para la autenticación, por lo que no será preciso que use TokenLibrary. 


## <a name="prerequisite"></a>Requisito previo
* Servicio vinculado: debe crear un servicio vinculado al origen de datos externo y hacer referencia al servicio vinculado desde TokenLibrary. Más información sobre los [servicios vinculados](../../data-factory/concepts-linked-services.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).


## <a name="connecting-to-adls-gen2-outside-of-synapse-workspace"></a>Conexión a ADLS Gen2 fuera del área de trabajo de Synapse

Synapse ofrece una experiencia de servicios vinculados integrados a Azure Data Lake Storage Gen2.

```scala
// Scala code
val sc = spark.sparkContext
sc.hadoopConfiguration.set("spark.storage.synapse.linkedServiceName", "<LINKED SERVICE NAME>")
sc.hadoopConfiguration.set("fs.azure.account.auth.type", "SAS")
sc.hadoopConfiguration.set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedSASProvider")

val df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<DIRECTORY PATH>")

df.show()
```

```python
# Python code
sc._jsc.hadoopConfiguration().set("spark.storage.synapse.linkedServiceName", "<lINKED SERVICE NAME>")
sc._jsc.hadoopConfiguration().set("fs.azure.account.auth.type", "SAS")
sc._jsc.hadoopConfiguration().set("fs.azure.sas.token.provider.type", "com.microsoft.azure.synapse.tokenlibrary.LinkedServiceBasedSASProvider")

df = spark.read.csv("abfss://<CONTAINER>@<ACCOUNT>.dfs.core.windows.net/<DIRECTORY PATH>")

df.show()
```
## <a name="using-the-token-library"></a>Uso de TokenLibrary

Para conectarse a otros servicios vinculados, puede realizar una llamada directa a TokenLibrary.

### <a name="getconnectionstring"></a>GetConnectionString
 Para recuperar la cadena de conexión, use la función <b>getConnectionString</b> y pase el <b>nombre del servicio vinculado</b>.

```scala
// Scala
// retrieve connectionstring from TokenLibrary

import com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

val connectionString: String = TokenLibrary.getConnectionString("<LINKED SERVICE NAME>")
println(connectionString)
```

```python
# Python
# retrieve connectionstring from TokenLibrary

from pyspark.sql import SparkSession

sc = SparkSession.builder.getOrCreate()
token_library = sc._jvm.com.microsoft.azure.synapse.tokenlibrary.TokenLibrary
connection_string = token_library.getConnectionString("<LINKED SERVICE NAME>")
print(connection_string)
```
```csharp
// C#
// retrieve connectionstring from TokenLibrary

using Microsoft.Spark.Extensions.Azure.Synapse.Analytics.Utils;

string connectionString = TokenLibrary.GetConnectionString(<LINKED SERVICE NAME>);
Console.WriteLine(connectionString);
```

### <a name="getconnectionstringasmap"></a>GetConnectionStringAsMap
Para analizar determinados valores de un par <i>clave=valor</i> de la cadena de conexión, como 

<i>DefaultEndpointsProtocol=https;AccountName=\<AccountName>;AccountKey=\<AccountKey></i>

use la función <b>getConnectionStringAsMap</b> y pase la clave para devolver el valor.
```scala
// Linked services can be used for storing and retreiving credentials (e.g, account key)
// Example connection string (for storage): "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
import com.microsoft.azure.synapse.tokenlibrary.TokenLibrary

val accountKey: String = TokenLibrary.getConnectionStringAsMap("<LINKED SERVICE NAME">).get("<KEY NAME>")
println(accountKey)
```

```python
# Linked services can be used for storing and retreiving credentials (e.g, account key)
# Example connection string (for storage): "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
from pyspark.sql import SparkSession

sc = SparkSession.builder.getOrCreate()
token_library = sc._jvm.com.microsoft.azure.synapse.tokenlibrary.TokenLibrary
accountKey = token_library.getConnectionStringAsMap("<LINKED SERVICE NAME>").get("<KEY NAME>")
print(accountKey)
```

## <a name="next-steps"></a>Pasos siguientes

- [Escritura en un grupo de SQL](./synapse-spark-sql-pool-import-export.md)

