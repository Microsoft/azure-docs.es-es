---
title: Qué es el aprovisionamiento automatizado de usuarios de aplicaciones SaaS en Azure AD
description: Una introducción sobre cómo puede usar Azure AD para el aprovisionamiento, el desaprovisionamiento y la actualización continua de cuentas de usuario de manera automática en varias aplicaciones SaaS de terceros.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: overview
ms.workload: identity
ms.date: 11/25/2019
ms.author: kenwith
ms.reviewer: arvinh, celested
ms.openlocfilehash: b6eb571c7c37a628d11f07b4e1b207e38830250b
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88235407"
---
# <a name="what-is-automated-saas-app-user-provisioning-in-azure-ad"></a>Qué es el aprovisionamiento automatizado de usuarios de aplicaciones SaaS en Azure AD

En Azure Active Directory (Azure AD), el término **aprovisionamiento de aplicaciones** hace referencia a la creación automática de identidades y roles de usuario en las aplicaciones de nube ([SaaS](https://azure.microsoft.com/overview/what-is-saas/)) a las que los usuarios necesitan acceso. Además de crear identidades de usuario, el aprovisionamiento automático incluye el mantenimiento y la eliminación de identidades de usuario a medida que el estado o los roles cambian. Algunos escenarios comunes son el aprovisionamiento de un usuario de Azure AD en aplicaciones como [Dropbox](../saas-apps/dropboxforbusiness-provisioning-tutorial.md), [Salesforce](../saas-apps/salesforce-provisioning-tutorial.md) y [ServiceNow](../saas-apps/servicenow-provisioning-tutorial.md), entre otras.

![Diagrama de información general del aprovisionamiento](./media/user-provisioning/provisioning-overview.png)

Esta característica le permite hacer lo siguiente:

- **Aprovisionamiento automatizado**: Crear cuentas automáticamente en los sistemas adecuados para los usuarios nuevos cuando se unen a su equipo u organización.
- **Desaprovisionamiento automatizado**: Desactivar las cuentas automáticamente en los sistemas adecuados cuando los usuarios dejan el equipo o la organización.
- **Sincronización de datos entre sistemas:** asegúrese de que las identidades de las aplicaciones y sistemas se mantienen actualizadas con los cambios del directorio o el sistema de recursos humanos.
- **Aprovisionamiento de grupos:** aprovisione grupos para las aplicaciones que los admiten.
- **Control del acceso:** supervise y audite a quién se ha aprovisionado en las aplicaciones.
- **Implementación sin problemas en escenarios que parten de recursos existentes:** haga coincidir las identidades actuales entre los sistemas y permita una integración sencilla, incluso cuando los usuarios ya existen en el sistema de destino.
- **Uso de personalización de voz:** aproveche las asignaciones de atributos personalizables que definen qué datos del usuario deben fluir del sistema de origen al sistema de destino.
- **Obtención de alertas para eventos críticos:** el servicio de aprovisionamiento proporciona alertas para los eventos críticos y permite la integración de Log Analytics, donde puede definir alertas personalizadas para establecer las necesidades de su empresa.

## <a name="benefits-of-automatic-provisioning"></a>Ventajas del aprovisionamiento automático

A medida que el número de aplicaciones usadas en las organizaciones modernas sigue creciendo, los administradores de TI tienen tareas de administración de acceso a escala. Los estándares como el Lenguaje de marcado de aserción de seguridad (SAML) u Open ID Connect (OIDC) permiten a los administradores configurar rápidamente el inicio de sesión único (SSO), pero el acceso también requiere que los usuarios se aprovisionen en la aplicación. Para muchos administradores, el aprovisionamiento significa crear manualmente cada cuenta de usuario o cargar los archivos CSV cada semana, pero estos procesos son lentos, caros y propensos a errores. Se han adoptado soluciones como Just-in-Time (JIT) de SAML para automatizar el aprovisionamiento, pero las empresas también necesitan una solución que permita desaprovisionar a los usuarios cuando abandonan la organización o ya no necesitan acceso a determinadas aplicaciones según el cambio de roles.

Algunos de los motivos comunes para usar el aprovisionamiento automático incluyen:

- Maximizar la eficacia y la precisión de los procesos de aprovisionamiento.
- El ahorro en los costos asociados al hospedaje y el mantenimiento de scripts y soluciones de aprovisionamiento desarrollados de forma personalizada.
- Proteger su organización mediante la eliminación instantánea de las identidades de los usuarios de aplicaciones SaaS claves cuando estos abandonan la organización.
- Importar fácilmente una gran cantidad de usuarios a una aplicación o sistema SaaS concretos.
- Tener un único conjunto de directivas para determinar quién se aprovisiona y quién puede iniciar sesión en una aplicación.

El aprovisionamiento de usuarios de Azure AD puede ayudar a resolver estos desafíos. Para más información sobre cómo usan los clientes el aprovisionamiento de usuarios de Azure AD, puede leer el [caso práctico de ASOS](https://aka.ms/asoscasestudy). El vídeo siguiente proporciona información general sobre el aprovisionamiento de usuarios en Azure AD:

> [!VIDEO https://www.youtube.com/embed/_ZjARPpI6NI]

## <a name="what-applications-and-systems-can-i-use-with-azure-ad-automatic-user-provisioning"></a>¿Qué aplicaciones y sistemas puedo usar con el aprovisionamiento automático de usuarios de Azure AD?

Azure AD ofrece compatibilidad preintegrada con muchas aplicaciones SaaS y sistemas de recursos humanos populares, así como compatibilidad genérica con aplicaciones que implementan determinadas partes del [estándar SCIM 2.0](https://techcommunity.microsoft.com/t5/Identity-Standards-Blog/Provisioning-with-SCIM-getting-started/ba-p/880010).

* **Aplicaciones preintegradas (aplicaciones SaaS de la galería)** . Para ver una lista de todas las aplicaciones en las que Azure AD admite un conector de aprovisionamiento integrado previamente, consulte la [lista de tutoriales de aplicaciones de aprovisionamiento de usuarios](../saas-apps/tutorial-list.md). Las aplicaciones integradas previamente que se enumeran en la galería suelen usar las API de administración de usuarios basadas en SCIM 2.0 para el aprovisionamiento. 

   ![Logotipo de Salesforce](./media/user-provisioning/gallery-app-logos.png)

   Si quiere solicitar una nueva aplicación para el aprovisionamiento, puede [solicitar que la aplicación se integre con la galería de aplicaciones](../azuread-dev/howto-app-gallery-listing.md). Si se da una solicitud de aprovisionamiento de usuarios, es necesario que la aplicación tenga un punto de conexión compatible con SCIM. Solicite al proveedor de la aplicación que siga el estándar SCIM para que podamos incorporar la aplicación a nuestra plataforma rápidamente.

* **Aplicaciones que admiten SCIM 2.0**. Para obtener información sobre cómo conectar genéricamente aplicaciones que implementan API de administración de usuarios basadas en SCIM 2.0, consulte [Aprovisionamiento de usuarios de SCIM con Azure Active Directory (Azure AD)](use-scim-to-provision-users-and-groups.md).

## <a name="what-is-system-for-cross-domain-identity-management-scim"></a>¿Qué es el sistema de administración de identidades entre dominios (SCIM)?

Para ayudar a automatizar el aprovisionamiento y el desaprovisionamiento, las aplicaciones exponen las API de grupo y de usuario propietario. Sin embargo, cualquier persona que intente administrar usuarios en más de una aplicación le indicará que cada una intenta realizar las mismas acciones sencillas, como crear o actualizar usuarios, agregar usuarios a grupos o cancelar el aprovisionamiento de usuarios. Aún así, todas estas acciones sencillas se implementan de forma que difieren ligeramente, con diferentes rutas de acceso de puntos de conexión, distintos métodos para especificar la información del usuario y un esquema distinto para representar cada elemento de información.

Para abordar estos desafíos, la especificación de SCIM proporciona un esquema de usuario común a fin de ayudar a los usuarios a entrar a las aplicaciones, salir de ellas y trabajar con ellas. SCIM se está convirtiendo en el estándar de facto para aprovisionar y, cuando se usa junto con estándares de federación como SAML u OpenID Connect, ofrece a los administradores una solución de un extremo a otro basada en estándares para la administración del acceso.

Para ver instrucciones detalladas acerca de cómo desarrollar un punto de conexión de SCIM para automatizar el aprovisionamiento y el desaprovisionamiento de usuarios y grupos en una aplicación, consulte [Creación de un punto de conexión de SCIM y configuración del aprovisionamiento de usuarios](use-scim-to-provision-users-and-groups.md). En el caso de las aplicaciones integradas previamente en la galería (Slack, Azure Databricks, Snowflake, etc.), puede obviar la documentación para desarrolladores y usar los tutoriales que se proporcionan [aquí](../saas-apps/tutorial-list.md).

## <a name="manual-vs-automatic-provisioning"></a>Aprovisionamiento manual y automático

Las aplicaciones de la galería de Azure AD admiten uno de los dos modos de aprovisionamiento:

* El aprovisionamiento **manual** significa que todavía no hay ningún conector de aprovisionamiento de Azure AD automático para la aplicación. Las cuentas de usuario deben crearse manualmente, por ejemplo, agregando usuarios directamente en el portal administrativo de la aplicación o cargando una hoja de cálculo con los detalles de estas cuentas. Consulte la documentación suministrada por la aplicación o póngase en contacto con su desarrollador para determinar qué mecanismos hay disponibles.

* **Automático** significa que se ha desarrollado un conector de aprovisionamiento de Azure AD para esta aplicación. Debe seguir el tutorial de configuración específico para configurar el aprovisionamiento de la aplicación. Puede encontrar tutoriales sobre aplicaciones en [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](../saas-apps/tutorial-list.md).

En la galería de Azure AD, las aplicaciones que admiten el aprovisionamiento automático se designan mediante un icono de **aprovisionamiento**. Cambie a la nueva experiencia en versión preliminar de la galería para ver estos iconos (en el banner situado en la parte superior de la **página para agregar una aplicación**, seleccione el vínculo que indica **Haga clic aquí para probar la nueva y mejorada experiencia de la galería de aplicaciones**).

![Icono de aprovisionamiento en la galería de aplicaciones](./media/user-provisioning/browse-gallery.png)

El modo de aprovisionamiento compatible con una aplicación también se puede ver en la pestaña **Aprovisionamiento** una vez que se ha agregado la aplicación a **Aplicaciones empresariales**.

## <a name="how-do-i-set-up-automatic-provisioning-to-an-application"></a>¿Cómo configuro el aprovisionamiento automático para una aplicación?

En el caso de las aplicaciones preintegradas que se enumeran en la galería, se ofrece una guía paso a paso para configurar el aprovisionamiento automático. Consulte la [lista de tutoriales para las aplicaciones integradas de la galería](../saas-apps/tutorial-list.md). En el vídeo siguiente se muestra cómo configurar el aprovisionamiento automático de usuarios para SalesForce.

> [!VIDEO https://www.youtube.com/embed/pKzyts6kfrw]

Para otras aplicaciones que admiten SCIM 2.0, siga los pasos descritos en el artículo [Aprovisionamiento de usuarios de SCIM con Azure Active Directory (Azure AD)](use-scim-to-provision-users-and-groups.md).


## <a name="next-steps"></a>Pasos siguientes

- [Lista de tutoriales sobre cómo integrar aplicaciones SaaS](../saas-apps/tutorial-list.md)
- [Personalización de asignaciones de atributos para el aprovisionamiento de usuarios](customize-application-attributes.md)
- [Filtros de ámbito para el aprovisionamiento de usuarios](define-conditional-rules-for-provisioning-user-accounts.md)
