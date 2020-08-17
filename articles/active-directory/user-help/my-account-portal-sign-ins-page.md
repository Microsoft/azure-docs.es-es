---
title: Visualización y búsqueda de la actividad de inicio de sesión reciente desde la página Mis inicios de sesión en Azure Active Directory | Microsoft Docs
description: Detalles acerca de cómo ver y buscar la actividad de inicio de sesión reciente en la página Mis inicios de sesión del portal Mi cuenta.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: rhicock
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 08/03/2020
ms.author: curtand
ms.openlocfilehash: 1816170da0fb6e5120f8cec6b6dda68d2ea1c678
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543240"
---
# <a name="view-and-search-your-recent-sign-in-activity-from-the-my-sign-ins-page"></a>Visualización y búsqueda de la actividad de inicio de sesión reciente desde la página Mis inicios de sesión

Puede ver toda la actividad de inicio de sesión de la cuenta profesional o educativa en la página **Mis inicios de sesión** del portal **Mi cuenta**. La revisión del historial de inicios de sesión lo ayuda a comprobar la actividad inusual al permitirlo ver:

- Si alguien intenta adivinar su contraseña.
- Si un atacante inició sesión correctamente en su cuenta y desde qué ubicación.
- Las aplicaciones a las que el atacante ha intentado acceder.

## <a name="view-your-recent-sign-in-activity"></a>Visualización de las actividades de inicio de sesión recientes

1. Inicie sesión en su cuenta profesional o educativa y vaya a la página https://myaccount.microsoft.com/.

2. Seleccione **Mis inicios de sesión** en el panel de navegación izquierdo o seleccione el vínculo **Revisar la actividad reciente** en el bloque **Mis inicios de sesión**.

    ![Página Mi cuenta, que muestra vínculos resaltados a Actividad reciente](media/my-account-portal/my-account-portal-sign-ins.png)

3. Expanda y revise cada uno de los elementos de inicio de sesión, asegurándose de que reconoce cada uno de ellos. Si encuentra un elemento de inicio de sesión que no le resulta familiar, le recomendamos que cambie la contraseña para proteger la cuenta en caso de que se viera comprometida.

    ![Página Actividad reciente con los detalles de inicio de sesión expandidos](media/my-account-portal-sign-ins-page/recent-activity.png)

### <a name="if-you-see-a-successful-sign-in"></a>Si ve un inicio de sesión correcto

A veces, al revisar su propia actividad de inicio de sesión normal, es posible que vea un inicio de sesión correcto desde una ubicación, un explorador o un sistema operativo desconocidos. Los inicios de sesión desconocidos pueden significar que un atacante ha obtenido acceso a su cuenta. Si ve alguna actividad que no ha autorizado, es recomendable que cambie inmediatamente la contraseña y vaya a [Información de seguridad](https://mysignins.microsoft.com/security-info) para actualizar la configuración de seguridad.

Antes de determinar si algo es incorrecto, asegúrese de que no está viendo un falso positivo (donde el elemento es cuestionable, pero es correcto). Por ejemplo, determinamos la ubicación aproximada y el mapa en función de su dirección IP. Las redes móviles son especialmente difíciles de identificar, ya que a veces enrutan el tráfico a través de ubicaciones lejanas. Incluso si ha iniciado sesión con el dispositivo móvil en el estado de Washington, la ubicación podría mostrar que el inicio de sesión procede de California. Por esta razón, es recomendable que consulte más detalles, además de la ubicación. Asegúrese también de que el sistema operativo, el explorador y la aplicación tienen sentido.

### <a name="if-you-see-an-unsuccessful-sign-in"></a>Si ve un inicio de sesión incorrecto

Si ve un inicio de sesión incorrecto, podría significar que escribió incorrectamente sus credenciales. También podría significar que un atacante estaba intentando adivinar la contraseña. Para responder a este riesgo, no es necesario cambiar la contraseña, pero es recomendable registrarse en Azure Multi-Factor Authentication (MFA). Con la autenticación multifactor, incluso si el hacker adivina su contraseña, no será suficiente para acceder a la cuenta.

![Icono de inicio de sesión incorrecto](media/my-account-portal-sign-ins-page/unsuccessful.png)

Si ve un inicio de sesión incorrecto con una nota bajo **Actividad de la sesión** que dice `Additional verification failed, invalid code`, significa que las credenciales de autenticación principales son correctas, pero la autenticación multifactor no se ha completado correctamente. Esta condición podría significar que un atacante ha adivinado correctamente la contraseña, pero no ha podido superar el desafío de la autenticación multifactor. Le recomendamos que cambie la contraseña, ya que el atacante podría tenerla, y que vaya a la página [Información de seguridad ](https://mysignins.microsoft.com/security-info) para actualizar la configuración de seguridad.

## <a name="search-for-specific-sign-in-activity"></a>Búsqueda de actividad de inicio de sesión específica

Puede buscar en la actividad de inicio de sesión reciente cualquier información disponible. Por ejemplo, puede buscar la actividad de inicio de sesión reciente por sistema operativo, ubicación, aplicación, etc.

1. En la página **Revisar la actividad reciente**, escriba la información que desea buscar en la barra **Búsqueda**. Por ejemplo, escriba `Unsuccessful` para buscar toda la actividad de inicio de sesión incorrecta que haya recopilado la aplicación Mi cuenta.

2. Seleccione el botón **Buscar** para empezar a buscar.

    ![Página Actividad reciente, donde se muestra la barra de búsqueda, el botón de búsqueda y los resultados resaltados](media/my-account-portal-sign-ins-page/sign-in-search.png)

### <a name="confirm-unusual-activity"></a>Confirmar actividad inusual

Los inicios de sesión marcados como actividad inusual se pueden confirmar en el icono de esa actividad en la página **Mis inicios de sesión**.

![Icono de inicio de sesión inusual para confirmar que intentó o no iniciar la sesión](media/my-account-portal-sign-ins-page/this-wasnt-me.png)

## <a name="next-steps"></a>Pasos siguientes

Después de ver la actividad de inicio de sesión reciente, puede:

- Ver o administrar la [información de seguridad](user-help-security-info-overview.md).

- Ver o administrar los dispositivos [conectados](my-account-portal-devices-page.md).

- Ver o administrar las [organizaciones](my-account-portal-organizations-page.md).

- Ver cómo se [usan los datos relacionados con la privacidad](my-account-portal-privacy-page.md) en la organización.

- Cambiar la [configuración del portal Mi cuenta](my-account-portal-settings.md).
