---
title: Ejemplos de JavaScript
titleSuffix: Azure AD B2C
description: Obtenga información acerca de cómo usar JavaScript en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.custom: devx-track-javascript
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/10/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 947111edf7741f78a40b7d3e77a1dfae22383500
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87169574"
---
# <a name="javascript-samples-for-use-in-azure-active-directory-b2c"></a>Ejemplos de JavaScript para usar en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-public-preview](../../includes/active-directory-b2c-public-preview.md)]

Puede agregar su propio código cliente de JavaScript a las aplicaciones de Azure Active Directory B2C (Azure AD B2C).

Para habilitar JavaScript para las aplicaciones:

* Agregue un elemento a la [directiva personalizada](custom-policy-overview.md)
* Seleccione un [diseño de página](page-layout.md)
* Use [b2clogin.com](b2clogin.md) en las solicitudes

En este artículo se describe cómo se puede cambiar su directiva personalizada para habilitar la ejecución del script.

> [!NOTE]
> Si quiere habilitar JavaScript para los flujos de usuario, consulte [Versiones de diseño de página y JavaScript en Azure Active Directory B2C](user-flow-javascript-overview.md).

## <a name="prerequisites"></a>Prerrequisitos

### <a name="select-a-page-layout"></a>Selección de un diseño de página

* Seleccione un [diseño de página](contentdefinitions.md#select-a-page-layout) para los elementos de la interfaz de usuario de su aplicación.

    Si piensa usar JavaScript, tendrá que [definir un diseño de página](contentdefinitions.md#migrating-to-page-layout) con la versión `contract` de página para *todas* las definiciones de contenido de la directiva personalizada.

## <a name="add-the-scriptexecution-element"></a>Adición del elemento ScriptExecution

Habilite la ejecución del script agregando el elemento **ScriptExecution** al elemento [RelyingParty](relyingparty.md).

1. Abra el archivo de la directiva personalizada. Por ejemplo, *SignUpOrSignin.xml*.
2. Agregue el elemento **ScriptExecution** al elemento **UserJourneyBehaviors** de **RelyingParty**:

    ```xml
    <RelyingParty>
      <DefaultUserJourney ReferenceId="B2CSignUpOrSignInWithPassword" />
      <UserJourneyBehaviors>
        <ScriptExecution>Allow</ScriptExecution>
      </UserJourneyBehaviors>
      ...
    </RelyingParty>
    ```
3. Guarde y cargue el archivo.

[!INCLUDE [active-directory-b2c-javascript-guidelines](../../includes/active-directory-b2c-javascript-guidelines.md)]

## <a name="javascript-samples"></a>Ejemplos de JavaScript

### <a name="show-or-hide-a-password"></a>Mostrar u ocultar una contraseña

Una forma habitual de ayudar a sus clientes para que se registren correctamente es permitirles ver la contraseña que escriben. Esta opción permite que los usuarios se registren permitiéndoles ver y realizar correcciones en su contraseña si es necesario. Cualquier campo de contraseña escrita tiene una casilla con la etiqueta **Mostrar contraseña**.  Esto permite al usuario ver la contraseña en texto sin formato. Incluya este fragmento de código en la plantilla de registro o de inicio de sesión para una página autoafirmada:

```Javascript
function makePwdToggler(pwd){
  // Create show-password checkbox
  var checkbox = document.createElement('input');
  checkbox.setAttribute('type', 'checkbox');
  var id = pwd.id + 'toggler';
  checkbox.setAttribute('id', id);

  var label = document.createElement('label');
  label.setAttribute('for', id);
  label.appendChild(document.createTextNode('show password'));

  var div = document.createElement('div');
  div.appendChild(checkbox);
  div.appendChild(label);

  // Add show-password checkbox under password input
  pwd.insertAdjacentElement('afterend', div);

  // Add toggle password callback
  function toggle(){
    if(pwd.type === 'password'){
      pwd.type = 'text';
    } else {
      pwd.type = 'password';
    }
  }
  checkbox.onclick = toggle;
  // For non-mouse usage
  checkbox.onkeydown = toggle;
}

function setupPwdTogglers(){
  var pwdInputs = document.querySelectorAll('input[type=password]');
  for (var i = 0; i < pwdInputs.length; i++) {
    makePwdToggler(pwdInputs[i]);
  }
}

setupPwdTogglers();
```

### <a name="add-terms-of-use"></a>Agregar términos de uso

Incluya el código siguiente en la página donde quiera la casilla **Términos de uso**. Normalmente, esta casilla suele ser necesaria en las páginas de registro de cuenta sociales o de cuentas locales.

```Javascript
function addTermsOfUseLink() {
    // find the terms of use label element
    var termsOfUseLabel = document.querySelector('#api label[for="termsOfUse"]');
    if (!termsOfUseLabel) {
        return;
    }

    // get the label text
    var termsLabelText = termsOfUseLabel.innerHTML;

    // create a new <a> element with the same inner text
    var termsOfUseUrl = 'https://docs.microsoft.com/legal/termsofuse';
    var termsOfUseLink = document.createElement('a');
    termsOfUseLink.setAttribute('href', termsOfUseUrl);
    termsOfUseLink.setAttribute('target', '_blank');
    termsOfUseLink.appendChild(document.createTextNode(termsLabelText));

    // replace the label text with the new element
    termsOfUseLabel.replaceChild(termsOfUseLink, termsOfUseLabel.firstChild);
}
```

En el código, reemplace `termsOfUseUrl` con el vínculo al contrato de los términos de uso. Para su directorio, cree un nuevo atributo de usuario denominado **termsOfUse** y, después, incluya **termsOfUse** como un atributo de usuario.

## <a name="next-steps"></a>Pasos siguientes

Encuentre más información acerca de cómo personalizar la interfaz de usuario de las aplicaciones en [Personalización de la interfaz de usuario de la aplicación mediante una directiva personalizada en Azure Active Directory B2C](custom-policy-ui-customization.md).
