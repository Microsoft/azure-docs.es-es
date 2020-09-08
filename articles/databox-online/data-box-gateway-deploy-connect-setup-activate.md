---
title: Conexión, configuración y activación de Azure Data Box Gateway en Azure Portal
description: Este tercer tutorial para implementar Data Box Gateway proporciona instrucciones para conectar, configurar y activar el dispositivo virtual.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: tutorial
ms.date: 03/18/2019
ms.author: alkohli
ms.openlocfilehash: 78c06cc7f08fe94a25ea63d9bf76cc1352d9f2b7
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "82561682"
---
# <a name="tutorial-connect-set-up-activate-azure-data-box-gateway"></a>Tutorial: Conexión, configuración y activación de Azure Data Box Gateway

## <a name="introduction"></a>Introducción

En este tutorial se describe cómo conectar, configurar y activar el dispositivo Data Box Gateway mediante la interfaz de usuario web local. 

El proceso de instalación y activación puede tardar unos 10 minutos en completarse. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Conectarse a un dispositivo virtual
> * Configuración y activación del dispositivo virtual

## <a name="prerequisites"></a>Prerrequisitos

Antes de instalar y configurar Data Box Gateway, asegúrese de que:

* Ha aprovisionado un dispositivo virtual y obtuvo una dirección URL conectada a este, según se detalla en el tutorial de [aprovisionamiento de Data Box Gateway en Hyper-V](data-box-gateway-deploy-provision-hyperv.md) o en el tutorial de [aprovisionamiento de Data Box Gateway en VMware](data-box-gateway-deploy-provision-vmware.md).
* Tiene la clave de activación del servicio Data Box Gateway que creó para administrar los dispositivos Data Box Gateway. Para más información, consulte [Preparación de la implementación de Azure Data Box Gateway](data-box-gateway-deploy-prep.md).


## <a name="connect-to-the-local-web-ui-setup"></a>Conexión a la configuración de la interfaz de usuario web local 

1. Abra una ventana del explorador y acceda a la interfaz de usuario web local del dispositivo en:
   
   https:\//ip-address-of-network-interface
   
   Use la dirección URL de conexión que anotó en el tutorial anterior. Ve un error o advertencia que indica que hay un problema con el certificado de seguridad del sitio web.

2. Seleccione **Continue to this webpage**(Continuar a esta página web). Estos pasos pueden variar en función del explorador que se use.
   
    ![Mensaje de error del certificado de seguridad del sitio web](./media/data-box-gateway-deploy-connect-setup-activate/image2.png)

3. Inicie sesión en la interfaz de usuario web del dispositivo virtual. La contraseña predeterminada es *Password1*. 
   
    ![Iniciar sesión en la interfaz de usuario web local](./media/data-box-gateway-deploy-connect-setup-activate/image3.png)

4. Cuando vea el aviso, cambie la contraseña del dispositivo. Recuerde que la nueva contraseña debe tener entre 8 y 16 caracteres. Asimismo, debe contener 3 de los siguientes requisitos: mayúsculas, minúsculas, números y caracteres especiales.

    ![Cambiar la contraseña del dispositivo](./media/data-box-gateway-deploy-connect-setup-activate/image4.png)

Ahora está en el **Panel** del dispositivo.

## <a name="set-up-and-activate-the-virtual-device"></a>Configuración y activación del dispositivo virtual
 
El panel muestra los distintos valores necesarios para configurar y registrar el dispositivo virtual en el servicio Data Box Gateway. Las opciones **Nombre de dispositivo**, **Configuración de red**, **Configuración de proxy web** y **Configuración horaria** son opcionales. Los únicos valores obligatorios son **Configuración de la nube**.
   
![Página "Panel" de la interfaz de usuario web local](./media/data-box-gateway-deploy-connect-setup-activate/image5.png)

1. En el panel izquierdo, seleccione **Nombre de dispositivo** y escriba un nombre descriptivo para el dispositivo. Dicho nombre debe contener entre 1 y 15 caracteres, así como letras, números y guiones. 

    ![Página de "Nombre de dispositivo" de la interfaz de usuario web local](./media/data-box-gateway-deploy-connect-setup-activate/image6.png)

2. (Opcional) En el panel izquierdo, seleccione **Configuración de red** y configure los distintos valores. En el dispositivo virtual verá al menos una interfaz de red o más, en función de cuántas se configuraron en la máquina virtual subyacente. La página **Configuración de red** de un dispositivo virtual con una interfaz de red habilitada es como la que se muestra a continuación.
    
    ![Página "Configuración de red" de la interfaz de usuario web local](./media/data-box-gateway-deploy-connect-setup-activate/image7.png)
   
    Al configurar la red, tenga en cuenta lo siguiente:

    - Si DHCP está habilitado en el entorno, las interfaces de red se configuran automáticamente. Se asignan automáticamente una dirección IP, la subred, la puerta de enlace y DNS.
    - Si DHCP no está habilitado, puede asignar direcciones IP estáticas si es necesario.
    - Puede configurar la interfaz de red como IPv4.

     >[!NOTE] 
     > Se recomienda no cambiar la dirección IP local de la interfaz de red de estática a DHCP, a menos que se tenga otra dirección IP para conectarse al dispositivo. Si se usara una interfaz de red y cambiara a DHCP, no habría forma de determinar la dirección DHCP. Si desea cambiar a una dirección DHCP, espere a que el dispositivo se haya registrado con el servicio y, después, realice el cambio. Después, puede ver las direcciones IP de todos los adaptadores de las **propiedades del dispositivo** de su servicio en Azure Portal.

3. (Opcional) Configure el servidor proxy web. Aunque la configuración del proxy web es opcional, si usa un proxy web, solo puede configurarlo en esta página.
   
   ![Página "Configuración de proxy web" de la interfaz de usuario web local](./media/data-box-gateway-deploy-connect-setup-activate/image8.png)
   
   En la página **Proxy web** haga lo siguiente:
   
   1. En el cuadro **URL de proxy web** , escriba la dirección URL en este formato: `http://&lt;host-IP address or FQDN&gt;:Port number`. No se admiten direcciones URL HTTPS.
   2. En **Autenticación**, seleccione **No** o **NTLM**.
   3. Si usa la autenticación, especifique el **nombre de usuario** y la **contraseña**.
   4. Para validar y aplicar la configuración del proxy web configurado, seleccione **Aplicar**.

   > [!NOTE]
   > No se admiten los archivos de configuración automática del proxy (PAC). Los archivos de PAC definen el número de exploradores web y otros agentes de usuario que pueden elegir automáticamente el servidor proxy (método de acceso) adecuado para obtener una dirección URL determinada.
   > Los servidores proxy que intentan interceptar y leer todo el tráfico (y luego vuelven a firmar todo con su propia certificación) no son compatibles, ya que el certificado del proxy no es de confianza.
   > Normalmente, los servidores proxy transparentes funcionan bien con Azure Data Box Gateway.

4. (Opcional) En el panel izquierdo, seleccione **Configuración horaria** y, después, configure la zona horaria y los servidores NTP principal y secundario del dispositivo. 

    Se requieren servidores NTP, ya que el dispositivo debe sincronizar la hora para que pueda autenticarse con los proveedores de servicios en la nube.
    
    ![Página "Configuración horaria" de la interfaz de usuario web local](./media/data-box-gateway-deploy-connect-setup-activate/image9.png)
    
    En la página **Configuración horaria**, haga lo siguiente:
    
    1. En la lista desplegable **Zona horaria**, seleccione la zona correspondiente a la ubicación geográfica en la que se va a implementar el dispositivo.
        La zona horaria predeterminada para el dispositivo es la hora del Pacífico (PST). El dispositivo usará esta zona horaria para todas las operaciones programadas.

    2. Especifique un **servidor NTP principal** para el dispositivo o acepte el valor predeterminado de `time.windows.com`.   
        Asegúrese de que su red permite que el tráfico NTP pase del centro de datos a Internet.

    3. Si lo desea, en el cuadro **Servidor NTP secundario**, escriba un servidor secundario para el dispositivo.

    4. Seleccione **Aplicar** para validar y aplicar la configuración horaria.

6. En el panel izquierdo, seleccione **Configuración de la nube** y active el dispositivo con el servicio Data Box Gateway en Azure Portal.
    
    1. En el cuadro **Clave de activación**, escriba la **clave de activación** que obtuvo en [Obtener la clave de activación](data-box-gateway-deploy-prep.md#get-the-activation-key) para Data Box Gateway.

    2. Seleccione **Activar**.
       
         ![Página "Configuración de nube" de la interfaz de usuario web local](./media/data-box-gateway-deploy-connect-setup-activate/image10a.png)
    
    3. El dispositivo está activado y las actualizaciones críticas, si están disponibles, se aplican automáticamente. Verá una notificación al respecto. Supervise el progreso de la actualización mediante Azure Portal.

        ![Página "Configuración de nube" de la interfaz de usuario web local](./media/data-box-gateway-deploy-connect-setup-activate/image12.png)
        
        **El cuadro de diálogo también tiene una clave de recuperación que debe copiar y guardar en una ubicación segura. Esta clave se usa para recuperar los datos en caso de que el dispositivo no pueda iniciarse.**


    4. Es posible que deba esperar varios minutos para que la actualización se complete correctamente. Una vez completada la actualización, inicie sesión en el dispositivo. La página **Configuración de nube** se actualizará para indicar que el dispositivo se ha activado correctamente.

        ![Página "Configuración de nube" actualizada de la interfaz de usuario web local](./media/data-box-gateway-deploy-connect-setup-activate/image13.png)

La instalación del dispositivo está completa. Ahora puede agregar recursos compartidos en el dispositivo.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Conectarse a un dispositivo virtual
> * Configuración y activación del dispositivo virtual

Para aprender a transferir datos con Data Box Gateway, consulte:

> [!div class="nextstepaction"]
> [Transferencia de datos con Data Box Gateway](./data-box-gateway-deploy-add-shares.md).
