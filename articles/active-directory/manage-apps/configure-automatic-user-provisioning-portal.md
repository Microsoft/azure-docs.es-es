---
title: Administración de aprovisionamiento de usuarios para aplicaciones empresariales en Azure Active Directory | Microsoft Docs
description: Aprenda a administrar el aprovisionamiento de cuentas de usuario para aplicaciones empresariales con Azure Active Directory.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/01/2019
ms.author: mimart
ms.reviewer: asmalser
ms.collection: M365-identity-device-management
ms.openlocfilehash: a7319a108cb52c603396f8c654697e16438fa7d6
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780980"
---
# <a name="managing-user-account-provisioning-for-enterprise-apps-in-the-azure-portal"></a>Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales en el portal de Azure

En este artículo se describe cómo usar [Azure Portal](https://portal.azure.com) para administrar el aprovisionamiento y el desaprovisionamiento automáticos de cuentas de usuario en aplicaciones que lo admitan. Para más información sobre el aprovisionamiento automático de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](user-provisioning.md).

## <a name="finding-your-apps-in-the-portal"></a>Búsqueda de aplicaciones en el portal

Usar el portal de Azure Active Directory para ver y administrar todas las aplicaciones que están configuradas para inicio de sesión único en un directorio. Las aplicaciones empresariales son aplicaciones que se implementan y se usan dentro de su organización. Siga estos pasos para ver y administrar las aplicaciones empresariales:

1. Abra el [portal de Azure Active Directory](https://aad.portal.azure.com).

1. Seleccione **aplicaciones empresariales** en el panel izquierdo. Se muestra una lista de todas las aplicaciones configuradas, incluidas las aplicaciones que se han agregado desde la galería.

1. Seleccione cualquier aplicación para cargar su panel de recursos, donde puede ver los informes y administrar la configuración de la aplicación.

1. Seleccione **aprovisionamiento** para administrar la configuración de la aplicación seleccionada de aprovisionamiento de cuentas de usuario.

   ![Panel Recurso de aplicación](./media/configure-automatic-user-provisioning-portal/enterprise-apps-provisioning.png)

## <a name="provisioning-modes"></a>Modos de aprovisionamiento

El **aprovisionamiento** panel comienza con un **modo** menú, que muestra los modos de aprovisionamiento admitidos para una aplicación empresarial y permite configurarlos. Las opciones disponibles incluyen:

* **Automática** -esta opción estará disponible si Azure AD admite el aprovisionamiento automático basado en API o cancelación del aprovisionamiento de cuentas de usuario para esta aplicación. Seleccione este modo para mostrar una interfaz que permite a los administradores:

  * Configurar Azure AD para conectarse a la API de administración de usuario de la aplicación
  * Crear asignaciones de cuentas y los flujos de trabajo que definen cómo deben fluir los datos de la cuenta de usuario entre Azure AD y la aplicación
  * Administrar el servicio de aprovisionamiento de Azure AD

* **Manual** -esta opción estará disponible si Azure AD no admite el aprovisionamiento automático de cuentas de usuario para esta aplicación. En este caso, cuenta de usuario de los registros almacenan en la aplicación debe administrarse mediante un proceso externo, según las capacidades de administración y aprovisionamiento de usuario proporcionadas por la aplicación (que puede incluir el aprovisionamiento Just In Time de SAML).

## <a name="configuring-automatic-user-account-provisioning"></a>Configuración del aprovisionamiento automático de cuentas de usuario

Seleccione el **automática** opción para especificar la configuración de credenciales de administrador, asignaciones, iniciar y detener y la sincronización.

### <a name="admin-credentials"></a>Credenciales de administrador

Expanda **las credenciales de administrador** que escriba las credenciales necesarias para que Azure AD para conectarse a la API de administración de usuario de la aplicación. La entrada necesaria varía dependiendo de la aplicación. Para más información sobre los requisitos y tipos de credenciales para aplicaciones específicas, consulte el [tutorial de configuración de la aplicación específica](user-provisioning.md).

Seleccione **Probar conexión** para probar las credenciales al hacer que Azure AD intente conectarse a la aplicación de aprovisionamiento de la aplicación con las credenciales proporcionadas.

### <a name="mappings"></a>Asignaciones

Expanda **asignaciones** para ver y editar los atributos de usuario que fluyen entre Azure AD y la aplicación de destino cuando se aprovisiona o se actualizan las cuentas de usuario.

Hay un conjunto preconfigurado de asignaciones entre los objetos de usuario de Azure AD y los objetos de usuario de cada aplicación SaaS. Algunas aplicaciones administran otros tipos de objetos, como grupos o contactos. Seleccione una asignación de la tabla para abrir el editor de asignación a la derecha, donde puede ver y personalizarlos.

![Panel Recurso de aplicación](./media/configure-automatic-user-provisioning-portal/enterprise-apps-provisioning-mapping.png)

Las personalizaciones compatibles incluyen:

* Habilitar y deshabilitar asignaciones para objetos específicos, como el objeto de usuario de Azure AD en el objeto de usuario de la aplicación SaaS.
* Editar los atributos que fluyen desde el objeto de usuario de Azure AD al objeto de usuario de la aplicación. Para más información sobre la asignación de atributos, consulte la sección [Información sobre los tipos de asignaciones de atributos](customize-application-attributes.md#understanding-attribute-mapping-types).
* Filtrado de las acciones de aprovisionamiento que Azure AD se ejecuta en la aplicación de destino. En lugar de tener Azure AD sincronice totalmente los objetos, puede limitar las acciones que se ejecute. 

  Por ejemplo, solo seleccione **actualización** y Azure AD únicamente actualiza las usuario existente en una aplicación de cuentas, pero no crea nuevos. Sólo seleccione **crear** y Azure únicamente crea nuevas cuentas de usuario, pero no actualiza los existentes. Esta característica permite a los administradores crear asignaciones diferentes para la creación de cuenta y actualizar los flujos de trabajo.

* Agregar una nueva asignación de atributos. Seleccione **agregar nueva asignación** en la parte inferior de la **asignación de atributos** panel. Rellene el **Editar atributo** formulario y seleccione **Aceptar** para agregar la nueva asignación a la lista. 

### <a name="settings"></a>Configuración

Puede iniciar y detener el servicio de la aplicación seleccionada en el aprovisionamiento de Azure AD la **configuración** área de la **aprovisionamiento** pantalla. También puede borrar la caché de aprovisionamiento y reiniciar el servicio.

Si el aprovisionamiento se habilita por primera vez para una aplicación, active el servicio mediante el cambio de **Estado de aprovisionamiento** a **Activado**. Este cambio hace que el servicio de aprovisionamiento de AD Azure ejecutar una sincronización inicial. Lee los usuarios asignados en el **usuarios y grupos** sección, consulta la aplicación de destino para ellos y, a continuación, ejecuta las acciones de aprovisionamiento definidas en Azure AD **asignaciones** sección. Durante este proceso, el servicio de aprovisionamiento almacena datos en caché sobre las cuentas de usuario que está administrando, por lo que las cuentas no administradas dentro de las aplicaciones de destino que nunca estaban en el ámbito de asignación no se ven afectadas por las operaciones de desaprovisionamiento. Después de la sincronización inicial, el servicio de aprovisionamiento sincroniza automáticamente los objetos de grupo y usuario en un intervalo de diez minutos.

Cambiar el **estado de aprovisionamiento** a **desactivar** para pausar el servicio de aprovisionamiento. En este estado, Azure no crear, actualizar o quitar cualquier objeto de grupo o usuario en la aplicación. Cambie el estado de nuevo al **en** y el servicio retoma donde lo dejó.

Seleccione el **Borrar estado actual y reiniciar sincronización** casilla y seleccione **guardar** para:

* Detener el servicio de aprovisionamiento
* Volcar los datos almacenados en caché sobre las cuentas que es la administración de Azure AD
* Reinicie los servicios y ejecute de nuevo la sincronización inicial

Esta opción permite a los administradores iniciar el proceso de implementación de aprovisionamiento de nuevo.

### <a name="synchronization-details"></a>Detalles de sincronización

Esta sección proporciona detalles adicionales sobre la operación del servicio de aprovisionamiento, incluidos las primeras y últimas veces que el servicio de aprovisionamiento ejecutado en la aplicación y cuántos objetos de usuario y grupo que administra.

Se proporciona un vínculo a la **informe de actividad de aprovisionamiento**, que proporciona un registro de todos los usuarios y grupos creado, actualizado y quitado entre Azure AD y la aplicación de destino. También se proporciona un vínculo a la **informe de errores de aprovisionamiento**, que proporciona más detallada de los mensajes de error para los objetos de usuario y grupo que no se pudieron leer, crear, actualizar o quitar.
