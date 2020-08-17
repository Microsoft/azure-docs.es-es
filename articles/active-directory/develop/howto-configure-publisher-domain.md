---
title: Configuración del dominio de editor de una aplicación | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo configurar el dominio de editor de una aplicación para que los usuarios sepan dónde se envía su información.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 07/23/2020
ms.author: ryanwi
ms.reviewer: lenalepa, sureshja, zachowd
ms.custom: aaddev
ms.openlocfilehash: d62328534157a1e2d9e53830be9d6b87e08c9e40
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87129277"
---
# <a name="how-to-configure-an-applications-publisher-domain"></a>Procedimientos: Configuración del dominio de editor de una aplicación

El dominio de editor de una aplicación se muestra a los usuarios en la [petición de consentimiento de la aplicación](application-consent-experience.md) para que los usuarios sepan dónde se envía su información. Las aplicaciones multiinquilino registradas después del 21 de mayo de 2019 que no tengan un dominio de editor se muestran como **no verificadas**. Las aplicaciones multiinquilino son aquellas que admiten cuentas ajenas a un único directorio organizativo; por ejemplo, admiten todas las cuentas de Azure AD o admiten todas las cuentas de Azure AD y las cuentas personales de Microsoft.

## <a name="new-applications"></a>Aplicaciones nuevas

Al registrar una aplicación nueva, el dominio de editor de la aplicación puede establecerse en un valor predeterminado. El valor depende de dónde esté registrada la aplicación, especialmente si esta está registrada en un inquilino y si el inquilino tiene dominios que haya verificado.

Si hay dominios verificados por el inquilino, el dominio de editor de la aplicación se configurará de manera predeterminada en el dominio verificado principal del inquilino. Si no hay dominios verificados por el inquilino (que es lo que sucede cuando la aplicación no está registrada en un inquilino), el dominio de editor de la aplicación se establecerá en null.

En la tabla siguiente se resume el comportamiento predeterminado del valor del dominio de editor.  

| Dominios verificados por el inquilino | Valor predeterminado de dominio de editor |
|-------------------------|----------------------------|
| null | null |
| *.onmicrosoft.com | *.onmicrosoft.com |
| - *.onmicrosoft.com<br/>- domain1.com<br/>- domain2.com (primary) | domain2.com |

Si no se ha establecido el dominio de editor de una aplicación multiinquilino, o si se establece en un dominio que termina en .onmicrosoft.com, en la petición de consentimiento de la aplicación se mostrará el texto **no verificado** en lugar del dominio de editor.

## <a name="grandfathered-applications"></a>Aplicaciones con derechos por antigüedad

Si su aplicación se ha registrado antes del 21 de mayo de 2019, en la petición de consentimiento de la aplicación no se mostrará el texto **no verificado** si no ha establecido un dominio de editor. Le recomendamos que establezca el valor del dominio de editor para que los usuarios puedan ver esta información en la petición de consentimiento de la aplicación.

## <a name="configure-publisher-domain-using-the-azure-portal"></a>Configuración del dominio de editor mediante Azure Portal

Para establecer el dominio de editor de la aplicación, siga estos pasos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.

1. Si su cuenta está presente en más de un inquilino de Azure AD:
   1. Seleccione su perfil en el menú en la esquina superior derecha de la página y, a continuación, **Cambiar directorio**.
   1. Cambie la sesión al inquilino de Azure AD en el que desea crear la aplicación.

1. Vaya a [Azure Active Directory > Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) para buscar y seleccionar la aplicación que quiere configurar.

   Cuando haya seleccionado la aplicación, verá la página **Introducción** de la aplicación.

1. En la página **Introducción** de la aplicación, seleccione la sección **Personalización de marca**.

1. Busque el campo **Dominio de editor** y seleccione una de las opciones siguientes:

   - Seleccione **Configurar un dominio** si aún no ha configurado un dominio.
   - Seleccione **Actualizar dominio** si ya ha configurado un dominio.

Si la aplicación está registrada en un inquilino, verá dos pestañas en que podrá seleccionar entre: **Seleccionar un dominio verificado** y **Verificar un dominio nuevo**.

Si la aplicación no está registrada en un inquilino, solo verá la opción para verificar un dominio nuevo para la aplicación.

### <a name="to-verify-a-new-domain-for-your-app"></a>Para verificar un dominio nuevo para la aplicación

1. Cree un archivo denominado `microsoft-identity-association.json` y pegue el siguiente fragmento de código JSON.

   ```json
   {
      "associatedApplications": [
         {
            "applicationId": "{YOUR-APP-ID-HERE}"
         },
         {
            "applicationId": "{YOUR-OTHER-APP-ID-HERE}"
         }
      ]
    }
   ```

1. Reemplace el marcador de posición *{YOUR-APP-ID-HERE}* por el identificador de aplicación (cliente) que corresponde a la aplicación.

1. Hospede el archivo en: `https://{YOUR-DOMAIN-HERE}.com/.well-known/microsoft-identity-association.json`. Reemplace el marcador de posición *{YOUR-DOMAIN-HERE}* para que coincida con el dominio verificado.

1. Haga clic en el botón **Verificar y guardar dominio**.

### <a name="to-select-a-verified-domain"></a>Para seleccionar un dominio verificado

- Si el inquilino ha verificado los dominios, seleccione uno de los dominios en la lista desplegable **Seleccionar un dominio verificado**.

>[!Note]
> El encabezado "Content-Type" esperado que se debe devolver es `application/json`. Es posible que reciba el error que se indica a continuación si usa cualquier otro elemento, como `application/json; charset=utf-8`. 
> 
>``` "Verification of publisher domain failed. Error getting JSON file from https:///.well-known/microsoft-identity-association. The server returned an unexpected content type header value. " ```
>

## <a name="implications-on-the-app-consent-prompt"></a>Implicaciones en la petición de consentimiento de la aplicación

Configurar el dominio de editor influye en lo que los usuarios ven en la petición de consentimiento de la aplicación. Para comprender completamente los componentes de la petición de consentimiento, consulte [Descripción de las experiencias de consentimiento de la aplicación](application-consent-experience.md).

En la tabla siguiente se describe el comportamiento de las aplicaciones creadas antes del 21 de mayo de 2019.

![Petición de consentimiento para aplicaciones creadas antes del 21 de mayo de 2019](./media/howto-configure-publisher-domain/old-app-behavior-table.png)

El comportamiento de las aplicaciones nuevas creadas después del 21 de mayo de 2019 dependerá del dominio de editor y del tipo de aplicación. En la tabla siguiente se describen los cambios que debe esperar ver con las distintas combinaciones de configuraciones.

![Petición de consentimiento para aplicaciones creadas después del 21 de mayo de 2019](./media/howto-configure-publisher-domain/new-app-behavior-table.png)

## <a name="implications-on-redirect-uris"></a>Implicaciones en los URI de redireccionamiento

Las aplicaciones en que inician sesión los usuarios con cualquier cuenta profesional o educativa o con cuentas personales de Microsoft ([multiinquilino](single-and-multi-tenant-apps.md)) están sujetas a algunas restricciones al especificar URI de redireccionamiento.

### <a name="single-root-domain-restriction"></a>Restricción de dominio de raíz único

Cuando se establece en null el valor del dominio de editor para aplicaciones multiinquilino, las aplicaciones están restringidas a compartir un dominio de raíz único para los URI de redireccionamiento. Por ejemplo, la siguiente combinación de valores no se permite porque el dominio raíz, contoso.com, no coincide con fabrikam.com.

```
"https://contoso.com",
"https://fabrikam.com",
```

### <a name="subdomain-restrictions"></a>Restricciones de subdominio

Se permiten los subdominios, pero debe registrar explícitamente el dominio raíz. Por ejemplo, si bien los URI siguientes comparten un dominio de raíz único, no se permite la combinación.

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

Los casos siguientes no están sujetos a la restricción de dominio de raíz único:

- Aplicaciones de inquilino único o destinadas a cuentas de un único directorio
- Uso de localhost como URI de redireccionamiento
- URI de redireccionamiento con esquemas personalizados (no HTTP o HTTPS)

## <a name="configure-publisher-domain-programmatically"></a>Configuración el dominio de editor mediante programación

Actualmente, no se admiten API de REST ni PowerShell para configurar el dominio de editor mediante programación.
