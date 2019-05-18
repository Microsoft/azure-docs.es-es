---
title: Configuración de la seguridad para el acceso y la administración de Azure Time Series Insights | Microsoft Docs
description: En este artículo se describe cómo configurar la seguridad y los permisos como directivas de acceso de administración y directivas de acceso a datos para proteger la versión preliminar de Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.workload: big-data
ms.topic: conceptual
ms.date: 05/01/2019
ms.custom: seodec18
ms.openlocfilehash: 6853637ba23e17f3a7ca5420bdd84425c81a67be
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65791017"
---
# <a name="grant-data-access-to-an-environment"></a>Concesión de acceso a datos en un entorno

En este artículo se tratan los dos tipos de directivas de acceso de la versión preliminar de Azure Time Series Insights.

## <a name="sign-in-to-time-series-insights"></a>Inicie sesión en Time Series Insights

1. Inicie sesión en el [Azure Portal](https://portal.azure.com/).
1. Busque el entorno de Time Series Insights. Especifique `Time Series` en el cuadro **Búsqueda**. Seleccione **Entorno de Time Series** en los resultados de búsqueda.
1. Seleccione el entorno de Time Series Insights de la lista.

## <a name="grant-data-access"></a>Concesión de acceso a datos

Siga estos pasos para conceder acceso a datos a una entidad de seguridad de usuario.

1. Seleccione **Directivas de acceso a datos** y, luego, **+ Agregar**.

    [![Uno de acceso de datos](media/data-access/data-access-one.png)](media/data-access/data-access-one.png#lightbox)

1. Elija **Seleccionar usuario**. Busque el nombre de usuario o la dirección de correo para encontrar al usuario que quiere agregar. Haga clic en **Seleccionar** para confirmar la selección.

    [![Data-access-two](media/data-access/data-access-two.png)](media/data-access/data-access-two.png#lightbox)

1. Elija **Seleccionar rol**. Elija el rol de acceso adecuado para el usuario:

    * Seleccione **Colaborador** si quiere permitir al usuario cambiar los datos de referencia y compartir las consultas y las perspectivas guardadas con otros usuarios del entorno.

    * En caso contrario, seleccione **Lector** para permitir al usuario consultar los datos del entorno y guardar consultas personales (no compartidas) en el entorno.

   Seleccione **Aceptar** para confirmar la elección del rol.

    [![Tres de acceso de datos](media/data-access/data-access-three.png)](media/data-access/data-access-three.png#lightbox)

1. Seleccione **Aceptar** en la página **Seleccionar rol de usuario**.

    [![Cuatro de acceso de datos](media/data-access/data-access-four.png)](media/data-access/data-access-four.png#lightbox)

1. Confirme que la página **Directivas de acceso a datos** enumera los usuarios y los roles de cada uno.

    [![Cinco de acceso de datos](media/data-access/data-access-five.png)](media/data-access/data-access-five.png#lightbox)

## <a name="provide-guest-access-from-another-aad-tenant"></a>Proporcionar acceso de invitados de otro inquilino AAD

`Guest` no es un rol de administración. Se trata de un término que se usa para una cuenta a la que se ha invitado desde un inquilino a otro. Después de que la cuenta de invitado se haya invitado al directorio del inquilino, se le puede aplicar el mismo control de acceso que a cualquier otra cuenta. Puede conceder acceso de administración a un entorno de Time Series Insights mediante la hoja de Control de acceso (IAM). O puede conceder acceso a los datos del entorno utilizando la hoja Directivas de acceso a datos. Para más información sobre el acceso de invitado de inquilino de Azure Active Directory (Azure AD), consulte [Incorporación de usuarios de colaboración B2B de Azure Active Directory en Azure Portal](https://docs.microsoft.com/azure/active-directory/b2b/add-users-administrator).

Siga estos pasos para conceder acceso de invitado a un entorno de Time Series Insights a un usuario de Azure AD desde otro inquilino.

1. Seleccione **Directivas de acceso a datos** y, luego, **+ Invitar**.

    [![Datos-access-seis](media/data-access/data-access-six.png)](media/data-access/data-access-six.png#lightbox)

1. Escriba la dirección de correo electrónico del usuario que desea invitar. Esta dirección de correo electrónico tiene que estar asociada con Azure AD. Opcionalmente puede incluir un mensaje personal con la invitación.

    [![Datos-access-siete](media/data-access/data-access-seven.png)](media/data-access/data-access-seven.png#lightbox)

1. Busque la burbuja de confirmación que aparece en la pantalla.

    [![Ocho de acceso de datos](media/data-access/data-access-eight.png)](media/data-access/data-access-eight.png#lightbox)

1. Elija **Seleccionar usuario**. Busque la dirección de correo electrónico del usuario que ha invitado para ubicar al usuario que quiere agregar. Haga clic en **Seleccionar** para confirmar la selección.

    [![Datos y acceso-nueve](media/data-access/data-access-nine.png)](media/data-access/data-access-nine.png#lightbox)

1. Elija **Seleccionar rol**. Elija el rol de acceso adecuado para el usuario invitado:

    * Seleccione **Colaborador** si quiere permitir al usuario cambiar los datos de referencia y compartir las consultas y las perspectivas guardadas con otros usuarios del entorno.

    * En caso contrario, seleccione **Lector** para permitir al usuario consultar los datos del entorno y guardar consultas personales (no compartidas) en el entorno.

   Seleccione **Aceptar** para confirmar la elección del rol.

    [![Diez de acceso de datos](media/data-access/data-access-ten.png)](media/data-access/data-access-ten.png#lightbox)

1. Seleccione **Aceptar** en la página **Seleccionar rol de usuario**.

1. Confirme que la página **Directivas de acceso a datos** aparece el usuario invitado y los roles de cada usuario invitado.

    [![Datos-access-eleven](media/data-access/data-access-eleven.png)](media/data-access/data-access-eleven.png#lightbox)

1. Ahora el usuario invitado tiene que realizar ciertos pasos para acceder al entorno que se encuentra en el inquilino de Azure al que se le ha invitado. En primer lugar, el usuario tiene que aceptar la invitación que le ha enviado. Esta invitación se envía por correo electrónico a la dirección que utilizó en el paso 5. Para aceptar el usuario invitado selecciona **Get Started** (Inicio).

    [![Data-access-twelve](media/data-access/data-access-twelve.png)](media/data-access/data-access-twelve.png#lightbox)

1. A continuación el usuario invitado debe aceptar los permisos asociados a la organización del administrador.

    [![Datos-access-trece](media/data-access/data-access-thirteen.png)](media/data-access/data-access-thirteen.png#lightbox)

1. Cuando el usuario invitado inicia sesión en la dirección de correo electrónico que utilizó para invitarle y acepta la invitación, tiene que ir a insights.azure.com. Una vez allí, el usuario selecciona el avatar situado junto a su dirección de correo electrónico en la esquina superior derecha de la pantalla.

    [![Catorce de datos de access](media/data-access/data-access-fourteen.png)](media/data-access/data-access-fourteen.png#lightbox)

1. Luego, el usuario invitado selecciona su inquilino de Azure en el menú desplegable del directorio. Este inquilino es aquel al que le ha invitado.

    [![Datos-access-quince](media/data-access/data-access-fifteen.png)](media/data-access/data-access-fifteen.png#lightbox)

Una vez que el usuario invitado selecciona su inquilino, verá el entorno de Time Series Insights al que le ha proporcionado acceso. Ahora tienen todas las funcionalidades asociadas con el rol que se les proporcionó con en **paso 5**.

## <a name="next-steps"></a>Pasos siguientes

* Aprenda [cómo agregar un origen del evento de Azure Event Hubs](./time-series-insights-how-to-add-an-event-source-eventhub.md) al entorno de Time Series Insights.

* Envíe [eventos al origen del evento](./time-series-insights-send-events.md).

* Visualice [el entorno en el explorador de la versión preliminar de Time Series Insights](./time-series-insights-update-explorer.md).
