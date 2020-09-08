---
title: Tutorial para pedir Azure Data Box Heavy | Microsoft Docs
description: En este tutorial obtendrá información sobre Azure Data Box Heavy, una solución híbrida que le permite importar datos locales en Azure y cómo pedir Data Box Heavy.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: tutorial
ms.date: 07/03/2019
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: 9b2b97f11c1493deca9b79907e894efbb7b9c456
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87921000"
---
# <a name="tutorial-order-azure-data-box-heavy"></a>Tutorial: Pedir Azure Data Box Heavy


Azure Data Box Heavy es una solución híbrida que permite importar datos locales en Azure de forma rápida, fácil y confiable. Transfiera los datos a un dispositivo de almacenamiento de 770 TB (capacidad utilizable aproximada) que suministra Microsoft y, después, devuelva el dispositivo. Luego, dichos datos se cargan en Azure.

En este tutorial se describe cómo se puede solicitar un dispositivo Azure Data Box Heavy. En este tutorial, obtendrá información sobre lo siguiente:

> [!div class="checklist"]
> * Requisitos previos para Data Box Heavy
> * Solicitar un dispositivo Data Box Heavy
> * Seguimiento del pedido
> * Cancelar el pedido

## <a name="prerequisites"></a>Prerrequisitos

Antes de implementar el dispositivo, complete los siguientes requisitos previos de configuración tanto del servicio Data Box como del dispositivo.

### <a name="for-installation-site"></a>Para el sitio de la instalación

Antes de comenzar, asegúrese de que:

- El dispositivo entra por puertas y entradas estándar. Sin embargo, asegúrese de que el dispositivo puede pasar por todas las entradas. Las dimensiones del dispositivo son: ancho: 66,04 cm largo: 121,92 cm alto: 71,12 cm.
- Si la instalación se realiza en una planta que no sea la baja, necesita acceso al dispositivo a través de un ascensor o una rampa. El dispositivo pesa aproximadamente 226 kg.
- Asegúrese de que tiene una sitio llano en el centro de datos que se encuentra próximo a una conexión de red disponible con espacio para un dispositivo con esta superficie.

### <a name="for-service"></a>Para el servicio

[!INCLUDE [Data Box service prerequisites](../../includes/data-box-supported-subscriptions.md)]

### <a name="for-device"></a>Para el dispositivo

Antes de comenzar, asegúrese de que:
- El dispositivo se ha desembalado.
- Tiene un equipo host conectado a la red del centro de datos. Data Box Heavy copiará los datos desde dicho equipo. El equipo host debe ejecutar un sistema operativo compatible, como se describe en [Azure Data Box Heavy system requirements](data-box-system-requirements.md) (Requisitos del sistema de Azure Data Box).
- Necesita tener un portátil con un cable RJ-45 para conectarse a la interfaz de usuario local y configurar el dispositivo. Use el portátil para configurar cada nodo del dispositivo una vez.
- El centro de datos debe tener una red de alta velocidad. Es muy recomendable tener una conexión de 10 GbE como mínimo.
- Necesita un cable de 40 Gbps o de 10 Gbps por nodo de dispositivo. Elija cables que sean compatibles con la interfaz de red [MCX314A-BCCT de Mellanox](https://store.mellanox.com/products/mellanox-mcx314a-bcct-connectx-3-pro-en-network-interface-card-40-56gbe-dual-port-qsfp-pcie3-0-x8-8gt-s-rohs-r6.html):

    - Para el cable de 40 Gbps, el extremo del cable del dispositivo debe ser QSFP+.
    - Para el cable de 10 Gbps, necesitará un cable SFP+ que se conecte a un conmutador 10-G en un extremo, con un adaptador de QSFP+ a SFP+ (o el adaptador QSA) en el extremo que se conecta al dispositivo.
    - Los cables de alimentación se incluyen con el dispositivo.

## <a name="order-data-box-heavy"></a>Solicitar Data Box Heavy

Para solicitar un dispositivo, siga estos pasos en Azure Portal.

1. Use sus credenciales de Microsoft Azure para iniciar sesión en esta dirección URL: [https://portal.azure.com](https://portal.azure.com).
2. Seleccione **+ Crear un recurso** y busque *Azure Data Box*. Seleccione **Azure Data Box**.
    
   [![Buscar Azure Data Box 1](media/data-box-deploy-ordered/search-azure-data-box1.png)](media/data-box-deploy-ordered/search-azure-data-box1.png#lightbox)

3. Seleccione **Crear**.

4. Compruebe si el servicio Data Box está disponible en su región. Escriba o seleccione la siguiente información y seleccione **Aplicar**.

    |Configuración  |Value  |
    |---------|---------|
    |Subscription     | Seleccione una suscripción patrocinada por EA, CSP o Azure para el servicio Data Box. <br> La suscripción está vinculada a la cuenta de facturación.       |
    |Tipo de transferencia     | Seleccione **Importar en Azure**.        |
    |País o región de origen     | Seleccione el país o región en que residen los datos actualmente.         |
    |Región de Azure de destino     | Seleccione la región de Azure a la que desea transferir los datos.        |

    [![Seleccionar disponibilidad de la familia de Data Box](media/data-box-deploy-ordered/select-data-box-option1.png)](media/data-box-deploy-ordered/select-data-box-option1.png#lightbox)

5. Seleccione **Data Box Heavy**. La capacidad máxima utilizable para un solo pedido es de 770 TB.

    [![Seleccionar Data Box Heavy](media/data-box-heavy-deploy-ordered/select-data-box-heavy.png)

6. En **Order** (Pedido), especifique el valor de **Order details** (Detalles del pedido). Escriba o seleccione la siguiente información y seleccione **Siguiente**.
    
    |Configuración  |Value  |
    |---------|---------|
    |Nombre     | Especifique un nombre descriptivo para hacer un seguimiento del pedido. <br> El nombre puede tener entre 3 y 24 caracteres que pueden ser letras, números y guiones. <br> El nombre debe empezar y terminar con una letra o un número.      |
    |Resource group     | Uso uno existente o cree uno nuevo. <br> Un grupo de recursos es un contenedor lógico para los recursos que se pueden administrar o implementar conjuntamente.         |
    |Región de Azure de destino     | Seleccione la región de la cuenta de almacenamiento. <br> Para más información, vaya a [Disponibilidad por región](https://azure.microsoft.com/global-infrastructure/services/?products=databox).        |
    |Destino de almacenamiento     | Elija una cuenta de almacenamiento o discos administrados, o ambos. <br> En función de la región de Azure especificada, seleccione una o varias cuentas de almacenamiento en la lista filtrada una cuenta de almacenamiento existente. <br>Data Box Heavy se puede vincular con un máximo de diez cuentas de almacenamiento. <br> También puede crear una nueva **cuenta de uso general v1**, **cuenta de uso general v2** o **cuenta de almacenamiento de blobs**. <br>Consulte la [cuentas de almacenamiento compatibles con el dispositivo](data-box-heavy-system-requirements.md#supported-storage-accounts). <br>Se admiten cuentas de almacenamiento con redes virtuales. Para permitir que el servicio de Data Box trabaje con cuentas de almacenamiento seguro, habilite los servicios de confianza dentro de la configuración de firewall de la red de la cuenta de almacenamiento. Para más información, vea cómo [agregar el servicio Azure Data Box como un servicio de confianza](../storage/common/storage-network-security.md#exceptions).|

    Si usa la cuenta de almacenamiento como el destino de almacenamiento, consulte la siguiente captura de pantalla:

    ![Pedido de Data Box Heavy para una cuenta de almacenamiento](media/data-box-heavy-deploy-ordered/order-storage-account.png)

    Si además de una cuenta de almacenamiento como destino de almacenamiento, también usa Data Box Heavy para crear discos administrados desde los discos duros virtuales de un entorno local, deberá proporcionar la siguiente información:

    |Configuración  |Value  |
    |---------|---------|
    |Grupos de recursos     | Cree grupos de recursos si pretende crear discos administrados desde los discos duros virtuales locales. Puede usar un grupo de recursos existente solo si este se creó previamente al crear un pedido de Data Box Heavy para el disco administrado por el servicio Data Box. <br> Especifique varios grupos de recursos separados por punto y coma. Se admite un máximo de 10 grupos de recursos.|

    ![Solicitud de Data Box Heavy para un disco administrado](media/data-box-heavy-deploy-ordered/order-managed-disks.png)

    La cuenta de almacenamiento especificada para los discos administrados se usa como una cuenta de almacenamiento provisional. El servicio Data Box carga los discos duros virtuales como blob en páginas en la cuenta de almacenamiento provisional y, a continuación, los convierte en discos administrados y los mueve a los grupos de recursos. Para más información, vea [Comprobación de la carga de datos en Azure](data-box-deploy-picked-up.md#verify-data-upload-to-azure).

7. En **Dirección de envío**, escriba su nombre y apellidos, el nombre y la dirección postal de la empresa y un número de teléfono válido. Seleccione **Validar la dirección**. 

    El servicio valida la dirección de envío para conocer la disponibilidad del servicio. Si el servicio está disponible para la dirección de envío especificada, recibirá una notificación al respecto. Seleccione **Next** (Siguiente).

8. En **Detalles de la notificación**, especifique las direcciones de correo electrónico. El servicio envía notificaciones por correo electrónico si se produce cualquier actualización en el estado del pedido a las direcciones de correo electrónico especificadas.

    Es aconsejable usar un correo electrónico de grupo, con el fin de que siga recibiendo notificaciones aunque algún administrador deje el grupo.

9. Examine el **resumen** de la información relacionada con el pedido, el contacto, la notificación y los términos de privacidad. Active la casilla correspondiente a contrato acuerdo con los términos de privacidad.

10. Seleccione **Pedido**. El pedido tarda unos minutos en crearse.


## <a name="track-the-order"></a>Seguimiento del pedido

Tras realizar el pedido, puede hacer un seguimiento del estado del mismo desde Azure Portal. Vaya al pedido de Data Box Heavy y, después, vaya a **Información general** para ver el estado. El portal muestra el pedido en estado **Ordered** (Pedido).

Si el dispositivo no está disponible, recibe una notificación. Si el dispositivo está disponible, Microsoft identifica que está listo para el envío y prepara dicho envío. Durante la preparación del dispositivo, se producen las siguientes acciones:

- Se crean recursos compartidos de SMB para cada cuenta de almacenamiento asociada con el dispositivo.
- Para cada cuenta, se generan las credenciales de acceso, como el nombre de usuario y la contraseña.
- También se genera la contraseña del dispositivo, que le ayuda a desbloquear el dispositivo.
- Data Box Heavy está bloqueado para evitar el acceso no autorizado al dispositivo en todo momento.

Una vez que se completa la preparación del dispositivo, el portal muestra el pedido en estado **Processed** (Procesado).

![Pedido de Data Box Heavy procesado](media/data-box-overview/data-box-order-status-processed.png)

Luego, Microsoft prepara y envía el disco a través de un operador regional. Una vez que se realiza el envío, recibe un número de seguimiento. El portal muestra el orden en estado **Dispatched** (Enviado).

![Pedido de Data Box Heavy enviado](media/data-box-overview/data-box-order-status-dispatched.png)

## <a name="cancel-the-order"></a>Cancelar el pedido

Para cancelar el pedido, en Azure Portal, vaya a **Información general** y haga clic en **Cancelar** en la barra de comandos.

Después de realizar un pedido, puede cancelarlo en cualquier momento, siempre que no se encuentre en estado procesado.
 
Para eliminar un pedido cancelado, vaya a **Información general** y haga clic en **Eliminar** desde la barra de comandos.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Azure Data Box Heavy, como:

> [!div class="checklist"]
> * Prerrequisitos
> * Solicitar Data Box Heavy
> * Seguimiento del pedido
> * Cancelar el pedido

En el siguiente tutorial aprenderá a configurar Data Box Heavy.

> [!div class="nextstepaction"]
> [Configuración de Azure Data Box Heavy](./data-box-heavy-deploy-set-up.md)
