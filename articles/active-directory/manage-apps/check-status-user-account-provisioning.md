---
title: Creación de informes sobre el aprovisionamiento automático de cuentas de usuario de Azure Active Directory para aplicaciones SaaS | Microsoft Docs
description: Aprenda a comprobar el estado de los trabajos de aprovisionamiento automático de cuentas de usuario y a solucionar problemas del aprovisionamiento de usuarios individuales.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: app-mgmt
ms.devlang: na
ms.topic: conceptual
ms.date: 09/09/2018
ms.author: mimart
ms.reviewer: asmalser
ms.collection: M365-identity-device-management
ms.openlocfilehash: 71816c8c7e4213ddea4f08e046dc9224eb52b5fd
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780995"
---
# <a name="tutorial-reporting-on-automatic-user-account-provisioning"></a>Tutorial: Creación de informes sobre el aprovisionamiento automático de cuentas de usuario


Azure Active Directory incluye un [servicio de aprovisionamiento de cuentas de usuario](user-provisioning.md) que ayuda a automatizar el aprovisionamiento y el desaprovisionamiento de cuentas de usuario en aplicaciones SaaS y otros sistemas, para la administración del ciclo de vida de las identidades completa. Azure AD admite conectores de aprovisionamiento de usuarios integrados previamente para todas las aplicaciones y sistemas de la sección "Destacadas" de la [Galería de aplicaciones de Azure AD](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps?page=1&subcategories=featured).

En este artículo se describe cómo comprobar el estado de aprovisionamiento de trabajos una vez configurados y cómo solucionar el aprovisionamiento de usuarios individuales y grupos.

## <a name="overview"></a>Información general

Los conectores de aprovisionamiento se configuran mediante [Azure Portal](https://portal.azure.com), siguiendo la [documentación proporcionada](../saas-apps/tutorial-list.md) para la aplicación admitida. Una vez configurados y en ejecución, es posible notificar trabajos de aprovisionamiento mediante uno de estos dos métodos:

* **Portal de administración de Azure**: en este artículo se describe principalmente la recuperación de información de informes en [Azure Portal](https://portal.azure.com), que proporciona tanto un informe de resumen de aprovisionamiento como registros detallados de auditoría de aprovisionamiento para una aplicación determinada.

* **API de auditoría**: Azure Active Directory también proporciona una API de auditoría que permite la recuperación mediante programación de registros detallados de auditoría de aprovisionamiento. Consulte [Referencia de la API de auditoría de Azure Active Directory](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/directoryaudit) para ver documentación específica sobre el uso de esta API. Aunque este artículo no trata específicamente del uso de la API, detalla los tipos de eventos de aprovisionamiento que se registran en el registro de auditoría.

### <a name="definitions"></a>Definiciones

En este artículo se utilizan los términos que se definen a continuación:

* **Sistema de origen**: el repositorio de usuarios desde el que se sincroniza el servicio de aprovisionamiento de Azure AD. Azure Active Directory es el sistema de origen para la mayoría de los conectores de aprovisionamiento integrados previamente, sin embargo, hay algunas excepciones (ejemplo: Sincronización de entrada de Workday).

* **Sistema de destino**: el repositorio de usuarios al que se sincroniza el servicio de aprovisionamiento de Azure AD. Esto suele ser una aplicación SaaS (ejemplos: Salesforce, ServiceNow, G Suite, Dropbox para empresas), pero en algunos casos puede ser un sistema local como Active Directory (ejemplo: Sincronización de entrada de Workday a Active Directory).


## <a name="getting-provisioning-reports-from-the-azure-management-portal"></a>Obtención de informes de aprovisionamiento desde el Portal de administración de Azure

Para obtener información de informes para una aplicación de aprovisionamiento determinada, inicie el [Portal de administración de Azure](https://portal.azure.com) y vaya a la aplicación empresarial para la que se ha configurado el aprovisionamiento. Por ejemplo, si va a aprovisionar usuarios para LinkedIn Elevate, la ruta de navegación a los detalles de la aplicación es la siguiente:

**Azure Active Directory &gt; Aplicaciones empresariales &gt; Todas las aplicaciones &gt; LinkedIn Elevate**

Desde aquí, puede acceder al informe de resumen de aprovisionamiento y a los registros de auditoría de aprovisionamiento, que se describen a continuación.


## <a name="provisioning-summary-report"></a>Informe de resumen de aprovisionamiento

El informe de resumen de aprovisionamiento está visible en la pestaña **Aprovisionamiento** de la aplicación específica. Se encuentra en la sección **Detalles de sincronización** debajo de **Configuración** y proporciona la siguiente información:

* El número total de usuarios y grupos que se han sincronizado y están actualmente en el ámbito para el aprovisionamiento entre el sistema de origen y el sistema de destino.

* La última vez que se ejecutó la sincronización. Las sincronizaciones suelen producirse cada 20-40 minutos, una vez que haya finalizado una [sincronización inicial](user-provisioning.md#what-happens-during-provisioning).

* Tanto si ha finalizado una [sincronización inicial](user-provisioning.md#what-happens-during-provisioning) como si no.

* Tanto si el proceso de aprovisionamiento se ha puesto en cuarentena como si no, y sea cual sea la razón para el estado de cuarentena (por ejemplo, error al comunicarse con el sistema de destino debido a que las credenciales del administrador no son válidas).

El informe de resumen de aprovisionamiento debe ser el primer lugar en el que busquen los administradores para comprobar el estado operativo del trabajo de aprovisionamiento.

 ![Informe de resumen](./media/check-status-user-account-provisioning/summary_report.PNG)

## <a name="provisioning-audit-logs"></a>Registros de auditoría de aprovisionamiento
Todas las actividades realizadas por el servicio de aprovisionamiento se registran en los registros de auditoría de Azure AD, que pueden verse en la pestaña **Registros de auditoría** en la categoría **Aprovisionamiento de cuentas**. Los tipos de eventos de actividades registradas incluyen:

* **Eventos de importación**: cada vez que el servicio de aprovisionamiento de Azure AD recupera información sobre un usuario individual o un grupo desde un sistema de origen o destino, se registra un evento de "importación". Durante la sincronización, los usuarios se recuperan en primer lugar del sistema de origen, con resultados que se registran como eventos de "importación". Los identificadores de coincidencias de los usuarios recuperados se consultan luego en el sistema de destino para comprobar si existen, con resultados que se registran también como eventos de "importación". Estos eventos registran todos los atributos de usuario asignados y sus valores, vistos por el servicio de aprovisionamiento de Azure AD en el momento del evento. 

* **Eventos de reglas de sincronización**: estos eventos notifican los resultados de las reglas de asignación de atributos y cualquier filtro de ámbito configurado, una vez importados y evaluados los datos de usuario a partir de los sistemas de origen y de destino. Por ejemplo, si se considera que un usuario en un sistema de origen está en el ámbito para el aprovisionamiento y que no existe en el sistema de destino, este evento registra que el usuario se aprovisionará en el sistema de destino. 

* **Eventos de exportación**: cada vez que el servicio de aprovisionamiento de Azure AD escribe una cuenta de usuario o un objeto de grupo para un sistema de destino, se registra un evento de "exportación". Estos eventos registran todos los atributos de usuario y sus valores, escritos por el servicio de aprovisionamiento de Azure AD en el momento del evento. Si se ha producido un error al escribir la cuenta de usuario o el objeto de grupo en el sistema de destino, se mostrará aquí.

* **Eventos de custodia de procesos**: las custodias de procesos se producen cuando el servicio de aprovisionamiento se encuentra con un error al intentar una operación y comienza a reintentar la operación en un intervalo de tiempo de espera. Cada vez que se reintenta una operación de aprovisionamiento, se registra un evento de "custodia".

Al examinar los eventos de aprovisionamiento para un usuario individual, los eventos se suelen producir en este orden:

1. Evento de importación: el usuario se recupera del sistema de origen.

2. Evento de importación: se consulta el sistema de destino para comprobar la existencia del usuario recuperado.

3. Evento de reglas de sincronización: los datos de usuario de los sistemas de origen y de destino se evalúan con las reglas de asignación del atributo configurado y los filtros de ámbito para determinar qué acción, si la hay, debe realizarse.

4. Evento de exportación: si el evento de reglas de sincronización ha dictado que se debe realizar una acción (agregar, actualizar o eliminar), los resultados de la acción se registran en un evento de exportación.

![Creación de un usuario de prueba de Azure AD](./media/check-status-user-account-provisioning/audit_logs.PNG)


### <a name="looking-up-provisioning-events-for-a-specific-user"></a>Búsqueda de eventos de aprovisionamiento para un usuario específico

El caso de uso más común para los registros de auditoría de aprovisionamiento es comprobar el estado de aprovisionamiento de una cuenta de usuario individual. Para buscar eventos de aprovisionamiento para un usuario específico:

1. Vaya a la sección **Registros de auditoría**.

2. En el menú **Categoría**, seleccione **Aprovisionamiento de cuentas**.

3. En el menú **Intervalo de fechas**, seleccione el intervalo de fechas que va a buscar.

4. En la barra **Buscar** barra, escriba el identificador del usuario que va a buscar. El formato del valor del identificador debe coincidir con el que ha seleccionado como identificador de coincidencia principal en la configuración de asignación de atributos (por ejemplo, userPrincipalName o número de Id. de empleado). El valor del identificador requerido estará visible en la columna Destinos.

5. Presione Entrar para buscar. Se devuelven en primer lugar los eventos de aprovisionamiento más recientes.

6. Si se devuelven eventos, tenga en cuenta los tipos de actividad y si se han realizado correctamente o no. Si no se devuelve ningún resultado, significa que el usuario no existe o que no ha sido detectado por el proceso de aprovisionamiento si no se ha realizado aún una sincronización completa.

7. Haga clic en los eventos individuales para ver detalles ampliados, incluidas todas las propiedades de usuario que se han recuperado, evaluado o escrito como parte del evento.

Para ver una demostración sobre cómo usar los registros de auditoría, vea el vídeo siguiente. Los registros de auditoría se muestran alrededor del minuto 5:30:

> [!VIDEO https://www.youtube.com/embed/pKzyts6kfrw]

### <a name="tips-for-viewing-the-provisioning-audit-logs"></a>Sugerencias para ver los registros de auditoría de aprovisionamiento

Para optimizar la legibilidad en Azure Portal, seleccione el botón **Columnas** y elija estas columnas:

* **Fecha**: muestra la fecha en la que se ha producido el evento.
* **Destinos**: muestra el identificador de usuario y el nombre de aplicación que son los temas del evento.
* **Actividad**: el tipo de actividad, tal como se ha descrito anteriormente.
* **Estado**: indica si el evento se ha realizado correctamente o no.
* **Motivo del estado**: resumen de lo que ha sucedido en el evento de aprovisionamiento.


## <a name="troubleshooting"></a>solución de problemas

El informe de resumen de aprovisionamiento y los registros de auditoría desempeñan un papel clave a la hora de ayudar a los administradores a solucionar diversos problemas de aprovisionamiento de cuentas de usuario.

Para ver instrucciones basadas en escenarios sobre cómo solucionar problemas de aprovisionamiento automático de usuarios, consulte [Problemas al configurar y aprovisionar usuarios en una aplicación](application-provisioning-config-problem.md).


## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](what-is-single-sign-on.md)
