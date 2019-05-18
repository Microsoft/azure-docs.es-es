---
title: Configuración de una cuenta de laboratorio con Azure Lab Services | Microsoft Docs
description: En este tutorial, va a configurar una cuenta de laboratorio con Azure Lab Services.
services: devtest-lab, lab-services, virtual-machines
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/24/2019
ms.author: spelluru
ms.openlocfilehash: 0977c4537e409b59be7f9031c488b3317f9f2f0f
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65415791"
---
# <a name="tutorial-set-up-a-lab-account-with-azure-lab-services"></a>Tutorial: Configuración de una cuenta de laboratorio con Azure Lab Services
En Azure Lab Services, una cuenta de laboratorio sirve como cuenta central en la que se administran los laboratorios de una organización. En su cuenta de laboratorio, puede conceder permiso a otros usuarios para crear laboratorios y establecer las directivas que se aplican a todos los laboratorios de la cuenta de laboratorio. En este tutorial, aprenderá a crear una cuenta de laboratorio como administrador de laboratorio. 

En este tutorial realizará lo siguiente:

> [!div class="checklist"]
> * Creación de una cuenta de laboratorio
> * Incorporación de un usuario al rol Creador de laboratorio
> * Especificación de las imágenes de Marketplace disponibles para los propietarios de laboratorio

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-a-lab-account"></a>Creación de una cuenta de laboratorio
Los pasos siguientes muestran cómo usar Azure Portal para crear una cuenta de laboratorio con Azure Lab Services. 

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** en el menú de la izquierda. Seleccione **Lab Services** en la sección **DEVOPS**. Si selecciona la estrella (`*`) junto a **Lab Services**, se agrega a la sección **FAVORITOS** en el menú de la izquierda. A partir de la próxima vez, seleccione **Lab Services** en **FAVORITOS**.

    ![Todos los servicios -> Lab Services](../media/tutorial-setup-lab-account/select-lab-accounts-service.png)
3. En la página **Lab Services**, seleccione **Agregar** en la barra de herramientas. 

    ![Seleccione Agregar en la página Cuentas de laboratorio](../media/tutorial-setup-lab-account/add-lab-account-button.png)
4. En la página **Cuenta de laboratorio**, realice las acciones siguientes: 
    1. En **Lab account name** (Nombre de la cuenta de laboratorio), escriba un nombre. 
    2. Seleccione la **suscripción de Azure** donde desea crear la cuenta de laboratorio.
    3. En **Grupo de recursos**, seleccione **Crear nuevo** y escriba un nombre para el grupo de recursos.
    4. En **Ubicación**, seleccione la ubicación o región en la que desea crear la cuenta de laboratorio. 
    5. Seleccione una **galería de imágenes compartidas** existente o créela. Puede guardar la plantilla de máquina virtual en la galería de imágenes compartidas para que otros usuarios puedan usarla. Para más información acerca de las galerías de imágenes compartidas, consulte [Use una galería de imágenes compartidas en Azure Lab Services](how-to-use-shared-image-gallery.md). 
    6. Para **Asociar red virtual**, seleccione una red virtual del mismo nivel como red de laboratorio. Los laboratorios creados en esta cuenta se conectan a la red virtual seleccionada y tienen acceso a los recursos de esta. 
    7. Especifique un **rango de direcciones** para las máquinas virtuales en el laboratorio. Dicho rango debe estar en la notación de enrutamiento de interdominios sin clases (CIDR) (por ejemplo: 10.20.0.0/23). Las máquinas virtuales del laboratorio se crearán en este rango de direcciones. Para más información, consulte el apartado [Especifique un intervalo de direcciones para las máquinas virtuales en el laboratorio](how-to-configure-lab-accounts.md#specify-an-address-range-for-vms-in-the-lab).
    8. En el campo **Allow lab creator to pick lab location** (Permitir que el creador del laboratorio seleccione su ubicación), especifique si quiere que los creadores del laboratorio puedan seleccionar una ubicación para el laboratorio. De forma predeterminada, la opción está deshabilitada. Cuando está deshabilitada, los creadores del laboratorio no pueden especificar una ubicación para el laboratorio que crean. Los laboratorios se crean en la ubicación geográfica más cercana a la cuenta del laboratorio. Cuando está habilitada, el creador del laboratorio puede seleccionar una ubicación en el momento de su creación. 
    9. Seleccione **Crear**. 

        ![Ventana de creación de una cuenta de laboratorio](../media/tutorial-setup-lab-account/lab-account-settings.png)
5. Seleccione el **icono de campana** en la barra de herramientas (**Notificaciones**), confirme que la implementación se realizó correctamente y, a continuación, seleccione **Ir al recurso**. 

    Como alternativa, seleccione **Actualizar** en la página **Cuentas de laboratorio** y seleccione la cuenta de laboratorio que creó. 

    ![Ventana de creación de una cuenta de laboratorio](../media/tutorial-setup-lab-account/go-to-lab-account.png)    
6. Verá la siguiente página de a **cuenta de laboratorio**:

    ![Página de la cuenta de laboratorio](../media/tutorial-setup-lab-account/lab-account-page.png)

## <a name="add-a-user-to-the-lab-creator-role"></a>Incorporación de un usuario al rol Creador de laboratorio
Para configurar un laboratorio de clase en una cuenta de laboratorio, el usuario debe ser miembro del rol **Creador de laboratorio** en la cuenta de laboratorio. La cuenta que usó para crear la cuenta de laboratorio se agrega automáticamente a este rol. Si pretende usar la misma cuenta de usuario para crear un laboratorio de clase, puede omitir este paso. Para usar otra cuenta de usuario para crear un laboratorio de clase, siga los pasos siguientes: 

Para proporcionar a los educadores el permiso para crear laboratorios para sus clases agréguelos al rol **Creador de laboratorios**:

1. En la página de la **cuenta de laboratorio**, seleccione **Control de acceso (IAM)**, **+ Agregar** en la barra de herramientas y, después, seleccione **+ Agregar asignación de roles** en la barra de herramientas. 

    ![Control de acceso -> botón Agregar asignación de roles](../media/tutorial-setup-lab-account/add-role-assignment-button.png)
1. En la página **Agregar asignación de roles**, seleccione **Creador de laboratorio** en **Rol**, seleccione el usuario que quiere agregar al rol Creador de laboratorio y seleccione **Guardar**. 

    ![Incorporación del creador de laboratorio](../media/tutorial-setup-lab-account/add-lab-creator.png)

## <a name="specify-marketplace-images-available-to-lab-creators"></a>Especificación de las imágenes de Marketplace disponibles para los creadores de laboratorios
Como propietario de una cuenta de laboratorio, puede especificar las imágenes de Marketplace que los creadores de laboratorios pueden usar para este fin en la cuenta de laboratorio. 

1. Seleccione **Imágenes de Marketplace** en el menú a la izquierda. De forma predeterminada, aparecerá la lista completa de las imágenes (habilitadas y deshabilitadas). Puede filtrar la lista para ver solamente las imágenes habilitadas o deshabilitadas seleccionando la opción **Enabled only**(Solo habilitadas)/**Disabled only** (Solo deshabilitadas) en la lista desplegable en la parte superior. 
    
    ![Página de imágenes de Marketplace](../media/tutorial-setup-lab-account/marketplace-images-page.png)

    Las imágenes de Marketplace que se muestran en la lista son las únicas que cumplen las condiciones siguientes:
        
    - Crea una sola máquina virtual.
    - Utiliza Azure Resource Manager para aprovisionar las máquinas virtuales.
    - No requiere la compra de un plan de licencias adicional.
2. Para **deshabilitar** una imagen de Marketplace que se ha habilitado, realice una de las siguientes acciones: 
    1. Seleccione **... (puntos suspensivos)**  en la última columna y seleccione **Disable image** (Deshabilitar imagen). 

        ![Deshabilitar una imagen](../media/tutorial-setup-lab-account/disable-one-image.png) 
    2. Seleccione una o más imágenes en la lista seleccionando las casillas que hay antes de los nombres de imagen en la lista y seleccione **Disable selected images** (Deshabilitar imágenes seleccionadas). 

        ![Deshabilitar varias imágenes](../media/tutorial-setup-lab-account/disable-multiple-images.png) 
1. De forma similar, para **habilitar** una imagen de Marketplace, realice una de las siguientes acciones: 
    1. Seleccione **... (puntos suspensivos)**  en la última columna y seleccione **Enable image** (Habilitar imagen). 
    2. Seleccione una o más imágenes en la lista seleccionando las casillas que hay antes de los nombres de imagen en la lista y seleccione **Enable selected images** (Habilitar imágenes seleccionadas). 

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha creado una cuenta de laboratorio. Para aprender a crear un laboratorio como profesor, pase al siguiente tutorial:

> [!div class="nextstepaction"]
> [Configuración de un laboratorio de clase](tutorial-setup-classroom-lab.md)

