---
title: Devolución del dispositivo de Azure Stack Edge | Microsoft Docs
description: Obtenga información acerca de cómo borrar los datos y devolver el dispositivo Azure Stack Edge y, a continuación, eliminar el recurso asociado con el dispositivo.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 07/27/2020
ms.author: alkohli
ms.openlocfilehash: aa917361ad3c967a697421e86d232e1a206c403e
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87923958"
---
# <a name="return-your-azure-stack-edge-device"></a>Devolución del dispositivo de Azure Stack Edge

En este artículo se explica cómo borrar los datos y luego devolver el dispositivo Azure Stack Edge. Una vez que haya devuelto el dispositivo, también puede eliminar el recurso asociado al dispositivo.

En este artículo aprenderá a:

> [!div class="checklist"]
>
> * Borrar los datos de los discos de datos del dispositivo
> * Iniciar la devolución del dispositivo en Azure Portal
> * Empaquetar el dispositivo y programar una recogida
> * Eliminar el recurso en Azure Portal

## <a name="erase-data-from-the-device"></a>Borrar los datos del dispositivo

Para borrar los datos de los discos de datos del dispositivo, debe restablecer el dispositivo. Puede restablecer el dispositivo mediante la interfaz de usuario web local o la interfaz de PowerShell.

Antes de restablecerlo, cree una copia de los datos locales del dispositivo si es necesario. Puede copiar los datos del dispositivo en un contenedor de Azure Storage.

Puede iniciar la devolución del dispositivo incluso antes de restablecer el dispositivo. 

Para restablecer el dispositivo mediante la interfaz de usuario web local, siga estos pasos.

1. En la interfaz de usuario web local, vaya a **Mantenimiento > Restablecer dispositivo**.
2. Seleccione **Restablecer dispositivo**.

    ![Restablecer el dispositivo](media/azure-stack-edge-return-device/device-reset-1.png)

3. Cuando se le pida confirmación, revise la advertencia y seleccione **Sí** para continuar.

    ![Confirmar el restablecimiento](media/azure-stack-edge-return-device/device-reset-2.png)  

Al restablecer el dispositivo se borran los datos de los discos de datos del dispositivo. Según la cantidad de datos del dispositivo, este proceso tarda entre 30 y 40 minutos.

Como alternativa, conéctese a la interfaz de PowerShell del dispositivo y utilice el cmdlet `Reset-HcsAppliance` para borrar los datos de los discos de datos. Para obtener más información, consulte [Restablecer el dispositivo](azure-stack-edge-connect-powershell-interface.md#reset-your-device).

> [!NOTE]
> - Si va a intercambiar o actualizar a un dispositivo nuevo, se recomienda restablecer el dispositivo solo después de que haya recibido el dispositivo nuevo.
> - Al restablecer el dispositivo solo se eliminan todos los datos locales del dispositivo. Los datos que están en la nube no se eliminan, sino que generan [cargos](https://azure.microsoft.com/pricing/details/storage/). Estos datos deben eliminarse por separado mediante una herramienta de administración de almacenamiento en la nube como [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/).

## <a name="initiate-device-return"></a>Inicio de la devolución del dispositivo

Para comenzar el proceso de devolución, siga los pasos siguientes.

1. Vaya al recurso de Azure Stack Edge/Data Box Gateway en Azure Portal. En **Información general**, vaya a la barra de comandos en el panel derecho y seleccione **Devolver dispositivo**. 

    ![Devolver dispositivo 1](media/azure-stack-edge-return-device/return-device-1.png)  

2. En la hoja **Devolver dispositivo**, en **Detalles básicos**:

    1. Indique el número de serie del dispositivo. Para obtener el número de serie del dispositivo, vaya a la interfaz de usuario web local del dispositivo y, a continuación, vaya a **Información general**.  
    
    ![Número de serie del dispositivo 1](media/azure-stack-edge-return-device/device-serial-number-1.png) 

    2. Escriba el número de etiqueta del servicio, que es un identificador de cinco o más caracteres que es único para el dispositivo. La etiqueta de servicio se encuentra en la esquina inferior derecha del dispositivo (cuando se está frente al dispositivo). Extraiga la etiqueta de información (es un panel de etiqueta deslizante). Este panel contiene información del sistema, como la etiqueta de servicio, la NIC, la dirección MAC, etc. 
    
    ![Número de etiqueta de servicio 1](media/azure-stack-edge-return-device/service-tag-number-1.png)

    3. En la lista desplegable, elija un motivo para la devolución.

    ![Devolver dispositivo 2](media/azure-stack-edge-return-device/return-device-2.png) 

3. En **Detalles de envío**:

    1. Escriba su nombre, el nombre de la empresa y la dirección completa de la empresa. Escriba un teléfono del trabajo, incluido el código de área y un identificador de correo electrónico para notificaciones.
    2. Si necesita una caja de devolución, puede solicitarla. Responda **Sí** a la pregunta **Necesita una caja vacía para la devolución**.

    ![Devolver dispositivo 3](media/azure-stack-edge-return-device/return-device-3.png)

4. Revise los **Términos de privacidad** y seleccione la casilla junto a la nota para indicar que ha revisado y acepta los términos de privacidad.

5. Seleccione **Initiate return** (Iniciar devolución).

    ![Devolver dispositivo 4](media/azure-stack-edge-return-device/return-device-4.png) 

6. Una vez que se capturan los detalles de la devolución del dispositivo, puede notificar al equipo de operaciones de Azure Stack Edge mediante un correo electrónico. Puede usar su aplicación de correo electrónico, suponiendo que la aplicación de correo electrónico esté instalada y configurada. También puede copiar los datos para crear y enviar un correo electrónico.

    ![Devolver dispositivo 5](media/azure-stack-edge-return-device/return-device-5.png) 

7. Una vez que el equipo de operaciones de Azure Stack Edge reciba el correo electrónico, le enviará una etiqueta de envío inverso. Cuando reciba esta etiqueta, puede programar la recogida del dispositivo con el transportista. 

## <a name="schedule-a-pickup"></a>Programar una recogida

Para programar una recogida, siga los pasos a continuación.

1. Apague el dispositivo. En la interfaz de usuario web local, vaya a **Mantenimiento > Configuración de encendido**.
2. Seleccione **Apagar**. Cuando se le pida confirmación, haga clic en **Sí** para continuar. Para obtener más información, consulte [Administración de los cables de alimentación](data-box-gateway-manage-access-power-connectivity-mode.md#manage-power).
3. Desconecte los cables de alimentación y quite todos los cables de red del dispositivo.
4. Prepare el paquete de envío con su propia caja o con la caja que ha recibido de Azure. Introduzca en la caja el dispositivo y los cables de alimentación que se han enviado con el dispositivo.
5. Coloque en el paquete la etiqueta de envío que ha recibido de Azure.
6. Programe una recogida con su transportista regional. Si devuelve el dispositivo en EE. UU., el transportista podría ser UPS o FedEx. Póngase en contacto con UPS para programar la recogida:

    1. Llame a la oficina local de UPS (número gratuito específico del país o región).
    2. En la llamada, indique el número de seguimiento del envío inverso que se muestra en la etiqueta impresa.
    3. Si no indica el número de seguimiento, UPS le exigirá que pague una cantidad adicional en la recogida.

    En lugar de programar la recogida, puede devolver Azure Stack Edge en el punto de recogida más cercano.

## <a name="delete-the-resource"></a>Eliminar el recurso

Una vez recibido en el centro de datos de Azure, el dispositivo se inspecciona para detectar daños o signos de manipulación.

- Si el dispositivo llega intacto y en buen estado, el medidor de facturación se detiene para ese recurso. El equipo de operaciones de Azure Stack Edge se pondrá en contacto con usted para confirmar que se ha devuelto el dispositivo. A continuación, puede eliminar el recurso asociado al dispositivo en Azure Portal.
- Si el dispositivo llega significativamente dañado, es posible que se apliquen sanciones. Para obtener más información, consulte las [preguntas más frecuentes sobre dispositivos perdidos o dañados](https://azure.microsoft.com/pricing/details/databox/edge/) y los [términos del servicio del producto](https://www.microsoft.com/licensing/product-licensing/products).  


Puede eliminar el dispositivo en Azure Portal:

- Después de realizar el pedido y antes de que Microsoft prepare el dispositivo.
- Tras devolver el dispositivo a Microsoft, pasa por la inspección física en el centro de datos de Azure, y el equipo de operaciones de Azure Stack Edge llama para confirmar que se ha devuelto.

Si ha activado el dispositivo en otra suscripción o ubicación, Microsoft moverá el pedido a la nueva suscripción o ubicación en el plazo de un día laborable. Una vez movido el pedido, puede eliminar este recurso.


Siga estos pasos para eliminar el dispositivo y el recurso en Azure Portal.

1. En Azure Portal, vaya al recurso y luego a **Introducción**. En la barra de comandos, seleccione **Eliminar**.

    ![Seleccionar Eliminar](media/azure-stack-edge-return-device/delete-resource-1.png)

2. En la hoja **Eliminar dispositivo**, escriba el nombre del dispositivo que quiere eliminar y seleccione **Eliminar**.

    ![Confirmar eliminación](media/azure-stack-edge-return-device/delete-resource-2.png)

Se le notificará una vez el dispositivo y el recurso asociado se hayan eliminado correctamente.


## <a name="next-steps"></a>Pasos siguientes

- Aprenda cómo [obtener el reemplazo de un dispositivo de Azure Stack Edge](azure-stack-edge-replace-device.md).
