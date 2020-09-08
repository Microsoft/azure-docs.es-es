---
title: 'Correcciones del error "No se puede llegar allí desde aquí": Azure AD'
description: Encuentre correcciones posibles para el motivo por el que obtiene el mensaje de error "No se puede llegar allí desde aquí".
services: active-directory
author: curtand
manager: daveba
ms.assetid: 8ad0156c-0812-4855-8563-6fbff6194174
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 10/10/2018
ms.author: curtand
ms.reviewer: jairoc
ms.custom: user-help, seo-update-azuread-jan
ms.openlocfilehash: df3941c895ce67862eb53b8e96bc7a6d53c1ed02
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799424"
---
# <a name="possible-fixes-for-the-you-cant-get-there-from-here-error-message"></a>Correcciones posibles para el mensaje de error "No se puede llegar allí desde aquí"

Durante el acceso a servicios o aplicaciones web internos de su organización, es posible que obtenga un mensaje de error que dice, **No se puede llegar allí desde aquí**. Este mensaje indica que la organización ha establecido una directiva que impide el acceso del dispositivo a los recursos de la organización. Si bien es posible que tenga que ponerse en contacto con su departamento de soporte técnico para solucionar este problema, aquí tiene algunas cosas que puede probar primero.

## <a name="make-sure-youre-using-a-supported-browser"></a>Comprobación de que usa un explorador compatible
Si recibe el mensaje **No se puede llegar allí desde aquí**, que indica que intenta tener acceso a sitios de su organización desde un explorador no compatible, compruebe el explorador que ejecuta.

![Mensaje de error relacionado con la compatibilidad con exploradores](media/user-help-device-remediation/browser-version.png)

Para corregir este problema, debe instalar y ejecutar un explorador compatible según su sistema operativo. Si usa Windows 10, entre los exploradores compatibles se incluyen Microsoft Edge, Internet Explorer y Google Chrome. Si usa otro sistema operativo, puede consultar la lista completa de [exploradores compatibles](../conditional-access/concept-conditional-access-conditions.md#supported-browsers).

## <a name="make-sure-youre-using-a-supported-operating-system"></a>Comprobación de que usa un sistema operativo compatible
Asegúrese de que ejecuta una versión compatible del sistema operativo, lo que incluye:

- **Cliente Windows.** Windows 7 o versión posterior.

- **Windows Server.** Windows Server 2008 R2 o versiones posteriores.

- **macOS.** macOS X o versión posterior.

- **iOS y Android.** Versión más reciente de los sistemas operativos para dispositivos móviles iOS y Android.

Para corregir este problema, debe instalar y ejecutar un sistema operativo compatible.

## <a name="make-sure-your-device-is-joined-to-your-network"></a>Comprobación de que el dispositivo está unido a la red
Si recibe el mensaje **No se puede llegar allí desde aquí**, que indica que el dispositivo no cumple con la directiva de acceso de la organización, asegúrese de que el dispositivo se ha unido a la red de la organización.

![Mensaje de error relacionado con si encuentra en la red](media/user-help-device-remediation/network-version.png)

### <a name="to-check-whether-your-device-is-joined-to-your-network"></a>Para comprobar que el dispositivo está unido a la red
1. Inicie sesión en Windows con la cuenta profesional o educativa. Por ejemplo, alain@contoso.com.

2. Conéctese a la red de la organización a través de una red privada virtual (VPN) o de DirectAccess.

3. Una vez conectado, presione a la vez la **tecla del logotipo de Windows y la tecla L** para bloquear el dispositivo.

4. Desbloquee el dispositivo con la cuenta profesional o educativa y luego intente obtener acceso de nuevo a la aplicación o el servicio con problemas.

    Si vuelve a ver el mensaje de error **No se puede llegar allí desde aquí**, seleccione el vínculo **Más detalles** y póngase en contacto con su departamento de soporte técnico con los detalles.

### <a name="to-join-your-device-to-your-network"></a>Para unir el dispositivo a la red
Si el dispositivo no está unido a la red de la organización, puede realizar una de estas dos acciones:

- **Unión del dispositivo de trabajo.** Una su dispositivo Windows 10 de trabajo a la red de su organización para poder tener acceso a recursos potencialmente restringidos. Para más información e instrucciones detalladas, vea el artículo sobre cómo [unir su dispositivo de trabajo a la red de su organización](user-help-join-device-on-network.md).

- **Registro de un dispositivo personal para el trabajo.** Registre su dispositivo personal, que suele ser un teléfono o una tablet, en la red de su organización. Una vez registrado, el dispositivo tendrá acceso a los recursos restringidos de su organización. Para más información e instrucciones detalladas, vea el artículo sobre cómo [registrar su dispositivo personal en la red de su organización](user-help-register-device-on-network.md).

## <a name="next-steps"></a>Pasos siguientes
- [¿Qué es el portal MyApps?](./my-apps-portal-end-user-access.md)

- [Inicie sesión con el teléfono, no con la contraseña](user-help-auth-app-sign-in.md)