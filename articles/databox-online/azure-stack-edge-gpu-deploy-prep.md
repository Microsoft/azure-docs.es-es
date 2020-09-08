---
title: Tutorial de preparación del entorno del centro de datos de Azure Portal para implementar Azure Stack Edge con GPU | Microsoft Docs
description: El primer tutorial para implementar Azure Stack Edge con GPU trata sobre la preparación de Azure Portal.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 08/29/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to prepare the portal to deploy Azure Stack Edge so I can use it to transfer data to Azure.
ms.openlocfilehash: c02af8983489651d8b5b83d5d3107a534c51f067
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89254685"
---
# <a name="tutorial-prepare-to-deploy-azure-stack-edge-with-gpu"></a>Tutorial: Preparación de la implementación de Azure Stack Edge con GPU 

Este es el primero de una serie de tutoriales de implementación necesarios para implementar Azure Stack Edge con GPU en su totalidad. En este tutorial se describe cómo preparar Azure Portal para implementar un recurso de Azure Stack Edge.

Para completar el proceso de instalación y configuración se necesitan privilegios de administrador. La preparación del portal dura menos de 10 minutos.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un nuevo recurso
> * Obtención de la clave de activación

### <a name="get-started"></a>Introducción

Para la implementación de Azure Stack Edge, primero debe preparar el entorno. Una vez que el entorno esté listo, siga los pasos requeridos y, si es necesario, los pasos y procedimientos opcionales para implementar el dispositivo por completo. Las instrucciones detalladas de implementación indican cuándo debe realizar cada uno de estos pasos requeridos y opcionales.

| Paso | Descripción |
| --- | --- |
| **Preparación** |Estos pasos se deben completar como preparación para la próxima implementación. |
| **[Lista de comprobación de la configuración de implementación](#deployment-configuration-checklist)** |Use esta lista de comprobación para recopilar y registrar información antes y durante la implementación. |
| **[Requisitos previos de implementación](#prerequisites)** |Validan que el entorno está listo para la implementación. |
|  | |
|**Tutoriales de implementación** |Estos tutoriales son necesarios para implementar el dispositivo de Azure Stack Edge en producción. |
|**[1. Preparación de Azure Portal para Azure Stack Edge](azure-stack-edge-gpu-deploy-prep.md)** |Cree y configure el recurso de Azure Stack Edge antes de instalar un dispositivo físico de Azure Stack Box Edge. |
|**[2. Instalación de Azure Stack Edge](azure-stack-edge-gpu-deploy-install.md)**|Desempaquete, monte en el bastidor y realice el cableado del dispositivo físico de Azure Stack Edge.  |
|**[3. Conexión a Azure Stack Edge](azure-stack-edge-gpu-deploy-connect.md)** |Una vez instalado el dispositivo, conéctese a la interfaz de usuario web local del dispositivo.  |
|**[4. Configuración de los valores de red para Azure Stack Edge](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md)** |Configure la red, incluida la red de proceso y los valores de proxy web para el dispositivo.   |
|**[5. Configuración de los valores del dispositivo para Azure Stack Edge](azure-stack-edge-gpu-deploy-set-up-device-update-time.md)** |Asigne un nombre de dispositivo y un dominio DNS, configure el servidor de actualización y la hora del dispositivo. |
|**[6. Configuración de los valores de seguridad para Azure Stack Edge](azure-stack-edge-gpu-deploy-configure-certificates.md)** |Configuración de certificados para el dispositivo. Use certificados generados por el dispositivo o aporte sus propios certificados.   |
|**[7. Activación de Azure Stack Edge](azure-stack-edge-gpu-deploy-activate.md)** |Use la clave de activación del servicio para activar el dispositivo. El dispositivo está listo para configurar recursos compartidos SMB o NFS o conectarse a través de REST. |
|**[8. Configuración de proceso](azure-stack-edge-gpu-deploy-configure-compute.md)** |Configure el rol de proceso en el dispositivo. Esto también creará un clúster de Kubernetes. |
|**[9A. Transferencia de datos con recursos compartidos de Edge](azure-stack-edge-j-series-deploy-add-shares.md)** |Agregue recursos compartidos y conéctese a ellos mediante SMB o NFS. |
|**[9B. Transferencia de datos con cuentas de almacenamiento de Edge](azure-stack-edge-j-series-deploy-add-storage-accounts.md)** |Agregue cuentas de almacenamiento y conéctese al almacenamiento de blobs a través de las API de REST. |


Ahora puede empezar a recopilar información relacionada con la configuración de software del dispositivo de Azure Stack Edge.

## <a name="deployment-configuration-checklist"></a>Lista de comprobación de la configuración de implementación

Antes de implementar el dispositivo, deberá recopilar información para configurar el software en el dispositivo de Azure Stack Edge. La preparación de parte de esta información con antelación simplifica el proceso de implementación del dispositivo en el entorno. Use la [lista de comprobación de la configuración de implementación de Azure Stack Edge](azure-stack-edge-gpu-deploy-checklist.md) para anotar los detalles de la configuración cuando implemente el dispositivo.


## <a name="prerequisites"></a>Prerrequisitos

A continuación, encontrará los requisitos previos para configurar su recurso de Azure Stack Edge, el dispositivo de Azure Stack Edge y la red de centros de datos.

### <a name="for-the-azure-stack-edge-resource"></a>Para el recurso de Azure Stack Edge

Antes de comenzar, asegúrese de que:

- Su suscripción de Microsoft Azure está habilitada para un recurso de Azure Stack Edge. Asegúrese de que ha usado una suscripción admitida como [Contrato Enterprise (EA) de Microsoft](https://azure.microsoft.com/overview/sales-number/), [Proveedor de soluciones en la nube (CSP)](https://docs.microsoft.com/partner-center/azure-plan-lp) o [Patrocinio de Microsoft Azure](https://azure.microsoft.com/offers/ms-azr-0036p/). No se admiten suscripciones de pago por uso.
- Tiene acceso de propietario o colaborador en el nivel de grupo de recursos para los recursos de Azure Stack Edge o Data Box Gateway, IoT Hub y Azure Storage.

    - Para crear cualquier recurso de Azure Stack Edge o Data Box Gateway, debe tener permisos de colaborador (o superiores) con ámbito en el nivel del grupo de recursos. También deberá asegurarse de que el proveedor `Microsoft.DataBoxEdge` está registrado. Para obtener información acerca de cómo registrarse, vaya a [Registro de proveedores de recursos](azure-stack-edge-manage-access-power-connectivity-mode.md#register-resource-providers).
    - Para crear cualquier recurso de IoT Hub, asegúrese de que el proveedor Microsoft.Devices está registrado. Para obtener información acerca de cómo registrarse, vaya a [Registro de proveedores de recursos](azure-stack-edge-manage-access-power-connectivity-mode.md#register-resource-providers).
    - Para crear un recurso de una cuenta de Storage también se necesita acceso de colaborador, o superior, con ámbito en el nivel de grupo de recursos. De forma predeterminada Azure Storage es un proveedor de recursos registrado.
- Debe tener acceso de administrador o usuario a Graph API de Azure Active Directory. Para más información, vea [Graph API de Azure Active Directory](https://docs.microsoft.com/previous-versions/azure/ad/graph/howto/azure-ad-graph-api-permission-scopes#default-access-for-administrators-users-and-guest-users-).
- Tiene una cuenta de almacenamiento de Microsoft Azure con credenciales de acceso.

### <a name="for-the-azure-stack-edge-device"></a>Para el dispositivo de Azure Stack Edge

Antes de implementar un dispositivo físico, asegúrese de que:

- Ha revisado la información de seguridad que se incluyó en el paquete de envío.
- Tiene una ranura 1U disponible en un bastidor estándar de 19" en el centro de datos para montar el dispositivo.
- Tiene acceso a una superficie de trabajo plana, estable y nivelada en la que el dispositivo pueda apoyarse con seguridad.
- La ubicación en la que desea efectuar la instalación del dispositivo dispone de corriente alterna estándar de una fuente independiente o una unidad de distribución de energía (PDU) en bastidor con un sistema de alimentación ininterrumpida (UPS).
- Tiene acceso a un dispositivo físico.


### <a name="for-the-datacenter-network"></a>Para la red del centro de datos

Antes de comenzar, asegúrese de que:

- La red del centro de datos está configurada según los requisitos de red para el dispositivo de Azure Stack Edge. Para más información, consulte [Requisitos del sistema de Azure Stack Edge](azure-stack-edge-system-requirements.md).

- Para que Azure Stack Edge funcione normalmente, necesita lo siguiente:

    - Un ancho de banda mínimo de 10 Mbps para garantizar que el dispositivo se mantiene actualizado.
    - Un ancho de banda mínimo de 20 Mbps dedicado a carga y descarga para transferir archivos.

## <a name="create-a-new-resource"></a>Crear un nuevo recurso

Si ya tiene un recurso de Azure Stack Edge para administrar el dispositivo físico, omita este paso y vaya a [Obtención de la clave de activación](#get-the-activation-key).

Siga estos pasos en Azure Portal para crear un recurso de Azure Stack Edge.

1. Use sus credenciales de Microsoft Azure para iniciar sesión en Azure Portal, en esta dirección URL: [https://portal.azure.com](https://portal.azure.com).

2. En el menú izquierdo, seleccione **+ Crear un recurso**. Busque y seleccione **Azure Stack Edge o Data Box Gateway**. Seleccione **Crear**. Si ve algún problema, vaya a [Solución de problemas de pedidos](azure-stack-edge-troubleshoot-ordering.md).

3. Seleccione la suscripción que quiere usar para el dispositivo de Azure Stack Edge. Seleccione el país al que desea enviar el dispositivo físico. Seleccione **Mostrar dispositivos**.

    ![Creación de un recurso 1](media/azure-stack-edge-gpu-deploy-prep/create-resource-1.png)

4. Seleccione el tipo de dispositivo. En **Azure Stack Edge Commercial**, elija **Azure Stack Edge con GPU** y seleccione **Registrarse**. 

    ![Creación de un recurso 2](media/azure-stack-edge-gpu-deploy-prep/create-resource-2.png)

5. Se muestra un breve formulario. Rellene el formulario y seleccione **Enviar**. Microsoft habilitará su suscripción.

    ![Creación de un recurso 3](media/azure-stack-edge-gpu-deploy-prep/create-resource-3.png)

5. Una vez habilitada la suscripción, debería poder continuar con la creación de recursos. En la hoja **Seleccione el tipo de dispositivo**, elija **Seleccionar**.

    ![Creación de un recurso 4](media/azure-stack-edge-gpu-deploy-prep/create-resource-4.png)


6. En la pestaña **Datos básicos**, escriba o seleccione los siguientes **detalles del proyecto**.
    
    |Configuración  |Value  |
    |---------|---------|
    |Subscription    |Este valor se rellena automáticamente según la selección anterior. La suscripción está vinculada a la cuenta de facturación. |
    |Resource group  |Cree un nuevo grupo o seleccione uno existente.<br>Más información sobre los [grupos de recursos de Azure](../azure-resource-manager/resource-group-overview.md).     |

4. Escriba o seleccione los siguientes **detalles de la instancia**.

    |Configuración  |Value  |
    |---------|---------|
    |Nombre   | Nombre descriptivo que identifique el recurso.<br>El nombre tiene entre 2 y 50 caracteres que contiene letras, números y guiones.<br> El nombre comienza y termina con una letra o un número.        |
    |Region     |Para una lista de todas las regiones en las que está disponible el recurso de Azure Stack Edge, consulte [Productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). Si usa Azure Government, todas las regiones de gobierno están disponibles como se muestra en las [regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions/).<br> Elija la ubicación más cercana a la región geográfica donde quiera implementar el dispositivo.|

    ![Creación de un recurso 5](media/azure-stack-edge-gpu-deploy-prep/create-resource-5.png)


5. Seleccione **Siguiente: Dirección de envío**.

    - Si ya tiene un dispositivo, seleccione el cuadro combinado **I have a Azure Stack Edge device** (Tengo un dispositivo de Azure Stack Edge).

        ![Creación de un recurso 6](media/azure-stack-edge-gpu-deploy-prep/create-resource-6.png)

    - Si este es el pedido del nuevo dispositivo, escriba el nombre de contacto, la empresa, la dirección para enviar el dispositivo y la información de contacto.

        ![Creación de un recurso 7](media/azure-stack-edge-gpu-deploy-prep/create-resource-7.png)

6. Seleccione **Siguiente: Review + create** (Revisar y crear).

7. En la pestaña **Revisar y crear**, revise la información de **Detalles de precios**, **Términos de uso** y los detalles de su recurso. Seleccione el cuadro combinado **I have reviewed the privacy terms** (He revisado los términos de privacidad).

    ![Creación de un recurso 8](media/azure-stack-edge-gpu-deploy-prep/create-resource-8.png)

8. Seleccione **Crear**.

Se tarda unos minutos en crear el recurso. Cuando el recurso se haya creado e implementado correctamente, recibirá una notificación. Haga clic en **Go to resource** (Ir al recurso).

![Ir al recurso de Azure Stack Edge](media/azure-stack-edge-deploy-prep/data-box-edge-resource3.png)

Tras realizar el pedido, Microsoft lo revisa y se pone en contacto con usted (por correo electrónico) para indicarle los detalles del envío.

![Notificación de revisión del pedido de Azure Stack Edge](media/azure-stack-edge-deploy-prep/data-box-edge-resource4.png)

Si surgen problemas durante el proceso de pedido, consulte [Solución de problemas de pedidos](azure-stack-edge-troubleshoot-ordering.md).

## <a name="get-the-activation-key"></a>Obtención de la clave de activación

Cuando el recurso de Azure Stack Edge esté en funcionamiento, tendrá que obtener la clave de activación. Esta clave se utiliza para activar y conectar el dispositivo de Azure Stack Edge con el recurso. Puede obtener esta clave ahora mientras está en Azure Portal.

1. Seleccione el recurso que ha creado. Seleccione **Información general** y, luego, **Instalación del dispositivo**.

    ![Selección de instalación del dispositivo](media/azure-stack-edge-deploy-prep/data-box-edge-select-devicesetup.png)

2. En el icono **Activar**, seleccione **Generar clave** para crear una clave de activación. Seleccione el icono de copia para copiar la clave y guárdela para su uso posterior.

    ![Obtención de la clave de activación](media/azure-stack-edge-deploy-prep/get-activation-key.png)

> [!IMPORTANT]
> - La clave de activación expira tres días después de generarla.
> - Si la clave ha expirado, genere una nueva. La clave antigua no es válida.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Azure Stack Edge, como:

> [!div class="checklist"]
> * Crear un nuevo recurso
> * Obtención de la clave de activación

En el siguiente tutorial aprenderá a instalar Azure Stack Edge.

> [!div class="nextstepaction"]
> [Instalación de Azure Stack Edge](./azure-stack-edge-gpu-deploy-install.md)



