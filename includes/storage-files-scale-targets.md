---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 05/06/2019
ms.author: tamram
ms.openlocfilehash: 213ecee34df46c0a408e7034a07ab864c96b340b
ms.sourcegitcommit: 0ae3139c7e2f9d27e8200ae02e6eed6f52aca476
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2019
ms.locfileid: "65751531"
---
| Recurso | Recursos compartidos de archivos estándar | Recursos compartidos de archivos Premium (versión preliminar) |
|----------|---------------|------------------------------------------|
| Tamaño máximo de un recurso compartido de archivos | No hay ningún mínimo; pago por uso | 100 GB; aprovisionado |
| Tamaño máximo de un recurso compartido de archivos | 5 TiB | 100 TB |
| Tamaño máximo de un archivo en un recurso compartido de archivos | 1 TiB | 1 TiB |
| Número máximo de archivos en un recurso compartido de archivos | Sin límite | Sin límite |
| Número máximo de IOPS por recurso compartido | 1000 E/S POR SEGUNDO | 100.000 IOPS |
| Número máximo de directivas de acceso almacenadas por cada archivo de recurso compartido | 5 | 5 |
| Rendimiento de un único recurso compartido de destino | Hasta 60 MiB/s | Vea los valores de entrada y salida de la cuota del archivo premium|
| Salida máxima para un recurso compartido de archivo único | Ver el rendimiento de destino de recurso compartido de archivos estándar | Hasta 6,204 MiB/s |
| Entrada máxima para un recurso compartido de archivo único | Ver el rendimiento de destino de recurso compartido de archivos estándar | Hasta 4.136 MiB/s |
| Número máximo de identificadores abiertos por archivo | 2.000 identificadores abiertos | 2.000 identificadores abiertos |
| Número máximo de instantáneas de recurso compartido | 200 instantáneas de recurso compartido | 200 instantáneas de recurso compartido |
| Longitud máxima del nombre de objeto (archivos y directorios) | 2.048 caracteres | 2.048 caracteres |
| Número máximo de componentes de la ruta de acceso (en la ruta de acceso \A\B\C\D, cada letra es un componente) | 255 caracteres | 255 caracteres |