---
title: 'Inicio rápido: Creación de una instancia de Azure Data Catalog'
description: En este inicio rápido, se describe cómo crear una instancia de Azure Data Catalog mediante Azure Portal.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: quickstart
ms.date: 05/26/2020
ms.openlocfilehash: e08527ae530b7b5ed5e5293a3af9e2d8dd3f795a
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88118797"
---
# <a name="quickstart-create-an-azure-data-catalog"></a>Inicio rápido: Creación de una instancia de Azure Data Catalog

Azure Data Catalog es un servicio en la nube totalmente administrado que actúa como sistema de registro y de detección para recursos de datos empresariales. Si desea información detallada, consulte [¿Qué es Azure Data Catalog](overview.md).

Este inicio rápido le muestra los primeros pasos para crear una instancia de Azure Data Catalog.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

> [!Note]
> Debido a los requisitos de seguridad de Azure, Azure Data Catalog ha aplicado el protocolo Seguridad de la capa de transporte (TLS) 1.2. Se han deshabilitado TLS 1.0 y TLS 1.1. Es posible que se produzcan errores al ejecutar la herramienta de registro si la máquina no está actualizada a TLS 1.2. Para actualizar su máquina a TLS 1.2, consulte [Habilitación de la Seguridad de la capa de transporte (1.2)](https://docs.microsoft.com/mem/configmgr/core/plan-design/security/enable-tls-1-2).

Para empezar, necesitará tener lo siguiente:

* Una suscripción de [Microsoft Azure](https://azure.microsoft.com/).
* Debe tener su propio [inquilino de Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

Para configurar Data Catalog, debe ser propietario o copropietario de una suscripción de Azure.

## <a name="create-a-data-catalog"></a>Creación de un catálogo de datos

Solo se puede aprovisionar un catálogo de datos por organización (dominio de Azure Active Directory). Por consiguiente, si el propietario o copropietario de una suscripción de Azure que pertenezca a dicho dominio de Active Directory ya ha creado un catálogo, no podrá volver a crear otro, aunque tenga varias suscripciones de Azure. Para comprobar si algún usuario ha creado un catálogo de datos en un dominio de Azure Active Directory, vaya a la [página principal de Azure Data Catalog](http://azuredatacatalog.com) y compruebe si ve un catálogo. Si ya hay un catálogo creado, omita el siguiente procedimiento y vaya a la siguiente sección.

1. Vaya a [Azure Portal](https://portal.azure.com) > **Crear un recurso** y seleccione **Data Catalog**.

    ![Botón Crear de Azure Data Catalog](media/data-catalog-get-started/data-catalog-create.png)

2. Especifique el **nombre**, la **suscripción** que desea usar, la **ubicación** y el **plan de tarifa** del catálogo de datos. Seleccione **Crear**.

3. Vaya a la [página principal de Azure Data Catalog](http://azuredatacatalog.com) y haga clic en **Publicar datos**.

   ![Azure Data Catalog: botón Publicar datos](media/data-catalog-get-started/data-catalog-publish-data.png)

   También puede acceder a la página principal de Data Catalog en la [página del servicio Data Catalog](https://azure.microsoft.com/services/data-catalog) seleccionando **Comenzar**.

   ![Azure Data Catalog: página de aterrizaje de marketing](media/data-catalog-get-started/data-catalog-marketing-landing-page.png)

4. Vaya a la página **Configuración**.

    ![Azure Data Catalog: aprovisionamiento del catálogo de datos](media/data-catalog-get-started/data-catalog-create-azure-data-catalog.png)

5. Expanda **Precios** y compruebe la **edición** de Azure Data Catalog (Gratis o Estándar).

    ![Azure Data Catalog: selección de edición](media/data-catalog-get-started/data-catalog-create-catalog-select-edition.png)

6. Si elige la edición *Estándar* como plan de tarifa, puede expandir **Grupos de seguridad** y autorizar a los grupos de seguridad de Active Directory para que puedan acceder a Data Catalog y habilitar el ajuste automático de la facturación.

    ![Grupos de seguridad de Azure Data Catalog](media/data-catalog-get-started/data-catalog-standard-security-groups.png)

7. Expanda **Usuarios de catálogo** y haga clic en **Agregar** para agregar usuarios al catálogo de datos. Se le agregará automáticamente a este grupo.

    ![Azure Data Catalog: usuarios](media/data-catalog-get-started/data-catalog-add-catalog-user.png)

8. Si elige la edición *Estándar* como plan de tarifa, puede expandir **Administradores de glosarios** y hacer clic en **Agregar** para agregar usuarios administradores de glosarios. Se le agregará automáticamente a este grupo.

    ![Administradores de glosarios de Azure Data Catalog](media/data-catalog-get-started/data-catalog-standard-glossary-admin.png)

9. Expanda **Administradores de catálogo** y haga clic en **Agregar** para agregar más administradores al catálogo de datos. Se le agregará automáticamente a este grupo.

    ![Azure Data Catalog: administradores](media/data-catalog-get-started/data-catalog-add-catalog-admins.png)

10. Expanda **Título del portal** y agregue el texto adicional que se mostrará en este.

    ![Azure Data Catalog: título del portal](media/data-catalog-get-started/data-catalog-portal-title.png)

11. Cuando haya completado los datos de la página **Configuración**, vaya a la página **Publicar**.

    ![Azure Data Catalog: creado](media/data-catalog-get-started/data-catalog-created.png)

## <a name="find-a-data-catalog-in-the-azure-portal"></a>Búsqueda de un catálogo de datos en el Portal de Azure

1. En otra pestaña o en otra ventana del explorador web, vaya al [Azure Portal](https://portal.azure.com) e inicie sesión con la misma cuenta que usó para crear el catálogo de datos en el paso anterior.

2. Seleccione **Todos los servicios** y haga clic en **Data Catalog**.

    ![Azure Data Catalog: exploración de Azure](media/data-catalog-get-started/data-catalog-browse-azure-portal.png)

    Verá el catálogo de datos que ha creado.

    ![Azure Data Catalog: visualización de catálogo en lista](media/data-catalog-get-started/data-catalog-azure-portal-show-catalog.png)

3. Haga clic en el catálogo que ha creado. Verá la hoja **Catálogo de datos** en el portal.

   ![Azure Data Catalog: hoja en el portal](media/data-catalog-get-started/data-catalog-blade-azure-portal.png)

4. Puede ver las propiedades del catálogo de datos y actualizarlas. Por ejemplo, haga clic en **Plan de tarifa** y cambie la edición.

    ![Azure Data Catalog: plan de tarifa](media/data-catalog-get-started/data-catalog-change-pricing-tier.png)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una instancia de Azure Data Catalog para su organización. Ahora puede registrar orígenes de datos en ella.

> [!div class="nextstepaction"]
> [Registro de orígenes de datos en Azure Data Catalog](data-catalog-how-to-register.md)
