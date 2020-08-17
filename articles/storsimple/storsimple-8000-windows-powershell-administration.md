---
title: PowerShell para la administración de dispositivos StorSimple
description: Obtenga más información acerca de cómo usar Windows PowerShell para que StorSimple administre su dispositivo StorSimple.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 01/09/2018
ms.author: alkohli
ms.openlocfilehash: bba3666fbce6a8ea591654d1abdad319f1e0857c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86999524"
---
# <a name="use-windows-powershell-for-storsimple-to-administer-your-device"></a>Usar Windows PowerShell para StorSimple para administrar su dispositivos

## <a name="overview"></a>Información general

Windows PowerShell para StorSimple es una interfaz de línea de comandos que puede utilizar para administrar el dispositivo de StorSimple de Microsoft Azure. Como sugiere su nombre, es una interfaz de línea de comandos basada en Windows PowerShell, que se compila en un espacio de ejecución restringido. Desde la perspectiva del usuario en la línea de comandos, un espacio de ejecución restringido aparece como una versión de Windows PowerShell restringida. Además de conservar algunas de las capacidades básicas de Windows PowerShell, esta interfaz tiene cmdlets adicionales dedicados orientados a administrar el dispositivo StorSimple de Microsoft Azure.

Este artículo describe las características de Windows PowerShell para StorSimple, incluido cómo puede conectarse a esta interfaz y vínculos a procedimientos detallados de flujos de trabajo que se pueden realizar mediante esta interfaz. Los flujos de trabajo incluyen cómo registrar el dispositivo, configurar la interfaz de red en el dispositivo, instalar las actualizaciones que requieren el dispositivo en modo de mantenimiento, cambiar el estado del dispositivo y solucionar los problemas que puede experimentar.

Después de leer este artículo, aprenderá a:

* Conectar el dispositivo StorSimple con Windows PowerShell para StorSimple.
* Administrar el dispositivo StorSimple con Windows PowerShell para StorSimple.
* Obtener ayuda de Windows PowerShell para StorSimple.

> [!NOTE]
> * Los cmdlets de Windows PowerShell para StorSimple permiten administrar el dispositivo StorSimple desde una consola en serie o de forma remota a través de la conexión remota de Windows PowerShell. Para obtener más información acerca de cada uno de los cmdlets individuales que pueden utilizarse en esta interfaz, vaya a [Referencia de cmdlets de Windows PowerShell para StorSimple](https://technet.microsoft.com/library/dn688168.aspx).
> * Los cmdlets de Azure PowerShell StorSimple son un conjunto de cmdlets de Windows PowerShell que permiten automatizar las tareas de nivel de servicio y de migración desde la línea de comandos. Para obtener más información sobre los cmdlets de Azure PowerShell, consulte la [Referencia de los cmdlets de Azure StorSimple](/powershell/module/servicemanagement/azure.service/?view=azuresmps-4.0.0&viewFallbackFrom=azuresmps-3.7.0#azure).


Puede tener acceso a Windows PowerShell para StorSimple mediante uno de los métodos siguientes:

* [Conectarse a la consola en serie del dispositivo StorSimple](#connect-to-windows-powershell-for-storsimple-via-the-device-serial-console)
* [Conectarse de modo remoto a StorSimple mediante Windows PowerShell](#connect-remotely-to-storsimple-using-windows-powershell-for-storsimple)

## <a name="connect-to-windows-powershell-for-storsimple-via-the-device-serial-console"></a>Conéctese a Windows PowerShell para StorSimple mediante la consola serie del dispositivo

Para conectarse a Windows PowerShell para StorSimple, puede [descargar PuTTY](https://www.putty.org/) o software de emulación de terminales similar. Necesitará configurar PuTTY específicamente para acceder al dispositivo de StorSimple de Microsoft Azure. Los temas siguientes contienen pasos detallados acerca de cómo configurar PuTTy y conectarse al dispositivo. También se explican varias opciones de menú de la consola en serie.

### <a name="putty-settings"></a>Configuración de PuTTY

Asegúrese de usar la siguiente configuración de PuTTY para conectarse a la interfaz de Windows PowerShell desde la consola en serie.

#### <a name="to-configure-putty"></a>Para configurar PuTTY

1. En el cuadro de diálogo **Reconfiguration** (Reconfiguración) de PuTTY, en el panel **Category** (Categoría), seleccione **Keyboard** (Teclado).
2. Asegúrese de que estén activadas las siguientes opciones (constituyen la configuración predeterminada cuando se inicia una nueva sesión).
   
   | Elemento de teclado | Seleccionar |
   | --- | --- |
   | Tecla retroceso |Control-? (127) |
   | Teclas de inicio y fin |Estándar |
   | Teclas de función y teclado numérico |ESC[n~ |
   | Estado inicial de las teclas de dirección |Normal |
   | Estado inicial del teclado numérico |Normal |
   | Habilitar las características de teclado adicionales |Control-Alt es diferente de AltGr |
   
    ![Configuración de Putty compatible](./media/storsimple-windows-powershell-administration/IC740877.png)
3. Haga clic en **Aplicar**.
4. En el panel **Category** (Categoría) seleccione **Translation** (Traducción).
5. En el cuadro de lista **Remote character set** (Conjunto de caracteres remotos), seleccione **UTF-8**.
6. En **Handling of line drawing characters** (Control de caracteres de dibujo de líneas), seleccione **Use Unicode line drawing code points** (Usar puntos de código de dibujo de líneas Unicode). La siguiente captura de pantalla muestra las selecciones de PuTTY correctas.
   
    ![Configuración de Putty de UTF](./media/storsimple-windows-powershell-administration/IC740878.png)
7. Haga clic en **Aplicar**.

Ahora puede usar PuTTY para conectarse a la consola en serie del dispositivo realizando los pasos siguientes.

[!INCLUDE [storsimple-use-putty](../../includes/storsimple-use-putty.md)]

### <a name="about-the-serial-console"></a>Información acerca de la consola serie

Cuando tiene acceso a la interfaz de Windows PowerShell del dispositivo StorSimple a través de la consola en serie, un mensaje de banner se presenta, seguido de las opciones de menú.

El mensaje de banner contiene información básica de dispositivo StorSimple como el modelo, el nombre, la versión del software instalado y el estado del controlador al que obtiene acceso. La imagen siguiente muestra un ejemplo de un mensaje de banner.

![Mensaje de banner en serie](./media/storsimple-windows-powershell-administration/IC741098.png)

> [!IMPORTANT]
> Puede usar el mensaje de banner para identificar si el controlador que está conectado es _Activo_ o _Pasivo_.

La imagen siguiente muestra las distintas opciones de espacio de ejecución que están disponibles en el menú de la consola en serie.

![Registrar el dispositivo 2](./media/storsimple-windows-powershell-administration/IC740906.png)

Puede elegir entre las siguientes opciones:

1. **Log in with full access** (Iniciar sesión con acceso completo): esta opción le permite conectarse (con las credenciales apropiadas) al espacio de ejecución **SSAdminConsole** en el controlador local. (El controlador local es el controlador al que tiene acceso actualmente a través de la consola en serie del dispositivo StorSimple). Esta opción también puede utilizarse para permitir que el servicio técnico de Microsoft tenga acceso al espacio de ejecución no restringido (sesión de soporte técnico) para solucionar los problemas de dispositivo posibles. Después de utilizar la opción 1 para iniciar sesión, puede permitir que el ingeniero del servicio técnico de Microsoft tenga acceso a un espacio de ejecución sin restricciones mediante la ejecución de un cmdlet específico. Para obtener más información, consulte [Iniciar una sesión de soporte](storsimple-8000-contact-microsoft-support.md#start-a-support-session-in-windows-powershell-for-storsimple).
   
2. **Log in to peer controller with full access** (Iniciar sesión en el controlador del mismo nivel con acceso completo): esta opción es igual a la opción 1, excepto que permite la conexión (con las credenciales apropiadas) al espacio de ejecución **SSAdminConsole** en el controlador del mismo nivel. Dado que el dispositivo StorSimple es un dispositivo de alta disponibilidad con dos controladores en una configuración activo/pasivo, el mismo nivel hace referencia al otro controlador en el dispositivo al que accede a través de la consola en serie).
   Similar a la opción 1, esta opción puede utilizarse para permitir que el servicio técnico de Microsoft tenga acceso a un espacio de ejecución sin restricciones en un controlador del mismo nivel.

3. **Conectarse con acceso limitado** : esta opción se utiliza para tener acceso a la interfaz de Windows PowerShell en modo limitado. No se le solicitarán las credenciales de acceso. Esta opción se conecta a un espacio de ejecución más restringido en comparación con las opciones 1 y 2.  Algunas de las tareas que están disponibles por medio de la opción 1 que **no se pueden* realizar en este espacio de ejecución son:
   
   * Restablecer la configuración de fábrica
   * Cambiar la contraseña
   * Habilitar o deshabilitar el acceso de soporte
   * Aplicación de actualizaciones
   * Instalar revisiones

     > [!NOTE]
     > Esta es la opción preferida si ha olvidado la contraseña del administrador de dispositivos y no se puede conectar a través de la opción 1 o 2.

4. **Cambiar idioma** : esta opción le permite cambiar el idioma para mostrar en la interfaz de Windows PowerShell. Los idiomas admitidos son inglés, japonés, ruso, francés, coreano de Corea del Sur, español, italiano, alemán, chino y portugués.

## <a name="connect-remotely-to-storsimple-using-windows-powershell-for-storsimple"></a>Conectarse remotamente a StorSimple con Windows PowerShell para StorSimple

Puede usar la conexión remota de Windows PowerShell para conectarse al dispositivo StorSimple. Cuando se conecta de este modo, no verá un menú. (Verá un menú solo si utiliza la consola en serie del dispositivo para conectarse. La conexión remota remite directamente al equivalente de "opción 1 – acceso total" en la consola serie). Con la conexión remota de Windows PowerShell, se conecta a un espacio de ejecución específico. También puede especificar el idioma de visualización.

El idioma de visualización es independiente del idioma que se configura mediante la opción **Cambiar idioma** en el menú de la consola en serie. Remote PowerShell seleccionará automáticamente la configuración regional del dispositivo desde el que se conecta si no es especifica uno.

> [!NOTE]
> Si trabaja con hosts virtuales de Microsoft Azure y dispositivos StorSimple Cloud Appliance, puede usar la conexión remota de Windows PowerShell y el host virtual para conectarse al dispositivo virtual. Si ha configurado una ubicación de recurso compartido en el host en el que se va a guardar la información de la sesión de Windows PowerShell, debe tener en cuenta que la entidad de seguridad _Todos_ incluye solo los usuarios autenticados. Por lo tanto, si ha configurado el recurso compartido para permitir el acceso a _todos_ los usuarios y se conecta sin especificar las credenciales, se usará la entidad de seguridad anónima no autenticada y aparecerá un error. Para solucionar este problema, en el recurso compartido de host, debe habilitar la cuenta de invitado y, a continuación, proporcionar al invitado acceso de cuenta completa al recurso compartido o se deben especificar credenciales válidas junto con el cmdlet de Windows PowerShell.


Puede utilizar HTTP o HTTPS para conectarse a través de la conexión remota de Windows PowerShell. Siga las instrucciones de los siguientes tutoriales:

* [Conectarse de forma remota mediante HTTP](storsimple-8000-remote-connect.md#connect-through-http)
* [Conectarse de forma remota mediante HTTPS](storsimple-8000-remote-connect.md#connect-through-https)

## <a name="connection-security-considerations"></a>Consideraciones de seguridad de conexión

Cuando decida cómo conectarse a Windows PowerShell para StorSimple, tenga en cuenta lo siguiente:

* Es seguro conectarse directamente a la consola en serie del dispositivo, pero la conexión a la consola en serie a través conmutadores de red no lo es. Tenga presente los riesgos de seguridad al conectarse al dispositivo en serie a través de los conmutadores de red.
* La conexión a través de una sesión HTTP puede ofrecer más seguridad que conectarse a través de la consola en serie a través de la red. Aunque este no es el método más seguro, es aceptable en redes de confianza.
* La conexión a través de una sesión HTTPS es la opción más segura y la recomendada.

## <a name="administer-your-storsimple-device-using-windows-powershell-for-storsimple"></a>Administrar el dispositivo StorSimple con Windows PowerShell para StorSimple

En la siguiente tabla se muestra un resumen de todas las tareas comunes de administración y flujos de trabajo complejos que pueden llevarse a cabo en la interfaz de Windows PowerShell del dispositivo de StorSimple. Para obtener más información sobre cada flujo de trabajo, haga clic en la entrada adecuada en la tabla.

#### <a name="windows-powershell-for-storsimple-workflows"></a>Flujos de trabajo de Windows PowerShell para StorSimple

| Si desea hacer esto... | Utilice este procedimiento. |
| --- | --- |
| Registrar el dispositivo |[Configurar y registrar el dispositivo con Windows PowerShell para StorSimple](storsimple-8000-deployment-walkthrough-u2.md#step-3-configure-and-register-the-device-through-windows-powershell-for-storsimple) |
| Configurar el proxy web</br>Ver la configuración de proxy web |[Configurar el proxy web para el dispositivo StorSimple](storsimple-8000-configure-web-proxy.md) |
| Modificar la configuración de interfaz de red de DATA 0 en el dispositivo |[Modificar la configuración de interfaz de red de DATA 0 en el dispositivo](storsimple-8000-modify-data-0.md) |
| Detener un controlador </br> Reiniciar o apagar un solo controlador </br> Apagar un dispositivo</br>Restablecer el dispositivo a los valores predeterminados de fábrica. |[Administrar controladores de dispositivo](storsimple-8000-manage-device-controller.md) |
| Instalar revisiones y actualizaciones de modo de mantenimiento |[Actualizar su dispositivo](storsimple-update-device.md) |
| Acceder al modo de mantenimiento </br>Salir del modo de mantenimiento |[Modos del dispositivo StorSimple](storsimple-8000-device-modes.md) |
| Crear un paquete de soporte</br>Descifrar y editar un paquete de soporte |[Crear y administrar paquetes de soporte técnico](storsimple-8000-create-manage-support-package.md) |
| Iniciar una sesión de soporte</br> |[Iniciar una sesión de soporte en Windows PowerShell para StorSimple](storsimple-8000-create-manage-support-package.md#create-a-support-package) |

## <a name="get-help-in-windows-powershell-for-storsimple"></a>Obtener ayuda de Windows PowerShell para StorSimple

En Windows PowerShell para StorSimple, la Ayuda del cmdlet está disponible. Una versión actualizada en línea de esta Ayuda también está disponible, que puede utilizar para actualizar la Ayuda en el sistema.

Obtener ayuda en esta interfaz es similar a la de Windows PowerShell y funcionará la mayoría de los cmdlets relacionados con la Ayuda. Puede encontrar ayuda para Windows PowerShell en línea: [Microsoft.PowerShell.Core](/powershell/module/Microsoft.PowerShell.Core/).

La siguiente es una breve descripción de los tipos de ayuda para esta interfaz de Windows PowerShell, incluso cómo actualizar la Ayuda.

### <a name="to-get-help-for-a-cmdlet"></a>Obtención de ayuda sobre un cmdlet

* Para obtener ayuda para cualquier cmdlet o función, utilice el siguiente comando: `Get-Help <cmdlet-name>`
* Para obtener ayuda en línea para cualquier cmdlet, use el cmdlet anterior con el parámetro `-Online`: `Get-Help <cmdlet-name> -Online`
* Para obtener ayuda completa, puede usar el parámetro `–Full` y, para ver ejemplos, use el parámetro `–Examples`.

### <a name="to-update-help"></a>Para actualizar la Ayuda

Puede actualizar fácilmente la Ayuda en la interfaz de Windows PowerShell. Realice los pasos siguientes para actualizar la Ayuda en el sistema.

#### <a name="to-update-cmdlet-help"></a>Para actualizar la Ayuda del cmdlet
1. Inicie Windows PowerShell con la opción **Ejecutar como administrador** .
2. En el símbolo del sistema, escriba: `Update-Help`
3. Se instalarán los archivos de Ayuda actualizados.
4. Una vez instalados los archivos de ayuda, escriba: `Get-Help Get-Command`. Esto mostrará una lista de cmdlets para que la Ayuda está disponible.

> [!NOTE]
> Para obtener una lista de todos los cmdlets disponibles en un espacio de ejecución, inicie sesión en la opción de menú correspondiente y ejecute el cmdlet `Get-Command`.


## <a name="next-steps"></a>Pasos siguientes

Si experimenta problemas con el dispositivo StorSimple al realizar uno de los flujos de trabajo anteriores, vea [Herramientas para solucionar problemas en implementaciones de StorSimple](storsimple-8000-troubleshoot-deployment.md#tools-for-troubleshooting-storsimple-deployments).

