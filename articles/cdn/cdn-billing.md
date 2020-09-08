---
title: Descripción de la facturación de Azure CDN | Microsoft Docs
description: Obtenga información sobre la estructura de facturación para el contenido hospedado en Azure Content Delivery Network, incluidas las regiones de facturación, los cargos de entrega y la administración de los costos.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/13/2019
ms.author: allensu
ms.openlocfilehash: bd6b4831b29a99ed6694f75e64202f339385b7ea
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88191162"
---
# <a name="understanding-azure-cdn-billing"></a>Descripción de la facturación de Azure CDN

En este documento de preguntas frecuentes se describe la estructura de facturación del contenido hospedado en Azure Content Delivery Network (CDN).

## <a name="what-is-a-billing-region"></a>¿Qué es una región de facturación?
Una región de facturación es un área geográfica que se usa para determinar qué tarifa se cobra por la entrega de objetos desde Azure CDN. Las zonas de facturación actuales y sus regiones son las siguientes:

- Zona 1: Norteamérica, Europa, Oriente Medio y África

- Zona 2: Asia Pacífico (incluido Japón)

- Zona 3: Sudamérica

- Zona 4: Australia y Nueva Zelanda

- Zona 5: India

Para obtener información sobre las regiones de punto de presencia (POP), vea [Ubicaciones POP de Azure CDN por región](https://docs.microsoft.com/azure/cdn/cdn-pop-locations). Por ejemplo, una ubicación POP de México se encuentra en la región de Norteamérica y, por tanto, se incluye en la zona 1. 

Para obtener información sobre los precios de Azure CDN, consulte [Precios de Content Delivery Network](https://azure.microsoft.com/pricing/details/cdn/).

## <a name="how-are-delivery-charges-calculated-by-region"></a>¿Cómo se calculan los gastos de entrega por región?
La región de facturación de Azure CDN se basa en la ubicación del servidor de origen que entrega el contenido al usuario final. El destino (ubicación física) del cliente no se considera región de facturación.

Por ejemplo, si un usuario ubicado en México emite una solicitud y la atiende un servidor de una ubicación POP de Estados Unidos debido a las condiciones de tráfico o emparejamiento, la región de facturación será Estados Unidos.

## <a name="what-is-a-billable-azure-cdn-transaction"></a>¿Qué es una transacción facturable de Azure CDN?
Cualquier solicitud HTTP(S) que termine en la red CDN es un evento facturable, que incluye todos los tipos de respuesta: correcto, error u otros. Sin embargo, respuestas distintas pueden generar cantidades de tráfico diferentes. Por ejemplo, *304 No modificado* y otras respuestas de solo encabezado generan poco tráfico porque son una respuesta de encabezado pequeña; de forma similar, las respuestas de error (por ejemplo, *404 No encontrado*) son facturables, pero comportan un costo pequeño debido a la escasa carga de la respuesta.

## <a name="what-other-azure-costs-are-associated-with-azure-cdn-use"></a>¿Qué otros costos de Azure están asociados con el uso de Azure CDN?
El uso de Azure CDN también conlleva algunos cargos por la utilización de servicios usados como origen de los objetos. Estos costos suelen ser una pequeña fracción del costo total de utilización de CDN.

Si usa el almacenamiento de blobs de Azure como origen del contenido, también comporta los cargos de almacenamiento siguientes para el llenado de la caché:

- GB reales usados: el almacenamiento real de los objetos de origen.

- Transacciones: según sea necesario para llenar la caché.

- Transferencias en GB: la cantidad de datos transferidos para llenar las cachés de CDN.

> [!NOTE]
> A partir de octubre de 2019, si usa Azure CDN de Microsoft, la transferencia de datos desde los orígenes hospedados en Azure a los pop de la red CDN es gratuita. Azure CDN de Verizon y Azure CDN de Akamai están sujetos a las tarifas que se describen a continuación.

Para obtener más información sobre la facturación de Azure Storage, consulte [Descripción de la facturación de Azure Storage: ancho de banda, transacciones y capacidad](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/07/08/understanding-windows-azure-storage-billing-bandwidth-transactions-and-capacity/).

Si usa la *entrega de servicios hospedados*, se generarán cargos, como se indica a continuación:

- Tiempo de proceso de Azure: las instancias de proceso que actúan como origen.

- Transferencia de proceso de Azure: las transferencias de datos desde las instancias de proceso para llenar las cachés de Azure CDN.

Si el cliente usa solicitudes de intervalo de bytes (sin tener en cuenta el servicio de origen), se aplican las consideraciones siguientes:

- Una *solicitud de intervalo de bytes* es una transacción facturable en la red CDN. Cuando un cliente envía una solicitud de intervalo de bytes, esta solicitud es para un subconjunto (intervalo) del objeto. La red CDN responde con solo una porción del contenido que se solicita. Esta respuesta parcial es una transacción facturable y la cantidad de transferencia se limita al tamaño de la respuesta de intervalo (más los encabezados).

- Cuando llega una solicitud de solamente una parte de un objeto (por la especificación de un encabezado de intervalo de bytes), puede que la red CDN capture todo el objeto en su caché. Como resultado, incluso aunque la transacción facturable de la red CDN sea una respuesta parcial, la transacción facturable desde el origen puede implicar el tamaño completo del objeto.

## <a name="how-much-transfer-activity-occurs-to-support-the-cache"></a>¿Cuánta actividad de transferencia se produce como soporte de la memoria caché?
Cada vez que una ubicación POP de CDN necesita llenar su caché, realiza una solicitud al origen del objeto que se almacena en caché. Como resultado, el origen genera una transacción facturable en cada error de caché. El número de errores de caché depende de una serie de factores:

- El grado de capacidad de almacenamiento en caché que tiene el contenido: si el contenido tiene valores altos de TTL (período de vida)/expiración y se obtiene acceso a él con frecuencia, por lo que continúa siendo popular en la caché, la red CDN controla la gran mayoría de la carga. Una buena proporción de aciertos de caché típica es superior al 90 %, lo que significa que menos del 10 % de las solicitudes de cliente tienen que volver al origen, ya sea por un error de caché o por la actualización de objetos.

- El número de nodos que se necesitan para cargar el objeto: cada vez que un nodo carga un objeto desde el origen, se genera una transacción facturable. Como consecuencia, más contenido global (al que se tiene acceso desde más nodos) da como resultado más transacciones facturables.

- Influencia de TTL: un valor más alto de TTL para un objeto significa que este debe capturarse del origen con menos frecuencia. También significa que los clientes, como los exploradores, pueden almacenar más tiempo el objeto en caché, lo que puede reducir las transacciones hacia la red CDN.

## <a name="which-origin-services-are-eligible-for-free-data-transfer-with-azure-cdn-from-microsoft"></a>¿Qué servicios de origen son válidos para la transferencia de datos gratuita con Azure CDN de Microsoft? 
Si usa uno de los siguientes servicios de Azure como origen de CDN, no se le cobrará la transferencia de datos desde el origen a los pop de CDN. 

- Azure Storage
- Azure Media Services
- Azure Virtual Machines
- Virtual Network
- Load Balancer
- Application Gateway
- Azure DNS
- ExpressRoute
- VPN Gateway
- Traffic Manager
- Network Watcher
- Azure Firewall
- Azure Front Door Service
- Azure Bastion
- Azure App Service
- Azure Functions
- Azure Data Factory
- Azure API Management
- Azure Batch 
- Explorador de datos de Azure
- HDInsight
- Azure Cosmos DB
- Azure Data Lake Store
- Azure Machine Learning 
- Azure SQL Database
- Instancia administrada de Azure SQL
- Azure Cache for Redis

## <a name="how-do-i-manage-my-costs-most-effectively"></a>¿Cómo administro mis costos con más eficacia?
Establezca el valor de TTL más largo posible en su contenido. 
