---
title: Personalización de asignaciones de atributos de Azure AD | Microsoft Docs
description: Conozca cuáles son las asignaciones de atributos para aplicaciones SaaS en Azure Active Directory y cómo puede modificarlas para satisfacer sus necesidades empresariales.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/03/2019
ms.author: mimart
ms.custom: H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: bc3aea059b6ac4244ea440d26717562e83fdbd09
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824904"
---
# <a name="customizing-user-provisioning-attribute-mappings-for-saas-applications-in-azure-active-directory"></a>Personalización de asignaciones de atributos de aprovisionamiento de usuarios para aplicaciones SaaS en Azure Active Directory de usuarios
Microsoft Azure AD proporciona soporte técnico para el aprovisionamiento de usuarios para aplicaciones de SaaS de terceros como Salesforce, G Suite y otros. Si habilita el aprovisionamiento de usuarios para una aplicación de SaaS de terceros, el portal de Azure controla sus valores de atributo a través de asignaciones de atributos.

Hay un conjunto preconfigurado de atributos y asignaciones de atributos entre los objetos de usuario de Azure AD y los objetos de usuario de cada aplicación SaaS. Algunas aplicaciones administran otros tipos de objetos, junto con los usuarios, como los grupos.

Puede personalizar las asignaciones de atributos predeterminadas según sus necesidades empresariales. Por lo tanto, puede cambiar o eliminar asignaciones de atributos existentes o crear nuevas asignaciones de atributos.
 
## <a name="editing-user-attribute-mappings"></a>Edición de asignaciones de atributos de usuario

Siga estos pasos para tener acceso a la **asignaciones** característica de aprovisionamiento de usuarios:

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com).

1. Seleccione **aplicaciones empresariales** en el panel izquierdo. Se muestra una lista de todas las aplicaciones configuradas, incluidas las aplicaciones que se han agregado desde la galería.

1. Seleccione cualquier aplicación para cargar el panel de administración de aplicaciones, donde puede ver los informes y administrar la configuración de la aplicación.

1. Seleccione **aprovisionamiento** para administrar la configuración de la aplicación seleccionada de aprovisionamiento de cuentas de usuario.

1. Expanda **asignaciones** para ver y editar los atributos de usuario que fluyen entre Azure AD y la aplicación de destino. Si la aplicación de destino lo admite, esta sección le permite configurar, opcionalmente, el aprovisionamiento de grupos y cuentas de usuario.

   ![Salesforce](./media/customize-application-attributes/21.png) 

1. Seleccione un **asignaciones** configuración para abrir el relacionados **asignación de atributos** pantalla. Algunas asignaciones de atributos requiere una aplicación SaaS para funcionar correctamente. Para los atributos necesarios, la característica **Eliminar** no está disponible.

   ![Salesforce](./media/customize-application-attributes/22.png)

   En esta captura de pantalla, puede ver que el **Username** atributo de un objeto administrado en Salesforce se rellena con el **userPrincipalName** valor de objeto de Active Directory de Azure vinculada.

1. Seleccione una existente **asignación de atributos** para abrir el **Editar atributo** pantalla. Aquí puede editar los atributos de usuario que fluyen entre Azure AD y la aplicación de destino.

   ![Salesforce](./media/customize-application-attributes/23.png)


### <a name="understanding-attribute-mapping-types"></a>Información sobre los tipos de asignación de atributos
Con asignaciones de atributos, puede controlar cómo se rellenan los atributos en una aplicación SaaS de terceros. Se admiten cuatro tipos de asignaciones diferentes:

* **Directa** : el atributo de destino se rellena con el valor de un atributo del objeto vinculado en Azure AD.
* **Constante** : el atributo de destino se rellena con una cadena específica que especificó.
* **Expresión** : el atributo de destino se rellena según el resultado de una expresión similar a un script. 
  Para más información, consulte [Escritura de expresiones para la asignación de atributos en Azure Active Directory](functions-for-customizing-application-data.md).
* **Ninguno** : el atributo de destino se deja sin modificar. Sin embargo, si el atributo de destino está vacío, se rellena con el valor predeterminado que especifique.

Junto con estos cuatro tipos básicos, asignaciones de atributos personalizadas admiten el concepto de un elemento opcional **predeterminada** asignación de valor. La asignación de valor predeterminada garantiza que un atributo de destino se rellena con un valor si no existe un valor en Azure AD o en el objeto de destino. La configuración más habitual consiste en dejarlo en blanco.


### <a name="understanding-attribute-mapping-properties"></a>Información sobre las propiedades de asignación de atributos

En la sección anterior, se introdujeron ya a la propiedad tipo de asignación de atributos.
Junto con esta propiedad, asignaciones de atributos también admiten los siguientes atributos:

- **Atributo de origen**: especifica el atributo de usuario del sistema de origen (por ejemplo, Azure Active Directory).
- **Atributo de destino**: especifica el atributo de usuario del sistema de destino (por ejemplo, ServiceNow).
- **Hacer coincidir objetos con este atributo** : indica si esta asignación se debe usar para identificar de forma exclusiva a los usuarios entre los sistemas de origen y de destino. Normalmente se establece en el atributo userPrincipalName o mail en Azure AD, que se suele asignar a un campo de nombre de usuario en una aplicación de destino.
- **Precedencia de coincidencia**: se pueden establecer varios atributos coincidentes. Cuando hay varios, se evalúan en el orden definido por este campo. En el momento en que se encuentre una coincidencia, no se evaluarán más atributos coincidentes.
- **Aplicar esta asignación**
    - **Siempre** : se aplica esta asignación en la creación del usuario y acciones de actualización.
    - **Solo durante la creación** -esta asignación se aplica solo en las acciones de creación de usuario.


## <a name="editing-group-attribute-mappings"></a>Edición de asignaciones de atributos de grupo

Un número seleccionado de aplicaciones, como ServiceNow, Box y G Suite, admite la capacidad de aprovisionar objetos de grupo y los objetos de usuario. Objetos de grupo pueden contener propiedades de grupo como nombres para mostrar y alias, junto con los miembros del grupo de correo electrónico.

![ServiceNow](./media/customize-application-attributes/24.png)

Aprovisionamiento de grupos puede, opcionalmente, habilitar o deshabilitar mediante la selección de la asignación de grupo en **asignaciones**y estableciendo **habilitado** a la opción que desee en el **deasignacióndeatributos** pantalla.

Los atributos suministrados como parte de los objetos de grupo se pueden personalizar de la misma manera que los objetos de usuario descritos anteriormente. 

>[!TIP]
>El aprovisionamiento de objetos de grupo (propiedades y miembros) es un concepto diferente al de [asignación de grupos](assign-user-or-group-access-portal.md) a una aplicación. Es posible asignar un grupo a una aplicación, pero solo se pueden aprovisionar los objetos de usuario contenidos en el grupo. El aprovisionamiento de objetos de grupo completos no es necesario para usar grupos en asignaciones.


## <a name="editing-the-list-of-supported-attributes"></a>Edición de la lista de atributos admitidos

Los atributos de usuario admitidos en una determinada aplicación están preconfigurados. Las API de administración de usuario de la aplicación de la mayoría no son compatibles con la detección de esquemas. Por lo tanto, el servicio de aprovisionamiento de Azure AD no puede generar dinámicamente la lista de atributos admitidos mediante llamadas a la aplicación. 

Sin embargo, algunas aplicaciones admiten atributos personalizados, y el servicio de aprovisionamiento de Azure AD puede leer y escribir en los atributos personalizados. Para especificar sus definiciones en el portal de Azure, seleccione el **Mostrar opciones avanzadas** casilla de verificación en la parte inferior de la **asignación de atributos** pantalla y, a continuación, seleccione **Editar lista de atributos para** la aplicación.

Las aplicaciones y sistemas que admiten la personalización de la lista de atributos son:

* Salesforce
* ServiceNow
* Workday
* Azure Active Directory (se admiten los [atributos predeterminados de Azure AD Graph API](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#user-entity) y las extensiones de directorios personalizadas)
* Aplicaciones que admiten [SCIM 2.0](https://tools.ietf.org/html/rfc7643), donde es necesario agregar los atributos definidos en el [esquema principal](https://tools.ietf.org/html/rfc7643)

>[!NOTE]
>La edición de la lista de atributos admitidos solo se recomienda para administradores que hayan personalizado el esquema de sus aplicaciones y sistemas, y tengan conocimiento de primera mano de cómo se han definido sus atributos personalizados. A veces, es necesario estar familiarizado con las API y las herramientas de los desarrolladores que se proporcionan en una aplicación o un sistema. 

Al editar la lista de atributos admitidos, se proporcionan las siguientes propiedades:

* **Nombre**: el nombre del sistema del atributo, tal como se define en el esquema del objeto de destino. 
* **Tipo** -el tipo de datos almacena el atributo, tal como se define en el esquema del objeto de destino, que puede ser uno de los siguientes tipos:
   * *Binario*: el atributo contiene datos binarios.
   * *Booleano*: el atributo contiene un valor True o False.
   * *DateTime*: el atributo contiene una cadena de fecha.
   * *Entero*: al atributo contiene un entero.
   * *Referencia*: el atributo contiene un identificador que hace referencia a un valor almacenado en otra tabla en la aplicación de destino.
   * *Cadena*: el atributo contiene una cadena de texto. 
* **¿Clave principal?** -Si el atributo se define como un campo de clave principal en el esquema del objeto de destino.
* **¿Necesario?** -Si el atributo es necesario que se debe rellenar en el sistema o aplicación de destino.
* **¿Varios valores?** -Si el atributo es compatible con varios valores.
* **¿Coincidir mayúsculas y minúsculas?** -Si se evalúan los valores de atributos de una manera distingue mayúsculas de minúsculas.
* **Expresión de API** -no se usa, a menos que se indique lo contrario en la documentación de un conector de aprovisionamiento específico (como Workday).
* **Al que hace referencia el atributo de objeto** : si es un atributo de tipo de referencia, este menú le permite seleccionar la tabla y el atributo en la aplicación de destino que contiene el valor asociado al atributo. Por ejemplo, si tiene un atributo llamado "Department" cuyo valor almacenado hace referencia a un objeto de una tabla "Departments" independiente, seleccionaría "Departments.Name". Las tablas de referencia y los campos de identificador principal admitidos en una determinada aplicación están preconfigurados y actualmente no puede modificarse mediante el portal de Azure, pero se pueden modificar utilizando la [Graph API](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-configure-with-custom-target-attributes).

Para agregar un nuevo atributo, desplácese hasta el final de la lista de atributos admitidos, rellene los campos anteriores mediante las entradas proporcionadas y seleccione **Agregar atributo**. Cuando termine de agregar atributos, seleccione **Guardar**. A continuación, deberá volver a cargar el **aprovisionamiento** pestaña para los nuevos atributos estén disponibles en el editor de asignación de atributos.

## <a name="restoring-the-default-attributes-and-attribute-mappings"></a>Restauración de los atributos predeterminados y las asignaciones de atributos

Si necesita volver a empezar y restablezca sus asignaciones existentes de nuevo a su estado predeterminado, puede seleccionar la **Restaurar asignaciones predeterminadas** casilla de verificación y guarde la configuración. Si lo hace, establece todas las asignaciones como si la aplicación se ha agregado a su inquilino de Azure AD desde la Galería de aplicaciones. 

Al seleccionar esta opción se fuerza una resincronización de todos los usuarios mientras se está ejecutando el servicio de aprovisionamiento. 

>[!IMPORTANT]
>Se recomienda encarecidamente que **el estado de aprovisionamiento** establecerse **desactivar** antes de invocar esta opción.


## <a name="what-you-should-know"></a>Qué debería saber

* Microsoft Azure AD proporciona una implementación eficaz de un proceso de sincronización. En un entorno inicializado, sólo los objetos que requieren actualizaciones se procesan durante un ciclo de sincronización. 

* Actualizar las asignaciones de atributos repercute en el rendimiento de un ciclo de sincronización. Una actualización de la configuración de la asignación de atributos requiere que se vuelvan a evaluar todos los objetos administrados. 

* Una práctica recomendada es mantener el número de cambios consecutivos para las asignaciones de atributos, como mínimo.

* Agregar un atributo photo se deben aprovisionar en una aplicación actualmente no se admite como no se puede especificar el formato para sincronizar la foto. Puede solicitar que la característica de [User Voice](https://feedback.azure.com/forums/169401-azure-active-directory)


## <a name="next-steps"></a>Pasos siguientes

* [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](user-provisioning.md)
* [Escritura de expresiones para la asignación de atributos](functions-for-customizing-application-data.md)
* [Filtros de ámbito para el aprovisionamiento de usuario](define-conditional-rules-for-provisioning-user-accounts.md)
* [Uso de SCIM para habilitar el aprovisionamiento automático de usuarios y grupos de Azure Active Directory a aplicaciones](use-scim-to-provision-users-and-groups.md)
* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS](../saas-apps/tutorial-list.md)


