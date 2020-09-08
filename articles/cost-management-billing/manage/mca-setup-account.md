---
title: 'Configuración de la facturación del Contrato de cliente de Microsoft: Azure'
description: Aprenda a configurar su cuenta de facturación para un contrato de cliente de Microsoft. Consulte los requisitos previos para la configuración y vea los recursos adicionales disponibles.
author: amberbhargava
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: afa95609e0bcb879b6ad4b3b50b5dff5feb9a9f7
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682797"
---
# <a name="set-up-your-billing-account-for-a-microsoft-customer-agreement"></a>Configurar la cuenta de facturación para un contrato de cliente de Microsoft

Si su inscripción del Contrato Enterprise expiró o está a punto de expirar, puede firmar un contrato de cliente de Microsoft para renovarla. En este artículo se describen los cambios en la facturación existente después de la configuración y lo guiará por la configuración de la nueva cuenta de facturación. La renovación incluye los siguientes pasos:

1. Aceptar el nuevo contrato de cliente de Microsoft. Trabajar con su representante de campo de Microsoft para repasar los detalles y aceptar el nuevo contrato.
2. Configurar la nueva cuenta de facturación que se crea para el nuevo Contrato de cliente de Microsoft.

Para configurar la cuenta de facturación, debe pasar la facturación de las suscripciones de Azure desde su inscripción del Contrato Enterprise a la nueva cuenta. La configuración no afecta a los servicios de Azure que se ejecutan en sus suscripciones. Sin embargo, cambia la manera en que va a administrar la facturación para las suscripciones.

- En lugar del [portal EA](https://ea.azure.com), administrará los servicios y la facturación de Azure en [Azure Portal](https://portal.azure.com).
- Recibirá una factura mensual y digital de los cargos. Puede ver y analizar la factura en la página Azure Cost Management + Facturación.
- En lugar de los departamentos y la cuenta de su inscripción del Contrato Enterprise, usará la estructura y los ámbitos de facturación de la cuenta nueva para administrar y organizar la facturación.

Antes de comenzar la configuración, recomendamos que haga siguiente:

- **Descripción de la nueva cuenta de facturación**
  - La nueva cuenta simplifica la facturación para la organización. [Obtener una introducción rápida de la nueva cuenta de facturación](../understand/mca-overview.md)
- **Comprobar el acceso para completar la configuración**
  - Solo los usuarios con determinados permisos de administrador pueden completar la configuración. Compruebe si tiene el [acceso necesario para completar la configuración](#access-required-to-complete-the-setup).
- **Descripción de los cambios realizados en la jerarquía de facturación**
  - Que la nueva cuenta de facturación se organiza de forma diferente que la suscrición del Contrato Enterprise. [Descripción de los cambios realizados en la jerarquía de la nueva cuenta de facturación](#understand-changes-to-your-billing-hierarchy).
- **Descripción de los cambios del acceso de los administradores a la facturación**
  - Los administradores de la inscripción del Contrato Enterprise obtienen acceso a los ámbitos de facturación en la nueva cuenta. [Descripción de los cambios a su acceso](#changes-to-billing-administrator-access).
- **Ver las características del Contrato Enterprise que la nueva cuenta reemplaza**
  - Consulte las características de la inscripción del Contrato Enterprise que se reemplazan por las características de la nueva cuenta.
- **Ver las respuestas a preguntas más frecuentes**
  - Obtenga [información adicional](#additional-information) sobre la configuración.

## <a name="access-required-to-complete-the-setup"></a>Acceso requerido para completar la configuración

Para completar la configuración, necesita el siguiente acceso:

- Propietario del perfil de facturación que se creó en el momento en que se firmó el contrato del cliente de Microsoft. Para más información sobre los perfiles de facturación, consulte [Descripción de los perfiles de facturación](../understand/mca-overview.md#billing-profiles).

- Administrador de Enterprise en la inscripción que se renueva.

### <a name="if-youre-not-an-enterprise-administrator-on-the-enrollment"></a>Si no es un administrador de Enterprise en la inscripción

Puede pedirles a los administradores de Enterprise de la inscripción que completen la configuración de la cuenta de facturación.

1. Inicie sesión en Azure Portal mediante el vínculo en el correo electrónico que recibió en el momento de firmar el contrato del cliente de Microsoft.

2. Si no tiene el correo electrónico, inicie sesión con el siguiente vínculo. Reemplace `<enrollmentNumber>` por el número de inscripción de su Contrato Enterprise que se haya renovado.

   `https://portal.azure.com/#blade/Microsoft_Azure_EA/EATransitionToMCA/enrollmentId/<enrollmentNumber>`

3. Seleccione los administradores de Enterprise a los que desee enviar la solicitud.

   ![Captura de pantalla que muestra la invitación a los administradores de Enterprise](./media/mca-setup-account/ea-mca-invite-admins.png)

4. Seleccione **Enviar solicitud**.

   Los administradores recibirán un correo electrónico con instrucciones para completar la configuración.

### <a name="if-youre-not-an-owner-of-the-billing-profile"></a>Si no es un propietario del perfil de facturación

El usuario de su organización que firmó el contrato del cliente de Microsoft se agrega como propietario en el perfil de facturación. Para poder completar la configuración, solicite al usuario que lo agregue como propietario.

## <a name="understand-changes-to-your-billing-hierarchy"></a>Descripción de los cambios realizados en la jerarquía de facturación

La nueva cuenta de facturación simplifica la facturación de la organización al proporcionarle capacidades mejoradas de facturación y administración de costes. En el diagrama siguiente se explica cómo se organiza la facturación en la nueva cuenta de facturación.

![Imagen de ea-mca-post-transition-hierarchy](./media/mca-setup-account/mca-post-transition-hierarchy.png)

1. Use la cuenta de facturación para administrar la facturación de su contrato de cliente de Microsoft. Los administradores de Enterprise se convierten en los propietarios de la cuenta de facturación. Para obtener más información sobre la cuenta de facturación, consulte [Descripción de la cuenta de facturación](../understand/mca-overview.md#your-billing-account).
2. El perfil de facturación se usa para administrar la facturación de la organización, de forma similar a la inscripción del Contrato Enterprise. Los administradores de Enterprise se convierten en los propietarios del perfil de facturación. Para más información sobre los perfiles de facturación, consulte [Descripción de los perfiles de facturación](../understand/mca-overview.md#billing-profiles).
3. Se usa una sección de factura para organizar los costes según sus necesidades, de manera similar a los departamentos de la inscripción del Contrato Enterprise. El departamento se convierte en las secciones de la factura y los administradores de departamento se convierten en los propietarios de las respectivas secciones de la factura. Para más información sobre las secciones de la factura, consulte [Descripción de las secciones de factura](../understand/mca-overview.md#invoice-sections).
4. No se admiten las cuentas que se crearon en el Contrato Enterprise en la nueva cuenta de facturación. Las suscripciones de la cuenta pertenecen a la respectiva sección de facturas correspondiente al departamento. Los propietarios de la cuenta pueden crear y administrar suscripciones para sus secciones de factura.

## <a name="changes-to-billing-administrator-access"></a>Cambios en el acceso de administrador de facturación

Según su acceso, los administradores de facturación de la inscripción del Contrato Enterprise obtienen acceso a los ámbitos de factura en la nueva cuenta. En la tabla siguiente se explica el cambio en el acceso durante la configuración:

| Rol existente | Rol posterior a la transición |
| --- | --- |
| **Administrador de empresa (solo lectura = no)** | **- Propietario de la cuenta de facturación** </br> Administrar todo el contenido en la cuenta de facturación </br> **- Propietario del perfil de facturación** </br> Administrar todo el contenido en el perfil de facturación </br> **-Propietario de la sección de factura en todas las secciones de factura** </br> Administrar todo el contenido de las secciones de factura |
| **Administrador de empresa (solo lectura = sí)** | **- Lector de la cuenta de facturación** </br> Vista de solo lectura de todo en la cuenta de facturación</br> **-Lector del perfil de facturación** </br> Vista de solo lectura de todo en el perfil de facturación</br>**- Lector de la sección de factura en todas las secciones de la factura**</br> Visualización de solo lectura de todo el contenido en las secciones de factura|
| **Administrador de departamento (solo lectura)** |**- Propietario de la sección de factura en la sección de factura que creó para su respectivo departamento** </br>Administrar todo el contenido de la sección de factura|
| **Administrador de departamento (solo lectura = sí)**|**- Lector de la sección de factura en la sección de factura que creó para su respectivo departamento**</br> Vista de solo lectura de todo en la sección de factura|
| **Propietario de cuenta** | **- Creador de la suscripción de Azure en la sección de factura que creó para su respectivo departamento** </br>  Crear suscripciones de Azure para su sección de factura|

En el momento de aceptar el contrato de cliente de Microsoft, se selecciona un inquilino de Azure Active Directory (AD) para la nueva cuenta de facturación. Si no existe un inquilino para la organización, se crea uno nuevo. El inquilino representa a la organización en Azure Active Directory. Los administradores de inquilinos globales de la organización usan el inquilino para administrar el acceso de las aplicaciones y los datos de la organización.

La nueva cuenta solo admite usuarios del inquilino seleccionado en el momento de firmar el contrato de cliente de Microsoft. Si los usuarios con permisos de administración en el Contrato Enterprise forman parte del inquilino, tendrán acceso a la nueva cuenta de facturación durante la configuración. Si no forman parte del inquilino, no podrán acceder a la nueva cuenta de facturación, a menos que los invite.

Cuando invite a los usuarios, estos se agregan al inquilino como usuarios invitados y obtienen acceso a la cuenta de facturación. Para invitar a los usuarios, el acceso de invitado debe estar activado para el inquilino. Para más información, consulte [Controlar el acceso de invitado en Azure Active Directory](https://docs.microsoft.com/microsoftteams/teams-dependencies#control-guest-access-in-azure-active-directory). Si el acceso de invitado está desactivado, póngase en contacto con los administradores globales del inquilino para activarlo. <!-- Todo - How can they find their global administrator -->

## <a name="view-replaced-features"></a>Ver características reemplazadas

Las siguientes características del Contrato Enterprise se reemplazan por las nuevas características de la cuenta de facturación de un contrato de cliente de Microsoft.

### <a name="enterprise-agreement-accounts"></a>Cuentas del Contrato Enterprise

En la nueva cuenta de no se admiten las cuentas que se crearon en la inscripción del Contrato Enterprise. Las suscripciones de la cuenta pertenecen a la sección de factura que creó para su respectivo departamento. Los propietarios de cuenta se convierten en creadores de suscripción de Azure y pueden crear y administrar suscripciones para sus secciones de factura.

### <a name="notification-contacts"></a>Contactos de notificación

A los contactos de notificación se les envían comunicaciones por correo electrónico sobre el Contrato Enterprise de Azure. No se admiten en la nueva cuenta de facturación. Los correos electrónicos sobre los créditos y las facturas de Azure se envían a los usuarios que tienen acceso a los perfiles de facturación de la cuenta de facturación.

### <a name="spending-quotas"></a>Cuotas de gasto

Las cuotas de gastos que se establecieron para los departamentos de la inscripción del Contrato Enterprise se reemplazan por los presupuestos de la nueva cuenta de facturación. Se crea un presupuesto para cada cuota de gasto establecida en los departamentos de la inscripción. Para más información sobre los presupuestos, vea [Tutorial: Creación y administración de presupuestos en Azure](../costs/tutorial-acm-create-budgets.md).

### <a name="cost-centers"></a>Centros de coste

Los centro de costes que se establecieron en las suscripciones de Azure en la inscripción del Contrato Enterprise se trasladan en la nueva cuenta de facturación. Sin embargo, no se admiten centros de costes para los departamentos y las cuentas del Contrato Enterprise.

## <a name="additional-information"></a>Información adicional

En las secciones siguientes se proporciona información adicional sobre cómo configurar la cuenta de facturación.

### <a name="no-service-downtime"></a>No hay ningún tiempo de inactividad del servicio

Los servicios de Azure de la suscripción se siguen ejecutando sin ninguna interrupción. Solo hacemos la transición de la relación de facturación para sus suscripciones de Azure. Los recursos existentes, los grupos de recursos o los grupos de administración no se verán afectados.

### <a name="user-access-to-azure-resources"></a>Acceso de usuarios a los recursos de Azure

El acceso a los recursos de Azure que se configuró con Azure RBAC (control de acceso basado en rol) no se ve afectado durante la transición.

### <a name="azure-reservations"></a>Reservas de Azure

Las reservas de Azure en la inscripción del Contrato Enterprise se trasladan a la nueva cuenta de facturación. Durante la transición, no habrá ningún cambio en los descuentos de reserva que se aplican a sus suscripciones.

### <a name="azure-marketplace-products"></a>Productos de Azure Marketplace

Los productos de Azure Marketplace en la inscripción del Contrato Enterprise se trasladan junto con las suscripciones. No habrá ningún cambio en el acceso al servicio de los productos de Marketplace durante la transición.

### <a name="support-plan"></a>Plan de soporte técnico

Los beneficios de soporte técnico no se transfieren como parte de la transición. Para obtener los beneficios para las suscripciones de Azure en la nueva cuenta de facturación, compre un nuevo plan de soporte técnico.

### <a name="past-charges-and-balance"></a>Cargos y saldo del pasado

Los cargos y los saldos de créditos anteriores a la transición se pueden ver en la inscripción del Contrato Enterprise a través de Azure Portal. <!--Todo - Add a link for this-->

### <a name="when-should-the-setup-be-completed"></a>¿Cuándo se debe completar la configuración?

Complete la configuración de su cuenta de facturación antes de que expire su inscripción del Contrato Enterprise. Si expira la inscripción, los servicios de las suscripciones de Azure siguen en funcionando sin interrupciones. Sin embargo, para los servicios se le cobrará las tarifas de pago por uso.

### <a name="changes-to-the-enterprise-agreement-enrollment-after-the-setup"></a>Cambios realizados en la inscripción del Contrato Enterprise después de la configuración

Las suscripciones de Azure que se crean para la inscripción del Contrato Enterprise después de la transición se pueden mover manualmente a la nueva cuenta de facturación. Para más información, consulte [Obtención de la propiedad de la facturación de las suscripciones de otros usuarios](mca-request-billing-ownership.md). Para mover las reservas de Azure que se compraron después de la transición, [póngase en contacto con el soporte técnico de Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). También puede proporcionar acceso a los usuarios a la cuenta de facturación después de la transición. Para más información, consulte [Administración de roles de facturación en Azure Portal](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal).

### <a name="revert-the-transition"></a>Revertir la transición

La transición no se puede revertir. Una vez que se realiza la transición de la facturación de las suscripciones de Azure a la nueva cuenta de facturación, no puede revertirla a la inscripción del Contrato Enterprise.

### <a name="closing-your-browser-during-setup"></a>Cerrar el explorador durante la configuración

Antes de hacer clic en **Iniciar transición**, puede cerrar el explorador. Puede volver a la configuración mediante el vínculo que recibió en el correo electrónico y comenzar la transición. Si cierra el explorador una vez iniciada la transición, la transición seguirá ejecutándose. Vuelva a la página de estado de la transición para supervisar el estado más reciente de la transición. Recibirá un correo electrónico cuando se complete la transición.

## <a name="complete-the-setup-in-the-azure-portal"></a>Completar la configuración en Azure Portal

Para completar la configuración, necesita acceso a la nueva cuenta de facturación y a la inscripción del Contrato Enterprise. Para más información, consulte [Acceso necesario para completar la configuración de su cuenta de facturación](#access-required-to-complete-the-setup).

1. Inicie sesión en Azure Portal mediante el vínculo en el correo electrónico que recibió en el momento de firmar el contrato del cliente de Microsoft.

2. Si no tiene el correo electrónico, inicie sesión con el siguiente vínculo. Reemplace `<enrollmentNumber>` por el número de inscripción de su Contrato Enterprise que se haya renovado.

   `https://portal.azure.com/#blade/Microsoft_Azure_EA/EATransitionToMCA/enrollmentId/<enrollmentNumber>`

3. Seleccione **Iniciar transición** en el último paso de la configuración. Una vez que seleccione la transición de inicio:

    ![Captura de pantalla que muestra al Asistente para la instalación](./media/mca-setup-account/ea-mca-set-up-wizard.png)

    - En la nueva cuenta de facturación se crea una jerarquía de facturación que corresponde a la jerarquía del Contrato Enterprise. Para más información, consulte [Descripción de los cambios realizados en la jerarquía de facturación](#understand-changes-to-your-billing-hierarchy).
    - Los administradores de la inscripción del Contrato Enterprise obtienen acceso a la nueva cuenta de facturación para que sigan administrando la facturación de la organización.
    - La facturación de las suscripciones de Azure se traslada a la nueva cuenta. **No habrá ningún impacto en los servicios de Azure durante esta transición. Estos siguen en ejecución sin interrupciones**.
    - Si tiene reservas de Azure, estas se mueven a la nueva cuenta de facturación sin cambios en las ventajas o el plazo.

4. Puede supervisar el estado de la transición en la página **Estado de la transición**.

   ![Captura de pantalla que muestra el estado de transición](./media/mca-setup-account/ea-mca-set-up-status.png)

## <a name="validate-billing-account-setup"></a>Validación de la configuración de la cuenta de facturación

 Valide lo siguiente para asegurarse de que la nueva cuenta de facturación se configuró correctamente:

### <a name="azure-subscriptions"></a>Suscripciones de Azure

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Administración de costos + facturación**.

   ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/mca-setup-account/search-cmb.png)

3. Seleccione el nombre de la cuenta de facturación. La cuenta de facturación será del tipo **Contrato de cliente de Microsoft**.

4. Seleccione **Suscripciones de Azure** en el lado izquierdo.

   ![Captura de pantalla que muestra la lista de suscripciones](./media/mca-setup-account/mca-subscriptions-post-transition.png)

Las suscripciones de Azure de las que se realiza la transición desde la inscripción del Contrato Enterprise a la nueva cuenta de facturación se muestran en la página de suscripciones de Azure. Si cree que falta alguna suscripción, haga la transición de la facturación de la suscripción manualmente en Azure Portal. Para información, consulte [Obtención de la propiedad de la facturación de las suscripciones de otros usuarios](mca-request-billing-ownership.md).

### <a name="azure-reservations"></a>Reservas de Azure

Las reservas de Azure en la inscripción del Contrato Enterprise se trasladarán a la nueva cuenta de facturación sin cambios en las ventajas o el plazo. Las transacciones completadas antes de la transición no aparecerán en la nueva cuenta de facturación. Sin embargo, para comprobar que las ventajas de las reservas se aplican a las suscripciones, visite la [página Reservas de Azure](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade).

### <a name="access-of-enterprise-administrators-on-the-billing-account"></a>Acceso de los administradores de Enterprise en la cuenta de facturación

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Administración de costos + facturación**.

   ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/mca-setup-account/search-cmb.png)

3. Seleccione la cuenta de facturación para el **Contrato de cliente de Microsoft**.

4. Seleccione **Control de acceso (IAM)** en el lado izquierdo.

   ![Captura de pantalla que muestra el acceso de los administradores de Enterprise después de la transición](./media/mca-setup-account/mca-ea-admins-ba-access-post-transition.png)

Los administradores de Enterprise se muestran como propietarios de cuentas de facturación mientras que los administradores de Enterprise con permisos de solo lectura aparecen como lectores de cuentas de facturación. Si cree que falta acceso para algún administrador de Enterprise, puede concederle acceso en Azure Portal. Para más información, consulte [Administración de roles de facturación en Azure Portal](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal).

### <a name="access-of-enterprise-administrators-on-the-billing-profile"></a>Acceso de los administradores de Enterprise en el perfil de facturación

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Administración de costos + facturación**.

   ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/mca-setup-account/search-cmb.png)

3. Seleccione el perfil de facturación que creó para la inscripción. Dependiendo de su acceso, es posible que tenga que seleccionar una cuenta de facturación. En la cuenta de facturación, seleccione Perfiles de facturación y, luego, el perfil de facturación.

4. Seleccione **Control de acceso (IAM)** en el lado izquierdo.

   ![Captura de pantalla que muestra el acceso de los administradores de Enterprise después de la transición](./media/mca-setup-account/mca-ea-admins-bp-access-post-transition.png)

Los administradores de Enterprise se muestran como propietarios de perfiles de facturación mientras que los administradores de Enterprise con permisos de solo lectura se enumeran como lectores de perfil de facturación. Si cree que falta acceso para algún administrador de Enterprise, puede concederle acceso en Azure Portal. Para más información, consulte [Administración de roles de facturación en Azure Portal](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal).

### <a name="access-of-enterprise-administrators-department-administrators-and-account-owners-on-invoice-sections"></a>Acceso de los administradores de Enterprise, administradores de departamento y propietarios de cuentas en las secciones de factura

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Administración de costos + facturación**.

   ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/mca-setup-account/search-cmb.png).

3. Seleccione una sección de factura. Las secciones de factura tienen el mismo nombre que sus respectivos departamentos en las inscripciones del Contrato Enterprise. Dependiendo de su acceso, es posible que tenga que seleccionar una cuenta de facturación. En la cuenta de facturación, seleccione **Perfiles de facturación** y elija **Secciones de factura**. En la lista de secciones de factura, seleccione una sección de factura.

   ![Captura de pantalla que muestra una lista de secciones de factura después de la transición](./media/mca-setup-account/mca-invoice-sections-post-transition.png)

4. Seleccione **Control de acceso (IAM)** en el lado izquierdo.

    ![Captura de pantalla que muestra el acceso de administrador de departamento y cuenta después de la transición](./media/mca-setup-account/mca-department-account-admins-access-post-transition.png)

Los administradores de Enterprise y los administradores de departamento se enumeran como propietarios o propietarios de la sección de factura mientras que los propietarios de cuenta en el departamento se enumeran como creadores de suscripciones de Azure. Repita el paso para todas las secciones de factura comprobar el acceso para todos los departamentos de la inscripción del Contrato Enterprise. Los propietarios de cuenta que no formaban parte de ningún departamento obtendrán permiso en una sección de factura denominada **Default invoice section** (Sección de factura predeterminada). Si cree que falta acceso para algún administrador, puede concederle acceso en Azure Portal. Para más información, consulte [Administración de roles de facturación en Azure Portal](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal).

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Ponerse en contacto con soporte técnico

Si necesita ayuda, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a la nueva cuenta de facturación](../understand/mca-overview.md)

- [Finalización de tareas del Contrato Enterprise en la cuenta de facturación para un contrato de cliente de Microsoft](mca-enterprise-operations.md)

- [Administrar el acceso a la cuenta de facturación](understand-mca-roles.md)
