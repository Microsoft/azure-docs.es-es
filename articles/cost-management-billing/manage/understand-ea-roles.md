---
title: Información sobre los roles de administrador para Enterprise en Azure
description: Conozca los roles de administrador de empresa en Azure. Puede asignar cinco roles administrativos distintos.
author: bandersmsft
ms.reviewer: adwise
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 787614afa1c71eee075a2e3efa81fa0cba17d1ba
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88686469"
---
# <a name="managing-azure-enterprise-roles"></a>Administración de roles de Azure Enterprise

Para ayudar a administrar el uso y el gasto de su organización, los clientes de Azure con un Contrato Enterprise (EA) pueden asignar cinco roles de administrador diferentes:

- Administrador de empresa
- Administrador de empresa (solo lectura)<sup>1</sup>
- Administrador de departamentos
- Administrador de departamento (solo lectura)
- Propietario de la cuenta <sup>2</sup>

<sup>1</sup> El contacto de facturación del Contrato Enterprise estará bajo este rol.

<sup>2</sup> No se puede agregar ni cambiar el contacto de destinatario de la factura en el portal de EA de Azure y se agregará a la inscripción de EA según el usuario que esté configurado como tal en el nivel de contrato. Para cambiar el contacto de destinatario de la factura, es necesario realizar una solicitud a través de un asesor de software o asociado al centro de operaciones regional (ROC).

El primer administrador de inscripciones que se configure durante el aprovisionamiento de la inscripción determina el tipo de autenticación de la cuenta de contacto de facturación. Cuando el contacto de facturación se agrega a Enterprise Portal como administrador de solo lectura, se le proporciona la autenticación de la cuenta Microsoft. 

Por ejemplo, si el tipo de autenticación inicial se establece en Mixto, EA se agregará como un cuenta Microsoft y el contacto de facturación tendrá privilegios de administrador de EA de solo lectura. Si el administrador de EA no aprueba la autorización de un cuenta Microsoft para un contacto de facturación existente, puede eliminar el usuario en cuestión y pedir al cliente que vuelva a agregarlo como administrador de solo lectura con una cuenta profesional o educativa establecida únicamente en el nivel de inscripción en el portal de EA.

Estos roles son específicos de la administración de Contratos Enterprise de Azure y son adicionales a los roles integrados que tiene Azure para controlar el acceso a los recursos. Para más información, consulte [Roles integrados en Azure](../../role-based-access-control/built-in-roles.md).

En las secciones siguientes se describen las limitaciones y funcionalidades de cada rol.

## <a name="user-limit-for-admin-roles"></a>Límite de usuarios para roles de administrador

|Role| Límite de usuarios|
|---|---|
|Administrador de empresa|Sin límite|
|Administrador de empresa (solo lectura)|Sin límite|
|Administrador de departamentos|Sin límite|
|Administrador de departamento (solo lectura)|Sin límite|
|Propietario de cuenta|1 por cuenta <sup>3</sup>|

<sup>3</sup> Cada cuenta requiere una única cuenta de Microsoft o una cuenta profesional o educativa.

## <a name="organization-structure-and-permissions-by-role"></a>Estructura de organización y permisos por rol

|Tareas| Administrador de empresa|Administrador de empresa (solo lectura)|Administrador de departamentos|Administrador de departamento (solo lectura)|Propietario de cuenta| Asociado|
|---|---|---|---|---|---|---|
|Ver administradores de empresa|✔|✔|✘|✘|✘|✔|
|Agregar o quitar administradores de empresa|✔|✘|✘|✘|✘|✘|
|Ver contactos de notificación<sup>4</sup> |✔|✔|✘|✘|✘|✔|
|Agregar o quitar contactos de notificación<sup>4</sup> |✔|✘|✘|✘|✘|✘|
|Crear y administrar departamentos |✔|✘|✘|✘|✘|✘|
|Ver administradores de departamento|✔|✔|✔|✔|✘|✔|
|Agregar o quitar administradores de departamento|✔|✘|✔|✘|✘|✘|
|Ver cuentas en la inscripción |✔|✔|✔<sup>5</sup>|✔<sup>5</sup>|✘|✔|
|Agregar cuentas a la inscripción y cambiar el propietario de la cuenta|✔|✘|✔<sup>5</sup>|✘|✘|✘|
|Crear y administrar suscripciones y permisos de suscripción|✘|✘|✘|✘|✔|✘|

- <sup>4</sup> A los contactos de notificación se les envían comunicaciones por correo electrónico sobre el Contrato Enterprise de Azure.
- <sup>5</sup> La tarea se limita a las cuentas de su departamento.

## <a name="add-a-new-enterprise-administrator"></a>Adición de un nuevo administrador de empresa

Los administradores de empresa tienen los máximos privilegios al administrar una inscripción del Contrato Enterprise de Azure. Se creó el administrador del Contrato Enterprise de Azure inicial cuando se configuró el acuerdo del Contrato Enterprise. Sin embargo, puede agregar o quitar nuevos administradores en cualquier momento. Solo los administradores existentes agregan nuevos administradores. Para más información sobre cómo agregar administradores de empresa adicionales, consulte [Creación de otro administrador de empresa](ea-portal-get-started.md#create-another-enterprise-administrator). Para más información acerca de los roles del perfil de facturación, consulte [Tareas y roles del perfil de facturación](understand-mca-roles.md#billing-profile-roles-and-tasks).

## <a name="update-account-owner-state-from-pending-to-active"></a>Actualización del estado del propietario de la cuenta, de pendiente a activo

Cuando se agregan nuevos propietarios de la cuenta a una inscripción del Contrato Enterprise de Azure por primera vez, su estado aparece como _pendiente_. Cuando un nuevo propietario de la cuenta recibe el correo electrónico de bienvenida de activación, puede iniciar sesión para activar su cuenta. Al activar su cuenta, el estado de la cuenta se actualiza de _pendiente_ a _activa_. El propietario de la cuenta tiene que leer el mensaje de advertencia y seleccionar **Continuar**. Es posible que se pida a los nuevos usuarios que escriban su nombre y apellidos para crear una cuenta comercial. En ese caso, deben agregar la información necesaria para continuar y, acto seguido, se activará la cuenta.

## <a name="add-a-department-admin"></a>Adición de un administrador de departamento

Cuando un administrador de Contrato Enterprise de Azure crea un departamento, el administrador puede agregar administradores de departamento y asociar cada uno de ellos a un departamento. Un administrador de departamento puede crear nuevas cuentas. Se necesitan nuevas cuentas para que se creen las suscripciones del Contrato Enterprise de Azure.

Para más información sobre cómo agregar un administrador de departamento, consulte [Creación de un administrador de departamento del Contrato Enterprise de Azure](ea-portal-get-started.md#add-a-department-administrator).

## <a name="usage-and-costs-access-by-role"></a>Acceso de uso y costos por rol

|Tareas| Administrador de empresa|Administrador de empresa (solo lectura)|Administrador de departamentos|Administrador de departamento (solo lectura) |Propietario de cuenta| Asociado|
|---|---|---|---|---|---|---|
|Ver saldo de crédito, incluido el prepago de Azure|✔|✔|✘|✘|✘|✔|
|Ver cuotas de gastos de departamento|✔|✔|✘|✘|✘|✔|
|Establecer cuotas de gasto de departamento|✔|✘|✘|✘|✘|✘|
|Ver la hoja de precios de EA de la organización|✔|✔|✘|✘|✘|✔|
|Ver los detalles de uso y costo|✔|✔|✔<sup>6</sup>|✔<sup>6</sup>|✔<sup>7</sup>|✔|
|Administrar recursos en Azure Portal|✘|✘|✘|✘|✔|✘|

- <sup>6</sup> Requiere que el administrador de empresa habilite la directiva de **visualización de cargos del administrador de departamento** en Enterprise Portal. El administrador de departamento puede entonces ver los detalles de costo del departamento.
- <sup>7</sup> Requiere que el administrador de empresa habilite la directiva de **visualización de cargos del propietario de la cuenta** en Enterprise Portal. El propietario de la cuenta puede ver entonces los detalles del costo de la cuenta.

## <a name="see-pricing-for-different-user-roles"></a>Consulte los precios de los diferentes roles de usuario

Es posible que vea un precio diferente en Azure Portal según su rol administrativo y cómo el administrador de empresa haya establecido las directivas de visualización de los cargos. Las dos directivas de Enterprise Portal que afectan a los precios que ve en Azure Portal son:

- Visualización de cargos del AD
- Visualización de cargos del PC

Para saber cómo establecer estas directivas, consulte [Administración del acceso a la información de facturación en Azure](manage-billing-access.md).

En la tabla siguiente se muestra la relación entre los roles de administrador del Contrato Enterprise, la directiva de visualización de cargos, el rol de Azure en Azure Portal y los precios que ve en Azure Portal. El administrador de empresa siempre ve los detalles de uso según los precios de EA de la organización. Sin embargo, el administrador de departamento y el propietario de la cuenta tienen vistas de precios diferentes según la directiva de visualización de cargos y su rol de Azure. El rol de administrador de departamento que se muestra en la tabla siguiente hace referencia a los roles de administrador de departamento y administrador de departamento (solo lectura).

|Rol de administrador de Contrato Enterprise|Directiva de visualización de cargos por rol|Rol de Azure|Vista de precios|
|---|---|---|---|
|Propietario de la cuenta o administrador de departamento|✔ Habilitado|Propietario|Precios de EA de la organización|
|Propietario de la cuenta o administrador de departamento|✘ Deshabilitado|Propietario|Precio de venta|
|Propietario de la cuenta o administrador de departamento|✔ Habilitado |None|No hay precios|
|Propietario de la cuenta o administrador de departamento|✘ Deshabilitado |None|No hay precios|
|None|No aplicable |Propietario|Precio de venta|

El rol de administrador de Enterprise y las directivas de visualización de cargos se establecen en Enterprise Portal. El rol de Azure puede actualizarse en Azure Portal. Para saber más, vea [Administración del acceso mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md).



## <a name="next-steps"></a>Pasos siguientes

- [Administración del acceso a la información de facturación de Azure](manage-billing-access.md)
- [Administración del acceso mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md)
- [Roles integrados en los recursos de Azure](../../role-based-access-control/built-in-roles.md)
