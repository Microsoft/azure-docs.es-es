---
title: Administración del acceso a la facturación de Azure
description: Obtenga información acerca de cómo acceder a la información de facturación de Azure para los miembros del equipo.
author: vikramdesai01
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.custom: seodec18
ms.openlocfilehash: 74c00678ca901d163d951e02167c89896c43e7b3
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281776"
---
# <a name="manage-access-to-billing-information-for-azure"></a>Administración del acceso a la información de facturación de Azure

Puede brindar a otros usuarios acceso a la información de facturación de su cuenta en Azure Portal. El tipo de roles de facturación y las instrucciones para proporcionar acceso a la información de facturación varía según el tipo de la cuenta de facturación. Para determinar el tipo de la cuenta de facturación, consulte [Comprobación del tipo de la cuenta de facturación](#check-the-type-of-your-billing-account).

Este artículo se aplica a los clientes con cuentas de Microsoft Online Service Program. Si es cliente de Azure con un Contrato Enterprise (EA) y es el administrador de Enterprise, puede otorgar permisos a los administradores de departamentos y a los propietarios de la cuenta en Enterprise Portal. Para más información, consulte [Roles administrativos del Contrato Enterprise de Azure en Azure](understand-ea-roles.md). Si es cliente con un contrato de cliente de Microsoft, consulte [Descripción de los roles administrativos del contrato de cliente de Microsoft en Azure](understand-mca-roles.md).

## <a name="account-administrators-for-microsoft-online-service-program-accounts"></a>Administradores de cuenta de las cuentas de Microsoft Online Service Program

Un administrador de cuenta es el único propietario de una cuenta de facturación de Microsoft Online Service Program. El rol está asignado a una persona que se suscribió en Azure. Los administradores de cuenta están autorizados para realizar varias tareas de facturación, como crear suscripciones, ver facturas o cambiar la facturación de una suscripción.

## <a name="give-others-access-to-view-billing-information"></a>Asignación de acceso a otros usuarios para ver la información de facturación

El administrador de cuenta puede conceder a otros usuarios acceso a la información de facturación de Azure mediante la asignación de uno de estos roles en una suscripción de su cuenta.

- Administrador de servicios
- Coadministrador
- Propietario
- Colaborador
- Lector
- Lector de facturación

Estos roles tienen acceso a la información de facturación en [Azure Portal](https://portal.azure.com/). Los usuarios que tienen asignados estos roles también pueden usar las [API de facturación](usage-rate-card-overview.md) para obtener detalles de uso y facturas mediante programación.

Para asignar los roles, consulte [Administración del acceso mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md).

** Si es cliente de EA, un propietario de la cuenta puede asignar el rol anterior a otros usuarios de su equipo. Pero para que estos usuarios puedan ver la información de facturación, el administrador de Enterprise debe permitir la visualización de cargos del PC en Enterprise Portal.


### <a name="allow-users-to-download-invoices"></a><a name="opt-in"></a> Permitir que los usuarios descarguen facturas

Una vez que un administrador de cuenta ha asignado los roles correspondientes a otros usuarios, estos deben activar el acceso para descargar facturas en Azure Portal. Las facturas anteriores a diciembre de 2016 solo están disponibles para el administrador de cuenta.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de cuenta.

1. Busque en **Administración de costos + facturación**.

    ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/manage-billing-access/billing-search-cost-management-billing.png)

1. Seleccione **Suscripciones** en el panel izquierdo. Según el acceso, es posible que deba seleccionar un ámbito de facturación y, a continuación, seleccione **Suscripciones**.

    ![Captura de pantalla que muestra la selección de las suscripciones](./media/manage-billing-access/billing-select-subscriptions.png)

1. Seleccione **Facturas** y, después, **Acceso a las facturas**.

    ![Captura de pantalla muestra cómo delegar acceso a las facturas](./media/manage-billing-access/aa-optin01.png)

1. Seleccione **On** (Activar) y guarde.

    ![Captura de pantalla muestra cómo activar o desactivar la delegación de acceso a la factura](./media/manage-billing-access/aa-optinallow01.png)

El administrador de la cuenta también puede configurar que las facturas se envíen por correo electrónico. Para obtener más información, vea [Obtención de la factura por correo electrónico](download-azure-invoice-daily-usage-date.md).

## <a name="give-read-only-access-to-billing"></a>Concesión de acceso de solo lectura a la facturación

Asigne el rol Lector de facturación a otro usuario que necesite tener acceso de solo lectura a la información de facturación de suscripción, pero no la capacidad de administrar ni crear los servicios de Azure. Este rol es adecuado para los usuarios de una organización que son responsables de la administración financiera y de costos para las suscripciones de Azure.

La característica Lector de facturación está en versión preliminar y aún no admite nubes no globales.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de cuenta.

1. Busque en **Administración de costos + facturación**.

    ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/manage-billing-access/billing-search-cost-management-billing.png)

1. Seleccione **Suscripciones** en el panel izquierdo. Según el acceso, es posible que deba seleccionar un ámbito de facturación y, a continuación, seleccione **Suscripciones**.

    ![Captura de pantalla que muestra la selección de las suscripciones](./media/manage-billing-access/billing-select-subscriptions.png)

1. Seleccione **Access Control (IAM)** .
1. Seleccione **Agregar** en la parte superior de la página.

    ![Captura de pantalla que muestra cuando se hace clic en Agregar asignación de roles](./media/manage-billing-access/billing-click-add-role-assignment.png)

1. En la lista desplegable **Rol**, elija **Lector de facturación**.
1. En el cuadro de texto **Seleccionar**, escriba el nombre o el correo electrónico del usuario que quiere agregar.
1. Seleccione el usuario.
1. Seleccione **Guardar**.
    ![Captura de pantalla que muestra cuando se hace clic en Agregar asignación de roles](./media/manage-billing-access/billing-save-role-assignment.png)

1. Transcurridos unos instantes, al usuario se le asigna el rol Lector de facturación para la suscripción.

** Si es un cliente de EA, un propietario de la cuenta o un administrador de departamento puede asignar el rol Lector de facturación a los miembros del equipo. Pero para que ese Lector de facturación vea la información de facturación correspondiente al departamento o la cuenta, el administrador de Enterprise debe habilitar las directivas **PC: ver los cargos** o **AD: ver los cargos** en Enterprise Portal.

## <a name="check-the-type-of-your-billing-account"></a>Comprobación del tipo de la cuenta de facturación
[!INCLUDE [billing-check-account-type](../../../includes/billing-check-account-type.md)]

## <a name="next-steps"></a>Pasos siguientes

- Los usuarios de otros roles, como Propietario o Colaborador, pueden tener acceso no solo a la información de facturación, sino también a los servicios de Azure. Para administrar estos roles, vea [Administración del acceso mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md).
- Para más información acerca de los roles, consulte [Roles integrados en Azure](../../role-based-access-control/built-in-roles.md).

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).
