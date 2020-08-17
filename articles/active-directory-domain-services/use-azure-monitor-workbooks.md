---
title: Uso de libros de Azure Monitor con Azure AD Domain Services | Microsoft Docs
description: Aprenda a usar libros de Azure Monitor para revisar las auditorías de seguridad y comprender los problemas de un dominio administrado de Azure Active Directory Domain Services.
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: iainfou
ms.openlocfilehash: 7d1cd032f6042f57a096c5c8e7624f66a4b01355
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87419604"
---
# <a name="review-security-audit-events-in-azure-active-directory-domain-services-using-azure-monitor-workbooks"></a>Revisión de los eventos de auditoría de seguridad en Azure Active Directory Domain Services mediante libros de Azure Monitor

Para ayudarle a entender el estado del dominio administrado de Azure Active Directory Domain Services (Azure AD DS), puede habilitar eventos de auditoría de seguridad. Estos eventos de auditoría de seguridad se pueden revisar posteriormente mediante libros de Azure Monitor que combinan texto, consultas de análisis y parámetros en informes interactivos enriquecidos. Azure AD DS incluye plantillas de libro para la información general de seguridad y la actividad de la cuenta, que le permiten profundizar en los eventos de auditoría y administrar el entorno.

En este artículo se muestra cómo usar los libros de Azure Monitor para revisar los eventos de auditoría de seguridad en Azure AD DS.

## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, complete el tutorial para [crear y configurar un dominio administrado de Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Eventos de auditoría de seguridad habilitados para el dominio administrado que transmiten los datos a un área de trabajo de Log Analytics.
    * Si es necesario, [habilite las auditorías de seguridad para Azure AD DS][enable-security-audits].

## <a name="azure-monitor-workbooks-overview"></a>Introducción a los libros de Azure Monitor

Cuando los eventos de auditoría de seguridad se activan en Azure AD DS, puede ser difícil analizar e identificar problemas en el dominio administrado. Azure Monitor permite agregar estos eventos de auditoría de seguridad y consultar los datos. Con los libros de Azure Monitor, puede visualizar estos datos para que sea más rápido y fácil identificar los problemas.

Las plantillas de libro son informes seleccionados diseñados para que varios usuarios y equipos puedan reutilizarlos de forma flexible. Al abrir una plantilla de libro, se cargan los datos del entorno de Azure Monitor. Puede usar las plantillas sin que afecte a otros usuarios de la organización y puede guardar libros propios basados en la plantilla.

Azure AD DS incluye las dos plantillas de libro siguientes:

* Informe general sobre seguridad
* Informe de actividades de la cuenta

Para obtener más información sobre cómo editar y administrar libros, vea [Información general sobre los libros de Azure Monitor](../azure-monitor/platform/workbooks-overview.md).

## <a name="use-the-security-overview-report-workbook"></a>Uso del libro Informe general sobre seguridad

Para ayudarle a comprender mejor el uso e identificar posibles amenazas de seguridad, en el informe general sobre seguridad se resumen los datos de inicio de sesión y se identifican las cuentas que es posible que quiera comprobar. Puede ver los eventos de un intervalo de fechas determinado y explorar en profundidad eventos de inicio de sesión específicos, como los intentos incorrectos de contraseña o la deshabilitación de la cuenta.

Para acceder a la plantilla de libro del informe general sobre seguridad, complete los pasos siguientes:

1. En Azure Portal, busque y seleccione **Azure Active Directory Domain Services**.
1. Seleccione el dominio administrado como, por ejemplo, *aaddscontoso.com*.
1. En el menú de la izquierda, elija **Supervisión > Libros**.

    ![Selección de la opción de menú Libros en Azure Portal](./media/use-azure-monitor-workbooks/select-workbooks-in-azure-portal.png)

1. Seleccione **Informe general sobre seguridad**.
1. En los menús desplegables de la parte superior del libro, seleccione la suscripción a Azure y, después, un área de trabajo de Azure Monitor.

    Elija un **intervalo de tiempo**, como *Últimos 7 días*, tal como se muestra en la siguiente captura de pantalla de ejemplo:

    ![Selección de la opción de menú Libros en Azure Portal](./media/use-azure-monitor-workbooks/select-query-filters.png)

    También se pueden cambiar las opciones **Vista de mosaico** y **Vista de gráfico** para analizar y visualizar los datos según sea necesario.

1. Para explorar en profundidad un tipo de evento específico, seleccione una de las tarjetas **Resultado del inicio de sesión** como *Cuenta bloqueada*, como se muestra en el ejemplo siguiente:

    ![Ejemplo de datos del informe Información general sobre seguridad visualizados en libros de Azure Monitor](./media/use-azure-monitor-workbooks/example-security-overview-report.png)

1. En la parte inferior del informe, debajo del gráfico, se desglosa el tipo de actividad seleccionado. En la parte derecha puede filtrar por los nombres de usuario implicados, como se muestra en el siguiente informe de ejemplo:

    [![Detalles de los bloqueos de cuenta en Workbooks de Azure Monitor](./media/use-azure-monitor-workbooks/account-lockout-details-cropped.png)](./media/use-azure-monitor-workbooks/account-lockout-details.png#lightbox)

## <a name="use-the-account-activity-report-workbook"></a>Uso del libro de Informe de actividades de la cuenta

Para ayudar a solucionar problemas de una cuenta de usuario específica, en el informe de actividades de la cuenta se desglosa información detallada del registro de eventos de auditoría. Puede revisar cuándo se ha proporcionado un nombre de usuario o una contraseña incorrectos durante el inicio de sesión y el origen del intento de inicio de sesión.

Para acceder a la plantilla de libro del informe de actividades de la cuenta, complete los pasos siguientes:

1. En Azure Portal, busque y seleccione **Azure Active Directory Domain Services**.
1. Seleccione el dominio administrado como, por ejemplo, *aaddscontoso.com*.
1. En el menú de la izquierda, elija **Supervisión > Libros**.
1. Seleccione el **Informe de actividades de la cuenta**.
1. En los menús desplegables de la parte superior del libro, seleccione la suscripción a Azure y, después, un área de trabajo de Azure Monitor.

    Elija un **Intervalo de tiempo** , como *Últimos 30 días* y, después, cómo quiere que se representen los datos en la **Vista de icono**.

    Puede filtrar por **Nombre de usuario de la cuenta**, por ejemplo *felix*, como se muestra en el siguiente informe de ejemplo:

    [![Informe de actividades de la cuenta en Workbooks de Azure Monitor](./media/use-azure-monitor-workbooks/account-activity-report-cropped.png)](./media/use-azure-monitor-workbooks/account-activity-report.png#lightbox)

    En el área situada debajo del gráfico se muestran los eventos de inicio de sesión individuales junto con información como el resultado de la actividad y la estación de trabajo de origen. Esta información puede ayudar a determinar los orígenes repetidos de eventos de inicio de sesión que pueden provocar bloqueos de la cuenta o indicar un posible ataque.

Como sucede con el informe general sobre seguridad, puede explorar en profundidad los distintos iconos de la parte superior del informe para visualizar y analizar los datos según sea necesario.

## <a name="save-and-edit-workbooks"></a>Guardado y modificación de los libros

Los dos libros de plantilla proporcionados por Azure AD DS son un buen lugar para comenzar un análisis de datos propio. Si necesita obtener más granularidad en las consultas de datos y las investigaciones, puede guardar libros propios y editar las consultas.

1. Para guardar una copia de una de las plantillas de libro, seleccione **Editar > Guardar como > Informes compartidos** y, después, proporcione un nombre y guárdelo.
1. En la copia de la plantilla propia, seleccione **Editar** para entrar en el modo de edición. Puede elegir el botón **Editar** de color azul situado junto a cualquier elemento del informe y cambiarlo.

Todos los gráficos y las tablas de los libros de Azure Monitor se generan mediante consultas de Kusto. Para obtener más información sobre cómo crear consultas propias, vea [Consultas de registros de Azure Monitor][azure-monitor-queries] y [Tutorial de consultas de Kusto][kusto-queries].

## <a name="next-steps"></a>Pasos siguientes

Si tiene que ajustar las directivas de contraseñas y bloqueo, vea [Directivas de bloqueo de cuentas y contraseñas en dominios administrados][password-policy].

Para los problemas con los usuarios, aprenda a solucionar [problemas de inicio de sesión de cuenta][troubleshoot-sign-in] o [problemas de bloqueo de cuenta][troubleshoot-account-lockout].

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[enable-security-audits]: security-audit-events.md
[password-policy]: password-policy.md
[troubleshoot-sign-in]: troubleshoot-sign-in.md
[troubleshoot-account-lockout]: troubleshoot-account-lockout.md
[azure-monitor-queries]: ../azure-monitor/log-query/query-language.md
[kusto-queries]: https://docs.microsoft.com/azure/kusto/query/tutorial?pivots=azuredataexplorer
