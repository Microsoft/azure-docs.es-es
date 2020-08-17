---
title: Cambio de la información de contacto de la cuenta de Azure
description: Describe cómo cambiar la información de contacto de su cuenta administrativa de Azure
author: genlin
ms.reviewer: dcscontentpm
tags: billing
ms.service: cost-management-billing
ms.topic: how-to
ms.date: 08/03/2020
ms.author: banders
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: cb7978dfa2c3ded2ae8e8c621bcb679f19620a2c
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87553485"
---
# <a name="change-the-contact-information-for-your-azure-account"></a>Cambio de la información de contacto de la cuenta de Azure

Este artículo le ayuda a actualizar la información de contacto de su cuenta en Azure Portal. Las instrucciones para actualizar la información de contacto varían en función del tipo de cuenta de facturación. Para más información sobre las cuentas de facturación y la identificación del tipo de cuenta de facturación, consulte [Visualización de las cuentas de facturación en Azure Portal](view-all-accounts.md).

*Dirección del comprador*: la dirección del comprador es la dirección e información de contacto de la organización o individuo responsables de una cuenta de facturación. Se muestra en todas las facturas generadas para la cuenta de facturación.

*Dirección de facturación*: la dirección del comprador es la dirección e información de contacto de la organización o individuo responsables de las facturas generadas para una cuenta de facturación. En el caso de una cuenta de facturación de un programa Microsoft Online Service Program (MOSP), hay una dirección de facturación, que se muestra en todas las facturas generadas para la cuenta. En el caso de una cuenta de facturación para un Contrato de cliente de Microsoft (MCA), hay una dirección de facturación para cada perfil de facturación y se muestra en la factura generada para el perfil de facturación.

*Dirección de correo electrónico de contacto para los correos electrónicos de servicio y marketing*: puede especificar una dirección de correo electrónico que sea distinta de la que usa para iniciar sesión para recibir notificaciones importantes relacionadas con la facturación y el servicio, así como recomendaciones sobre su cuenta de Azure. Las notificaciones del servicio por correo electrónico (como los problemas de seguridad urgentes, los cambios de precio o los cambios importantes en los servicios que usa su cuenta) siempre se envían a la dirección de inicio de sesión.

## <a name="update-an-mosp-billing-account-address"></a>Actualización de la dirección de la cuenta de facturación de MOSP

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) mediante la dirección de correo electrónico que tenga el permiso de administrador de cuenta en la cuenta.
1. Busque **Administración de costos + facturación**.  
    ![Captura de pantalla que muestra la búsqueda en el portal para la administración de costos y facturación](./media/change-azure-account-profile/search-cmb.png)
1. Seleccione **Propiedades** en el lado izquierdo.  
    ![Captura de pantalla que muestra la página de la dirección de actualización](./media/change-azure-account-profile/update-contact-information-select-properties.png)
1. Seleccione **Actualizar dirección de facturación** para actualizar las direcciones del comprador y de facturación. Escriba la dirección nueva y seleccione **Guardar**.  
    ![Captura de pantalla que muestra la página de la dirección de actualización](./media/change-azure-account-profile/update-contact-information-mosp.png)

## <a name="update-an-mca-billing-account-sold-to-address"></a>Actualización de la dirección del comprador de una cuenta de facturación de MCA

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) mediante la dirección de correo electrónico, que tiene un rol de colaborador o propietario en la cuenta de facturación de un contrato de cliente de Microsoft.
1. Busque **Administración de costos + facturación**.  
    ![Captura de pantalla que muestra la búsqueda en el portal para la administración de costos y facturación](./media/change-azure-account-profile/search-cmb.png)
1. Seleccione **Propiedades** en el lado izquierdo y, después, seleccione **Update sold-to** (Actualizar comprador).  
    ![Captura de pantalla que muestra Update sold-to (Actualizar comprador) seleccionado](./media/change-azure-account-profile/update-sold-to-list-properties-mca.png)
1. Escriba la dirección nueva y seleccione **Guardar**.  
    ![Captura de pantalla que muestra la actualización de la dirección](./media/change-azure-account-profile/update-sold-to-save-mca.png)

    > [!IMPORTANT]
    > Algunas cuentas requieren una comprobación adicional para poder actualizar sus datos de comprador. Si su cuenta requiere aprobación manual, se le pedirá que se ponga en contacto con el soporte técnico de Azure.

## <a name="update-an-mca-billing-account-address"></a>Actualización de la dirección de la cuenta de facturación de MCA

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) mediante la dirección de correo electrónico, que tiene un rol de colaborador o propietario en una cuenta de facturación o un perfil de facturación de un MCA.
1. Busque **Administración de costos + facturación**.  
    ![Captura de pantalla que muestra la búsqueda en el portal para la administración de costos y facturación](./media/change-azure-account-profile/search-cmb.png)
1. Seleccione **Perfiles de facturación** en el lado izquierdo.
1. Seleccione un perfil de facturación para actualizar la dirección de facturación.  
    ![Captura de pantalla que muestra la búsqueda en el portal para la administración de cosos y facturación](./media/change-azure-account-profile/update-bill-to-list-profiles-mca.png)
1. Seleccione **Propiedades** en el lado izquierdo.
1. Seleccione **Actualizar dirección**.  
    ![Captura de pantalla que muestra la búsqueda en el portal para la administración de costos y facturación](./media/change-azure-account-profile/update-bill-to-list-properties-mca.png)
1. Escriba la dirección nueva y seleccione **Guardar**.  
    ![Captura de pantalla que muestra la actualización de la dirección](./media/change-azure-account-profile/update-bill-to-save-mca.png)

## <a name="service-and-marketing-emails"></a>Correos electrónicos de servicio y marketing

Se le solicita en [Azure Portal](https://portal.azure.com) que compruebe o actualice la dirección de correo electrónico cada 90 días. Microsoft envía correos electrónicos a esta dirección con información relacionada con la cuenta de Azure para:

- Notificaciones de servicio
- Alertas de seguridad
- Propósitos de facturación
- Soporte técnico
- Comunicaciones de marketing
- Procedimientos recomendados, según el uso de Azure

Escriba la dirección de correo electrónico donde quiera recibir comunicaciones sobre su cuenta. Al escribir una dirección de correo electrónico, está dando su consentimiento para recibir comunicaciones de Microsoft.

![Ejemplo de la solicitud para actualizar la información de contacto](./media/change-azure-account-profile/update-contact-information.png)

### <a name="change-your-contact-email-address"></a>Cambiar la dirección de correo electrónico de contacto

Para cambiar la dirección de correo electrónico de contacto, use uno de estos métodos. La actualización de la dirección de correo electrónico de contacto no actualiza la dirección de correo electrónico con la que se inicia sesión.

* Si es administrador de cuenta de una cuenta de MOSP, siga las instrucciones que encontrará en el apartado [Actualización de la dirección de la cuenta de facturación de MOSP](#update-an-mosp-billing-account-address) y seleccione **Actualizar información de contacto** en el último paso. A continuación, escriba la nueva dirección de correo electrónico.

* Vaya al área [Información de contacto](https://portal.azure.com/#blade/HubsExtension/ContactInfoBlade) en Azure Portal y escriba la nueva dirección de correo electrónico. 

* En [Azure Portal](https://portal.azure.com/#blade/HubsExtension/ContactInfoBlade), seleccione el icono con sus iniciales o imagen. A continuación, seleccione el menú de contexto ( **...** ). Seleccione **Mi información de contacto** en el menú y escriba la nueva dirección de correo electrónico.

![Ejemplo de actualización de una dirección de correo electrónico en Azure](./media/change-azure-account-profile/azure-contact-information.png)

### <a name="opt-out-of-marketing-emails"></a>Optar por no recibir correos electrónicos de marketing

Para optar por no recibir correos electrónicos de marketing:

1. Vaya al [formulario de solicitud](https://account.microsoft.com/profile/permissions-link-request) para enviar una solicitud utilizando su dirección de correo electrónico de perfil. Recibirá un vínculo por correo electrónico para actualizar sus preferencias.
1. Seleccione el vínculo para abrir la página **Administrar permisos de comunicación**. Esta página le muestra los tipos de comunicaciones de marketing en las que está habilitada la dirección de correo electrónico. Borre todas las opciones seleccionadas que quiera excluir y seleccione **Guardar**.  
    ![Ejemplo de la página para administrar los permisos de comunicación](./media/change-azure-account-profile/manage-communication-permissions.png)

Cuando opta por no recibir comunicaciones de marketing, seguirá recibiendo notificaciones del servicio, en función de su cuenta.

## <a name="update-the-email-address-that-you-sign-in-with"></a>Actualización de la dirección de correo electrónico con la que se inicia sesión

La dirección de correo electrónico que se usa para acceder a la cuenta propia no se puede actualizar. Sin embargo, si tiene una cuenta de facturación para un MOSP, puede iniciar registrarse en otra cuenta mediante la nueva dirección de correo electrónico y transferir la propiedad de las suscripciones a la siguiente cuenta. En el caso de una cuenta de facturación de MCA, [puede dar a la nueva dirección de correo electrónico permisos en su cuenta](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal).

## <a name="update-your-credit-card"></a>Actualización de la tarjeta de crédito

Para aprender a actualizar una tarjeta de crédito, consulte el artículo en que se explica cómo [cambiar la tarjeta de crédito usada para pagar una suscripción de Azure](change-credit-card.md).

## <a name="update-your-country-or-region"></a>Actualización de su país o región

No se admite el cambio de país o región de las cuentas existentes. Sin embargo, se puede crear una cuenta en otro país o región y, luego, ponerse en contacto con el soporte técnico de Azure para transferir una suscripción a la cuenta nueva.

## <a name="change-the-subscription-name"></a>Cambio del nombre de la suscripción

1. Inicie sesión en [Azure Portal](https://portal.azure.com), seleccione **Suscripción** en el panel izquierdo y después seleccione la suscripción cuyo nombre quiera cambiar.
1. Seleccione **Introducción** y después **Cambiar nombre** en la barra de comandos.  
    ![Ejemplo de cambio de nombre de una suscripción de Azure](./media/change-azure-account-profile/rename-sub.png)
1. Después de cambiar el nombre, seleccione **Guardar**.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- [Ver las cuentas de facturación](view-all-accounts.md)
