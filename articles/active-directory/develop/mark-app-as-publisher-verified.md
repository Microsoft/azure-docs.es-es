---
title: 'Marcación de una aplicación como comprobada por el publicador: plataforma de identidad de Microsoft | Azure'
description: Describe cómo marcar una aplicación como marcada por el publicador. Cuando una aplicación está marcada como comprobada por el publicador, significa que el publicador ha comprobado su identidad con una cuenta de Microsoft Partner Network que ha completado el proceso de verificación y ha asociado esta cuenta de MPN con el registro de aplicación.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 05/08/2020
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: jesakowi
ms.openlocfilehash: acb4b601118b341d14bc5e7c549d22eef23b2cc2
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90085712"
---
# <a name="mark-your-app-as-publisher-verified"></a>Marcación de la aplicación como comprobada por el publicador

Cuando una aplicación se marca como comprobada por el publicador, significa que el publicador ha comprobado su identidad con la cuenta de Microsoft Partner Network (MPN) y ha asociado esta cuenta de MPN con el registro de aplicación. En este artículo se describe cómo realizar el proceso de [verificación del editor](publisher-verification-overview.md).

## <a name="quickstart"></a>Guía de inicio rápido
Si ya está inscrito en Microsoft Partner Network (MPN) y cumple los [requisitos previos](publisher-verification-overview.md#requirements), puede empezar inmediatamente: 

1. Inicie sesión en el [portal de registro de aplicaciones](https://aka.ms/PublisherVerificationPreview) mediante la [autenticación multifactor](../fundamentals/concept-fundamentals-mfa-get-started.md).

1. Elija una aplicación y haga clic en **Personalización de marca**. 

1. Haga clic en **Add MPN ID to verify publisher** (Agregar id. de MPN para comprobación del publicador) de publicador y revise los requisitos enumerados.

1. Escriba el id. de MPN y haga clic en **Comprobar y guardar**.

Para obtener más información sobre ventajas específicas, requisitos y preguntas más frecuentes, consulte la [información general](publisher-verification-overview.md).


## <a name="mark-your-app-as-publisher-verified"></a>Marcación de la aplicación como comprobada por el publicador
Asegúrese de que ha cumplido los [requisitos previos](publisher-verification-overview.md#requirements) y, después, siga estos pasos para marcar las aplicaciones como comprobadas por el publicador.  

1. Asegúrese de haber iniciado sesión mediante la [autenticación multifactor](../fundamentals/concept-fundamentals-mfa-get-started.md) con una cuenta de organización (Azure AD) que esté autorizada para realizar cambios en las aplicaciones que quiera marcar como verificadas por el editor y en la cuenta de MPN del Centro de partners.

    - En Azure AD, este usuario debe ser miembro de alguno de los siguientes roles: Administrador de aplicaciones, administrador de aplicaciones en la nube o administrador global. 

    - En el Centro de partners, este usuario debe tener uno de los siguientes roles: Administrador de MPN, administrador de cuentas o administrador global (se trata de un rol compartido que se controla en Azure AD). 

1. Vaya al portal de registro de aplicaciones:  

1. Haga clic en una aplicación que quiera marcar como comprobada por el publicador y abra la hoja Personalización de marca. 

1. Asegúrese de que el [dominio del editor](howto-configure-publisher-domain.md) de la aplicación esté establecido. 

1. Asegúrese de que el dominio del editor o un [dominio personalizado](../fundamentals/add-custom-domain.md) verificado por DNS en el inquilino coincida con el dominio de la dirección de correo electrónico usada durante el proceso de verificación de la cuenta de MPN.

1. Haga clic en **Add MPN ID to verify publisher** (Agregar id. de MPN para comprobación del publicador) casi al final de la página. 

1. Escriba el **id. de MPN**. Este id. de MPN debe ser: 

    - De una cuenta de Microsoft Partner Network válida que haya completado el proceso de comprobación.  

    - De la cuenta global de asociado (PGA) de su organización. 

1. Haga clic en **Comprobar y guardar**. 

1. Espere a que se procese la solicitud; esta operación puede tardar unos minutos. 

1. Si la comprobación se realizó correctamente, se cerrará la ventana de comprobación del publicador y volverá a la hoja Personalización de marca. Verá un distintivo de verificación azul junto al **nombre para mostrar del publicador** verificado. 

1. Los usuarios que soliciten su consentimiento a su aplicación comenzarán a ver el distintivo poco después de terminar el proceso correctamente, aunque puede que tarde algún tiempo en replicarse en todo el sistema. 

1. Para probar esta funcionalidad, inicie sesión en la aplicación y asegúrese de que el distintivo de verificación aparece en la pantalla de consentimiento. Si ha iniciado sesión como un usuario que ya ha concedido consentimiento a la aplicación, puede usar el parámetro de consulta *prompt=consent* para forzar una petición de consentimiento. Este parámetro debe usarse solo para realizar pruebas, nunca se puede codificar de forma rígida en las solicitudes de la aplicación.

1. Repita este proceso según sea necesario para cualquier aplicación adicional para la que quiera que se muestre el distintivo. Puede usar Microsoft Graph para hacerlo más rápidamente en masa y los cmdlets de PowerShell pronto estarán disponibles. Consulte [Realización de llamadas a la API de Microsoft Graph](troubleshoot-publisher-verification.md#making-microsoft-graph-api-calls) para obtener más información. 

¡Ya está! Háganoslo saber si tiene algún comentario sobre el proceso, los resultados o la característica en general. 

## <a name="next-steps"></a>Pasos siguientes
Si experimenta problemas, lea la [información de solución de problemas](troubleshoot-publisher-verification.md).
