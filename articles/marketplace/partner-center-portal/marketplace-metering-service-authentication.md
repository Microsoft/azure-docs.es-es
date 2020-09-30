---
title: Estrategias de autenticación del servicio de medición de Marketplace | Azure Marketplace
description: Estrategias de autenticación del servicio de medición admitidas en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 05/21/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: e8f9a8e1d10e39e37480e06a25fcc0e203a104ec
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378736"
---
# <a name="marketplace-metering-service-authentication-strategies"></a>Estrategias de autenticación del servicio de medición de Marketplace

El servicio de medición de Marketplace admite dos estrategias de autenticación:

* [Token de seguridad de Azure AD](../../active-directory/develop/access-tokens.md)
* [Identidades administradas](../../active-directory/managed-identities-azure-resources/overview.md) 

Se explicará cuándo y cómo usar las diferentes estrategias de autenticación para enviar de forma segura medidores personalizados mediante el servicio de medición de Marketplace.

## <a name="using-the-azure-ad-security-token"></a>Usar el token de seguridad de Azure AD

Los tipos de oferta aplicables son aplicaciones comercializables SaaS y de Azure con el tipo de plan de aplicación administrada.  

Envíe medidores personalizados mediante un identificador de aplicación fijo de Azure AD predefinido para autenticarse.

En el caso de las ofertas de SaaS, esta es la única opción disponible. Es un paso obligatorio para publicar cualquier oferta de SaaS tal y como se describe en [Registro de una aplicación SaaS](./pc-saas-registration.md).

En el caso de las aplicaciones de Azure con un plan de aplicación administrado, debe considerar la posibilidad de usar esta estrategia en los casos siguientes:

* Ya tiene un mecanismo para comunicarse con los servicios back-end y desea extender este mecanismo para emitir medidores personalizados desde un servicio central.
* Dispone de lógica de medidores personalizados compleja.  Ejecute esta lógica en una ubicación central, en lugar de en los recursos de la aplicación administrada.

Una vez que se ha registrado la aplicación, puede solicitar un token de seguridad de Azure AD mediante programación. Se espera que el anunciante use este token y realice una solicitud para resolverlo.

Para más información sobre estos tokens, consulte [Tokens de acceso de Azure Active Directory](../../active-directory/develop/access-tokens.md).

### <a name="get-a-token-based-on-the-azure-ad-app"></a>Obtención de un token basado en la aplicación de Azure AD

#### <a name="http-method"></a>Método HTTP

**POST**

#### <a name="request-url"></a>*Request URL* (URL de la solicitud)

**`https://login.microsoftonline.com/*{tenantId}*/oauth2/token`**

#### <a name="uri-parameter"></a>*Parámetro de URI*

|  **Nombre de parámetro** |  **Obligatorio**  |  **Descripción**          |
|  ------------------ |--------------- | ------------------------  |
|  `tenantId`         |   True         | Identificador de inquilino de la aplicación de Azure AD registrada.   |
| | | |

#### <a name="request-header"></a>*Encabezado de solicitud*

|  **Nombre de encabezado**    |  **Obligatorio**  |  **Descripción**          |
|  ------------------ |--------------- | ------------------------  |
|  `Content-Type`     |   True         | Tipo de contenido asociado a la solicitud. El valor predeterminado es `application/x-www-form-urlencoded`.  |
| | | |

#### <a name="request-body"></a>*Cuerpo de la solicitud*

|  **Nombre de propiedad**  |  **Obligatorio**  |  **Descripción**          |
|  ------------------ |--------------- | ------------------------  |
|  `Grant_type`       |   True         | Tipo de concesión. Mediante `client_credentials`. |
|  `Client_id`        |   True         | Identificador de cliente o aplicación asociado a la aplicación de Azure AD.|
|  `client_secret`    |   True         | Secreto asociado a la aplicación de Azure AD.  |
|  `Resource`         |   True         | Recurso de destino para el que se solicita el token. Mediante `20e940b3-4c77-4b0b-9a53-9e16a1b010a7`. |
| | | |

#### <a name="response"></a>*Respuesta*

|  **Nombre**    |  **Tipo**  |  **Descripción**          |
|  ------------------ |--------------- | ----------------------  |
|  `200 OK`     |   `TokenResponse`    | La solicitud se realizó correctamente.  |
| | | |

#### <a name="tokenresponse"></a>*TokenResponse*

Ejemplo de token de respuesta:

```JSON
  {
      "token_type": "Bearer",
      "expires_in": "3600",
      "ext_expires_in": "0",
      "expires_on": "15251…",
      "not_before": "15251…",
      "resource": "20e940b3-4c77-4b0b-9a53-9e16a1b010a7",
      "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImlCakwxUmNxemhpeTRmcHhJeGRacW9oTTJZayIsImtpZCI6ImlCakwxUmNxemhpeTRmcHhJeGRacW9oTTJZayJ9…"
  }
```

## <a name="using-the-azure-managed-identities-token"></a>Uso del token de identidades administradas por Azure

El tipo de oferta aplicable son las aplicaciones de Azure con el tipo de plan de aplicación administrada.

El uso de este enfoque permitirá que la identidad de los recursos implementados se autentique para enviar eventos de uso de medidores personalizados.  Puede insertar el código que emite el uso dentro de los límites de la implementación.

>[!Note]
>El anunciante debe asegurarse de que los recursos que emiten el uso están bloqueados, para que no se puedan modificar.

La aplicación administrada puede contener diferentes tipos de recursos, desde Virtual Machines hasta Azure Functions.  Para más información sobre cómo autenticarse con identidades administradas en diferentes servicios, consulte [¿Cómo se usan las identidades administradas para recursos de Azure?](../../active-directory/managed-identities-azure-resources/overview.md#how-can-i-use-managed-identities-for-azure-resources).

Por ejemplo, siga los pasos que se indican a continuación para autenticarse con una máquina virtual Windows.

1. Asegúrese de que la identidad administrada está configurada mediante uno de estos métodos:
    * [Interfaz de usuario de Azure Portal](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
    * [CLI](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
    * [PowerShell](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
    * [Plantilla de Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
    * [REST](../../active-directory/managed-identities-azure-resources/qs-configure-rest-vm.md#system-assigned-managed-identity)
    * [SDK de Azure](../../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

1. Obtenga un token de acceso para el identificador de la aplicación del servicio de medición de Marketplace (`20e940b3-4c77-4b0b-9a53-9e16a1b010a7`) mediante la identidad del sistema, conéctese a la máquina virtual mediante RDP, abra la consola de PowerShell y ejecute el comando siguiente.

    ```powershell
    # curl is an alias to Web-Invoke PowerShell command
    # Get system identity access tokenn
    $MetadataUrl = "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F"
    $Token = curl -H @{"Metadata" = "true"} $MetadataUrl | Select-Object -Expand Content | ConvertFrom-Json
    $Headers = @{}
    $Headers.Add("Authorization","$($Token.token_type) "+ " " + "$($Token.access_token)")
    ```

1. Obtenga el identificador de la aplicación administrada de la propiedad "ManagedBy" del grupo de recursos actual.

    ```powershell
    # Get subscription and resource group
    $metadata = curl -H @{'Metadata'='true'} http://169.254.169.254/metadata/instance?api-version=2019-06-01 | select -ExpandProperty Content | ConvertFrom-Json 
    
    # Make sure the system identity has at least reader permission on the resource group
    $managementUrl = "https://management.azure.com/subscriptions/" + $metadata.compute.subscriptionId + "/resourceGroups/" + $metadata.compute.resourceGroupName + "?api-version=2019-10-01"
    $resourceGroupInfo = curl -Headers $Headers $managementUrl | select -ExpandProperty Content | ConvertFrom-Json
    $managedappId = $resourceGroupInfo.managedBy 
    ```

1. El servicio de medición de Marketplace necesita notificar el uso mediante `resourceID` y `resourceUsageId` si se trata de una aplicación administrada.

    ```powershell
    # Get resourceUsageId from the managed app
    $managedAppUrl = "https://management.azure.com/subscriptions/" + $metadata.compute.subscriptionId + "/resourceGroups/" + $metadata.compute.resourceGroupName + "/providers/Microsoft.Solutions/applications/" + $managedappId + "\?api-version=2019-07-01"
    $ManagedApp = curl $managedAppUrl -H $Headers | Select-Object -Expand Content | ConvertFrom-Json
    # Use this resource ID to emit usage 
    $resourceUsageId = $ManagedApp.properties.billingDetails.resourceUsageId
    ```

1. Utilice la [API del servicio de medición de Marketplace](./marketplace-metering-service-apis.md) para emitir el uso.

## <a name="next-steps"></a>Pasos siguientes

* [Creación de una oferta de aplicación de Azure](./create-new-azure-apps-offer.md)
* [Planificación de una oferta de SaaS](../plan-saas-offer.md)