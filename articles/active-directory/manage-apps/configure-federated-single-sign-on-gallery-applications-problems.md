---
title: Problemas en la configuración del inicio de sesión único federado para una aplicación de la galería de Azure AD | Microsoft Docs
description: Resolver algunos de los problemas comunes que pueden surgir al configurar un inicio de sesión único federado con SAML para las aplicaciones que se muestran en la galería de aplicaciones de Azure AD
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/11/2017
ms.author: mimart
ms.collection: M365-identity-device-management
ms.openlocfilehash: 554fb73ea7bf4f866eda3e9f3d629f871cb5e582
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65781162"
---
# <a name="problem-configuring-federated-single-sign-on-for-an-azure-ad-gallery-application"></a>Problemas en la configuración del inicio de sesión único federado para una aplicación de la galería de Azure AD

Si se produce un problema al configurar una aplicación. Compruebe que ha seguido todos los pasos del tutorial de la aplicación. En la configuración de la aplicación, tiene documentación en línea sobre cómo configurarla. También para instrucciones detalladas paso a paso, acceda a la [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-saas-tutorial-list/).

## <a name="cant-add-another-instance-of-the-application"></a>No se puede agregar otra instancia de la aplicación

Para agregar una segunda instancia de una aplicación, tiene que poder:

-   Configurar un identificador único para la segunda instancia. No podrá configurar el mismo identificador que usó para la primera instancia.

-   Configurar un certificado diferente al utilizado para la primera instancia.

Si la aplicación no es compatible con cualquiera de los pasos anteriores. Entonces, no podrá configurar una segunda instancia.

## <a name="cant-add-the-identifier-or-the-reply-url"></a>No se puede agregar el identificador o la dirección URL de respuesta

Si no puede configurar el identificador o la dirección URL de respuesta, confirme que los valores de identificador y la dirección URL de respuesta coincidan con los patrones configurados previamente para la aplicación.

Para conocer los patrones preconfigurados para la aplicación:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.** Vaya al paso 7. Si ya está en la hoja de configuración de aplicación en Azure AD.

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación que desea configurar para el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. En la lista desplegable **Modo**, seleccione **Inicio de sesión basado en SAML**.

9. Vaya al cuadro de texto **Identificador** o **URL de respuesta** en la **sección Dominio y direcciones URL.**

10. Hay tres maneras de saber los patrones admitidos para la aplicación:

    * En el cuadro de texto, verá los patrones admitidos como un marcador de posición *Ejemplo:* <https://contoso.com>.

    * Si el patrón no se admite, verá un signo de exclamación rojo al intentar escribir el valor en el cuadro de texto. Si sitúa el mouse sobre el signo de exclamación rojo, verá los patrones admitidos.

    * En el tutorial de la aplicación, también puede obtener información acerca de los modelos admitidos. En la sección **Configuración del inicio de sesión único de Azure AD**. Vaya al paso para configurar los valores en el **Dominio y direcciones URL**.

Si los valores no coinciden con los patrones preconfigurados en Azure AD. Puede:

-   Trabajar con el proveedor de la aplicación para obtener los valores que coinciden con el patrón preconfigurado en Azure AD

-   O ponerse en contacto con el equipo de Azure AD en <aadapprequest@microsoft.com> o puede dejar un comentario en el tutorial para solicitar la actualización de los patrones admitidos para la aplicación

## <a name="where-do-i-set-the-entityid-user-identifier-format"></a>¿Dónde se puede establecer el formato de EntityID (Identificador de usuario)?

No podrá seleccionar el formato de EntityID (Identificador de usuario) que Azure AD envía a la aplicación en la respuesta a la autenticación de usuario.

AD Azure selecciona el formato del atributo NameID (Identificador de usuario) en función del valor seleccionado o del formato que solicite la aplicación en el elemento AuthRequest de SAML. Para más información, visite la sección NameIDPolicy del artículo [Protocolo SAML de inicio de sesión único](https://docs.microsoft.com/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference#authnrequest).

## <a name="cant-find-the-azure-ad-metadata-to-complete-the-configuration-with-the-application"></a>No se encuentran los metadatos de Azure AD para completar la configuración con la aplicación

Para descargar el certificado o los metadatos de la aplicación de Azure AD, siga estos pasos:

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global** o **coadministrador.**

2. Abra la **extensión de Azure Active Directory** haciendo clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.

3. Escriba **"Azure Active Directory**" en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory**.

4. Haga clic en **Aplicaciones empresariales** en el menú de navegación izquierdo de Azure Active Directory.

5. Haga clic en **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

   * Si no ve la aplicación que desea que aparezca aquí, use el control **Filtro** de la parte superior de la lista **Todas las aplicaciones** y establezca la opción **Mostrar** en **Todas las aplicaciones.**

6. Seleccione la aplicación para la que ha configurado el inicio de sesión único.

7. Cuando se cargue la aplicación, haga clic en **Inicio de sesión único** desde el menú de navegación izquierdo de la aplicación.

8. Vaya a la sección **Certificado de firma de SAML** y haga clic en el valor de la columna **Descargar**. Según lo que necesite la aplicación para configurar el inicio de sesión único, verá la opción para descargar el archivo XML de metadatos o el certificado.

Azure AD no proporciona una dirección URL para obtener los metadatos. Solo se pueden recuperar como un archivo XML.

## <a name="dont-know-how-to-customize-saml-claims-sent-to-an-application"></a>No sabe cómo personalizar las notificaciones SAML que se han enviado a una aplicación

Para obtener información sobre cómo personalizar las notificaciones de atributo SAML que se han enviado a su aplicación, vea [Asignación de notificaciones en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-claims-mapping).

## <a name="next-steps"></a>Pasos siguientes
[Administración de aplicaciones con Azure Active Directory](what-is-application-management.md)
