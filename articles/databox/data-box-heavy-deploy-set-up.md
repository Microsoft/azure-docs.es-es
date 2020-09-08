---
title: Tutorial para configurar Azure Data Box Heavy | Microsoft Docs
description: En este tutorial, aprenderá a cablear, conectar y encender un dispositivo Azure Data Box Heavy.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: tutorial
ms.date: 09/03/2019
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: c38ff7e642017afe5f220ae26d3a04c2c0b706ee
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87920949"
---
::: zone target = "docs"

# <a name="tutorial-cable-and-connect-to-your-azure-data-box-heavy"></a>Tutorial: Cableado y conexión a Azure Data Box Heavy

::: zone-end

::: zone target = "chromeless"

## <a name="get-started-with-azure-data-box-heavy"></a>Introducción a Azure Data Box Heavy

::: zone-end

::: zone target = "docs"

En este tutorial se describe cómo cablear, conectar y encender un dispositivo Azure Data Box Heavy.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Cablear Data Box Heavy
> * Conectarse a un dispositivo Data Box Heavy

::: zone-end

::: zone target = "chromeless"

En esta guía se proporcionan instrucciones para revisar los requisitos previos, conectar el dispositivo, copiar datos, realizar la carga en Azure y, después, comprobar los datos cargados.

::: zone-end

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, asegúrese de que:

1. Ha completado el [Tutorial: Pedir Azure Data Box Heavy](data-box-heavy-deploy-ordered.md).
2. Ha recibido su dispositivo Data Box Heavy y el estado del pedido en el portal se actualiza a **Delivered** (Entregado).
3. Ha revisado la [directrices de seguridad de Data Box Heavy](data-box-safety.md).
4. Debe tener acceso a un sitio llano en el centro de datos que se encuentre próximo a una conexión de red disponible que pueda acomodar un dispositivo con esta superficie. Este dispositivo no se puede montar en bastidor.
5. Ha recibido cuatro cables de alimentación con toma de tierra para usarlos con el dispositivo de almacenamiento.
6. Tiene un equipo host conectado a la red del centro de datos. Data Box Heavy copiará los datos desde dicho equipo. El equipo host debe ejecutar un [sistema operativo compatible](data-box-heavy-system-requirements.md).
7. El centro de datos debe tener una red de alta velocidad. Es muy recomendable tener una conexión de 10 GbE como mínimo. 
8. Necesita tener un portátil con un cable RJ-45 para conectarse a la interfaz de usuario local y configurar el dispositivo. Use el portátil para configurar cada nodo del dispositivo una vez.
9. Necesita un cable de 40 Gbps o un cable de 10 Gbps por nodo de dispositivo.
    - Elija cables que sean compatibles con la interfaz de red [MCX314A-BCCT de Mellanox](https://store.mellanox.com/products/mellanox-mcx314a-bcct-connectx-3-pro-en-network-interface-card-40-56gbe-dual-port-qsfp-pcie3-0-x8-8gt-s-rohs-r6.html).
    - Para el cable de 40 Gbps, el extremo del cable del dispositivo debe ser QSFP+.
    - Para el cable de 10 Gbps, se necesita un cable SFP+ que se conecte a un conmutador de 10 Gbps en un extremo, con un adaptador de QSFP+ a SFP+ (o el adaptador QSA) en el extremo que se conecta al dispositivo.

::: zone target = "docs"

## <a name="cable-your-device-for-power"></a>Cableado del dispositivo para tener alimentación

Siga estos pasos para cablear el dispositivo.

1. Inspeccione el dispositivo para comprobar si hay alguna evidencia de que haya sido alterado o de que haya sufrido algún otro daño evidente. Si el dispositivo está alterado o presenta daños graves, no continúe. [Póngase en contacto con el Soporte técnico de Microsoft](data-box-disk-contact-microsoft-support.md) de inmediato para que le ayude a evaluar si el dispositivo está en buenas condiciones y si debe devolverlo para que se lo cambien.
2. Mueva el dispositivo al sitio de la instalación.

    ![Sitio de instalación del dispositivo Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-install-site.png)

3. Bloquee las ruedas traseras del dispositivo como se muestra a continuación.

    ![Las ruedas del dispositivo Data Box Heavy están bloqueadas](media/data-box-heavy-deploy-set-up/data-box-heavy-casters-locked.png)

4. Busque los botones que abren las puertas de la parte delantera y trasera del dispositivo. Abra y mueva la puerta delantera hasta que esté alineada con el lateral del dispositivo. Repita este paso con la puerta trasera también.
    Ambas puertas deben permanecer abiertas cuando el dispositivo esté en funcionamiento para que haya un flujo de aire óptimo desde la parte delantera a la trasera del dispositivo.

    ![Puertas de Data Box Heavy abiertas](media/data-box-heavy-deploy-set-up/data-box-heavy-doors-open.png)

5. La bandeja de la parte posterior del dispositivo debe tener cuatro cables de alimentación. Quite todos los cables de la bandeja y apártelos.

    ![Cables de alimentación de Data Box Heavy en la bandeja](media/data-box-heavy-deploy-set-up/data-box-heavy-power-cords-tray.png)

6. El siguiente paso es identificar los distintos puertos de la parte posterior del dispositivo. Hay dos nodos de dispositivo: **NODE1** y **NODE2**. Cada nodo tiene cuatro interfaces de red: **MGMT**, **DATA1**, **DATA2** y **DATA3**. **MGMT** se usa para configurar la administración durante la configuración inicial del dispositivo. **DATA1**-**DATA3** son puertos de datos. Los puertos **MGMT** y **DATA3** son de 1 Gbps, mientras que **DATA1** y **DATA2** puede funcionar como puertos de 40 Gbps o de 10 Gbps. En la parte inferior de los dos nodos del dispositivo hay cuatro fuentes de alimentación (PSU) que se comparten entre los dos nodos de dispositivo. Si mira al dispositivo, las **PSU** son **PSU1**, **PSU2**, **PSU3**, y **PSU4**, de izquierda a derecha.

    ![Puertos de Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-ports.png)

7. Conecte los cuatro cables de alimentación a las fuentes de alimentación del dispositivo. Los LED verde se encienden y parpadean.
8. Use los botones de encendido del plano frontal para encender los nodos del dispositivo. Mantenga el botón de encendido pulsado unos segundos hasta que se enciendan las luces azules. Todos los LED de color verde verdes de las fuentes de alimentación de la parte posterior del dispositivo deben estar fijos. El panel frontal de funcionamiento del dispositivo también contiene indicadores LED de error. Cuando se enciende un LED de error, indica que alguna fuente de alimentación está defectuosa o un problema con las unidades de disco.  

    ![Panel de operaciones frontal de Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-front-ops-panel.png)

## <a name="cable-first-node-for-network"></a>Conecte por cable el primer nodo de la red

En uno de los nodos del dispositivo, realice los pasos siguientes para realizar la conexión por cable a la red.

1. Utilice un cable de red RJ-45 CAT 6 (el cable de la parte superior derecha de la imagen, el que está conectado la toma con la etiqueta MGMT) para conectar el equipo host al puerto de administración de 1 Gbps.
2. Use un cable QSFP+ (de fibra o cobre) para conectar al menos una interfaz de red de 40 Gbps (de preferencia, más de 1 Gbps) para los datos. Si usa un conmutador de 10 Gbps, use un cable SFP+ con un adaptador de QSFP+ a SFP+ (el adaptador QSA) para conectar la interfaz de red de 40 Gbps para los datos.

    ![Puertos de Data Box Heavy cableados](media/data-box-heavy-deploy-set-up/data-box-heavy-ports-cabled.png)

    > [!IMPORTANT]
    > DATA 1 y DATA2 cambian y no coinciden con lo que se muestra en la interfaz de usuario web local.
    > El adaptador del cable de 40 Gbps se conecta cuando se inserta de la forma que se muestra a continuación.

    ![Adaptador de cable de datos de 40 Gbps de Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-cable-adaptor.png)

## <a name="configure-first-node"></a>Configuración del primer nodo

Realice los pasos siguientes para configurar el dispositivo mediante la configuración local y Azure Portal.

1. Descargue las credenciales del dispositivo del portal. Vaya a **General > Device details** (General > Detalles del dispositivo). Copie el valor de **Device password** (Contraseña del dispositivo). Estas contraseñas está asociadas a un determinado orden en el portal. Verá los dos números de serie de dispositivo, que corresponden a los dos nodos de Data Box Heavy. La contraseña del administrador de dispositivos es la misma en ambos nodos.

    ![Credenciales del dispositivo Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-device-credentials.png)

2. Conecte la estación de trabajo cliente al dispositivo a través de un cable de red RJ-45 CAT 6.
3. Configure el adaptador Ethernet en el equipo que usa para conectarse al dispositivo con la dirección IP estática `192.168.100.5` y la subred `255.255.255.0`.

    ![Data Box Heavy se conecta a una interfaz de usuario web local](media/data-box-heavy-deploy-set-up/data-box-heavy-connect-local-web-ui.png)

4. Conéctese a la interfaz de usuario web local en la siguiente dirección URL: `http://192.168.100.10`. Haga clic en **Opciones avanzadas** y, después, en **Proceed to 192.168.100.10 (unsafe)** [Ir a 192.168.100.10 (no es segura)].
5. Verá la página de **inicio de sesión** de la interfaz de usuario web local.
    
    - Uno de los números de serie del nodo de esta página coincide con la interfaz de usuario del portal como con la interfaz de usuario web local. Tome nota del número de nodo para la asignación del número de serie. Hay dos nodos y dos números de serie de dispositivo en el portal. Esta asignación le ayuda a saber qué nodo corresponde a cada número de serie.
    - En este momento, el dispositivo está bloqueado.
    - Especifique la contraseña de administrador de dispositivos que obtuvo en el paso anterior para iniciar sesión en el dispositivo. Haga clic en **Iniciar sesión**.

    ![Inicie sesión en la interfaz de usuario web local pesado de Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-unlock-device.png)

5. En el Panel, asegúrese de que las interfaces de red están configuradas. Hay cuatro interfaces de red en el nodo del dispositivo, dos de 1 Gbps y dos de 40 Gbps. Una de las interfaces de 1 Gbps es de administración, por lo que no puede configurarla el usuario. Las tres interfaces de red restantes están dedicadas a los datos y las puede configurar el usuario.

- Si DHCP está habilitado en el entorno, las interfaces de red se configuran automáticamente.
- Si DHCP no está habilitado, vaya a Set network interfaces (Establecer interfaces de red) y asigne direcciones IP estáticas, en caso de que sea necesario.

    ![Nodo 1 del panel de Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-dashboard-1.png)

## <a name="configure-second-node"></a>Configuración del segundo nodo

Siga los pasos que se detallan en la sección [Configuración del primer nodo](#configure-first-node) para el segundo nodo del dispositivo.

![Nodo 2 del panel de Data Box Heavy](media/data-box-heavy-deploy-set-up/data-box-heavy-dashboard-2.png)

Después de completada la configuración del dispositivo, puede conectarse a los recursos compartidos del dispositivo y copiar los datos desde el equipo en el dispositivo.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Azure Data Box Heavy, como:

> [!div class="checklist"]
> * Cablear Data Box Heavy
> * Conectarse a un dispositivo Data Box Heavy

Pase al siguiente tutorial, donde aprenderá a copiar datos en Data Box Heavy.

> [!div class="nextstepaction"]
> [Copia de datos a un dispositivo Azure Data Box](./data-box-heavy-deploy-copy-data.md)

::: zone-end
