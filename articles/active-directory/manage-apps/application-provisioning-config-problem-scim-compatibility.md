---
title: Problemas conocidos y soluciones con el cumplimiento de protocolo SCIM 2.0 del servicio de aprovisionamiento de usuarios de Azure AD | Microsoft Docs
description: Cómo solucionar los problemas comunes de compatibilidad de protocolo a los que nos enfrentamos cuando agregamos una aplicación situada fuera de la galería compatible con SCIM 2.0 en Azure AD
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 12/03/2018
ms.author: mimart
ms.reviewer: asmalser
ms.collection: M365-identity-device-management
ms.openlocfilehash: 952853b32b5fadb980e9e96a7c56c05ac63d9931
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784606"
---
# <a name="known-issues-and-resolutions-with-scim-20-protocol-compliance-of-the-azure-ad-user-provisioning-service"></a>Problemas conocidos y soluciones con el cumplimiento de protocolo SCIM 2.0 del servicio de aprovisionamiento de usuarios de Azure AD

Azure Active Directory (Azure AD) puede aprovisionar automáticamente a usuarios y grupos de cualquier aplicación o sistema dirigidos por un servicio web con la interfaz definida en la [especificación del protocolo System for Cross-Domain Identity Management (SCIM) 2.0](https://tools.ietf.org/html/draft-ietf-scim-api-19). 

El soporte técnico de Azure AD para el protocolo SCIM 2.0 se describe en [Uso de System for Cross-Domain Identity Management (SCIM) para aprovisionar automáticamente a los usuarios y grupos de Azure Active Directory para aplicaciones](use-scim-to-provision-users-and-groups.md), que muestra las partes específicas del protocolo que se implementan con el fin de aprovisionar automáticamente usuarios y grupos de Azure AD a las aplicaciones que admiten SCIM 2.0.

En este artículo se describen los problemas actuales y pasados con el cumplimiento del protocolo SCIM 2.0 en el servicio de aprovisionamiento de usuarios de Azure AD, así como la forma de solucionar estos problemas.

> [!IMPORTANT]
> La actualización más reciente en el cliente de SCIM del servicio de aprovisionamiento de usuarios de Azure AD se realizó el 18 de diciembre de 2018. Esta actualización resolvió los problemas de compatibilidad conocidos que aparecen en la tabla siguiente. Consulte estas preguntas más frecuentes para obtener información adicional acerca de esta actualización.

## <a name="scim-20-compliance-issues-and-status"></a>Problemas de cumplimiento y estado de SCIM 2.0

| **Problema de compatibilidad de SCIM 2.0** |  **¿Corregido?** | **Fecha de corrección**  |  
|---|---|---|
| Azure AD requiere "/scim" en la raíz de la dirección URL del punto de conexión SCIM de la aplicación.  | Sí  |  18 de diciembre de 2018 | 
| Los atributos de extensión utilizan la notación de punto "." antes de los nombres de atributo en lugar de la notación de dos puntos ":". |  Sí  | 18 de diciembre de 2018  | 
|  Las solicitudes de revisión para los atributos multivalor tienen una sintaxis de filtro de ruta no válida. | Sí  |  18 de diciembre de 2018  | 
|  Las solicitudes de creación de grupos contienen un URI de esquema no válido. | Sí  |  18 de diciembre de 2018  |  

## <a name="were-the-services-fixes-described-automatically-applied-to-my-pre-existing-scim-app"></a>¿Las correcciones de los servicios que se describen se aplican automáticamente a mi aplicación SCIM ya existente?

No. Dado que hubiera constituido un cambio importante para las aplicaciones SCIM cuyo código estaba diseñado para trabajar con el comportamiento anterior, los cambios no se han aplicado automáticamente a las aplicaciones existentes.

Los cambios se aplican a todas las nuevas [aplicaciones SCIM fuera de la galería](configure-single-sign-on-non-gallery-applications.md) configuradas en Azure Portal, tras la fecha de la corrección.

Para obtener información sobre cómo migrar un trabajo de aprovisionamiento de usuarios existente a fin de incluir las correcciones más recientes, consulte la sección siguiente.

## <a name="can-i-migrate-an-existing-scim-based-user-provisioning-job-to-include-the-latest-service-fixes"></a>¿Puedo migrar un trabajo de aprovisionamiento de usuarios basado en SCIM para incluir las correcciones del servicio más recientes?

Sí. Si ya usa esta instancia de aplicación para el inicio de sesión único y debe migrar el trabajo de aprovisionamiento existente para incluir las correcciones más recientes, siga el procedimiento siguiente. Este procedimiento describe cómo usar Microsoft Graph API y el explorador de Microsoft Graph API para quitar su antiguo trabajo de aprovisionamiento de la aplicación SCIM existente y crear uno nuevo que muestre el nuevo comportamiento.

> [!NOTE]
> Si la aplicación aún está en desarrollo y no se ha implementado todavía para el inicio de sesión único o el aprovisionamiento de usuarios, la solución más sencilla consiste en eliminar la entrada de la aplicación en la sección **Azure Active Directory > Aplicaciones empresariales** de Azure Portal y simplemente agregar una nueva entrada para la aplicación mediante la opción **Crear aplicación > Fuera de la galería**. Esta es una alternativa mejor a la de ejecutar el siguiente procedimiento.
 
1. Inicie sesión en Azure Portal en https://portal.azure.com.
2. En la sección **Azure Active Directory > Aplicaciones empresariales** de Azure Portal, busque y seleccione la aplicación SCIM existente.
3. En la sección **Propiedades** de la aplicación SCIM existente, copie el **id. de objeto**.
4. En una nueva ventana del explorador web, vaya a https://developer.microsoft.com/graph/graph-explorer e inicie sesión como administrador para el inquilino de Azure AD donde se agrega la aplicación.
5. En el Probador de Graph, ejecute el comando siguiente para buscar el identificador del trabajo de aprovisionamiento. Reemplace "[object-id]" por el id. de la entidad de servicio (id. de objeto) que se copió en el tercer paso.
 
   `GET https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs` 

   ![Obtener trabajos](./media/application-provisioning-config-problem-scim-compatibility/get-jobs.PNG "Get jobs") 


6. En los resultados, copie la cadena "ID" completa que comienza con "customappsso" o "scim".
7. Ejecute el comando siguiente para recuperar la configuración de asignación de atributos, de modo que pueda realizar una copia de seguridad. Utilice el mismo [object-id] que antes y reemplace [job-id] con el identificador del trabajo de aprovisionamiento copiado en el paso anterior.
 
   `GET https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs/[job-id]/schema`
 
   ![Obtener esquema](./media/application-provisioning-config-problem-scim-compatibility/get-schema.PNG "Get Schema") 

8. Copie la salida JSON desde el último paso y guárdela en un archivo de texto. Este archivo contiene las asignaciones de atributos personalizadas que ha agregado a la aplicación anterior y deben ser aproximadamente unas miles de líneas de JSON.
9. Ejecute el comando siguiente para eliminar el trabajo de aprovisionamiento:
 
   `DELETE https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs/[job-id]`

10. Ejecute el comando siguiente para crear un nuevo trabajo de aprovisionamiento que tenga las correcciones del servicio más recientes.

 `POST https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs`
 `{   templateId: "scim"   }`
   
11. En los resultados del último paso, copie la cadena "ID" completa que comienza con "scim". Si lo desea, vuelva a aplicar las asignaciones de atributos anteriores ejecutando el comando siguiente, reemplazando [new-job-id] con el nuevo id. de trabajo que acaba de copiar y escribiendo la salida de JSON desde el paso n.° 7 como el cuerpo de la solicitud.

 `POST https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs/[new-job-id]/schema`
 `{   <your-schema-json-here>   }`

12. Vuelva a la primera ventana de explorador web y seleccione la pestaña **Aprovisionamiento** de la aplicación.
13. Compruebe la configuración y, a continuación, inicie el trabajo de aprovisionamiento. 

## <a name="can-i-add-a-new-non-gallery-app-that-has-the-old-user-provisioning-behavior"></a>¿Puedo agregar una nueva aplicación fuera de la galería que tenga el comportamiento anterior de aprovisionamiento de usuarios?

Sí. Si programó una aplicación con el comportamiento anterior que existía antes de las correcciones y necesita implementar una nueva instancia de ella, siga el procedimiento siguiente. Este procedimiento describe cómo usar Microsoft Graph API y el explorador de Microsoft Graph API para crear un trabajo de aprovisionamiento de SCIM que muestre el nuevo comportamiento.
 
1. Inicie sesión en Azure Portal en https://portal.azure.com.
2. En la sección **Azure Active Directory > Aplicaciones empresariales > Crear aplicación** del portal de Azure, cree una nueva aplicación **fuera de la galería**.
3. En la sección **Propiedades** de la nueva aplicación personalizada, copie el **id. de objeto**.
4. En una nueva ventana del explorador web, vaya a https://developer.microsoft.com/graph/graph-explorer e inicie sesión como administrador para el inquilino de Azure AD donde se agrega la aplicación.
5. En el Probador de Graph, ejecute el comando siguiente para inicializar la configuración de aprovisionamiento de la aplicación.
   Reemplace "[object-id]" por el id. de la entidad de servicio (id. de objeto) que se copió en el tercer paso.

   `POST https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs`
   `{   templateId: "customappsso"   }`
 
6. Vuelva a la primera ventana de explorador web y seleccione la pestaña **Aprovisionamiento** de la aplicación.
7. Complete la configuración del aprovisionamiento de usuarios como lo haría normalmente.


## <a name="next-steps"></a>Pasos siguientes
[Más información sobre aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS](user-provisioning.md)

