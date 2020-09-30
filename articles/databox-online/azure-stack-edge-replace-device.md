---
title: Reemplazo de un dispositivo Azure Stack Edge Pro | Microsoft Docs
description: Se describe cómo obtener el reemplazo de un dispositivo Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 07/20/2020
ms.author: alkohli
ms.openlocfilehash: ec16a2b42b818e96399b8fdbad4a0951f84ef825
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90893899"
---
# <a name="replace-your-azure-stack-edge-pro-device"></a>Reemplazo de un dispositivo Azure Stack Edge Pro

En este artículo se describe cómo obtener el reemplazo de un dispositivo Azure Stack Edge Pro. Se necesita un dispositivo de sustitución cuando el dispositivo existente tiene un error de hardware o necesita una actualización. 


En este artículo aprenderá a:

> [!div class="checklist"]
>
> * Abrir una incidencia de soporte técnico para el problema de hardware.
> * Crear un nuevo recurso para el dispositivo de reemplazo en Azure Portal.
> * Instalar y activar el dispositivo de reemplazo.
> * Devolver el dispositivo original.

## <a name="open-a-support-ticket"></a>Abrir una incidencia de soporte técnico

Si el dispositivo existente tiene un error de hardware, abra una incidencia de soporte técnico. El equipo de Soporte técnico de Microsoft determinará si no está disponible para esta instancia una unidad de reemplazo de campo (FRU) o si el dispositivo necesita una actualización de hardware. En cualquier caso, el equipo de soporte técnico pedirá un dispositivo de sustitución.

1. Abra una incidencia de soporte técnico con Microsoft Support para indicar que quiere devolver el dispositivo. Seleccione el tipo de problema **Hardware de Azure Stack Edge Pro**.

    ![Abrir una incidencia de soporte técnico](media/azure-stack-edge-replace-device/open-support-ticket-1.png)  

2. Un ingeniero de soporte técnico de Microsoft se pondrá en contacto con usted. Proporcione los detalles del envío.
<!--3. If you need a return shipping box, you can request it. Answer **Yes** to the question **Need an empty box to return**.-->


## <a name="create-a-resource-for-replacement-device"></a>Creación de un recurso para el dispositivo de reemplazo

Siga estos pasos para crear un recurso.

1. Siga los pasos de [Crear un nuevo recurso](azure-stack-edge-deploy-prep.md#create-a-new-resource) para crear un recurso para el dispositivo de reemplazo. 

2. Asegúrese de activar la casilla **I have an Azure Stack Edge Pro device** (Tengo un dispositivo Azure Stack Edge Pro). 

    ![Recurso para el dispositivo de reemplazo](media/azure-stack-edge-replace-device/replace-resource-1.png)  

## <a name="install-and-activate-the-replacement-device"></a>Instalación y activación del dispositivo de reemplazo

Siga estos pasos para instalar y activar el dispositivo de reemplazo:

1. [Instale el dispositivo](azure-stack-edge-deploy-install.md).

2. [Active el dispositivo](azure-stack-edge-deploy-connect-setup-activate.md) con el nuevo recurso que creó anteriormente.

## <a name="return-your-existing-device"></a>Devolución del dispositivo existente

Siga todos los pasos para devolver el dispositivo original:

1. [Borre los datos del dispositivo](azure-stack-edge-return-device.md#erase-data-from-the-device).
2. [Iniciar la devolución del dispositivo](azure-stack-edge-return-device.md#initiate-device-return) original.
3. [Programe una recogida](azure-stack-edge-return-device.md#schedule-a-pickup).
4. [Elimine el recurso](azure-stack-edge-return-device.md#delete-the-resource) asociado al dispositivo devuelto.


## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [devolver un dispositivo Azure Stack Edge Pro](azure-stack-edge-return-device.md).
