---
title: Configuración de un nombre de dominio personalizado para la instancia de Azure API Management
titleSuffix: Azure API Management
description: En este tema, se explica cómo configurar un nombre de dominio personalizado para la instancia de Azure API Management.
services: api-management
documentationcenter: ''
author: vladvino
manager: anneta
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: article
ms.date: 01/13/2020
ms.author: apimpm
ms.openlocfilehash: 53442f0727544221f28bfdb58d8069163eb374e5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87024867"
---
# <a name="configure-a-custom-domain-name-for-your-azure-api-management-instance"></a>Configuración de un nombre de dominio personalizado para la instancia de Azure API Management

Cuando se crea una instancia del servicio Azure API Management, Azure le asigna un subdominio de `azure-api.net` (por ejemplo, `apim-service-name.azure-api.net`). Sin embargo, los puntos de conexión de APIM se pueden exponer con su propio nombre de dominio personalizado, como **contoso.com**. En este tutorial se muestra cómo asignar un nombre DNS personalizado existente a los puntos de conexión expuestos por una instancia de API Management.

> [!IMPORTANT]
> API Management solo acepta solicitudes con valores de [encabezado de host](https://tools.ietf.org/html/rfc2616#section-14.23) que coinciden con el nombre de dominio predeterminado o con cualquiera de los nombres de dominio personalizados configurados.

> [!WARNING]
> Los clientes que quieran usar la asignación de certificados para mejorar la seguridad de sus aplicaciones deben usar un nombre de dominio personalizado y el certificado que administran, no el predeterminado. Los clientes que asignen el certificado predeterminado en su lugar tendrán una gran dependencia de las propiedades del certificado que no controlen, lo que no es un procedimiento recomendado.

## <a name="prerequisites"></a>Prerrequisitos

Para seguir los pasos que se describen en este artículo, debe tener:

-   Una suscripción de Azure activa.

    [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

-   Una instancia de API Management Para más información, vea [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).
-   Nombre de dominio personalizado que propiedad de su organización o suyo. En este tema no se dan instrucciones para adquirir un nombre de dominio personalizado.
-   Un registro CNAME hospedado en un servidor DNS que asigna el nombre de dominio personalizado al nombre de dominio predeterminado de una instancia de API Management. En este tema no se dan instrucciones para hospedar un registro CNAME.
-   Debe tener un certificado válido con una clave pública y privada (. PFX). El firmante o el nombre alternativo del firmante (SAN) debe coincidir con el nombre de dominio. De este modo, API Management puede exponer de forma segura direcciones URL mediante TLS.

## <a name="use-the-azure-portal-to-set-a-custom-domain-name"></a>Uso de Azure Portal para configurar un nombre de dominio personalizado

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com/).
1. Seleccione **Dominios personalizados**.

    Hay varios puntos de conexión a los que puede asignar un nombre de dominio personalizado. En la actualidad, están disponibles los siguientes:

    - **Puerta de enlace** (el valor predeterminado es `<apim-service-name>.azure-api.net`),
    - **Portal** (el valor predeterminado es `<apim-service-name>.portal.azure-api.net`),
    - **Management** (el valor predeterminado es `<apim-service-name>.management.azure-api.net`),
    - **SCM** (el valor predeterminado es `<apim-service-name>.scm.azure-api.net`),
    - **NewPortal** (el valor predeterminado es `<apim-service-name>.developer.azure-api.net`).

    > [!NOTE]
    > Solo el punto de conexión **Puerta de enlace** está disponible para la configuración en el nivel Consumo.
    > Puede actualizar algunos o todos los puntos de conexión. Por lo general, los clientes actualizan **Puerta de enlace** (esta dirección URL se utiliza para llamar a la API expuesta a través de API Management) y **Portal** (dirección URL del portal del desarrollador).
    > Los puntos de conexión **Management** y **SCM** los usan internamente los propietarios de la instancia de API Management y, por tanto, se les asigna con menor frecuencia un nombre de dominio personalizado.
    > El nivel **Premium** admite la configuración de varios nombres de host para el punto de conexión **Puerta de enlace**.

1. Seleccione el punto de conexión que desee actualizar.
1. En la ventana de la derecha, haga clic en **Personalizar**.

    - En **Nombre de dominio personalizado**, especifique el nombre que desee usar. Por ejemplo, `api.contoso.com`.
    - En **Certificado**, seleccione un certificado de Key Vault. También puede cargar un archivo .PFX válido y proporcionar su **contraseña**, si el certificado está protegido con una contraseña.

    > [!NOTE]
    > Los nombres de dominio con caracteres comodín, como por ejemplo,`*.contoso.com`, se admiten en todos los niveles, salvo en el nivel de consumo.

    > [!TIP]
    > Se recomienda usar [Azure Key Vault para administrar certificados](../key-vault/certificates/about-certificates.md) y configurarlos para la renovación automática.
    > Si usa Azure Key Vault para administrar el certificado TLS/SSL de dominio personalizado, asegúrese de que el certificado se inserta en Key Vault [como un _certificado_](/rest/api/keyvault/createcertificate/createcertificate), no como un _secreto_.
    >
    > Para capturar un certificado TLS/SSL, API Management debe tener la lista y obtener los permisos de secretos en la instancia de Azure Key Vault que contiene el certificado. Cuando se usa Azure Portal, todos los pasos de configuración necesarios se realizarán automáticamente. Cuando se usan las herramientas de línea de comandos o API Management, estos permisos se deben conceder manualmente. Para ello, debe realizar dos pasos. En primer lugar, use la página de identidades administradas de la instancia de API Management para asegurarse de que está habilitada la identidad administrada, y anote el identificador de la entidad que se muestra en esa página. En segundo lugar, proporcione la lista de permisos y obtenga los permisos de secretos de este identificador de entidad en la instancia de Azure Key Vault que contiene el certificado.
    >
    > Si el certificado está establecido para renovarse automáticamente, API Management seleccionará automáticamente la versión más reciente sin tiempo de inactividad para el servicio (si el plan de API Management tiene SLA en todos los planes, excepto el plan de desarrollador).

1. Haga clic en Aplicar.

    > [!NOTE]
    > El proceso de asignación del certificado puede tardar unos 15 minutos o más según el tamaño de la implementación. La SKU de desarrollo tiene un tiempo de inactividad, mientras que las SKU básica y superiores no lo tienen.

[!INCLUDE [api-management-custom-domain](../../includes/api-management-custom-domain.md)]

## <a name="dns-configuration"></a>Configuración de DNS

Al configurar DNS para el nombre de dominio personalizado, tiene dos opciones:

-   Configurar un registro CNAME que apunte al punto de conexión del nombre de dominio personalizado configurado.
-   Configurar un registro A que apunte a la dirección IP de la puerta de enlace de API Management.

> [!NOTE]
> Si bien la dirección IP de la instancia de API Management es estática, es posible que cambien en algunos escenarios. Debido a esto, se recomienda usar CNAME al configurar un dominio personalizado. Tenga en cuenta eso cuando elija el método de configuración de DNS. Más información en el [artículo de la documentación de IP](api-management-howto-ip-addresses.md#changes-to-the-ip-addresses) y las [Preguntas más frecuentes sobre API Management](api-management-faq.md#how-can-i-secure-the-connection-between-the-api-management-gateway-and-my-back-end-services).

## <a name="next-steps"></a>Pasos siguientes

[Actualización y escalado del servicio](upgrade-and-scale.md)
