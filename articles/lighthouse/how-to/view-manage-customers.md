---
title: Ver y administrar clientes y recursos delegados
description: Como proveedor de servicio o empresa que usa Azure Lighthouse, puede ver todos los recursos y suscripciones delegados desde la sección Mis clientes de Azure Portal.
ms.date: 08/12/2020
ms.topic: how-to
ms.openlocfilehash: 165110556282e90bbacf4ce3b3c59e34b203a528
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88167205"
---
# <a name="view-and-manage-customers-and-delegated-resources"></a>Ver y administrar clientes y recursos delegados

Los proveedores de servicio que usan [Azure Lighthouse](../overview.md) pueden usar la página **Mis clientes** de [Azure Portal](https://portal.azure.com) para ver las suscripciones y los recursos de clientes delegados. 

> [!TIP]
> Aunque aquí nos referiremos a los proveedores de servicios y clientes, las [empresas que administren varios inquilinos](../concepts/enterprise.md) pueden usar el mismo proceso para consolidar su experiencia de administración.

Para acceder a la página **Mis clientes** en Azure Portal, seleccione **Todos los servicios**, busque **Mis clientes** y seleccione esta opción. También puede acceder aquí escribiendo "Mis clientes" en el cuadro de búsqueda situado cerca de la parte superior de Azure Portal.

Tenga en cuenta que la sección superior **Clientes** de la página **Mis clientes** solo muestra información sobre los clientes que tienen suscripciones o grupos de recursos delegados. Si trabaja con otros clientes (por ejemplo, mediante el [programa Proveedor de soluciones en la nube](/partner-center/csp-overview)), no verá información sobre sus clientes en la sección **Clientes** a menos que haya [incorporado sus recursos a Azure Lighthouse](onboard-customer.md).

Más abajo en la página, una sección independiente llamada **Proveedor de soluciones en la nube (versión preliminar)** muestra la información de facturación y los recursos para los clientes de CSP que han [firmado el Contrato de cliente de Microsoft (MCA)](/partner-center/confirm-customer-agreement) y usan [el plan de Azure](/partner-center/azure-plan-get-started). Para obtener más información, consulte [Introducción a la cuenta de facturación para un contrato Microsoft Partner Agreement](../../cost-management-billing/understand/mpa-overview.md). Tenga en cuenta que esos clientes de CSP aparecen en esta sección independientemente de si también los ha incorporado a Azure Lighthouse. Del mismo modo, no es necesario que un cliente de CSP aparezca en la sección **Proveedor de soluciones en la nube (versión preliminar)** de **Mis clientes** para poder incorporarlo a Azure Lighthouse.

> [!NOTE]
> Los clientes pueden ver información sobre los proveedores de servicios navegando a **Proveedores de servicios** en Azure Portal. Para obtener más información, consulte [View and manage service providers](view-manage-service-providers.md) (Ver y administrar los proveedores de servicios).

## <a name="view-and-manage-customer-details"></a>Ver y administrar los detalles del usuario

Para ver los detalles del cliente, seleccione **Clientes** en el lado izquierdo de la página **Mis clientes**.

Para cada cliente, verá el nombre del cliente, el identificador de cliente (id. de inquilino) y la oferta asociada con la involucración. En la columna **Delegaciones**, verá el número de suscripciones delegadas o el número de grupos de recursos delegados.

> [!IMPORTANT]
> Para ver una delegación, a los usuarios se les debe haber concedido el rol [lector](../../role-based-access-control/built-in-roles.md#reader) (u otro rol integrado que incluya acceso de lectura) en el proceso de incorporación.

Los filtros de la parte superior de la página le permiten ordenar y agrupar la información del cliente o filtrar por clientes, ofertas o palabras clave específicos.

Puede ver la siguiente información desde esta página:

- Para ver todas las suscripciones, ofertas y delegaciones asociadas con un cliente, seleccione el nombre del cliente.
- Para ver más detalles sobre una oferta y sus delegaciones, seleccione el nombre de la oferta.
- Para ver más detalles acerca de las asignaciones de roles en las suscripciones o grupos de recursos delegados, seleccione la entrada de la columna **Delegaciones**.

## <a name="view-and-manage-delegations"></a>Visualización y administración de delegaciones

Las delegaciones muestran el grupo de recursos o la suscripción que se ha delegado, junto con los usuarios y permisos que tienen acceso a ella. Para ver esta información, seleccione **Delegaciones** en el lado izquierdo de la página **Mis clientes**.

Los filtros de la parte superior de la página le permiten ordenar y agrupar la información de asignación de acceso o filtrar por clientes, ofertas o palabras clave específicos.

### <a name="view-role-assignments"></a>Visualización de asignaciones de roles

Los usuarios y los permisos asociados con cada delegación aparecen en la columna **Asignaciones de roles**. Puede seleccionar cada entrada para ver la lista completa de usuarios, grupos y entidades de servicio a los que se ha concedido acceso a la suscripción o al grupo de recursos. Desde allí, puede seleccionar un usuario, grupo o nombre de entidad de seguridad de servicio determinado para obtener más detalles.

### <a name="remove-delegations"></a>Eliminación de delegaciones

Si ha incluido usuarios con el [rol para eliminar la asignación de registros de servicios administrados](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) al incorporar el cliente a Azure Lighthouse, esos usuarios pueden eliminar una delegación mediante la selección del icono de la papelera que aparece en la fila de dicha delegación. Al hacerlo, ningún usuario en el inquilino del proveedor de servicios podrá tener acceso a los recursos que se habían delegado previamente.

## <a name="work-in-the-context-of-a-delegated-subscription"></a>Trabajar en el contexto de una suscripción delegada

Puede trabajar directamente en el contexto de una suscripción delegada en Azure Portal sin cambiar el directorio en el que está trabajando. Para ello:

1. Seleccione el icono de **directorio y suscripción** situado cerca de la parte superior de Azure Portal.
2. En el filtro **Suscripción global**, asegúrese de que solo está seleccionado el cuadro de la suscripción delegada. Puede usar el cuadro **directorios actuales y delegados** para mostrar solo las suscripciones de un directorio específico. (No use la **Cambiar directorio**, ya que cambia el directorio en el que ha iniciado sesión).

Si después accede a un servicio que admite [experiencias de administración entre inquilinos](../concepts/cross-tenant-management-experience.md), el servicio se establecerá de forma predeterminada en el contexto de la suscripción delegada que haya seleccionado. Para cambiarlo, siga los pasos anteriores y active la casilla **Seleccionar todo** (o elija una o más suscripciones para trabajar en su lugar).

> [!NOTE]
> Si se le ha concedido acceso a uno o varios grupos de recursos, en lugar de acceso a una suscripción completa, puede seleccionar la suscripción a la que pertenece ese grupo de recursos. Después, trabajará en el contexto de esa suscripción, pero solo podrá tener acceso a los grupos de recursos designados.

También puede tener acceso a la funcionalidad relacionada con las suscripciones o grupos de recursos delegados desde los servicios que admiten experiencias de administración entre inquilinos seleccionando la suscripción o el grupo de recursos desde el servicio.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [experiencias de administración entre inquilinos](../concepts/cross-tenant-management-experience.md).
- Aprenda cómo los clientes pueden [ver y administrar proveedores de servicios](view-manage-service-providers.md) desde **Proveedores de servicios**, en Azure Portal.
