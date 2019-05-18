---
title: Terminación SSL con certificados de Azure Key Vault
description: Obtenga información sobre cómo integrar Azure Application Gateway con Key Vault para certificados de servidor que se adjuntan a los agentes de escucha habilitado para HTTPS.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 4/25/2019
ms.author: victorh
ms.openlocfilehash: 18af315c58c838a7237acfbcc32f622a0edbd3b3
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827640"
---
# <a name="ssl-termination-with-key-vault-certificates"></a>Terminación SSL con certificados de Key Vault

[Azure Key Vault](../key-vault/key-vault-whatis.md) es un secreto administrado plataforma almacenar que puede usar para proteger los secretos, claves y certificados SSL. Azure Application Gateway admite la integración con Key Vault (en versión preliminar pública) para los certificados de servidor que se adjuntan a los agentes de escucha HTTPS habilitado. Esta compatibilidad está limitada a v2 SKU de Application Gateway.

> [!IMPORTANT]
> Integración de Application Gateway con Key Vault está actualmente en versión preliminar pública. Esta versión preliminar se proporciona sin un contrato de nivel de servicio (SLA) y no se recomienda para las cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Esta versión preliminar pública ofrece dos modelos para la terminación SSL:

- Puede proporcionar explícitamente los certificados SSL asociados al agente de escucha. Este modelo es la forma tradicional para pasar los certificados SSL en Application Gateway para la terminación SSL.
- También puede proporcionar una referencia a un certificado de Key Vault existente o un secreto al crear un agente de escucha habilitado para HTTPS.

Integración de la puerta de enlace de aplicaciones con Key Vault ofrece muchas ventajas, como:

- Mayor seguridad, porque los certificados SSL no controlan directamente al equipo de desarrollo de aplicaciones. Integración permite que un equipo de seguridad independientes para:
  * Configurar las puertas de enlace de la aplicación.
  * Controlar los ciclos de vida de puerta de enlace de aplicaciones.
  * Conceder permisos a las puertas de enlace de la aplicación seleccionada para tener acceso a los certificados que se almacenan en el almacén de claves.
- Compatibilidad para la importación de certificados existentes en el almacén de claves. O use la API de Key Vault para crear y administrar certificados nuevos con cualquiera de los asociados de almacén de claves de confianza.
- Compatibilidad con la renovación automática de certificados que se almacenan en el almacén de claves.

Application Gateway admite actualmente sólo a certificados validados por software. Módulo de seguridad de hardware (HSM): no se admiten certificados validados. Después de Application Gateway está configurado para usar certificados de Key Vault, sus instancias de recuperar el certificado de Key Vault e instalación localmente para la terminación SSL. Además, las instancias sondean Key Vault a intervalos de 24 horas para recuperar una versión renovada del certificado, si existe. Si se encuentra un certificado actualizado, el certificado SSL asociado actualmente con el agente de escucha HTTPS se gira automáticamente.

## <a name="how-integration-works"></a>Cómo funciona la integración

Integración de la puerta de enlace de aplicaciones con Key Vault requiere un proceso de configuración de tres pasos:

1. **Cree una identidad administrada asignada por el usuario**

   Crear o volver a usar una asignada por el usuario identidad administrada existente, que Application Gateway se usa para recuperar los certificados de Key Vault en su nombre. Para más información, consulte [¿Qué es Managed Identities for Azure Resources?](../active-directory/managed-identities-azure-resources/overview.md) Este paso crea una nueva identidad en el inquilino de Azure Active Directory. La identidad es de confianza mediante la suscripción que se usa para crear la identidad.

1. **Configurar el almacén de claves**

   , A continuación, importe un certificado existente o crea uno nuevo en el almacén de claves. Las aplicaciones que se ejecutan a través de la puerta de enlace de la aplicación, se usará el certificado. En este paso, también puede usar un secreto de almacén de claves que se almacena como base, sin contraseña codificada en 64 archivo PFX. Se recomienda usar un tipo de certificado debido a la capacidad de renovación automática está disponible con los objetos de tipo de certificado en el almacén de claves. Después de crear un certificado o clave secreta, definir directivas de acceso en el almacén de claves para permitir que la identidad que se deben conceder *obtener* acceso al secreto.

1. **Configurar la puerta de enlace de aplicaciones**

   Después de completar los dos pasos anteriores, puede configurar o modificar una puerta de enlace de aplicaciones existente para usar la identidad administrada asignada por el usuario. También puede configurar el certificado SSL de la escucha HTTP para que apunte al URI completo del certificado de Key Vault o identificador de secreto.

   ![Certificados del almacén de claves](media/key-vault-certs/ag-kv.png)

## <a name="next-steps"></a>Pasos siguientes

[Configure la terminación SSL con certificados de Key Vault mediante Azure PowerShell](configure-keyvault-ps.md)
