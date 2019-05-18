---
title: Administrar certificados de federación en Azure AD | Microsoft Docs
description: Aprenda a personalizar la fecha de expiración de los certificados de federación y a renovar certificados que expiran pronto.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/04/2019
ms.author: mimart
ms.reviewer: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: ecadb499d140ccfc993820080cae0b749977fc61
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824739"
---
# <a name="manage-certificates-for-federated-single-sign-on-in-azure-active-directory"></a>Administrar certificados para inicio de sesión único federado en Azure Active Directory

En este artículo, trataremos las preguntas comunes y la información relacionada con los certificados que crea Azure Active Directory (Azure AD) para establecer federado sesión único (SSO) con el software como servicio (SaaS). Estas aplicaciones se agregan desde la galería de aplicaciones de Azure AD o mediante una plantilla de aplicación ajena a la galería. Configure la aplicación mediante la opción de inicio de sesión único federado.

En este artículo es relevante solo para las aplicaciones que están configuradas para usar SSO de Azure AD a través de [lenguaje de marcado de aserción de seguridad](https://wikipedia.org/wiki/Security_Assertion_Markup_Language) federación (SAML).

## <a name="auto-generated-certificate-for-gallery-and-non-gallery-applications"></a>Generación automática de certificado para aplicaciones de la galería y ajenas a la misma

Al agregar una nueva aplicación desde la galería y configurar un basado en SAML sign-on (seleccionando **inicio de sesión único** > **SAML** desde la página de información general de la aplicación), Azure AD genera un certificado para la aplicación que es válida durante tres años. Para descargar el certificado activo como un certificado de seguridad (**.cer**) archivo, vuelva a esa página (**basado en SAML sign-on**) y seleccione un vínculo de descarga en el **decertificadodefirmadeSAML** encabezado. Puede elegir entre el certificado sin formato (binario) o el certificado (base texto cifrado de base64) en base 64. Para aplicaciones de la galería, esta sección también podría mostrar un vínculo para descargar el certificado como archivo XML de metadatos de federación (un **.xml** archivo), según el requisito de la aplicación.

![SAML active certificado descarga opciones de firma](./media/manage-certificates-for-federated-single-sign-on/active-certificate-download-options.png)

También puede descargar un certificado activo o inactivo seleccionando el **certificado de firma SAML** del encabezado **editar** icono (un lápiz), que muestra la **decertificadodefirmadeSAML** página. Seleccione los puntos suspensivos (**...** ) junto al certificado que desea descargar y, a continuación, elija el formato de certificado que desee. Tiene la opción adicional para descargar el certificado en formato de correo mejorados de privacidad (PEM). Este formato es idéntico a Base64, pero con un **.pem** extensión, que no se reconoce en Windows como un formato de certificado.

![Opciones de descarga del certificado (activas e inactivas) de firma de SAML](./media/manage-certificates-for-federated-single-sign-on/all-certificate-download-options.png)

## <a name="customize-the-expiration-date-for-your-federation-certificate-and-roll-it-over-to-a-new-certificate"></a>Personalizar la fecha de expiración para el certificado de federación y sustituirlo por un certificado nuevo

De forma predeterminada, Azure configura un certificado para que expiren después de tres años, cuando se crea automáticamente durante la configuración de inicio de sesión único de SAML. Dado que no se puede cambiar la fecha de un certificado después de guardarla, tendrá que:

1. Cree un nuevo certificado con la fecha deseada.
2. Guarde el nuevo certificado.
3. Descargue el nuevo certificado en el formato correcto.
4. Cargue el certificado nuevo a la aplicación.
5. Activar el nuevo certificado en el portal de Azure Active Directory.

Las dos secciones siguientes le ayudan a realizar estos pasos.

### <a name="create-a-new-certificate"></a>Crear un nuevo certificado

En primer lugar, cree y guarde el nuevo certificado con una fecha de expiración diferente:

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com/). El **centro de administración de Azure Active Directory** aparece la página.

2. En el panel izquierdo, seleccione **Aplicaciones empresariales**. Aparece una lista de las aplicaciones empresariales en su cuenta.

3. Seleccione la aplicación afectada. Aparece una página de información general de la aplicación.

4. En el panel izquierdo de la página de información general de la aplicación, seleccione **inicio de sesión único**.

5. Si el **seleccionar un método de inicio de sesión único** aparece en la página, seleccione **SAML**.

6. En el **establecer seguridad de sesión único con SAML - versión preliminar** página, busque el **certificado de firma SAML** encabezado y seleccione el **editar** icono (un lápiz). El **certificado de firma SAML** aparece la página, que muestra el estado (**Active** o **inactivo**), fecha de expiración y la huella digital (una cadena de hash) de cada certificado.

7. Seleccione **nuevo certificado**. Aparece una nueva fila debajo de la lista de certificados, donde la fecha de expiración predeterminado es exactamente tres años después de la fecha actual. (Los cambios aún no ha guardado todavía, por lo que todavía se puede modificar la fecha de expiración.)

8. En la nueva fila de certificado, mantenga el mouse sobre la columna de fecha de expiración y seleccione el **Seleccionar fecha** icono (un calendario). Un control de calendario y muestra los días de un mes de la fecha de expiración de la nueva fila actual.

9. Use el control de calendario para establecer una nueva fecha. Puede establecer cualquier fecha entre la fecha actual y tres años después de la fecha actual.

10. Seleccione **Guardar**. El nuevo certificado aparece ahora con el estado **inactivo**, la expiración de la fecha que ha elegido y una huella digital.

11. Seleccione el **X** para volver a la **establecer seguridad de sesión único con SAML - versión preliminar** página.

### <a name="upload-and-activate-a-certificate"></a>Cargar y activar un certificado

A continuación, descargue el nuevo certificado en el formato correcto, cargarlo en la aplicación y que esté activo en Azure Active Directory:

1. Ver instrucciones de configuración de inicio de sesión de SAML adicionales de la aplicación, ya sea por:
   - Seleccionar el **Guía de configuración** vínculo para ver en una ventana del explorador independiente o una pestaña, o
   - va a la **configurar** encabezado y seleccione **ver instrucciones paso a paso** para ver en una barra lateral.

2. En las instrucciones, tenga en cuenta el formato de codificación necesario para la carga del certificado.

3. Siga las instrucciones de la [certificado generado automáticamente para las aplicaciones que no sean galería y](#auto-generated-certificate-for-gallery-and-non-gallery-applications) sección anterior. Este paso descarga el certificado en el formato de codificación que se requiere para la carga de la aplicación.

4. Cuando desea sustituir el certificado nuevo, vuelva a la **certificado de firma SAML** página y, en la fila de certificado recién guardada, seleccione los puntos suspensivos (**...** ) y seleccione **activar el certificado**. El estado del nuevo certificado cambia a **Active**, y el certificado activo anteriormente se cambia a un estado de **inactivo**.

5. Continuar siguientes instrucciones de configuración de inicio de sesión de SAML de la aplicación que se ha mostrado anteriormente, para que pueda cargar la firma de SAML de certificado en el formato de codificación correcto.

## <a name="add-email-notification-addresses-for-certificate-expiration"></a>Agregar direcciones de notificación de correo electrónico para la expiración del certificado

Azure AD enviará una días de 7, 30 y 60 de notificación por correo electrónico antes de que expire el certificado SAML. Puede agregar más de una dirección de correo electrónico para recibir notificaciones. Para especificar las direcciones de correo electrónico a que desea que las notificaciones se envíen:

1. En el **certificado de firma SAML** página, vaya a la **direcciones de correo electrónico de notificación** encabezado. De forma predeterminada, este encabezado usa la dirección de correo electrónico del administrador que ha agregado la aplicación.

2. Debajo de la dirección de correo electrónico final, escriba la dirección de correo electrónico que debe recibir el aviso de expiración del certificado y, a continuación, presione ENTRAR.

3. Repita el paso anterior para cada dirección de correo electrónico que desea agregar.

4. Para cada dirección de correo electrónico que desea eliminar, seleccione el **eliminar** icono (una lata de elementos no utilizados) junto a la dirección de correo electrónico.

5. Seleccione **Guardar**.

Recibirá el correo electrónico de notificación de aadnotification@microsoft.com. Para evitar el correo electrónico desde su ubicación de correo basura, agregue este correo electrónico a sus contactos.

## <a name="renew-a-certificate-that-will-soon-expire"></a>Renovar un certificado que expirará pronto

Si un certificado está a punto de expirar, se puede renovar mediante un procedimiento que da como resultado ningún tiempo de inactividad considerable para los usuarios. Para renovar un certificado que va a expirar:

1. Siga las instrucciones de la [crear un nuevo certificado](#create-a-new-certificate) sección anterior, con una fecha que se superpone con el certificado existente. Esa fecha limita la cantidad de tiempo de inactividad causado por la expiración del certificado.

2. Si la aplicación puede sustituir automáticamente un certificado, establezca el nuevo certificado en active siguiendo estos pasos:
   1. Vuelva a la **certificado de firma SAML** página.
   2. En la fila de certificado recién guardada, seleccione los puntos suspensivos (**...** ) y, a continuación, seleccione **activar el certificado**.
   3. Omita los dos pasos siguientes.

3. Si la aplicación solo puede controlar un certificado a la vez, elija un intervalo de tiempo de inactividad para realizar el paso siguiente. (En caso contrario, si la aplicación no recoge automáticamente el nuevo certificado, pero puede controlar más de un certificado de firma, puede realizar el paso siguiente en cualquier momento.)

4. Antes de que expire el certificado antiguo, siga las instrucciones de la [cargar y activar un certificado](#upload-and-activate-a-certificate) sección anterior.

5. Inicie sesión en la aplicación para asegurarse de que el certificado funciona correctamente.

## <a name="related-articles"></a>Artículos relacionados

* [Tutoriales para integrar aplicaciones SaaS con Azure Active Directory](../saas-apps/tutorial-list.md)
* [Administración de aplicaciones con Azure Active Directory](what-is-application-management.md)
* [Inicio de sesión único en aplicaciones en Azure Active Directory](what-is-single-sign-on.md)
* [Depurar basado en SAML único inicio de sesión en aplicaciones de Azure Active Directory](../develop/howto-v1-debug-saml-sso-issues.md)
