---
title: Transferencia de la propiedad de suscripción de Azure a otra cuenta | Microsoft Docs
description: Transferencia de una suscripción de Azure a otro usuario y algunas preguntas frecuentes (P+F) sobre el proceso
keywords: transferencia de la suscripción de Azure, suscripción de transferencia de Azure, traslado de la suscripción de Azure a otro propietario, cambio de propietario de la suscripción de Azure, transferencia de la suscripción de Azure a otra cuenta
author: bandersmsft
manager: amberb
tags: billing,top-support-issue
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/16/2019
ms.author: banders
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: b3f070cafa026679eb9322ac7ba2eafe4c7e233d
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65826759"
---
# <a name="transfer-ownership-of-an-azure-subscription-to-another-account"></a>Transferencia de la propiedad de una suscripción de Azure a otra cuenta

Transfiera su suscripción a otro usuario del Centro de cuentas para cambiar el administrador de la cuenta y transferir la titularidad de la facturación. Para cambiar su suscripción por una oferta diferente, consulte [Cambio de la suscripción de Azure a otra oferta](billing-how-to-switch-azure-offer.md).

> [!IMPORTANT]
>
> Si se transfiere una suscripción a un nuevo inquilino de Azure AD, todas las asignaciones de roles en el [control de acceso basado en roles (RBAC)](../role-based-access-control/overview.md) se eliminan permanentemente del inquilino de origen y no se migran al inquilino de destino. También debe volver a crear manualmente las identidades administradas para los recursos de Azure. Para obtener más información, consulte [preguntas más frecuentes y problemas conocidos con managed identidades](../active-directory/managed-identities-azure-resources/known-issues.md).

## <a name="transfer-ownership-of-an-azure-subscription"></a>Transferencia de la propiedad de una suscripción de Azure

> [!VIDEO https://channel9.msdn.com/Series/Microsoft-Azure-Tutorials/Transfer-an-Azure-subscription/player]


1. Inicie sesión en el [Centro de cuentas de Azure](https://account.windowsazure.com/Subscriptions) como administrador de la cuenta. Para saber quién es el administrador de la cuenta de la suscripción, consulte [Preguntas más frecuentes](#faq).

1. Seleccione la suscripción que va a transferir.

1. Compruebe que la suscripción es apta para la transferencia de autoservicio comprobando los valores de **Oferta** e **Id. de oferta** en la [lista de ofertas admitidas](#supported).

   ![Comprobación del identificador de oferta de la suscripción en el Centro de cuentas](./media/billing-subscription-transfer/image0.png)
1. Haga clic en **Transferir suscripción**.

   ![Pestaña Suscripciones de cuenta de Azure](./media/billing-subscription-transfer/image1.png)
1. Especifique el destinatario.

   > [!IMPORTANT]
   >
   > Si se transfiere una suscripción a un nuevo inquilino de Azure AD, todas las asignaciones de roles en el [control de acceso basado en roles (RBAC)](../role-based-access-control/overview.md) se eliminan permanentemente del inquilino de origen y no se migran al inquilino de destino. También debe volver a crear manualmente las identidades administradas para los recursos de Azure. Para obtener más información, consulte [preguntas más frecuentes y problemas conocidos con managed identidades](../active-directory/managed-identities-azure-resources/known-issues.md).

   ![Cuadro de diálogo Transferir suscripción](./media/billing-subscription-transfer/image2.PNG)

1. El destinatario obtiene automáticamente un correo electrónico con un vínculo de aceptación.

   ![Correo electrónico de transferencia de suscripción a destinatario](./media/billing-subscription-transfer/image3.png)
1. El destinatario hace clic en el vínculo y sigue las instrucciones, incluyendo la especificación de su información de pago.

   ![Página web de primera transferencia de suscripción](./media/billing-subscription-transfer/image4.png)

   ![Página web de segunda transferencia de suscripción](./media/billing-subscription-transfer/image5.png)
1. Correcto La suscripción ya está transferida.

<a id="EA"></a>

## <a name="transfer-subscription-ownership-for-ea-customers"></a>Transferir la propiedad de suscripción para los clientes de EA

El administrador de la empresa puede transferir la propiedad de las suscripciones de una inscripción. Para empezar, vea [Transfer Account Ownership ](https://ea.azure.com/helpdocs/changeAccountOwnerForASubscription) (Transferencia de la propiedad de la cuenta) en el portal de EA.

## <a name="next-steps-after-accepting-ownership"></a>Pasos siguientes después de aceptar la propiedad

1. Ahora es el administrador de cuenta. Revise y actualice el administrador del servicio, los coadministradores y otros roles de RBAC. Para obtener más información, vea [Agregar o cambiar los administradores de la suscripción de Azure](billing-add-change-azure-subscription-administrator.md) y [Administración del acceso mediante RBAC y Azure Portal](../role-based-access-control/role-assignments-portal.md).
1. Actualice las credenciales asociadas a los servicios de esta suscripción:
   1. Certificados de administración que conceden al usuario derechos administrativos a los recursos de la suscripción. Para obtener más información, consulte [Crear y cargar un certificado de administración para Azure](../cloud-services/cloud-services-certs-create.md)
   1. Claves de acceso para servicios como Almacenamiento. Para más información, consulte [Acerca de las cuentas de almacenamiento de Azure](../storage/common/storage-create-storage-account.md).
   1. Credenciales de acceso remoto para servicios como Azure Virtual Machines.
1. Si trabaja con un asociado, considere la posibilidad de actualizar el identificador del asociado en esta suscripción. Puede actualizar el identificador de asociado en [Azure Portal](https://portal.azure.com).

<a id="supported"></a>

## <a name="supported-offers"></a>Ofertas admitidas

La transferencia de las suscripciones de autoservicio está disponible para las ofertas o tipos de suscripciones que se muestran en la tabla siguiente. Actualmente no se puede transferir una evaluación gratuita o las suscripciones de [Azure bajo licencia Open (AIO)](https://azure.microsoft.com/offers/ms-azr-0111p/). Para ver una solución alternativa, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/resource-group-move-resources.md). Para transferir otras suscripciones, como [Patrocinio](https://azure.microsoft.com/offers/ms-azr-0036p/) o planes de soporte técnico, [póngase en contacto con el soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

| Nombre de la oferta                                                                             | Número de la oferta |
|----------------------------------------------------------------------------------------|--------------|
| [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/)\*|MS-AZR-0017P        |
| [Microsoft Partner Network](https://azure.microsoft.com/offers/ms-azr-0025p/)          | MS-AZR-0025P        |
| [Plataformas de MSDN](https://azure.microsoft.com/offers/ms-azr-0062p/)                     | MS-AZR-0062P        |
| [Pay-As-You-Go](https://azure.microsoft.com/offers/ms-azr-0003p/)                      | MS-AZR-0003P        |
| [Desarrollo/pruebas - Pago por uso](https://azure.microsoft.com/offers/ms-azr-0023p/)             | MS-AZR-0023P        |
| [Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p/)           | MS-AZR-0063P        |
| [Visual Studio Enterprise: BizSpark](https://azure.microsoft.com/offers/ms-azr-0064p/) | MS-AZR-0064P        |
| [Visual Studio Professional](https://azure.microsoft.com/offers/ms-azr-0059p/)         | MS-AZR-0059P        |
| [Visual Studio Test Professional](https://azure.microsoft.com/offers/ms-azr-0060p/)    | MS-AZR-0060P        |

\*[A través del portal de EA](#EA)

<a id="faq"></a>

## <a name="frequently-asked-questions-faq"></a>Preguntas más frecuentes

### <a name="whoisaa"></a>¿Quién es el administrador de la cuenta de la suscripción?

El administrador de cuenta es la persona que se inscribió o que adquirió la suscripción de Azure. Están autorizados para tener acceso al [Centro de cuentas](https://account.azure.com/Subscriptions) y realizar diversas tareas de administración como crear suscripciones, cancelar suscripciones, cambiar la facturación de una suscripción o cambiar el administrador de servicios. Si no está seguro de quién es el administrador de cuenta de la suscripción, use los pasos siguientes para averiguarlo.

1. Visite la [página Suscripciones de Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).
1. Seleccione la suscripción que desee comprobar y, luego, consulte **Configuración**.
1. Seleccione **Propiedades**. El administrador de cuentas de la suscripción se muestra en el cuadro **Administrador de cuentas** .

### <a name="does-everything-transfer-including-resource-groups-vms-disks-and-other-running-services"></a>¿Se transfiere todo? ¿Incluidos los grupos de recursos, las máquinas virtuales, los discos y otros servicios en ejecución?

Todos los recursos, como las máquinas virtuales, los discos y los sitios web, se transfieren al nuevo propietario. Sin embargo, cualquier [rol de administrador](billing-add-change-azure-subscription-administrator.md) y directiva del [control de acceso basado en rol (RBAC)](../role-based-access-control/role-assignments-portal.md) que haya configurado no se transferirán entre los distintos directorios. Tampoco se transferirán los [registros de aplicaciones](../active-directory/develop/quickstart-v1-integrate-apps-with-azure-ad.md) ni otros servicios específicos del inquilino.

### <a id="no-button"></a>¿Por qué no veo el botón "Transfer subscription" (Transferir suscripción)?

Por desgracia, la transferencia de suscripción de autoservicio no está disponible para su oferta. Ver la lista de ofertas admitidas en el [admite ofertas](#supported-offers) sección de este artículo.

### <a name="can-i-transfer-ownership-to-an-account-in-another-country"></a>¿Puedo transferir la propiedad a una cuenta en otro país?

Lamentablemente, Azure no permite la transferencia entre país. Para transferir su suscripción cross country, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).


### <a name="does-a-subscription-transfer-result-in-any-service-downtime"></a>¿Provocan las transferencias de suscripciones un tiempo de inactividad en el servicio?

No afectan al servicio. Si se transfiere la suscripción, se cancela la suscripción en el administrador de cuenta actual y se crea una suscripción en la cuenta del destinatario. La nueva suscripción está asociada a los servicios de Azure subyacentes. El identificador de la suscripción no cambia.

### <a name="how-do-i-use-this-process-to-change-the-directory-for-subscription"></a>¿Cómo puedo usar este proceso para cambiar el directorio de suscripción?

Se crea una suscripción de Azure en el directorio al que pertenece el administrador de cuenta. Para cambiar el directorio, transfiera la suscripción a una cuenta de usuario en el directorio de destino. Cuando el usuario completa los pasos para aceptar la transferencia, la suscripción se mueve automáticamente al directorio de destino.

### <a name="if-i-take-over-billing-ownership-of-a-subscription-from-another-organization-do-they-continue-to-have-access-to-my-resources"></a>¿Si asumo la propiedad de la facturación de una suscripción de otra organización, estos seguirán teniendo acceso a mis recursos?

Si la suscripción se transfiere a otro inquilino, los usuarios asociados al inquilino anterior perderán el acceso a la suscripción. Aunque un usuario deje de ser administrador o coadministrador de servicios, puede seguir teniendo acceso a la suscripción a través de otros mecanismos de seguridad, como los siguientes:

* Certificados de administración que conceden al usuario derechos administrativos a los recursos de la suscripción. Para más información, consulte [Create and Upload a Management Certificate for Azure](../cloud-services/cloud-services-certs-create.md) (Creación y actualización de un certificado de administración para Azure).
* Claves de acceso para servicios como Almacenamiento. Para más información, consulte [Acerca de las cuentas de almacenamiento de Azure](../storage/common/storage-create-storage-account.md).
* Credenciales de acceso remoto para servicios como Azure Virtual Machines.

El destinatario debe restringir el acceso a sus recursos y considerar la actualización de todos los secretos asociados al servicio. La mayoría de los recursos se pueden actualizar mediante el uso de los siguientes pasos:

  1. Vaya a [Azure Portal](https://portal.azure.com).
  2. En el menú Concentrador, seleccione **Todos los recursos**.
  3. Seleccione el recurso.
  4. En la hoja de recursos, haga clic en **Configuración**. Aquí puede ver y actualizar los secretos existentes.

### <a name="if-i-transfer-the-subscription-in-the-middle-of-the-billing-cycle-does-the-recipient-pay-for-the-entire-billing-cycle"></a>Si transfiero la suscripción en medio del ciclo de facturación, ¿paga el destinatario el ciclo completo?

El remitente es el responsable del pago de cualquier uso que se notificó hasta el momento en que se completó la transferencia. El destinatario es el responsable del uso notificado desde el momento de la transferencia en adelante. Puede haber una parte del uso que se realizara antes de la transferencia, pero que se notificara con posterioridad. El uso se incluye en la factura del destinatario.

### <a name="does-the-recipient-have-access-to-usage-and-billing-history"></a>¿Tiene el destinatario acceso al historial de facturación y uso?

  La única información disponible para el destinatario es la cantidad de la última factura o si la suscripción se transfirió antes de que se generara la primera factura, el balance actual. El resto del historial de facturación y uso no se transfiere con la suscripción.

### <a name="can-the-offer-be-changed-during-a-transfer"></a>¿Se puede cambiar la oferta durante una transferencia?

La oferta debe permanecer igual. Para cambiar la oferta, consulte [Cambio de la suscripción de Azure a otra oferta](billing-how-to-switch-azure-offer.md).

### <a name="can-i-transfer-a-subscription-to-a-user-account-in-another-countryregion"></a>¿Puedo transferir una suscripción a una cuenta de usuario en otro país o región?

No, no se admite la transferencia de una suscripción a una cuenta de usuario en otro país o región. Cuenta de usuario del destinatario debe estar en el mismo país o región.

### <a name="can-the-recipient-use-a-different-payment-method"></a>¿Puede usar el destinatario un método de pago diferente?

Sí. Sin embargo, el historial de facturación de la suscripción se divide en dos cuentas.  

### <a name="is-the-payment-method-impacted-after-i-transferred-an-azure-subscription"></a>¿Se verá afectado el método de pago después de transferir una suscripción de Azure?

Para aceptar una transferencia de suscripción, se debe proporcionar una tarjeta de crédito o un método de pago similar para pagar la suscripción. Por ejemplo, si Roberto transfiere una suscripción a Julia y esta acepta la transferencia, Julia debe proporcionar el método de pago que usará para pagar la suscripción. Una vez completada la transferencia, será a Julia a quien se le facture por la suscripción, no Roberto.

### <a name="how-do-i-migrate-data-and-services-for-my-azure-subscription-to-new-subscription"></a>¿Cómo se migran los datos y servicios de una suscripción de Azure a otra?

Si no se puede transferir la propiedad de suscripción, puede migrar manualmente los recursos. Consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/resource-group-move-resources.md).

## <a name="need-help-contact-us"></a>¿Ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [crear una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- Revise y actualice el administrador del servicio, los coadministradores y otros roles de RBAC. Para obtener más información, vea [Agregar o cambiar los administradores de la suscripción de Azure](billing-add-change-azure-subscription-administrator.md) y [Administración del acceso mediante RBAC y Azure Portal](../role-based-access-control/role-assignments-portal.md).
