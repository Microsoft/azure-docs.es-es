---
title: 'Instalación de paquetes de idioma en VM con Windows 10 en Windows Virtual Desktop: Azure'
description: Procedimiento para instalar paquetes de idioma para VM multisesión con Windows 10 en Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 04/03/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 542163511a1b4fc0acde9b44d73be6ffc042d5d3
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008770"
---
# <a name="install-language-packs"></a>Instalación de paquetes de idioma

Al configurar implementaciones de Windows Virtual Desktop internacionalmente, es conveniente asegurarse de que la implementación admita varios idiomas. Puede instalar paquetes de idioma en una imagen de máquina virtual (VM) multisesión con Windows 10 Enterprise para admitir tantos idiomas como su organización necesite. En este artículo se explica cómo instalar paquetes de idioma y capturar imágenes que permitan a los usuarios elegir sus propios idiomas para mostrar.

Obtenga más información sobre cómo implementar una VM en Azure en [Creación de una máquina virtual Windows en una zona de disponibilidad con Azure Portal](../virtual-machines/windows/create-portal-availability-zone.md).

>[!NOTE]
>Este artículo se aplica a las VM multisesión con Windows 10 Enterprise.

## <a name="install-a-language-pack"></a>Instalación de un paquete de idioma

Para crear una imagen de una VM con paquetes de idioma, primero debe instalar los paquetes de idioma en una máquina y capturar una imagen de esta.

Para instalar paquetes de idioma:

1. Inicie sesión como administrador.
2. Asegúrese de haber instalado todas las actualizaciones más recientes de Windows y Microsoft Store.
3. Vaya a **Configuración** > **Hora e idioma** > **Región**.
4. En **País o región**, seleccione su país o región de preferencia en el menú desplegable.
    En este ejemplo, vamos a seleccionar **Francia**, tal como se muestra en la siguiente captura de pantalla:

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la página Región. La región seleccionada actualmente es Francia.](media/region-page-france.png)

5. Después, seleccione **Idioma** y, a continuación, seleccione **Agregar un idioma**. Elija de la lista el idioma que desea instalar y, a continuación, seleccione **Siguiente**.
6. Cuando se abra la ventana **Install language features** (Instalar características de idioma), active la casilla **Install language pack and set as my Windows display language** (Instalar paquete de idioma y establecer como mi idioma para mostrar de Windows).
7. Seleccione **Instalar**.
8. Para agregar varios idiomas a la vez, seleccione **Agregar un idioma** y, a continuación, repita el proceso para agregar un idioma de los pasos 5 y 6. Repita este proceso para cada idioma que desee instalar. Sin embargo, solo puede establecer un idioma como idioma para mostrar a la vez.

    Vamos a ver una demostración visual rápida. En las imágenes siguientes se muestra cómo instalar los paquetes de idioma francés y holandés y, a continuación, establecer francés como idioma para mostrar.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la página Idioma al principio del proceso. El idioma para mostrar de Windows seleccionado es el inglés.](media/language-page-default.png)

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la ventana de selección de idioma. El usuario ha escrito "francés" en la barra de búsqueda para encontrar los paquetes de idioma francés.](media/select-language-french.png)

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la página Instalar características de idioma. Francés está seleccionado como idioma preferido. Las opciones seleccionadas son "Set as my display language" (Establecer como mi idioma para mostrar), "Instalar paquetes de idioma", "Reconocimiento de voz" y "Escritura a mano".](media/install-language-features.png)

    Una vez instalados los paquetes de idioma, debería ver los nombres de los paquetes de idioma en la lista de idiomas.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la página Idioma con los nuevos paquetes de idioma instalados. Los paquetes de idioma francés y holandés aparecen en la lista "Idiomas preferidos".](media/language-page-complete.png)

9. Si aparece una ventana donde se le pide que cierre la sesión, cierre la sesión y vuelva a iniciarla. El idioma para mostrar debería ser ahora el idioma seleccionado.

10.  Vaya a **Panel de control** > **Reloj y región** > **Región**.

11.  Cuando se abra la ventana **Región**, seleccione la pestaña **Administración** y, a continuación, seleccione **Copiar configuración**.

12.  Active las casillas **Pantalla de inicio de sesión y cuentas del sistema** y **Nuevas cuentas de usuario**.

13.  Seleccione **Aceptar**.

14.  Se abrirá una ventana que le indicará que reinicie la sesión. Seleccione **Reiniciar ahora**.

15.  Después de volver a iniciar sesión, vuelva a **Panel de control** > **Reloj y región** > **Región**.

16.  Seleccione la pestaña **Administración**.

17.  Seleccione **Cambiar la configuración regional del sistema**.

18. En el menú desplegable **Configuración regional del sistema actual**, seleccione el idioma de configuración regional que desee usar. Después, seleccione **Aceptar**.

19. Seleccione **Reiniciar ahora** para reiniciar la sesión una vez más.

¡Enhorabuena, ha instalado los paquetes de idioma!

Antes de continuar, asegúrese de que el sistema tenga instaladas las versiones más recientes de Windows y Microsoft Store.

## <a name="sysprep"></a>Sysprep

A continuación, tendrá que ejecutar Sysprep para preparar la máquina para el proceso de captura de imágenes.

Para ejecutar Sysprep en la máquina:

1. Abra PowerShell como administrador.
2. Ejecute el siguiente cmdlet para ir al directorio correcto:

    ```powershell
    cd Windows\System32\Sysprep
    ```

3. A continuación, ejecute el cmdlet siguiente:

    ```powershell
    .\sysprep.exe
    ```

4. Cuando se abra la ventana Herramientas de preparación del sistema, active la casilla **Generalizar**, vaya a **Opciones de apagado** y seleccione **Apagar** en el menú desplegable.

>[!NOTE]
>El proceso de Sysprep tardará unos minutos en finalizar. Cuando la VM se apague, se desconectará la sesión remota.

### <a name="resolve-sysprep-errors"></a>Resolución de errores de Sysprep

Si ve un mensaje de error durante el proceso de Sysprep, esto es lo que debe hacer:

1. Abra la **unidad C** y vaya a **Windows** > **System32 Sysprep** > **Panther** y, a continuación, abra el archivo **setuperr**.

   El texto del archivo de error le indicará que debe desinstalar un paquete de idioma específico, como se muestra en la siguiente imagen. Copie el nombre del paquete de idioma para el siguiente paso.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla del archivo setuperr. El texto con el nombre del paquete está resaltado en azul marino.](media/setuperr-package-name.png)

2. Abra una nueva ventana de PowerShell y ejecute el siguiente cmdlet con el nombre del paquete que copió en el paso 2 para quitar el paquete de idioma:

   ```powershell
   Remove-AppxPackage <package name>
   ```

3. Para asegurarse de que ha quitado el paquete, ejecute el cmdlet `Remove-AppxPackage` nuevamente. Si ha quitado correctamente el paquete, debería ver un mensaje que indica que no se encuentra el paquete que está intentando quitar.

4. Ejecute el cmdlet `sysprep.exe` nuevamente.

## <a name="capture-the-image"></a>Capturar la imagen

Ahora que el sistema está listo, puede capturar una imagen para que otros usuarios puedan empezar a usar VM basadas en el sistema sin tener que repetir el proceso de configuración.

Para capturar una imagen:

1. Vaya a Azure Portal y seleccione el nombre de la máquina que configuró en [Instalar un paquete de idioma](#install-a-language-pack) y [sysprep](#sysprep).

2. Seleccione **Capturar**.

3. Escriba un nombre para la imagen en el campo **Nombre** y asígnela al grupo de recursos mediante el menú desplegable **Grupo de recursos**, como se muestra en la siguiente imagen.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de la ventana Crear imagen. El nombre que el usuario ha dado a esta imagen de prueba es "vmwvd-image-fr" y la ha asignado al grupo de recursos "testwvdimagerg".](media/create-image.png)

4. Seleccione **Crear**.

5. Espere unos minutos para que finalice el proceso de captura. Cuando la imagen esté lista, debería ver un mensaje en el centro de notificaciones, que le avisa que la imagen se ha capturado.

Ahora puede implementar una VM con la nueva imagen. Al implementar la VM, asegúrese de seguir las instrucciones de [Creación de una máquina virtual Windows en una zona de disponibilidad con Azure Portal](../virtual-machines/windows/create-portal-availability-zone.md).

### <a name="how-to-change-display-language-for-standard-users"></a>Procedimiento para cambiar el idioma para mostrar para los usuarios estándar

Los usuarios estándar pueden cambiar el idioma para mostrar en sus VM.

Para cambiar el idioma para mostrar:

1. Vaya a **Configuración de idioma**. Si no sabe dónde está, puede escribir **Idioma** en la barra de búsqueda del menú Inicio.

2. En el menú desplegable Idioma para mostrar de Windows, seleccione el idioma que desea usar como idioma para mostrar.

3. Cierre la sesión y vuelva a iniciarla. El idioma para mostrar debería ser ahora el seleccionado en el paso 2.
