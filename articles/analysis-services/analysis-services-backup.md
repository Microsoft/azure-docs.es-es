---
title: Copia de seguridad y restauración de bases de datos de Azure Analysis Services | Microsoft Docs
description: En este artículo se describe cómo realizar copias de seguridad y restaurar metadatos y datos del modelo de una base de datos de Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: references_regions
ms.openlocfilehash: 66d09c2faa52cee3e94402be708d654b548c0de1
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86507002"
---
# <a name="analysis-services-database-backup-and-restore"></a>Copia de seguridad de datos y restauración de Analysis Services

Realizar una copia de seguridad de bases de datos de modelo tabular en Azure Analysis Services es casi lo mismo que en Analysis Services local. La diferencia principal es el lugar donde se almacenan los archivos de copia de seguridad. Los archivos de copia de seguridad se deben guardar en un contenedor de una [cuenta de almacenamiento de Azure](../storage/common/storage-create-storage-account.md). Puede usar una cuenta de almacenamiento y un contenedor que ya tenga o puede crear estos al configurar el almacenamiento para el servidor.

> [!NOTE]
> La creación de una cuenta de almacenamiento puede dar lugar a un nuevo servicio facturable. Para más información, consulte [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/blobs/).
> 
> 

> [!NOTE]
> Si la cuenta de almacenamiento está en otra región, configure el firewall de la cuenta de almacenamiento para permitir el acceso desde **Redes seleccionadas**. En el **intervalo de direcciones** del firewall, especifique el intervalo de direcciones IP para la región en la que se encuentra el servidor de Analysis Services. Se admite la configuración del firewall de la cuenta de almacenamiento para permitir el acceso desde todas las redes, aunque es preferible elegir Redes seleccionadas y especificar un intervalo de direcciones IP. Para más información, consulte [Preguntas frecuentes acerca de la conectividad de red](analysis-services-network-faq.md#backup-and-restore).

Las copias de seguridad se guardan con una extensión .abf. Para los modelos tabulares en memoria, se almacenan los datos y los metadatos del modelo. Para los modelos tabulares de consulta directa, solo se almacenan los metadatos del modelo. Las copias de seguridad se pueden comprimir y cifrar, según las opciones que elija.


## <a name="configure-storage-settings"></a>Configuración de los valores de almacenamiento
Antes de realizar copias de seguridad, debe configurar los valores de almacenamiento para el servidor.


### <a name="to-configure-storage-settings"></a>Para configurar los valores de almacenamiento:
1.  En el portal de Azure > **Configuración**, haga clic en **Backup**.

    ![Copias de seguridad en Configuración](./media/analysis-services-backup/aas-backup-backups.png)

2.  Haga clic en **Habilitado** y, luego, haga clic en **Configuración de Storage**.

    ![Habilitar](./media/analysis-services-backup/aas-backup-enable.png)

3. Seleccione su cuenta de almacenamiento o cree una nueva.

4. Seleccione un contenedor o cree uno nuevo.

    ![Seleccionar el contenedor](./media/analysis-services-backup/aas-backup-container.png)

5. Guarde la configuración de copia de seguridad.

    ![Guardar la configuración de copia de seguridad](./media/analysis-services-backup/aas-backup-save.png)

## <a name="backup"></a>Copia de seguridad

### <a name="to-backup-by-using-ssms"></a>Para realizar una copia de seguridad mediante SSMS:

1. En SSMS, haga clic con el botón derecho en una base de datos > **Copia de seguridad**.

2. En **Copia de seguridad de la base de datos** > **Archivo de copia de seguridad**, haga clic en **Examinar**.

3. En el cuadro de diálogo **Guardar archivo como**, compruebe la ruta de acceso de la carpeta y luego escriba un nombre para el archivo de copia de seguridad. 

4. En el cuadro de diálogo **Copia de seguridad de la base de datos**, seleccione opciones.

    **Permitir la sobrescritura de archivos**: seleccione esta opción para sobrescribir archivos de copia de seguridad del mismo nombre. Si no se selecciona esta opción, el archivo que se guarde no podrá tener el mismo nombre que un archivo que ya exista en la misma ubicación.

    **Aplicar compresión**: seleccione esta opción para comprimir el archivo de copia de seguridad. Los archivos de copia de seguridad comprimidos ahorran espacio en disco, pero requieren un uso de CPU algo mayor. 

    **Cifrar archivo de copia de seguridad**: seleccione esta opción para cifrar el archivo de copia de seguridad. Esta opción requiere una contraseña suministrada por el usuario para proteger el archivo de copia de seguridad. La contraseña impide la lectura de los datos de copia de seguridad por cualquier medio que no sea una operación de restauración. Si elige cifrar las copias de seguridad, almacene la contraseña en una ubicación segura.

5. Haga clic en **Aceptar** para crear y guardar el archivo de copia de seguridad.


### <a name="powershell"></a>PowerShell
Use el cmdlet [Backup-ASDatabase](https://docs.microsoft.com/powershell/module/sqlserver/backup-asdatabase).

## <a name="restore"></a>Restauración
Para la restauración, el archivo de copia de seguridad debe estar en la cuenta de almacenamiento que configurada para el servidor. Si debe mover un archivo de copia de seguridad de una ubicación local a la cuenta de almacenamiento, utilice [Microsoft Azure Storage Explorer](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer) o la utilidad de línea de comandos [AzCopy](../storage/common/storage-use-azcopy.md). 



> [!NOTE]
> Si va a restaurar desde un servidor local, debe quitar todos los usuarios del dominio de los roles del modelo y agregarlos de nuevo a los roles como usuarios de Azure Active Directory.
> 
> 

### <a name="to-restore-by-using-ssms"></a>Para realizar la restauración con SSMS:

1. En SSMS, haga clic con el botón derecho en una base de datos > **Restaurar**.

2. En el cuadro de diálogo **Copia de seguridad de la base de datos**, en **Archivo de copia de seguridad**, haga clic en **Examinar**.

3. En el cuadro de diálogo **Buscar archivos de base de datos**, seleccione el archivo que quiere restaurar.

4. En **Restaurar base de datos**, seleccione la base de datos.

5. Especifique las opciones. Las opciones de seguridad deben coincidir con las opciones de copia de seguridad que usó al realizar copias de seguridad.


### <a name="powershell"></a>PowerShell

Use el cmdlet [Restore-ASDatabase](https://docs.microsoft.com/powershell/module/sqlserver/restore-asdatabase).


## <a name="related-information"></a>Información relacionada

[Cuentas de almacenamiento de Azure](../storage/common/storage-create-storage-account.md)  
[Alta disponibilidad](analysis-services-bcdr.md)      
[Preguntas frecuentes acerca de la conectividad de red de Analysis Services](analysis-services-network-faq.md)