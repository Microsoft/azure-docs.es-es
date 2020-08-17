---
title: Movimiento de una instancia de EC2 Windows de AWS a Azure
description: Migre una instancia de EC2 Windows de Amazon Web Services (AWS) a una máquina virtual Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 06/01/2018
ms.author: cynthn
ms.openlocfilehash: 9f66653996cb36e58054756ac877f859559609be
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87267105"
---
# <a name="move-a-windows-vm-from-amazon-web-services-aws-to-an-azure-virtual-machine"></a>Migración de una máquina virtual Windows de Amazon Web Services (AWS) a una máquina virtual Azure

Si va a evaluar las máquinas virtuales de Azure para hospedar las cargas de trabajo, puede exportar una instancia existente de la máquina virtual Windows de Amazon Web Services (AWS) EC2 y cargar el disco duro virtual (VHD) en Azure. Una vez cargado el disco duro virtual, puede crear una nueva máquina virtual en Azure desde el disco duro virtual. 

En este artículo se cubre la migración de una sola máquina virtual de AWS a Azure. Si quiere migrar máquinas virtuales de AWS a Azure en escala, consulte [Migrar máquinas virtuales de Amazon Web Services (AWS) a Azure con Azure Site Recovery](../../site-recovery/migrate-tutorial-aws-azure.md).

## <a name="prepare-the-vm"></a>Preparación de la VM 
 
Puede cargar VHD generalizados y especializados en Azure. Todos requieren la preparación de la máquina virtual antes de la exportación desde AWS. 

- **VHD generalizado**: Se ha quitado toda la información de cuenta personal con Sysprep de un VHD generalizado. Si tiene previsto usar VHD como una imagen desde la que crear nuevas VM, debe: 
 
    * [Preparación de una máquina virtual Windows](prepare-for-upload-vhd-image.md).  
    * Generalice la máquina virtual mediante Sysprep.  

 
- **VHD especializado**: un disco duro virtual especializado mantiene las cuentas de usuario, las aplicaciones y otros datos de estado de la máquina virtual original. Si tiene previsto usar el VHD como está para crear una nueva VM, asegúrese de completar los pasos siguientes.  
    * [Preparar de un VHD de Windows para cargar en Azure](prepare-for-upload-vhd-image.md). **No** generalice la VM mediante Sysprep. 
    * Quite todas las herramientas de virtualización de invitado y los agentes instalados en la VM (es decir, herramientas de VMware). 
    * Asegúrese de que la VM se configura para extraer su dirección IP y la configuración de DNS a través de DHCP. Esto garantiza que el servidor obtiene una dirección IP dentro de la red virtual cuando se inicia.  


## <a name="export-and-download-the-vhd"></a>Exportación y descarga del disco duro virtual 

Exporte la instancia de EC2 a un disco duro virtual de un cubo de Amazon S3. Siga los pasos del artículo [Exporting an Instance as a VM Using VM Import/Export](https://docs.aws.amazon.com/vm-import/latest/userguide/vmexport.html) (Exportación de una instancia como una máquina virtual con la importación/exportación de máquinas virtuales) de la documentación de Amazon y ejecute el comando [create-instance-export-task](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-instance-export-task.html) para exportar la instancia de EC2 a un archivo VHD. 

El archivo VHD exportado se guarda en el cubo de Amazon S3 que haya especificado. La sintaxis básica para exportar el disco duro virtual se encuentra justo debajo, solo tiene que reemplazar el texto del marcador de posición de \<brackets> con su información.

```
aws ec2 create-instance-export-task --instance-id <instanceID> --target-environment Microsoft \
  --export-to-s3-task DiskImageFormat=VHD,ContainerFormat=ova,S3Bucket=<bucket>,S3Prefix=<prefix>
```

Una vez exportado el disco duro virtual, siga las instrucciones de [How Do I Download an Object from an S3 Bucket?](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/download-objects.html) (Descarga de objetos desde un cubo de S3) para descargar el archivo VHD desde el cubo de S3. 

> [!IMPORTANT]
> Por la descarga del disco duro virtual, AWS cobra una tarifa de transferencia de datos. Consulte [Precios de Amazon S3](https://aws.amazon.com/s3/pricing/) para más información.


## <a name="next-steps"></a>Pasos siguientes

Ahora puede cargar el disco duro virtual en Azure y crear una máquina virtual. 

- Si ha ejecutado Sysprep en el origen para la **generalización** antes de exportarlo, consulte el artículo de [Carga de un disco duro virtual generalizado y creación de máquinas virtuales en Azure con él](upload-generalized-managed.md)
- Si no ejecutó Sysprep antes de la exportación, se considera que el disco duro virtual es **especializado**; consulte el artículo de [Carga de un disco duro virtual especializado en Azure y creación de máquinas virtuales con él](create-vm-specialized.md)

 
