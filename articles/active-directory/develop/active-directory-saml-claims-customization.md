---
title: Personalización de las notificaciones de token de SAML para las aplicaciones
titleSuffix: Microsoft identity platform
description: Aprenda a personalizar las notificaciones que emite la Plataforma de identidad de Microsoft en el token SAML para aplicaciones empresariales.
services: active-directory
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 10/22/2019
ms.author: kenwith
ms.reviewer: luleon, paulgarn, jeedes
ms.custom: aaddev
ms.openlocfilehash: 5de505ff9573fb186ca2bbe4f5bd6783022eb3ef
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89421465"
---
# <a name="how-to-customize-claims-issued-in-the-saml-token-for-enterprise-applications"></a>Cómo: personalizar las notificaciones emitidas en el token SAML para aplicaciones empresariales

Hoy en día, la Plataforma de identidad de Microsoft admite el inicio de sesión único (SSO) con la mayoría de las aplicaciones empresariales, incluidas las aplicaciones preintegradas en la galería de aplicaciones de Azure AD, así como las aplicaciones personalizadas. Cuando un usuario se autentica en una aplicación mediante la Plataforma de identidad de Microsoft con el protocolo SAML 2.0, la Plataforma de identidad de Microsoft envía un token a la aplicación (mediante HTTP POST). A continuación, la aplicación valida y usa el token para que el usuario inicie sesión en lugar de solicitar un nombre de usuario y una contraseña. Estos tokens SAML contienen trozos de información sobre el usuario conocidos como *notificaciones*.

Una *notificación* es información que un proveedor de identidades declara sobre un usuario dentro del token que se emite para dicho usuario. En [token SAML](https://en.wikipedia.org/wiki/SAML_2.0), estos datos suelen incluirse en la instrucción SAML Attribute. El identificador único del usuario suele representarse en SAML Subject, también denominado NameIdentifier.

De forma predeterminada, la Plataforma de identidad de Microsoft emite un token SAML a la aplicación que contiene una notificación `NameIdentifier` con un valor del nombre de usuario (también denominado nombre principal de usuario) en Azure AD, que puede identificar al usuario de forma única. El token SAML también contiene notificaciones adicionales con la dirección de correo electrónico, el nombre y el apellido del usuario.

Para ver o editar las notificaciones emitidas en el token SAML a la aplicación, abra la aplicación en Azure Portal. Después, abra la sección **Atributos y notificaciones de usuario**.

![Apertura de la sección Atributos y notificaciones de usuario en Azure Portal](./media/active-directory-saml-claims-customization/sso-saml-user-attributes-claims.png)

Hay dos razones posibles por las que tendría que editar las notificaciones emitidas en el token SAML:

* La aplicación requiere que la notificación `NameIdentifier` o NameID tenga un valor que no sea el del nombre de usuario (o nombre principal de usuario) almacenado en Azure AD.
* La aplicación se ha creado para requerir un conjunto diferente de URI o valores de notificación.

## <a name="editing-nameid"></a>Edición de nameID

Para editar la notificación NameID (valor de identificador de nombre):

1. Abra la página **Valor de identificador de nombre**.
1. Seleccione el atributo o la transformación que quiera aplicar al atributo. Si quiere, puede especificar el formato que quiere que tenga la notificación NameID.

   ![Edición del valor de la notificación NameID (identificador de nombre)](./media/active-directory-saml-claims-customization/saml-sso-manage-user-claims.png)

### <a name="nameid-format"></a>Formato de NameID

Si la solicitud SAML contiene el elemento NameIDPolicy con un formato específico, la Plataforma de identidad de Microsoft respetará el formato en la solicitud.

Si la solicitud SAML no contiene ningún elemento para NameIDPolicy, la Plataforma de identidad de Microsoft emitirá la notificación NameID con el formato que especifique. Si no se especifica ningún formato, la Plataforma de identidad de Microsoft usará el formato de origen predeterminado asociado con el origen de notificación seleccionado.

En el menú desplegable **Elija el formato del identificador de nombre**, puede seleccionar una de las opciones siguientes.

| Formato de NameID | Descripción |
|---------------|-------------|
| **Valor predeterminado** | La Plataforma de identidad de Microsoft usará el formato de origen predeterminado. |
| **Persistent** | La Plataforma de identidad de Microsoft usará el valor Persistent como formato de NameID. |
| **EmailAddress** | La Plataforma de identidad de Microsoft usará el valor EmailAddress como formato de NameID. |
| **Unspecified** | La Plataforma de identidad de Microsoft usará el valor Unspecified como formato de NameID. |
| **Nombre completo del dominio de Windows** | La Plataforma de identidad de Microsoft usará el valor WindowsDomainQualifiedName como formato de NameID. |

También se admite NameID transitorio, pero no está disponible en la lista desplegable y no se puede configurar en Azure. Para obtener más información sobre el atributo NameIDPolicy, consulte [Protocolo SAML de inicio de sesión único](single-sign-on-saml-protocol.md).

### <a name="attributes"></a>Atributos

Seleccione el origen que desee para la notificación `NameIdentifier` (o NameID). Puede seleccionar entre las opciones siguientes:

| Nombre | Descripción |
|------|-------------|
| Email | Dirección de correo electrónico del usuario |
| userprincipalName | Nombre principal de usuario (UPN) del usuario. |
| onpremisessamaccount | Nombre de cuenta SAM que se ha sincronizado desde Azure AD local. |
| objectid | ObjectId del usuario en Azure AD |
| employeeid | Id. de empleado del usuario |
| Sincronización de Azure AD Connect: Extensiones de directorio | Extensiones de directorio [sincronizadas desde Active Directory local con Azure AD Connect Sync](../hybrid/how-to-connect-sync-feature-directory-extensions.md) |
| Atributos de extensión 1-15 | Los atributos de extensión locales usados para extender el esquema de AD Azure. |

Para obtener más información, consulte [Tabla 3: Valores de Id. válidos por origen](active-directory-claims-mapping.md#table-3-valid-id-values-per-source).

También puede asignar cualquier valor constante (estático) a cualquier notificación que defina en Azure AD. Siga los pasos que se indican a continuación para asignar un valor constante:

1. En [Azure Portal](https://portal.azure.com/), en la sección **Atributos y notificaciones de usuario**, haga clic en el icono de **edición** para modificar las notificaciones.

1. Haga clic en la notificación que desea modificar.

1. Escriba el valor constante sin comillas en el **Atributo de origen** según su organización y haga clic en **Guardar**.

    ![Sección Org Attributes & Claims (Atributos y notificaciones de organización) de Azure Portal](./media/active-directory-saml-claims-customization/organization-attribute.png)

1. El valor constante se mostrará a continuación.

    ![Sección Edit Attributes & Claims (Editar atributos y notificaciones) de Azure Portal](./media/active-directory-saml-claims-customization/edit-attributes-claims.png)

### <a name="special-claims---transformations"></a>Notificaciones especiales: transformaciones

También puede usar las funciones de transformaciones de notificaciones.

| Función | Descripción |
|----------|-------------|
| **ExtractMailPrefix()** | Quita el sufijo de dominio de la dirección de correo electrónico o el nombre principal de usuario. De este modo se extrae solo la primera parte del nombre de usuario por la que se pasa (por ejemplo, "joe_smith" en lugar de joe_smith@contoso.com). |
| **Join()** | Combina un atributo con un dominio comprobado. Si el valor de identificador de usuario seleccionado tiene un dominio, extraerá el nombre de usuario para anexar el dominio comprobado seleccionado. Por ejemplo, si selecciona el correo electrónico (joe_smith@contoso.com) como valor de identificador de usuario y selecciona contoso.onmicrosoft.com como dominio comprobado, el resultado será joe_smith@contoso.onmicrosoft.com. |
| **ToLower()** | Convierte los caracteres del atributo seleccionado en caracteres en minúscula. |
| **ToUpper()** | Convierte los caracteres del atributo seleccionado en caracteres en mayúscula. |

## <a name="adding-application-specific-claims"></a>Incorporación de notificaciones específicas de la aplicación

Para agregar notificaciones específicas de la aplicación:

1. En **Atributos y notificaciones de usuario**, seleccione **Agregar nueva notificación** para abrir la página **Administrar las notificaciones del usuario**.
1. Escriba el **nombre** de las notificaciones. El valor no tiene que seguir un patrón de URI estrictamente, de acuerdo con la especificación SAML. Si necesita un patrón de URI, puede colocarlo en el campo **Espacio de nombres**.
1. Seleccione el **origen** en el que la notificación va a recuperar su valor. Puede seleccionar un atributo de usuario en la lista desplegable de atributos de origen o aplicar una transformación al atributo de usuario antes de emitirlo como una notificación.

### <a name="claim-transformations"></a>Transformaciones de notificación

Para aplicar una transformación a un atributo de usuario:

1. En **Administrar notificaciones**, seleccione *Transformación* como origen de la notificación para abrir la página **Administrar la transformación**.
2. Seleccione la función en la lista desplegable transformación. Dependiendo de la función seleccionada, tendrá que proporcionar parámetros y un valor constante para evaluar en la transformación. Consulte la tabla siguiente para más información sobre las funciones disponibles.
3. Para aplicar varias transformaciones, haga clic en **Agregar transformación**. Puede aplicar un máximo de dos transformaciones a una notificación. Por ejemplo, puede extraer primero el prefijo de correo del `user.mail`. Después, convierta la cadena en mayúsculas.

   ![Transformación de varias notificaciones](./media/active-directory-saml-claims-customization/sso-saml-multiple-claims-transformation.png)

Puede utilizar las siguientes funciones para transformar notificaciones.

| Función | Descripción |
|----------|-------------|
| **ExtractMailPrefix()** | Quita el sufijo de dominio de la dirección de correo electrónico o el nombre principal de usuario. De este modo se extrae solo la primera parte del nombre de usuario por la que se pasa (por ejemplo, "joe_smith" en lugar de joe_smith@contoso.com). |
| **Join()** | Crea un nuevo valor al combinar dos atributos. Si quiere, puede usar un separador entre los dos atributos. Para la transformación de notificaciones NameID, la combinación está restringida a un dominio comprobado. Si el valor de identificador de usuario seleccionado tiene un dominio, extraerá el nombre de usuario para anexar el dominio comprobado seleccionado. Por ejemplo, si selecciona el correo electrónico (joe_smith@contoso.com) como valor de identificador de usuario y selecciona contoso.onmicrosoft.com como dominio comprobado, el resultado será joe_smith@contoso.onmicrosoft.com. |
| **ToLowercase()** | Convierte los caracteres del atributo seleccionado en caracteres en minúscula. |
| **ToUppercase()** | Convierte los caracteres del atributo seleccionado en caracteres en mayúscula. |
| **Contains()** | Genera un atributo o una constante si la entrada coincide con el valor especificado. En caso contrario, puede especificar otra salida si no hay ninguna coincidencia.<br/>Por ejemplo, si quiere emitir una notificación en la que el valor es la dirección de correo electrónico del usuario si contiene el dominio "@contoso.com"; en caso contrario, quiere obtener el nombre principal de usuario. Para ello, configuraría los siguientes valores:<br/>*Parámetro 1 (entrada)* : user.email<br/>*Valor*: "@contoso.com"<br/>Parámetro 2 (salida): user.email<br/>Parámetro 3 (salida si no hay ninguna coincidencia): user.userprincipalname |
| **EndWith()** | Genera un atributo o una constante si la entrada finaliza con el valor especificado. En caso contrario, puede especificar otra salida si no hay ninguna coincidencia.<br/>Por ejemplo, si quiere emitir una notificación en la que el valor es el Id. de empleado del usuario, si el Id. de empleado termina con "000", de lo contrario, quiere generar un atributo de extensión. Para ello, configuraría los siguientes valores:<br/>*Parámetro 1 (entrada)* : user.employeeid<br/>*Valor*: "000"<br/>Parámetro 2 (salida): user.employeeid<br/>Parámetro 3 (salida si no hay ninguna coincidencia): user.extensionattribute1 |
| **StartWith()** | Genera un atributo o una constante si la entrada empieza con el valor especificado. En caso contrario, puede especificar otra salida si no hay ninguna coincidencia.<br/>Por ejemplo, si quiere emitir una notificación en la que el valor es el Id. de empleado del usuario, si el país o la región comienza por "US", en caso contrario, quiere generar un atributo de extensión. Para ello, configuraría los siguientes valores:<br/>*Parámetro 1 (entrada)* : user.country<br/>*Valor*: "US"<br/>Parámetro 2 (salida): user.employeeid<br/>Parámetro 3 (salida si no hay ninguna coincidencia): user.extensionattribute1 |
| **Extract(): después de la coincidencia** | Devuelve el valor de substring que aparece después de la coincidencia con el valor especificado.<br/>Por ejemplo, si el valor de la entrada es "Finance_BSimon", el valor coincidente es "Finance_" y, por lo tanto, el resultado de la notificación es "BSimon". |
| **Extract(): antes de la coincidencia** | Devuelve el valor de substring que aparece antes de la coincidencia con el valor especificado.<br/>Por ejemplo, si el valor de la entrada es "BSimon_US", el valor coincidente es "_US" y, por lo tanto, el resultado de la notificación es "BSimon". |
| **Extract(): entre coincidencias** | Devuelve el valor de substring que aparece antes de la coincidencia con el valor especificado.<br/>Por ejemplo, si el valor de la entrada es "Finance_BSimon_US", el valor coincidente es "Finance_", el segundo valor coincidente es "_US" y, por lo tanto, el resultado de la notificación es "BSimon". |
| **ExtractAlpha(): prefijo** | Devuelve la parte alfabética del prefijo de la cadena.<br/>Por ejemplo, si el valor de la entrada es "BSimon_123", devuelve "BSimon". |
| **ExtractAlpha(): sufijo** | Devuelve la parte alfabética del sufijo de la cadena.<br/>Por ejemplo, si el valor de la entrada es "123_Simon", devuelve "Simon". |
| **ExtractNumeric(): prefijo** | Devuelve la parte numérica del prefijo de la cadena.<br/>Por ejemplo, si el valor de la entrada es "123_BSimon", devuelve "123". |
| **ExtractNumeric(): sufijo** | Devuelve la parte numérica del sufijo de la cadena.<br/>Por ejemplo, si el valor de la entrada es "BSimon_123", devuelve "123". |
| **IfEmpty()** | Genera un atributo o una constante si la entrada es nula o está vacía.<br/>Por ejemplo, si quiere generar un atributo almacenado en un extensionattribute si el Id. de empleado de un usuario determinado está vacío. Para ello, configuraría los siguientes valores:<br/>Parámetro 1 (entrada): user.employeeid<br/>Parámetro 2 (salida): user.extensionattribute1<br/>Parámetro 3 (salida si no hay ninguna coincidencia): user.employeeid |
| **IfNotEmpty()** | Genera un atributo o una constante si la entrada no es nula ni está vacía.<br/>Por ejemplo, si quiere generar un atributo almacenado en un extensionattribute si el Id. de empleado para un usuario determinado no está vacío. Para ello, configuraría los siguientes valores:<br/>Parámetro 1 (entrada): user.employeeid<br/>Parámetro 2 (salida): user.extensionattribute1 |

Si necesita transformaciones adicionales, envíe su idea al [foro de comentarios de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=160599), en la categoría *Aplicación SaaS*.

## <a name="emitting-claims-based-on-conditions"></a>Emisión de notificaciones basadas en condiciones

Puede especificar el origen de una notificación en función del tipo de usuario y el grupo al que pertenece el usuario. 

El tipo de usuario puede ser:
- **Cualquiera**: Todos los usuarios pueden tener acceso a la aplicación.
- **Miembros**: Miembro nativo del inquilino
- **Todos los invitados**: El usuario se traslada de una organización externa con o sin Azure AD.
- **Invitados de AAD**: El usuario invitado pertenece a otra organización mediante Azure AD.
- **Invitados externos**: El usuario invitado pertenece a una organización externa que no tiene Azure AD.


Un escenario en el que esto resulta útil es cuando el origen de una notificación es diferente para un invitado y para un empleado que tiene acceso a una aplicación. Es posible que quiera especificar que, si el usuario es un empleado, el origen del NameID es user.email, pero, si el usuario es un invitado, el NameID se obtiene de user.extensionattribute1.

Para agregar una condición de notificaciones:

1. En **Administrar notificaciones**, expanda las Condiciones de la notificación.
2. Seleccione el tipo de usuario.
3. Seleccione los grupos a los que debe pertenecer el usuario. Puede seleccionar hasta 50 grupos únicos en todas las notificaciones para una aplicación determinada. 
4. Seleccione el **origen** en el que la notificación va a recuperar su valor. Puede seleccionar un atributo de usuario en la lista desplegable de atributos de origen o aplicar una transformación al atributo de usuario antes de emitirlo como una notificación.

El orden en que se agregan las condiciones es importante. Azure AD evalúa las condiciones de arriba a abajo para decidir qué valor se va a emitir en la notificación. 

Por ejemplo, Britta Simon es un usuario invitado en el suscriptor de Contoso. Pertenece a otra organización que también usa Azure AD. Dada la siguiente configuración para la aplicación de Fabrikam, cuando Britta intenta iniciar sesión en Fabrikam, la Plataforma de identidad de Microsoft evaluará las condiciones como se indica a continuación.

En primer lugar, la Plataforma de identidad de Microsoft comprueba si el tipo de usuario de Britta es `All guests`. Dado que lo es, la Plataforma de identidad de Microsoft asigna el origen de la notificación a `user.extensionattribute1`. En segundo lugar, la Plataforma de identidad de Microsoft comprueba si el tipo de usuario de Britta es `AAD guests`. Dado que también lo es, la Plataforma de identidad de Microsoft asigna el origen de la notificación a `user.mail`. Por último, la notificación se emite con valor de `user.mail` para Britta.

![Configuración condicional de notificaciones](./media/active-directory-saml-claims-customization/sso-saml-user-conditional-claims.png)

## <a name="next-steps"></a>Pasos siguientes

* [Administración de aplicaciones en Azure AD](../manage-apps/what-is-application-management.md)
* [Configuración del inicio de sesión único en aplicaciones que no están en la galería de aplicaciones de Azure AD](../manage-apps/configure-federated-single-sign-on-non-gallery-applications.md)
* [Solución de problemas del inicio de sesión único basado en SAML](../azuread-dev/howto-v1-debug-saml-sso-issues.md)
