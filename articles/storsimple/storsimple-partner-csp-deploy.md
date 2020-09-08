---
title: Implementación de StorSimple Virtual Array para el programa de proveedores de soluciones en la nube
description: Conozca cómo un asociado de CSP puede agregar un cliente o una suscripción nueva a un cliente existente y, luego, crear un servicio para implementar una instancia de StorSimple Virtual Array en CSP.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/08/2017
ms.author: alkohli
ms.openlocfilehash: dc0cf718ee51e23fb749bdf57d5344977de009d8
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88182228"
---
# <a name="deploy-storsimple-virtual-array-for-cloud-solution-provider-program"></a>Implementación de StorSimple Virtual Array para el programa de proveedores de soluciones en la nube

## <a name="overview"></a>Información general

Los asociados del programa de proveedores de soluciones en la nube (CSP) pueden implementar StorSimple Virtual Array para sus clientes. Un asociado de CSP puede crear un servicio StorSimple Device Manager. Este servicio luego se puede usar para implementar y administrar StorSimple Virtual Array y los recursos compartidos, volúmenes y copias de seguridad asociados.

En este artículo se describe cómo un asociado de CSP puede agregar un cliente o una suscripción nueva a un cliente existente y, luego, crear un servicio para implementar una instancia de StorSimple Virtual Array en CSP.

## <a name="prerequisites"></a>Prerequisites

Antes de comenzar, asegúrese de lo siguiente:

- Está inscrito en el programa CSP.
- Tiene credenciales válidas de inicio de sesión en el [Centro de partners](https://partnercenter.microsoft.com/). Las credenciales le permiten iniciar sesión en el portal para asociados para agregar clientes nuevos, buscar clientes o navegar a la cuenta de un cliente desde el Panel del asociado. CSP puede funcionar como administrador de StorSimple en nombre del cliente en Azure Portal.
                             
## <a name="add-a-customer"></a>Agregar un cliente

Si agrega un cliente, se crea automáticamente una suscripción. Para agregar un cliente (y crear automáticamente una suscripción), lleve a cabo estos pasos en el portal para asociados.

1. Vaya al [Centro de partners](https://partnercenter.microsoft.com/) e inicie sesión con sus credenciales de CSP. Haga clic en **Panel**.

     ![Panel del Centro de partners](./media/storsimple-partner-csp-deploy/image1.png)
                              
2. En el panel de la izquierda, haga clic en **Clientes**. En el panel de la derecha, haga clic en **Agregar clientes**. Escriba los detalles del cliente. Haga clic en **Next: Subscriptions** (Siguiente: suscripciones) para crear una suscripción de cliente.

    ![Agregar un cliente](./media/storsimple-partner-csp-deploy/image2.png)

3.  Seleccione la oferta de **Microsoft Azure**. Desplácese a la parte inferior de la página y haga clic en **Revisar**.

    ![Revisar la información de la suscripción](./media/storsimple-partner-csp-deploy/image3.png)
                              
4. Revise la información y haga clic en **Enviar**.

    ![Enviar la suscripción](./media/storsimple-partner-csp-deploy/image4.png)

5. Guarde la información de confirmación para consultarla en el futuro.

    ![Guardar la confirmación](./media/storsimple-partner-csp-deploy/image5.png)

6. Busque o vaya al cliente que acaba de agregar. Haga clic en el **nombre de la empresa** para explorar en profundidad los detalles.

    ![Buscar el cliente](./media/storsimple-partner-csp-deploy/image6.png)  

7. En el panel de la izquierda, seleccione **Administración de servicios**. En el panel de la derecha, en **Administer services** (Administrar servicios), haga clic en **Portal de administración de Microsoft Azure** para iniciar sesión como el administrador de Azure del cliente.

    ![Iniciar sesión en Azure Portal](./media/storsimple-partner-csp-deploy/image9.png)

8. Para crear una instancia de StorSimple Device Manager, haga clic en **+ Nuevo** y busque **StorSimple Virtual Device Series** (Serie de dispositivos virtuales de StorSimple) o vaya a esa opción. Para más información, vaya a [Implementación del servicio StorSimple Device Manager](storsimple-virtual-array-manage-service.md).

    ![Crear el servicio StorSimple Device Manager](./media/storsimple-partner-csp-deploy/image8.png)


## <a name="add-a-subscription"></a>Agregar una suscripción

En algunos casos, es posible que tenga un cliente existente y que necesite agregar una suscripción. Para agregar una suscripción a un cliente existente, lleve a cabo estos pasos en el portal para asociados.

1. Vaya al [Centro de partners](https://partnercenter.microsoft.com/) e inicie sesión con sus credenciales de CSP. Haga clic en **Panel**.

     ![Panel del Centro de partners](./media/storsimple-partner-csp-deploy/image1.png)
                              
2. En el panel de la izquierda, haga clic en **Clientes**. Busque o vaya al cliente al que desea agregar una suscripción. Haga clic en el icono ![Expand check icon](./media/storsimple-partner-csp-deploy/expand_pane_icon.png) (Expandir icono de marca de verificación) para expandir la fila del nombre de la empresa del cliente. En los detalles, haga clic en **Agregar suscripciones**.

    ![Clientes](./media/storsimple-partner-csp-deploy/image10.png)

3. Active **Microsoft Azure** en las **ofertas preferidas** de la suscripción y haga clic en **Enviar**. De esta forma se crea una suscripción nueva.

    ![Agregar nueva suscripción](./media/storsimple-partner-csp-deploy/image11.png)

6. Una vez que se crea una suscripción nueva, haga clic en **<-- Clientes** en el panel de la izquierda para volver a la página **Clientes**. Busque el cliente para el que acaba de crear una suscripción. Haga clic en el **nombre de la empresa** para explorar en profundidad los detalles.

    ![Buscar el cliente](./media/storsimple-partner-csp-deploy/image6.png)  

7. En el panel de la izquierda, seleccione **Administración de servicios**. En el panel de la derecha, en **Administer services** (Administrar servicios), haga clic en **Portal de administración de Microsoft Azure** para iniciar sesión como el administrador de Azure del cliente.

    ![Iniciar sesión en Azure Portal](./media/storsimple-partner-csp-deploy/image9.png)

8. Para crear una instancia de StorSimple Device Manager, haga clic en **+ Nuevo** y busque **StorSimple Virtual Device Series** (Serie de dispositivos virtuales de StorSimple) o vaya a esa opción. Para más información, vaya a [Implementación del servicio StorSimple Device Manager](storsimple-virtual-array-manage-service.md).

    ![Crear el servicio StorSimple Device Manager](./media/storsimple-partner-csp-deploy/image8.png)

## <a name="next-steps"></a>Pasos siguientes

- Si tiene más preguntas sobre StorSimple en CSP, vaya a [StorSimple in CSP: Frequently asked questions](storsimple-partner-csp-faq.md) (StorSimple en CSP: preguntas más frecuentes).
- Si está listo para implementar StorSimple, vaya a [Deploy StorSimple Virtual Array for Cloud Solution Provider Program](storsimple-partner-csp-deploy.md) (Implementación de StorSimple Virtual Array para el programa Proveedor de soluciones en la nube).
