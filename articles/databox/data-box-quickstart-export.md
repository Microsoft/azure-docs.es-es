---
title: 'Inicio rápido: Exportación de datos a Microsoft Azure Data Box'
description: Aprenda a exportar rápidamente datos de Azure Data Box en Azure Portal
services: databox
author: twooley
ms.service: databox
ms.subservice: pod
ms.topic: quickstart
ms.date: 07/17/2020
ms.author: twooley
ms.localizationpriority: high
ms.openlocfilehash: 55366096a9eef7380ad8df855931c4d675973631
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783066"
---
# <a name="quickstart-get-started-with-azure-data-box-to-export-data-from-azure"></a>Inicio rápido: Introducción a Azure Data Box para exportar datos de Azure

En este inicio rápido se describe cómo exportar datos de Azure en su ubicación mediante Azure Portal. Los pasos incluyen cómo cablear, configurar y copiar datos de Azure. La guía de inicio rápido se realiza en Azure Portal y en la interfaz de usuario web local del dispositivo.

Para obtener instrucciones detalladas para realizar una implementación y el seguimiento del proceso, vaya a [Tutorial: Creación de un pedido de exportación para Azure Data Box](data-box-deploy-export-ordered.md)

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar:

* Asegúrese de que la suscripción que utilice para el servicio Data Box sea de uno de los siguientes tipos:
  * Contrato Enterprise (EA) de Microsoft. Más información acerca de [las suscripciones de EA](https://azure.microsoft.com/pricing/enterprise-agreement/).
  * Proveedor de soluciones en la nube (CSP). Más información acerca del [programa Azure CSP](https://docs.microsoft.com/azure/cloud-solution-provider/overview/azure-csp-overview).
  * Patrocinio de Microsoft Azure Obtenga más información sobre el [programa de patrocinio de Azure](https://azure.microsoft.com/offers/ms-azr-0036p/).

* Asegúrese de que tiene acceso de propietario o colaborador a la suscripción para crear un pedido de Data Box.
* Revise las [directrices de seguridad para su Data Box](data-box-safety.md).
* Tiene un dispositivo Azure Data Box para mover datos a su ubicación. El equipo host debe:
  * Ejecutar un [sistema operativo admitido](data-box-system-requirements.md).
  * Estar conectado a una red de alta velocidad. Es muy recomendable tener una conexión de 10 GbE como mínimo. Si no hay disponible una conexión de 10 GbE, se puede usar un vínculo de datos de 1 GbE, pero las velocidades de copia resultarán afectadas.
* Debe tener acceso a una superficie plana en la que puede colocar su dispositivo Data Box. Si quiere colocar el dispositivo en un bastidor estándar, necesitará una ranura de 7U en el bastidor del centro de datos. Puede colocar el dispositivo en posición horizontal o vertical en el bastidor.
* Dispone de los siguientes cables para conectar su dispositivo Data Box al equipo host.
  * Dos cables de cobre 10 GbE SFP+ Twinax (se usa con las interfaces de red DATA 1, DATA 2)
  * Un cable de red RJ-45 CAT 6 (se usa con la interfaz de red de administración [MGMT])
  * Un cable de red RJ-45 CAT 6A o RJ-45 CAT 6 (se usa con la interfaz de red DATA 3 configurada como 10 Gbps o 1 Gbps, respectivamente)

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="order"></a>Pedido de

Este paso tarda aproximadamente 10 minutos.

1. Cree un nuevo recurso de Azure Data Box en Azure Portal.
2. Seleccione una suscripción existente habilitada para este servicio y elija el tipo de transferencia **Export to Azure** (Exportar a Azure). En **Source Azure region** (Región de Azure de origen), especifique la región en que se encuentran los datos y en **Destination country** (País de destino) el país al que se va a realizar la transferencia de datos.
3. Seleccione **Data Box**. La capacidad máxima utilizable es de 80 TB y puede crear varios pedidos para tamaños de datos mayores.
4. Seleccione **Add storage account and export type** (Agregar cuenta de almacenamiento y tipo de exportación) y, después, **Select Export option** (Seleccionar opción de exportación).
5. Escriba los detalles del pedido y la información de envío. Si el servicio está disponible en su región, proporcione las direcciones de correo electrónico de notificación, revise el resumen y, a continuación, cree el pedido.

Una vez que se creó el pedido, el dispositivo está preparado para su envío.

## <a name="cable"></a>Cable

Este paso tarda aproximadamente 10 minutos.

Cuando reciba su Data Box, siga estos pasos para cablear el dispositivo, conectarse a él y encenderlo. Este paso tarda aproximadamente 10 minutos.

1. Si hay alguna evidencia de que el dispositivo se haya alterado o de que esté dañado, no siga. Póngase en contacto con el Soporte técnico de Microsoft para recibir un dispositivo de reemplazo.
2. Antes de cablear el dispositivo, asegúrese de tener los cables siguientes:

   * Cable de alimentación con tierra (incluido) con calificación de 10 A, o más, con un conector IEC60320 C-13 en un extremo para conectar al dispositivo.
   * Un cable de red RJ-45 CAT 6 (se usa con la interfaz de red de administración [MGMT])
   * Dos cables de cobre 10 GbE SFP+ Twinax (uso con las interfaces de red DATA 1, DATA 2 de 10 Gbps)
   * Un cable de red RJ-45 CAT 6A o RJ-45 CAT 6 (se usa con la interfaz de red DATA 3 configurada como 10 Gbps o 1 Gbps, respectivamente)

3. Retire el dispositivo y colóquelo en una superficie plana.

4. Cablee el dispositivo como se muestra a continuación.  

    ![Panel posterior del dispositivo Data Box con los cables](media/data-box-deploy-set-up/data-box-cabled-dhcp.png)  

    1. Conecte el cable de alimentación al dispositivo.
    2. Use el cable de red RJ-45 CAT 6 para conectar un equipo host al puerto MGMT del dispositivo.
    3. Use el cable de cobre SFP+ Twinax para conectar al menos una interfaz de red de 10 Gbps (se prefiere frente a la de 1 Gbps), DATA 1 o DATA 2 para los datos.
    4. Encienda el dispositivo. El botón de encendido se encuentra en el panel frontal del dispositivo.

## <a name="connect"></a>Conectar

Este paso tarda entre 5 y 7 minutos en completarse.

1. Para obtener la contraseña del dispositivo, vaya a **General** > **Device details**(Detalles del dispositivo) en [Azure Portal](https://portal.azure.com).
2. Asigne la dirección IP estática 192.168.100.5 y la subred 255.255.255.0 al adaptador Ethernet del equipo que use para conectarse a Data Box. Acceda a la interfaz de usuario web local del dispositivo en `https://192.168.100.10`. La conexión puede tardar hasta 5 minutos tras encender el dispositivo.
3. Inicie sesión con la contraseña desde Azure Portal. Ve un error que indica un problema con el certificado de seguridad del sitio web. Siga las instrucciones específicas del explorador para continuar a la página web.
4. De manera predeterminada, la configuración de red para la interfaz de datos de 10 Gbps (o 1 Gbps) se establece como DHCP. Si es necesario, puede configurar esta interfaz como estática y proporcionar una dirección IP.

## <a name="copy-data"></a>Copia de datos

El tiempo en completar esta operación depende del tamaño de los datos y la velocidad de la red.

1. Si usa un cliente Windows, utilice una herramienta de copia de archivos compatible con SMB, como Robocopy. Para el host NFS, use el comando `cp` o `rsync` para copiar los datos. Para obtener más información sobre cómo usar Robocopy para copiar los datos, vaya a [Robocopy](https://technet.microsoft.com/library/ee851678.aspx).
2. Conéctese a los recursos compartidos del dispositivo y empiece a copiar datos en el equipo host.
<!-- 1. Connect to the device shares using the path:`\\<IP address of your device>\ShareName`. To get the share access credentials, go to the **Connect & copy** page in the local web UI of the Data Box. --> 

## <a name="ship-to-azure"></a>Envío a Azure

Esta operación tarda unos 10 a 15 minutos en completarse.

1. Vaya a la página **Preparar para enviar** de la interfaz de usuario web local y comience la preparación del envío.
2. Desactive el dispositivo desde la interfaz de usuario web local. Quite los cables del dispositivo.
3. Enrolle y coloque de forma segura el cable de alimentación que se proporcionó junto con el dispositivo en la parte posterior del mismo.
4. La etiqueta de envío para devolución debe verse en la pantalla de tinta electrónica. Si la pantalla de E-ink no muestra la etiqueta, vaya a **Información general** > **Descargar la etiqueta de envío** en Azure Portal. Descargue la etiqueta de envío y péguela en el dispositivo.
5. Si va a devolver el dispositivo, programe su recogida con su transportista regional.
6. Una vez que el transportista recoge y examina el dispositivo Data Box, el estado del pedido en el portal se actualiza a **Picked up** (Recogido). También se muestra un identificador de seguimiento.

## <a name="clean-up-resources"></a>Limpieza de recursos

Este paso tarda de 2 a 3 minutos en completarse.

* Puede cancelar el pedido de Data Box en Azure Portal antes de procesar el pedido. Una vez que el pedido se procesa, no se puede cancelar. El pedido sigue su curso hasta que alcanza la fase de finalización. Para cancelar el pedido, vaya a **Información general** y seleccione **Cancelar** en la barra de comandos.

* Podrá eliminar el pedido una vez que el estado se muestre como **Completed** (Completado) o **Canceled** (Cancelado) en Azure Portal. Para eliminar el pedido, vaya a **Información general** y seleccione **Eliminar** en la barra de comandos.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado una orden de exportación de Azure en un dispositivo Azure Data Box. Para más información sobre la administración de Azure Data Box, pase al tutorial siguiente:

> [!div class="nextstepaction"]
> [Uso de Azure Portal para administrar Azure Data Box](data-box-portal-admin.md)
