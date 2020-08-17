---
title: Realización de una revisión de acceso de grupos y aplicaciones - Azure AD
description: Obtenga información sobre cómo completar una revisión de acceso de los miembros del grupo o de la aplicación en las revisiones de acceso de Azure Active Directory.
services: active-directory
documentationcenter: ''
author: ajburnle
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: 23266043d5efd65a4e6e79dd515a54a2c321b077
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87798841"
---
# <a name="complete-an-access-review-of-groups-and-applications-in-azure-ad-access-reviews"></a>Realización de una revisión de acceso de los grupos y las aplicaciones en las revisiones de acceso de Azure AD

Como administrador, [debe crear una revisión de acceso de grupos o aplicaciones](create-access-review.md). Los revisores se encargan de [realizar la revisión de acceso](perform-access-review.md). En este artículo se describe cómo ver los resultados de la revisión de acceso y se aplican los resultados.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

## <a name="prerequisites"></a>Prerrequisitos

- Azure AD Premium P2
- Administrador global, administrador de usuarios, administrador de seguridad o lector de seguridad

Para obtener más información, consulte [Requisitos de licencia](access-reviews-overview.md#license-requirements).

## <a name="view-an-access-review"></a>Visualización de una revisión de acceso

Puede hacer un seguimiento del progreso de las revisiones a medida que los revisores las completan.

1. Inicie sesión en Azure Portal y abra la [página de Identity Governance](https://portal.azure.com/#blade/Microsoft_AAD_ERM/DashboardBlade/).

1. En el menú izquierdo, haga clic en **Revisiones de acceso**.

1. En la lista, haga clic en una revisión de acceso.

    Para ver una serie de revisiones de acceso, vaya a la revisión de acceso y encontrará los próximos eventos en Revisiones programadas.

    En la página **Información general**, puede ver el progreso. Los derechos de acceso no se cambian en el directorio hasta que la revisión finaliza.

    ![Progreso de las revisiones de acceso](./media/complete-access-review/overview-progress.png)

1. Si quiere detener una revisión de acceso antes de que alcance la fecha de finalización programada, haga clic en el botón **Detener**.

    Al detener una revisión, los revisores ya no podrán proporcionar respuestas. No puede reiniciar una revisión una vez detenida.

1. Si ya no está interesado en la revisión de acceso, puede hacer clic en el botón **Eliminar** para eliminarla.

## <a name="apply-the-changes"></a>Aplicación de cambios

Si la opción **Aplicar automáticamente los resultados al recurso** estaba habilitado y basada en sus selecciones de **Configuración de finalización**, la aplicación automática se ejecutará después de la fecha de finalización de la revisión o cuando esta se detenga manualmente.

Si **Aplicar automáticamente los resultados al recurso** no estaba habilitado para la revisión, haga clic en **Aplicar** para aplicar los cambios manualmente. Si el acceso de un usuario se denegó en la revisión, al hacer clic en **Aplicar**, Azure AD quita la asignación de pertenencia o de la aplicación.

![Aplicación de cambios de la revisión de acceso](./media/complete-access-review/apply-changes.png)

El estado de la revisión cambiará de **Completado** a estados intermedios como **Aplicando** y, por último, a **Resultado aplicado**. Debería esperar ver a los usuarios denegados, si es que los hay, eliminados de la pertenencia al grupo o la asignación de aplicaciones en unos minutos.

Una revisión de aplicación automática configurada o la selección de **Aplicar** no tiene ningún efecto en un grupo que se origina en un directorio local o en un grupo dinámico. Si desea cambiar un grupo que se origina en un directorio local, descargue los resultados y aplique esos cambios a la representación del grupo en ese directorio.

## <a name="retrieve-the-results"></a>Recuperación de los resultados

Para ver los resultados de una revisión de acceso de un solo uso, haga clic en la página **Resultados**. Para ver solo el acceso de un usuario, en el cuadro Buscar escriba el nombre para mostrar o el nombre principal de usuario de un usuario cuyo acceso se ha revisado.

![Recuperación de los resultados de una revisión de acceso](./media/complete-access-review/retrieve-results.png)

Para ver el progreso de una revisión de acceso activa que es periódica, haga clic en la página **Resultados**.

Para ver los resultados de una instancia finalizada de una revisión de acceso que sea recurrente, haga clic en **Historial de revisiones** y, a continuación, seleccione la instancia específica de la lista de instancias de la revisión de acceso finalizada, en función de las fechas inicial y final de la instancia. Se pueden obtener los resultados de esta instancia en la página **Resultados**.

Para recuperar todos los resultados de una revisión de acceso, haga clic en el botón **Descargar**. El archivo CSV resultante puede verse en Excel o en otros programas que abren archivos CSV codificados en UTF-8.

## <a name="remove-users-from-an-access-review"></a>Eliminación de usuarios de una revisión de acceso

 De forma predeterminada, un usuario eliminado permanecerá eliminado en Azure AD durante 30 días, tiempo durante el cual un administrador puede restaurarlo si es necesario.  Después de 30 días, ese usuario se eliminará definitivamente.  Además, mediante el portal de Azure Active Directory, un administrador global puede [eliminar permanentemente un usuario eliminado recientemente](../fundamentals/active-directory-users-restore.md) explícitamente antes de que se alcance ese período de tiempo.  Una vez que un usuario se ha eliminado permanentemente, los datos posteriores sobre ese usuario se eliminarán de las revisiones de acceso activas.  La información de auditoría de los usuarios eliminados se conserva en el registro de auditoría.

## <a name="next-steps"></a>Pasos siguientes

- [Administración del acceso de los usuarios con las revisiones de acceso de Azure AD](manage-user-access-with-access-reviews.md)
- [Administración del acceso de los invitados con las revisiones de acceso de Azure AD](manage-guest-access-with-access-reviews.md)
- [Creación de una revisión de acceso de grupos o aplicaciones](create-access-review.md)
- [Creación de una revisión de acceso de los usuarios en un rol administrativo de Azure AD](../privileged-identity-management/pim-how-to-start-security-review.md)
