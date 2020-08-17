---
title: Configuración de un nombre de dominio personalizado para su puerta de enlace autohospedada de API Management | Microsoft Docs
description: Este tema describe los pasos para configurar un nombre de dominio personalizado para la puerta de enlace autohospedada de Azure API Management.
services: api-management
documentationcenter: ''
author: vladvino
manager: gwallace
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: article
ms.date: 03/31/2020
ms.author: apimpm
ms.openlocfilehash: 0894203be4867e305c8e15467a2a867b9bfdc727
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86506815"
---
# <a name="configure-a-custom-domain-name"></a>Configurar un nombre de dominio personalizado

Al aprovisionar una [puerta de enlace autohospedada de Azure API Management](self-hosted-gateway-overview.md) no se le asigna un nombre de host y se debe hacer referencia a ella mediante su dirección IP. En este artículo se muestra cómo asignar un nombre DNS personalizado existente (también conocido como nombre de host) a una puerta de enlace autohospedada.

## <a name="prerequisites"></a>Prerrequisitos

Para seguir los pasos que se describen en este artículo, debe tener:

-   Una suscripción de Azure activa.

    [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

-   Una instancia de API Management Para más información, vea [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).
- Una puerta de enlace autohospedada. Para obtener más información, consulte [cómo aprovisionar una puerta de enlace autohospedada](api-management-howto-provision-self-hosted-gateway.md)
-   Nombre de dominio personalizado que propiedad de su organización o suyo. En este tema no se dan instrucciones para adquirir un nombre de dominio personalizado.
-   Un registro DNS hospedado en un servidor DNS que asigna el nombre de dominio personalizado a la dirección IP de la puerta de enlace autohospedada. En este tema no se dan instrucciones para hospedar un registro DNS.
-   Debe tener un certificado válido con una clave pública y privada (. PFX). El firmante o el nombre alternativo del firmante (SAN) debe coincidir con el nombre de dominio. De este modo, API Management puede exponer de forma segura direcciones URL mediante TLS.

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="add-custom-domain-certificate-to-your-api-management-service"></a>Incorporación de un certificado de dominio personalizado a su servicio de API Management

1. Seleccione **Certificados** en **Seguridad**.
2. Seleccione **+Agregar**.
3. Escriba un nombre de recurso para el certificado en el campo **ID.**
4. Seleccione el archivo que contiene el certificado (.PFX). Para ello, seleccione el campo de **Certificado** o el icono de carpeta adyacente.
5. Escriba la contraseña del certificado en el campo de **Contraseña**.
6. Seleccione **Crear** para agregar el certificado a su servicio de API Management.

## <a name="use-the-azure-portal-to-set-a-custom-domain-name-for-your-self-hosted-gateway"></a>Uso de Azure Portal para configurar un nombre de dominio personalizado para la puerta de enlace autohospedada

1. Seleccione las **Puertas de enlace** en **Implementación e infraestructura**.
2. Seleccione la puerta de enlace autohospedada para la que desea configurar el nombre de dominio.
3. Seleccione **nombres de host** en **Configuración**.
4. Seleccione **+ Agregar**.
5. Escriba el nombre de recurso del nombre de host en el campo de **Nombre**.
6. Escriba el nombre de dominio en el campo de **Nombre de host**.
7. Seleccione un certificado del menú desplegable **Certificado**.
8. Seleccione la casilla **Negociar certificado de cliente** si alguna de las API expuestas a través de esta puerta de enlace usa la autenticación de certificados de clientes.
    > [!WARNING]
    > Esta configuración se comparte con todos los nombres de dominio configurados para la puerta de enlace.
9. Seleccione **Agregar** para asignar el nombre de dominio personalizado a la puerta de enlace autohospedada seleccionada.

## <a name="next-steps"></a>Pasos siguientes

[Actualización y escalado del servicio](upgrade-and-scale.md)
