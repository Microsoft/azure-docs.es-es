---
title: 'Azure Defender para App Service: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para App Service.
author: memildin
ms.author: memildin
ms.date: 9/22/2020
ms.topic: conceptual
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: d15e881ca1246d463d8054fe5bd094efaff0cdd5
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90977167"
---
# <a name="introduction-to-azure-defender-for-app-service"></a>Introducción a Azure Defender para App Service

Azure App Service es una plataforma totalmente administrada para crear y hospedar aplicaciones web y API sin preocuparse por tener que administrar la infraestructura. Proporciona administración, supervisión y conclusiones operativas para satisfacer requisitos de seguridad, cumplimiento normativo y rendimiento de nivel empresarial. Para más información, consulte [Azure App Service](https://azure.microsoft.com/services/app-service/).

**Azure Defender para App Service** usa la escala de la nube para identificar ataques dirigidos a aplicaciones que se ejecutan mediante App Service. Los atacantes sondean las aplicaciones web para buscar y aprovechar los puntos débiles. Antes de enrutarse a entornos específicos, las solicitudes para las aplicaciones que se ejecutan en Azure atraviesan varias puertas de enlace donde se las inspecciona y registra. A continuación, estos datos se usan para identificar vulnerabilidades y atacantes, así como para aprender nuevos patrones que se usarán más adelante.

Al usar la visibilidad que Azure tiene como proveedor de nube, Security Center analiza los registros internos de App Service para metodología de ataques en múltiples destinos, como, por ejemplo, la metodología incluye el análisis generalizado y los ataques distribuidos. Este tipo de ataque normalmente procede de un pequeño subconjunto de direcciones IP y muestra patrones de rastreo a puntos de conexión similares en varios hosts. Los ataques buscan una página o complemento vulnerables y no se pueden identificar desde el punto de vista de un solo host.


## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Versión preliminar|
|Precios:|[Azure Defender para App Service](azure-defender.md) se factura como se muestra en la [página de precios](security-center-pricing.md).|
|Planes de App Service admitidos:|![Sí](./media/icons/yes-icon.png) Básico, Estándar, Premium, Aislado o Linux<br>![No](./media/icons/no-icon.png) Gratis, Compartido ni Consumo<br>[Obtenga más información sobre los planes de App Service](https://azure.microsoft.com/pricing/details/app-service/plans/).|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||

## <a name="what-does-azure-defender-for-app-service-protect"></a>¿Qué protege Azure Defender para App Service?

Con el plan de App Service habilitado, Security Center evalúa los recursos incluidos en el plan de App Service y genera recomendaciones de seguridad en función de sus hallazgos. Security Center protege la instancia de la VM en la que se está ejecutando la instancia de App Service y la interfaz de administración. También supervisa las solicitudes y respuestas enviadas a las aplicaciones que se ejecutan en App Service, y desde ellas.

Si está usando un plan de App Service basado en Windows, Security Center también tiene acceso a las máquinas virtuales y los espacios aislados subyacentes. Junto con los datos de registro mencionados anteriormente, la infraestructura puede contar lo que ocurre, desde un nuevo ataque que circula por la red hasta riesgos concretos para las máquinas de los clientes. Por lo tanto, incluso si se implementa Security Center una vez que se han aprovechado las vulnerabilidades de una aplicación web, es posible que pueda detectar los ataques en curso.


## <a name="protect-your-azure-app-service-web-apps-and-apis"></a>Protección de las aplicaciones web y API de Azure App Service
Para proteger el plan de Azure App Service con Azure Defender para App Service:

- Asegúrese de que tiene un plan de App Service compatible asociado a máquinas dedicadas. Los planes admitidos se enumeran anteriormente en [Disponibilidad](#availability).

- Habilite **Azure Defender** en su suscripción (opcionalmente, puede habilitar solo el plan **Azure Defender para App Service** ) como se describe en [Precios de Azure Security Center](security-center-pricing.md).

Security Center está integrado de forma nativa con App Service, lo que elimina la necesidad de implementación e incorporación; la integración es transparente.

>[!NOTE]
> En la página precios y configuración se muestran varias instancias de la **cantidad de recursos**. Esto representa el número total de instancias de proceso, en todos los planes de App Service en esta suscripción, en ejecución en el momento en que ha abierto la hoja del plan de tarifa.
>
> Azure App Service ofrece una variedad de planes. El plan de App Service define el conjunto de recursos de proceso para que una aplicación web se ejecute. Son equivalentes a las granjas de servidores en el hospedaje web convencional. Pueden configurarse una o varias aplicaciones para que se ejecuten en los mismos recursos informáticos (o en el mismo plan de App Service).
>
>Para validar el recuento, vaya a "planes de App Service" en Azure Portal, donde puede ver el número de instancias de proceso que se usa en cada plan. 



## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre Azure Defender para App Service. 

Para obtener material relacionado, consulte los siguientes artículos: 

- Tanto si Security Center genera una alerta, como si la recibe de un producto de seguridad diferente, puede exportarla. Para exportar las alertas a Azure Sentinel, a cualquier SIEM de terceros o a cualquier otra herramienta externa, siga las instrucciones de [Exportación de alertas a un SIEM](continuous-export.md).
- Para obtener una lista de las alertas de Azure App Service, consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-azureappserv).
- Para más información sobre los planes de App Service, consulte [Planes de App Service](https://azure.microsoft.com/pricing/details/app-service/plans/).
- > [!div class="nextstepaction"]
    > [Habilitación de Azure Defender](security-center-pricing.md)