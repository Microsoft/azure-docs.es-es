---
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: include
ms.date: 06/05/2020
ms.author: alkohli
ms.openlocfilehash: 5aaf0ce747b14b2fa9f2fcd9a65b774aa7d2db3b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87102999"
---
En función de la cuenta de almacenamiento seleccionada, Data Box crea hasta:

* Tres recursos compartidos para cada cuenta de almacenamiento asociada (GPv1 y GPv2).
* Un recurso compartido para Premium Storage.
* Un recurso compartido para una cuenta de Blob Storage.

En los recursos compartidos de blob en bloques y en páginas, las entidades de primer nivel son contenedores y las entidades de segundo nivel son blobs. En los recursos compartidos de Azure Files, las entidades de primer nivel son los recursos compartidos y las entidades de segundo nivel son los archivos.

En la tabla siguiente se muestra la ruta de acceso UNC a los recursos compartidos en la dirección URL de la ruta de acceso de Data Box y Azure Storage donde se cargan los datos. La dirección URL final de la ruta de acceso de Azure Storage se puede derivar a partir de la ruta de acceso UNC al recurso compartido.
 
| Blobs y archivos | Rutas de acceso y direcciones URL |
| --------------- | -------------- |
| Blobs en bloques de Azure | <li>Ruta de acceso UNC a recursos compartidos: `\\<DeviceIPAddress>\<StorageAccountName_BlockBlob>\<ContainerName>\files\a.txt`</li><li>Dirección URL de Azure Storage: `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li> |  
| Blobs en páginas de Azure  | <li>Ruta de acceso UNC a recursos compartidos: `\\<DeviceIPAddres>\<StorageAccountName_PageBlob>\<ContainerName>\files\a.txt`</li><li>Dirección URL de Azure Storage: `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li>   |  
| Azure Files       |<li>Ruta de acceso UNC a recursos compartidos: `\\<DeviceIPAddres>\<StorageAccountName_AzFile>\<ShareName>\files\a.txt`</li><li>Dirección URL de Azure Storage: `https://<StorageAccountName>.file.core.windows.net/<ShareName>/files/a.txt`</li>        |      

