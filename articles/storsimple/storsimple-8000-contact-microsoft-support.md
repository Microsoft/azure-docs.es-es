---
title: Creación de un vale de soporte técnico para la serie 8000 de StorSimple | Microsoft Docs
description: Aprenda a registrar una solicitud de soporte técnico y a iniciar una sesión de soporte técnico en su dispositivo de la serie StorSimple 8000.
services: storsimple
documentationcenter: ''
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/09/2018
ms.author: alkohli;
ms.openlocfilehash: 77050ad37862394785cf348a242f585cc089ba26
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65606866"
---
# <a name="contact-microsoft-support"></a>Póngase en contacto con el servicio de soporte técnico de Microsoft

StorSimple Device Manager proporciona la posibilidad de **registrar una nueva solicitud de soporte técnico** en la hoja de resumen del servicio. Si tiene algún problema con la solución de StorSimple, puede crear una solicitud de servicio de soporte técnico. En una sesión en línea con su ingeniero de soporte técnico, también deberá iniciar una sesión de soporte técnico en su dispositivo StorSimple. Este artículo le enseñará a:

* Crear una solicitud de soporte.
* Administrar el ciclo de vida de una solicitud de soporte técnico desde dentro del portal.
* Iniciar una sesión de soporte en la interfaz de Windows PowerShell del dispositivo de StorSimple.

Revise la sección [Información y contratos de nivel de servicio de soporte de la serie StorSimple 8000](https://msdn.microsoft.com/library/mt433077.aspx) antes de crear una solicitud de soporte técnico.

## <a name="create-a-support-request"></a>Crear una solicitud de soporte

En función de su [plan de soporte técnico](https://azure.microsoft.com/support/plans/), puede crear vales de soporte para un problema en su dispositivo de StorSimple directamente desde la hoja de resumen del servicio StorSimple Device Manager. Lleve a cabo los siguientes pasos para crear una solicitud de soporte.

#### <a name="to-create-a-support-request"></a>Para crear una solicitud de soporte

1. Vaya al servicio StorSimple Device Manager. En la configuración de la hoja de resumen del servicio, vaya a la sección **Soporte técnico y solución de problemas** y haga clic en **Nueva solicitud de soporte técnico**.
     
    ![Ponerse en contacto con el soporte técnico de MS a través del nuevo portal](./media/storsimple-8000-contact-microsoft-support/contactsupport1.png)
   
2. En la hoja **Nueva solicitud de soporte técnico**, haga clic en **Datos básicos**. En la hoja **Aspectos básicos**, realice los pasos siguientes:
   1. En la lista desplegable **Tipo de problema**, seleccione **Técnico**.
   2. Se eligen automáticamente la **Suscripción** actual, el tipo de **Servicio** y el **Recurso** (servicio StorSimple Device Manager). 
   3. Seleccione un **plan de soporte técnico** en la lista desplegable si tiene varios planes asociados a la suscripción. Para habilitar el soporte técnico, debe contar con un plan de soporte técnico de pago.
   4. Haga clic en **Next**.

       ![Ponerse en contacto con el soporte técnico de MS a través del nuevo portal](./media/storsimple-8000-contact-microsoft-support/contactsupport2.png)

3. En la hoja **Nueva solicitud de soporte técnico**, seleccione **Step 2 Problem** (Paso 2 Problema). En la hoja **Problema**, siga estos pasos:
    
    1. Elija la **gravedad**.
    2. Especifique si el problema está relacionado con el dispositivo o con el servicio StorSimple Device Manager.
    3. Elija una **categoría** para este problema y especifique más **detalles** acerca del problema.
    4. Especifique la fecha y hora de inicio del problema.
    5. En **Carga de archivos**, haga clic en el icono de la carpeta para navegar a su paquete de soporte técnico.
    6. Seleccione **Compartir información de diagnóstico**.
    7. Haga clic en **Next**.

       ![Ponerse en contacto con el soporte técnico de MS a través del nuevo portal](./media/storsimple-8000-contact-microsoft-support/contactsupport3.png) 

4. En la hoja **Nueva solicitud de soporte**, haga clic en **Step 3 Contact information** (Paso 3 Información de contacto). En la hoja **Información de contacto**, realice los siguientes pasos:

   1. En **Opciones de contacto**, especifique el método de contacto preferido (teléfono o correo electrónico) y el idioma. El tiempo de respuesta se selecciona automáticamente en función del plan de suscripción.
   2. En la información de contacto, proporcione su nombre, correo electrónico, póngase en contacto con opcional, país o región. Seleccione la casilla de verificación **Guardar cambios de contacto para solicitudes futuras de soporte técnico** .
   3. Haga clic en **Create**(Crear).
   
       ![Ponerse en contacto con el soporte técnico de MS a través del nuevo portal](./media/storsimple-8000-contact-microsoft-support/contactsupport5.png)   

      El Soporte técnico de Microsoft usará esta información para ponerse en contacto con usted para solicitar más información y comunicar el diagnóstico y la resolución.
      Una vez que ha enviado la solicitud, un ingeniero de soporte se pondrá en contacto con usted tan pronto como sea posible para procesar su solicitud.

## <a name="manage-a-support-request"></a>Administración de una solicitud de soporte técnico

Después de crear una incidencia de soporte técnico, puede administrar el ciclo de vida de la incidencia desde el portal.

#### <a name="to-manage-your-support-requests"></a>Para administrar las solicitudes de soporte técnico

1. Para tener acceso a la página de ayuda y soporte técnico, vaya a **Examinar > Ayuda y soporte técnico**.

    ![Administrar solicitudes de soporte técnico](./media/storsimple-8000-contact-microsoft-support/managesupport1.png)

2. Se muestra una lista tabular de todas las solicitudes de soporte técnico en la hoja **Ayuda y soporte técnico**.

    ![Administrar solicitudes de soporte técnico](./media/storsimple-8000-contact-microsoft-support/managesupport2.png)

3. Seleccione una solicitud de soporte técnico y haga clic en ella. Puede ver el estado y los detalles de esta solicitud. Haga clic en **+ Nuevo mensaje** si desea realizar un seguimiento de esta solicitud.

    ![Administrar solicitudes de soporte técnico](./media/storsimple-8000-contact-microsoft-support/managesupport3.png)

## <a name="start-a-support-session-in-windows-powershell-for-storsimple"></a>Iniciar una sesión de soporte en Windows PowerShell para StorSimple

Para solucionar los problemas que pudiera experimentar con el dispositivo StorSimple, deberá ponerse en contacto con el equipo de soporte técnico de Microsoft. Es posible que el soporte técnico de Microsoft necesite utilizar una sesión de soporte para iniciar sesión en su dispositivo.

Lleve a cabo los siguientes pasos para iniciar una sesión de soporte.

#### <a name="to-start-a-support-session"></a>Para iniciar una sesión de soporte

1. Acceda al dispositivo directamente desde la consola serie o a través de una sesión de telnet desde un equipo remoto. Para ello, siga los pasos descritos en [Uso de PuTTy para conectarse a la consola serie del dispositivo](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console).
2. En la sesión que se abre, presione la tecla **Intro** para obtener un símbolo del sistema.
3. En el menú de la consola serie, seleccione la opción 1, **iniciar sesión con acceso completo**.
4. En el símbolo del sistema, escriba la siguiente contraseña:
   
    `Password1`
5. En el símbolo del sistema, escriba el siguiente comando:
   
    `Enable-HcsSupportAccess`
6. Aparecerá una cadena cifrada. Copie esta cadena en un editor de texto como el Bloc de notas.
7. Guarde esta cadena y envíela por correo electrónico al soporte técnico de Microsoft.

> [!IMPORTANT]
> Puede deshabilitar el acceso al soporte técnico ejecutando `Disable-HcsSupportAccess`. El dispositivo StorSimple también intenta deshabilitar el acceso al soporte técnico 8 horas después de iniciada la sesión. Es un procedimiento recomendado cambiar las credenciales de su dispositivo StorSimple después de una sesión de soporte técnico.


## <a name="next-steps"></a>Pasos siguientes

Aprenda a [diagnosticar y solucionar problemas relacionados un dispositivo de la serie StorSimple 8000](storsimple-8000-troubleshoot-deployment.md)
