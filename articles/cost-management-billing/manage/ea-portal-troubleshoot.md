---
title: Solución de problemas de acceso al portal de Azure EA
description: En este artículo se describen algunos problemas comunes que pueden producirse en una Contrato Enterprise de Azure (EA) en el portal de Azure EA.
author: bandersmsft
ms.author: banders
ms.date: 08/20/2020
ms.topic: troubleshooting
ms.service: cost-management-billing
ms.subservice: enterprise
ms.reviewer: boalcsva
ms.openlocfilehash: 5402ee3283e5ae11f43884b4002486e8bdefae2c
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89294414"
---
# <a name="troubleshoot-azure-ea-portal-access"></a>Solución de problemas de acceso al portal de Azure EA

En este artículo se describen algunos problemas comunes que pueden producirse en una Contrato Enterprise de Azure (EA). El portal del Contrato Enterprise de Azure se utiliza para administrar los costos y usuarios del Contrato Enterprise. Al configurar o actualizar el acceso al portal del Contrato Enterprise de Azure es posible que surjan estos problemas.

## <a name="issues-adding-an-admin-to-an-enrollment"></a>Problemas al agregar un administrador a una inscripción

Hay diferentes tipos de niveles de autenticación para las inscripciones empresariales. Si los niveles de autenticación se aplican de forma incorrecta, es posible que tenga problemas al intentar iniciar sesión en el portal del Contrato Enterprise de Azure.

El portal del Contrato Enterprise de Azure se usa para conceder acceso a los usuarios con distintos niveles de autenticación. Un administrador de empresa puede actualizar el nivel de autenticación para cumplir los requisitos de seguridad de su organización.

### <a name="authentication-level-types"></a>Tipos de nivel de autenticación

- Solo cuenta de Microsoft: para organizaciones que deseen usar, crear y administrar usuarios mediante cuentas de Microsoft.
- Cuenta profesional o educativa: para organizaciones que han configurado Active Directory con Federación en la nube y todas las cuentas están en un solo inquilino.
- Cuenta profesional o educativa entre inquilinos: para organizaciones que han configurado Active Directory con Federación en la nube y van a tener cuentas en varios inquilinos.
- Cuenta mixta: permite agregar usuarios con una cuenta Microsoft o con una cuenta profesional o educativa.

La primera cuenta profesional o educativa agregada a la inscripción determina el dominio _predeterminado_. Para agregar una cuenta profesional o educativa con otro inquilino, es preciso cambiar el nivel de autenticación de la inscripción a autenticación entre inquilinos.

Para actualizar el nivel de autenticación:

1. Inicie sesión en el portal del Contrato Enterprise de Azure como administrador de empresa.
2. Haga clic en **Administrar** en el panel de navegación izquierdo.
3. Haga clic en la pestaña **Inscripción**.
4. En **Detalles de la inscripción**, seleccione **Auth Level** (Nivel de autenticación).
5. Haga clic en el símbolo del lápiz.
6. Haga clic en **Save**(Guardar).

![Ejemplo que muestra los niveles de autenticación ](./media/ea-portal-troubleshoot/create-ea-authentication-level-types.png)

Las cuentas de Microsoft deben tener un identificador asociado creado en [https://signup.live.com](https://signup.live.com/).

Las cuentas profesionales o educativas están disponibles para las organizaciones que hayan configurado Active Directory con federación y donde todas las cuentas estén en un solo inquilino. Los usuarios se pueden agregar con autenticación de usuario federado profesional o educativa si la instancia de Active Directory interna de la empresa está federado.

Si la organización no usa la federación de Active Directory, no puede usar su dirección de correo electrónico profesional o educativa. En su lugar, registre o cree una nueva dirección de correo electrónico y regístrela como cuenta de Microsoft.

## <a name="unable-to-access-the-azure-ea-portal"></a>No se puede acceder al portal del Contrato Enterprise de Azure

Si recibe un mensaje de error al intentar iniciar sesión en el portal del Contrato Enterprise de Azure, use los siguientes pasos para solucionarlo:

- Asegúrese de que usa la dirección URL correcta del portal del Contrato Enterprise de Azure, que es https://ea.azure.com.
- Determine si el acceso al portal del Contrato Enterprise de Azure se ha agregado como una cuenta profesional o educativa, o bien como cuenta de Microsoft.
  - Si usa una cuenta profesional, escriba su correo electrónico y contraseña profesionales. La contraseña se la proporciona su organización. Si tiene algún problema, puede consultar al departamento de TI cómo restablecer la contraseña.
  - Si utiliza una cuenta de Microsoft, escriba su dirección de correo electrónico y contraseña de la cuenta de Microsoft. Si ha olvidado la contraseña de cuenta de Microsoft, puede restablecerla en [https://account.live.com/password/reset](https://account.live.com/password/reset).
- Use una sesión InPrivate o de incógnito en el explorador para iniciar sesión, con el fin de que no se guarden cookies ni la información almacenada en la caché de las sesiones anteriores o existentes. Borre la memoria caché del explorador y use una ventana InPrivate o de incógnito para abrir https://ea.azure.com.
- Si recibe el error _Usuario no válido_  al usar un cuenta de Microsoft, podría deberse a que tiene varias cuentas de Microsoft. La dirección con la que intenta iniciar sesión no es la principal.
También aparecería el error _Usuario no válido_ si se ha usado un tipo de cuenta incorrecto al agregar el usuario a la inscripción. Por ejemplo, una cuenta profesional o educativa, en lugar de una cuenta de Microsoft. En este ejemplo, pida a administrador de EA que agregue la cuenta correcta o póngase en contacto con el [servicio de soporte técnico](https://support.microsoft.com/supportforbusiness/productselection?sapId=cf791efa-485b-95a3-6fad-3daf9cd4027c).
  - Si necesita comprobar el alias principal, vaya a [https://account.live.com](https://account.live.com). Luego, haga clic en **Your Info** (Tu información) y, después, en **Manage how to sign in to Microsoft** (Administrar el modo de iniciar sesión en Microsoft). Siga las indicaciones para comprobar otra dirección de correo electrónico y obtener un código para acceder a la información confidencial. Escriba el código de seguridad. Seleccione **Configurarlo más tarde**  si no desea configurar la autenticación en dos fases.
  - Verá la página **Manage how to sign in to Microsoft** (Administrar el modo de iniciar sesión en Microsoft), donde podrá ver los alias de cuenta que tiene. Compruebe que el alias principal es el que usa para iniciar sesión en el portal de Azure EA. Si no lo es, puede convertirlo en el alias principal. O bien, puede usar el alias principal del portal del Contrato Enterprise de Azure.

## <a name="azure-ea-activation-faq"></a>Preguntas frecuentes sobre la activación de Azure EA

En esta sección del artículo se describen las soluciones a problemas comunes relacionados con la activación de Azure EA.

### <a name="i-didnt-receive-an-activation-email"></a>No he recibido un correo electrónico de activación

Se envía un correo electrónico de activación del portal del Contrato Enterprise de Azure desde *waep@microsoft.com* . Si no ha recibido un correo electrónico de activación, búsquelo en la carpeta de correo no deseado. Se envía con el asunto _Invitation to View/Manage the Microsoft Azure service_ (Invitación para ver o administrar el servicio de Microsoft Azure). Se envía a todos los administradores de EA recién agregados.

Si está seguro de que en su configuración es administrador de EA, no es preciso que espere a recibir el correo electrónico de activación para iniciar sesión en el portal del Contrato Enterprise de Azure. En su lugar, puede ir a https://ea.azure.com e iniciar sesión con su dirección de correo electrónico (cuenta profesional, educativa o de Microsoft) y su contraseña.

### <a name="i-would-like-to-add-a-new-ea-administrator-to-my-enrollment"></a>Me gustaría agregar un nuevo administrador de EA a mi inscripción

Los administradores de empresa existentes pueden agregar otro nuevo administrador de empresa. Si es el administrador de EA, inicie sesión en EA Portal > haga clic en **Administrar** > haga clic en **+ Agregar administrador** en la esquina superior derecha para agregar un nuevo administrador de EA. Asegúrese de que tiene su dirección de correo electrónico y un método de inicio de sesión preferido, por ejemplo, mediante la autenticación profesional/educativa o Microsoft Live ID para agregar a los usuarios.

Si no es el administrador de EA, póngase en contacto con los administradores de EA de su empresa para solicitar que le agreguen a la inscripción. Una vez que le hayan agregado a la inscripción, recibirá un correo electrónico de activación.

Sin embargo, si los administradores de EA no pueden ayudarlo, podremos agregarlo en su nombre si puede proporcionarnos lo siguiente:
- El número de inscripción.
- La dirección de correo electrónico que se debe agregar y el tipo de autenticación (cuenta profesional, educativa o de Microsoft).
- La aprobación del correo electrónico por parte del administrador de EA.

Una vez que tenga toda la información necesaria, envíe una solicitud en [https://aka.ms/AzureEntSupport](https://aka.ms/AzureEntSupport).

### <a name="i-would-like-to-update-the-first-ea-admin-on-the-enrollment"></a>Me gustaría actualizar el primer administrador de EA en la inscripción

El primer administrador de EA puede actualizarse en el centro de servicios de licencias por volumen mediante la actualización del contacto de avisos y el administrador en línea en el portal. Se tardarán aproximadamente 24 horas en actualizar EA Portal. Una vez que se actualice, el nuevo administrador de EA recibirá el correo electrónico de activación.

Si no tiene acceso al portal de VLSC o si el administrador de EA inicial ya no puede administrar la inscripción y no tiene acceso a EA Portal, envíe una solicitud en [https://aka.ms/AzureEntSupport](https://aka.ms/AzureEntSupport) para solicitar una actualización y proporcione la siguiente información:
- Número de inscripción
- La dirección de correo electrónico que se debe agregar y el tipo de autenticación (cuenta profesional, educativa o de Microsoft)
- El motivo para cambiar el administrador de EA inicial
- La aprobación del correo electrónico por parte del administrador de EA inicial

### <a name="my-current-ea-admin-is-no-longer-with-the-company"></a>Mi administrador de EA actual ya está en la compañía

Una inscripción de EA puede tener varios administradores de EA. puede ponerse en contacto con otro para que agregue a los nuevos propietarios de cuentas y administradores de EA o administradores de departamento. Sin embargo, si no está claro quién es el administrador de EA de su empresa o no hay ningún otro disponible en la inscripción, póngase en contacto con nosotros con la siguiente información:
- Número de inscripción
- La dirección de correo electrónico que se debe agregar y el tipo de autenticación (cuenta profesional, educativa o de Microsoft)
- Información sobre que el administrador de EA actual ya no está en la compañía

Tenga en cuenta que, si hay otros administradores de EA en la inscripción, nos pondremos en contacto con ellos para solicitar la aprobación de los cambios administrativos en la inscripción.

### <a name="my-enrollment-is-showing-in-pending-status-how-do-i-activate-my-enrollment"></a>Mi inscripción aparece en estado pendiente. ¿Cómo activo mi inscripción?

Las inscripciones estarán en estado pendiente si el administrador de EA inicial no ha iniciado sesión en la inscripción antes. Si es el administrador de EA, inicie sesión en Azure EA Portal. En la página de aterrizaje con todos los números de inscripción, es posible que no vea su inscripción pendiente. Desactive la casilla "activa" en la esquina superior derecha de EA Portal; así se mostraría la inscripción pendiente. Haga clic en la inscripción para acceder a la información y, una vez que haya llegado a la página de administración de la inscripción, el estado se actualizará de pendiente a activo.

### <a name="why-is-my-account-stuck-in-pending-status"></a>¿Por qué mi cuenta está bloqueada en el estado pendiente?

Cuando se agregan nuevos propietarios de la cuenta a una inscripción por primera vez, su estado aparece como "pendiente". Al recibir el correo electrónico de bienvenida de activación, el propietario de la cuenta puede iniciar sesión para activarla. Al iniciar sesión, se actualizará el estado de la cuenta de "pendiente" a "activo".

### <a name="i-received-an-error-when-signing-in-to-azure-ea-portal"></a>He recibido un error al iniciar sesión en Azure EA Portal

Hay varias razones posibles para un mensaje de error en Azure EA Portal durante el proceso de iniciar sesión. Siga estos pasos para solucionar problemas:

 1. Asegúrese de que está usando la dirección URL correcta de EA Portal en [https://ea.azure.com](https://ea.azure.com).
 1. Determine si el acceso a Azure EA Portal se ha agregado como una cuenta profesional o educativa, o bien como Microsoft Live ID. Si usa una cuenta profesional, escriba su correo electrónico y contraseña profesionales. Si usa Microsoft Live ID, escriba su correo electrónico de Live ID y la contraseña de Microsoft Live ID. Si olvidó su contraseña de Microsoft Live ID, vuelva a establecerla en [https://account.live.com/password/reset](https://account.live.com/password/reset).
 1. Se recomienda que use un explorador privado para iniciar sesión de modo que no se conserven cookies ni caché de sesiones anteriores o existentes. Borre la memoria caché y use el modo de exploración privado/ventana incógnito para abrir [https://ea.azure.com](https://ea.azure.com).
 1. Si recibe un error de usuario no válido al usar un cuenta de Microsoft, puede deberse a que tiene varias cuentas de Microsoft y en la que está intentando iniciar sesión no es el alias principal. Para comprobar el alias principal, vaya a account.live.com:
    - Vaya a "Tu información" > "Manage your sign-in email or phone number" (Administrar el correo electrónico o el número de teléfono de inicio de sesión).
    - Siga las indicaciones de la pantalla para comprobar otra dirección de correo electrónico y obtener un código para acceder a la información confidencial.
    - Escriba el código de seguridad.
    - Si prefiere configurar más tarde la autenticación en dos pasos, seleccione "Configurarlo más tarde".
    - En la página "Administrar los alias de la cuenta" verá los alias de cuenta que tiene. Compruebe que el alias principal es el que usa para iniciar sesión en Azure EA Portal. Si no es así, puede convertirlo en su alias principal o puede usar el alias principal de EA Portal en su lugar.

Si se produce un error en los pasos de solución de problemas anteriores, envíe una solicitud en [https://aka.ms/AzureEntSupport](https://aka.ms/AzureEntSupport) con información como la siguiente:
- Exploradores y versión usados.
- Captura de pantalla del mensaje de error.
- Dirección URL de la página que muestra el error.  
- Fecha, hora y zona horaria de cuando se ha producido el error.
- Además, será útil obtener un archivo de registro. Estos son los pasos para capturar un seguimiento de red mediante la información siguiente:
  1. Abra Internet Explorer.
  1. Presione F12, lo que abrirá un cuadro en la parte inferior de IE.
  1. Seleccione la pestaña **Red**.
  1. Haga clic en **Iniciar la captura**.
  1. Realice la acción que está causando el error.
  1. Una vez que reciba el error, haga clic en **Detener la captura**.
  1. Guarde el archivo e incluya la información en la solicitud de soporte técnico.
  1. Asegúrese de proporcionar el número de inscripción y la dirección de correo electrónico dentro de la solicitud de soporte técnico.

### <a name="what-is-the-difference-between-a-workschool-account-and-microsoft-account"></a>¿Cuál es la diferencia entre una cuenta profesional o educativa, y una cuenta de Microsoft?

**Cuenta de Microsoft:** cuentas que se han asociado a Live ID en [https://signup.live.com](https://signup.live.com).

**Cuenta profesional o educativa:** solo disponible para compañías que han configurado Active Directory con Federación en la nube y todas las cuentas están en un solo inquilino. Los usuarios se pueden agregar con el tipo de autorización profesional o educativa si la instancia de Active Directory interna activa de la empresa está federada en la nube.

  A partir de septiembre de 2016, Microsoft ya no permite que el correo electrónico profesional o educativo se registre como cuentas de Microsoft. Para más información, consulte los siguientes materiales: [https://blogs.technet.microsoft.com/enterprisemobility/2016/09/15/cleaning-up-the-azure-ad-and-microsoft-account-overlap/](https://blogs.technet.microsoft.com/enterprisemobility/2016/09/15/cleaning-up-the-azure-ad-and-microsoft-account-overlap/).

  Si su organización no está federada en la nube, no podrá usar su dirección de correo electrónico profesional o educativa. Registre o cree una nueva dirección de correo electrónico y regístrela como cuenta de Microsoft.

### <a name="i-forgot-my-password-to-azure-ea-portal"></a>Olvidé mi contraseña en Azure EA Portal

Si olvidó su contraseña de Microsoft Live ID, vuelva a establecerla en [https://account.live.com/password/reset](https://account.live.com/password/reset).

Si olvidó su contraseña de trabajo, póngase en contacto con el administrador de TI de su empresa.

### <a name="i-have-a-valid-work-or-school-account-but-i-cant-add-it-to-the-ea-portal"></a>Tengo una cuenta profesional o educativa válida, pero no puedo agregarla a EA Portal

Si tiene una cuenta profesional o educativa en otro inquilino, cambie el nivel de autorización en la página Detalles de inscripción a "Work or School Account Cross Tenant" (Cuenta profesional o educativa entre inquilinos) y podrá agregar la cuenta.

## <a name="next-steps"></a>Pasos siguientes

- Los administradores del portal del Contrato Enterprise de Azure deben leer el artículo acerca de la [administración del portal del Contrato Enterprise de Azure](ea-portal-administration.md) para obtener información sobre las tareas administrativas comunes.
