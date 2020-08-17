---
title: Preparación del portal para StorSimple Virtual Array
description: El primer tutorial para implementar la matriz virtual de StorSimple trata sobre la preparación de Azure Portal
author: alkohli
ms.assetid: 68a4cfd3-94c9-46cb-805c-46217290ce02
ms.service: storsimple
ms.topic: conceptual
ms.date: 07/25/2019
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c1a3d0b05f04ad8c182d428843479160666b96ee
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87070327"
---
# <a name="deploy-storsimple-virtual-array---prepare-the-azure-portal"></a>Implementación de la matriz virtual de StorSimple: preparación de Azure Portal

![Diagrama que muestra los pasos necesarios para implementar una matriz virtual. El primer paso se etiqueta como Get started y está resaltado.](./media/storsimple-virtual-array-deploy1-portal-prep/getstarted4.png)


## <a name="overview"></a>Información general

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

Este es el primer artículo de la serie de tutoriales de implementación requeridos para implementar completamente una matriz virtual como un servidor de archivos o un servidor iSCSI mediante el modelo con Resource Manager. En este artículo se describe la preparación necesaria para crear y configurar el servicio de StorSimple Device Manager antes de aprovisionar una matriz virtual. Este artículo también incluye vínculos a una lista de comprobación de la configuración de la implementación, así como a los requisitos previos de configuración.

Para completar el proceso de instalación y configuración se necesitan privilegios de administrador. Se recomienda que revise la lista de comprobación de configuración antes de comenzar. La preparación del portal dura menos de 10 minutos.

La información publicada en este artículo se aplica a la implementación de instancias de StorSimple Virtual Array en Azure Portal, así como en la nube de administración pública de Microsoft Azure.

### <a name="get-started"></a>Primeros pasos
El flujo de trabajo de implementación consta de preparar el portal, aprovisionar una matriz virtual en su entorno virtualizado y completar la instalación. Para comenzar con la implementación de StorSimple Virtual Array como servidor de archivos o servidor iSCSI, es preciso que haga referencia a los siguientes recursos tabulados.

#### <a name="deployment-articles"></a>Artículos de implementación

Para implementar StorSimple Virtual Array, consulte los artículos siguientes en la secuencia recomendada.

| **#** | **En este paso** | **Se hace lo siguiente…** | **Y use estos documentos.** |
| --- | --- | --- | --- |
| 1. |**Configurar Azure Portal** |Crear y configurar el servicio StorSimple Device Manager antes de aprovisionar StorSimple Virtual Array. |[Deploy StorSimple Virtual Array - Prepare the portal](storsimple-virtual-array-deploy1-portal-prep.md) (Implementación de StorSimple Virtual Array: preparar el portal) |
| 2. |**Aprovisionar la matriz virtual** |En Hyper-V, aprovisionar StorSimple Virtual Array en un sistema host que ejecuta Hyper-V en Windows Server 2012 R2, Windows Server 2012 o Windows Server 2008 R2 y conectarse a él. <br></br> <br></br> En VMware, aprovisionar StorSimple Virtual Array en un sistema host que ejecuta VMware ESXi 5.0, 5.5, 6.0 o 6.5 y conectarse a él.<br></br> |[Aprovisionamiento de una matriz virtual en Hyper-V](storsimple-virtual-array-deploy2-provision-hyperv.md) <br></br> <br></br> [Deploy StorSimple Virtual Array - Provision a Virtual Array in VMware](storsimple-virtual-array-deploy2-provision-vmware.md) (Implementación de StorSimple Virtual Array: aprovisionamiento de una matriz virtual en VMware) |
| 3. |**Configurar la matriz virtual** |Para el servidor de archivos, realice la instalación inicial, registre el servidor de archivos de StorSimple y complete la instalación del dispositivo. A continuación, puede aprovisionar los recursos compartidos de SMB. <br></br> <br></br> Para el servidor iSCSI, realice la configuración inicial, registre el servidor iSCSI de StorSimple y complete la configuración del dispositivo. A continuación, puede aprovisionar los volúmenes iSCSI. |[Deploy StorSimple Virtual Array - Set up as file server](storsimple-virtual-array-deploy3-fs-setup.md) (Implementación de StorSimple Virtual Array: configurar como servidor de archivos)<br></br> <br></br>[Implementar una matriz virtual de StorSimple: Configurar el dispositivo virtual como servidor iSCSI (versión preliminar)](storsimple-virtual-array-deploy3-iscsi-setup.md) |

Ya puede empezar a configurar Azure Portal.

## <a name="configuration-checklist"></a>Lista de comprobación de la configuración

La lista de comprobación de la configuración describe la información que se necesita recopilar antes de configurar el software en StorSimple Virtual Array. La preparación de esta información con antelación simplifica el proceso de implementación del dispositivo StorSimple en el entorno. En función de si StorSimple se StorSimple Virtual Array como servidor de archivos o como servidor iSCSI, necesitará una de las siguientes listas de comprobación.

* Descargue la [lista de comprobación de configuración del servidor de archivos de la matriz virtual de StorSimple](https://download.microsoft.com/download/E/E/6/EE690BB0-B442-4B84-8165-4731EE727ACF/MicrosoftAzureStorSimpleVirtualArrayFileServerConfigurationChecklist.pdf).
* Descargue la [lista de comprobación de configuración del servidor iSCSI de la matriz virtual de StorSimple](https://download.microsoft.com/download/E/E/6/EE690BB0-B442-4B84-8165-4731EE727ACF/MicrosoftAzureStorSimpleVirtualArrayiSCSIServerConfigurationChecklist.pdf).

## <a name="prerequisites"></a>Requisitos previos

Aquí encontrará los requisitos previos de configuración del servicio StorSimple Device Manager, StorSimple Virtual Array y la red del centro de datos.

### <a name="for-the-storsimple-device-manager-service"></a>Para el servicio StorSimple Device Manager

Antes de comenzar, asegúrese de que:

* Tiene una cuenta Microsoft con credenciales de acceso.
* Tiene una cuenta de almacenamiento de Microsoft Azure con credenciales de acceso.
* Su suscripción a Microsoft Azure debe estar habilitada para el servicio StorSimple Device Manager.

### <a name="for-the-storsimple-virtual-array"></a>Para StorSimple Virtual Array

Antes de implementar una matriz virtual, asegúrese de que:

* Tiene acceso a un sistema host que ejecuta Hyper-V en Windows Server 2008 R2 o versiones posteriores o VMware (ESXi 5.0, 5.5, 6.0 o 6.5) que puede usarse para aprovisionar un dispositivo.
* El sistema host es capaz de dedicar los recursos siguientes para aprovisionar su dispositivo virtual:
  
  * Un mínimo de 4 núcleos.
  * Al menos 8 GB de RAM. Si desea configurar la matriz virtual como servidor de archivos, 8 GB admite 2 millones de archivos. Necesita 16 GB de RAM para admitir 2-4 millones de archivos.
  * Una interfaz de red.
  * Un disco virtual de 500 GB para datos del sistema.

### <a name="for-the-datacenter-network"></a>Para la red del centro de datos

Antes de comenzar, asegúrese de que:

* La red en su centro de datos esté configurada según los requisitos de red para el dispositivo StorSimple. Para más información, consulte [Requisitos del sistema de la matriz virtual de StorSimple](storsimple-ova-system-requirements.md).
* StorSimple Virtual Array tiene un ancho de banda de Internet de 5 Mbps (o más) disponible en todo momento. Este ancho de banda no debe compartirse con otras aplicaciones.

## <a name="step-by-step-preparation"></a>Preparación de paso a paso

Utilice las siguientes instrucciones detalladas para preparar su portal para el servicio StorSimple Device Manager.

## <a name="step-1-create-a-new-service"></a>Paso 1: Creación de un nuevo servicio

Una instancia individual del servicio StorSimple Device Manager puede administrar varias instancias de StorSimple Virtual Array. Siga estos pasos para crear una instancia del servicio StorSimple Device Manager. Si tiene un servicio StorSimple Device Manager existente para administrar sus matrices virtuales, omita este paso y vaya al [Paso 2: Obtener la clave de registro del servicio](#step-2-get-the-service-registration-key).

[!INCLUDE [storsimple-virtual-array-create-new-service](../../includes/storsimple-virtual-array-create-new-service.md)]

> [!IMPORTANT]
> Si no habilitó la creación automática de una cuenta de almacenamiento con el servicio, debe crear al menos una cuenta de almacenamiento después de crear correctamente un servicio.
> 
> * Si no creó automáticamente una cuenta de almacenamiento, vaya a [Configurar una nueva cuenta de almacenamiento para el servicio](#optional-step-configure-a-new-storage-account-for-the-service) para obtener instrucciones detalladas.
> * Si habilitó la creación automática de una cuenta de almacenamiento, vaya al [Paso 2: Obtener la clave de registro del servicio](#step-2-get-the-service-registration-key).
> 
> 

## <a name="step-2-get-the-service-registration-key"></a>Paso 2: Obtener la clave de registro del servicio

Una vez que el servicio StorSimple Device Manager esté en funcionamiento, necesitará obtener la clave de registro del servicio. Esta clave se usa para registrar y conectar el dispositivo StorSimple con el servicio.

Siga estos pasos en [Azure Portal](https://portal.azure.com/).

[!INCLUDE [storsimple-virtual-array-get-service-registration-key](../../includes/storsimple-virtual-array-get-service-registration-key.md)]

> [!NOTE]
> La clave de registro del servicio se usa para registrar todos los dispositivos de StorSimple Device Manager que deben registrarse en el servicio StorSimple Device Manager.
> 
> 

## <a name="step-3-download-the-virtual-array-image"></a>Paso 3: Descargar la imagen de la matriz virtual

Una vez que tenga la clave de registro del servicio, tendrá que descargar la imagen apropiada de la matriz virtual para aprovisionar una matriz virtual en el sistema host. Las imágenes de matrices virtuales son específicas del sistema operativo y se pueden descargar de la página Inicio rápido de Azure Portal.

> [!IMPORTANT]
> El software que se ejecuta en StorSimple Virtual Array solo puede usarse junto con el servicio StorSimple Device Manager.
> 
> 

Siga estos pasos en [Azure Portal](https://portal.azure.com/).

#### <a name="to-get-the-virtual-array-image"></a>Para obtener la imagen de matriz virtual

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com/). 
2. En Azure Portal, haga clic en **Examinar > Administradores de dispositivos de StorSimple**.
3. Para un servicio StorSimple Device Manager existente. En la hoja **Administradores de dispositivos de StorSimple**, haga clic en **Inicio rápido**. 
4. Haga clic en el vínculo correspondiente a la imagen que quiere descargar del Centro de descarga de Microsoft. Los archivos de imagen tienen un tamaño aproximado de 4,8 GB.
   
   * VHDX para Hyper-V en Windows Server 2012 y versiones posteriores
   * VHD para Hyper-V en Windows Server 2008 R2 y versiones posteriores
   * VMDK para VMWare ESXi 5.0, 5.5, 6.0 o 6.5
5. Descargue y descomprima el archivo en una unidad local y tome nota de dónde se encuentra el archivo sin comprimir.

## <a name="optional-step-configure-a-new-storage-account-for-the-service"></a>Paso opcional: Configurar una nueva cuenta de almacenamiento para el servicio

Este paso es opcional y solo se debe llevar a cabo si no habilitó la creación automática de una cuenta de almacenamiento con su servicio.

Si necesita crear una cuenta de Azure Storage en una región distinta, consulte [Crear una cuenta de almacenamiento](../storage/common/storage-account-create.md) para instrucciones detalladas.

Siga estos pasos en [Azure Portal](https://ms.portal.azure.com/) en la página del servicio StorSimple Device Manager para agregar una cuenta de Microsoft Azure Storage existente.

#### <a name="to-add-a-storage-account-credential-that-has-the-same-azure-subscription-as-the-device-manager-service"></a>Agregar una credencial de cuenta de almacenamiento que tenga la misma suscripción de Azure que el servicio Device Manager

1. Vaya al servicio Device Manager, selecciónelo y haga doble clic en él. Se abrirá la hoja **Introducción**.
2. Seleccione **Credenciales de cuenta de almacenamiento** dentro de la sección **Configuración**.
3. Haga clic en **Agregar**.
4. En la hoja **Agregar una cuenta de almacenamiento**, haga lo siguiente:
   
   1. En **Suscripción**, seleccione **Actual**.
   
   2. Proporcione el nombre de la cuenta de almacenamiento de Azure.
   
   3. Seleccione **Habilitar** para crear un canal seguro para la comunicación de red entre su dispositivo de StorSimple y la nube. Seleccione **Deshabilitar** solo si está trabajando en una nube privada.
   
   4. Haga clic en **Agregar**. Recibirá una notificación cuando la cuenta de almacenamiento se cree correctamente.<br></br>
   
      ![Incorporación de una credencial de cuenta de almacenamiento existente](./media/storsimple-virtual-array-manage-storage-accounts/ova-add-storageacct.png)

## <a name="next-step"></a>Paso siguiente

El siguiente paso es aprovisionar una máquina virtual para StorSimple Virtual Array. Según el sistema operativo host, consulte las instrucciones detalladas en:

* [Aprovisionamiento de la matriz virtual de StorSimple en Hyper-V](storsimple-virtual-array-deploy2-provision-hyperv.md)
* [Aprovisionamiento de la matriz virtual de StorSimple en VMware](storsimple-virtual-array-deploy2-provision-vmware.md)

