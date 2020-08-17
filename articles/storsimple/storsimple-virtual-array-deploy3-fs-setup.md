---
title: Configuración de una matriz virtual de StorSimple como servidor de archivos | Microsoft Docs
description: En este tercer tutorial sobre la implementación de matrices virtuales StorSimple se enseña a configurar un dispositivo virtual como servidor de archivos.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: f609f6ff-0927-48bb-a68a-6d8985d2fe34
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 07/25/2019
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 4e544d135883d0c936b3f23b3e50e385268c992c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87006303"
---
# <a name="deploy-storsimple-virtual-array---set-up-as-file-server-via-azure-portal"></a>Implementación de una matriz virtual de StorSimple: configurar un servidor de archivos mediante Azure Portal
![Diagrama que muestra los pasos necesarios para implementar una matriz virtual. La primera parte del tercer paso está etiquetada como "Configurar como servidor de archivos" y aparece resaltada.](./media/storsimple-virtual-array-deploy3-fs-setup/fileserver4.png)

## <a name="introduction"></a>Introducción

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

En este artículo se describe cómo realizar la instalación inicial, registrar el servidor de archivos de StorSimple, completar la instalación del dispositivo y crear y conectarse a recursos compartidos de SMB. Este es el último artículo de la serie de tutoriales de implementación necesarios para implementar completamente la matriz virtual como servidor de archivos o servidor iSCSI.

El proceso de instalación y configuración puede tardar unos 10 minutos en completarse. La información de este artículo solo se aplica a la implementación de StorSimple Virtual Array. Para la implementación de dispositivos StorSimple 8000 Series, vaya a: [Implementación del dispositivo StorSimple local (Update 3 u otra posterior)](storsimple-deployment-walkthrough-u2.md).

## <a name="setup-prerequisites"></a>Requisitos previos de instalación
Antes de instalar y configurar StorSimple Virtual Array, asegúrese de que:

* Ha aprovisionado una matriz virtual y se ha conectado a ella según se detalla en [Implementación de una matriz virtual de StorSimple: aprovisionamiento de una matriz virtual en Hyper-V](storsimple-virtual-array-deploy2-provision-hyperv.md) o en [Implementación de una matriz virtual de StorSimple: aprovisionamiento de una matriz virtual en VMware](storsimple-virtual-array-deploy2-provision-vmware.md).
* Tiene la clave de registro del servicio del administrador de dispositivos de StorSimple que creó para administrar instancias de StorSimple Virtual Array. Para más información, consulte [Paso 2: Obtener la clave de registro del servicio](storsimple-virtual-array-deploy1-portal-prep.md#step-2-get-the-service-registration-key) de StorSimple Virtual Array.
* Si este es la segunda o subsiguiente matriz virtual que registra en un servicio StorSimple Device Manager existente, debe tener la clave de cifrado de datos del servicio. Esta clave se generó cuando el primer dispositivo se registró correctamente en este servicio. Si perdió esta clave, consulte la sección [Obtener la clave de cifrado de datos del servicio](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key) para la matriz virtual de StorSimple.

## <a name="step-by-step-setup"></a>Configuración paso a paso
Use las siguientes instrucciones detalladas para instalar y configurar StorSimple Virtual Array.

## <a name="step-1-complete-the-local-web-ui-setup-and-register-your-device"></a>Paso 1: Completar la configuración de la interfaz de usuario web local y registrar el dispositivo
#### <a name="to-complete-the-setup-and-register-the-device"></a>Para completar la instalación y registrar el dispositivo
1. Abra una ventana del explorador y conéctese a la interfaz de usuario web local. Escriba:
   
   `https://<ip-address of network interface>`
   
   Use la dirección URL de conexión que anotó en el paso anterior. Ve un error que indica que hay un problema con el certificado de seguridad del sitio web. Haga clic en **Continue to this webpage**(Continuar a esta página web).
   
   ![Captura de pantalla de una ventana del explorador que informa de un problema con el certificado de seguridad. Pueden verse dos vínculos, uno para el sitio web y otro para la página principal del usuario.](./media/storsimple-virtual-array-deploy3-fs-setup/image2.png)
2. Inicie sesión en la interfaz de usuario web de la matriz virtual como **StorSimpleAdmin**. Escriba la contraseña del administrador de dispositivos que cambió en el Paso 3: Iniciar la matriz virtual en [Implementación de una matriz virtual de StorSimple: aprovisionar una matriz virtual en Hyper-V](storsimple-virtual-array-deploy2-provision-hyperv.md) o en [Implementación una matriz virtual de StorSimple: aprovisionar una matriz virtual en VMware](storsimple-virtual-array-deploy2-provision-vmware.md).
   
   ![Captura de pantalla de la página de inicio de sesión de StorSimple. Se puede ver el nombre de usuario de StorSimpleAdmin y un cuadro de contraseña rellenado con caracteres indeterminados.](./media/storsimple-virtual-array-deploy3-fs-setup/image3.png)
3. Lo llevará a la página **principal**. En esta página se describen los distintos parámetros requeridos para configurar y registrar la matriz virtual en el servicio StorSimple Device Manager. Las opciones **Configuración de red**, **Configuración de proxy web** y **Configuración horaria** son opcionales. Los únicos parámetros obligatorios son **Configuración del dispositivo** y **Configuración de la nube**.
   
   ![Captura de pantalla de la página principal. El texto indica que el dispositivo no está configurado. Se pueden ver vínculos a varios tipos diferentes de configuraciones.](./media/storsimple-virtual-array-deploy3-fs-setup/image4.png)
4. En la página **Configuración de red** en **Interfaces de red**, DATA 0 se configurará automáticamente. Cada interfaz de red se establece de forma predeterminada para obtener la dirección IP automáticamente (DHCP). Por lo tanto, se asignan automáticamente una dirección IP, una subred y una puerta de enlace (tanto para IPv4 como para IPv6).
   
   ![Captura de pantalla de la página Network settings (Configuración de red) que muestra las direcciones IP configuradas para varias versiones del protocolo de Internet.](./media/storsimple-virtual-array-deploy3-fs-setup/image5.png)
   
   Si agrega más de una interfaz de red durante el aprovisionamiento del dispositivo, se pueden configurar aquí. Tenga en cuenta que puede configurar la interfaz de red como IPv4 únicamente o como IPv4 e IPv6. No se admiten las configuraciones de solo IPv6.
5. Los servidores DNS son necesarios porque se utilizan cuando el dispositivo intenta comunicarse con sus proveedores de servicios de almacenamiento en la nube o para resolver el dispositivo por nombre cuando se configura como servidor de archivos. En la página **Configuración de red** en **Servidores DNS**:
   
   1. Se configuran automáticamente un servidor DNS principal y uno secundario. Si decide configurar direcciones IP estáticas, puede especificar servidores DNS. Para lograr la alta disponibilidad, se recomienda que configure un servidor DNS principal y uno secundario.
   2. Haga clic en **Aplicar** para aplicar y validar la configuración de red.
6. En la página **Device settings** (Configuración de dispositivo):
   
   1. Asigne un **nombre** exclusivo al dispositivo. Este nombre puede tener de 1 a 15 caracteres y puede contener letras, números y guiones.
   2. Haga clic en el icono **Servidor de archivos**:::image type="icon" source="./media/storsimple-virtual-array-deploy3-fs-setup/image6.png"::: del **tipo** de dispositivo que crea. Un servidor de archivos le permitirá crear carpetas compartidas.
   3. Como el dispositivo es un servidor de archivos, debe unir el dispositivo a un dominio. Escriba un valor en **nombre de dominio**.
   4. Haga clic en **Aplicar**.
7. Aparece un cuadro de diálogo. Escriba las credenciales del dominio en el formato especificado. Haga clic en el icono de marca de verificación. Se comprueban las credenciales del dominio. Si las credenciales no son correctas, aparece un mensaje de error.
   
   ![Captura de pantalla que muestra un cuadro de diálogo con el nombre de usuario y la contraseña rellenados.](./media/storsimple-virtual-array-deploy3-fs-setup/image7.png)
8. Haga clic en **Aplicar**. Esto aplicará y validará la configuración de dispositivo.
   
   ![Captura de pantalla de la página Device settings (Configuración de dispositivo). Están rellenados el nombre del dispositivo y el nombre de dominio.](./media/storsimple-virtual-array-deploy3-fs-setup/image8.png)
   
   > [!NOTE]
   > Asegúrese de que su matriz virtual está en su propia unidad organizativa (UO) de Active Directory y de que no se le aplica ni hereda ningún objeto de directiva de grupo (GPO). La directiva de grupo puede instalar aplicaciones, como software antivirus, en la matriz virtual de StorSimple. La instalación de software adicional no se admite y puede provocar daños en los datos. 
   > 
   > 
9. Configure el servidor proxy web (de manera opcional). Aunque la configuración del proxy web es opcional, tenga en cuenta que, si usa un proxy web, solo puede configurarlo aquí.
   
   ![Captura de pantalla de la página Web proxy settings (Configuración de proxy web). La opción Enable web proxy (Habilitar proxy web) está desactivada y la autenticación está establecida en None (Ninguna). No está configurado ningún otro valor.](./media/storsimple-virtual-array-deploy3-fs-setup/image9.png)
   
   En la página **Proxy web** :
   
   1. Proporcione la **URL de proxy web** en este formato: *http://&lt;dirección IP o FQDN de host&gt;:Número de puerto*. Tenga en cuenta que no se admiten direcciones URL HTTPS.
   2. Especifique **Autenticación** como **Básica** o **Ninguna**.
   3. Si utiliza autenticación, también debe proporcionar un **Nombre de usuario** y una **Contraseña**.
   4. Haga clic en **Aplicar**. Esto validará y aplicará los parámetros de proxy web configurados.
10. (Opcionalmente) configure las opciones de hora para el dispositivo, como la zona horaria y los servidores NTP principal y secundario. Se requieren servidores NTP, ya que el dispositivo debe sincronizar la hora para que pueda autenticarse con los proveedores de servicios en la nube.
    
    ![Captura de pantalla de la página Time settings (Configuración horaria). Están rellenados la zona horaria y el servidor NTP principal. El servidor NTP secundario está en blanco.](./media/storsimple-virtual-array-deploy3-fs-setup/image10.png)
    
    En la página **Time settings** (Configuración de hora):
    
    1. En la lista desplegable, seleccione la **zona horaria** según la ubicación geográfica en la que se va a implementar el dispositivo. La zona horaria predeterminada para el dispositivo es la hora del Pacífico (PST). El dispositivo usará esta zona horaria para todas las operaciones programadas.
    2. Especifique un **servidor NTP principal** para el dispositivo o acepte el valor predeterminado de time.windows.com. Asegúrese de que su red permite que el tráfico NTP pase del centro de datos a Internet.
    3. Opcionalmente, especifique un **servidor NTP secundario** para el dispositivo.
    4. Haga clic en **Aplicar**. Esto validará y aplicará los parámetros de hora configurados.
11. Configure las opciones de nube para el dispositivo. En este paso, completará la configuración del dispositivo local y, después, registrará el dispositivo en el servicio StorSimple Device Manager.
    
    1. Escriba la **Clave de registro del servicio** que obtuvo en el [Paso 2: Obtener la clave de registro del servicio](storsimple-virtual-array-deploy1-portal-prep.md#step-2-get-the-service-registration-key) de StorSimple Virtual Array.
    2. Si es el primer dispositivo que registra en este servicio, aparecerá la **clave de cifrado de datos del servicio**. Copie esta clave y guárdela en un lugar seguro, ya que se necesita, junto con la clave de registro del servicio, para registrar dispositivos adicionales en el servicio StorSimple Device Manager. 
       
       Si no es el primer dispositivo que registra en este servicio, será preciso que especifique la clave de cifrado de datos del servicio. Para más información, consulte [Obtener la clave de cifrado de los datos del servicio](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key) en la interfaz de usuario web local.
    3. Haga clic en **Registrar**. Se reiniciará el dispositivo. Debe esperar de 2 a 3 minutos antes de que el dispositivo se registre correctamente. Una vez que se haya reiniciado el dispositivo, irá a la página de inicio de sesión.
       
       ![Captura de pantalla de la página Cloud settings (Configuración de la nube). Los cuadros de clave de registro y clave de cifrado están rellenados, pero los valores están ocultos.](./media/storsimple-virtual-array-deploy3-fs-setup/image13.png)
12. Vuelva a Azure Portal. Vaya a **Todos los recursos** y busque su servicio StorSimple Device Manager.
    
    ![Captura de pantalla de la página Todos los recursos de Azure Portal. Se resalta el servicio Administrador de dispositivos.](./media/storsimple-virtual-array-deploy3-fs-setup/searchdevicemanagerservice1.png) 
13. En la lista filtrada, seleccione su servicio StorSimple Device Manager y, a continuación, navegue hasta **Administración > Dispositivos**. En la hoja **Dispositivos**, compruebe que el dispositivo se ha conectado correctamente al servicio y tiene el estado **Listo para configurar**.
    
    ![Configurar un servidor de archivos](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs2m.png)

## <a name="step-2-configure-the-device-as-file-server"></a>Paso 2: Configurar el dispositivo como servidor de archivos
Siga estos pasos en [Azure Portal](https://portal.azure.com/) para completar la configuración requerida del dispositivo.

#### <a name="to-configure-the-device-as-file-server"></a>Para configurar el dispositivo como servidor de archivos
1. Vaya al servicio StorSimple Device Manager y, después, a **Administración > Dispositivos**. En la hoja **Dispositivos**, seleccione el dispositivo que acaba de crear. Este dispositivo aparecerá como **Listo para configurar**.
   
   ![Configurar un servidor de archivos](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs2m.png) 
2. Haga clic en el dispositivo y verá un mensaje del banner que indica que el dispositivo está listo para configurar.
   
    ![Configurar un servidor de archivos](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs3m.png)
3. Haga clic en **Configurar** en la barra de comandos. Se abrirá la hoja **Configurar**. En **dicha hoja**, realice estas acciones:
   
   1. El nombre del servidor de archivos se rellena automáticamente.
    
   2. Asegúrese de que el cifrado del almacenamiento en la nube está establecido en **Habilitado**. Así se cifrarán todas los datos que se envían a la nube. 
    
   3. Para el cifrado se usa una clave AES de 256 bits con la clave definida por el usuario. Especifique una clave de 32 caracteres y vuelva a escribirla para confirmarla. Regístrela en cualquier aplicación de administración de claves, por si la necesita en el futuro.
    
   4. Haga clic en **Configurar los valores obligatorios** para especificar las credenciales de la cuenta de almacenamiento que va a usar en su dispositivo. Haga clic en **Agregar nuevo** si no hay ninguna credencial de la cuenta de almacenamiento configurada. **Asegúrese de que la cuenta de almacenamiento que usa es compatible con los blobs en bloques. No se admiten blobs en páginas.** Más información sobre [blobs en bloques y blobs en páginas](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs).
   
      ![Configurar un servidor de archivos](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs6m.png) 
4. En la hoja **Agregar credenciales de cuenta de almacenamiento**, realice las siguientes operaciones: 

    1. Elija la suscripción actual si la cuenta de almacenamiento está en la misma suscripción que el servicio. Si la cuenta de almacenamiento está fuera de la suscripción del servicio, especifique otra. 
    
    2. En la lista desplegable, elija un cuenta de almacenamiento existente. 
    
    3. La ubicación se rellenará automáticamente según la cuenta de almacenamiento especificada. 
    
    4. Habilite TLS para garantizar un canal de comunicación de red segura entre el dispositivo y la nube.
    
    5. Haga clic en **Agregar** para agregar esta credencial. 
   
        ![Configurar un servidor de archivos](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs8m.png)

5. Una vez que la credencial se haya creado correctamente, se actualizará la hoja **Configurar** para mostrar las credenciales de la cuenta de almacenamiento especificada. Haga clic en **Configurar**.
   
   ![Configurar un servidor de archivos](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs11m.png)
   
   Verá que se crea un servidor de archivos. Cuando el servidor de archivos se cree correctamente, recibirá una notificación.
   
   ![Configurar un servidor de archivos](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs13m.png)
   
   El estado del dispositivo también cambiará a **En línea**.
   
   ![Configurar un servidor de archivos](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs14m.png)
   
   Puede pasar a agregar un recurso compartido.

## <a name="step-3-add-a-share"></a>Paso 3: Agregar un recurso compartido
Siga estos pasos en [Azure Portal](https://portal.azure.com/) para crear un recurso compartido.

#### <a name="to-create-a-share"></a>Para crear un recurso compartido
1. Seleccione el dispositivo del servidor de archivos que configuró en el paso anterior y haga clic en **...** (o haga clic con el botón derecho). En el menú contextual, seleccione **Agregar recurso compartido**. Como alternativa, puede hacer clic en **+Agregar recurso compartido** en la barra de comandos.
   
   ![Agregar un recurso compartido](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs15m.png)
2. Especifique la configuración del recurso compartido:

   1. Un nombre exclusivo para el recurso compartido. El nombre debe ser una cadena que contenga entre 3 y 127 caracteres.
    
   2. **Descripción** opcional del recurso compartido. La descripción ayudará a identificar a los propietarios del recurso compartido.
    
   3. El **tipo** de recurso compartido. Dicho tipo puede ser **En capas** o **Anclado localmente**. La primera opción es la predeterminada. Para las cargas de trabajo que requieren garantías locales, latencias bajas y un rendimiento más alto, seleccione un recurso compartido **Anclado localmente** . Para todos los demás datos, seleccione un recurso compartido **En capas** .
      Un recurso compartido anclado localmente se aprovisiona de forma intensa y garantiza que los datos principales del recurso compartido continúen siendo locales en el dispositivo y que no se traspasan a la nube. Por otro lado, un recurso compartido en capas tiene aprovisionamiento reducido. Cuando se crea un recurso compartido en capas, el 10 % del espacio se aprovisiona en la capa local y el 90 % del espacio se aprovisiona en la nube. Por ejemplo, si se aprovisiona un volumen de 1 TB, 100 GB residirían en el espacio local y 900 GB se utilizarían en la nube cuando se apilen los datos. A su vez, esto hace que, si agota todo el espacio local en el dispositivo, no se puede aprovisionar un recurso compartido en capas.
   
   4. En el campo **Set default full permissions to** (Establecer permisos totales predeterminados en), asigne los permisos al usuario o al grupo que va a acceder a este recurso compartido. Especifique el nombre del usuario o del grupo de usuarios en el formato *john\@contoso.com*. Se recomienda que utilice un grupo de usuarios (en lugar de un único usuario) para otorgar los privilegios de administrador para tener acceso a estos recursos compartidos. Después de haber asignado los permisos aquí, puede utilizar el Explorador de archivos para modificarlos.
   
   5. Haga clic en **Agregar** para crear el recurso compartido. 
    
       ![Agregar un recurso compartido](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs18m.png)
   
       Se le notifica que la creación del recurso compartido está en curso.
   
       ![Agregar un recurso compartido](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs19m.png)
   
      Una vez creado el recurso compartido con la configuración especificada, la hoja **Recursos compartidos** se actualizará para reflejar el nuevo recurso compartido. De forma predeterminada, las opciones de supervisión y copia de seguridad están habilitadas en el recurso compartido.
   
      ![Agregar un recurso compartido](./media/storsimple-virtual-array-deploy3-fs-setup/deployfs22m.png)

## <a name="step-4-connect-to-the-share"></a>Paso 4: Conexión al recurso compartido
Ahora, necesitará conectarse a uno o varios recursos compartidos que creó en el paso anterior. Realice estos pasos en su host de Windows Server conectado a StorSimple Virtual Array.

#### <a name="to-connect-to-the-share"></a>Para conectarse al recurso compartido
1. Presione :::image type="icon" source="./media/storsimple-virtual-array-deploy3-fs-setup/image22.png"::: + R. En la ventana Ejecutar, especifique el *&#92;&#92;&lt;nombre del servidor de archivos&gt;* como ruta de acceso, pero reemplace *nombre del servidor de archivos* por el nombre del dispositivo que asignó al servidor de archivos. Haga clic en **OK**.
   
   ![Captura de pantalla del cuadro de diálogo Ejecutar. El cuadro Abrir está rellenado con una ruta de acceso que lleva al servidor de archivos.](./media/storsimple-virtual-array-deploy3-fs-setup/image23.png)
2. Se abre el Explorador de archivos. Ahora podrá ver los recursos compartidos que creó como carpetas. Seleccione y haga doble clic en un recurso compartido (carpeta) para ver el contenido.
   
   ![Captura de pantalla del Explorador de archivos. Se pueden ver varias carpetas que representan los recursos compartidos recién creados.](./media/storsimple-virtual-array-deploy3-fs-setup/image24.png)
3. Ahora puede agregar archivos a estos recursos compartidos y realizar una copia de seguridad.

## <a name="next-steps"></a>Pasos siguientes
Aprenda a usar la interfaz de usuario web local para [administrar la matriz virtual de StorSimple](storsimple-ova-web-ui-admin.md).

