---
title: Configurar el dominio del publicador de la aplicación | Azure
description: Obtenga información sobre cómo configurar el dominio del publicador de la aplicación para que los usuarios sepan dónde se envían su información.
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/05/2019
ms.author: ryanwi
ms.reviewer: lenalepa, sureshja, zachowd
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: d47075f9e18b299341a98983ffb8a47389fd7063
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540237"
---
# <a name="how-to-configure-an-applications-publisher-domain-preview"></a>Procedimientos para: Configurar el dominio del publicador de la aplicación (versión preliminar)

Dominio del publicador de la aplicación se muestra a los usuarios en el [de consentimiento de la aplicación](application-consent-experience.md) para que los usuarios sepan dónde se envían su información. Las aplicaciones multiinquilino registradas después de 21 de mayo de 2019 que no tienen un dominio de publicador se muestran como **sin comprobar**. Las aplicaciones de varios inquilinos son aplicaciones que admitan cuentas fuera de un único directorio de la organización; Por ejemplo, admite todas las cuentas de Azure AD, o admitir todas las cuentas de Azure AD y las cuentas personales de Microsoft.

## <a name="new-applications"></a>Nuevas aplicaciones

Al registrar una nueva aplicación, el dominio del publicador de la aplicación puede establecerse en un valor predeterminado. El valor depende de donde está registrada la aplicación, especialmente si la aplicación esté registrada en un inquilino y si el inquilino ha de inquilino de dominios comprobados.

Si hay dominios comprobados de inquilino, el dominio del publicador de la aplicación predeterminada será el principal dominio comprobado del inquilino. Si no hay ningún inquilino de dominios comprobados (que es el caso cuando la aplicación no está registrada en un inquilino), el dominio del publicador de la aplicación se establecerá en null.

En la tabla siguiente se resume el comportamiento predeterminado del valor del dominio de publicador.  

| Dominios comprobados de inquilino | Valor predeterminado de dominio del publicador |
|-------------------------|----------------------------|
| NULL | NULL |
| *.onmicrosoft.com | *.onmicrosoft.com |
| - *.onmicrosoft.com<br/>-domain1.com<br/>-domain2.com (principal) | domain2.com |

Si no se ha configurado el dominio del publicador de la aplicación de varios inquilinos, o si se establece en un dominio que termina en. onmicrosoft.com, se mostrará la petición de consentimiento de la aplicación **sin comprobar** en lugar del dominio del publicador.

## <a name="grandfathered-applications"></a>Aplicaciones de derechos por antigüedad

Si se registró la aplicación antes del 21 de mayo de 2019, no se mostrará el consentimiento de su aplicación **sin comprobar** si no ha configurado un dominio del publicador. Recomendamos que establezca al publicador de valor de dominio para que los usuarios pueden ver esta información en la petición de consentimiento de la aplicación.

## <a name="configure-publisher-domain-using-the-azure-portal"></a>Configurar el dominio del publicador mediante el portal de Azure

Para establecer el dominio del publicador de la aplicación, siga estos pasos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.

1. Si su cuenta está presente en más de un inquilino de Azure AD:
   1. Seleccione su perfil en el menú en la esquina superior derecha de la página y, a continuación, **Cambiar directorio**.
   1. Cambiar la sesión para el inquilino de Azure AD donde desea crear la aplicación.

1. Vaya a [Azure Active Directory > registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) para buscar y seleccionar la aplicación que desea configurar.

   Una vez que haya seleccionado la aplicación, verá la aplicación **Introducción** página.

1. Desde la aplicación **Introducción** página, seleccione el **Branding** sección.

1. Buscar el **dominio del publicador** campo y seleccione una de las siguientes opciones:

   - Seleccione **configurar un dominio** si aún no ha configurado un dominio ya.
   - Seleccione **Actualizar dominio** si ya se ha configurado un dominio.

Si la aplicación se registra en un inquilino, verá dos fichas para seleccionar entre: **Seleccione un dominio comprobado** y **comprobar un dominio nuevo**.

Si la aplicación no está registrada en un inquilino, solo verá la opción para comprobar un dominio nuevo para la aplicación.

### <a name="to-verify-a-new-domain-for-your-app"></a>Para comprobar un dominio nuevo para la aplicación

1. Cree un archivo denominado `microsoft-identity-association.json` y pegue el siguiente fragmento de código JSON.

   ```json
   {
      "associatedApplications": [
        {
           "applicationId": "{YOUR-APP-ID-HERE}"
        }
      ]
    }
   ```

1. Reemplace el marcador de posición *{YOUR-APP-ID-HERE}* con el identificador de aplicación (cliente) que corresponde a la aplicación.

1. Hospedar el archivo: `https://{YOUR-DOMAIN-HERE}.com/.well-known/microsoft-identity-association.json`. Reemplace el marcador de posición *{YOUR-DOMAIN-HERE}* para que coincida con el dominio comprobado.

1. Haga clic en el **comprobar y guardar dominio** botón.

### <a name="to-select-a-verified-domain"></a>Para seleccionar un dominio comprobado

- Si el inquilino ha comprobado los dominios, seleccione uno de los dominios de la **seleccione un dominio comprobado** lista desplegable.

## <a name="implications-on-the-app-consent-prompt"></a>Implicaciones en la aplicación de consentimiento

Configuración del dominio del publicador tiene un impacto en lo que ven los usuarios en la petición de consentimiento de aplicación. Para comprender los componentes de la petición de consentimiento, consulte [comprender el consentimiento de la aplicación experimenta](application-consent-experience.md).

En la tabla siguiente se describe el comportamiento de las aplicaciones creadas antes del 21 de mayo de 2019.

![Petición de consentimiento para aplicaciones creadas antes del 21 de mayo de 2019](./media/howto-configure-publisher-domain/old-app-behavior-table.png)

El comportamiento de las nuevas aplicaciones creadas después de 21 de mayo de 2019 dependerá del dominio del publicador y el tipo de aplicación. En la tabla siguiente se describe los cambios que se debe esperar para ver con las distintas combinaciones de configuraciones.

![Petición de consentimiento para aplicaciones creadas después de 21 de mayo de 2019](./media/howto-configure-publisher-domain/new-app-behavior-table.png)

## <a name="implications-on-redirect-uris"></a>Implicaciones en los URI de redireccionamiento

Las aplicaciones que inician sesión en los usuarios con cualquier cuenta profesional o educativa o cuentas personales de Microsoft ([multiinquilino](single-and-multi-tenant-apps.md)) están sujetos a algunas restricciones al especificar URI de redireccionamiento.

### <a name="single-root-domain-restriction"></a>Restricción de dominio de raíz única

Cuando se establece el valor de dominio del publicador para aplicaciones de varios inquilinos a las aplicaciones es nulos, están restringidas a compartir un único dominio raíz para los URI de redireccionamiento. Por ejemplo, la siguiente combinación de valores no se permite porque el dominio raíz, contoso.com, no coincide con fabrikam.com.

```
"https://contoso.com",
"https://fabrikam.com",
```

### <a name="subdomain-restrictions"></a>Restricciones de subdominio

Se permiten los subdominios, pero debe registrar explícitamente el dominio raíz. Por ejemplo, si bien los URI siguientes comparten un único dominio raíz, no se permite la combinación.

```
"https://app1.contoso.com",
"https://app2.contoso.com",
```

Sin embargo, si el desarrollador agrega explícitamente el dominio raíz, se permite la combinación.

```
"https://contoso.com",
"https://app1.contoso.com",
"https://app2.contoso.com",
```

### <a name="exceptions"></a>Excepciones

Los casos siguientes no están sujetos a la restricción de dominio de raíz única:

- Aplicaciones de inquilino único o las aplicaciones destinadas a las cuentas en un único directorio
- Uso de localhost como URI de redireccionamiento
- URI de redireccionamiento con esquemas personalizados (que no sea HTTP o HTTPS)

## <a name="configure-publisher-domain-programmatically"></a>Configurar el dominio del publicador mediante programación

Actualmente, no hay ninguna compatibilidad de API de REST o PowerShell para configurar el dominio del publicador mediante programación.
