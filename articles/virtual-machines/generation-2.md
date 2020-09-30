---
title: Soporte técnico de Azure con máquinas virtuales de generación 2
description: Información general de compatibilidad de Azure para máquinas virtuales de generación 2
author: ju-shim
ms.service: virtual-machines
ms.subservice: sizes
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 08/28/2020
ms.author: jushiman
ms.openlocfilehash: a54de6dbfb320ca530e9c885af9ebe5d833274cc
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322856"
---
# <a name="support-for-generation-2-vms-on-azure"></a>Compatibilidad para máquinas virtuales de generación 2 en Azure

La compatibilidad para las máquinas virtuales (VM) de generación 2 ahora está disponible en Azure. No se puede cambiar la generación de una máquina virtual después de haberla creado, así que revise las consideraciones de esta página antes de elegir una generación.

Las máquinas virtuales de generación 2 admiten características clave que no se admiten en las VM de generación 1. Estas características incluyen una memoria mayor, Intel Software Guard Extensions (SGX Intel) y memoria persistente virtualizada (vPMEM). Las VM de generación 2 que se ejecutan en el entorno local también tienen algunas características que aún no se admiten en Azure. Para obtener más información, consulte la sección [Características y funcionalidades](#features-and-capabilities).

Las VM de generación 2 usan la nueva arquitectura de arranque basado en UEFI en lugar de la arquitectura basada en BIOS que utilizan las VM de generación 1. En comparación con las VM de generación 1, es posible las de generación 2 tengan tiempos de arranque e instalación mejorados. Para obtener una visión general de las VM de generación 2 y algunas de las diferencias entre la generación 1 y la generación 2, consulte [¿Debo crear una máquina virtual de generación 1 o 2 en Hyper-V?](/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v)

## <a name="generation-2-vm-sizes"></a>Tamaños de VM de generación 2

Las máquinas virtuales de primera generación son compatibles con todos los tamaños de máquina virtual de Azure (salvo con las de la serie Mv2). Azure ahora ofrece compatibilidad de generación 2 para las siguientes series de VM seleccionadas:


* [Serie B](sizes-b-series-burstable.md)
* [Serie DCsv2](dcv2-series.md)
* [DSv2-series](dv2-dsv2-series.md)
* [Serie Dsv3](dv3-dsv3-series.md)
* [Serie Dsv4](dv4-dsv4-series.md)
* [Serie Dasv4](dav4-dasv4-series.md)
* [Serie Ddsv4](ddv4-ddsv4-series.md)
* [Serie Esv3](ev3-esv3-series.md)
* [Serie Easv4](eav4-easv4-series.md)
* [Serie Fsv2](fsv2-series.md)
* [Serie GS](sizes-previous-gen.md#gs-series)
* [Serie HB](hb-series.md)
* [Serie HC](hc-series.md)
* [Serie Ls](sizes-previous-gen.md#ls-series) 
* [Serie Lsv2](lsv2-series.md)
* [Serie M](m-series.md)
* [Serie Mv2](mv2-series.md)<sup>1</sup>
* [Serie NCv2](ncv2-series.md) 
* [Serie NCv3](ncv3-series.md)
* [Serie ND](nd-series.md)
* [Serie NVv3](nvv3-series.md)

<sup>1</sup> La serie MV2 no admite imágenes de máquina virtual de generación 1 y solo admite un subconjunto de imágenes de generación 2. Consulte la [documentación de la serie MV2](mv2-series.md) para obtener más información.


## <a name="generation-2-vm-images-in-azure-marketplace"></a>Imágenes de VM de generación 2 en Azure Marketplace

Las VM de generación 2 admiten las siguientes imágenes de Marketplace:

* Windows Server 2019, 2016, 2012 R2, 2012
* Windows 10 Pro, Windows 10 Enterprise
* SUSE Linux Enterprise Server 15 SP1
* SUSE Linux Enterprise Server 12 SP4
* Ubuntu Server 16.04, 18.04, 19.04, 19.10 
* RHEL 8.1, 8.0, 7.7, 7.6, 7.5, 7.4, 7.0
* Cent OS 8.1, 8.0, 7.7, 7.6, 7.5, 7.4
* Oracle Linux 7.7, 7.7-CI

> [!NOTE]
> Los tamaños de máquina virtual específicos, como la serie MV2, solo pueden admitir un subconjunto de estas imágenes. Consulte la documentación sobre el tamaño de máquina virtual correspondiente para obtener información completa.

## <a name="on-premises-vs-azure-generation-2-vms"></a>Máquinas virtuales locales frente a VM de generación 2 de Azure

Actualmente Azure no admite algunas de las características que admite Hyper-V en el entorno local para VM de generación 2.

| Características de la generación 2                | Hyper-V local | Azure |
|-------------------------------------|---------------------|-------|
| Arranque seguro                         | :heavy_check_mark:  | :x:   |
| VM Blindada                         | :heavy_check_mark:  | :x:   |
| vTPM                                | :heavy_check_mark:  | :x:   |
| Seguridad basada en virtualización (VBS) | :heavy_check_mark:  | :x:   |
| Formato VHDX                         | :heavy_check_mark:  | :x:   |

## <a name="features-and-capabilities"></a>Características y funcionalidades

### <a name="generation-1-vs-generation-2-features"></a>Características de la generación 1 frente a la generación 2

| Característica | Generación 1 | Generación 2 |
|---------|--------------|--------------|
| Arranque             | PCAT                      | UEFI                               |
| Controladoras de disco | IDE                       | SCSI                               |
| Tamaños de VM         | Todos los tamaños de VM | [Consulte los tamaños disponibles](https://docs.microsoft.com/azure/virtual-machines/windows/generation-2#generation-2-vm-sizes). |

### <a name="generation-1-vs-generation-2-capabilities"></a>Funcionalidades de la generación 1 frente a la generación 2

| Capacidad | Generación 1 | Generación 2 |
|------------|--------------|--------------|
| Disco de SO > 2 TB                    | :x:                | :heavy_check_mark: |
| Disco personalizado/imagen/intercambiar SO         | :heavy_check_mark: | :heavy_check_mark: |
| Compatibilidad con los conjuntos de escalado de máquinas virtuales | :heavy_check_mark: | :heavy_check_mark: |
| Azure Site Recovery               | :heavy_check_mark: | :heavy_check_mark: |
| Copia de seguridad y restauración                    | :heavy_check_mark: | :heavy_check_mark: |
| Galería de imágenes compartidas              | :heavy_check_mark: | :heavy_check_mark: |
| Azure Disk Encryption             | :heavy_check_mark: | :x:                |

## <a name="creating-a-generation-2-vm"></a>Creación de una VM de generación 2

### <a name="marketplace-image"></a>Imagen de Marketplace

En Azure Portal o la CLI de Azure, puede crear VM de generación 2 a partir de una imagen de Marketplace que admita el arranque UEFI.

#### <a name="azure-portal"></a>Azure portal

A continuación se indican los pasos para crear una máquina virtual de segunda generación (Gen2) en Azure Portal.

1. Inicie sesión en Azure Portal en https://portal.azure.com.
1. Seleccione **Crear un recurso**.
1. Haga clic en **Ver todo** junto a Azure Marketplace, en el lado izquierdo.
1. Seleccione una imagen que sea compatible con Gen2.
1. Haga clic en **Crear**.
1. En la pestaña **Avanzado**, en la sección **Generación de VM**, seleccione la opción **Gen 2**.
1. En la pestaña **Datos básicos**, en **Detalles de instancia**, vaya a **Tamaño** y abra la hoja **Seleccionar un tamaño de máquina virtual**.
1. Seleccione una [máquina virtual compatible con la generación 2](#generation-2-vm-sizes).
1. Recorra el resto de las páginas para terminar de crear la máquina virtual.

![Selección de la máquina virtual Generación 1 o Generación 2](./media/generation-2/gen1-gen2-select.png)

#### <a name="powershell"></a>PowerShell

También puede usar PowerShell para crear una máquina virtual haciendo referencia directamente a la SKU de generación 1 o de generación 2.

Por ejemplo, use el siguiente cmdlet de PowerShell para obtener una lista de las SKU de la oferta `WindowsServer`.

```powershell
Get-AzVMImageSku -Location westus2 -PublisherName MicrosoftWindowsServer -Offer WindowsServer
```

Si va a crear una máquina virtual con Windows Server 2012 como sistema operativo, seleccione la SKU de máquina virtual de generación 1 (BIOS) o de generación 2 (UEFI), que tiene un aspecto similar al siguiente:

```powershell
2012-Datacenter
2012-datacenter-gensecond
```

Consulte la sección [Características y funcionalidades](#features-and-capabilities) para obtener una lista de imágenes compatibles de Marketplace.

#### <a name="azure-cli"></a>Azure CLI

También puede usar la CLI de Azure para ver las imágenes de segunda generación disponibles, organizadas por **publicador**.

```azurecli
az vm image list --publisher Canonical --sku gen2 --output table --all
```

### <a name="managed-image-or-managed-disk"></a>Imagen administrada o disco administrado

Puede crear una VM de generación 2 desde una imagen administrada o un disco administrado de la misma manera que crearía una VM de generación 1.

### <a name="virtual-machine-scale-sets"></a>Conjuntos de escalado de máquinas virtuales

También puede crear VM de generación 2 usando conjuntos de escalado de VM. En la CLI de Azure, use los conjuntos de escalado de Azure para crear VM de generación 2.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

* **¿Están disponibles las máquinas virtuales de generación 2 en todas las regiones de Azure?**  
    Sí. Pero no todos los [tamaños de VM de generación 2](#generation-2-vm-sizes) están disponibles en todas las regiones. La disponibilidad de las VM de generación 2 depende de la disponibilidad del tamaño de máquina virtual.

* **¿Hay diferencia de precio entre las VM de generación 1 y de generación 2?**  
   No.

* **Tengo un archivo .vhd de mi VM de generación 2 local. ¿Puedo usar ese archivo .vhd para crear una VM de generación 2 en Azure?**
  Sí, puede llevar el archivo .vhd de generación 2 a Azure y usarlo para crear una VM de generación 2. Para ello, siga estos pasos:
    1. Cargue el archivo . vhd en una cuenta de almacenamiento en la misma región en la que quiere crear la VM.
    1. Cree un disco administrado a partir del archivo .vhd. Establezca la propiedad de generación de Hyper-V en V2. Los siguientes comandos de PowerShell establecen la propiedad de generación de Hyper-V al crear el disco administrado.

        ```powershell
        $sourceUri = 'https://xyzstorage.blob.core.windows.net/vhd/abcd.vhd'. #<Provide location to your uploaded .vhd file>
        $osDiskName = 'gen2Diskfrmgenvhd'  #<Provide a name for your disk>
        $diskconfig = New-AzDiskConfig -Location '<location>' -DiskSizeGB 127 -AccountType Standard_LRS -OsType Windows -HyperVGeneration "V2" -SourceUri $sourceUri -CreateOption 'Import'
        New-AzDisk -DiskName $osDiskName -ResourceGroupName '<Your Resource Group>' -Disk $diskconfig
        ```

    1. Una vez que el disco esté disponible, cree una VM mediante la conexión de este disco. La VM creada será de generación 2.
    Cuando se crea la VM de generación 2, puede generalizar la imagen de esta VM de forma opcional. Al generalizar la imagen, puede usarla para crear varias máquinas virtuales.

* **¿Cómo se puede aumentar el tamaño del disco del SO?**  

  Los discos del sistema operativo mayores de 2 TB son nuevos en las máquinas virtuales de segunda generación. De forma predeterminada, los discos del sistema operativo son menores que 2 TB para las máquinas virtuales de segunda generación. Puede aumentar el tamaño del disco hasta un máximo recomendado de 4 TB. Use la CLI de Azure o Azure Portal para aumentar el tamaño del disco del SO. Para más información sobre cómo expandir los discos mediante programación, consulte el artículo sobre cómo **cambiar el tamaño de un disco** para [Windows](./windows/expand-os-disk.md) o [Linux](./linux/resize-os-disk-gpt-partition.md).

  Para aumentar el tamaño del disco del SO desde Azure Portal:

  1. En Azure Portal, vaya a la página de propiedades de la máquina virtual.
  1. Para apagar y desasignar la VM, haga clic en el botón **Detener**.
  1. En la sección **Discos**, seleccione el disco del SO que quiere aumentar.
  1. En la sección **Discos**, seleccione **Configuración** y actualice el **Tamaño** con el valor que quiera.
  1. Vuelva a la página de propiedades de la máquina virtual e **inicie** la VM.
  
  Es posible que vea una advertencia para los discos de sistema operativo mayores de 2 TB. La advertencia no se aplica a las máquinas virtuales de generación 2. Sin embargo, no se admiten los tamaños de disco de sistema operativo mayores de 4 TiB.

* **¿Las VM de generación 2 admiten las redes aceleradas?**  
    Sí. Para más información vea [Creación de una máquina virtual con redes aceleradas](../virtual-network/create-vm-accelerated-networking-cli.md).

* **¿Las máquinas virtuales de generación 2 admiten el arranque seguro o vTPM en Azure?**
    Ni las máquinas virtuales de generación 1 ni las de generación 2 de Azure admiten el arranque seguro o vTPM. 
    
* **¿Admite la generación 2 VHDX?**  
    No, las VM de generación 2 solo admiten VHD.

* **¿Las máquinas virtuales de generación 2 admiten Almacenamiento en disco Ultra de Azure?**  
    Sí.

* **¿Puedo migrar una VM de generación 1 a la generación 2?**  
    No, no se puede cambiar la generación de una VM después de crearla. Si tiene que cambiar entre varias generaciones de VM, cree una nueva máquina virtual de otra generación.

* **¿Por qué el tamaño de la máquina virtual no está habilitado en el selector de tamaño cuando intento crear una máquina virtual de Gen2?**

    Para solucionarlo, haga lo siguiente:

    1. Compruebe que la propiedad **Generación de VM** está establecida en **Gen 2** en la pestaña **Avanzado**.
    1. Compruebe que está buscando un [tamaño de máquina virtual compatible con máquinas virtuales de Gen2](#generation-2-vm-sizes).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre [las VM de generación 2 en Hyper-V](/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v).

