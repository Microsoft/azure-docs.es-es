---
title: Seguridad de la capa de transporte en Azure HDInsight
description: Seguridad de la capa de transporte (TLS) y Capa de sockets seguros (SSL) son protocolos criptográficos que proporcionan la seguridad de las comunicaciones a través de una red de equipos.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: seoapr2020
ms.date: 04/21/2020
ms.openlocfilehash: 6e46cca28c049a794db617797d5f09e9f3b5720d
ms.sourcegitcommit: 3c66bfd9c36cd204c299ed43b67de0ec08a7b968
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "90006898"
---
# <a name="transport-layer-security-in-azure-hdinsight"></a>Seguridad de la capa de transporte en Azure HDInsight

Las conexiones al clúster de HDInsight a través del punto de conexión de clúster público `https://CLUSTERNAME.azurehdinsight.net` se procesan a través de los nodos de puerta de enlace del clúster. Estas conexiones están protegidas mediante un protocolo denominado TLS. La aplicación de versiones superiores de TLS en las puertas de enlace mejora la seguridad de estas conexiones. Para obtener más información sobre por qué debe usar las versiones más recientes de TLS, consulte [Resolución del problema de TLS 1.0](https://docs.microsoft.com/security/solving-tls1-problem).

De manera predeterminada, los clústeres de Azure HDInsight aceptan conexiones TLS 1.2 en los puntos de conexión HTTPS públicos, así como versiones más antiguas por compatibilidad con versiones anteriores. Puede controlar la versión de TLS mínima admitida en los nodos de puerta de enlace durante la creación del clúster mediante Azure Portal o una plantilla de Resource Manager. En el portal, seleccione la versión de TLS en la pestaña **Seguridad y redes** durante la creación del clúster. En el caso de una plantilla de Resource Manager, en el momento de la implementación use la propiedad **minSupportedTlsVersion**. Para ver una plantilla de ejemplo, consulte la [plantilla de inicio rápido de TLS 1.2 (versión mínima) de HDInsight](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-minimum-tls). Esta propiedad admite tres valores: "1.0", "1.1" y "1.2", que corresponden a TLS 1.0+, TLS 1.1+ y TLS 1.2+, respectivamente.


## <a name="next-steps"></a>Pasos siguientes

* [Planificación de una red virtual para Azure HDInsight](./hdinsight-plan-virtual-network-deployment.md)
* [Creación de redes virtuales para clústeres de Azure HDInsight](hdinsight-create-virtual-network.md).
* [Grupos de seguridad de red](../virtual-network/security-overview.md).
