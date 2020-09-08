---
title: Elección de la solución de Azure para la transferencia de datos | Microsoft Docs
description: Aprenda a elegir una solución de Azure para la transferencia de datos según el tamaño de estos y el ancho de banda de red disponible en el entorno.
services: storage
author: alkohli
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.date: 06/03/2019
ms.author: alkohli
ms.openlocfilehash: 123fa86714ae3cefe7de5bebda9e0fccd385ecb4
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836605"
---
# <a name="choose-an-azure-solution-for-data-transfer"></a>Elección de la solución de Azure para la transferencia de datos

En este artículo se proporciona información general de algunas de las soluciones de transferencia de datos de Azure. En el artículo también se incluyen vínculos a las opciones recomendadas en función del ancho de banda de red del entorno y el tamaño de los datos que quiera transferir.

## <a name="types-of-data-movement"></a>Tipos de movimientos de datos

La transferencia de datos puede realizarse sin conexión o mediante la conexión de red. Elija la solución en función de lo siguiente:

- **Tamaño de los datos** que quiere transferir.
- **Frecuencia de transferencia**: ingesta de datos única o periódica.
- **Red**: ancho de banda disponible para la transferencia de datos en el entorno.

El movimiento de datos puede ser de los siguientes tipos:

- **Transferencia sin conexión mediante dispositivos entregables**: use dispositivos entregables físicos para la transferencia de datos única en bloque sin conexión. Microsoft envía un disco o un dispositivo seguro especializado. Como alternativa, puede comprar y enviar sus propios discos. Copie los datos en el dispositivo y envíelo a Azure para cargarlos.  Las opciones disponibles para este caso son Data Box Disk, Data Box, Data Box Heavy e Import/Export (con sus propios discos).

- **Transferencia de red**: los datos se transfieren a Azure mediante la conexión de red. Esto puede hacerse de diferentes maneras.

    - **Interfaz gráfica**: si únicamente transfiere algunos datos de manera esporádica y no necesita automatizar la transferencia, puede elegir una herramienta de interfaz gráfica, como el Explorador de Azure Storage o una herramienta de exploración basada en web de Azure Portal.
    - **Transferencia mediante scripts o programación**: puede usar las herramientas de software optimizadas que ofrecemos o llamar directamente a los SDK o las API REST. Las herramientas de scripts disponibles son AzCopy, Azure PowerShell y la CLI de Azure. Para la interfaz de programación, utilice uno de los SDK para .NET, Java, Python, Node/JS, C++, Go, PHP o Ruby.
    - **Dispositivos locales**: nosotros ofrecemos un dispositivo virtual o físico que reside en el centro de datos y optimiza la transferencia de datos a través de la red. Estos dispositivos también proporcionan una caché local de los archivos usados frecuentemente. El dispositivo físico es Azure Stack Edge y el virtual, Data Box Gateway. Ambos se ejecutan permanentemente en el entorno local y se conectan a Azure a través de la red.
    - **Canalización de datos administrados**: puede configurar una canalización en la nube para transferir archivos periódicamente entre distintos servicios de Azure, locales o de ambos tipos. Utilice Azure Data Factory para configurar y administrar canalizaciones de datos, y mover y transformar los datos para el análisis.

En la siguiente ilustración se muestran las directrices para elegir las herramientas de transferencia de datos de Azure en función del ancho de banda de red disponible para la transferencia, el tamaño de los datos que se van a transferir y la frecuencia de las transferencias.

![Herramientas de transferencia de datos de Azure](media/storage-choose-data-transfer-solution/azure-data-transfer-options-3.png)

**Los límites superiores de los dispositivos de transferencia sin conexión (Data Box Disk, Data Box y Data Box Heavy) pueden ampliarse mediante la realización de varios pedidos de un tipo de dispositivo.*

## <a name="selecting-a-data-transfer-solution"></a>Selección de una solución de transferencia de datos

Responda a las siguientes preguntas, que le ayudarán a seleccionar una solución de transferencia de datos:

- ¿Es el ancho de banda de red disponible limitado o inexistente, y desea transferir grandes conjuntos de datos?
  
    Si es así, consulte: [Escenario 1: Transferencia de grandes conjuntos de datos sin ancho de banda de red o con ancho de banda de red reducido](storage-solution-large-dataset-low-network.md).
- ¿Desea transferir grandes conjuntos de datos a través de la red y tiene un ancho de banda de red moderado o alto?

    Si es así, consulte: [Escenario 2: Transferencia de grandes conjuntos de datos con ancho de banda de red moderado o alto](storage-solution-large-dataset-moderate-high-network.md).
- ¿Desea transferir solo unos archivos de manera esporádica a través de la red?

    Si es así, consulte el [Escenario 3: Transferencia de pequeños conjuntos de datos con ancho de banda de red limitado o moderado](storage-solution-small-dataset-low-moderate-network.md).
- ¿Lo que busca es una transferencia de datos periódica?

    En caso afirmativo, utilice las opciones de script o de programación que se describen en el [Escenario 4: Transferencias de datos periódicas](storage-solution-periodic-data-transfer.md).
- ¿Desea una transferencia de datos continua y a intervalos regulares?

    En caso afirmativo, utilice las opciones del [Escenario 4: Transferencias de datos periódicas](storage-solution-periodic-data-transfer.md).

## <a name="data-transfer-feature-in-azure-portal"></a>Característica Transferencia de datos en Azure Portal

También puede ir a su cuenta de Azure Storage en Azure Portal y seleccionar la característica **Transferencia de datos**. Proporcione el ancho de banda de red en su entorno, el tamaño de los datos que desea transferir y la frecuencia de transferencia de datos. Verá las soluciones de transferencia de datos óptimas correspondientes a la información que ha proporcionado. 

## <a name="next-steps"></a>Pasos siguientes

- [Introducción al Explorador de Azure Storage](https://azure.microsoft.com/resources/videos/introduction-to-microsoft-azure-storage-explorer/)
- [Introducción a AzCopy](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-v10)
- [Inicio rápido: Carga, descarga y enumeración de blobs mediante PowerShell](../blobs/storage-quickstart-blobs-powershell.md)
- [Inicio rápido: Creación, descarga y enumeración de blobs mediante la CLI de Azure](../blobs/storage-quickstart-blobs-cli.md)
- Más información sobre lo siguiente:

    - [Azure Data Box, Azure Data Box Disk y Azure Data Box Heavy para las transferencias sin conexión](https://docs.microsoft.com/azure/databox/).
    - [Azure Data Box Gateway y Azure Data Box Edge para las transferencias en línea](https://docs.microsoft.com/azure/databox-online/).
- [Qué es Azure Data Factory](https://docs.microsoft.com/azure/data-factory/copy-activity-overview).
- Use las API REST para transferir datos

    - [En .NET](https://docs.microsoft.com/dotnet/api/overview/azure/storage)
    - [En Java](https://docs.microsoft.com/java/api/overview/azure/storage)
