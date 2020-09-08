---
title: Reparación de un trabajo de exportación de Azure Import/Export (versión 1) | Microsoft Docs
description: Obtenga información sobre cómo reparar un trabajo de exportación que se creó y ejecutó con el servicio Azure Import/Export.
author: twooley
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 01/23/2017
ms.author: twooley
ms.subservice: common
ms.openlocfilehash: 0731848e1ff187afb6e9f607516dd74b6c16de9b
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88520488"
---
# <a name="repairing-an-export-job"></a>Reparación de un trabajo de exportación
Cuando haya finalizado un trabajo de exportación, puede ejecutar la herramienta Microsoft Azure Import/Export de manera local para:  
  
1.  Descargar los archivos que el servicio Azure Import/Export no pudo exportar.  
  
2.  Validar que los archivos de la unidad se exportaron correctamente.  
  
Debe tener conectividad con Azure Storage para utilizar esta funcionalidad.  
  
El comando para reparar un trabajo de importación es **RepairExport**.

## <a name="repairexport-parameters"></a>Parámetros RepairExport

Se pueden modificar los parámetros siguientes con **RepairExport**:  
  
|Parámetro|Descripción|  
|---------------|-----------------|  
|**/r:&lt;ArchivoReparación\>**|Necesario. Ruta de acceso al archivo de reparación, que realiza un seguimiento del progreso de la reparación y le permite reanudar una reparación interrumpida. Cada unidad debe tener un solo archivo de reparación. Cuando inicie una reparación para una unidad determinada, se pasa la ruta de acceso a un archivo de reparación, que aún no existe. Para reanudar una reparación interrumpida, debe pasar el nombre de un archivo de reparación existente. Especifique siempre el archivo de reparación correspondiente a la unidad de destino.|  
|**/logdir:&lt;DirectorioRegistro\>**|Opcional. Directorio de registro. Los archivos de registro detallados se escribirán en este directorio. Si no se especifica un directorio de registro, se usará el directorio actual como directorio de registro.|  
|**/d:&lt;DirectorioDestino\>**|Necesario. El directorio para validar y reparar. Suele ser el directorio raíz de la unidad de exportación, pero podría también ser un recurso compartido de archivos de red que contiene una copia de los archivos exportados.|  
|**/bk:&lt;ClaveBitLocker\>**|Opcional. Especifique la clave de BitLocker si desea que la herramienta desbloquee una unidad de cifrado donde se almacenan los archivos exportados.|  
|**/sn:&lt;NombreCuentaAlmacenamiento\>**|Necesario. El nombre de la cuenta de almacenamiento para el trabajo de exportación.|  
|**/sk:&lt;ClaveCuentaAlmacenamiento\>**|**Necesario** únicamente si no se especifica un SAS del contenedor. La clave de cuenta para la cuenta de almacenamiento correspondiente al trabajo de exportación.|  
|**/csas:&lt;SasContenedor\>**|**Necesario** únicamente si no se especifica la clave de cuenta de almacenamiento. El SAS del contenedor para acceder a los blobs asociados al trabajo de exportación.|  
|**/CopyLogFile:&lt;ArchivoRegistroCopiaUnidad\>**|Necesario. La ruta de acceso al archivo de registro de copia de la unidad. El archivo lo genera el servicio Microsoft Windows Import/Export y se puede descargar desde el almacenamiento de blobs asociado al trabajo. El archivo de registro de copia contiene información sobre blobs con errores o archivos que deben repararse.|  
|**/ManifestFile:&lt;ArchivoManifiestoUnidad\>**|Opcional. La ruta de acceso al archivo de manifiesto de la unidad de exportación. Este archivo lo genera el servicio Microsoft Azure Import/Export y se almacena en la unidad de exportación. Opcionalmente, en un blob de la cuenta de almacenamiento asociada al trabajo.<br /><br /> El contenido de los archivos de la unidad de exportación se comprobará con los hash MD5 contenidos en este archivo. Todos los archivos dañados se descargarán y reescribirán en los directorios de destino.|  
  
## <a name="using-repairexport-mode-to-correct-failed-exports"></a>Uso del modo RepairExport para corregir las exportaciones con error  
Puede utilizar la herramienta Azure Import/Export para descargar los archivos que no se pudieron exportar. El archivo de registro de copia contendrá una lista de archivos que no se pudieron exportar.  
  
Las causas de errores de exportación incluyen las siguientes posibilidades:  
  
-   Unidades dañadas  
  
-   La clave de cuenta de almacenamiento cambiada durante el proceso de transferencia  
  
Para ejecutar la herramienta en modo **RepairExport**, primero debe conectar la unidad de disco que contiene los archivos exportados al equipo. A continuación, ejecute la herramienta Azure Import/Export, especificando la ruta de acceso a esa unidad con el parámetro `/d`. También debe especificar la ruta de acceso al archivo de registro de copia de la unidad que descargó. En el siguiente ejemplo de línea de comandos se ejecuta la herramienta para reparar todos los archivos que no se pudieron exportar:  
  
```  
WAImportExport.exe RepairExport /r:C:\WAImportExport\9WM35C3U.rep /d:G:\ /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C3U.log  
```  
  
A continuación, se muestra un ejemplo de un archivo de registro de copia que muestra que un bloque del blob no se pudo exportar:  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveLog>  
  <DriveId>9WM35C2V</DriveId>  
  <Blob Status="CompletedWithErrors">  
    <BlobPath>pictures/wild/desert.jpg</BlobPath>  
    <FilePath>\pictures\wild\desert.jpg</FilePath>  
    <LastModified>2012-09-18T23:47:08Z</LastModified>  
    <Length>163840</Length>  
    <BlockList>  
      <Block Offset="65536" Length="65536" Id="AQAAAA==" Status="Failed" />  
    </BlockList>  
  </Blob>  
  <Status>CompletedWithErrors</Status>  
</DriveLog>  
```  
  
El archivo de registro de copia indica que se produjo un error mientras el servicio Microsoft Windows Import/Export descargaba uno de los bloques del blob en el archivo de la unidad de exportación. Los demás componentes del archivo se descargaron correctamente y la longitud del archivo se estableció correctamente. En este caso, la herramienta abrirá el archivo en la unidad, descargará el bloque de la cuenta de almacenamiento y lo escribirá en el intervalo de archivo a partir del desplazamiento 65536 con la longitud 65536.  
  
## <a name="using-repairexport-to-validate-drive-contents"></a>Uso de RepairExport para validar el contenido de la unidad  
También puedes usar la importación y exportación de Azure con la opción **RepairExport** para validar que el contenido de la unidad es correcto. El archivo de manifiesto de cada unidad de exportación contiene MD5 para el contenido de la unidad.  
  
El servicio Azure Import/Export también puede guardar los archivos de manifiesto en una cuenta de almacenamiento durante el proceso de exportación. La ubicación de los archivos de manifiesto estará disponible a través de la operación [Get Job](/rest/api/storageimportexport/jobs) cuando se haya completado el trabajo. Para más información sobre el formato de un archivo de manifiesto de unidad, vea [Formato del archivo de manifiesto del servicio Import/Export](storage-import-export-file-format-metadata-and-properties.md).  
  
En el ejemplo siguiente se muestra cómo ejecutar la herramienta Azure Import/Export con los parámetros **/MANIFESTFILE** y **/CopyLogFile**:  
  
```  
WAImportExport.exe RepairExport /r:C:\WAImportExport\9WM35C3U.rep /d:G:\ /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C3U.log /ManifestFile:G:\9WM35C3U.manifest  
```  
  
En el ejemplo siguiente se muestra un archivo de manifiesto:  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveManifest Version="2011-10-01">  
  <Drive>  
    <DriveId>9WM35C3U</DriveId>  
    <ClientCreator>Windows Azure Import/Export service</ClientCreator>  
    <BlobList>
      <Blob>  
        <BlobPath>pictures/city/redmond.jpg</BlobPath>  
        <FilePath>\pictures\city\redmond.jpg</FilePath>  
        <Length>15360</Length>  
        <PageRangeList>  
          <PageRange Offset="0" Length="3584" Hash="72FC55ED9AFDD40A0C8D5C4193208416" />  
          <PageRange Offset="3584" Length="3584" Hash="68B28A561B73D1DA769D4C24AA427DB8" />  
          <PageRange Offset="7168" Length="512" Hash="F521DF2F50C46BC5F9EA9FB787A23EED" />  
        </PageRangeList>  
        <PropertiesPath Hash="E72A22EA959566066AD89E3B49020C0A">\pictures\city\redmond.jpg.properties</PropertiesPath>  
      </Blob>  
      <Blob>  
        <BlobPath>pictures/wild/canyon.jpg</BlobPath>  
        <FilePath>\pictures\wild\canyon.jpg</FilePath>  
        <Length>10884</Length>  
        <BlockList>  
          <Block Offset="0" Length="2721" Id="AAAAAA==" Hash="263DC9C4B99C2177769C5EBE04787037" />  
          <Block Offset="2721" Length="2721" Id="AQAAAA==" Hash="0C52BAE2CC20EFEC15CC1E3045517AA6" />  
          <Block Offset="5442" Length="2721" Id="AgAAAA==" Hash="73D1CB62CB426230C34C9F57B7148F10" />  
          <Block Offset="8163" Length="2721" Id="AwAAAA==" Hash="11210E665C5F8E7E4F136D053B243E6A" />  
        </BlockList>  
        <PropertiesPath Hash="81D7F81B2C29F10D6E123D386C3A4D5A">\pictures\wild\canyon.jpg.properties</PropertiesPath>  
      </Blob> 
    </BlobList>  
 </Drive>  
</DriveManifest>  
``` 
  
Después de finalizar el proceso de reparación, la herramienta leerá por completo los archivos a los que hace se referencia en el archivo de manifiesto y comprobará la integridad de dichos archivos con los hash MD5. Para el manifiesto anterior, recorrerá los siguientes componentes.  

```  
G:\pictures\city\redmond.jpg, offset 0, length 3584  
  
G:\pictures\city\redmond.jpg, offset 3584, length 3584  
  
G:\pictures\city\redmond.jpg, offset 7168, length 3584  
  
G:\pictures\city\redmond.jpg.properties  
  
G:\pictures\wild\canyon.jpg, offset 0, length 2721  
  
G:\pictures\wild\canyon.jpg, offset 2721, length 2721  
  
G:\pictures\wild\canyon.jpg, offset 5442, length 2721  
  
G:\pictures\wild\canyon.jpg, offset 8163, length 2721  
  
G:\pictures\wild\canyon.jpg.properties  
```

Cualquier componente que no supere la comprobación lo descargará la herramienta y se reescribirá en el mismo archivo en la unidad.  
  
## <a name="next-steps"></a>Pasos siguientes
 
* [Configuración de la herramienta Azure Import/Export](storage-import-export-tool-setup-v1.md)   
* [Preparación de unidades de disco duro para un trabajo de importación](../storage-import-export-tool-preparing-hard-drives-import-v1.md)   
* [Revisión del estado del trabajo con archivos de registro de copia](storage-import-export-tool-reviewing-job-status-v1.md)   
* [Reparación de un trabajo de importación](storage-import-export-tool-repairing-an-import-job-v1.md)
