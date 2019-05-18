---
title: Registro en Azure NetApp Files | Microsoft Docs
description: Describe cómo registrarse para usar Azure Files de NetApp.
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
ms.date: 05/06/2019
ms.author: b-juche
ms.openlocfilehash: fbe0b82008d7b15332c4e2cd62c49c611f20fe89
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65794709"
---
# <a name="register-for-azure-netapp-files"></a>Registro en Azure NetApp Files

> [!IMPORTANT] 
> Antes de registrar el proveedor de recursos de Azure Files de NetApp, debe haber recibido un correo electrónico desde el equipo de Azure Files de NetApp que confirma que se le ha concedido acceso al servicio. 

En este artículo, obtenga información sobre cómo registrarse para Azure Files de NetApp para que pueda comenzar a usar el servicio.

## <a name="waitlist"></a>Enviar una solicitud de lista de espera para acceder al servicio

1. Enviar una solicitud de lista de espera para acceder al servicio de Azure Files de NetApp a través de la [página de envío de la lista de espera de Azure Files de NetApp](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR8cq17Xv9yVBtRCSlcD_gdVUNUpUWEpLNERIM1NOVzA5MzczQ0dQR1ZTSS4u). 

    Registro de la lista de espera no garantiza el acceso de servicio inmediato. 

2. Espere un correo electrónico de confirmación oficial desde el equipo de Azure Files de NetApp antes de continuar con otras tareas. 

## <a name="resource-provider"></a>Registrar el proveedor de recursos de NetApp

Para usar el servicio, debe registrar el proveedor de recursos de Azure para Azure NetApp Files.

> [!NOTE] 
> Podrá registrar correctamente el proveedor de recursos de NetApp incluso sin que se concede acceso para el servicio. Sin embargo, sin la autorización de acceso, cualquier solicitud de API para crear una cuenta de NetApp o cualquier otro recurso de Azure Files de NetApp o el portal de Azure se rechazará con el siguiente error:  
>
> `{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"NotFound","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidResourceType\",\r\n \"message\": \"The resource type could not be found in the namespace 'Microsoft.NetApp' for api version '2017-08-15'.\"\r\n }\r\n}"}]}`


1. En Azure Portal, haga clic en el icono de Azure Cloud Shell en la esquina superior derecha:

      ![Icono de Azure Cloud Shell](../media/azure-netapp-files/azure-netapp-files-azure-cloud-shell.png)

2. Si tiene varias suscripciones en su cuenta de Azure, seleccione una que figure en la lista blanca para Azure NetApp Files:
    
        az account set --subscription <subscriptionId>

3. En la consola de Azure Cloud Shell, escriba el siguiente comando para comprobar que su suscripción se ha incluido en la lista blanca:
    
        az feature list | grep NetApp

   La salida del comando tiene el siguiente aspecto:
   
       "id": "/subscriptions/<SubID>/providers/Microsoft.Features/providers/Microsoft.NetApp/features/publicPreviewADC",  
       "name": "Microsoft.NetApp/publicPreviewADC" 
       
   `<SubID>` es su identificador de suscripción.

    Si no ve el nombre de la característica `Microsoft.NetApp/publicPreviewADC`, no tiene acceso al servicio. Se detendrá en este paso. Siga las instrucciones de [enviar una solicitud de lista de espera para acceder al servicio](#waitlist) para solicitar el acceso del servicio antes de continuar. 

4. En la consola de Azure Cloud Shell, escriba el comando siguiente para registrar el proveedor de recursos de Azure: 
    
        az provider register --namespace Microsoft.NetApp --wait

   El parámetro `--wait` le indica a la consola que espere a que se complete el registro. El proceso de registro puede tardar algún tiempo en completarse.

5. En la consola de Azure Cloud Shell, escriba el comando siguiente para comprobar que se ha registrado el proveedor de recursos de Azure: 
    
        az provider show --namespace Microsoft.NetApp

   La salida del comando tiene el siguiente aspecto:
   
        {
        "id": "/subscriptions/<SubID>/providers/Microsoft.NetApp",
        "namespace": "Microsoft.NetApp", 
        "registrationState": "Registered", 
        "resourceTypes": […. 

   `<SubID>` es su identificador de suscripción.  El valor del parámetro `state` indica `Registered`.

6. Desde Azure Portal, haga clic en la hoja **Suscripciones**.
7. En la hoja Suscripciones, haga clic en su identificador de suscripción. 
8. En la configuración de la suscripción, haga clic en **Proveedores de recursos** para comprobar que el proveedor de Microsoft.NetApp muestra el estado registrado: 

      ![Microsoft.NetApp registrado](../media/azure-netapp-files/azure-netapp-files-registered-resource-providers.png)


## <a name="next-steps"></a>Pasos siguientes

[Creación de una cuenta de NetApp](azure-netapp-files-create-netapp-account.md)