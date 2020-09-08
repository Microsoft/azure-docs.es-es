---
title: Solución de problemas al registrar una cuenta nueva en Azure Portal o en el centro de cuentas de Azure
description: Resolución de problemas al intentar registrar una nueva cuenta en el centro de cuentas de Microsoft Azure Portal.
services: cost-management-billing
author: v-miegge
manager: dcscontentpm
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: troubleshooting
ms.date: 08/20/2020
ms.author: v-miegge
ms.openlocfilehash: 86f9f92810cff241f816dadfc6e1e8ca120f1f4a
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88686656"
---
# <a name="troubleshoot-issues-when-you-sign-up-for-a-new-account-in-azure-portal-or-azure-account-center"></a>Solución de problemas al registrar una cuenta nueva en Azure Portal o en el centro de cuentas de Azure

Puede surgir algún problema al intentar registrar una nueva cuenta en Microsoft Azure Portal o en el Centro de cuentas de Azure. Esta guía breve le ayudará en el proceso de registro y trata algunos problemas comunes en cada paso.

> [!NOTE]
> Si ya tiene una cuenta existente y busca instrucciones para solucionar problemas de inicio de sesión, consulte [Solución de problemas de inicio de sesión en la suscripción de Azure](https://docs.microsoft.com/azure/cost-management-billing/manage/troubleshoot-sign-in-issue).

## <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar, compruebe lo siguiente:

- La información de perfil de cuenta de Azure (que incluye la dirección de correo electrónico de contacto, la dirección postal y el número de teléfono) es correcta.
- La información de la tarjeta de crédito es correcta.
- No tiene ninguna cuenta Microsoft con la misma información.

## <a name="guided-walkthrough-of-azure-sign-up"></a>Tutorial guiado del proceso de registro de Azure

La experiencia de inicio de sesión de Azure consta de cuatro secciones:

- Acerca de usted
- Verificación de identidad mediante teléfono
- Verificación de identidad mediante tarjeta
- Contrato

En este tutorial se proporcionan ejemplos de la información correcta para registrarse y obtener una cuenta de Azure. En esta sección se abordan algunos problemas comunes y cómo resolverlos.

## <a name="about-you"></a>Acerca de usted

![Acerca de usted](./media/troubleshoot-azure-sign-up/1.png)
 
### <a name="common-issues-and-solutions"></a>Problemas comunes y soluciones

#### <a name="you-see-the-message-we-cannot-proceed-with-sign-up-due-to-an-issue-with-your-account-please-contact-billing-support"></a>Aparece el mensaje "No se puede continuar con el registro debido a un problema con su cuenta. Póngase en contacto con el servicio de soporte técnico de facturación".

Para solucionar este error, siga estos pasos:

1.  Inicie sesión en el [Centro de cuentas de Azure](https://account.azure.com/Profile) con la credencial de administrador de cuenta.
1.  Seleccione **Editar detalles**.
1.  Asegúrese de que todos los campos de dirección están completados y son válidos.
1.  Cuando se registre para obtener una suscripción de Azure, compruebe que la dirección de facturación del registro de la tarjeta de crédito coincide con la de los datos bancarios.

Si sigue recibiendo el mensaje, pruebe a usar otro explorador para registrarse.

¿Qué le parece la exploración de InPrivate?

#### <a name="free-trial-is-not-available"></a>La prueba gratuita no está disponible

¿Ha utilizado una suscripción de Azure en el pasado? El contrato de términos de uso de Azure limita la activación de la evaluación gratuita solo para un usuario que es nuevo en Azure. Si tiene cualquier otro tipo de suscripción de Azure, no puede activar una evaluación gratuita. Considere la posibilidad de suscribirse a una [suscripción de pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/).

#### <a name="you-see-the-message-you-are-not-eligible-for-an-azure-subscription"></a>Aparece el mensaje "No es apto para una suscripción de Azure".

Para resolver este problema, compruebe si se cumplen las siguientes condiciones:

- La información de perfil de cuenta de Azure que ha facilitado (incluidos la dirección de correo electrónico de contacto, la dirección postal y el número de teléfono) es correcta.
- La información de la tarjeta de crédito es correcta.
- No tiene ninguna cuenta Microsoft en la que se use la misma información.

#### <a name="you-see-the-message-your-current-account-type-is-not-supported"></a>Aparece el mensaje "Tipo de cuenta actual no admitido".

Este problema puede producirse si la cuenta está registrada en [un directorio de Azure AD no administrado](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-self-service-signup) y no está en el directorio de Azure AD de la organización.
Para solucionar este problema, inicie sesión en la cuenta de Azure usando otra cuenta o tome el control del directorio de AD no administrado. Para más información, vea [Adquisición de un directorio no administrado como administrador en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/users-groups-roles/domains-admin-takeover).

## <a name="identity-verification-by-phone"></a>Verificación de identidad mediante teléfono

![Verificación de identidad mediante teléfono](./media/troubleshoot-azure-sign-up/2.png)
 
Cuando reciba el mensaje de texto o la llamada telefónica, escriba en el cuadro de texto el código proporcionado.

### <a name="common-issues-and-solutions"></a>Problemas comunes y soluciones

#### <a name="no-verification-text-message-or-phone-call"></a>No recibo el mensaje de texto de verificación ni la llamada telefónica

Aunque el proceso de verificación del registro suele ser rápido, un código de verificación puede tardar hasta cuatro minutos en enviarse.

Estas son algunas sugerencias adicionales:

- Puede usar cualquier número de teléfono para la verificación siempre que cumpla los requisitos. El número de teléfono especificado a efectos de verificación no se guarda como un número de contacto para la cuenta.
  - En el proceso de verificación telefónica no se puede usar un número de teléfono de voz sobre IP (VoIP).
  - Asegúrese de que el teléfono puede recibir llamadas o mensajes SMS desde un número de teléfono de Estados Unidos.
- Revise el número de teléfono especificado, incluido el código de país seleccionado en el menú desplegable.
- Si el teléfono no recibe mensajes de texto (SMS), pruebe la opción **Llamarme**.

## <a name="identity-verification-by-card"></a>Verificación de identidad mediante tarjeta

![Verificación de identidad mediante tarjeta](./media/troubleshoot-azure-sign-up/3.png)
 
### <a name="common-issues-and-solutions"></a>Problemas comunes y soluciones

#### <a name="credit-card-declined-or-not-accepted"></a>Tarjeta de crédito rechazada o no aceptada

Las tarjetas de crédito o débito virtuales o de prepago no se aceptan como opciones de pago para suscripciones de Azure. Para saber qué otros motivos pueden dar lugar a que se rechace la tarjeta, vea [Solución de problemas de una tarjeta rechazada en el registro de Azure](https://docs.microsoft.com/azure/cost-management-billing/manage/troubleshoot-declined-card).

#### <a name="credit-card-form-doesnt-support-my-billing-address"></a>Incompatibilidad del formulario de tarjeta de crédito con la dirección de facturación

La dirección de facturación debe estar en el país que seleccionó en la sección **Acerca de usted**. Compruebe que ha seleccionado el país correcto.

#### <a name="progress-bar-hangs-in-identity-verification-by-card-section"></a>La barra de progreso se bloquea en la sección Verificación de identidad mediante tarjeta.

Para completar la verificación de identidad mediante tarjeta, se deben permitir las cookies de terceros en explorador.

Utilice los pasos siguientes para actualizar la configuración de las cookies del explorador.

1. Actualice la configuración de cookies.
   - Si usa **Chrome**:
     - Seleccione **Configuración** > **Mostrar configuración avanzada** > **Privacidad** > **Configuración de contenido**. Desactive la opción **Bloquear los datos de sitios y las cookies de terceros**.

   - Si usa **Microsoft Edge**:
     - Seleccione **Configuración** > **Ver configuración avanzada** > **Cookies** > **No bloquear cookies**.

1. Actualice la página de registro de Azure y compruebe si el problema se ha resuelto.
1. Si la actualización no soluciona el problema, cierre y reinicie el explorador e inténtelo de nuevo.

### <a name="i-saw-a-charge-on-my-free-trial-account"></a>Hay un cargo reflejado en mi cuenta de evaluación gratuita

Es posible que vea reflejado un pequeño cargo de verificación temporal en la tarjeta de crédito después de registrarse, que desaparecerá en un plazo de entre tres y cinco días. Si le preocupan los costos de administración, lea más información en la página sobre [prevención de costos inesperados](https://docs.microsoft.com/azure/cost-management-billing/manage/getting-started).

## <a name="agreement"></a>Contrato

Formalización del contrato.

## <a name="other-issues"></a>Otros problemas

### <a name="cant-activate-azure-benefit-plan-like-msdn-bizspark-bizsparkplus-or-mpn"></a>No se puede activar un plan de prestaciones de Azure como MSDN, BizSpark, BizSparkPlus o MPN

Asegúrese de que usa las credenciales de inicio de sesión correctas. A continuación, compruebe el programa de beneficios para asegurarse de que es apto.
- MSDN
  - Compruebe su estado de idoneidad en la [página de la cuenta de MSDN](https://msdn.microsoft.com/subscriptions/manage/default.aspx).
  - Si no puede verificar su estado, póngase en contacto con los [Centros de servicio al cliente de Suscripciones a MSDN](https://msdn.microsoft.com/library/aa493452.aspx).
- Microsoft for Startups
  - Inicie sesión en el [portal de Microsoft for Startups](https://startups.microsoft.com/#start-two) para verificar su estado de idoneidad para Microsoft for Startups.
  - Si no puede verificar el estado, puede obtener ayuda en los [foros de Microsoft for Startups](https://www.microsoftpartnercommunity.com/t5/Microsoft-for-Startups/ct-p/Microsoft_Startups).
- MPN
  - Inicie sesión en el [portal de MPN](https://mspartner.microsoft.com/Pages/Locale.aspx) y compruebe su estado de idoneidad. Si tiene las [competencias de la plataforma de nube](https://mspartner.microsoft.com/pages/membership/cloud-platform-competency.aspx) pertinentes, puede ser apto para obtener más ventajas.
  - Si no puede verificar el estado, póngase en contacto con el [soporte técnico de MPN](https://mspartner.microsoft.com/Pages/Support/Premium/contact-support.aspx).


### <a name="cant-activate-new-azure-in-open-subscription"></a>No se puede activar una nueva suscripción de Azure bajo licencia Open

Para crear una suscripción de Azure bajo licencia Open, debe contar con una clave válida de Activación del servicio en línea (OSA) que tenga asociado al menos un token de Azure bajo licencia Open. Si no tiene una clave OSA, póngase en contacto con alguno de los asociados de Microsoft que aparecen en [Microsoft Pinpoint](https://pinpoint.microsoft.com/).

## <a name="additional-help-resources"></a>Más recursos de ayuda

Otros artículos de solución de problemas relativos a la facturación y las suscripciones de Azure

- [Tarjeta rechazada](https://docs.microsoft.com/azure/cost-management-billing/manage/troubleshoot-declined-card)
- [Problemas de inicio de sesión de suscripción](https://docs.microsoft.com/azure/cost-management-billing/manage/troubleshoot-sign-in-issue)
- [No se han encontrado suscripciones](https://docs.microsoft.com/azure/cost-management-billing/manage/no-subscriptions-found)
- [Se ha deshabilitado la vista de costos empresariales](https://docs.microsoft.com/azure/cost-management-billing/manage/enterprise-mgmt-grp-troubleshoot-cost-view)

## <a name="contact-us-for-help"></a>Póngase en contacto con nosotros para obtener ayuda

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="find-out-more-about-azure-cost-management"></a>Más información sobre Azure Cost Management.

- [Documentación de Azure Cost Management y facturación](https://docs.microsoft.com/azure/cost-management-billing)
