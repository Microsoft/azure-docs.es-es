---
title: Azure AD Connect y privacidad del usuario | Microsoft Docs
description: En este documento se describe cómo obtener el cumplimiento con el GDPR en Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 05/21/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8f1d1bf35d03f0f7c4d11e4cc7e9b6f98f38d8ab
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279574"
---
# <a name="user-privacy-and-azure-ad-connect"></a>Privacidad del usuario y Azure AD Connect 

[!INCLUDE [Privacy](../../../includes/gdpr-intro-sentence.md)]

>[!NOTE] 
>Este artículo trata sobre Azure AD Connect y la privacidad del usuario.  Para más información sobre Azure AD Connect Health y la privacidad del usuario, consulte el artículo que se indica [aquí](reference-connect-health-user-privacy.md).

Puede mejorar la privacidad del usuario para las instalaciones de Azure AD Connect de dos maneras:

1.  Previa solicitud, extraer los datos de una persona y quitarlos de las instalaciones.
2.  Asegurarse de que ningún dato se conserva más de 48 horas.

El equipo de Azure AD Connect recomienda la segunda opción, ya que es mucho más fácil de implementar y mantener.

Un servidor de sincronización de Azure AD Connect almacena los datos de privacidad de usuario siguientes:
1.  Datos sobre una persona en la **base de datos de Azure AD Connect**
2.  Datos de los archivos del **registro de eventos de Windows** que pueden contener información sobre una persona
3.  Datos de los **archivos de registro de instalación de Azure AD Connect** que pueden contener información sobre una persona.

Los clientes de Azure AD Connect deben usar las siguientes directrices al eliminar datos de usuarios:
1.  Elimine el contenido de la carpeta que contiene los archivos de registro de instalación de Azure AD Connect con regularidad, al menos cada 48 horas.
2.  Este producto también puede crear registros de eventos.  Para más información sobre los registros de eventos, consulte la [documentación aquí](/windows/win32/wes/windows-event-log).

Los datos sobre una persona se eliminan automáticamente de la base de datos de Azure AD Connect cuando se quitan del sistema de origen de donde proceden. No es necesaria ninguna acción específica por parte de los administradores para el cumplimiento del GDPR.  Sin embargo, es necesario que los datos de Azure AD Connect se sincronicen con el origen de datos al menos cada dos días.

## <a name="delete-the-azure-ad-connect-installation-log-file-folder-contents"></a>Eliminación del contenido de la carpeta de archivos de registro de instalación de Azure AD Connect
Compruebe y elimine periódicamente el contenido de la carpeta **c:\programdata\aadconnect**, excepto el archivo **PersistedState.Xml**. Este archivo mantiene el estado de la instalación anterior de Azure AD Connect y se usa cuando se realiza una instalación de actualizaciones. Este archivo no contiene ningún dato sobre una persona y no debe eliminarse.

>[!IMPORTANT]
>No elimine el archivo PersistedState.xml.  Este archivo no contiene ninguna información del usuario y mantiene el estado de la instalación anterior.

Estos archivos se pueden revisar y eliminar mediante el Explorador de Windows, o bien puede usar un script como el siguiente para realizar las acciones necesarias:


```
$Files = ((Get-childitem -Path "$env:programdata\aadconnect" -Recurse).VersionInfo).FileName
Foreach ($file in $files) {
If ($File.ToUpper() -ne "$env:programdata\aadconnect\PERSISTEDSTATE.XML".toupper()) # Do not delete this file
    {Remove-Item -Path $File -Force}
    } 
```

### <a name="schedule-this-script-to-run-every-48-hours"></a>Programación de este script para ejecutarse cada 48 horas
Use los pasos siguientes para programar que el script se ejecute cada 48 horas.

1.  Guarde el script en un archivo con la extensión **&#46;PS1** y luego abra el Panel de control y haga clic en **Sistemas y seguridad**.
    ![Sistema](./media/reference-connect-user-privacy/gdpr2.png)

2.  En el encabezado Herramientas administrativas, haga clic en **Programar tareas**.
    ![Task](./media/reference-connect-user-privacy/gdpr3.png)
3.  En el Programador de tareas, haga clic con el botón derecho en **Biblioteca del Programador de tareas** y haga clic en **Crear tarea básica...**
4.  Escriba el nombre de la nueva tarea y haga clic en **Siguiente**.
5.  Seleccione **Diariamente** para el desencadenador de tareas y haga clic en **Siguiente**.
6.  Establezca la repetición en **2 días** y haga clic en **Siguiente**.
7.  Seleccione **Iniciar un programa** como la acción y haga clic en **Siguiente**.
8.  En el cuadro Programa o script, escriba **PowerShell** y, en el cuadro **Agregar argumentos (opcional)** , escriba la ruta de acceso completa al script que creó anteriormente y, luego, haga clic en **Siguiente**.
9.  En la siguiente pantalla se muestra un resumen de la tarea que se dispone a crear. Compruebe los valores y haga clic en **Finalizar** para crear la tarea.



## <a name="next-steps"></a>Pasos siguientes
* [Revisar la directiva de privacidad de Microsoft en Trust Center](https://www.microsoft.com/trustcenter)
* [Azure AD Connect Health y privacidad del usuario](reference-connect-health-user-privacy.md)