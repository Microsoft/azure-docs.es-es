---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 07/30/2019
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 8817f5c7ab53c5b0ebb0e8adf8c6f0f7b8a2ff42
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372336"
---
[Azure Files](../articles/storage/files/storage-files-introduction.md) admite la autenticación basada en identidades a través del Bloque de mensajes del servidor (SMB) mediante [Active Directory Domain Services (AD DS) local](https://docs.microsoft.com/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) y [Azure Active Directory Domain Services (Azure AD DS)](../articles/active-directory-domain-services/overview.md). En este artículo se describe la forma en que los recursos compartido de archivos de Azure pueden usar los servicios de dominio, ya sea de forma local o en Azure, para admitir el acceso basado en identidad a los recursos compartidos de archivos de Azure a través de SMB. La habilitación del acceso basado en identidad de los recursos compartidos de archivos de Azure permite reemplazar los servidores de archivos existentes por recursos compartidos de archivos de Azure sin reemplazar el servicio de directorio existente, con lo que se mantiene el acceso sin problemas de los usuarios a los recursos compartidos. 

Azure Files aplica la autorización en el acceso del usuario tanto al recurso compartido como a los niveles de directorio o archivo. La asignación de permisos de nivel de recurso compartido se puede realizar en usuarios o grupos administrados de Azure Active Directory (Azure AD) mediante el modelo de [control de acceso basado en rol (RBAC)](../articles/role-based-access-control/overview.md). Con RBAC, las credenciales que se usan para el acceso a archivos deben estar disponibles o sincronizadas con Azure AD. Puede asignar roles integrados de Azure, como el lector de recursos compartidos de SMB de datos de archivos de almacenamiento, a usuarios o grupos en Azure AD para conceder acceso de lectura a un recurso compartido de archivos de Azure.

En el nivel de directorio o archivo, Azure Files admite la conservación, la herencia y la aplicación de [DACL de Windows](https://docs.microsoft.com/windows/win32/secauthz/access-control-lists) igual que cualquier servidor de archivos de Windows. Se puede optar por mantener las DACL de Windows al copiar datos a través de SMB entre el recurso compartido de archivos existente y los recursos compartidos de archivos de Azure. Independientemente de que se tenga previsto aplicar la autorización o no, se pueden usar los recursos compartidos de archivos de Azure para hacer copias de seguridad de las ACL junto con los datos. 
