---
title: Cifrado doble en Microsoft Azure
description: En este artículo se describe cómo Microsoft Azure proporciona cifrado doble para datos en reposo y datos en tránsito.
services: security
documentationcenter: na
author: TerryLanfear
manager: rkarlin
ms.assetid: 9dcb190e-e534-4787-bf82-8ce73bf47dba
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/13/2020
ms.author: terrylan
ms.openlocfilehash: 020e8249f57ccb1a14da798a717a00dcc3962389
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88226954"
---
# <a name="double-encryption"></a>Cifrado doble
El cifrado doble es aquel en que dos o más capas independientes de cifrado están habilitadas para proteger frente a los peligros de cualquier otra capa de cifrado. El uso de dos capas de cifrado reduce las amenazas que surgen con el cifrado de datos. Por ejemplo:

- Errores de configuración en el cifrado de datos
- Errores de implementación en el algoritmo de cifrado
- Poner en peligro una única clave de cifrado

Azure proporciona cifrado doble para datos en reposo y datos en tránsito.

## <a name="data-at-rest"></a>Datos en reposo
El enfoque de Microsoft para habilitar dos capas de cifrado para los datos en reposo es:

- **Cifrado de datos con claves administradas por el cliente**. El usuario proporciona su propia clave para el cifrado de discos. Puede traer sus propias claves a su instancia de Key Vault (BYOK – Bring Your Own Key) o generar otras nuevas en Azure Key Vault para cifrar los recursos deseados.
- **Cifrado de la infraestructura mediante claves administradas por la plataforma**.  De forma predeterminada, los discos se cifran automáticamente en reposo mediante claves de cifrado administradas por la plataforma.

## <a name="data-in-transit"></a>Datos en tránsito
El enfoque de Microsoft para habilitar dos capas de cifrado para los datos en tránsito es el siguiente:

- **Cifrado del tránsito mediante Seguridad de la capa de transporte (TLS) 1.2 para proteger los datos cuando viajan entre los servicios en la nube y el usuario**. Todo el tráfico que sale de un centro de datos se cifra en tránsito, incluso si el destino del tráfico es otro controlador de dominio de la misma región. TLS 1.2 es el protocolo de seguridad predeterminado que se usa. TLS proporciona una autenticación sólida, privacidad de mensajes e integridad (lo que permite la detección de la manipulación, interceptación y falsificación de mensajes), interoperabilidad, flexibilidad de algoritmo, y facilidad de implementación y uso.
- **Capa adicional de cifrado que se proporciona en la capa de infraestructura**. Un método de cifrado de capa de vínculo de datos que usa los estándares de seguridad MAC IEEE 802.1AE (también conocido como MACsec) se aplica de punto a punto a través del hardware de red subyacente. Cada vez que el tráfico de los clientes de Azure se mueve entre los centros de datos (fuera de los límites físicos no controlados por Microsoft o en nombre de Microsoft), los paquetes se cifran y descifran en los dispositivos antes de enviarse, lo que evita ataques físicos de tipo "man-in-the-middle" o de supervisión y escucha telefónica. Dado que esta tecnología se integra en el propio hardware de red, proporciona cifrado de velocidad de línea en el hardware de red sin aumento de la latencia de vínculo mensurable. Este cifrado de MACsec está activado de forma predeterminada para todo el tráfico de Azure que viaja dentro de una región o entre regiones, y no se requiere ninguna acción por parte de los clientes para su habilitación.

## <a name="next-steps"></a>Pasos siguientes
Aprenda cómo se [usa el cifrado en Azure](encryption-overview.md).
