---
title: Creación de una suscripción de Azure adicional
description: Aprenda a agregar una nueva suscripción de Azure en Azure Portal. Consulte información sobre los formularios de la cuenta de facturación y vea los recursos adicionales disponibles.
author: amberbhargava
ms.service: cost-management-billing
ms.topic: conceptual
ms.date: 02/12/2020
ms.author: banders
ms.openlocfilehash: d27120f6bd0978b69d664ab3ab2e86bfee4f1755
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87460973"
---
# <a name="create-an-additional-azure-subscription"></a>Creación de una suscripción de Azure adicional

Puede crear una suscripción adicional para su cuenta de facturación del [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/), [Contrato de cliente de Microsoft](https://azure.microsoft.com/pricing/purchase-options/microsoft-customer-agreement/) o [Microsoft Partner Agreement](https://www.microsoft.com/licensing/news/introducing-microsoft-partner-agreement) en Azure Portal. Es posible que quiera una suscripción adicional para evitar alcanzar los límites de suscripción, crear entornos independientes para seguridad o aislar los datos por motivos de cumplimiento.

Si tiene una cuenta de facturación de Microsoft online Service Program (MOSP), puede crear suscripciones adicionales en el [portal de inicio de sesión de Azure](https://account.azure.com/signup?offer=ms-azr-0003p).

Para más información sobre las cuentas de facturación y la identificación del tipo de la cuenta de facturación, consulte [Visualización de las cuentas de facturación en Azure Portal](view-all-accounts.md).

## <a name="permission-required-to-create-azure-subscriptions"></a>Permisos necesarios para crear suscripciones a Azure

Necesita los siguientes permisos para crear suscripciones:

|Cuenta de facturación  |Permiso  |
|---------|---------|
|Contrato Enterprise (EA) |  Rol de propietario de la cuenta en la inscripción del Contrato Enterprise. Para más información, consulte [Roles administrativos del Contrato Enterprise de Azure en Azure](understand-ea-roles.md).    |
|Contrato de cliente de Microsoft (MCA) |  Rol de propietario o colaborador en la sección factura, perfil de facturación o cuenta de facturación. O rol de creador de suscripciones a Azure en la sección de factura.  Para más información, consulte [Tareas y roles de la facturación de suscripción](understand-mca-roles.md#subscription-billing-roles-and-tasks).    |
|Microsoft Partner Agreement (MPA) |   Roles Administrador global y Agente de administrador en la organización de asociado de CSP. Para más información, consulte [Centro de partners: Asignar roles y permisos de usuarios](https://docs.microsoft.com/partner-center/permissions-overview).  El usuario debe iniciar sesión en el inquilino de asociados para crear suscripciones de Azure.   |

## <a name="create-a-subscription-in-the-azure-portal"></a>Creación de una suscripción en Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque **Suscripciones**.

   ![Captura de pantalla que muestra la búsqueda de suscripciones en el portal](./media/create-subscription/billing-search-subscription-portal.png)

1. Seleccione **Agregar**.

   ![Captura de pantalla que muestra el botón Agregar en la vista Suscripciones](./media/create-subscription/subscription-add.png)

1. Si tiene acceso a varias cuentas de facturación, seleccione la cuenta de facturación para la que desea crear la suscripción.

1. Complete el formulario y haga clic en **Crear**. En las tablas siguientes se enumeran los campos del formulario para cada tipo de cuenta de facturación.

**Contrato Enterprise**

|Campo  |Definición  |
|---------|---------|
|Nombre     | El nombre para mostrar ayuda a identificar fácilmente la suscripción en Azure Portal.  |
|Oferta     | Seleccione Desarrollo/pruebas EA si va a usar esta suscripción para las cargas de trabajo de desarrollo o pruebas; si no, utilice Microsoft Azure Enterprise. La oferta de Desarrollo/pruebas debe estar habilitada para que la cuenta de inscripción cree suscripciones de desarrollo/pruebas del Contrato Enterprise.|

**Contrato de cliente de Microsoft**

|Campo  |Definición  |
|---------|---------|
|Perfil de facturación     | Los cargos correspondientes a su suscripción se facturarán en el perfil de facturación que haya seleccionado. Si tiene acceso a solo un perfil de facturación, se atenúa la selección.     |
|Sección de factura     | Los cargos de la suscripción aparecerán en esta sección de factura del perfil de facturación. Si tiene acceso a solo una sección de factura, se atenúa la selección.  |
|Plan     | Seleccione Plan de Microsoft Azure para Desarrollo/pruebas si va a usar esta suscripción para las cargas de trabajo de desarrollo o pruebas; si no, utilice Plan de Microsoft Azure. Si solo hay un plan para el perfil de facturación, la selección aparecerá atenuada.  |
|Nombre     | El nombre para mostrar ayuda a identificar fácilmente la suscripción en Azure Portal.  |

**Microsoft Partner Agreement**

|Campo  |Definición  |
|---------|---------|
|Customer    | La suscripción se crea para el cliente que seleccione. Si solo tiene un cliente, la selección aparece atenuada.  |
|Vendedor    | El revendedor que proporciona servicios al cliente. Este campo es opcional, solo se aplica a los proveedores indirectos en el modelo de dos niveles de CSP. |
|Nombre     | El nombre para mostrar ayuda a identificar fácilmente la suscripción en Azure Portal.  |

## <a name="create-an-additional-azure-subscription-programmatically"></a>Creación de una suscripción a Azure adicional mediante programación

También puede crear suscripciones adicionales mediante programación. Para más información, consulte [Creación de suscripciones a Azure Enterprise mediante programación](../../azure-resource-manager/management/programmatically-create-subscription.md).

## <a name="next-steps"></a>Pasos siguientes

- [Adición o cambio de los administradores de la suscripción de Azure](add-change-subscription-administrator.md)
- [Move resources to new resource group or subscription](../../azure-resource-manager/management/move-resource-group-and-subscription.md) (Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción)
- [Creación de grupos de administración para la organización y la administración de recursos](../../governance/management-groups/create.md)
- [Cancelación de la suscripción de Azure](cancel-azure-subscription.md)

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).
