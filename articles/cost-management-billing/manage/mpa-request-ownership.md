---
title: Obtención de la propiedad de la facturación de las suscripciones a Azure para Microsoft Partner Agreement (MPA)
description: Aprenda a solicitar la propiedad de la facturación de las suscripciones a Azure de otros usuarios.
author: amberbhargava
tags: billing
ms.service: cost-management-billing
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: a0643b42a7d306a9e41a80cb8529926eff42c00a
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289006"
---
# <a name="get-billing-ownership-of-azure-subscriptions-to-your-mpa-account"></a>Obtención de la propiedad de la facturación de las suscripciones a Azure para la cuenta de MPA

Para proporcionar una única factura combinada que incluya los servicios administrados y el consumo de Azure, el proveedor de soluciones en la nube (CSP) puede asumir la propiedad de la facturación de las suscripciones a Azure de sus clientes con el Contrato Enterprise (EA) directo.

Esta característica solo está disponible para los asociados de factura directa de CSP certificados como [MSP expertos de Azure](https://partner.microsoft.com/membership/azure-expert-msp). Está sujeta a las directivas y a la gobernanza de Microsoft, y puede requerir la revisión y aprobación en el caso de determinados clientes.

Para solicitar la propiedad de la facturación, debe tener el rol **Administrador global**  o **Agentes de administrador**. Para más información, consulte [Centro de partners: Asignar roles y permisos de usuarios](https://docs.microsoft.com/partner-center/permissions-overview).

Este artículo se aplica a las cuentas de facturación de Microsoft Partner Agreement. Estas cuentas se crean para que los proveedores de soluciones en la nube (CSP) administren la facturación de sus clientes en la nueva experiencia comercial. La nueva experiencia solo está disponible para asociados que tengan al menos un cliente que haya aceptado un Contrato de cliente de Microsoft (MCA) y que tenga un plan de Azure. [Compruebe si tiene acceso a un contrato Microsoft Partner Agreement](#check-access-to-a-microsoft-partner-agreement).

## <a name="prerequisites"></a>Requisitos previos

1. Establezca la [relación de revendedor](https://docs.microsoft.com/partner-center/request-a-relationship-with-a-customer) con el cliente. Consulte [Introducción a la autorización regional de CSP](https://docs.microsoft.com/partner-center/regional-authorization-overview) para asegurarse de que el cliente y el inquilino del asociado estén dentro de las mismas regiones autorizadas.
1. [Confirme que el cliente haya aceptado el contrato de cliente de Microsoft](https://docs.microsoft.com/partner-center/confirm-customer-agreement).
1. Configure un [plan de Azure](https://docs.microsoft.com/partner-center/purchase-azure-plan) para el cliente. Si el cliente está comprando a través de varios revendedores, debe configurar un plan de Azure para cada combinación de cliente y revendedor.

## <a name="request-billing-ownership"></a>Solicitud de propiedad de la facturación

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con las credenciales del agente de administrador de CSP en el inquilino de CSP.
1. Busque **Administración de costos + facturación**.  
    ![Captura de pantalla que muestra la búsqueda en Azure Portal de Administración de costos + facturación](./media/mpa-request-ownership/search-cmb.png)
1. Seleccione **Clientes** en la parte izquierda y, a continuación, un cliente de la lista.  
    [![Captura de pantalla que muestra la selección de clientes](./media/mpa-request-ownership/mpa-select-customers.png)](./media/mpa-request-ownership/mpa-select-customers.png#lightbox)
1. Seleccione **Solicitudes de transferencia** en el lado inferior izquierdo y, luego, **Agregar una nueva solicitud**.  
    [![Captura de pantalla que muestra la selección de solicitudes de transferencia](./media/mpa-request-ownership/mpa-select-transfer-requests.png)](./media/mpa-request-ownership/mpa-select-transfer-requests.png#lightbox)
1. Escriba la dirección de correo electrónico del usuario en la organización del cliente que va a aceptar la solicitud de transferencia. El usuario debe ser propietario de la cuenta en un Contrato Enterprise. Seleccione **Enviar solicitud de transferencia**.  
    [![Captura de pantalla que muestra el envío de una solicitud de transferencia](./media/mpa-request-ownership/mpa-send-transfer-requests.png)](./media/mpa-request-ownership/mpa-send-transfer-requests.png#lightbox)
1. El usuario recibe un correo electrónico con instrucciones para revisar la solicitud de transferencia.  
    ![Captura de pantalla que muestra un correo electrónico de solicitud de transferencia](./media/mpa-request-ownership/mpa-review-transfer-request-email.png)
1. Para aprobar la solicitud de transferencia, el usuario selecciona el vínculo en el correo electrónico y sigue las instrucciones.  
    [![Captura de pantalla que muestra la revisión de la solicitud de transferencia](./media/mpa-request-ownership/review-transfer-requests.png)](./media/mpa-request-ownership/review-transfer-requests.png#lightbox) El usuario puede seleccionar la cuenta de facturación desde la que desea transferir productos de Azure. Una vez seleccionada, se muestran los productos que se pueden transferir. **Nota:** las suscripciones deshabilitadas no se pueden transferir y se mostrarán en la lista de productos de Azure que no se pueden transferir, si procede. Una vez seleccionados los productos de Azure que se van a transferir, seleccione **Validar**.
1. El área **Transfer Validation Result** (Resultado de validación de transferencia) mostrará el impacto de los productos de Azure que se van a transferir. Estos son los posibles estados:
    * **Superado**: se ha superado la validación de este producto de Azure y se puede transferir.
    * **Advertencia**: hay una advertencia para el producto de Azure seleccionado. Aunque el producto se puede transferir, el usuario debe tener en cuenta que esto tendrá consecuencias, por si desea realizar acciones encaminadas a reducirlas. Por ejemplo, la suscripción de Azure que se va a transferir se beneficia de una instancia reservada de máquina virtual. Después de la transferencia, dicha suscripción dejará de recibir esa ventaja. Para maximizar el ahorro, asegúrese de que la instancia reservada de máquina virtual esté asociada a otra suscripción que puede usar sus ventajas. En su lugar, el usuario también puede elegir volver a la página de selección y anular la selección de esta suscripción de Azure.
    * **Error**: el producto de Azure seleccionado no se puede transferir debido a un error. El usuario tendrá que volver a la página de selección y anular la selección de este producto para transferir los restantes productos de Azure seleccionados.  
    ![Captura de pantalla que muestra la experiencia de validación](./media/mpa-request-ownership/validate-transfer-request.png)

## <a name="check-the-transfer-request-status"></a>Comprobación del estado de una solicitud de transferencia

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque **Administración de costos + facturación**.  
    ![Captura de pantalla que muestra la búsqueda en Azure Portal de Administración de costos + facturación](./media/mpa-request-ownership/billing-search-cost-management-billing.png)
1. Seleccione **Clientes** en el lado izquierdo.  
    [![Captura de pantalla que muestra la selección de clientes](./media/mpa-request-ownership/mpa-select-customers.png)](./media/mpa-request-ownership/mpa-select-customers.png#lightbox)
1. Seleccione en la lista el cliente para el que envío la solicitud de transferencia.
1. Seleccione **Solicitudes de transferencia** en el lado inferior izquierdo. En la página Solicitudes de transferencia se muestra la siguiente información: [![Captura de pantalla que muestra una lista de solicitudes de transferencia](./media/mpa-request-ownership/mpa-select-transfer-requests-for-status.png)](./media/mpa-request-ownership/mpa-select-transfer-requests-for-status.png#lightbox)

   |Columna|Definición|
   |---------|---------|
   |Fecha de solicitud|La fecha en la que se envió la solicitud de transferencia.|
   |Recipient|La dirección de correo electrónico del usuario que envió la solicitud para transferir la propiedad de la facturación|
   |Fecha de expiración|La fecha en que expira la solicitud|
   |Estado|El estado de la solicitud de transferencia|

    La solicitud de transferencia puede tener uno de los siguientes estados:

   |Estado|Definición|
   |---------|---------|
   |En curso|El usuario no ha aceptado la solicitud de transferencia.|
   |Processing|El usuario aprueba la solicitud de transferencia. La facturación de las suscripciones seleccionadas por el usuario está en proceso de transferencia a su cuenta.|
   |Completed| La facturación de las suscripciones seleccionadas por el usuario está transferida a su cuenta.|
   |Finalizado con errores|La solicitud se ha completado pero la facturación de algunas suscripciones que el usuario ha seleccionado no se pudo transferir.|
   |Expirada|El usuario no aceptó la solicitud a tiempo y ha expirado.|
   |Canceled|Un usuario con acceso a la solicitud de transferencia ha cancelado la solicitud.|
   |Rechazado|El usuario ha rechazado la solicitud de transferencia.|

1. Seleccione una solicitud de transferencia para ver los detalles. En la página de detalles de la transferencia se muestra la siguiente información: [![Captura de pantalla que muestra la lista de las suscripciones transferidas](./media/mpa-request-ownership/mpa-transfer-completed.png)](./media/mpa-request-ownership/mpa-transfer-completed.png#lightbox)

   |Columna  |Definición|
   |---------|---------|
   |Identificador de la solicitud de transferencia|El identificador único de la solicitud de transferencia. Si envía una solicitud de soporte técnico, comparta el identificador con el Soporte técnico de Azure para agilizar la solicitud.|
   |Transferencia solicitada el|La fecha en la que se envió la solicitud de transferencia.|
   |Transferencia solicitada por|La dirección de correo electrónico del usuario que envió la solicitud de transferencia.|
   |La solicitud de transferencia caduca el| La fecha en la que expira la solicitud de transferencia.|
   |Dirección de correo electrónico del destinatario|La dirección de correo electrónico del usuario que envió la solicitud para transferir la propiedad de la facturación|
   |Vínculo de transferencia enviado al destinatario|La dirección URL que se envió al usuario para revisar la solicitud de transferencia.|

## <a name="supported-subscription-types"></a>Tipos de suscripciones admitidos

Puede solicitar la propiedad de facturación de los tipos de suscripción que se describen a continuación.

* [Desarrollo/pruebas - Enterprise](https://azure.microsoft.com/offers/ms-azr-0148p/)\*
* [Contrato Enterprise (EA) de Microsoft](https://azure.microsoft.com/pricing/enterprise-agreement/)

\* Debe convertir una suscripción de Desarrollo/pruebas en una oferta de Contrato Enterprise mediante una incidencia de soporte técnico. Una suscripción de Desarrollo/pruebas - Enterprise se facturará según la tarifa de pago por uso después de transferirla. Cualquier descuento ofrecido a través de la oferta de Desarrollo/pruebas - Enterprise mediante el Contrato Enterprise del cliente no estará disponible para el asociado de CSP.

## <a name="additional-information"></a>Información adicional

En la sección siguiente se proporciona información adicional acerca de las suscripciones de transferencia.

### <a name="no-service-downtime"></a>No hay ningún tiempo de inactividad del servicio

Los servicios de Azure de la suscripción se siguen ejecutando sin interrupción. Solo cambiamos la relación de facturación de las suscripciones de Azure que el usuario selecciona para transferir.

### <a name="disabled-subscriptions"></a>Suscripciones deshabilitadas

Las suscripciones deshabilitadas no se pueden transferir. Las suscripciones deben estar en estado activo para transferir su propiedad de la facturación.

### <a name="azure-resources-transfer"></a>Transferencia de recursos de Azure

Todos los recursos de las suscripciones, como máquinas virtuales, discos y sitios web se transfieren.

### <a name="azure-marketplace-products-transfer"></a>Transferencia de productos de Azure Marketplace

Los productos de Azure Marketplace, que están disponibles para las suscripciones administradas por proveedores de soluciones en la nube (CSP), se transfieren junto con sus respectivas suscripciones. No se pueden transferir las suscripciones que tengan productos de Azure Marketplace no habilitados para CSP.

### <a name="azure-reservations-transfer"></a>Transferencia de reservas de Azure

Las reservas de Azure no se mueven automáticamente con las suscripciones. Se pueden mantener en EA para otras suscripciones o [cancelar](https://docs.microsoft.com/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations) y que los asociados las puedan comprar de nuevo en CSP.

### <a name="access-to-azure-services"></a>Acceso a servicios de Azure

El acceso de los usuarios, grupos o entidades de servicio existentes que se asignó mediante [Azure RBAC (control de acceso basado en rol)](../../role-based-access-control/overview.md)no se ve afectado durante la transición. El asociado no obtendrá ningún nuevo acceso RBAC a las suscripciones.

Los asociados deben trabajar con el cliente para obtener acceso a las suscripciones. Los asociados deben tener incidencias de soporte técnico abiertas de acceso de [Administración en nombre de - AOBO ](https://channel9.msdn.com/Series/cspdev/Module-11-Admin-On-Behalf-Of-AOBO) o [Azure Lighthouse](https://docs.microsoft.com/azure/lighthouse/concepts/cloud-solution-provider).

### <a name="azure-support-plan"></a>Plan de soporte técnico de Azure

El soporte técnico de Azure no se transfiere con las suscripciones. Si el usuario transfiere todas las suscripciones de Azure, pídale que cancele su plan de soporte técnico. Después de la transferencia, el asociado de CSP es responsable del soporte técnico. El cliente debe trabajar con el asociado de CSP para las solicitudes de soporte técnico.  

### <a name="charges-for-transferred-subscription"></a>Cargos por la suscripción transferida

El propietario de la facturación original de las suscripciones es responsable de cualquier cargo que se haya informado hasta el momento en que se complete la transferencia. Es responsable de los cargos notificados desde el momento de la transferencia en adelante. Puede haber algunos cargos que se realizaran antes de la transferencia, pero que se notificaron con posterioridad. Estos cargos se muestran en la factura.

### <a name="cancel-a-transfer-request"></a>Cancelación de una solicitud de transferencia

Puede cancelar la solicitud de transferencia hasta que se apruebe o rechace la solicitud. Para cancelar la solicitud de transferencia, vaya a la [página de detalles de la transferencia](#check-the-transfer-request-status) y seleccione Cancelar en la parte inferior de la página.

### <a name="software-as-a-service-saas-transfer"></a>Transferencia del Software como servicio (SaaS)

No se transfieren los productos de SaaS con las suscripciones. Pida al usuario que [se ponga en contacto con el soporte técnico de Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para transferir la propiedad de la facturación de los productos de SaaS. Junto con la propiedad de la facturación, el usuario también puede transferir la propiedad de los recursos. La propiedad de los recursos le permite realizar operaciones de administración como eliminar y consultar los detalles del producto. El usuario debe ser propietario de un recurso en el producto de SaaS para poder transferir su propiedad.

### <a name="additional-approval-for-certain-customers"></a>Aprobación adicional para determinados clientes

Algunas de las solicitudes de transición del cliente pueden requerir un proceso de revisión adicional con Microsoft debido a la naturaleza de la estructura actual de inscripción de la empresa del cliente. El asociado recibirá notificaciones de estos requisitos cuando intenten enviar una invitación a los clientes. Se solicita a los asociados que trabajen con el administrador de desarrollo de asociados y el equipo de cuentas de cliente para completar este proceso de revisión.

### <a name="azure-subscription-directory"></a>Directorio de suscripciones a Azure

El directorio de las suscripciones a Azure que se transfieren debe coincidir con el directorio del cliente que se seleccionó al establecer la relación de CSP.

Si estos dos directorios no coinciden, no se pueden transferir las suscripciones. Debe establecer una nueva relación de revendedor de CSP con el cliente seleccionando el directorio de las suscripciones a Azure o cambiándolo para que coincida con el directorio de la relación de CSP con el cliente. Para más información, consulte [Asociación de una suscripción existente al directorio de Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory#to-associate-an-existing-subscription-to-your-azure-ad-directory).

### <a name="ea-subscription-in-the-non-orgnization-directory"></a>Suscripción a EA en un directorio que no es de la organización

Las suscripciones EA de directorios que no son de la organización se pueden transferir siempre que el directorio tenga una relación de revendedor con el CSP. Si el directorio no tiene una relación de revendedor, debe asegurarse de tener un usuario de la organización en el directorio como *administrador global* que pueda aceptar la relación de asociado. La parte del nombre de dominio del nombre de usuario debe ser el nombre de dominio predeterminado inicial "[nombre de dominio]. onmicrosoft.com" o un nombre de dominio personalizado no federado comprobado, como"contoso.com".  

Para agregar un nuevo usuario al directorio, consulte [Inicio rápido: Adición de nuevos usuarios a Azure Active Directory](https://docs.microsoft.com/azure/active-directory/add-users-azure-active-directory) para agregar el nuevo usuario al directorio de AAD.

## <a name="check-access-to-a-microsoft-partner-agreement"></a>Comprobación del acceso a un contrato Microsoft Partner Agreement

[!INCLUDE [billing-check-mpa](../../../includes/billing-check-mpa.md)]

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Ponerse en contacto con soporte técnico

Si necesita ayuda, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.

## <a name="next-steps"></a>Pasos siguientes

* Se le transfiere la propiedad de la facturación de las suscripciones a Azure. Realice un seguimiento de los cargos de estas suscripciones en [Azure Portal](https://portal.azure.com).
* Trabaje con el cliente para obtener acceso a las suscripciones a Azure transferidas. [Administración del acceso a los recursos de Azure mediante RBAC](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal).
