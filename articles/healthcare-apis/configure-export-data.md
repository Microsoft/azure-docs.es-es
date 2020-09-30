---
title: Definición de la configuración de exportación en Azure API for FHIR
description: En este artículo se describe cómo definir la configuración de exportación en Azure API for FHIR
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 3/5/2020
ms.author: matjazl
ms.openlocfilehash: 7ed4e471764fbf8ecc9b82a7d614ae52b97360f1
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267265"
---
# <a name="configure-export-setting-and-set-up-the-storage-account"></a>Definición de la configuración de la exportación y de la configuración de la cuenta de almacenamiento

Azure API for FHIR admite el comando $export, que le permite exportar los datos de la cuenta de Azure API for FHIR a una cuenta de almacenamiento.

Para configurar la exportación en Azure API for FHIR es necesario realizar tres pasos:

1. Habilitación de la identidad administrada en el servicio Azure API for FHIR
2. Creación de una cuenta de Azure Storage (si no se ha hecho antes) y asignación de permisos para Azure API for FHIR a la cuenta de almacenamiento
3. Selección de la cuenta de almacenamiento en Azure API for FHIR como cuenta de almacenamiento de exportación

## <a name="enabling-managed-identity-on-azure-api-for-fhir"></a>Habilitación de la identidad administrada en Azure API for FHIR

El primer paso para configurar Azure API for FHIR para la exportación es habilitar la identidad administrada en todo el sistema en el servicio. [Aquí](../active-directory/managed-identities-azure-resources/overview.md) puede leer todas las identidades administradas en Azure.

Para ello, vaya al servicio Azure API for FHIR y seleccione la hoja Identidad. Al cambiar el estado a Activado, se habilitará la identidad administrada en el servicio Azure API for FHIR.

![Habilitación de una entidad administrada](media/export-data/fhir-mi-enabled.png)

Ahora podemos pasar al siguiente paso y crear una cuenta de almacenamiento y asignar permiso a nuestro servicio.

## <a name="adding-permission-to-storage-account"></a>Concesión de permisos a la cuenta de almacenamiento

El siguiente paso de la exportación es asignar el permiso para que el servicio Azure API for FHIR escriba en la cuenta de almacenamiento.

Después de crear una cuenta de almacenamiento, vaya a la hoja Control de acceso (IAM) en Cuenta de almacenamiento y seleccione Agregar asignación de roles.

![Habilitación de una entidad administrada](media/export-data/fhir-export-role-assignment.png)

A continuación, se agregará el rol Colaborador de datos de Storage Blob a nuestro nombre de servicio.

![Habilitación de una entidad administrada](media/export-data/fhir-export-role-add.png)

Ahora estamos preparados para el paso siguiente, donde podemos seleccionar la cuenta de almacenamiento en Azure API for FHIR como cuenta de almacenamiento predeterminada para el comando $export.

## <a name="selecting-the-storage-account-for-export"></a>Selección de la cuenta de almacenamiento para el comando $export

El último paso consiste en asignar la cuenta de almacenamiento de Azure en la que Azure API for FHIR exportará los datos. Para ello, vaya a la hoja Integración en el servicio Azure API for FHIR en Azure Portal y seleccione la cuenta de almacenamiento.

![Habilitación de una entidad administrada](media/export-data/fhir-export-storage.png)

Después de estas operaciones, estamos preparados para exportar los datos mediante el comando $export.

>[!div class="nextstepaction"]
>[Configuración adicional](azure-api-for-fhir-additional-settings.md)
