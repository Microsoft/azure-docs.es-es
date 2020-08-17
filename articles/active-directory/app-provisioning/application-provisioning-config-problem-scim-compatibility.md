---
title: 'Problemas conocidos de cumplimiento del protocolo System for Cross-domain Identity Management (SCIM) 2.0: Azure AD'
description: Cómo solucionar los problemas comunes de compatibilidad de protocolo a los que nos enfrentamos cuando agregamos una aplicación situada fuera de la galería compatible con SCIM 2.0 en Azure AD
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: reference
ms.date: 08/05/2020
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: c54478282cb1106ae95fe1c9e3fbb15e9c37bbf9
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87808582"
---
# <a name="known-issues-and-resolutions-with-scim-20-protocol-compliance-of-the-azure-ad-user-provisioning-service"></a>Problemas conocidos y soluciones con el cumplimiento de protocolo SCIM 2.0 del servicio de aprovisionamiento de usuarios de Azure AD

Azure Active Directory (Azure AD) puede aprovisionar automáticamente a usuarios y grupos de cualquier aplicación o sistema dirigidos por un servicio web con la interfaz definida en la [especificación del protocolo System for Cross-Domain Identity Management (SCIM) 2.0](https://tools.ietf.org/html/draft-ietf-scim-api-19). 

El soporte técnico de Azure AD para el protocolo SCIM 2.0 se describe en [Uso de System for Cross-Domain Identity Management (SCIM) para aprovisionar automáticamente a los usuarios y grupos de Azure Active Directory para aplicaciones](use-scim-to-provision-users-and-groups.md), que muestra las partes específicas del protocolo que se implementan con el fin de aprovisionar automáticamente usuarios y grupos de Azure AD a las aplicaciones que admiten SCIM 2.0.

En este artículo se describen los problemas actuales y pasados con el cumplimiento del protocolo SCIM 2.0 en el servicio de aprovisionamiento de usuarios de Azure AD, así como la forma de solucionar estos problemas.

## <a name="understanding-the-provisioning-job"></a>Descripción del trabajo de aprovisionamiento
El servicio de aprovisionamiento usa el concepto de trabajo para operar con una aplicación. El valor de jobID (identificador de trabajo) se puede encontrar en la [barra de progreso](application-provisioning-when-will-provisioning-finish-specific-user.md#view-the-provisioning-progress-bar). Todas las nuevas aplicaciones en aprovisionamiento se crean con un jobID que empieza por "scim". El trabajo de SCIM representa el estado actual del servicio. Los trabajos más antiguos tienen el identificador "customappsso". Este trabajo representa el estado del servicio en 2018. 

Si usa una aplicación de la galería, el trabajo contiene generalmente el nombre de la aplicación (por ejemplo, Zoom snowFlake, Databricks, etc.). Puede omitir esta documentación cuando se usa una aplicación de la galería. Este artículo se aplica principalmente a las aplicaciones que no son de la galería con el valor de jobID establecido en SCIM o customAppSSO.

## <a name="scim-20-compliance-issues-and-status"></a>Problemas de cumplimiento y estado de SCIM 2.0
En la tabla siguiente, cualquier elemento marcado como Corregido significa que se puede esperar el comportamiento adecuado del trabajo de SCIM. Hemos realizado trabajos para garantizar la compatibilidad con versiones anteriores de los cambios que hemos realizado. Sin embargo, no se recomienda implementar el comportamiento anterior. Se recomienda usar el nuevo comportamiento para todas las nuevas implementaciones y actualizar las implementaciones existentes.

> [!NOTE]
> En el caso de los cambios realizados en 2018, puede revertir al comportamiento de customappsso. En el caso de los cambios realizados desde 2018, puede usar las direcciones URL para revertir al comportamiento anterior. Hemos realizado trabajos para garantizar la compatibilidad con versiones anteriores de los cambios que hemos realizado, permitiéndole revertir al valor de jobID anterior o mediante el uso de una marca. Sin embargo, como se mencionó anteriormente, no se recomienda implementar el comportamiento anterior. Se recomienda usar el nuevo comportamiento para todas las nuevas implementaciones y actualizar las implementaciones existentes.

| **Problema de compatibilidad de SCIM 2.0** |  **¿Corregido?** | **Fecha de corrección**  |  **Compatibilidad con versiones anteriores** |
|---|---|---|
| Azure AD requiere "/scim" en la raíz de la dirección URL del punto de conexión SCIM de la aplicación.  | Sí  |  18 de diciembre de 2018 | cambiar a una versión anterior: customappSSO |
| Los atributos de extensión utilizan la notación de punto "." antes de los nombres de atributo en lugar de la notación de dos puntos ":". |  Sí  | 18 de diciembre de 2018  | cambiar a una versión anterior: customappSSO |
| Las solicitudes de revisión para los atributos multivalor tienen una sintaxis de filtro de ruta no válida. | Sí  |  18 de diciembre de 2018  | cambiar a una versión anterior: customappSSO |
| Las solicitudes de creación de grupos contienen un URI de esquema no válido. | Sí  |  18 de diciembre de 2018  |  cambiar a una versión anterior: customappSSO |
| Actualización del comportamiento de la revisión para garantizar el cumplimiento | No | TBD| usar marca en versión preliminar |

## <a name="flags-to-alter-the-scim-behavior"></a>Marcas para modificar el comportamiento de SCIM
Utilice las marcas siguientes en la dirección URL del inquilino de la aplicación para cambiar el comportamiento predeterminado del cliente de SCIM.

:::image type="content" source="media/application-provisioning-config-problem-scim-compatibility/scim-flags.jpg" alt-text="Marcas de SCIM para modificar el comportamiento.":::

* Actualización del comportamiento de la revisión para garantizar el cumplimiento
  * **Referencias de RFC de SCIM:** 
    * https://tools.ietf.org/html/rfc7644#section-3.5.2
  * **Dirección URL (compatible con SCIM):** AzureAdScimPatch062020
  * **Comportamiento:**
    * Eliminaciones de pertenencia a grupos compatibles:
  ```json
   {
     "schemas":
      ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
     "Operations":[{
       "op":"remove",
       "path":"members[value eq \"2819c223-7f76-...413861904646\"]"
     }]
   }
  ```
  * **Dirección URL (no compatible con SCIM):** AzureAdScimPatch2017
  * **Comportamiento:**
    * Eliminaciones de pertenencia a grupos no compatibles:
   ```json
   {
     "schemas":
     ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
     "Operations":[{
       "op":"Remove",  
       "path":"members",
       "value":[{"value":"2819c223-7f76-...413861904646"}]
     }]
   }
   ```

## <a name="upgrading-from-the-older-customappsso-job-to-the-scim-job"></a>Actualización desde el trabajo customappsso anterior al trabajo de SCIM
En los pasos siguientes se eliminará el trabajo customappsso existente y se creará un nuevo trabajo de SCIM. 
 
1. Inicie sesión en Azure Portal en https://portal.azure.com.
2. En la sección **Azure Active Directory > Aplicaciones empresariales** de Azure Portal, busque y seleccione la aplicación SCIM existente.
3. En la sección **Propiedades** de la aplicación SCIM existente, copie el **id. de objeto**.
4. En una nueva ventana del explorador web, vaya a https://developer.microsoft.com/graph/graph-explorer e inicie sesión como administrador para el inquilino de Azure AD donde se agrega la aplicación.
5. En el Probador de Graph, ejecute el comando siguiente para buscar el identificador del trabajo de aprovisionamiento. Reemplace "[object-id]" por el id. de la entidad de servicio (id. de objeto) que se copió en el tercer paso.
 
   `GET https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs` 

   ![Obtener trabajos](media/application-provisioning-config-problem-scim-compatibility/get-jobs.PNG "Obtener trabajos") 


6. En los resultados, copie la cadena "ID" completa que comienza con "customappsso" o "scim".
7. Ejecute el comando siguiente para recuperar la configuración de asignación de atributos, de modo que pueda realizar una copia de seguridad. Utilice el mismo [object-id] que antes y reemplace [job-id] con el identificador del trabajo de aprovisionamiento copiado en el paso anterior.
 
   `GET https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs/[job-id]/schema`
 
   ![Obtener esquema](media/application-provisioning-config-problem-scim-compatibility/get-schema.PNG "Obtener esquema") 

8. Copie la salida JSON desde el último paso y guárdela en un archivo de texto. El código JSON contiene las asignaciones de atributos personalizadas que ha agregado a la aplicación anterior y deben ser aproximadamente unas miles de líneas de código JSON.
9. Ejecute el comando siguiente para eliminar el trabajo de aprovisionamiento:
 
   `DELETE https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs/[job-id]`

10. Ejecute el comando siguiente para crear un nuevo trabajo de aprovisionamiento que tenga las correcciones del servicio más recientes.

 `POST https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs`
 `{   templateId: "scim"   }`
   
11. En los resultados del último paso, copie la cadena "ID" completa que comienza con "scim". Si lo desea, vuelva a aplicar las asignaciones de atributos anteriores; para ello, ejecute el comando siguiente, reemplace [new-job-id] por el nuevo identificador de trabajo que ha copiado y escriba la salida JSON del paso 7 como cuerpo de la solicitud.

 `POST https://graph.microsoft.com/beta/servicePrincipals/[object-id]/synchronization/jobs/[new-job-id]/schema`
 `{   <your-schema-json-here>   }`

12. Vuelva a la primera ventana de explorador web y seleccione la pestaña **Aprovisionamiento** de la aplicación.
13. Compruebe la configuración y, a continuación, inicie el trabajo de aprovisionamiento. 

## <a name="downgrading-from-the-scim-job-to-the-customappsso-job-not-recommended"></a>Cambio a una versión anterior desde el trabajo de SCIM al trabajo customappsso (no recomendado)
 Está permitido retroceder al comportamiento anterior, pero no se recomienda, ya que customappsso no se beneficia de algunas de las actualizaciones que hemos realizado y es posible que no se admita siempre. 

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

