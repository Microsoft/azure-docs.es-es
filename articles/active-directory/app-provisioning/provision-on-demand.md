---
title: Aprovisionamiento de un usuario a petición mediante Azure Active Directory
description: Sincronización forzada
services: active-directory
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: how-to
ms.date: 06/23/2020
ms.author: mimart
ms.reviewer: arvinh
ms.openlocfilehash: 52819fc37cf0d10cb36009feb82dec234184752c
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88235543"
---
# <a name="on-demand-provisioning"></a>Aprovisionamiento a petición
Use el aprovisionamiento a petición para aprovisionar un usuario en una aplicación en cuestión de segundos. Entre otras cosas, puede usar esta capacidad para:

* Soluciar problemas de configuración rápidamente.
* Validar las expresiones que haya definido.
* Probar filtros de ámbito.

## <a name="how-to-use-on-demand-provisioning"></a>Procedimiento para usar el aprovisionamiento a petición

1. Inicie sesión en **Azure Portal**.
2. Vaya a **Todos los servicios** > **Aplicaciones empresariales**.
3. Seleccione la aplicación y vaya a la página de configuración del aprovisionamiento.
4. Configure el aprovisionamiento al proporcionar las credenciales de administrador.
5. Seleccione **Aprovisionamiento a petición**.
6. Busque un usuario por nombre, apellidos, nombre para mostrar, nombre principal de usuario o dirección de correo electrónico.
7. Seleccione **Aprovisionar** en la parte inferior de la página.

:::image type="content" source="media/provision-on-demand/on-demand-provision-user.jpg" alt-text="Captura de pantalla que muestra la interfaz de usuario de Azure Portal para aprovisionar a un usuario a petición.":::

## <a name="understand-the-provisioning-steps"></a>Descripción de los pasos de aprovisionamiento

El proceso de aprovisionamiento a petición intenta mostrar los pasos que realiza el servicio de aprovisionamiento al aprovisionar un usuario. Normalmente, hay cinco pasos para aprovisionar a un usuario. Durante la experiencia de aprovisionamiento a petición se mostrarán uno o varios de estos pasos, que se explican en las secciones siguientes.

### <a name="step-1-test-connection"></a>Paso 1: Prueba de conexión

El servicio de aprovisionamiento intenta autorizar el acceso a la aplicación de destino mediante la solicitud de un "usuario de prueba". El servicio de aprovisionamiento espera una respuesta que indique que está autorizado para continuar con los pasos de aprovisionamiento. Este paso solo se muestra cuando se produce un error. No se muestra en la experiencia de aprovisionamiento a petición cuando el paso se realiza correctamente.

#### <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

* Asegúrese de que haya proporcionado credenciales válidas, como el token secreto y la dirección URL del inquilino, a la aplicación de destino. Las credenciales requeridas varían según la aplicación. Para ver tutoriales de configuración detallados, consulte la [lista de tutoriales](../saas-apps/tutorial-list.md). 
* Asegúrese de que la aplicación de destino admita el filtrado por los atributos emparejados definidos en el panel **Asignaciones de atributos**. Es posible que deba consultar la documentación de la API proporcionada por el desarrollador de la aplicación a fin de entender los filtros admitidos.
* En el caso de las aplicaciones System for Cross-domain Identity Management (SCIM), puede usar una herramienta como Postman. Estas herramientas le ayudan a asegurarse de que la aplicación responde a las solicitudes de autorización de la forma que espera el servicio de aprovisionamiento de Azure Active Directory (Azure AD). Consulte una [solicitud de ejemplo](./use-scim-to-provision-users-and-groups.md#request-3).

### <a name="step-2-import-user"></a>Paso 2: Importación de usuario

Después, el servicio de aprovisionamiento recupera el usuario desde el sistema de origen. Los atributos de usuario que el servicio recupera se usan más adelante para:

* Evaluar si el usuario está en el ámbito del aprovisionamiento.
* Comprobar si hay un usuario existente en el sistema de destino.
* Determinar qué atributos de usuario se exportarán al sistema de destino.

#### <a name="view-details"></a>Ver detalles


En la sección **Ver detalles** se muestran las propiedades del usuario que se han importado desde el sistema de origen (por ejemplo, Azure AD).

#### <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

* Se puede producir un error al importar el usuario si falta el atributo emparejado en el objeto de usuario del sistema de origen. Para resolver este error, pruebe uno de estos métodos:

  * Actualice el objeto de usuario con un valor para el atributo coincidente.
  * Cambie el atributo coincidente en la configuración de aprovisionamiento.

* Si falta un atributo que esperaba en la lista que se ha importado, asegúrese de que el atributo tenga un valor en el objeto de usuario del sistema de origen. El servicio de aprovisionamiento no admite actualmente el aprovisionamiento de atributos NULL.
* Asegúrese de que la página **Asignación de atributos** de la configuración de aprovisionamiento contiene el atributo que espera.

### <a name="step-3-determine-if-user-is-in-scope"></a>Paso 3: Determinación de si el usuario está en el ámbito

A continuación, el servicio de aprovisionamiento determina si el usuario está en el [ámbito](./how-provisioning-works.md#scoping) del aprovisionamiento. El servicio tiene en cuenta aspectos como:

* Si el usuario está asignado a la aplicación.
* Si el ámbito está establecido en **Sync assigned** (Sincronizar asignación) o **Sincronizar todo**.
* Los filtros de ámbito definidos en la configuración de aprovisionamiento.  

#### <a name="view-details"></a>Ver detalles

En la sección **Ver detalles** se muestran las condiciones de ámbito que se han evaluado. Puede ver una o varias de las siguientes propiedades:

* **Activo en sistema de origen** indica que el usuario tiene la propiedad `IsActive` establecida en **true** en Azure AD.
* **Asignado a aplicación** indica que el usuario está asignado a la aplicación en Azure AD.
* **Ámbito sincronizar todo** indica que la configuración de ámbito permite todos los usuarios y grupos del inquilino.
* **Usuario tiene rol requerido** indica que el usuario tiene los roles necesarios para aprovisionarse en la aplicación. 
* **Filtros de ámbito** también aparece si se han definido filtros de ámbito para la aplicación. El filtro se muestra con el siguiente formato: {título del filtro de ámbito} {atributo del filtro de ámbito} {operador del filtro de ámbito} {valor del filtro de ámbito}.

#### <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

* Asegúrese de que haya definido un rol de ámbito válido. Por ejemplo, evite usar el [operador Mayor que](./define-conditional-rules-for-provisioning-user-accounts.md#create-a-scoping-filter) con un valor no entero.
* Si el usuario no tiene el rol necesario, revise las [recomendaciones para aprovisionar a los usuarios asignados al rol de acceso predeterminado](./application-provisioning-config-problem-no-users-provisioned.md#provisioning-users-assigned-to-the-default-access-role).

### <a name="step-4-match-user-between-source-and-target"></a>Paso 4: Emparejamiento del usuario entre origen y destino

En este paso, el servicio intenta emparejar el usuario que se ha recuperado en el paso de importación con un usuario del sistema de destino.

#### <a name="view-details"></a>Ver detalles

En la página **Ver detalles** se muestran las propiedades de los usuarios que se han emparejado en el sistema de destino. Las propiedades que se ven en el panel de contexto varían así:

* Si no hay usuarios emparejados en el sistema de destino, no se ve ninguna propiedad.
* Si hay un usuario emparejado en el sistema de destino, se ven las propiedades de ese usuario emparejado del sistema de destino.
* Si hay varios usuarios emparejados, se ven las propiedades de ambos usuarios emparejados.
* Si hay varios atributos emparejados que forman parte de las asignaciones de atributos, cada atributo emparejado se evalúa secuencialmente y se muestran los usuarios emparejados del atributo.

#### <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

* Es posible que el servicio de aprovisionamiento no pueda emparejar de forma única a un usuario del sistema de origen con un usuario del destino. Resuelva este problema asegurándose de que el atributo emparejado sea único.
* Asegúrese de que la aplicación de destino admita el filtrado por el atributo definido como atributo emparejado.  

### <a name="step-5-perform-action"></a>Paso 5: Realización de acción

Por último, el servicio de aprovisionamiento realiza una acción, como crear, actualizar, eliminar u omitir el usuario.

Este es un ejemplo de lo que podría ver después del aprovisionamiento a petición correcto de un usuario:

:::image type="content" source="media/provision-on-demand/success-on-demand-provision.jpg" alt-text="Captura de pantalla que muestra el aprovisionamiento a petición correcto de un usuario.":::

#### <a name="view-details"></a>Ver detalles

La sección **Ver detalles** muestra los atributos que se han modificado en la aplicación de destino. Esta visualización representa la salida final de la actividad del servicio de aprovisionamiento y los atributos que se han exportado. Si se produce un error en este paso, los atributos mostrados representan los atributos que el servicio de aprovisionamiento ha intentado modificar.

#### <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

* Los errores para exportar cambios pueden variar considerablemente. Consulte la [documentación de los registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md#error-codes) para conocer los errores habituales.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

* **¿Es necesario desactivar el aprovisionamiento para usar el aprovisionamiento a petición?** En las aplicaciones que usan un token de portador de larga duración o un nombre de usuario y una contraseña para la autorización no se necesitan pasos adicionales. Las aplicaciones que usan OAuth para la autorización actualmente requieren que el trabajo de aprovisionamiento se detenga para usar el aprovisionamiento a petición. En esta categoría se encuentran aplicaciones como G Suite, Box, Workplace by Facebook y Slack. Se está trabajando para permitir la ejecución del aprovisionamiento a petición para todas las aplicaciones, sin necesidad de detener los trabajos de aprovisionamiento.

* **¿Cuánto tarda el aprovisionamiento a petición?** El aprovisionamiento a petición normalmente tarda menos de 30 segundos.

## <a name="known-limitations"></a>Restricciones conocidas

Actualmente, hay algunas limitaciones conocidas para el aprovisionamiento a petición. Publique sus [comentarios y sugerencias](https://aka.ms/appprovisioningfeaturerequest) para que podamos determinar mejor las mejoras que deben realizarse a continuación.

> [!NOTE]
> Las siguientes limitaciones son específicas de la capacidad de aprovisionamiento a petición. Para obtener información sobre si una aplicación admite el aprovisionamiento de grupos, las eliminaciones u otras funciones, consulte el tutorial de esa aplicación.

* Las aplicaciones WorkDay, Amazon Web Services (AWS) y SuccessFactors no admiten el aprovisionamiento a petición. 
* No se admite el aprovisionamiento a petición de grupos y roles.
* El aprovisionamiento a petición admite la deshabilitación de usuarios que se han desasignado de la aplicación. Sin embargo, no admite la deshabilitación o eliminación de usuarios que se han deshabilitado o eliminado de Azure AD. Estos usuarios no aparecerán al buscar un usuario.

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de aprovisionamiento](./application-provisioning-config-problem.md)