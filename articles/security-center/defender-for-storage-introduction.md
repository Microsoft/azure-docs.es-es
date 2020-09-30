---
title: 'Azure Defender para Storage: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para Storage.
author: memildin
ms.author: memildin
ms.date: 9/12/2020
ms.topic: conceptual
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 31d1bde1555f98164ccba32d4615ba51837c5ef7
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932573"
---
# <a name="introduction-to-azure-defender-for-storage"></a>Introducción a Azure Defender para Storage

**Azure Defender para Storage** detecta actividades potencialmente dañinas en las cuentas de Azure Storage. Los datos se pueden proteger tanto si se almacenan como contenedores de blobs, recursos compartidos de archivos o lagos de datos.

Esta capa de protección le permite afrontar las amenazas *sin* necesidad de ser un experto en seguridad y le ayuda a administrar sus sistemas de supervisión de la seguridad.


## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponible con carácter general|
|Precios:|**Azure Defender para Storage** se factura como se muestra en la [página de precios](security-center-pricing.md).|
|Tipos de almacenamiento protegido|[Blob Storage](https://azure.microsoft.com/services/storage/blobs/)<br>[Archivos de Azure](https://docs.microsoft.com/azure/storage/files/storage-files-introduction)<br>[Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-introduction)|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) US Gov<br>![No](./media/icons/no-icon.png) China Gov, otros gobiernos|
|||


## <a name="what-kind-of-alerts-does-azure-defender-for-storage-provide"></a>¿Qué tipos de alertas ofrece Azure Defender para Storage?

Las alertas de seguridad se desencadenan en los escenarios siguientes:

- **Actividad sospechosa**: por ejemplo, se ha accedido correctamente a la cuenta correctamente desde una dirección IP conocida como nodo de salida activo de Tor.
- **Comportamiento anómalo**: por ejemplo, se producen cambios en el patrón de acceso a una cuenta de almacenamiento.
- **Malware potencial cargado**: el análisis de reputación de hash indica que un archivo cargado contiene malware.

Las alertas incluyen detalles acerca del incidente que las desencadenó, así como recomendaciones acerca de cómo investigar y corregir las amenazas.

> [!TIP]
> Para simular alertas de almacenamiento, siga las instrucciones de [esta entrada de blog](https://techcommunity.microsoft.com/t5/azure-security-center/validating-atp-for-azure-storage-detections-in-azure-security/ba-p/1068131).


## <a name="what-is-hash-reputation-analysis-for-malware"></a>¿Qué es el análisis de reputación de hash para malware?

Para determinar si un archivo cargado es sospechoso, Azure Defender para Storage usa el análisis de reputación de hash compatible con la [inteligencia sobre amenazas de Microsoft](https://go.microsoft.com/fwlink/?linkid=2128684). Las herramientas de protección contra amenazas no examinan los archivos cargados, sino que examinan los registros de almacenamiento y comparan los valores hash de los archivos recién cargados con los de virus conocidos, troyanos, spyware y ransomware. 

Cuando se sospecha que un archivo contiene malware, Security Center muestra una alerta y, opcionalmente, puede enviar un correo electrónico al propietario del almacenamiento para que elimine el archivo sospechoso. Para configurar esta eliminación automática de archivos que el análisis de reputación de hash indica que contienen malware, implemente una [automatización del flujo de trabajo para desencadenar alertas que contengan el "malware potencial cargado en una cuenta de almacenamiento"](https://techcommunity.microsoft.com/t5/azure-security-center/how-to-respond-to-potential-malware-uploaded-to-azure-storage/ba-p/1452005).

> [!NOTE]
> Para habilitar las funcionalidades de protección contra amenazas de Security Center, debe habilitar Azure Defender en la suscripción que contiene las cargas de trabajo aplicables.
>
> Puede habilitar **Azure Defender para Storage** en el nivel de suscripción o de recursos.



## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre Azure Defender para Storage.

Para obtener material relacionado, consulte los siguientes artículos: 

- Tanto si Security Center genera una alerta, como si la recibe de un producto de seguridad diferente, puede exportarla. Para exportar las alertas a Azure Sentinel, a cualquier SIEM de terceros o a cualquier otra herramienta externa, siga las instrucciones de [Exportación de alertas a un SIEM](continuous-export.md).
- [Habilitar la protección avanzada de Azure Defender para Storage](https://docs.microsoft.com/azure/storage/common/storage-advanced-threat-protection)
- [Lista de alertas de Azure Defender para Storage](alerts-reference.md#alerts-azurestorage)
- [Funcionalidades de inteligencia sobre amenazas de Microsoft](https://go.microsoft.com/fwlink/?linkid=2128684)