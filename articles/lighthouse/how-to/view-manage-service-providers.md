---
title: Visualización y administración de proveedores de servicios
description: Los clientes pueden usar la página proveedores de servicios en Azure Portal para ver información acerca de los proveedores de servicios, ofertas de proveedores de servicios y recursos delegados.
ms.date: 08/12/2020
ms.topic: how-to
ms.openlocfilehash: c22408a52d973a244d67528a73d4eaa487f166ba
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88167171"
---
# <a name="view-and-manage-service-providers"></a>Visualización y administración de proveedores de servicios

Los clientes pueden usar la página de **Proveedores de servicios** de [Azure Portal](https://portal.azure.com) para ver detalles sobre proveedores de servicios y ofertas de proveedores de servicios, delegar recursos específicos a [Azure Lighthouse](../overview.md) y comprar nuevas ofertas de proveedores de servicios.

> [!TIP]
> Aunque aquí nos referiremos a los proveedores de servicios y clientes, las [empresas que administren varios inquilinos](../concepts/enterprise.md) pueden usar el mismo proceso para consolidar su experiencia de administración.

Para acceder a la página **Proveedores de servicios** de Azure Portal, el cliente puede seleccionar **Todos los servicios** y, a continuación, buscar **Proveedores de servicios** y seleccionar esta opción. También pueden encontrarla si escribe "Proveedores de servicios" o "Azure Lighthouse" en el cuadro de búsqueda situado cerca de la parte superior de Azure Portal.

> [!NOTE]
> Para ver la página **Proveedores de servicios**, un usuario del inquilino del cliente debe tener el [rol integrado Lector](../../role-based-access-control/built-in-roles.md#reader) (u otro que incluya el acceso de lectura).
>
> Para agregar o actualizar ofertas, delegar recursos y quitar ofertas, el usuario debe tener el [rol integrado Propietario](../../role-based-access-control/built-in-roles.md#owner) para la suscripción.

Tenga en cuenta que la página de **Proveedores de servicios** solo muestra información sobre los proveedores de servicios que tienen acceso a las suscripciones o grupos de recursos del cliente mediante Azure Lighthouse. Si un cliente trabaja con proveedores de servicios adicionales que no usan Azure Lighthouse para acceder a los recursos del cliente, la información sobre esos proveedores de servicios no se muestra aquí.

> [!TIP]
> Para ver información sobre sus clientes, los proveedores de servicios pueden desplazarse hasta la sección **Mis clientes** de Azure Portal. Para obtener más información, consulte [View and manage customers and delegated resources](view-manage-customers.md) (Ver y administrar clientes y recursos delegados).

## <a name="view-service-provider-details"></a>Ver detalles del proveedor de servicios

Para ver detalles sobre los proveedores de servicios, el cliente puede seleccionar **Ofertas del proveedor de servicios** en el lado izquierdo de la página **Proveedores de servicios**.

Para cada oferta de proveedor de servicios, el cliente verá el nombre del proveedor de servicios y la oferta asociada a él, junto con el nombre que el cliente especificó durante el proceso de incorporación.

En la columna **Delegaciones**, el cliente ve el número de suscripciones o grupos de recursos que se han delegado en el proveedor de servicios de esa oferta. El proveedor de servicios podrá tener acceso a estas suscripciones o grupos de recursos y administrarlos de acuerdo con los niveles de acceso especificados en la oferta.

## <a name="add-or-remove-service-provider-offers"></a>Agregar o quitar ofertas del proveedor de servicios

Un cliente puede agregar una nueva oferta del proveedor de servicios desde la página **Ofertas del proveedor de servicios**. Para ello, debe seleccionar **Agregar oferta**. El proveedor de servicios debe haber publicado una oferta para este cliente. Después, el cliente puede seleccionar esa oferta en la pantalla **Ofertas privadas** y, a continuación, seleccionar **Crear**.

Si el cliente quiere quitar una oferta del proveedor de servicios, puede seleccionar el icono de la papelera en la fila de esa oferta. Después de confirmar la eliminación, ese proveedor de servicios ya no tendrá acceso a los recursos del cliente que anteriormente se delegaron para esa oferta.

## <a name="delegate-resources"></a>Recursos delegados

Para que un proveedor de servicios pueda acceder a los recursos de un cliente y administrarlos, se deben delegar. Si un cliente ha aceptado una oferta, pero aún no ha delegado los recursos, verá una nota en la parte superior de la sección **Ofertas del proveedor de servicios**. Esto permite al cliente saber que deben tomar medidas para que el proveedor de servicios pueda acceder a los recursos del cliente.

Para delegar suscripciones o grupos de recursos:

1. Active la casilla de la fila que contiene el proveedor de servicios, la oferta y el nombre. A continuación, seleccione **Delegar recursos** en la parte superior de la pantalla.
1. En la sección **Detalles de la oferta** de la página **Delegar recursos**, revise los detalles sobre el proveedor de servicios y la oferta. Para revisar las asignaciones de roles de la oferta, seleccione **Hacer clic aquí para ver los detalles de la oferta seleccionada**.
1. En la sección **Delegar**, seleccione **Delegar suscripciones** o **Delegar grupos de recursos**.
1. Elija las suscripciones o los grupos de recursos que quiera delegar para esta oferta y, después, seleccione **Agregar**.
1. Active la casilla situada en la parte inferior de la página para confirmar que quiere conceder a este proveedor de servicios acceso a los recursos que ha seleccionado y, después, seleccione **Delegar**.

## <a name="update-service-provider-offers"></a>Actualización de las ofertas de proveedores de servicios

Una vez que un cliente ha agregado una oferta, un proveedor de servicios puede publicar una versión actualizada de la misma oferta en Azure Marketplace. Por ejemplo, puede querer agregar una nueva definición de roles. Si se ha publicado una versión nueva de la oferta, la página **Ofertas del proveedor de servicios** mostrará un icono de actualización en la fila de esa oferta. El cliente puede seleccionar este icono para ver las diferencias entre la versión actual de la oferta y la nueva.

 ![Icono de actualización de la oferta](../media/update-offer.jpg)

Después de revisar los cambios, el cliente puede optar por actualizar a la nueva versión. Una vez que lo haga, las autorizaciones y otras opciones especificadas en la nueva versión se aplicarán a todas las suscripciones o grupos de recursos que se hayan delegado para esa oferta.

## <a name="view-delegations"></a>Ver delegaciones

Las delegaciones representan las asignaciones de roles que conceden permisos al proveedor del servicio para los recursos que un cliente ha delegado. Para ver esta información, seleccione **Delegaciones** en el lado izquierdo de la página **Proveedores de servicios**.

Los filtros de la parte superior de la página permiten ordenar y agrupar la información de delegación. También puede filtrar por clientes, ofertas o palabras clave específicos.

> [!NOTE]
> Los clientes no verán estas asignaciones de roles o los usuarios del inquilino de proveedor de servicios a los que se hayan concedido estos roles, cuando [ver la asignación de roles para el ámbito delegado en Azure Portal](../../role-based-access-control/role-assignments-list-portal.md#list-role-assignments-at-a-scope) o a través de las API.

## <a name="audit-delegations-in-your-environment"></a>Auditoría de delegaciones en el entorno

Es posible que los clientes quieran obtener visibilidad sobre las suscripciones o los grupos de recursos que se han delegado Azure Lighthouse. Esto es especialmente útil para los clientes con un gran número de suscripciones o para los que tienen muchos usuarios que realizan tareas de administración.

Se proporciona una [definición de directiva integrada de Azure Policy](../../governance/policy/samples/built-in-policies.md#lighthouse) para auditar la delegación de ámbitos en un inquilino de administración. Esta directiva se puede asignar a un grupo de administración que incluya todas las suscripciones que quiera auditar. Al comprobar el cumplimiento de esta directiva, las suscripciones o los grupos de recursos delegados (dentro del grupo de administración al que se asigna la directiva) se mostrarán en un estado no conforme. Luego, puede revisar los resultados y confirmar que no hay ninguna delegación inesperada.

Para más información sobre cómo asignar una directiva y ver los resultados del estado de cumplimiento, consulte [Inicio rápido: Creación de una asignación de directiva](../../governance/policy/assign-policy-portal.md).

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de [Azure Lighthouse](../overview.md).
- Descubra cómo los proveedores de servicios pueden [ver y administrar sus clientes](view-manage-customers.md) en la página **Mis clientes** de Azure Portal.
