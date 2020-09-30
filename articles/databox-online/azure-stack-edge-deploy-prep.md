---
title: 'Tutorial: Preparación del entorno del centro de datos de Azure Portal para implementar Azure Stack Edge Pro | Microsoft Docs'
description: El primer tutorial para implementar Azure Stack Edge Pro trata sobre la preparación de Azure Portal.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 07/22/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to prepare the portal to deploy Azure Stack Edge Pro so I can use it to transfer data to Azure.
ms.openlocfilehash: 224131d1b17c4ed34ee847638633a5a1a494ccaf
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90894129"
---
# <a name="tutorial-prepare-to-deploy-azure-stack-edge-pro"></a>Tutorial: Preparación de la implementación de Azure Stack Edge Pro  

Este es el primero de una serie de tutoriales de implementación necesarios para implementar Azure Stack Edge Pro en su totalidad. En este tutorial se describe cómo preparar Azure Portal para implementar un recurso de Azure Stack Edge.

Para completar el proceso de instalación y configuración se necesitan privilegios de administrador. La preparación del portal dura menos de 10 minutos.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear un nuevo recurso
> * Obtención de la clave de activación

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="get-started"></a>Introducción

Para implementar Azure Stack Edge Pro, consulte los siguientes tutoriales en el orden indicado.

| **#** | **En este paso** | **Use estos documentos** |
| --- | --- | --- | 
| 1. |**[Preparación de Azure Portal para Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md)** |Cree y configure el recurso de Azure Stack Edge antes de instalar un dispositivo físico de Azure Stack Edge. |
| 2. |**[Instalación de Azure Stack Edge Pro](azure-stack-edge-deploy-install.md)**|Desempaquete, monte en el bastidor y realice el cableado del dispositivo físico de Azure Stack Edge Pro.  |
| 3. |**[Conexión, configuración y activación de Azure Stack Edge Pro](azure-stack-edge-deploy-connect-setup-activate.md)** |Conéctese a la interfaz de usuario web local, complete la configuración del dispositivo y active el dispositivo. El dispositivo está listo para configurar recursos compartidos de SMB o NFS.  |
| 4. |**[Transferencia de datos con Azure Stack Edge Pro](azure-stack-edge-deploy-add-shares.md)** |Agregue recursos compartidos y conéctese a ellos mediante SMB o NFS. |
| 5. |**[Transformación de datos con Azure Stack Edge Pro](azure-stack-edge-deploy-configure-compute.md)** |Configure los módulos de proceso en el dispositivo para que transformen los datos a medida que estos se mueven a Azure. |

Ya puede empezar a configurar Azure Portal.

## <a name="prerequisites"></a>Prerrequisitos

A continuación, encontrará los requisitos previos para configurar su recurso de Azure Stack Edge, el dispositivo de Azure Stack Edge Pro y la red de centros de datos.

### <a name="for-the-azure-stack-edge-resource"></a>Para el recurso de Azure Stack Edge

Antes de comenzar, asegúrese de que:

* Su suscripción de Microsoft Azure debe estar habilitada para un recurso de Azure Stack Edge. Asegúrese de que ha usado una suscripción admitida como [Contrato Enterprise (EA) de Microsoft](https://azure.microsoft.com/overview/sales-number/), [Proveedor de soluciones en la nube (CSP)](https://docs.microsoft.com/partner-center/azure-plan-lp) o [Patrocinio de Microsoft Azure](https://azure.microsoft.com/offers/ms-azr-0036p/). No se admiten suscripciones de pago por uso.

* Tiene acceso de propietario o colaborador a nivel de grupo de recursos para los recursos de Azure Stack Edge o Data Box Gateway, IoT Hub y Azure Storage.

  * Para conceder acceso de colaborador debe ser **Propietario** en el nivel de suscripción. Para conceder acceso de colaborador a otra persona, en Azure Portal, vaya a **Todos los servicios** > **Suscripciones** > **Control de acceso (IAM)**  >  **+ Agregar** > **Agregar asignación de roles**. Para más información, consulte el [Tutorial: Concesión de acceso de usuario a los recursos de Azure mediante Azure Portal](https://docs.microsoft.com/azure/role-based-access-control/quickstart-assign-role-user-portal).

  * Para crear cualquier recurso de Azure Stack Edge o Data Box Gateway, debe tener permisos de colaborador (o superiores) con ámbito en el nivel del grupo de recursos. También tendrá que asegurarse de que el proveedor de recursos `Microsoft.DataBoxEdge` está registrado. Para obtener información sobre cómo registrar un proveedor de recursos, consulte [Registro de proveedores de recursos](azure-stack-edge-manage-access-power-connectivity-mode.md#register-resource-providers).
  * Para crear cualquier recurso de IoT Hub, asegúrese de que el proveedor Microsoft.Devices está registrado. Para obtener información acerca de cómo registrarse, vaya a [Registro de proveedores de recursos](azure-stack-edge-manage-access-power-connectivity-mode.md#register-resource-providers).
  * Para crear un recurso de una cuenta de Storage también se necesita acceso de colaborador, o superior, con ámbito en el nivel de grupo de recursos. De forma predeterminada Azure Storage es un proveedor de recursos registrado.
* Debe tener acceso de administrador o usuario a Graph API de Azure Active Directory. Para más información, vea [Graph API de Azure Active Directory](https://docs.microsoft.com/previous-versions/azure/ad/graph/howto/azure-ad-graph-api-permission-scopes#default-access-for-administrators-users-and-guest-users-).
* Tiene una cuenta de almacenamiento de Microsoft Azure con credenciales de acceso.
* No está bloqueado por ninguna directiva de Azure configurada por el administrador del sistema. Para más información sobre las directivas, consulte [Inicio rápido: Creación de una asignación de directiva para identificar recursos no compatibles](https://docs.microsoft.com/azure/governance/policy/assign-policy-portal).

### <a name="for-the-azure-stack-edge-pro-device"></a>Para el dispositivo de Azure Stack Edge Pro

Antes de implementar un dispositivo físico, asegúrese de que:

* Ha revisado la información de seguridad que se incluyó en el paquete de envío.
* Tiene una ranura 1U disponible en un bastidor estándar de 19" en el centro de datos para montar el dispositivo.
* Tiene acceso a una superficie de trabajo plana, estable y nivelada en la que el dispositivo pueda apoyarse con seguridad.
* La ubicación en la que desea efectuar la instalación del dispositivo dispone de corriente alterna estándar de una fuente independiente o una unidad de distribución de energía (PDU) en bastidor con un sistema de alimentación ininterrumpida (UPS).
* Tiene acceso a un dispositivo físico.

### <a name="for-the-datacenter-network"></a>Para la red del centro de datos

Antes de comenzar, asegúrese de que:

* La red del centro de datos está configurada según los requisitos de red para el dispositivo de Azure Stack Edge Pro. Para más información, consulte [Requisitos del sistema de Azure Stack Edge Pro](azure-stack-edge-system-requirements.md).

* Para que Azure Stack Edge Pro funcione normalmente, necesita lo siguiente:

  * Un ancho de banda mínimo de 10 Mbps para garantizar que el dispositivo se mantiene actualizado.
  * Un ancho de banda mínimo de 20 Mbps dedicado a carga y descarga para transferir archivos.

## <a name="create-a-new-resource"></a>Crear un nuevo recurso

Si ya tiene un recurso de Azure Stack Edge para administrar el dispositivo físico, omita este paso y vaya a [Obtención de la clave de activación](#get-the-activation-key).

Siga estos pasos en Azure Portal para crear un recurso de Azure Stack Edge.

1. Use sus credenciales de Microsoft Azure para iniciar sesión en: 

    - Azure Portal en esta dirección URL: [https://portal.azure.com](https://portal.azure.com).
    - O bien, el portal de Azure Government en esta dirección URL: [https://portal.azure.us](https://portal.azure.us). Para más información, vaya a [Connect to Azure Government using the portal](https://docs.microsoft.com/azure/azure-government/documentation-government-get-started-connect-with-portal) (Conexión a Azure Government mediante el portal).

2. En el menú izquierdo, seleccione **+ Crear un recurso**. Busque y seleccione **Azure Stack Edge o Data Box Gateway**. Seleccione **Crear**.
3. Seleccione la suscripción que quiere usar para el dispositivo de Azure Stack Edge Pro. Seleccione la región donde desea implementar el recurso de Azure Stack Edge. Para una lista de todas las regiones en las que está disponible el recurso de Azure Stack Edge, consulte [Productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all).

    Elija la ubicación más cercana a la región geográfica donde quiera implementar el dispositivo. La región almacena solo los metadatos para la administración de dispositivos. Los datos reales se pueden almacenar en cualquier cuenta de almacenamiento.
    
    En la opción **Azure Stack Edge Pro**, seleccione **Crear**.

    ![Búsqueda del servicio Azure Stack Edge](media/azure-stack-edge-deploy-prep/data-box-edge-sku.png)

3. En la pestaña **Datos básicos**, escriba o seleccione los siguientes **detalles del proyecto**.
    
    |Configuración  |Valor  |
    |---------|---------|
    |Subscription    |Este valor se rellena automáticamente según la selección anterior. La suscripción está vinculada a la cuenta de facturación. |
    |Resource group  |Cree un nuevo grupo o seleccione uno existente.<br>Más información sobre los [grupos de recursos de Azure](../azure-resource-manager/resource-group-overview.md).     |

4. Escriba o seleccione los siguientes **detalles de la instancia**.

    |Configuración  |Value  |
    |---------|---------|
    |Nombre   | Nombre descriptivo que identifique el recurso.<br>El nombre tiene entre 2 y 50 caracteres que contiene letras, números y guiones.<br> El nombre comienza y termina con una letra o un número.        |
    |Region     |Para una lista de todas las regiones en las que está disponible el recurso de Azure Stack Edge, consulte [Productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). Si usa Azure Government, todas las regiones de gobierno están disponibles como se muestra en las [regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions/).<br> Elija la ubicación más cercana a la región geográfica donde quiera implementar el dispositivo.|

    ![Detalles del proyecto e instancia](media/azure-stack-edge-deploy-prep/data-box-edge-resource.png)

5. Seleccione **Siguiente: Dirección de envío**.

    - Si ya tiene un dispositivo, seleccione el cuadro combinado **I have a Azure Stack Edge Pro device** (Tengo un dispositivo de Azure Stack Edge Pro).
    - Si este es el pedido del nuevo dispositivo, escriba el nombre de contacto, la empresa, la dirección para enviar el dispositivo y la información de contacto.

    ![Dirección de envío del nuevo dispositivo](media/azure-stack-edge-deploy-prep/data-box-edge-resource1.png)

6. Seleccione **Siguiente: Review + create** (Revisar y crear).

7. En la pestaña **Revisar y crear**, revise la información de **Detalles de precios**, **Términos de uso** y los detalles de su recurso. Seleccione el cuadro combinado **I have reviewed the privacy terms** (He revisado los términos de privacidad).

    ![Revisión de los detalles del recurso y los términos de privacidad de Azure Stack Edge](media/azure-stack-edge-deploy-prep/data-box-edge-resource2.png)

8. Seleccione **Crear**.

Se tarda unos minutos en crear el recurso. Cuando el recurso se haya creado e implementado correctamente, recibirá una notificación. Haga clic en **Go to resource** (Ir al recurso).

![Ir al recurso de Azure Stack Edge](media/azure-stack-edge-deploy-prep/data-box-edge-resource3.png)

Tras realizar el pedido, Microsoft lo revisa y se pone en contacto con usted (por correo electrónico) para indicarle los detalles del envío.

![Notificación de revisión del pedido de Azure Stack Edge Pro](media/azure-stack-edge-deploy-prep/data-box-edge-resource4.png)

## <a name="get-the-activation-key"></a>Obtención de la clave de activación

Cuando el recurso de Azure Stack Edge esté en funcionamiento, tendrá que obtener la clave de activación. Esta clave se usa para activar y conectar el dispositivo de Azure Stack Edge Pro con el recurso. Puede obtener esta clave ahora mientras está en Azure Portal.

1. Seleccione el recurso que ha creado. Seleccione **Información general** y, luego, **Instalación del dispositivo**.

    ![Selección de instalación del dispositivo](media/azure-stack-edge-deploy-prep/data-box-edge-select-devicesetup.png)

2. En el icono **Activar**, seleccione **Generar clave** para crear una clave de activación. Seleccione el icono de copia para copiar la clave y guárdela para su uso posterior.

    ![Obtención de la clave de activación](media/azure-stack-edge-deploy-prep/get-activation-key.png)

> [!IMPORTANT]
>
> * La clave de activación expira tres días después de generarla.
> * Si la clave ha expirado, genere una nueva. La clave antigua no es válida.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Azure Stack Edge Pro, tales como:

> [!div class="checklist"]
>
> * Crear un nuevo recurso
> * Obtención de la clave de activación

En el siguiente tutorial aprenderá a instalar Azure Stack Edge Pro.

> [!div class="nextstepaction"]
> [Instalación de Azure Stack Edge Pro](./azure-stack-edge-deploy-install.md)
