---
title: 'Roles de facturación del Contrato de cliente de Microsoft: Azure'
description: Aprenda sobre los roles de facturación para las cuentas de facturación de Azure para los contratos de cliente de Microsoft.
author: amberbhargava
ms.service: cost-management-billing
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: 84cb68c4cae962fd985a02a72cc85c6b2cf1aa1d
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286587"
---
# <a name="understand-microsoft-customer-agreement-administrative-roles-in-azure"></a>Descripción de los roles administrativos del contrato de cliente de Microsoft en Azure

Para administrar la cuenta de facturación para un contrato de cliente de Microsoft, use los roles descritos en las secciones siguientes. Estos roles son adicionales a los roles integrados que tiene Azure para controlar el acceso a los recursos. Para más información, consulte [Roles integrados en Azure](../../role-based-access-control/built-in-roles.md).

Este artículo se aplica a una cuenta de facturación para un contrato de cliente de Microsoft. Compruebe si tiene acceso a un contrato de cliente de Microsoft.

## <a name="billing-role-definitions"></a>Definiciones de roles de facturación

En la tabla siguiente se describen los roles de facturación que se utilizan para administrar la cuenta de facturación, los perfiles de facturación y las secciones de factura.

|Role|Descripción|
|---|---|
|Propietario de la cuenta de facturación|Administrar todo para la cuenta de facturación|
|Colaborador de la cuenta de facturación|Administrar todo, excepto los permisos, en la cuenta de facturación|
|Lector de la cuenta de facturación|Vista de solo lectura de todo en la cuenta de facturación|
|Propietario del perfil de facturación|Administrar todo para el perfil de facturación|
|Colaborador del perfil de facturación|Administrar todo, excepto los permisos, en el perfil de facturación|
|Lector del perfil de facturación|Vista de solo lectura de todo en el perfil de facturación|
|Administrador de facturación|Ver y pagar las facturas para el perfil de facturación|
|Propietario de la sección de facturas|Administrar todo en la sección de factura|
|Colaborador de la sección de factura|Administrar todo, excepto los permisos, en la sección de factura|
|Lector de la sección de facturas|Vista de solo lectura de todo en la sección de factura|
|Creador de la suscripción a Azure|Creación de suscripciones a Azure|

## <a name="billing-account-roles-and-tasks"></a>Tareas y roles de la cuenta de facturación

La cuenta de facturación le permite administrar la facturación de la organización. Utilice la cuenta de facturación para organizar costos, supervisar los cargos y facturas y controlar el acceso de facturación para la organización. Para más información, consulte [Descripción de la cuenta de facturación](../understand/mca-overview.md#your-billing-account).

En las tablas siguientes se muestra qué rol necesita para completar las tareas en el contexto de la cuenta de facturación.

### <a name="manage-billing-account-permissions-and-properties"></a>Administración de las propiedades y permisos de la cuenta de facturación

|Tarea|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación|
|---|---|---|---|
|Ver los permisos existentes para la cuenta de facturación|✔|✔|✔|
|Conceder a otros usuarios permisos para consultar y administrar la cuenta de facturación|✔|✘|✘|
|Consultar las propiedades de la cuenta de facturación, como el nombre de la compañía, la dirección y mucho más|✔|✔|✔|

### <a name="manage-billing-profiles-for-billing-account"></a>Administrar perfiles de facturación para la cuenta de facturación

|Tarea|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación|
|---|---|---|---|
|Ver todos los perfiles de facturación de la cuenta|✔|✔|✔|

### <a name="manage-invoices-for-billing-account"></a>Administrar las facturas de la cuenta de facturación

|Tarea|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación|
|---|---|---|---|
|Consultar todas las facturas de la cuenta|✔|✔|✔|
|Descargar facturas, los archivos de uso y de cargos de Azure, las hojas de precios y los documentos fiscales de la cuenta|✔|✔|✔|

### <a name="manage-invoice-sections-for-billing-account"></a>Administrar las secciones de factura para la cuenta de facturación

|Tarea|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación|
|---|---|---|---|
|Ver todas las secciones de factura de la cuenta|✔|✔|✔|

### <a name="manage-transactions-for-billing-account"></a>Administrar las transacciones para la cuenta de facturación

|Tarea|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación|
|---|---|---|---|
|Ver todas las transacciones de facturación para la cuenta|✔|✔|✔|
|Ver todos los productos comprados para la cuenta|✔|✔|✔|

### <a name="manage-subscriptions-for-billing-account"></a>Administración de suscripciones para la cuenta de facturación

|Tarea|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación|
|---|---|---|---|
|Ver todas las suscripciones de Azure en la cuenta de facturación|✔|✔|✔|

## <a name="billing-profile-roles-and-tasks"></a>Tareas y roles del perfil de facturación

El perfil de facturación le permite administrar las facturas y los métodos de pago. Se genera una factura mensual para las suscripciones de Azure y otros productos comprados mediante el perfil de facturación. Utilice los métodos de pago para pagar la factura. Para más información, consulte [Descripción de los perfiles de facturación](../understand/mca-overview.md#billing-profiles).

En las tablas siguientes se muestra qué rol necesita para completar las tareas en el contexto del perfil de facturación.

### <a name="manage-billing-profile-permissions-properties-and-policies"></a>Administración de las directivas, propiedades y permisos de perfil de facturación

|Tarea|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación|Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver los permisos existentes para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|
|Conceder a otros usuarios permisos para consultar y administrar el perfil de facturación|✔|✘|✘|✘|✘|✘|✘|
|Consultar las propiedades del perfil de facturación, como el número de pedido de compra, las preferencias de la factura de correo electrónico y mucho más|✔|✔|✔|✔|✔|✔|✔|
|Actualizar las propiedades del perfil de facturación |✔|✔|✘|✘|✘|✘|✘|
|Ver las directivas aplicadas en el perfil de facturación, como habilitar las compras de reservas de Azure, habilitar las compras de Azure Marketplace, y mucho más|✔|✔|✔|✔|✔|✔|✔|
|Aplicar las directivas en el perfil de facturación |✔|✔|✘|✘|✘|✘|✘|

### <a name="manage-invoices-for-billing-profile"></a>Administrar las facturas para el perfil de facturación

|Tarea|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación|Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver todas las facturas para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|
|Descargar facturas, los archivos de uso y de cargos de Azure, las hojas de precios y los documentos fiscales para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|

### <a name="manage-invoice-sections-for-billing-profile"></a>Administración de las secciones de factura para el perfil de facturación

|Tarea|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación|Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver todas las secciones de factura para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|
|Crear una nueva sección de factura para el perfil de facturación|✔|✔|✘|✘|✘|✘|✘|

### <a name="manage-transactions-for-billing-profile"></a>Administrar las transacciones para el perfil de facturación

|Tarea|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación|Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver todas las transacciones de facturación para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|

### <a name="manage-payment-methods-for-billing-profile"></a>Administrar los métodos de pago para el perfil de facturación

|Tarea|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación|Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver los métodos de pago para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|
|Realizar el seguimiento del saldo del crédito de Azure para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|

### <a name="manage-subscriptions-for-billing-profile"></a>Administrar las suscripciones para el perfil de facturación

|Tarea|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación|Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver todas las suscripciones de Azure para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|

## <a name="invoice-section-roles-and-tasks"></a>Tareas y roles de la sección de factura

Una sección de factura le permite organizar los costos en la factura. Puede crear una sección para organizar los costos por departamento, entorno de desarrollo o en función de las necesidades de la organización. Conceda permiso a otros usuarios para crear suscripciones a Azure para la sección. Los cargos por uso y las compras de las suscripciones se muestran entonces en la sección de la factura. Para más información, consulte [Descripción de las secciones de factura](../understand/mca-overview.md#invoice-sections).

En las tablas siguientes se muestra qué rol necesita para completar las tareas en el contexto de las secciones de factura.

### <a name="manage-invoice-section-permissions-and-properties"></a>Administración de los permisos y propiedades de la sección de factura

|Tareas|Propietario de la sección de facturas|Colaborador de la sección de factura|Lector de la sección de facturas|Creador de la suscripción a Azure|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación |
|---|---|---|---|---|---|---|---|
|Ver todos los permisos en la sección de factura|✔|✔|✔|✔|✔|✔|✔|
|Conceder a otros usuarios permisos para consultar y administrar la sección de factura|✔|✘|✘|✘|✘|✘|✘|
|Ver las propiedades de la sección de factura|✔|✔|✔|✔|✔|✔|✔|
|Actualizar las propiedades de la sección de factura|✔|✔|✘|✘|✘|✘|✘|

### <a name="manage-products-for-invoice-section"></a>Administrar los productos para la sección de factura

|Tareas|Propietario de la sección de facturas|Colaborador de la sección de factura|Lector de la sección de facturas|Creador de la suscripción a Azure|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver todos los productos que ha comprado en la sección de factura|✔|✔|✔|✘|✔|✔|✔|
|Administrar la facturación para los productos para la sección de factura, como cancelar, desactivar la renovación automática y mucho más|✔|✔|✘|✘|✘|✘|✘|
|Cambiar la sección de factura para los productos|✔|✔|✘|✘|✘|✘|✘|

### <a name="manage-subscriptions-for-invoice-section"></a>Administrar las suscripciones para la sección de factura

|Tareas|Propietario de la sección de facturas|Colaborador de la sección de factura|Lector de la sección de facturas|Creador de la suscripción a Azure|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Visualización de todas las suscripciones a Azure para la sección de factura|✔|✔|✔|✘|✔|✔|✔|
|Cambiar la sección de factura para las suscripciones|✔|✔|✘|✘|✘|✘|✘|
|Solicitud de la propiedad de la facturación de las suscripciones de usuarios de otras cuentas de facturación|✔|✔|✘|✘|✘|✘|✘|

## <a name="subscription-billing-roles-and-tasks"></a>Tareas y roles de la facturación de suscripción

En la tabla siguiente se muestra qué rol necesita para completar las tareas en el contexto de una suscripción.

|Tareas|Propietario de la sección de facturas|Colaborador de la sección de factura|Lector de la sección de facturas|Creador de la suscripción a Azure|
|---|---|---|---|---|
|Creación de suscripciones a Azure|✔|✔|✘|✔|
|Actualizar el centro de coste para la suscripción|✔|✔|✘|✘|
|Cambiar la sección de factura para la suscripción|✔|✔|✘|✘|

## <a name="manage-billing-roles-in-the-azure-portal"></a>Administración de roles integrados en Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Administración de costos + facturación**.

   ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/understand-mca-roles/billing-search-cost-management-billing.png)

3. Seleccione **Control de acceso (IAM)** en un ámbito como cuenta de facturación, perfil de facturación o sección de factura, donde desee dar acceso.

4. La página de Control de acceso (IAM) muestra los usuarios y grupos asignados a cada rol para ese ámbito.

   ![Captura de pantalla que muestra una lista de administradores para la cuenta de facturación](./media/understand-mca-roles/billing-list-admins.png)

5. Para dar acceso a un usuario, seleccione **Agregar** en la parte superior de la página. En la lista desplegable Rol, seleccione un rol. Escriba la dirección de correo electrónico del usuario al que desea dar acceso. Seleccione **Guardar** para asignar el rol.

   ![Captura de pantalla que muestra cómo agregar un administrador a una cuenta de facturación](./media/understand-mca-roles/billing-add-admin.png)

6. Para quitar el acceso a un usuario, seleccione dicho usuario con la asignación de roles que desea quitar. Seleccione Quitar.

   ![Captura de pantalla que se muestra cómo eliminar un administrador de una cuenta de facturación](./media/understand-mca-roles/billing-remove-admin.png)

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Comprobación del acceso a un contrato de cliente de Microsoft
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Ponerse en contacto con soporte técnico
Si necesita ayuda, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para obtener información sobre la cuenta de facturación:

- [Empezar a trabajar con la cuenta de facturación para un contrato de cliente de Microsoft](../understand/mca-overview.md)
- [Creación de una suscripción a Azure para la cuenta de facturación del contrato de cliente de Microsoft](create-subscription.md)
