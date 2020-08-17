---
title: Envío autoadministrado de Microsoft Azure Data Box | Microsoft Docs en datos
description: Describe el flujo de trabajo de envío autoadministrado para dispositivos Azure Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: how-to
ms.date: 08/12/2020
ms.author: alkohli
ms.openlocfilehash: cfa2dee41f07e273d933da356f74063453cb1775
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88142543"
---
# <a name="use-self-managed-shipping-for-azure-data-box-in-the-azure-portal"></a>Uso del envío autoadministrado para Azure Data Box en Azure Portal

En este artículo se describen las tareas de envío autoadministradas para pedir, recoger y descartar un dispositivo Azure Data Box. Puede administrar el dispositivo Data Box mediante Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

El envío autoadministrado está disponible como opción cuando [realiza un pedido de Azure Data Box](data-box-deploy-ordered.md). Solo está disponible en las siguientes regiones:

* US Gov
* Europa Occidental
* Japón
* Singapur
* Corea del Sur
* India
* Sudáfrica

## <a name="use-self-managed-shipping"></a>Uso del envío autoadministrado

Al realizar el pedido de Data Box, puede elegir la opción de envío autoadministrado.

1. En el pedido de Azure Data Box, en **Detalles del contacto**, seleccione **+ Agregar dirección de envío**.
   ![Envío autoadministrado](media\data-box-portal-customer-managed-shipping\choose-self-managed-shipping-1.png)

2. Al elegir el tipo de envío, seleccione la opción **Envío autoadministrado**. Esta opción solo está disponible si se encuentra en una región admitida, tal como se describe en los requisitos previos.

3. Cuando proporcione la dirección de envío, tendrá que validarla y completar el pedido.
   ![Envío autoadministrado](media\data-box-portal-customer-managed-shipping\choose-self-managed-shipping-2.png)

4. Cuando el dispositivo esté preparado y haya recibido una notificación por correo electrónico, podrá programar una recogida.

   En el pedido de Azure Data Box, vaya a **Información general** y seleccione **Programar recogida**.

   ![Pedir un dispositivo Data Box para su recogida](media\data-box-portal-customer-managed-shipping\data-box-portal-schedule-pickup-01.png)

5. Siga las instrucciones indicadas en **Programar recogida para Azure**.

   Antes de poder obtener el código de autorización, debe enviar un correo electrónico a [adbops@microsoft.com](mailto:adbops@microsoft.com) para programar la recogida del dispositivo en el centro de datos de su región.

   ![Programación de la recogida](media\data-box-portal-customer-managed-shipping\data-box-portal-schedule-pickup-email-01.png)

6. Una vez que haya programado la recogida del dispositivo, podrá ver el código de autorización del dispositivo en el panel **Programar recogida para Azure**.

   ![Visualización del código de autorización](media\data-box-portal-customer-managed-shipping\data-box-portal-auth-01b.png)

   Tome nota de este **código de autorización**. Según los requisitos de seguridad, en el momento de programar la recogida, es necesario presentar el nombre de la persona que se encargará de hacerlo.

   También debe proporcionar detalles de quién va a acudir al centro de datos a recoger el dispositivo. Usted o el punto de contacto deben llevar un identificador con una foto aprobado por el gobierno, que se validará en el centro de datos.

   Además, la persona que recoge el dispositivo también debe tener el **código de autorización**. El código de autorización se valida en el centro de datos en el momento de la recogida.

7. El pedido cambia automáticamente al estado **Recogido** una vez que el dispositivo se ha recogido en el centro de datos.

    ![Picked up (Recogido)](media\data-box-portal-customer-managed-shipping\data-box-portal-picked-up-boxed-01.png)

8. Después de recogerlo, copie los datos al dispositivo Data Box en su sitio. Una vez completada la copia, prepare para enviar el dispositivo Data Box. Para obtener más información, vaya a [Preparación para el envío](data-box-deploy-picked-up.md#prepare-to-ship).

   El paso **Preparación para el envío** debe completarse sin errores críticos. De lo contrario, tendrá que volver a ejecutarlo después de realizar las correcciones necesarias. Cuando la preparación para el envío se complete correctamente, podrá ver el código de autorización para la entrega en la interfaz de usuario local del dispositivo.

   > [!NOTE]
   > No comparta el código de autorización por correo electrónico. Este solo debe verificarse en el centro de datos durante la entrega.

9. Si ha recibido una cita para una entrega, el pedido debe encontrarse en el estado **Listo para recibir el paquete en el centro de datos de Azure** en Azure Portal. Siga las instrucciones que aparecen en **Programar la entrega** para devolver el dispositivo.

   ![Visualización del código de autorización](media\data-box-portal-customer-managed-shipping\data-box-portal-received-complete-02b.png)

10. Después de verificar el identificador y el código de autorización y de entregar el dispositivo en el centro de datos, el estado del pedido debe ser **Recibido**.

    ![Recepción completada](media\data-box-portal-customer-managed-shipping\data-box-portal-received-complete-01.png)

11. Una vez que se recibe el dispositivo, la copia de datos continuará. El pedido se completará cuando finalice la copia.

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Azure Data Box](data-box-quickstart-portal.md)
