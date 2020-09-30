---
title: Administración de áreas de trabajo de Azure Sentinel a gran escala
description: Obtenga información sobre cómo a administrar de forma eficaz Azure Sentinel en recursos delegados de clientes.
ms.date: 08/27/2020
ms.topic: how-to
ms.openlocfilehash: 78896d6458f6b1c40655bc36d2bf63e05ddb0a59
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90019740"
---
# <a name="manage-azure-sentinel-workspaces-at-scale"></a>Administración de áreas de trabajo de Azure Sentinel a gran escala

Como proveedor de servicios, es posible que haya incorporado varios inquilinos de cliente para [Azure Lighthouse](../overview.md). Azure Lighthouse permite a los proveedores de servicios realizar operaciones a escala a través de varios inquilinos de Azure Active Directory (Azure AD) a la vez, lo que hace que las tareas de administración sean más eficaces.

Azure Sentinel ofrece análisis de seguridad e inteligencia frente a amenazas, de forma que proporciona una única solución para la detección de alertas, la visibilidad de amenazas, la búsqueda proactiva y la respuesta a amenazas. Con Azure Lighthouse, puede administrar varias áreas de trabajo de Azure Sentinel en varios inquilinos a gran escala. Esto permite escenarios como la ejecución de consultas en varias áreas de trabajo o la creación de libros para visualizar y supervisar los datos de los orígenes de datos conectados para obtener información. La dirección IP, como las consultas y los cuadernos de estrategias, permanecen en el inquilino de administración, pero se puede usar para realizar la administración de seguridad en los inquilinos del cliente.

Este tema proporciona información general sobre cómo usar [Azure Sentinel](../../sentinel/overview.md) de forma escalable para la visibilidad entre inquilinos y los servicios de seguridad administrada.

> [!TIP]
> Aunque en este tema hacemos referencia a proveedores de servicios y clientes, esta guía es aplicable también a [empresas que usan Azure Lighthouse para administrar varios inquilinos](../concepts/enterprise.md).

## <a name="architectural-considerations"></a>Consideraciones arquitectónicas

Para un proveedor de servicios de seguridad administrada (MSSP) que quiera construir una oferta de seguridad como servicio con Azure Sentinel, es posible que necesite un solo centro de operaciones de seguridad (SOC) para supervisar, administrar y configurar de forma centralizada en varias áreas de trabajo de Azure Sentinel implementadas en inquilinos de clientes individuales. Del mismo modo, las empresas con múltiples inquilinos de Azure AD pueden querer administrar de forma centralizada múltiples áreas de trabajo de Azure Sentinel implementadas entre sus inquilinos.

Este modelo centralizado de implementación presenta las siguientes ventajas:

- La propiedad de los datos sigue siendo de cada inquilino administrado.
- Admite requisitos para almacenar datos dentro de los límites geográficos.
- Garantiza el aislamiento de datos, ya que los datos de varios clientes no se almacenan en la misma área de trabajo.
- Evita la filtración de datos de los inquilinos administrados, lo que ayuda a garantizar el cumplimiento de los datos.
- Los costos relacionados se cargan a cada inquilino administrado, en lugar de al inquilino administrador.
- Los datos de todos los orígenes de datos y conectores de datos que se integran con Azure Sentinel (como los registros de actividad de Azure AD, los registros de Office 365 o las alertas de protección contra amenazas de Microsoft) permanecerán dentro de cada inquilino de cliente.
- Reduce la latencia de red.
- Facilidad para agregar o quitar nuevas subsidiarias o clientes.

## <a name="granular-role-based-access-control-rbac"></a>Control de acceso basado en rol (RBAC) pormenorizado.

Cada suscripción de cliente que administrará un MSSP debe [incorporarse a Azure Lighthouse](onboard-customer.md). Esto permite a los usuarios designados del inquilino de administración tener acceso y realizar operaciones de administración en áreas de trabajo de Azure Sentinel implementadas en inquilinos del cliente.

Al crear las autorizaciones, puede asignar los roles integrados de Azure Sentinel a usuarios, grupos o entidades de servicio en el inquilino de administración:

- [Lector de Azure Sentinel](../../role-based-access-control/built-in-roles.md#azure-sentinel-reader)
- [Respondedor de Azure Sentinel](../../role-based-access-control/built-in-roles.md#azure-sentinel-responder)
- [Colaborador de Azure Sentinel](../../role-based-access-control/built-in-roles.md#azure-sentinel-contributor)

También puede asignar roles integrados adicionales para realizar funciones adicionales. Para obtener información sobre los roles específicos que se pueden usar con Azure Sentinel, consulte [Permisos de Azure Sentinel](../../sentinel/roles.md).

Una vez que haya incorporado a sus clientes, los usuarios designados pueden iniciar sesión en el inquilino de administración y [tener acceso directamente al área de trabajo de Azure Sentinel del cliente](../../sentinel/multiple-tenants-service-providers.md) con los roles asignados.

## <a name="view-and-manage-incidents-across-workspaces"></a>Ver y administrar incidentes en áreas de trabajo

Si va a administrar los recursos de Azure Sentinel para varios clientes, puede ver y administrar incidentes en varias áreas de trabajo a la vez en varios inquilinos. Para más información, consulte [Procesamiento de incidentes en varias áreas de trabajo a la vez](../../sentinel/multiple-workspace-view.md) y [Extender Azure Sentinel por área de trabajo e inquilinos](../../sentinel/extend-sentinel-across-workspaces-tenants.md).

> [!NOTE]
> Asegúrese de que se asignaron permisos de lectura y escritura a los usuarios del inquilino de administración en todas las áreas de trabajo que se administran. Si un usuario solo tiene permisos de lectura en algunas áreas de trabajo, pueden aparecer mensajes de advertencia al seleccionar incidentes en esas áreas de trabajo, y el usuario no podrá modificar esos incidentes ni ningún otro que haya seleccionado con ellos (aunque tenga permisos para los demás).

## <a name="configure-playbooks-for-mitigation"></a>Configurar los cuadernos de estrategias para la mitigación

[Los cuadernos de estrategias](../../sentinel/tutorial-respond-threats-playbook.md) pueden usarse para la mitigación automática cuando se desencadena una alerta. Estos cuadernos de estrategias se pueden ejecutar de forma manual o automática cuando se desencadenan alertas específicas. Los cuadernos de estrategias se pueden implementar en el inquilino de administración o en el inquilino del cliente, con los procedimientos de respuesta configurados en función de los usuarios del inquilino que deberán tomar medidas en respuesta a una amenaza de seguridad.

## <a name="create-cross-tenant-workbooks"></a>Crear libros entre inquilinos

Los [libros de Azure Monitor en Azure Sentinel](../../sentinel/overview.md#workbooks) le ayudan a visualizar y supervisar los datos de los orígenes de datos conectados para obtener información. Puede usar las plantillas de libro integradas en Azure Sentinel o crear libros personalizados para sus escenarios.

Puede implementar libros en el inquilino de administración y crear paneles a escala para supervisar y consultar datos en los inquilinos de los clientes. Para obtener más información consulte [Supervisión entre áreas de trabajo](../../sentinel/extend-sentinel-across-workspaces-tenants.md#using-cross-workspace-workbooks). 

También puede implementar libros directamente en un inquilino individual que administra para escenarios específicos de ese cliente.

## <a name="run-log-analytics-and-hunting-queries-across-azure-sentinel-workspaces"></a>Ejecución de Log Analytics y consultas de búsqueda en áreas de trabajo de Azure Sentinel

Cree y guarde consultas de Log Analytics para la detección de amenazas de forma centralizada en el inquilino de administración, incluidas las [consultas de búsqueda](../../sentinel/extend-sentinel-across-workspaces-tenants.md#cross-workspace-hunting). Estas consultas se pueden realizar en todas las áreas de trabajo de Azure Sentinel de sus clientes mediante el operador de unión y la expresión de área de trabajo (). Para obtener más información consulte [Consultas entre áreas de trabajo](../../sentinel/extend-sentinel-across-workspaces-tenants.md#cross-workspace-querying).

## <a name="use-automation-for-cross-workspace-management"></a>Usar la automatización para la administración entre áreas de trabajo

Puede usar automatización para administrar varias áreas de trabajo de Azure Sentinel y configurar [consultas de búsqueda](../../sentinel/hunting.md), guías y libros. Para más información consulte [Administración entre áreas de trabajo con automatización](../../sentinel/extend-sentinel-across-workspaces-tenants.md#cross-workspace-management-using-automation).

## <a name="manage-security-of-office-365-environments"></a>Administración de la seguridad de los entornos de Office 365

Use Azure Lighthouse junto con Azure Sentinel para administrar la seguridad de los entornos de Office 365 entre inquilinos. En primer lugar, los [conectores de datos de Office 365 integrados deben estar habilitados en el inquilino administrado](../../sentinel/connect-office-365.md) para que la información sobre las actividades de usuarios y administradores en Exchange y SharePoint (incluido OneDrive) se pueda ingerir en un área de trabajo de Azure Sentinel en el inquilino administrado. Esto incluye detalles sobre acciones como descargas de archivos, solicitudes de acceso enviadas, cambios en eventos de grupo y operaciones de buzón de correo, junto con información sobre los usuarios que realizaron dichas acciones. Las [alertas de DLP de Office 365](https://techcommunity.microsoft.com/t5/azure-sentinel/ingest-office-365-dlp-events-into-azure-sentinel/ba-p/1031820) también se admiten como parte del conector integrado de Office 365.

Puede habilitar el [conector de Microsoft Cloud App Security (MCAS)](../../sentinel/connect-cloud-app-security.md) para transmitir alertas y registros de Cloud Discovery a Azure Sentinel. Esto le permite adquirir visibilidad sobre las aplicaciones en la nube, obtener análisis sofisticados para identificar y combatir las ciberamenazas y controlar cómo viajan los datos. Los registros de actividad de MCAS se pueden [consumir mediante el formato de evento común (CEF)](https://techcommunity.microsoft.com/t5/azure-sentinel/ingest-box-com-activity-events-via-microsoft-cloud-app-security/ba-p/1072849).

Después de configurar los conectores de datos de Office 365, puede usar las funcionalidades de Azure Sentinel entre inquilinos, como la visualización y análisis los datos de los libros, el uso de consultas para crear alertas personalizadas y la configuración de los cuadernos de estrategias para responder a las amenazas.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [Azure Sentinel](../../sentinel/overview.md).
- Revise la [página de precios de Azure Sentinel](https://azure.microsoft.com/pricing/details/azure-sentinel/).
- Más información sobre las [experiencias de administración entre inquilinos](../concepts/cross-tenant-management-experience.md).

