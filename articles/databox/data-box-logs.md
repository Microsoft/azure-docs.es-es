---
title: Seguimiento y registro de eventos de Azure Data Box y Azure Data Box Heavy para pedidos de importación| Microsoft Docs
description: En este artículo se describe cómo realizar un seguimiento de los eventos y registrarlos en las distintas fases de un pedido de importación de Azure Data Box y Azure Data Box Heavy.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 07/10/2020
ms.author: alkohli
ms.openlocfilehash: 50dbbe3a6a1af1e73cdf1ee7f5bd3a63cf2f6a50
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498810"
---
# <a name="tracking-and-event-logging-for-your-azure-data-box-and-azure-data-box-heavy-import-order"></a>Seguimiento y registro de eventos para un pedido de importación de Azure Data Box y Azure Data Box Heavy

Un pedido de importación de Data Box o Data Box Heavy recorre los pasos siguientes: pedido, configuración, copia de datos, devolución, carga en Azure y comprobación, y eliminación de datos. Acorde a cada paso del pedido, puede realizar varias acciones para controlar el acceso al pedido, auditar los eventos, hacer un seguimiento del pedido e interpretar los distintos registros que se generan.

En la tabla siguiente se muestra un resumen de los pasos del pedido de importación de Data Box o Data Box Heavy y las herramientas disponibles para realizar un seguimiento y auditar el pedido durante cada paso.

| Fase del pedido de importación de Data Box       | Herramienta de seguimiento y auditoría                                                                        |
|----------------------------|------------------------------------------------------------------------------------------------|
| Crear pedido               | [Configuración del control de acceso en el pedido a través de RBAC](#set-up-access-control-on-the-order)                                                    |
| Pedido procesado            | [Seguimiento del pedido](#track-the-order) a través de: <ul><li> Azure portal </li><li> Sitio web del transportista </li><li>Notificaciones por correo electrónico</ul> |
| Configuración de un dispositivo              | Registro del acceso de las credenciales del dispositivo en los [registros de actividad](#query-activity-logs-during-setup)                                              |
| Copia de los datos a un dispositivo        | [Visualización de los archivos *error.xml*](#view-error-log-during-data-copy) para la copia de datos                                                             |
| Preparación para el envío            | [Inspección de los archivos BOM](#inspect-bom-during-prepare-to-ship) o los archivo de manifiesto en el dispositivo                                      |
| Carga de datos en Azure       | [Revisión de los registros de copia](#review-copy-log-during-upload-to-azure) en busca de errores durante la carga de datos en el centro de datos de Azure                         |
| Eliminación de datos del dispositivo   | [Visualización de los registros de la cadena de custodia](#get-chain-of-custody-logs-after-data-erasure), incluidos los registros de auditoría y el historial de pedidos                |

En este artículo se describen de forma detallada los distintos mecanismos o herramientas disponibles para realizar el seguimiento de un pedido de importación de Data Box o Data Box Heavy y auditarlo. La información de este artículo se aplica a pedidos de importación tanto de Data Box como de Data Box Heavy. En las secciones posteriores, todas las referencias a Data Box también se aplican a Data Box Heavy.

## <a name="set-up-access-control-on-the-order"></a>Configuración del control de acceso en el pedido

Puede controlar quién puede acceder a su pedido cuando se cree por primera vez. Configure los roles de Azure en distintos ámbitos para controlar el acceso al pedido de Data Box. Los roles de Azure determinan el tipo de acceso: lectura y escritura, solo lectura, o lectura y escritura para un subconjunto de operaciones.

Los dos roles que se pueden definir para el servicio de Azure Data Box son los siguientes:

- **Lector de Data Box**: tiene acceso de solo lectura a los pedidos, como defina el ámbito. Solo puede ver los detalles de un pedido. No puede acceder a otros detalles relacionados con las cuentas de almacenamiento ni editar los detalles del pedido, como la dirección y otros datos.
- **Colaborador de Data Box**: solo puede crear un pedido para transferir datos a una cuenta de almacenamiento determinada *si ya tiene acceso de escritura a una cuenta de almacenamiento*. Si no tiene acceso a una cuenta de almacenamiento, ni siquiera puede crear un pedido de Data Box para copiar datos a la cuenta. Este rol no define permisos relacionados con la cuenta de almacenamiento ni concede acceso a las cuentas de almacenamiento.  

Para restringir el acceso a un pedido, puede hacer lo siguiente:

- Asignar un rol en el nivel de un pedido. El usuario solo tiene los permisos que definan los roles para interactuar únicamente con ese pedido de Data Box específico.
- Asignar un rol en el nivel del grupo de recursos. El usuario tiene acceso a todos los pedidos de Data Box dentro de un grupo de recursos.

Para más información sobre el uso sugerido de RBAC, vea [Procedimientos recomendados para Azure RBAC](../role-based-access-control/best-practices.md).

## <a name="track-the-order"></a>Seguimiento del pedido

Puede realizar un seguimiento del pedido a través de Azure Portal y del sitio web del transportista. Tiene a su disposición los mecanismos siguientes para hacer un seguimiento del pedido de Data Box en cualquier momento:

- Para hacer un seguimiento del pedido cuando el dispositivo está en el centro de datos de Azure o en su entorno local, vaya a su **pedido de Data Box > Información general** en Azure Portal.

    ![Visualización del estado del pedido y el número de seguimiento](media/data-box-logs/overview-view-status-1.png)

- Para hacer un seguimiento del pedido mientras el dispositivo está en tránsito, visite el sitio web del operador regional, por ejemplo, el sitio web de UPS en EE. UU. Proporcione el número de seguimiento asociado con su pedido.
- Data Box también envía notificaciones por correo electrónico cada vez que cambia el estado del pedido en función de los correos electrónicos que se proporcionaron cuando se creó el pedido. Para obtener una lista de todos los estados de pedido de Data Box, consulte [Visualización del estado del pedido](data-box-portal-admin.md#view-order-status). Para cambiar la configuración de notificaciones asociada con el pedido, vea [Edición de los detalles de la notificación](data-box-portal-admin.md#edit-notification-details).

## <a name="query-activity-logs-during-setup"></a>Consulta de los registros de actividad durante la configuración

- Data Box llega a sus instalaciones en un estado de bloqueado. Puede usar las credenciales de dispositivo disponibles en Azure Portal para su pedido.  

    Cuando se configura Data Box, deberá saber quién ha accedido a las credenciales del dispositivo. Para averiguar quién ha accedido a la hoja **Credenciales del dispositivo** puede consultar los registros de actividad.  Cualquier acción que implique el acceso a la hoja **Detalles del dispositivo > Credenciales** se registra en los registros de actividad como una acción `ListCredentials`.

    ![Consulta de los registros de actividad](media/data-box-logs/query-activity-log-1.png)

- Cada inicio de sesión en Data Box se registra en tiempo real. Pero esta información solo está disponible en los [registros de auditoría](#audit-logs) después de que el pedido se haya completado correctamente.

## <a name="view-error-log-during-data-copy"></a>Visualización del registro de errores durante la copia de datos

Durante la copia de datos en Data Box o Data Box Heavy, se genera un archivo de error si hay algún problema con los datos copiados.

### <a name="errorxml-file"></a>Archivo error.xml

Asegúrese de que los trabajos de copia se hayan completado sin errores. Si hay errores durante el proceso de copia, descargue los registros de la página **Conectar y copiar**.

- Si copió un archivo que no tiene 512 bytes alineados a una carpeta de disco administrado en Data Box, el archivo no se carga como blob en páginas en la cuenta de almacenamiento provisional. Verá un error en los registros. Quite el archivo y copie uno que tenga 512 bytes alineados.
- Si copió un archivo VHDX, un VHD dinámico o un VHD diferenciado (estos archivos no son compatibles), verá un error en los registros.

Este es un ejemplo del archivo *error.xml* para los distintos errores al copiar a discos administrados.

```xml
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\differencing-vhd-022019.vhd</file>
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\dynamic-vhd-022019.vhd</file>
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\insidefixedvhdx-022019.vhdx</file>
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\insidediffvhd-022019.vhd</file>
```

Este es un ejemplo del archivo *error.xml* para los distintos errores al copiar a blobs en páginas.

```xml
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File100Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File786Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File513Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File10Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File500Bytes</file>
```


Este es un ejemplo del archivo *error.xml* para los distintos errores al copiar a blobs en bloques.

```xml
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\ab</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\invalid dns name</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\morethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortesting</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\testdirectory-~!@#$%^&amp;()_+{}</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\test__doubleunderscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\-startingwith-hyphen</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\Starting with Capital</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\_startingwith_underscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\55555555--4444--3333--2222--111111111111</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\1</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\11111111-_2222-_3333-_4444-_555555555555</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\test--doublehyphen</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5Ni3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMS3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMy3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_CONTROL">\InvalidUnicodeFiles\Ã.txt</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwNS3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5OS3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMi3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwNC3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5OC3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMC3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5Ny3vv70=</file>
```

Este es un ejemplo del archivo *error.xml* para los distintos errores al copiar a Azure Files.

```xml
<file error="ERROR_BLOB_OR_FILE_SIZE_LIMIT">\AzFileMorethan1TB\AzFile1.2TB</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\testdirectory-~!@#$%^&amp;()_+{}</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\55555555--4444--3333--2222--111111111111</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\-startingwith-hyphen</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\11111111-_2222-_3333-_4444-_555555555555</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\test--doublehyphen</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\ab</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\invalid dns name</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\test__doubleunderscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\morethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortesting</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\_startingwith_underscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\1</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\Starting with Capital</file>
```

En cada uno de los casos anteriores, resuelva los errores antes de continuar al paso siguiente. Para obtener más información sobre los errores recibidos durante la copia de datos a Data Box a través de los protocolos SMB o NFS, vaya a [Solución de problemas relacionados con Data Box y Data Box Heavy](data-box-troubleshoot.md). Para obtener más información sobre los errores recibidos durante la copia de datos a Data Box a través de REST, vaya a [Solución de problemas relacionados con el almacenamiento de blobs de Azure Data Box](data-box-troubleshoot-rest.md).

## <a name="inspect-bom-during-prepare-to-ship"></a>Inspección de BOM durante la preparación para el envío

Durante la preparación para el envío, se crea una lista de archivos que se conoce como la lista de materiales (BOM) o archivo de manifiesto.

- Use este archivo para comprobar los nombres reales y el número de archivos que se copiaron en el Data Box.
- Use este archivo para comprobar los tamaños reales de los archivos.
- Compruebe que *crc64* corresponde a una cadena distinta de cero. <!--A null value for crc64 indicates that there was a reparse point error)-->

Para obtener más información sobre los errores recibidos durante la preparación para el envío, vaya a [Solución de problemas relacionados con Data Box y Data Box Heavy](data-box-troubleshoot.md).

### <a name="bom-or-manifest-file"></a>El archivo de BOM o de manifiesto

El archivo de BOM o de manifiesto contiene la lista de todos los archivos que se copian en el dispositivo Data Box. El archivo de BOM contiene los nombres de archivo y los tamaños correspondientes, así como la suma de comprobación. Se crea un archivo de BOM independiente para los blobs en bloques, los blobs en páginas, Azure Files, para copiar a través de las API de REST y para la copia de discos administrados en Data Box. Puede descargar los archivos de BOM de la interfaz de usuario web local del dispositivo durante la preparación para el envío.

Estos archivos también residen en el dispositivo Data Box y se cargan en la cuenta de almacenamiento asociada en el centro de datos de Azure.

### <a name="bom-file-format"></a>Formato de archivo de BOM

El archivo de BOM o de manifiesto tiene el formato general siguiente:

`<file size = "file-size-in-bytes" crc64="cyclic-redundancy-check-string">\folder-path-on-data-box\name-of-file-copied.md</file>`

Este es un ejemplo de un manifiesto generado cuando se copiaron los datos en el recurso compartido de blobs de bloque en Data Box.

```
<file size="10923" crc64="0x51c78833c90e4e3f">\databox\media\data-box-deploy-copy-data\connect-shares-file-explorer1.png</file>
<file size="15308" crc64="0x091a8b2c7a3bcf0a">\databox\media\data-box-deploy-copy-data\get-share-credentials2.png</file>
<file size="53486" crc64="0x053da912fb45675f">\databox\media\data-box-deploy-copy-data\nfs-client-access.png</file>
<file size="6093" crc64="0xadb61d0d7c6d4deb">\databox\data-box-cable-options.md</file>
<file size="6499" crc64="0x080add29add367d9">\databox\data-box-deploy-copy-data-via-nfs.md</file>
<file size="11089" crc64="0xc3ce6b13a4fe3001">\databox\data-box-deploy-copy-data-via-rest.md</file>
<file size="7749" crc64="0xd2e346a4588e307a">\databox\data-box-deploy-ordered.md</file>
<file size="14275" crc64="0x260296e5d1b1608a">\databox\data-box-deploy-copy-data.md</file>
<file size="4077" crc64="0x2bb0a170225bceec">\databox\data-box-deploy-picked-up.md</file>
<file size="15447" crc64="0xcec0ca8527720b3c">\databox\data-box-portal-admin.md</file>
<file size="9126" crc64="0x820856b5a54321ad">\databox\data-box-overview.md</file>
<file size="10963" crc64="0x5e9a14f9f4784fd8">\databox\data-box-safety.md</file>
<file size="5941" crc64="0x8631d62fbc038760">\databox\data-box-security.md</file>
<file size="12536" crc64="0x8c8ff93e73d665ec">\databox\data-box-system-requirements-rest.md</file>
<file size="3220" crc64="0x7257a263c434839a">\databox\data-box-system-requirements.md</file>
```

Los archivos de BOM o de manifiesto también se copian en la cuenta de almacenamiento de Azure. Puede usar los archivos de BOM o de manifiesto para comprobar que los archivos cargados en Azure coinciden con los datos que se copiaron en el Data Box.

## <a name="review-copy-log-during-upload-to-azure"></a>Revisión del registro de copia durante la carga en Azure

Durante la carga de datos en Azure, se crea un registro de copia.

### <a name="copy-log"></a>Registro de copia

Para cada pedido que se procesa, el servicio Data Box crea un registro de copia en la cuenta de almacenamiento asociada. El registro de copia tiene el número total de archivos que se han cargado y el número de archivos que dieron errores durante la copia de datos de Data Box en su cuenta de almacenamiento de Azure.

Se realiza una prueba cíclica de redundancia (CRC) durante la carga en Azure. Se comparan las CRC de la copia de datos y después de que se carguen los datos. Un error de coincidencia de CRC indica que no se pudieron cargar los archivos correspondientes.

De forma predeterminada, los registros se escriben en un contenedor denominado `copylog`. Los registros se almacenan con la convención de nomenclatura siguiente:

`storage-account-name/databoxcopylog/ordername_device-serial-number_CopyLog_guid.xml`.

La ruta de acceso del registro de copia también se muestra en la hoja **Información general** del portal.

![Ruta de acceso al registro de copia en la hoja Información general cuando se completa](media/data-box-logs/copy-log-path-1.png)

### <a name="upload-completed-successfully"></a>La carga se completó correctamente 

En el ejemplo siguiente se describe el formato general de un registro de copia para una carga de Data Box que se ha completado correctamente:

```
<?xml version="1.0"?>
-<CopyLog Summary="Summary">
<Status>Succeeded</Status>
<TotalFiles>45</TotalFiles>
<FilesErrored>0</FilesErrored>
</CopyLog>
```

### <a name="upload-completed-with-errors"></a>La carga se completó con errores 

La carga en Azure también puede finalizar con errores.

![Ruta de acceso al registro de copia en la hoja Información general cuando se completa con errores](media/data-box-logs/copy-log-path-2.png)

Este es un ejemplo de un registro de copia donde la carga se completó con errores:

```xml
<ErroredEntity Path="iso\samsungssd.iso">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>409</ErrorCode>
  <ErrorMessage>The blob type is invalid for this operation.</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity Path="iso\iSCSI_Software_Target_33.iso">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>409</ErrorCode>
  <ErrorMessage>The blob type is invalid for this operation.</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><CopyLog Summary="Summary">
  <Status>Failed</Status>
  <TotalFiles_Blobs>72</TotalFiles_Blobs>
  <FilesErrored>2</FilesErrored>
</CopyLog>
```
### <a name="upload-completed-with-warnings"></a>Carga completada con advertencias

La carga en Azure se completa con advertencias si los datos tenían nombres de contenedor, blob o archivo que no se ajustaban a las convenciones de nomenclatura de Azure y los nombres se modificaron para cargar los datos en Azure.

![Ruta de acceso al registro de copia en la hoja Información general cuando se completa con advertencias](media/data-box-logs/copy-log-path-3.png)

Este es un ejemplo de un registro de copia donde los contenedores que no se ajustaban a las convenciones de nomenclatura de Azure cambiaron de nombre durante la carga de datos en Azure.

Los nuevos nombres únicos para los contenedores tienen el formato: `DataBox-GUID` y los datos del contenedor se colocan en el nuevo contenedor. El registro de copia especifica el nombre del contenedor nuevo y antiguo.

```xml
<ErroredEntity Path="New Folder">
   <Category>ContainerRenamed</Category>
   <ErrorCode>1</ErrorCode>
   <ErrorMessage>The original container/share/blob has been renamed to: DataBox-3fcd02de-bee6-471e-ac62-33d60317c576 :from: New Folder :because either the name has invalid character(s) or length is not supported</ErrorMessage>
  <Type>Container</Type>
</ErroredEntity>
```

Este es un ejemplo de un registro de copia donde los blobs o archivos que no se ajustaban a las convenciones de nomenclatura de Azure cambiaron de nombre durante la carga de datos en Azure. Los nuevos nombres de blob o de archivo se convierten en el resumen de SHA256 de la ruta de acceso relativa al contenedor y se cargan en la ruta de acceso basada en el tipo de destino. El destino puede ser blobs en bloques, blobs en páginas o Azure Files.

El `copylog` especifica el nombre de archivo o blob antiguo y el nuevo, y la ruta de acceso en Azure.

```xml
<ErroredEntity Path="TesDir028b4ba9-2426-4e50-9ed1-8e89bf30d285\Ã">
  <Category>BlobRenamed</Category>
  <ErrorCode>1</ErrorCode>
  <ErrorMessage>The original container/share/blob has been renamed to: PageBlob/DataBox-0xcdc5c61692e5d63af53a3cb5473e5200915e17b294683968a286c0228054f10e :from: Ã :because either name has invalid character(s) or length is not supported</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity Path="TesDir9856b9ab-6acb-4bc3-8717-9a898bdb1f8c\Ã">
  <Category>BlobRenamed</Category>
  <ErrorCode>1</ErrorCode>
  <ErrorMessage>The original container/share/blob has been renamed to: AzureFile/DataBox-0xcdc5c61692e5d63af53a3cb5473e5200915e17b294683968a286c0228054f10e :from: Ã :because either name has invalid character(s) or length is not supported</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity Path="TesDirf92f6ca4-3828-4338-840b-398b967d810b\Ã">
  <Category>BlobRenamed</Category>
  <ErrorCode>1</ErrorCode>
  <ErrorMessage>The original container/share/blob has been renamed to: BlockBlob/DataBox-0xcdc5c61692e5d63af53a3cb5473e5200915e17b294683968a286c0228054f10e :from: Ã :because either name has invalid character(s) or length is not supported</ErrorMessage>
  <Type>File</Type>
</ErroredEntity>
```

## <a name="get-chain-of-custody-logs-after-data-erasure"></a>Obtener la cadena de registros de custodia después de la eliminación de datos

Cuando se borren los datos de los discos de Data Box las directrices de revisión 1 de NIST SP 800-88, estará disponible la cadena de custodia de registros. Estos registros incluyen los registros de auditoría y el historial de pedidos. También se copian los archivos de manifiesto o de BOM junto con los registros de auditoría.

### <a name="audit-logs"></a>Registros de auditoría

Los registros de auditoría contienen información sobre cómo encender recursos compartidos y acceder a ellos en Data Box o Data Box Heavy cuando están fuera del centro de datos de Azure. Estos registros se encuentran en `storage-account/azuredatabox-chainofcustodylogs`.

Este es un ejemplo del registro de auditoría de un Data Box:

```
9/10/2018 8:23:01 PM : The operating system started at system time ‎2018‎-‎09‎-‎10T20:23:01.497758400Z.
9/10/2018 8:23:42 PM : An account was successfully logged on.
Subject:
    Security ID:        S-1-5-18
    Account Name:       WIN-DATABOXADMIN
    Account Domain: Workgroup
    Logon ID:       0x3E7
Logon Information:
    Logon Type:     3
    Restricted Admin Mode:  -
    Virtual Account:        No
    Elevated Token:     No
Impersonation Level:        Impersonation
New Logon:
    Security ID:        S-1-5-7
    Account Name:       ANONYMOUS LOGON
    Account Domain: NT AUTHORITY
    Logon ID:       0x775D5
    Linked Logon ID:    0x0
    Network Account Name:   -
    Network Account Domain: -
    Logon GUID:     {00000000-0000-0000-0000-000000000000}
Process Information:
    Process ID:     0x4
    Process Name:       
Network Information:
    Workstation Name:   -
    Source Network Address: -
    Source Port:        -
Detailed Authentication Information:
    Logon Process:      NfsSvr
    Authentication Package:MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Transited Services: -
    Package Name (NTLM only):   -
    Key Length:     0
This event is generated when a logon session is created. It is generated on the computer that was accessed. 
The subject fields indicate the account on the local system which requested the logon. This is most commonly a service such as the Server service, or a local process such as Winlogon.exe or Services.exe. 
The logon type field indicates the kind of logon that occurred. The most common types are 2 (interactive) and 3 (network).
The New Logon fields indicate the account for whom the new logon was created, i.e. the account that was logged on.
The network fields indicate where a remote logon request originated. Workstation name is not always available and may be left blank in some cases.
The impersonation level field indicates the extent to which a process in the logon session can impersonate.
The authentication information fields provide detailed information about this specific logon request.
    - Logon GUID is a unique identifier that can be used to correlate this event with a KDC event.
    - Transited services indicate which intermediate services have participated in this logon request.
    - Package name indicates which sub-protocol was used among the NTLM protocols.
    - Key length indicates the length of the generated session key. This will be 0 if no session key was requested.
9/10/2018 8:25:58 PM : An account was successfully logged on.
```


## <a name="download-order-history"></a>Descarga del historial de pedidos

El historial de pedidos está disponible en Azure Portal. Si el pedido está completo y se completa la limpieza del dispositivo (eliminación de datos de los discos), vaya al pedido de su dispositivo y navegue hasta **Detalles de pedido**. La opción **Descargar el historial de pedidos** está disponible. Para más información, consulte [Descargar historial de pedidos](data-box-portal-admin.md#download-order-history).

Si se desplaza por el historial de pedidos, verá:

- Información de seguimiento del operador para el dispositivo.
- Los eventos con actividad *SecureErase*. Estos eventos se corresponden con el borrado de los datos en el disco.
- Vínculos de registro de Data Box. Se presentan las rutas de acceso para los archivos de *registros de auditoría*, *registros de copia* y *BOM*.

Este es un ejemplo del registro del historial de pedidos de Azure Portal:

```
-------------------------------
Microsoft Data Box Order Report
-------------------------------
Name                                               : gus-poland                              
StartTime(UTC)                              : 9/19/2018 8:49:23 AM +00:00                       
DeviceType                                     : DataBox                                           
-------------------
Data Box Activities
-------------------
Time(UTC)                 | Activity                       | Status          | Description

9/19/2018 8:49:26 AM      | OrderCreated                   | Completed       |
10/2/2018 7:32:53 AM      | DevicePrepared                 | Completed       |
10/3/2018 1:36:43 PM      | ShippingToCustomer             | InProgress      | Shipment picked up. Local Time : 10/3/2018 1:36:43 PM at AMSTERDAM-NLD                                                                                
10/4/2018 8:23:30 PM      | ShippingToCustomer             | InProgress      | Processed at AMSTERDAM-NLD. Local Time : 10/4/2018 8:23:30 PM at AMSTERDAM-NLD                                                                        
10/4/2018 11:43:34 PM     | ShippingToCustomer             | InProgress      | Departed Facility in AMSTERDAM-NLD. Local Time : 10/4/2018 11:43:34 PM at AMSTERDAM-NLD
10/5/2018 8:13:49 AM      | ShippingToCustomer             | InProgress      | Arrived at Delivery Facility in BRIGHTON-GBR. Local Time : 10/5/2018 8:13:49 AM at LAMBETH-GBR                                                         
10/5/2018 9:13:24 AM      | ShippingToCustomer             | InProgress      | With delivery courier. Local Time : 10/5/2018 9:13:24 AM at BRIGHTON-GBR                                                                               
10/5/2018 12:03:04 PM     | ShippingToCustomer             | Completed       | Delivered - Signed for by. Local Time : 10/5/2018 12:03:04 PM at BRIGHTON-GBR                                                                          
1/25/2019 3:19:25 PM      | ShippingToDataCenter           | InProgress      | Shipment picked up. Local Time : 1/25/2019 3:19:25 PM at BRIGHTON-GBR                                                                                       
1/25/2019 8:03:55 PM      | ShippingToDataCenter           | InProgress      | Processed at BRIGHTON-GBR. Local Time : 1/25/2019 8:03:55 PM at LAMBETH-GBR                                                                            
1/25/2019 8:04:58 PM      | ShippingToDataCenter           | InProgress      | Departed Facility in BRIGHTON-GBR. Local Time : 1/25/2019 8:04:58 PM at BRIGHTON-GBR                                                                    
1/25/2019 9:06:09 PM      | ShippingToDataCenter           | InProgress      | Arrived at Sort Facility LONDON-HEATHROW-GBR. Local Time : 1/25/2019 9:06:09 PM at LONDON-HEATHROW-GBR                                                
1/25/2019 9:48:54 PM      | ShippingToDataCenter           | InProgress      | Processed at LONDON-HEATHROW-GBR. Local Time : 1/25/2019 9:48:54 PM at LONDON-HEATHROW-GBR                                                            
1/25/2019 10:30:20 PM     | ShippingToDataCenter           | InProgress      | Departed Facility in LONDON-HEATHROW-GBR. Local Time : 1/25/2019 10:30:20 PM at LONDON-HEATHROW-GBR
1/28/2019 7:11:35 AM      | ShippingToDataCenter           | InProgress      | Arrived at Delivery Facility in AMSTERDAM-NLD. Local Time : 1/28/2019 7:11:35 AM at AMSTERDAM-NLD                                                     
1/28/2019 9:07:57 AM      | ShippingToDataCenter           | InProgress      | With delivery courier. Local Time : 1/28/2019 9:07:57 AM at AMSTERDAM-NLD                                                                             
1/28/2019 1:35:56 PM      | ShippingToDataCenter           | InProgress      | Scheduled for delivery. Local Time : 1/28/2019 1:35:56 PM at AMSTERDAM-NLD                                                                            
1/28/2019 2:57:48 PM      | ShippingToDataCenter           | Completed       | Delivered - Signed for by. Local Time : 1/28/2019 2:57:48 PM at AMSTERDAM-NLD
1/29/2019 2:18:43 PM      | PhysicalVerification           | Completed       |
1/29/2019 3:49:50 PM      | DeviceBoot                     | Completed       | Appliance booted up successfully.
1/29/2019 3:49:51 PM      | AnomalyDetection               | Completed       | No anomaly detected.
2/12/2019 10:37:03 PM     | DataCopy                       | Resumed         |
2/13/2019 12:05:15 AM     | DataCopy                       | Resumed         |
2/15/2019 7:07:34 PM      | DataCopy                       | Completed       | Copy Completed.
2/15/2019 7:47:32 PM      | SecureErase                    | Started         |
2/15/2019 8:01:10 PM      | SecureErase                    | Completed       | Azure Data Box:<Device-serial-no> has been sanitized according to NIST 800-88 Rev 1.
------------------
Data Box Log Links
------------------
Account Name         : gusacct
Copy Logs Path       : databoxcopylog/gus-poland_<Device-serial-no>_CopyLog_<GUID>.xml
Audit Logs Path      : azuredatabox-chainofcustodylogs\<GUID>\<Device-serial-no>
BOM Files Path       : azuredatabox-chainofcustodylogs\<GUID>\<Device-serial-no>
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [solucionar problemas relacionados con el almacenamiento de blobs de Azure Data Box](data-box-troubleshoot.md).
