---
title: 'Azure Defender para Key Vault: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para Key Vault.
author: memildin
ms.author: memildin
ms.date: 9/12/2020
ms.topic: conceptual
ms.service: security-center
ms.custom: references_regions
manager: rkarlin
ms.openlocfilehash: 06818e443568918e2ee87bbfbec81836ea85648b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932369"
---
# <a name="introduction-to-azure-defender-for-key-vault"></a>Introducción a Azure Defender para Key Vault

Azure Key Vault es un servicio en la nube que protege las claves de cifrado y los secretos, como certificados, cadenas de conexión y contraseñas. 

Habilite **Azure Defender para Key Vault** para la protección frente a amenazas avanzada, nativa de Azure, para Azure Key Vault, que proporciona una capa adicional de inteligencia de seguridad. 

## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponible con carácter general|
|Precios:|**Azure Defender para Key Vault** se factura como se muestra en la [página de precios](security-center-pricing.md).|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||

## <a name="what-are-the-benefits-of-azure-defender-for-key-vault"></a>¿Cuáles son las ventajas de Azure Defender para Key Vault?

Azure Defender detecta intentos inusuales y potencialmente perjudiciales de acceder o explotar las cuentas de Key Vault. Esta capa de protección le permite afrontar las amenazas sin ser un experto en seguridad y sin la necesidad de administrar sistemas de supervisión de la seguridad de terceros.  

Cuando se producen actividades anómalas, Azure Defender muestra alertas y, opcionalmente, las envía por correo electrónico a los miembros pertinentes de la organización. Dichas alertas incluyen detalles acerca de cualquier actividad sospechosa y recomendaciones acerca de cómo investigar amenazas y mitigarlas. 

## <a name="azure-defender-for-key-vault-alerts"></a>Alertas de Azure Defender para Key Vault
Cuando reciba una alerta de Azure Defender para Key Vault, se recomienda investigar y responder a la alerta, tal y como se describe en [Respuesta a las alertas de Azure Defender para Key Vault](defender-for-key-vault-usage.md). Azure Defender para Key Vault protege las aplicaciones y las credenciales, por lo que, incluso si está familiarizado con la aplicación o el usuario que desencadenó la alerta, es importante comprobar la situación relativa a cada alerta.

Las alertas aparecen en la página **Seguridad** de Key Vault, el panel de Azure Defender y la página de alertas de Security Center.

:::image type="content" source="./media/defender-for-key-vault-intro/key-vault-security-page.png" alt-text="Página de seguridad de Azure Key Vault":::

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre Azure Defender para Key Vault.

Para obtener material relacionado, consulte los siguientes artículos: 

- [Alertas de seguridad de Key Vault](alerts-reference.md#alerts-azurekv): la sección de Key Vault de la tabla de referencia para todas las alertas de Azure Security Center
- [Exportación de alertas a un SIEM](continuous-export.md)
- [Eliminación de alertas de Azure Defender](alerts-suppression-rules.md)