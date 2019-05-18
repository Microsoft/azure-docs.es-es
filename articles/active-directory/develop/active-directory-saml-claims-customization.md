---
title: Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales en Azure AD | Microsoft Docs
description: Aprenda a personalizar las notificaciones emitidas en el token SAML para aplicaciones empresariales en Azure AD.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: f1daad62-ac8a-44cd-ac76-e97455e47803
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/03/2019
ms.author: ryanwi
ms.reviewer: luleon, paulgarn, jeedes
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4c1f8640918d433956935e9428e23aac59e36334
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65764664"
---
# <a name="how-to-customize-claims-issued-in-the-saml-token-for-enterprise-applications"></a>Procedimientos para: Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales

En la actualidad, Azure Active Directory (Azure AD) admite single sign-on (SSO) con mayoría de las aplicaciones de empresa, incluidas las aplicaciones previamente integradas en la Galería de aplicaciones de Azure AD, así como las aplicaciones personalizadas. Cuando un usuario se autentica en una aplicación a través de Azure AD con el protocolo SAML 2.0, Azure AD envía un token a la aplicación (mediante HTTP POST). A continuación, la aplicación valida y usa el token para que el usuario inicie sesión en lugar de solicitar un nombre de usuario y una contraseña. Estos tokens SAML contienen trozos de información sobre el usuario que se conoce como *notificaciones*.

Una *notificación* es información que un proveedor de identidades declara sobre un usuario dentro del token que se emite para dicho usuario. En [token SAML](https://en.wikipedia.org/wiki/SAML_2.0), estos datos suelen incluirse en la instrucción SAML Attribute. El identificador único del usuario suele representarse en SAML Subject, también denominado NameIdentifier.

De forma predeterminada, Azure AD emite un token SAML a la aplicación que contiene un `NameIdentifier` notificaciones con el valor de nombre de usuario (también conocido como el nombre principal de usuario) en Azure AD, que puede identificar al usuario. El token SAML también contiene notificaciones adicionales con la dirección de correo electrónico, el nombre y el apellido del usuario.

Para ver o editar las notificaciones emitidas en el token SAML a la aplicación, abra la aplicación en Azure Portal. A continuación, abra el **atributos de usuario y notificaciones** sección.

![En la sección atributos de usuario y notificaciones](./media/active-directory-saml-claims-customization/sso-saml-user-attributes-claims.png)

Hay dos razones posibles por las que tendría que editar las notificaciones emitidas en el token SAML:

* La aplicación requiere la `NameIdentifier` o NameID dicen que algo distinto al nombre de usuario (o el nombre principal de usuario) almacenado en Azure AD.
* La aplicación se ha creado para requerir un conjunto diferente de URI o valores de notificación.

## <a name="editing-nameid"></a>Edición NameID

Para editar la notificación NameID (valor de identificador de nombre):

1. Abra el **denominar el valor de identificador** página.
1. Seleccione el atributo o la transformación que desee aplicar al atributo. Si lo desea, puede especificar el formato que desee la notificación NameID tener.

   ![Edite el valor de NameID (identificador de nombre)](./media/active-directory-saml-claims-customization/saml-sso-manage-user-claims.png)

### <a name="nameid-format"></a>Formato de NameID

Si la solicitud SAML contiene el elemento NameIDPolicy con un formato específico, Azure AD respetará el formato en la solicitud.

Si la solicitud SAML no contiene ningún elemento para NameIDPolicy, Azure AD emitirá la notificación NameID con el formato especificado. Si se especifica ningún formato utilizará el formato de fuente predeterminado asociado con el origen de notificación seleccionado en Azure AD.

Desde el **Elegir formato de identificador de nombre** lista desplegable, puede seleccionar una de las opciones siguientes.

| Formato de NameID | DESCRIPCIÓN |
|---------------|-------------|
| **Valor predeterminado** | Utilizará el formato de origen predeterminado de Azure AD. |
| **Persistent** | Azure AD usará persistente como el formato de NameID. |
| **EmailAddress** | Azure AD usará la dirección de correo electrónico como el formato de NameID. |
| **No se especifica** | Azure AD usará no especificado como el formato de NameID. |
| **Transitorio** | Azure AD usará a transitorio como el formato de NameID. |

Para obtener más información sobre el atributo NameIDPolicy, consulte [protocolo SAML de inicio de sesión único](single-sign-on-saml-protocol.md).

### <a name="attributes"></a>Atributos

Seleccione el origen que desee para la notificación `NameIdentifier` (o NameID). Puede seleccionar entre las opciones siguientes:

| NOMBRE | DESCRIPCIÓN |
|------|-------------|
| Email | Dirección de correo electrónico del usuario |
| userprincipalName | Nombre principal de usuario (UPN) del usuario |
| onpremisessamaccount | Nombre de cuenta SAM que se ha sincronizado desde Azure AD local. |
| objectid | objectID del usuario en Azure AD |
| employeeid | EmployeeID del usuario |
| Extensiones de directorio | Extensiones de directorio [sincronizadas desde Active Directory local con Azure AD Connect Sync](../hybrid/how-to-connect-sync-feature-directory-extensions.md) |
| Atributos de extensión 1-15 | Los atributos de extensión locales usados para extender el esquema de AD Azure. |

Para obtener más información, consulte [tabla 3: Los valores de identificador válidos por origen](active-directory-claims-mapping.md#table-3-valid-id-values-per-source).

### <a name="special-claims---transformations"></a>Notificaciones especiales - transformaciones

También puede usar las funciones de transformación de notificaciones.

| Función | DESCRIPCIÓN |
|----------|-------------|
| **ExtractMailPrefix()** | Quita el sufijo de dominio de la dirección de correo electrónico o nombre principal de usuario. De este modo se extrae solo la primera parte del nombre de usuario por la que se pasa (por ejemplo, "joe_smith" en lugar de joe_smith@contoso.com). |
| **Join()** | Combina un atributo con un dominio comprobado. Si el valor de identificador de usuario seleccionado tiene un dominio, extraerá el nombre de usuario para anexar el dominio comprobado seleccionado. Por ejemplo, si selecciona el correo electrónico (joe_smith@contoso.com) como valor de identificador de usuario y selecciona contoso.onmicrosoft.com como dominio comprobado, el resultado será joe_smith@contoso.onmicrosoft.com. |
| **ToLower()** | Convierte los caracteres del atributo seleccionado en caracteres en minúscula. |
| **ToUpper()** | Convierte los caracteres del atributo seleccionado en caracteres en mayúscula. |

## <a name="adding-application-specific-claims"></a>Incorporación de notificaciones específicas de la aplicación

Para agregar notificaciones específicas de la aplicación:

1. En **atributos de usuario y notificaciones**, seleccione **agregar nueva notificación** para abrir el **Administrar notificaciones de usuario** página.
1. Escriba el **nombre** de las notificaciones. El valor estrictamente no tiene que seguir un patrón de URI, por la especificación SAML. Si necesita un patrón de URI, que puede colocar en el **Namespace** campo.
1. Seleccione el **origen** donde la notificación se va a recuperar su valor. Puede seleccionar un atributo de usuario en la lista desplegable de atributo de origen o aplicar una transformación en el atributo de usuario antes de emitirlo como una notificación.

### <a name="application-specific-claims---transformations"></a>Notificaciones específicas de la aplicación - transformaciones

También puede usar las funciones de transformación de notificaciones.

| Función | DESCRIPCIÓN |
|----------|-------------|
| **ExtractMailPrefix()** | Quita el sufijo de dominio de la dirección de correo electrónico o nombre principal de usuario. De este modo se extrae solo la primera parte del nombre de usuario por la que se pasa (por ejemplo, "joe_smith" en lugar de joe_smith@contoso.com). |
| **Join()** | Crea un nuevo valor mediante la combinación de dos atributos. Si lo desea, puede utilizar un separador entre los dos atributos. |
| **ToLower()** | Convierte los caracteres del atributo seleccionado en caracteres en minúscula. |
| **ToUpper()** | Convierte los caracteres del atributo seleccionado en caracteres en mayúscula. |
| **Contains()** | Genera un atributo o una constante si la entrada coincide con el valor especificado. En caso contrario, puede especificar otra salida si no hay ninguna coincidencia.<br/>Por ejemplo, si desea emitir una notificación donde el valor es la dirección de correo electrónico del usuario si contiene el dominio "@contoso.com", en caso contrario, desea obtener el nombre principal de usuario. Para ello, configuraría los siguientes valores:<br/>*Parámetro 1(input)*: user.email<br/>*Valor*: "@contoso.com"<br/>Parámetro 2 (salida): user.email<br/>Parámetro 3 (si no hay ninguna coincidencia de salida): user.userprincipalname |
| **EndWith()** | Genera un atributo o una constante si finaliza la entrada con el valor especificado. En caso contrario, puede especificar otra salida si no hay ninguna coincidencia.<br/>Por ejemplo, si desea emitir una notificación cuando el valor sea employeeid del usuario si el valor employeeid termina con "000", en caso contrario, desea un atributo de extensión de salida. Para ello, configuraría los siguientes valores:<br/>*Parámetro 1(input)*: user.employeeid<br/>*Valor*: "000"<br/>Parámetro 2 (salida): user.employeeid<br/>Parámetro 3 (si no hay ninguna coincidencia de salida): extensionAttribute1 |
| **StartWith()** | Genera un atributo o una constante si la entrada comienza con el valor especificado. En caso contrario, puede especificar otra salida si no hay ninguna coincidencia.<br/>Por ejemplo, si desea emitir una notificación cuando el valor sea employeeid del usuario si el país o región comienza con "US", en caso contrario, desea un atributo de extensión de salida. Para ello, configuraría los siguientes valores:<br/>*Parámetro 1(input)*: user.country<br/>*Valor*: "NOSOTROS"<br/>Parámetro 2 (salida): user.employeeid<br/>Parámetro 3 (si no hay ninguna coincidencia de salida): extensionAttribute1 |
| **Extract() - después de la coincidencia** | Devuelve la subcadena después de coincide con el valor especificado.<br/>Por ejemplo, si el valor de la entrada es "Finance_BSimon", el valor coincidente es "Finance_", entonces el resultado de la notificación es "BSimon". |
| **Extract() - antes de coincidencia** | Devuelve la subcadena hasta que coincida con el valor especificado.<br/>Por ejemplo, si el valor de la entrada es "BSimon_US", el valor coincidente es "_US", entonces el resultado de la notificación es "BSimon". |
| **Extract() - entre coincidentes** | Devuelve la subcadena hasta que coincida con el valor especificado.<br/>Por ejemplo, si el valor de la entrada es "Finance_BSimon_US", el primer valor coincidente es "Finance_", el segundo valor coincidente es "_US", entonces el resultado de la notificación es "BSimon". |
| **ExtractAlpha() - Prefix** | Devuelve la parte alfabético de prefijo de la cadena.<br/>Por ejemplo, si el valor de la entrada es "BSimon_123", devuelve "BSimon". |
| **ExtractAlpha() - Suffix** | Devuelve la parte alfabético de sufijo de la cadena.<br/>Por ejemplo, si el valor de la entrada es "123_Simon", devuelve a "Simon". |
| **ExtractNumeric() - Prefix** | Devuelve la parte numérica del prefijo de la cadena.<br/>Por ejemplo, si el valor de la entrada es "123_BSimon", devuelve "123". |
| **ExtractNumeric() - sufijo** | Devuelve la parte numérica del sufijo de la cadena.<br/>Por ejemplo, si el valor de la entrada es "BSimon_123", devuelve "123". |
| **IfEmpty()** | Genera un atributo o una constante si la entrada es nulo o está vacío.<br/>Por ejemplo, si desea dar salida a un atributo que se almacenan en un extensionattribute si el identificador de empleado para un usuario determinado está vacío. Para ello, configuraría los siguientes valores:<br/>Parámetro 1(input): user.employeeid<br/>Parámetro 2 (salida): extensionAttribute1<br/>Parámetro 3 (si no hay ninguna coincidencia de salida): user.employeeid |
| **IfNotEmpty()** | Genera un atributo o una constante si la entrada no es nulo o está vacío.<br/>Por ejemplo, si desea dar salida a un atributo que se almacenan en un extensionattribute si el identificador de empleado para un usuario determinado no está vacía. Para ello, configuraría los siguientes valores:<br/>Parámetro 1(input): user.employeeid<br/>Parámetro 2 (salida): extensionAttribute1 |

Si necesita transformaciones adicionales, envíe su idea en el [foro de comentarios de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=160599) bajo el *aplicación SaaS* categoría.

## <a name="next-steps"></a>Pasos siguientes

* [Administración de aplicaciones en Azure AD](../manage-apps/what-is-application-management.md)
* [Configuración del inicio de sesión único en aplicaciones que no están en la galería de aplicaciones de Azure AD](../manage-apps/configure-federated-single-sign-on-non-gallery-applications.md)
* [Solución de problemas del inicio de sesión único basado en SAML](howto-v1-debug-saml-sso-issues.md)
