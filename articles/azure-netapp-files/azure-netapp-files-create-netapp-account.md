---
title: Creación de una cuenta de NetApp para obtener acceso a Azure NetApp Files | Microsoft Docs
description: Describe cómo tener acceso a Azure NetApp Files y crear una cuenta de NetApp para poder configurar un grupo de capacidad y crear un volumen.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/28/2018
ms.author: b-juche
ms.openlocfilehash: 25cae58663f6fa7ef27995c10509eb33e49dd4c7
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522828"
---
# <a name="create-a-netapp-account"></a>Creación de una cuenta de NetApp
La creación de una cuenta de NetApp le permite configurar un grupo de capacidad y crear después un volumen. Use la hoja de Azure NetApp Files para crear una nueva cuenta de NetApp.

## <a name="before-you-begin"></a>Antes de empezar
Debe haber recibido un correo electrónico desde el equipo de Azure Files de NetApp que confirma que se le ha concedido acceso al servicio. Consulte [enviar una solicitud de lista de espera para acceder al servicio](azure-netapp-files-register.md#waitlist).

También debe haber registrado su suscripción para usar el proveedor de recursos de NetApp. Consulte [registrar el proveedor de recursos de NetApp](azure-netapp-files-register.md#resource-provider).

## <a name="steps"></a>Pasos 

1. Inicie sesión en el Portal de Azure. 
2. Acceda a la hoja de Azure NetApp Files mediante uno de los métodos siguientes:  
   * Busque **Azure NetApp Files** en el cuadro de búsqueda de Azure Portal.  
   * Haga clic en **Todos los servicios** en la navegación y, a continuación, filtre Azure NetApp Files.  

   Puede incluir en "Favoritos" la hoja de Azure NetApp Files haciendo clic en el icono de estrella. 

3. Haga clic en **+ Agregar** para crear una nueva cuenta de NetApp.  
   Aparecerá la ventana de la nueva cuenta de NetApp.  

4. Especifique la siguiente información acerca de la cuenta de NetApp: 
   * **Nombre de cuenta**  
     Especifique un nombre único para la suscripción.
   * **Suscripción**  
     Seleccione una suscripción de las existentes.
   * **Grupo de recursos**   
     Use un grupo de recursos existente o cree uno.
   * **Ubicación**  
     Seleccione la región donde desea que la cuenta y sus recursos secundarios se encuentren.  

     ![Nueva cuenta de NetApp](../media/azure-netapp-files/azure-netapp-files-new-netapp-account.png)


5. Haga clic en **Create**(Crear).     
   La cuenta de NetApp que creó aparece ahora en la hoja de Azure NetApp Files. 

> [!NOTE] 
> Si no se ha concedido acceso al servicio de Azure Files de NetApp, recibirá el siguiente error al intentar crear la primera cuenta de NetApp:  
>
> `{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"NotFound","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidResourceType\",\r\n \"message\": \"The resource type could not be found in the namespace 'Microsoft.NetApp' for api version '2017-08-15'.\"\r\n }\r\n}"}]}`

## <a name="next-steps"></a>Pasos siguientes  

[Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md)

