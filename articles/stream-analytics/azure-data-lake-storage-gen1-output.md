---
title: Salida de Azure Data Lake Storage Gen 1 desde Azure Stream Analytics
description: En este artículo se describe Azure Data Lake Storage Gen 1 como una opción de salida para Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 98dfd51783ad3bc0b89f441f89436d3c3d928104
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88875492"
---
# <a name="azure-data-lake-storage-gen-1-output-from-azure-stream-analytics"></a>Salida de Azure Data Lake Storage Gen 1 desde Azure Stream Analytics

Stream Analytics admite salidas de [Azure Data Lake Storage Gen 1](../data-lake-store/data-lake-store-overview.md). Azure Data Lake Storage es un repositorio hiperescalado de nivel empresarial para cargas de trabajo de análisis de macrodatos. Puede usar Data Lake Storage para almacenar datos de cualquier tamaño, tipo y velocidad de ingesta para realizar análisis exploratorios y operativos. Stream Analytics debe tener autorización para acceder a Data Lake Storage.

La salida de Azure Data Lake Storage desde Stream Analytics no está disponible en las regiones de Azure China (21Vianet ) y Azure Alemania (T-Systems International).

## <a name="output-configuration"></a>Configuración de salida

En la siguiente tabla se muestra una lista de nombres de propiedades y su descripción para configurar la salida de Data Lake Storage Gen 1.

| Nombre de propiedad | Descripción |
| --- | --- |
| Alias de salida | Es un nombre descriptivo utilizado en las consultas para dirigir la salida de la consulta a Data Lake Storage. |
| Subscription | Suscripción que contiene la cuenta de Azure Data Lake Storage. |
| Nombre de cuenta | Nombre de la cuenta de Data Lake Storage donde va a enviar la salida. Se le presentará una lista desplegable de cuentas de Data Lake Storage que están disponibles en la suscripción. |
| Patrón del prefijo de la ruta de acceso | Ruta de acceso utilizada para escribir sus archivos en la cuenta de Data Lake Storage especificada. Puede especificar una o más instancias de las variables {date} y {time}:<br /><ul><li>Ejemplo 1: carpeta1/registros/{date}/{time}</li><li>Ejemplo 2: carpeta1/registros/{date}</li></ul><br />La marca de tiempo de la estructura de carpetas que creó usa la zona horaria UTC y no la hora local.<br /><br />Si el patrón de la ruta de acceso al archivo no contiene una barra diagonal (/) final, el patrón final de la ruta de acceso al archivo se considera un prefijo del nombre de archivo. <br /><br />Se crean nuevos archivos en las circunstancias siguientes:<ul><li>Cambio en el esquema de salida</li><li>Reinicio externo o interno de un trabajo</li></ul> |
| Formato de fecha | Opcional. Si el token de fecha se usa en la ruta de acceso de prefijo, puede seleccionar el formato de fecha en el que se organizan los archivos. Ejemplo: AAAA/MM/DD |
|Formato de hora | Opcional. Si el token de hora se usa en la ruta de acceso de prefijo, puede seleccionar el formato de hora en el que se organizan los archivos. Actualmente, el único valor admitido es HH. |
| Formato de serialización de eventos | Formato de serialización para los datos de salida. Se admiten JSON, CSV y Avro.|
| Encoding | Si usa el formato CSV o JSON, debe especificar una codificación. Por el momento, UTF-8 es el único formato de codificación compatible.|
| Delimitador | Solo se aplica para la serialización de CSV. Stream Analytics admite un número de delimitadores comunes para la serialización de datos CSV. Los valores admitidos son la coma, punto y coma, espacio, tabulador y barra vertical.|
| Formato | Solo se aplica para la serialización de JSON. La opción **Separado por líneas** especifica que en el formato de la salida cada objeto JSON está separado por una línea nueva. Si selecciona **Separado por líneas**, el objeto JSON se lee un objeto a la vez. El contenido total en sí mismo no sería un JSON válido.  La opción **Matriz** especifica que el formato de la salida es una matriz de objetos JSON. Esta matriz se cierra cuando el trabajo se detenga o Stream Analytics haya pasado a la siguiente ventana de tiempo. En general, es preferible utilizar JSON separado por líneas, ya que no requiere ningún control especial mientras todavía se esté escribiendo en el archivo de salida.|
| Modo de autenticación | Puede autorizar el acceso a su cuenta de Data Lake Storage mediante [identidades administradas](stream-analytics-managed-identities-adls.md) o un token de usuario. Una vez que concede acceso, puede revocarlo si cambia la contraseña de la cuenta de usuario, elimina la salida de Data Lake Storage para este trabajo o elimina el trabajo de Stream Analytics. |

## <a name="partitioning"></a>Creación de particiones

Para la clave de partición, use los tokens {date} y {time} en el patrón de prefijo de la ruta de acceso. Elija el formato de fecha, como AAAA/MM/DD, DD/MM/AAAA o MM-DD-AAAA. Use HH para el formato de hora. El número de escritores de salida sigue las particiones de entrada para [consultas que se pueden paralelizar totalmente](stream-analytics-scale-jobs.md).

## <a name="output-batch-size"></a>Tamaño de lote de salida

Para obtener el tamaño máximo del mensaje, consulte los [límites de Data Lake Storage](../azure-resource-manager/management/azure-subscription-service-limits.md#data-lake-store-limits). Para optimizar el tamaño del lote, use hasta 4 MB por operación de escritura.

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante la CLI de Azure](quick-create-azure-cli.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante una plantilla de ARM](quick-create-azure-resource-manager.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics con Visual Studio](stream-analytics-quick-create-vs.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-vs-code.md)