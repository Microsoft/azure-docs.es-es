---
title: Registro en Azure NetApp Files | Microsoft Docs
description: Obtenga información sobre cómo registrarse para Azure NetApp Files mediante el envío de una solicitud de espera y el registro del proveedor de recursos de Azure para Azure NetApp Files.
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
ms.topic: how-to
ms.date: 06/09/2020
ms.author: b-juche
ms.openlocfilehash: b8707b6fb006a45d63f1b2b426530a7e25f5b497
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87512931"
---
# <a name="register-for-azure-netapp-files"></a>Registro en Azure NetApp Files

> [!IMPORTANT] 
> Antes de registrar el proveedor de recursos de Azure NetApp Files, debe haber recibido un correo electrónico del equipo de Azure NetApp Files en el que se le confirme que se le ha concedido acceso al servicio. 

En este artículo, obtendrá información sobre cómo registrarse en Azure NetApp Files para comenzar a usar el servicio.

## <a name="submit-a-waitlist-request-for-accessing-the-service"></a><a name="waitlist"></a>Envío de una solicitud de lista de espera para acceder al servicio

1. Envíe una solicitud de lista de espera para acceder al servicio de Azure NetApp File a través de la [página de envío de lista de espera de Azure NetApp Files](https://aka.ms/azurenetappfiles). 

    El registro de la lista de espera no garantiza un acceso inmediato al servicio. 

2. Antes de continuar con otras tareas, debe esperar un correo electrónico de confirmación oficial del equipo de Azure NetApp Files. 

## <a name="register-the-netapp-resource-provider"></a><a name="resource-provider"></a>Registro del proveedor de recursos de NetApp

Para usar el servicio, debe registrar el proveedor de recursos de Azure para Azure NetApp Files.

> [!NOTE] 
> Podrá registrar correctamente el proveedor de recursos de NetApp incluso sin que se le conceda acceso para el servicio. Sin embargo, sin la autorización de acceso, se rechazará cualquier solicitud de la API o de Azure Portal para crear una cuenta de NetApp o cualquier otro recurso de Azure NetApp Files con el siguiente error:  
>
> `{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"NotFound","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidResourceType\",\r\n \"message\": \"The resource type could not be found in the namespace 'Microsoft.NetApp' for api version '2017-08-15'.\"\r\n }\r\n}"}]}`


1. En Azure Portal, haga clic en el icono de Azure Cloud Shell en la esquina superior derecha:

      ![Icono de Azure Cloud Shell](../media/azure-netapp-files/azure-netapp-files-azure-cloud-shell.png)

2. Si tiene varias suscripciones en su cuenta de Azure, seleccione una que figure en la lista blanca para Azure NetApp Files:
    
    ```azurepowershell
    az account set --subscription <subscriptionId>
    ```

3. En la consola de Azure Cloud Shell, escriba el siguiente comando para comprobar que su suscripción se ha incluido en la lista blanca:
    
    ```azurepowershell
    az feature list | grep NetApp
    ```

   La salida del comando tiene el siguiente aspecto:
   
    ```output
    "id": "/subscriptions/<SubID>/providers/Microsoft.Features/providers/Microsoft.NetApp/features/ANFGA",  
    "name": "Microsoft.NetApp/ANFGA" 
    ```
       
   `<SubID>` es su identificador de suscripción.

    Si no visualiza el nombre de la característica `Microsoft.NetApp/ANFGA`, significa que no dispone de acceso al servicio. Deténgase en este paso. Siga las instrucciones de [Envío de una solicitud de lista de espera para acceder al servicio](#waitlist) para solicitar el acceso del servicio antes de continuar. 

4. En la consola de Azure Cloud Shell, escriba el comando siguiente para registrar el proveedor de recursos de Azure: 
    
    ```azurepowershell
    az provider register --namespace Microsoft.NetApp --wait
    ```

   El parámetro `--wait` le indica a la consola que espere a que se complete el registro. El proceso de registro puede tardar algún tiempo en completarse.

5. En la consola de Azure Cloud Shell, escriba el comando siguiente para comprobar que se ha registrado el proveedor de recursos de Azure: 
    
    ```azurepowershell
    az provider show --namespace Microsoft.NetApp
    ```

   La salida del comando tiene el siguiente aspecto:
   
    ```output
    {
     "id": "/subscriptions/<SubID>/providers/Microsoft.NetApp",
     "namespace": "Microsoft.NetApp", 
     "registrationState": "Registered", 
     "resourceTypes": […. 
    ```

   `<SubID>` es su identificador de suscripción.  El valor del parámetro `state` indica `Registered`.

6. Desde Azure Portal, haga clic en la hoja **Suscripciones**.
7. En la hoja Suscripciones, haga clic en su identificador de suscripción. 
8. En la configuración de la suscripción, haga clic en **Proveedores de recursos** para comprobar que el proveedor de Microsoft.NetApp muestra el estado registrado: 

      ![Microsoft.NetApp registrado](../media/azure-netapp-files/azure-netapp-files-registered-resource-providers.png)


## <a name="next-steps"></a>Pasos siguientes

[Creación de una cuenta de NetApp](azure-netapp-files-create-netapp-account.md)
