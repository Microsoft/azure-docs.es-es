---
author: memildin
ms.author: memildin
manager: rkarlin
ms.date: 06/30/2020
ms.topic: include
ms.openlocfilehash: 1b650fa5a0e9ba2f7019e6e67690d9d1fd65e72a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90894918"
---
Security Center proporciona protección contra amenazas en tiempo real para los entornos en contenedores y genera alertas de actividades sospechosas. Puede usar esta información para corregir problemas de seguridad y mejorar la seguridad de los contenedores rápidamente.

Security Center proporciona protección contra amenazas en diferentes niveles: 

* **Nivel de host (proporcionado por Azure Defender para servidores)** : con el mismo agente de Log Analytics que Security Center usa en otras máquinas virtuales, Azure Defender supervisa los nodos de AKS de Linux en busca de actividades sospechosas, como la detección de shell web y la conexión con direcciones IP sospechosas conocidas. El agente también supervisa análisis específicos de contenedores, como la creación de contenedores con privilegios, las actividades sospechosas de acceso a los servidores de API y los servidores de Secure Shell (SSH) que se ejecutan dentro de un contenedor de Docker.

    >[!IMPORTANT]
    > Si decide no instalar los agentes en los hosts, solo recibirá un subconjunto de las ventajas de la protección contra amenazas y las alertas de seguridad. Aún así, seguirá recibiendo alertas relacionadas con el análisis de redes y las comunicaciones con servidores malintencionados.

    Para obtener una lista de las alertas de nivel de host de AKS, consulte la [Tabla de referencia de alertas](https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-containerhost).


* **Nivel de clúster de AKS (proporcionado por Azure Defender para Kubernetes)** : en el nivel de clúster, la protección contra amenazas se basa en analizar los registros de auditoría de Kubernetes. Para habilitar esta supervisión **sin agente**, habilite Azure Defender. Para generar alertas en este nivel, Security Center supervisa los servicios que administra AKS con los registros que recupera el mismo AKS. Entre los ejemplos de eventos en este nivel se incluyen los paneles de Kubernetes expuestos y la creación de roles con privilegios elevados y de montajes confidenciales.

    >[!NOTE]
    > Igualmente, Security Center genera alertas de seguridad para las acciones y las implementaciones de Azure Kubernetes Service que se producen después de que la opción de Kubernetes esté habilitada en la configuración de la suscripción. 

    Para obtener una lista de las alertas de nivel de clúster de AKS, consulte la [Tabla de referencia de alertas](https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-akscluster).

Además, nuestro equipo global de investigadores de seguridad supervisa constantemente el panorama de las amenazas. Agregan alertas específicas del contenedor y vulnerabilidades a medida que se detectan.

> [!TIP]
> Puede simular las alertas de contenedor al seguir las instrucciones de [esta entrada de blog](https://techcommunity.microsoft.com/t5/azure-security-center/how-to-demonstrate-the-new-containers-features-in-azure-security/ba-p/1011270).