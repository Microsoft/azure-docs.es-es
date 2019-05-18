---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 04/25/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 499aeccdf00980eeb66ac6ee06e45267fd515143
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546714"
---
Galerías de imágenes compartidas le permiten compartir las imágenes mediante RBAC. Puede utilizar RBAC para compartir imágenes dentro de su inquilino e incluso personas fuera de su inquilino. Sin embargo, si desea compartir las imágenes fuera de su inquilino de Azure a escala, debe crear un registro de aplicación para facilitar el uso compartido.  Uso de un registro de aplicación puede habilitar escenarios de uso compartidos más complejos, como: 

* Administración de imágenes compartidos cuando una compañía adquiere otra y la infraestructura de Azure se reparte entre los inquilinos independientes. 
* Los asociados de Azure administración la infraestructura de Azure en nombre de sus clientes. Personalización de imágenes se realiza en el inquilino de asociados, pero las implementaciones de infraestructura se realizarán en el inquilino del cliente. 


## <a name="create-the-app-registration"></a>Crear el registro de aplicación

Crear un registro de la aplicación que se usará ambos inquilinos para compartir los recursos de galería de imágenes.
1. Abra el [registros de aplicaciones (versión preliminar) en el portal de Azure](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType//sourceType/).    
1. Seleccione **nuevo registro** en el menú en la parte superior de la página.
1. En **nombre**, tipo *myGalleryApp*.
1. En **admite tipos de cuenta**, seleccione **cuentas en cualquier directorio de organización y las cuentas personales de Microsoft**.
1. En **URI de redireccionamiento**, tipo *https://www.microsoft.com* y, a continuación, seleccione **registrar**. Una vez creado el registro de aplicación, se abrirá la página información general.
1. En la página información general, copie el **Id. de aplicación (cliente)** y guárdelas para su uso posterior.   
1. Seleccione **certificados y secretos**y, a continuación, seleccione **nuevo secreto de cliente**.
1. En **descripción**, tipo *secreto de aplicación entre inquilinos de galería de imagen compartido*.
1. En **Expires**, deje el valor predeterminado de **en 1 año** y, a continuación, seleccione **agregar**.
1. Copie el valor del secreto y guardarlo en un lugar seguro. No podrá recuperarlo después de salir de la página.


Conceder el permiso de registro de aplicación para usar la Galería de imágenes compartidas.
1. En el portal de Azure, seleccione la Galería de imágenes compartido que desea compartir con otro inquilino.
1. Seleccione **seleccione control de acceso (IAM)** y en **Agregar asignación de roles** seleccione *agregar*. 
1. En **rol**, seleccione **lector**.
1. En **asignar acceso a:**, déjelo como **usuario, grupo o entidad de servicio de Azure AD**.
1. En **seleccione**, tipo *myGalleryApp* y selecciónelo cuando aparezca en la lista. Cuando haya terminado, seleccione **guardar**.


## <a name="give-tenant-2-access"></a>Conceder acceso de inquilino 2

Conceder acceso de inquilino 2 a la aplicación mediante la solicitud en un inicio de sesión mediante un explorador. Reemplace *<Tenant2 ID>* con el identificador del inquilino del inquilino que le gustaría compartir con la Galería de imágenes. Reemplace *< Id. de aplicación (cliente) >* con el identificador de aplicación del registro de la aplicación que creó. Cuando haya terminado de realizar los reemplazos, pegue la dirección URL en un explorador y siga las instrucciones de inicio de sesión para iniciar sesión en el inquilino 2.

```
https://login.microsoftonline.com/<Tenant 2 ID>/oauth2/authorize?client_id=<Application (client) ID>&response_type=code&redirect_uri=https%3A%2F%2Fwww.microsoft.com%2F 
```

En el [portal Azure](https://portal.azure.com) iniciar sesión como inquilino 2 y dar acceso de registro de la aplicación al grupo de recursos donde desea crear la máquina virtual.

1. Seleccione el grupo de recursos y, a continuación, seleccione **control de acceso (IAM)**. En **Agregar asignación de roles** seleccione **agregar**. 
1. En **rol**, tipo **colaborador**.
1. En **asignar acceso a:**, déjelo como **usuario, grupo o entidad de servicio de Azure AD**.
1. En **seleccione** tipo *myGalleryApp* , a continuación, selecciónelo cuando aparezca en la lista. Cuando haya terminado, seleccione **guardar**.

> [!NOTE]
> Deberá esperar a que finalice por completo que se compila y se replican para poder usar la misma imagen administrada para crear otra versión de la imagen la versión de imagen.

