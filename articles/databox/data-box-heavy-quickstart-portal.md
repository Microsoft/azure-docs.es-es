---
title: Inicio rápido de Microsoft Azure Data Box Heavy| Microsoft Docs
description: En este inicio rápido, aprenderá a implementar Azure Data Box Heavy desde Azure Portal, lo que incluye el cableado, la configuración y la copia de los datos que se cargan en Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: quickstart
ms.date: 09/03/2019
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: 5a0a040d80911b086561213bd0884ed67545c618
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87920796"
---
::: zone target = "docs"

# <a name="quickstart-deploy-azure-data-box-heavy-using-the-azure-portal"></a>Inicio rápido: Implementación de Azure Data Box Heavy mediante Azure Portal

En este inicio rápido se describe cómo implementar Azure Data Box Heavy mediante Azure Portal. Los pasos incluyen cómo cablear y configurar una unidad Data Box Heavy y cómo copiar datos en ella para que se carguen en Azure. La guía de inicio rápido se realiza en Azure Portal y en la interfaz de usuario web local del dispositivo.

Para obtener instrucciones detalladas para realizar una implementación y el seguimiento del proceso, vaya a [Tutorial: Pedir Azure Data Box Heavy](data-box-heavy-deploy-ordered.md)

## <a name="prerequisites"></a>Prerrequisitos

Antes de implementar el dispositivo, complete los siguientes requisitos previos de configuración del sitio de instalación, del servicio Data Box y del dispositivo.

### <a name="for-installation-site"></a>Para el sitio de la instalación

Antes de comenzar, asegúrese de que:

- El dispositivo puede pasar por todas las entradas. Las dimensiones del dispositivo son: ancho: 66,04 cm largo: 121,92 cm alto: 71,12 cm.
- Tiene acceso al dispositivo mediante un ascensor o una rampa si no tiene pensado instalarlo en una planta baja.
- Cuenta con dos personas para administrar el dispositivo. El dispositivo pesa aproximadamente 226 kg. y viene con ruedas.
- Tiene un sitio llano en el centro de datos que se encuentra próximo a una conexión de red disponible con espacio para un dispositivo con esta superficie.

### <a name="for-service"></a>Para el servicio

[!INCLUDE [Data Box service prerequisites](../../includes/data-box-supported-subscriptions.md)]

### <a name="for-device"></a>Para el dispositivo

Antes de comenzar, asegúrese de que:

- Ha revisado la [directrices de seguridad de su unidad Data Box Heavy](data-box-safety.md).
- Tiene un equipo host conectado a la red del centro de datos. Data Box Heavy copiará los datos desde dicho equipo. El equipo host debe ejecutar un [sistema operativo compatible](data-box-heavy-system-requirements.md).
- Tiene un equipo portátil con un cable RJ-45 para conectarse a la interfaz de usuario local y configurar el dispositivo. Use el portátil para configurar cada nodo del dispositivo una vez.
- El centro de datos tiene red de alta velocidad y dispone de al menos una conexión de 10 GbE.
- Necesita un cable de 40 Gbps o un cable de 10 Gbps por nodo de dispositivo. Elija cables que sean compatibles con la interfaz de red MCX314A-BCCT de Mellanox:
    - Para el cable de 40 Gbps, el extremo del cable del dispositivo debe ser QSFP+.
    - Para el cable de 10 Gbps, necesitará un cable SFP+ que se conecte a un conmutador 10-G en un extremo, con un adaptador de QSFP+ a SFP+ (o el adaptador QSA) en el extremo que se conecta al dispositivo.
- Los cables de alimentación se incluyen en una bandeja en la parte posterior del dispositivo.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="order"></a>Pedido de

Este paso tarda aproximadamente 5 minutos.

1. Cree un nuevo recurso de Azure Data Box en Azure Portal.
2. Seleccione una suscripción existente habilitada para este servicio y elija el tipo de transferencia como de **importación**. Proporcione el **país de origen** donde residen los datos y la **región de destino de Azure** para la transferencia de datos.
3. Seleccione **Data Box Heavy**. La capacidad máxima útil es de 770 TB y puede crear varios pedidos para tamaños de datos mayores.
4. Escriba los detalles del pedido y la información de envío. Si el servicio está disponible en su región, proporcione las direcciones de correo electrónico de notificación, revise el resumen y, a continuación, cree el pedido.

Una vez que se creó el pedido, el dispositivo está preparado para su envío.

::: zone-end

::: zone target = "chromeless"

## <a name="cable-and-connect-to-your-device"></a>Cableado y conexión a su dispositivo

Una vez que haya revisado los requisitos previos, deberá conectarse al dispositivo.

::: zone-end

## <a name="cable-for-power"></a>Cable de alimentación

Este paso tarda unos 5 minutos.

Cuando reciba la unidad de Data Box Heavy, haga lo siguiente para conectar el cable de alimentación al dispositivo y encenderlo.

1. Si hay cualquier evidencia de que el dispositivo ha sido manipulado o está dañado, no continúe. Póngase en contacto con el Soporte técnico de Microsoft para recibir un dispositivo de reemplazo.
2. Mueva el dispositivo hasta el sitio de instalación y bloquee las ruedas traseras.
3. Conecte los cuatro cables de alimentación a las fuentes de alimentación de la parte posterior del dispositivo.
4. Use los botones de encendido del plano frontal para encender los nodos del dispositivo.

## <a name="cable-first-node-for-network"></a>Conecte por cable el primer nodo de la red

Este paso tarda entre 10 y 15 minutos en completarse.

1. Use el cable de red RJ-45 cat. 6 para conectar el equipo host al puerto de administración (MGMT) del dispositivo.
2. Use el cable de cobre Twinax QSFP+ para conectar al menos una interfaz de red de 40 Gbps (preferible más de 1 Gbps), DATA1 o DATA2 para los datos. Si usa un conmutador de 10 Gbps, utilice un cable de cobre Twinax SFP+ con un adaptador de QSFP+ a SFP+ (el adaptador QSA) para conectar la interfaz de red de 40 Gbps para los datos.
3. Cablee el dispositivo como se muestra a continuación.  

    ![Cableado de Data Box Heavy](media/data-box-heavy-quickstart-portal/data-box-heavy-ports-cabled.png)  

## <a name="configure-first-node"></a>Configuración del primer nodo

Este paso tarda entre 5 y 7 minutos en completarse.

1. Para obtener la contraseña del dispositivo, vaya a **General > Detalles del dispositivo** en [Azure Portal](https://portal.azure.com). Se usa la misma contraseña para ambos nodos del dispositivo.
2. Asigne una dirección IP estática 192.168.100.5 y la subred 255.255.255.0 al adaptador Ethernet en el equipo que usa para conectarse a Data Box Heavy. Acceda a la interfaz de usuario web local del dispositivo en `https://192.168.100.10`. La conexión puede tardar hasta 5 minutos tras encender el dispositivo.
3. Inicie sesión con la contraseña desde Azure Portal. Ve un error que indica un problema con el certificado de seguridad del sitio web. Siga las instrucciones específicas del explorador para continuar a la página web.
4. De forma predeterminada, la red de las interfaces (excepto MGMT) se configura como DHCP. Si es necesario, puede configurar estas interfaces como estáticas y proporcionar una dirección IP.

## <a name="cable-and-configure-the-second-node"></a>Cableado y configuración del segundo nodo

Este paso tarda entre 15 y 20 minutos en completarse.

Siga los pasos usado con el primer nodo para cablear y configurar el segundo nodo del dispositivo.  


::: zone target = "docs"

## <a name="copy-data"></a>Copia de datos

El tiempo para completar esta operación depende del tamaño de los datos y de la velocidad de la red a través de la que se copian estos datos.
 
1. Copie los datos en ambos nodos del dispositivo mediante ambas interfaces de datos de 40 Gbps en paralelo.

    - Si usa un host Windows, utilice una herramienta de copia de archivos compatible con SMB, como [Robocopy](https://technet.microsoft.com/library/ee851678.aspx).
    - Para el host NFS, use el comando `cp` o `rsync` para copiar los datos.
2. Conéctese a los recursos compartidos del dispositivo mediante la ruta de acceso:`\\<IP address of your device>\ShareName`. Para obtener las credenciales de acceso a los recursos compartidos, vaya a la página **Connect & copy** (Conectar y copiar) de la interfaz de usuario web local de Data Box Heavy.
3. Asegúrese de que los nombres de los recursos compartidos y de las carpetas, así como los datos, sigan las directrices descritas en los [límites de servicio de Azure Storage y Data Box Heavy](data-box-heavy-limits.md).

## <a name="prepare-to-ship"></a>Preparación para el envío

El tiempo en completar esta operación depende del tamaño de los datos.

1. Una vez finalizada la copia de datos sin errores, vaya a la página **Preparar para enviar** en la interfaz de usuario web local e inicie la preparación del envío.
2. Una vez que el paso **Preparar para enviar** se ha completado correctamente en ambos nodos, apague el dispositivo desde la interfaz de usuario web local.

## <a name="ship-to-azure"></a>Envío a Azure

Esta operación tarda entre 15 y 20 minutos en completarse.

1. Retire los cables y devuélvalos a la bandeja de la parte posterior del dispositivo.
2. Programe una recogida con su transportista regional.
3. Póngase en contacto con [Operaciones de Data Box](mailto:DataBoxOps@microsoft.com) para informar sobre la recogida y obtener la etiqueta de envío de devolución.
4. La etiqueta de envío de devolución debe ser visible en el panel frontal claro del dispositivo.

## <a name="verify-data"></a>Comprobación de datos

El tiempo en completar esta operación depende del tamaño de los datos.

1. Cuando el dispositivo Data Box Heavy se conecta a la red del centro de datos de Azure, los datos se cargan automáticamente en Azure.
2. El servicio Data Box le notifica a través de Azure Portal que la copia de datos se ha completado.

    1. Compruebe en los registros de errores si hay errores y tome las medidas adecuadas.
    2. Compruebe que los datos estén en las cuentas de almacenamiento antes de eliminarlos del origen.

## <a name="clean-up-resources"></a>Limpieza de recursos

Este paso tarda de 2 a 3 minutos en completarse.

- Puede cancelar el pedido de Data Box Heavy en Azure Portal antes de procesar el pedido. Una vez que el pedido se procesa, no se puede cancelar. El pedido sigue su curso hasta que alcanza la fase de finalización. Para cancelar el pedido, vaya a **Información general** y haga clic en **Cancelar** desde la barra de comandos.

- Podrá eliminar el pedido una vez que el estado se muestre como **Completed** (Completado) o **Canceled** (Cancelado) en Azure Portal. Para eliminar el pedido, vaya a **Información general** y haga clic en **Eliminar** desde la barra de comandos.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado un dispositivo Data Box Heavy para ayudarle a importar los datos en Azure. Para más información sobre la administración de Azure Data Box Heavy, avance al tutorial siguiente:

> [!div class="nextstepaction"]
> [Uso de Azure Portal para administrar Data Box Heavy](data-box-portal-admin.md)

::: zone-end
