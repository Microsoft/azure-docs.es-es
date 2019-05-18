---
title: Aprovisionamiento de aplicaciones con filtros de ámbito | Microsoft Docs
description: Obtenga información sobre cómo usar los filtros de ámbito para evitar el aprovisionamiento de los objetos de las aplicaciones que admiten el aprovisionamiento automático de usuarios, en caso de que un objeto no satisfaga los requisitos empresariales.
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
ms.date: 09/11/2018
ms.author: mimart
ms.custom: H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2c831fc7ab1a646d41c0dc08d0e1a66380fe1232
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824733"
---
# <a name="attribute-based-application-provisioning-with-scoping-filters"></a>Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito
El objetivo de este artículo es explicar cómo usar filtros de ámbito para definir reglas basadas en atributos que determinarán qué usuarios se aprovisionarán en una aplicación.

## <a name="scoping-filter-use-cases"></a>Casos en los que se usan los filtros de ámbito

Un filtro de ámbito permite que el servicio de aprovisionamiento de Azure Active Directory (Azure AD) incluya o excluya a cualquier usuario que tenga un atributo que coincida con un valor específico. Por ejemplo, al aprovisionar los usuarios de Azure AD a una aplicación de SaaS que use un equipo de ventas, puede especificar que solo los usuarios que tengan un atributo de "departamento de ventas" deben estar incluidos en el proceso de aprovisionamiento.

Puede usar los filtros de ámbito de diversas maneras, según el tipo de conector de aprovisionamiento:

* **Aprovisionamiento de salida desde Azure AD a aplicaciones SaaS**. Cuando Azure AD es el sistema de origen, las [asignaciones de usuarios y grupos](assign-user-or-group-access-portal.md) son el método más común para determinar qué usuarios deben incluirse en el ámbito de aprovisionamiento. Estas asignaciones también se usan para habilitar el inicio de sesión único y proporcionan un único método para administrar el acceso y el aprovisionamiento. Igualmente, puede usar los filtros de ámbito de forma opcional como complemento a las asignaciones, o simplemente puede filtrar los usuarios según los valores de los atributos.

    >[!TIP]
    > Puede deshabilitar la opción de aprovisionamiento basada en asignaciones para una aplicación empresarial estableciendo la opción del menú [Ámbito](user-provisioning.md#how-do-i-set-up-automatic-provisioning-to-an-application) que se encuentra en la configuración de aprovisionamiento en **Sincronizar todos los usuarios y grupos**. Gracias a esta opción y a los filtros de ámbito basados en los atributos, obtendrá un rendimiento más rápido que si usa asignaciones basadas en grupos.  

* **Aprovisionamiento de entrada desde aplicaciones HCM a Azure AD y Active Directory**. Cuando una [aplicación HCM como Workday](../saas-apps/workday-tutorial.md) es el sistema de origen, los filtros de ámbito son el método principal para determinar qué usuarios deben ser aprovisionados desde la aplicación HCM en Active Directory o Azure AD.

De forma predeterminada, los conectores de aprovisionamiento de Azure AD no tienen configurados los filtros de ámbito basados en atributos. 

## <a name="scoping-filter-construction"></a>Estructura de un filtro de ámbito

Un filtro de ámbito consta de una o más *cláusulas*. Las cláusulas determinan qué usuarios pueden atravesar el filtro de ámbito mediante la evaluación de los atributos de cada usuario. Por ejemplo, podría tener una cláusula que requiere que el atributo "Estado" del usuario sea igual a "Nueva York", de modo que solo se aprovisionará a los usuarios de Nueva York en la aplicación. 

Una sola cláusula define una única condición de un solo valor de atributo. Si se crean varias cláusulas en un filtro de ámbito único, se evalúan juntas mediante la función lógica "AND". Esto significa que todas las cláusulas deben evaluarse como "true" para que un usuario pueda ser aprovisionado.

Por último, puede crear varios filtros de ámbito para una sola aplicación. Si hay varios filtros de ámbito, se evalúan juntos mediante la función lógica "OR". Esto significa que si todas las cláusulas de cualquiera de los filtros de ámbito configurados resultan ser "true", el usuario será aprovisionado.

Cada usuario o grupo que procesa el servicio de aprovisionamiento de Azure AD se evalúa siempre de forma individual con cada filtro de ámbito.

Por ejemplo, teniendo en cuenta el siguiente filtro de ámbito:

![Filtro de ámbito](./media/define-conditional-rules-for-provisioning-user-accounts/scoping-filter.PNG) 

Según este filtro de ámbito, los usuarios deben cumplir los siguientes criterios para ser aprovisionados:

* Deben estar en Nueva York.
* Deben trabajar en el departamento de ingeniería.
* Su identificador de empleado de la empresa debe estar entre 1.000.000 y 2.000.000.
* El puesto no debe estar vacío ni ser un valor nulo.

## <a name="create-scoping-filters"></a>Creación de filtros de ámbito
Los filtros de ámbito se configuran como parte de las asignaciones de atributos de cada conector de aprovisionamiento de usuarios de Azure AD. En el siguiente procedimiento se da por supuesto que ya ha configurado el aprovisionamiento automático de [una de las aplicaciones compatibles](../saas-apps/tutorial-list.md), para el que agregaremos un filtro de ámbito.

### <a name="create-a-scoping-filter"></a>Creación de un filtro de ámbito
1. En [Azure Portal](https://portal.azure.com), vaya a la sección **Azure Active Directory** > **Aplicaciones empresariales** > **Todas las aplicaciones**.

2. Seleccione la aplicación para la que desea configurar el aprovisionamiento automático: por ejemplo, "ServiceNow".

3. Seleccione la pestaña **Aprovisionamiento**.

4. En la sección **Asignaciones**, seleccione la asignación para la que desea configurar un filtro de ámbito: por ejemplo, "Sincronizar usuarios de Azure Active Directory con ServiceNow".

5. Seleccione el menú **Ámbito del objeto de origen**.

6. Seleccione **Agregar filtro de ámbito**.

7. Defina una cláusula seleccionando un **nombre de atributo** de origen, un **operador** y un **valor del atributo** para comparar. Se admiten los siguientes operadores:

    a. **EQUALS**. La cláusula devuelve "true" si el atributo que se evalúa coincide exactamente con el valor de la cadena de entrada (se distingue entre mayúsculas y minúsculas).

   b. **NOT EQUALS**. La cláusula devuelve "true" si el atributo que se evalúa no coincide con el valor de la cadena de entrada (se distingue entre mayúsculas y minúsculas).

   c. **IS TRUE**. La cláusula devuelve "true" si el atributo que se evalúa contiene un valor booleano de tipo "true".

   d. **IS FALSE**. La cláusula devuelve "true" si el atributo que se evalúa contiene un valor booleano de tipo "false".

   e. **IS NULL**. La cláusula devuelve "true" si el atributo que se evalúa está vacío.

   f. **IS NOT NULL**. La cláusula devuelve "true" si el atributo que se evalúa no está vacío.

   g. **REGEX MATCH**. La cláusula devuelve "true" si el atributo que se evalúa coincide con el patrón de una expresión regular. Ejemplo: ([1-9][0-9]) coincide con cualquier número entre 10 y 99.

   h. **NOT REGEX MATCH**. La cláusula devuelve "true" si el atributo que se evalúa no coincide con el patrón de una expresión regular.

8. Seleccione **Agregar nueva cláusula de control**.

9. Si lo desea, repita los pasos 7 y 8 para agregar más cláusulas de ámbito.

10. En **Título del filtro de ámbito**, escriba el nombre del filtro de ámbito.

11. Seleccione **Aceptar**.

12. Seleccione **Aceptar** de nuevo en la pantalla **Filtros de ámbito**. Si lo desea, repita los pasos del 6 al 11 para agregar otro filtro de ámbito.

13. Seleccione **Guardar** en la pantalla **Asignación de atributos**. 

>[!IMPORTANT] 
> Si guarda un nuevo filtro de ámbito, se realizará una sincronización completa de la aplicación y todos los usuarios del sistema de origen volverán a ser evaluados mediante el nuevo filtro de ámbito. Si un usuario de la aplicación que estaba en el ámbito de aprovisionamiento se encuentra ahora fuera del mismo, su cuenta se deshabilita o se desaprovisiona de la aplicación.


## <a name="related-articles"></a>Artículos relacionados
* [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS](user-provisioning.md)
* [Personalización de asignaciones de atributos para el aprovisionamiento de usuarios](customize-application-attributes.md)
* [Escritura de expresiones para la asignación de atributos](functions-for-customizing-application-data.md)
* [Notificaciones de aprovisionamiento de cuentas](user-provisioning.md)
* [Uso de SCIM para habilitar el aprovisionamiento automático de usuarios y grupos de Azure Active Directory a aplicaciones](use-scim-to-provision-users-and-groups.md)
* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS](../saas-apps/tutorial-list.md)

