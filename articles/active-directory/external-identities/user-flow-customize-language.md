---
title: Personalización de idioma en flujos de usuario de Azure AD
description: Más información acerca de cómo personalizar la experiencia de lenguaje en los flujos de usuario.
services: active-directory
author: msmimart
manager: celestedg
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 05/06/2020
ms.author: mimart
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: abd6423ebbdba11cd1b0e0c2d00cfd36aa745e72
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907644"
---
# <a name="language-customization-in-azure-active-directory-preview"></a>Personalización de idioma en Azure Active Directory (versión preliminar)

> [!NOTE]
> El registro de autoservicio es la característica en versión preliminar pública de Azure Active Directory. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

La personalización de idioma en Azure Active Directory (Azure AD) permite que el flujo de usuario albergue distintos idiomas a fin de satisfacer las necesidades del usuario. Microsoft proporciona las traducciones de [36 idiomas](#supported-languages). Incluso si su experiencia se proporciona únicamente para un idioma, puede personalizar los nombres de atributos en la página de la colección de atributos.

## <a name="how-language-customization-works"></a>¿Cómo funciona la personalización de idioma?

De manera predeterminada, la personalización de idioma está habilitada para que los usuarios se registren a fin de garantizar una experiencia de registro coherente. Puede usar idiomas para modificar las cadenas que se muestran a los usuarios como parte del proceso de la colección de atributos durante el registro.

> [!NOTE]
> Si va a usar atributos de usuario personalizados, debe proporcionar sus propias traducciones. Para más información, consulte [Personalización de las cadenas](#customize-your-strings).

## <a name="customize-your-strings"></a>Personalización de las cadenas

La personalización de idioma le permite personalizar cualquier cadena del flujo de usuario.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de Azure AD.
2. En **Servicios de Azure**, seleccione **Azure Active Directory**.
3. En el menú de la izquierda, seleccione **External Identities**.
4. Seleccione **Flujos de usuario (versión preliminar)** .
3. Seleccione el flujo de usuario que quiere habilitar para las traducciones.
4. Seleccione **Idiomas**.
5. En la página **Idiomas** del flujo de usuario, seleccione un idioma que desee personalizar.
6. Expanda la **página Colección de atributos**.
7. Seleccione **Descargar valores predeterminados** (o **Descargar invalidaciones** si ha editado anteriormente este idioma).

Estos pasos le proporcionan un archivo JSON que puede usar para comenzar a editar las cadenas.

### <a name="change-any-string-on-the-page"></a>Cambio de cualquier cadena en la página

1. Abra el archivo JSON descargado según las instrucciones anteriores en un editor JSON.
1. Busque el elemento que desea cambiar. Puede encontrar el valor de `StringId` de la cadena que busca o buscar el atributo `Value` que quiere cambiar.
1. Actualice el atributo `Value` con el que quiere que se muestre.
1. Para cada cadena que desea cambiar, cambie `Override` a `true`.
1. Guarde el archivo y cargue los cambios. (Puede encontrar el control de carga en el mismo lugar que donde descargó el archivo JSON).

> [!IMPORTANT]
> Si necesita reemplazar una cadena, asegúrese de establecer el valor `Override` en `true`. Si no se cambia el valor, se omite la entrada.

### <a name="change-extension-attributes"></a>Cambio de los atributos de extensión

Si quiere cambiar la cadena de un atributo de usuario personalizado, o quiere agregar una a JSON, debe estar en el siguiente formato:

```JSON
{
  "LocalizedStrings": [
    {
      "ElementType": "ClaimType",
      "ElementId": "extension_<ExtensionAttribute>",
      "StringId": "DisplayName",
      "Override": true,
      "Value": "<ExtensionAttributeValue>"
    }
    [...]
}
```

Reemplace `<ExtensionAttribute>` por el nombre de su atributo de usuario personalizado.

Reemplace `<ExtensionAttributeValue>` por la nueva cadena que se mostrará.

### <a name="provide-a-list-of-values-by-using-localizedcollections"></a>Entrega de una lista de valores mediante LocalizedCollections

Si desea proporcionar una lista establecida de valores para respuestas, debe crear un atributo `LocalizedCollections`. `LocalizedCollections` es una matriz de pares de `Name` y `Value`. El orden de los elementos será el orden en el que se muestran. Para agregar `LocalizedCollections`, utilice el siguiente formato:

```JSON
{
  "LocalizedStrings": [...],
  "LocalizedCollections": [{
      "ElementType":"ClaimType",
      "ElementId":"<UserAttribute>",
      "TargetCollection":"Restriction",
      "Override": true,
      "Items":[
           {
                "Name":"<Response1>",
                "Value":"<Value1>"
           },
           {
                "Name":"<Response2>",
                "Value":"<Value2>"
           }
     ]
  }]
}
```

* `ElementId` es el atributo de usuario para el que este atributo `LocalizedCollections` es una respuesta.
* `Name` es el valor que se muestra al usuario.
* `Value` es lo que se devuelve en la notificación cuando se selecciona esta opción.

### <a name="upload-your-changes"></a>Carga de los cambios

1. Una vez completados los cambios en el archivo JSON, vuelva al inquilino.
1. Seleccione **Flujos de usuario** y haga clic en el flujo de usuario que desea habilitar para las traducciones.
1. Seleccione **Idiomas**.
1. Seleccione el idioma al que quiere traducir.
1. Seleccione la **página Colección de atributos**.
1. Seleccione el icono de carpeta y el archivo JSON para cargar.

Este cambio se guarda en el flujo de usuario automáticamente.

## <a name="additional-information"></a>Información adicional

### <a name="page-ui-customization-labels-as-overrides"></a>Etiquetas de personalización de la interfaz de usuario de página como invalidaciones

Cuando se habilita la personalización de idioma, las ediciones anteriores de etiquetas que usan la personalización de la interfaz de usuario de página se almacenan en un archivo JSON para inglés (en). Para seguir cambiando las etiquetas y otras cadenas, cargue los recursos de idioma en Personalización de idioma.

### <a name="up-to-date-translations"></a>Actualización de traducciones

Microsoft se compromete a proporcionar las traducciones más actualizadas para que haga uso de ellas. Microsoft mejora continuamente las traducciones y garantiza su cumplimiento. Microsoft identificará errores y cambios en la terminología global y creará actualizaciones que funcionen perfectamente en su flujo de usuario.

### <a name="support-for-right-to-left-languages"></a>Compatibilidad con idiomas que se leen de derecha a izquierda

Microsoft no proporciona actualmente compatibilidad con idiomas que se leen de derecha a izquierda. Para ello, puede usar configuraciones regionales personalizadas y CSS para cambiar la manera en la que se muestran las cadenas. Si necesita esta característica, vote por ella en [Comentarios de Azure](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/19393000-provide-language-support-for-right-to-left-languag).

### <a name="social-identity-provider-translations"></a>Traducciones de proveedores de identidades sociales

Microsoft proporciona el parámetro OIDC `ui_locales` a los inicios de sesión de redes sociales. Pero algunos proveedores de identidades de redes sociales, incluidas Facebook y Google, no los respetan.

### <a name="browser-behavior"></a>Comportamiento del explorador

Tanto Chrome como Firefox solicitan su idioma establecido. Si es un idioma admitido, se muestra antes el valor predeterminado. Microsoft Edge no solicita actualmente un idioma y va directamente al predeterminado.

## <a name="supported-languages"></a>Idiomas compatibles

Azure AD incluye compatibilidad con los idiomas siguientes. Azure AD proporciona los idiomas de flujo de usuario. [Azure MFA](https://docs.microsoft.com/azure/active-directory/authentication/concept-mfa-howitworks) proporciona los idiomas de notificación de autenticación multifactor (MFA).

| Idioma              | Código de lenguaje | Flujos de usuario         | Notificaciones de MFA  |
|-----------------------| :-----------: | :----------------: | :----------------: |
| Árabe                | ar            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Búlgaro             | bg            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Bengalí                | bn            | ![sí](./media/user-flow-customize-language/yes.png) | ![no](./media/user-flow-customize-language/no.png) |
| Catalán               | ca            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Checo                 | cs            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Danés                | da            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Alemán                | de            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Griego                 | el            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Inglés               | en            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Español               | es            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Estonio              | et            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Vasco                | eu            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Finés               | fi            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Francés                | fr            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Gallego              | gl            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Gujarati              | gu            | ![sí](./media/user-flow-customize-language/yes.png) | ![no](./media/user-flow-customize-language/no.png) |
| Hebreo                | he            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Hindi                 | hi            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Croata              | hr            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Húngaro             | hu            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Indonesio            | id            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Italiano               | it            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Japonés              | ja            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Kazajo                | kk            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Canarés               | kn            | ![sí](./media/user-flow-customize-language/yes.png) | ![no](./media/user-flow-customize-language/no.png) |
| Coreano                | ko            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Lituano            | lt            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Letón               | lv            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Malayalam             | ml            | ![sí](./media/user-flow-customize-language/yes.png) | ![no](./media/user-flow-customize-language/no.png) |
| Maratí               | mr            | ![sí](./media/user-flow-customize-language/yes.png) | ![no](./media/user-flow-customize-language/no.png) |
| Malayo                 | ms            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Noruego Bokmal      | nb            | ![sí](./media/user-flow-customize-language/yes.png) | ![no](./media/user-flow-customize-language/no.png) |
| Neerlandés                 | nl            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Noruego             | no            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Punjabi               | pa            | ![sí](./media/user-flow-customize-language/yes.png) | ![no](./media/user-flow-customize-language/no.png) |
| Polaco                | pl            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Portugués (Brasil)   | pt-br         | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Portugués (Portugal) | pt-pt         | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Rumano              | ro            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Ruso               | ru            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Eslovaco                | sk            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Esloveno             | sl            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Serbio (cirílico)    | sr-cryl-cs    | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Serbio (latino)       | sr-latn-cs    | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Sueco               | sv            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Tamil                 | ta            | ![sí](./media/user-flow-customize-language/yes.png) | ![no](./media/user-flow-customize-language/no.png) |
| Telugu                | te            | ![sí](./media/user-flow-customize-language/yes.png) | ![no](./media/user-flow-customize-language/no.png) |
| Tailandés                  | th            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Turco               | tr            | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Ucraniano             | uk            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Vietnamita            | vi            | ![no](./media/user-flow-customize-language/no.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Chino (simplificado)  | zh-hans       | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |
| Chino (tradicional) | zh-hant       | ![sí](./media/user-flow-customize-language/yes.png) | ![sí](./media/user-flow-customize-language/yes.png) |