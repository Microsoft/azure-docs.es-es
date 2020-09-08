---
title: Tutorial para configurar Azure Data Box | Microsoft Docs
description: En este tutorial, aprenderá a cablear, conectar y encender un dispositivo Azure Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 07/10/2020
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: 53309023a838f4b256327a37866408f556c81d29
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87924163"
---
::: zone target="docs"

# <a name="tutorial-cable-and-connect-to-your-azure-data-box"></a>Tutorial: Cableado y conexión de un dispositivo Azure Data Box

::: zone-end

::: zone target="chromeless"

## <a name="cable-and-connect-to-your-device"></a>Cableado y conexión a su dispositivo

::: zone-end

::: zone target="docs"

En este tutorial se describe cómo cablear, conectar y encender un dispositivo Azure Data Box.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Cableado del dispositivo Data Box
> * Conexión al dispositivo Data Box

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que:

1. Ha realizado el pedido de Azure Data Box.
    - Si necesita un pedido de importación, consulte el [Tutorial: Realización de pedidos de Azure Data Box](data-box-deploy-ordered.md).
    - En caso de un pedido de exportación, consulte el [Tutorial: Realización de pedidos de Azure Data Box](data-box-deploy-export-ordered.md)
1. Ha recibido su dispositivo Data Box y el estado del pedido en el portal se actualiza a **Delivered** (Entregado). 
    - Hay una etiqueta de envío en el bolsillo transparente adherida al dispositivo bajo la etiqueta actual. Manténgala en un lugar seguro, ya que la usará para el envío de devolución.
    - Algunas regiones de Europa pueden recibir el dispositivo empaquetado en una caja. Asegúrese de desempaquetar el dispositivo y guarde la caja para posibles devoluciones.
1. Ha revisado la [directrices de seguridad de Data Box](data-box-safety.md).
1. Ha recibido un cable con toma de tierra para usar con el dispositivo de almacenamiento de 100 TB.
1. Tiene un equipo host que se usa para copiar datos en (pedido de importación) o desde (pedido de exportación) Data Box. El equipo host debe:
    - Ejecutar un [sistema operativo admitido](data-box-system-requirements.md).
    - Estar conectado a una red de alta velocidad. Es muy recomendable tener una conexión de 10 GbE como mínimo. Si no hay disponible una conexión de 10 GbE, se puede usar un vínculo de datos de 1 GbE, pero las velocidades de copia resultarán afectadas. 
1. Debe tener acceso a una superficie plana en la que puede colocar su dispositivo Data Box. Si quiere colocar el dispositivo en un bastidor estándar, necesitará una ranura de 7U en el bastidor del centro de datos. Puede colocar el dispositivo en posición horizontal o vertical en el bastidor.
1. Cuenta con los cables siguientes para conectar su dispositivo Data Box al equipo host.
    - Uno o varios cables de fibra óptica SFP+ o de cobre 10 GbE SFP+ Twinax (se usan con las interfaces de red DATA 1, DATA 2). Data Box cuenta con los adaptadores Mellanox ConnectX®-3 Pro EN de puerto dual 10GBASE-T con interfaz de red PCI Express 3.0, por lo que deben funcionar los cables que son compatibles con esta interfaz. Por ejemplo, se ha utilizado un cable CISCO SFP-H10GB-CU3M 10GBASE-CU TWINAX SFP +3M para pruebas internas. Para más información, consulte la [lista de cables y conmutadores admitidos de Mellanox](https://www.mellanox.com/pdf/firmware/ConnectX3-FW-2_42_5000-release_notes.pdf).
    - Un cable de red RJ-45 CAT 6 (se usa con la interfaz de red MGMT)
    - Un cable de red RJ-45 CAT 6A o RJ-45 CAT 6 (se usa con la interfaz de red DATA 3 configurada como 10 Gbps o 1 Gbps, respectivamente)

## <a name="cable-your-device"></a>Cableado del dispositivo

Realice los pasos siguientes para cablear el dispositivo.

1. Inspeccione el dispositivo para comprobar si hay alguna evidencia de que haya sido alterado o de que haya sufrido algún otro daño evidente. Si el dispositivo está alterado o presenta daños graves, no continúe. Póngase en contacto inmediatamente con el Soporte técnico de Microsoft para que le ayude a evaluar si el dispositivo está en buenas condiciones y si debe devolverlo para que se lo cambien.
2. Transporte el dispositivo a la ubicación donde desea encenderlo. Coloque el dispositivo en una superficie plana. También se puede colocar el dispositivo en un bastidor estándar.
3. Conecte los cables de alimentación y red. A continuación se muestra el panel posterior de un dispositivo conectado con una configuración común. Dependiendo de su entorno, puede escoger otras [opciones de cableado](data-box-cable-options.md).
    
    ![Panel posterior del dispositivo Data Box con los cables](media/data-box-deploy-set-up/data-box-cabled-dhcp.png)

    1. Conecte el cable de alimentación a la entrada de alimentación etiquetada. El otro extremo del cable de alimentación debe conectarse a una unidad de distribución de energía.
    2. Utilice el cable RJ-45 CAT 6 para conectar el puerto MGMT en un extremo y un equipo portátil en el otro extremo.            
    3. Utilice el cable RJ-45 CAT 6A para conectarse al puerto DATA 3 en un extremo. DATA 3 se configura como 10 GbE si se conecta mediante el cable RJ-45 CAT 6A y como 1 GbE si se conecta mediante el cable RJ-45 CAT 6.
    4. En función de las interfaces de red que desee conectar para la transferencia de datos, utilice hasta dos cables de cobre 10 GbE SFP + Twinax y fibra óptica SFP+ para conectar los puertos DATA 1 y DATA 2, respectivamente. 
    5. El otro extremo de los cables de los puertos de datos se conectan al equipo host mediante un conmutador de 10 GbE.

4. Busque el botón de encendido en el panel frontal del dispositivo. Encienda el dispositivo.

    ![Botón de encendido del dispositivo Data Box](media/data-box-deploy-set-up/data-box-powered-door-open.png)

::: zone-end

::: zone target="chromeless"

Una vez que haya recibido el dispositivo, tiene que realizar el cableado y conexión del mismo. 

## <a name="cable-your-device"></a>Cableado del dispositivo

1. Si hay cualquier evidencia de que el dispositivo ha sido manipulado o está dañado, no continúe. Póngase en contacto con el Soporte técnico de Microsoft para recibir un dispositivo de reemplazo.
2. Antes de cablear el dispositivo, asegúrese de tener los cables siguientes:
    
    - Cable de alimentación con tierra (incluido) con calificación de 10 A o más, con un conector IEC60320 C-13 en un extremo para conectar al dispositivo.
    - Un cable de red RJ-45 CAT 6 (se usa con la interfaz de red MGMT)
    - Dos cables de cobre 10 GbE SFP+ Twinax (uso con las interfaces de red DATA 1, DATA 2 de 10 Gbps)
    - Un cable de red RJ-45 CAT 6A o RJ-45 CAT 6 (se usa con la interfaz de red DATA 3 configurada como 10 Gbps o 1 Gbps, respectivamente)

3. Retire el dispositivo y colóquelo en una superficie plana. 
    
4. Cablee el dispositivo como se muestra a continuación.  

    ![Panel posterior del dispositivo Data Box con los cables](media/data-box-deploy-set-up/data-box-cabled-dhcp.png)  

    1. Conecte el cable de alimentación al dispositivo.
    2. Use el cable de red RJ-45 cat. 6 para conectar el equipo host al puerto de administración (MGMT) del dispositivo. 
    3. Use el cable de cobre SFP+ Twinax para conectar al menos una interfaz de red de 10 Gbps (se prefiere frente a la de 1 Gbps), DATA1 o DATA2 para los datos. 
    4. Encienda el dispositivo. El botón de encendido se encuentra en el panel frontal del dispositivo.

::: zone-end

::: zone target="docs"


## <a name="connect-to-your-device"></a>Conexión al dispositivo

Realice los pasos siguientes para configurar el dispositivo mediante la interfaz de usuario web local y la interfaz de usuario del portal.

1. Configure el adaptador Ethernet en portátil que usa para conectarse al dispositivo con la dirección IP estática 192.168.100.5 y la subred 255.255.255.0. 
2. Conéctese al puerto MGMT del dispositivo y acceda a la interfaz de usuario web local en https\://192.168.100.10. Esto puede tardar hasta 5 minutos tras encender el dispositivo.
3. Haga clic en **Details** (Detalles) y haga clic en **Go on to the webpage** (Continuar a la página web).

   ![Conexión a la interfaz de usuario web local](media/data-box-deploy-set-up/data-box-connect-local-web-ui.png) 

4. Verá la página de **inicio de sesión** de la interfaz de usuario web local. Asegúrese de que el número de serie del dispositivo coincide tanto en la interfaz de usuario del portal como en la interfaz de usuario web local. En este momento, el dispositivo está bloqueado.
5. Inicie sesión en el [Portal de Azure](https://portal.azure.com).
6. Descargue las credenciales del dispositivo del portal. Vaya a **General > Device details** (General > Detalles del dispositivo). Copie el valor de **Device password** (Contraseña del dispositivo). La contraseña del dispositivo está asociada a un determinado orden en el portal. 

    ![Obtención de las credenciales del dispositivo](media/data-box-deploy-set-up/data-box-device-credentials.png)
    
    
7. Proporcione la contraseña del dispositivo que obtuvo en Azure Portal en el paso anterior para iniciar sesión en la interfaz de usuario web local del dispositivo. Haga clic en **Iniciar sesión**.
8. En el **Panel**, asegúrese de que las interfaces de red están configuradas. 
   - Si DHCP está habilitado en el entorno, las interfaces de red se configuran automáticamente. 
   - Si DHCP no está habilitado, vaya a **Set network interfaces** (Establecer interfaces de red) y asigne direcciones IP estáticas si es necesario.

     ![Panel del dispositivo](media/data-box-deploy-set-up/data-box-dashboard-1.png)

Una vez configuradas las interfaces de red de datos, también puede utilizar la dirección IP de cualquiera de las interfaces DATA 1 - DATA 3 para acceder a la interfaz de usuario web local en `https://<IP address of a data network interface>`. 

Después de haber completado la configuración del dispositivo, puede conectarse a los recursos compartidos del dispositivo y copiar los datos. 

::: zone-end

::: zone target="chromeless"

## <a name="connect-your-device"></a>Conexión del dispositivo

1. Para obtener la contraseña del dispositivo, vaya a **General > Detalles del dispositivo** en [Azure Portal](https://portal.azure.com).
2. Asigne una dirección IP estática 192.168.100.5 y la subred 255.255.255.0 al adaptador Ethernet en el equipo que usa para conectarse a Data Box. Acceda a la interfaz de usuario web local del dispositivo en `https://192.168.100.10`. La conexión puede tardar hasta 5 minutos tras encender el dispositivo. 
3. Inicie sesión con la contraseña desde Azure Portal. Ve un error que indica un problema con el certificado de seguridad del sitio web. Siga las instrucciones específicas del explorador para continuar a la página web.
4. De manera predeterminada, la configuración de red para la interfaz de datos de 10 Gbps (o 1 Gbps) se establece como DHCP. Si es necesario, puede configurar esta interfaz como estática y proporcionar una dirección IP. 

::: zone-end


::: zone target="docs"

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Azure Data Box, como:

> [!div class="checklist"]
> * Cableado del dispositivo Data Box
> * Conexión al dispositivo Data Box

Prosiga con el siguiente tutorial para aprender a copiar los datos.

> [!div class="nextstepaction"]
> [Copia de los datos en Azure Data Box: pedido de importación](./data-box-deploy-copy-data.md)

Or

> [!div class="nextstepaction"]
> [Copia de los datos desde Azure Data Box: pedido de exportación](./data-box-deploy-export-copy-data.md)

::: zone-end

