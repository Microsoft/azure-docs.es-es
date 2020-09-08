---
title: Creación de un vale de soporte técnico para el trabajo de Azure Import/Export | Microsoft Docs
description: Aprenda a registrar una solicitud de soporte para los problemas relacionados con su trabajo de Import/Export.
services: storsimple
author: alkohli
ms.service: storage
ms.topic: conceptual
ms.date: 03/29/2018
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: ee30e9bba35ad39606fbe201c49e4eed3e656c97
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548281"
---
# <a name="open-a-support-ticket-for-an-importexport-job"></a>Abra un vale de soporte para un trabajo de Import/Export

Si tiene algún problema con el servicio Import/Export, puede crear una solicitud de servicio de soporte técnico. Este artículo le enseñará a:

* Crear una solicitud de soporte.
* Administrar el ciclo de vida de una solicitud de soporte técnico desde dentro del portal.

## <a name="create-a-support-request"></a>Crear una solicitud de soporte

Lleve a cabo los siguientes pasos para crear una solicitud de soporte.

1. Vaya a su trabajo de Import/Export. Vaya a la sección **Soporte técnico y solución de problemas** y luego seleccione **Nueva solicitud de soporte técnico**.
     
    ![Aspectos básicos](./media/storage-import-export-contact-microsoft-support/import-export-support1.png)
   
2. En **Nueva solicitud de soporte técnico**, seleccione **Datos básicos**. En **Datos básicos**, siga estos pasos:
    
    1. En la lista desplegable **Tipo de problema**, seleccione **Técnico**.
    2. Elija la **suscripción**.
    3. En **Servicio**, compruebe **Mis servicios**. En la lista desplegable, puede seleccionar una de las opciones: **Administración de cuentas de almacenamiento**, **Blob** o **Archivo**. 
        - Si elige **Administración de cuentas de almacenamiento**, seleccione **Recurso** y **Plan de soporte técnico**.
            ![Elección de administración de cuentas de almacenamiento](./media/storage-import-export-contact-microsoft-support/import-export-support3.png)
        - Si elige **Blob**, seleccione **Recurso**, **Nombres de contenedor** (opcional) y **Plan de soporte técnico**.
            ![Elección de blob](./media/storage-import-export-contact-microsoft-support/import-export-support2.png)
        - Si elige **Archivo**, seleccione **Recurso**, **Nombres de recurso compartido de archivo** (opcional) y **Plan de soporte técnico** ![Elegir archivo](./media/storage-import-export-contact-microsoft-support/import-export-support4.png)
    4. Seleccione **Next** (Siguiente).

3. En **Nueva solicitud de soporte técnico**, seleccione **Paso 2: Problema**. En **Problema**, siga estos pasos:
    
    1. Elija la **Gravedad** como **C - Impacto mínimo**. Soporte técnico la actualizará si es necesario.
    2. Seleccione el **Tipo de problema** como **Migración de datos**.
    3. Elija la **Categoría** como **Import/Export**.
    4. Proporcione un **Título** para el problema y más **Detalles**.
    5. Especifique la fecha y hora de inicio del problema.
    6. En **Carga de archivos**, seleccione el icono de la carpeta para navegar a cualquier otro archivo que quiera cargar.
    7. Seleccione **Compartir información de diagnóstico**.
    8. Seleccione **Next** (Siguiente).

       ![Problema](./media/storage-import-export-contact-microsoft-support/import-export-support5.png)

4. En **Nueva solicitud de soporte**, seleccione **Paso 3: Información de contacto**. En **Información de contacto**, siga estos pasos:

   1. En **Opciones de contacto**, especifique el método de contacto preferido (teléfono o correo electrónico) y el idioma. El tiempo de respuesta se selecciona automáticamente en función del plan de suscripción.
   2. En Información de contacto, especifique su nombre, correo electrónico, contacto opcional y país o región. Active la casilla **Guardar cambios de contacto para solicitudes futuras de soporte técnico**.
   3. Seleccione **Crear**.
   
       ![Información de contacto](./media/storage-import-export-contact-microsoft-support/import-export-support7.png)   

      El Soporte técnico de Microsoft usará esta información para ponerse en contacto con usted para solicitar más información y comunicar el diagnóstico y la resolución.
      Una vez que ha enviado la solicitud, un ingeniero de soporte técnico se pondrá en contacto con usted tan pronto como sea posible para procesar su solicitud.

## <a name="manage-a-support-request"></a>Administración de una solicitud de soporte técnico

Después de crear una incidencia de soporte técnico, puede administrar el ciclo de vida de la incidencia desde el portal.

#### <a name="to-manage-your-support-requests"></a>Para administrar las solicitudes de soporte técnico

1. Para tener acceso a la página de ayuda y soporte técnico, vaya a **Examinar > Ayuda y soporte técnico**.

    ![Administrar solicitudes de soporte técnico](./media/storage-import-export-contact-microsoft-support/manage-support-ticket2.png)   

2. Se muestra una lista tabular de **Solicitudes de soporte técnico recientes** en **Ayuda y soporte técnico**.

    ![Administrar solicitudes de soporte técnico](./media/storage-import-export-contact-microsoft-support/manage-support-ticket1.png) 

3. Seleccione una solicitud de soporte técnico y haga clic en ella. Puede ver el estado y los detalles de esta solicitud. Seleccione **+ Nuevo mensaje** si desea realizar un seguimiento de esta solicitud.

    ![Administrar solicitudes de soporte técnico](./media/storage-import-export-contact-microsoft-support/manage-support-ticket3.png) 


## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo [usar Azure Import/Export para transferir datos hacia y desde Azure Storage](storage-import-export-service.md).
