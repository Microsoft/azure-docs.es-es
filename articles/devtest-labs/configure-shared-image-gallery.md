---
title: Configuración de una galería de imágenes compartidas en Azure DevTest Labs | Microsoft Docs
description: Aprenda a configurar una galería de imágenes compartidas en Azure DevTest Labs, que permite a los usuarios acceder a imágenes desde una ubicación compartida mientras se crean recursos de laboratorio.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 7ec08fa741c1b52d3dd1d1e2b4247d3689190020
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88271046"
---
# <a name="configure-a-shared-image-gallery-in-azure-devtest-labs"></a>Configuración de una galería de imágenes compartidas en Azure DevTest Labs
DevTest Labs ofrece ahora la característica [Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md). Esta característica permite que los usuarios de laboratorios accedan a imágenes de una ubicación compartida cuando crean recursos de laboratorio. También facilita la estructuración y la organización de las imágenes de máquina virtual administradas y personalizadas. La característica Shared Image Gallery admite lo siguiente:

- Replicación administrada global de imágenes.
- Control de versiones y agrupación de imágenes para facilitar su administración.
- Alta disponibilidad para sus imágenes con cuentas de almacenamiento con redundancia de zona (ZRS) en las regiones que admitan zonas de disponibilidad. ZRS ofrece mejor resistencia a errores de zona.
- Uso compartido entre suscripciones e, incluso, entre inquilinos, con el control de acceso basado en rol (RBAC).

Para más información, consulte la [documentación de Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md). 
 
Si tiene un gran número de imágenes administradas que se deben mantener y quiere que estén disponibles en toda la empresa, puede usar una galería de imágenes compartidas como repositorio que facilite la tarea de actualizar y compartir las imágenes. Como propietario de un laboratorio, puede asociar al laboratorio una galería de imágenes compartidas que ya exista. Una vez asociada la galería, los usuarios del laboratorio pueden crear máquinas con estas últimas imágenes. Una ventaja fundamental de esta característica es que ahora DevTest Labs puede aprovechar el uso compartido de imágenes entre laboratorios, suscripciones y regiones. 

> [!NOTE]
> Para más información sobre los costos asociados al servicio Shared Image Gallery, consulte [Facturación para Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md#billing).

## <a name="considerations"></a>Consideraciones
- Solo es posible asociar una galería de imágenes compartidas a un laboratorio cada vez. Si quiere asociar otra galería, deberá desasociar la existente y asociar otra. 
- DevTest Labs no admite actualmente la carga de imágenes en la galería mediante el laboratorio. 
- Al crear una máquina virtual mediante una imagen de la galería de imágenes compartidas, DevTest Labs usa siempre la última versión publicada de esta imagen. Sin embargo, si una imagen tiene varias versiones, el usuario puede optar por crear una máquina a partir de una versión anterior. Para ello, vaya a la pestaña Configuración avanzada durante la creación de la máquina virtual.  
- Aunque DevTest Labs hace automáticamente todo lo que puede por garantizar que la galería de imágenes compartidas replica las imágenes en la región en la que existe el laboratorio, no siempre resulta posible. Para evitar que los usuarios tengan problemas al crear máquinas virtuales con estas imágenes, asegúrese de que las imágenes ya se hayan replicado en la región del laboratorio.

## <a name="use-azure-portal"></a>Usar Azure Portal
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Todos los servicios** en el menú de navegación izquierdo.
1. Seleccione **DevTest Labs** en la lista.
1. En la lista de laboratorios, seleccione el **suyo**.
1. Seleccione **Configuración y directivas** en la sección **Configuración** en el menú de la izquierda.
1. Seleccione **Galerías de imágenes compartidas** en **Bases para máquinas virtuales** en el menú izquierdo.

    ![Menú Galerías de imágenes compartidas](./media/configure-shared-image-gallery/shared-image-galleries-menu.png)
1. Asocie una galería de imágenes compartidas existente al laboratorio; para ello, haga clic en el botón **Asociar** y seleccione su galería en la lista desplegable.

    ![Attach](./media/configure-shared-image-gallery/attach-options.png)
1. Después de adjuntar la galería de imágenes, selecciónela para ir a la galería adjunta. Configure la galería para **habilitar o deshabilitar** imágenes compartidas para la creación de VM. Seleccione una galería de imágenes de la lista para configurarla. 

    De forma predeterminada **Permitir que se usen todas las imágenes como bases de máquinas virtuales** está establecido en **Sí**. Significa que todas las imágenes disponibles de la galería de imágenes compartidas asociadas estarán disponibles para un usuario de laboratorio al crear una nueva máquina virtual de laboratorio. Si es necesario restringir el acceso a determinadas imágenes, cambie **Permitir que se usen todas las imágenes como bases de máquinas virtuales** a **No**, seleccione las imágenes que desea permitir al crear máquinas virtuales y, después, seleccione el botón **Guardar**.

    :::image type="content" source="./media/configure-shared-image-gallery/enable-disable.png" alt-text="Habilitación o deshabilitación de imágenes":::

    > [!NOTE]
    > Se admiten tanto imágenes generalizadas como especializadas en la galería de imágenes compartidas. 
1. Los usuarios del laboratorio pueden crear entonces una máquina virtual con las imágenes habilitadas con solo hacer clic en **+ Agregar** y buscar la imagen en la página **choose your base** (Elegir la base).

    ![Usuarios de laboratorios](./media/configure-shared-image-gallery/lab-users.png)
## <a name="use-azure-resource-manager-template"></a>Usar plantillas de Azure Resource Manager

### <a name="attach-a-shared-image-gallery-to-your-lab"></a>Asociación de una galería de imágenes compartidas al laboratorio
Si va a usar una plantilla de Azure Resource Manager para asociar una galería de imágenes compartidas al laboratorio, deberá agregarla en la sección de recursos de la plantilla de Resource Manager, como se muestra en el ejemplo siguiente:

```json
"resources": [
{
    "apiVersion": "2018-10-15-preview",
    "type": "Microsoft.DevTestLab/labs",
    "name": "mylab",
    "location": "eastus",
    "resources": [
    {
        "apiVersion":"2018-10-15-preview",
        "name":"myGallery",
        "type":"sharedGalleries",
        "properties": {
            "galleryId":"/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/mySharedGalleryRg/providers/Microsoft.Compute/galleries/mySharedGallery",
            "allowAllImages": "Enabled"
        }
    }
    ]
}
```

Para ver un ejemplo completo de una plantilla de Resource Manager, consulte estos ejemplos de plantilla de Resource Manager en nuestro repositorio público de GitHub: [Configuración de una galería de imágenes compartidas al crear un laboratorio](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates/101-dtl-create-lab-shared-gallery-configured).

## <a name="use-rest-api"></a>Use la API de REST

### <a name="get-a-list-of-labs"></a>Obtención de una lista de laboratorios 

```rest
GET  https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DevTestLab/labs?api-version= 2018-10-15-preview
```

### <a name="get-the-list-of-shared-image-galleries-associated-with-a-lab"></a>Obtención de la lista de galerías de imágenes compartidas asociadas a un laboratorio

```rest
GET  https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DevTestLab/labs/{labName}/sharedgalleries?api-version= 2018-10-15-preview
   ```

### <a name="create-or-update-shared-image-gallery"></a>Creación o actualización de una galería de imágenes compartidas

```rest
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DevTestLab/labs/{labName}/sharedgalleries/{name}?api-version= 2018-10-15-preview
Body: 
{
    "properties":{
        "galleryId": "[Shared Image Gallery resource Id]",
        "allowAllImages": "Enabled"
    }
}

```

### <a name="list-images-in-a-shared-image-gallery"></a>Listado de imágenes en una galería de imágenes compartidas

```rest
GET  https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DevTestLab/labs/{labName}/sharedgalleries/{name}/sharedimages?api-version= 2018-10-15-preview
```



## <a name="next-steps"></a>Pasos siguientes
Consulte los siguientes artículos sobre la creación de una máquina virtual mediante una imagen de la galería de imágenes compartidas adjunta: [Creación de una máquina virtual con una imagen compartida de la galería](add-vm-use-shared-image.md)
