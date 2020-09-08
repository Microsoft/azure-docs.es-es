---
title: 'Tutorial: Asignación de acceso con Cloudyn en Azure'
description: En este tutorial, aprenderá a asignar acceso a los datos de Cloudyn con cuentas de usuario que definen los niveles de acceso para las entidades.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: tutorial
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.custom: seodec18
ms.reviewer: benshy
ROBOTS: NOINDEX
ms.openlocfilehash: f981e347b809f5df87e3b5379c69aab83ecf74eb
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88687627"
---
# <a name="tutorial-assign-access-to-cloudyn-data"></a>Tutorial: Asignación del acceso a los datos de Cloudyn

El acceso a los datos de Cloudyn se proporciona mediante la administración de la identidad o del usuario. Las cuentas de usuario de Cloudyn determinan el acceso a las *entidades* y las funciones administrativas. Existen dos tipos de acceso: administrador y usuario. A menos que lo modifique el usuario, el acceso de administrador permite un uso no restringido de todas las funciones del portal de Cloudyn, incluida la administración de usuarios y listas de destinatarios, y el acceso a la entidad raíz para todos los datos de la entidad. Con el acceso de usuario se pretende que los usuarios finales puedan ver y crear informes mediante el acceso que tienen a los datos de entidad.

Las entidades se usan para reflejar la estructura jerárquica de la organización empresarial. Identifican departamentos, divisiones y equipos de la organización en Cloudyn. La jerarquía de entidad le ayuda a realizar un seguimiento preciso de los gastos de las entidades.

Al registrar su cuenta o contrato de Azure, se creó una cuenta con permisos de administrador en Cloudyn, con la que podrá seguir todos los pasos de este tutorial. Este tutorial trata el acceso a los datos de Cloudyn, y se incluye la administración de usuarios y entidades. Aprenderá a:

> [!div class="checklist"]
> * Crear un usuario con acceso de administrador
> * Crear un usuario con acceso de usuario
> * Eliminar un usuario
> * Eliminar o exportar datos personales
> * Crear y administrar entidades


Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

- Debe tener una cuenta de Azure.
- Debe tener un registro de prueba o una suscripción de pago en Cloudyn.

## <a name="create-a-user-with-admin-access"></a>Crear un usuario con acceso de administrador

Usted ya tiene acceso de administrador, pero puede que los compañeros de su organización también lo necesiten. En el portal de Cloudyn, haga clic en el símbolo de engranaje en la esquina superior derecha y seleccione **User Management** (Administración de usuarios). Haga clic en **Add New User** (Agregar nuevo usuario) para agregar un usuario nuevo.

Especifique la información del usuario obligatoria. El **identificador de inicio de sesión** debe ser una dirección de correo electrónico válida. Elija Allow User Management (Permitir administración de usuarios) para que el usuario pueda crear y modificar otros usuarios. La administración de listas de destinatarios permite al usuario editar las listas de destinatarios. Si selecciona **Notify user by email** (Notificar al usuario por correo electrónico), se envía al usuario un correo de Cloudyn que contiene vínculo con información de inicio de sesión. En el primer inicio de sesión, el usuario establece una contraseña.

En **User has admin access** (El usuario tiene acceso de administrador), se selecciona la entidad raíz de la organización. Deje la raíz seleccionada y guarde la información de usuario. Al seleccionar la entidad raíz, se permite al usuario tener permiso de administrador no solo para la entidad raíz del árbol, sino también para todas sus entidades secundarias.  
  ![Ejemplo que muestra el acceso de administrador en el cuadro Agregar nuevo usuario](./media/tutorial-user-access/new-admin-access.png)

## <a name="create-a-user-with-user-access"></a>Crear un usuario con acceso de usuario
Los usuarios típicos que necesitan acceder a datos de Cloudyn, como paneles e informes, deben tener acceso de usuario para verlos. Cree un usuario nuevo con un acceso de usuario similar al que creó con acceso de administrador, con las siguientes diferencias:

- Desmarque **Allow User Management** (Permitir administración de usuarios), **Allow Recipient lists Management** (Permitir administración de listas de destinatarios) y todas las opciones de la lista **User has admin access** (El usuario tiene acceso de administrador).
- Seleccione las entidades a las que necesita acceso el usuario en la lista **User has user access** (El usuario tiene acceso de usuario).
- También puede permitir que el administrador acceda a entidades concretas, según sea necesario.

![Ejemplo que muestra el acceso de usuario en el cuadro Agregar nuevo usuario](./media/tutorial-user-access/new-user-access.png)

Para ver un tutorial en vídeo acerca de cómo agregar usuarios, consulte [Adding Users to Cloudyn](https://youtu.be/Nzn7GLahx30) (Adición de usuarios a Cloudyn).

## <a name="delete-a-user"></a>Eliminar un usuario

Cuando se elimina un usuario, todas las entidades a las que el usuario tiene acceso permanecen intactas. Los informes *personales* que están guardados se eliminan cuando se elimina el usuario. Los informes *públicos* creados por el usuario no se eliminan.

No puede eliminarse a sí mismo como usuario.

> [!WARNING]
> Cuando se elimina un usuario, no se puede restaurar.

1.    En el portal de Cloudyn, haga clic en el símbolo de engranaje en la esquina superior derecha y seleccione **User Management** (Administración de usuarios).
2.    En la lista de usuarios, seleccione el usuario que desea eliminar y, a continuación, haga clic en **Climinar usuario** (símbolo de la papelera).
3.    En el cuadro Eliminar el usuario, haga clic en **Sí** y, a continuación, haga clic en **Aceptar**.


## <a name="delete-or-export-personal-data"></a>Eliminar o exportar datos personales

Si desea eliminar o exportar datos personales de Cloudyn, deberá crear una incidencia de soporte técnico. Una vez creada la incidencia de soporte técnico, esta actúa como una solicitud formal: una solicitud de los titulares de los datos. Microsoft actúa con rapidez para eliminar la cuenta y cualquier dato personal o del cliente. Para más información sobre cómo puede solicitar que se eliminen o exporten sus datos, consulte [Data Subject Requests of Cloudyn Data](https://www.cloudyn.com/cloudyn-gdpr-requests) (Solicitud de los titulares de datos de Cloudyn).

## <a name="create-and-manage-entities"></a>Crear y administrar entidades

Cuando se define la jerarquía de entidades de costo, identificar la estructura de la organización es un procedimiento recomendado. Las entidades le permiten segmentar gastos por cuentas o suscripciones individuales. Va a crear entidades de costo para crear grupos lógicos y poder administrar y realizar un seguimiento de los gastos. Al crear el árbol, considere cómo quiere o necesita ver sus costos separados por unidades de negocio, centros de costo, entornos y departamentos de ventas. El árbol de entidades de Cloudyn es flexible debido a la herencia de la entidad.

Las suscripciones individuales para las cuentas de nube están vinculadas a entidades concretas. Puede asociar una entidad con una cuenta o suscripción de un proveedor de servicios en la nube. Por lo tanto, las entidades tienen varios inquilinos. Puede asignar acceso a usuarios específicos únicamente a su segmento del negocio mediante entidades. Si lo hace, los datos se mantienen aislados, incluso en grandes partes de un negocio, como subsidiarias. Y el aislamiento de los datos facilita la gobernanza.  

Cuando registró su cuenta o contrato de Azure con Cloudyn, se copiaron los datos de recursos de Azure, como el uso, el rendimiento, la facturación y los datos de etiquetas de las suscripciones a su cuenta de Cloudyn. Sin embargo, debe crear manualmente el árbol de entidades. Si omitió el registro de Azure Resource Manager, solo estarán disponibles en el portal de Cloudyn los datos de facturación y algunos informes de recursos.

En el portal de Cloudyn, haga clic en el símbolo de engranaje en la esquina superior derecha y seleccione **Cloud Accounts** (Cuentas en la nube). Empiece con una sola entidad (raíz) y cree el árbol de entidades a partir de la raíz. A continuación se incluye un ejemplo de una jerarquía de entidad que puede ser similar a muchas organizaciones de TI una vez completado el árbol:

![Ejemplo de un árbol de identidad que se muestra en la página Administración de cuentas](./media/tutorial-user-access/entity-tree.png)

Junto a **Entities** (Entidades), haga clic en **Add Entity** (Agregar entidad). Especifique información acerca de la persona o departamento que quiere agregar. No es necesario que los campos **Full Name** (Nombre completo) y **Email** (Correo electrónico) coincidan con usuarios existentes. Si quiere ver una lista de niveles de acceso, busque información en la ayuda para *agregar una entidad*.

![Ejemplo que muestra el nombre de la entidad y los niveles de acceso en el cuadro Agregar identidad](./media/tutorial-user-access/add-entity.png)

Al acabar, **guarde** la entidad.

### <a name="entity-access-levels"></a>Niveles de acceso de entidades

Los niveles de acceso de entidades junto con un acceso de usuario le permiten definir qué tipo de acciones están disponibles en el portal Cloudyn.

- **Enterprise** (Empresa): ofrece la posibilidad de crear y administrar entidades de costo secundarias.
- **Enterprise + Cost Allocation** (Empresa + Asignación de costos): ofrece la posibilidad de crear y administrar las entidades de costo secundarias incluida la asignación de costo para las cuentas consolidadas.
- **Enterprise, Cost based on parent cost allocation** (Empresa, costo basado en la asignación de costos principales): ofrece la posibilidad de crear y administrar entidades de costo secundarias. El costo de la cuenta se basa en el modelo de asignación de costos del elemento primario.
- **Custom Dashboards Only** (Solo paneles personalizados): hace que el usuario solo pueda ver los paneles personalizados predefinidos.
- **Dashboards Only** (Solo paneles) proporciona al usuario la capacidad de ver solo los paneles.

### <a name="create-a-cost-entity-hierarchy"></a>Creación de una jerarquía de entidades de costo

Para crear una jerarquía de entidades de costo, debe tener una cuenta con Enterprise o Enterprise + cost allocation (Empresa o Empresa + Asignación de costos).

En el portal de Cloudyn, haga clic en el símbolo de engranaje en la esquina superior derecha y seleccione **Cloud Accounts** (Cuentas en la nube). El árbol de **entidades** se muestra en el panel izquierdo. Si es necesario, expanda el árbol de entidades para que pueda ver la entidad que desea asociar a una cuenta.  Las cuentas de proveedor de servicios en la nube se muestran en pestañas en el panel derecho. Seleccione una pestaña y, a continuación, haga clic en una cuenta o suscripción y arrástrela a la entidad y colóquela. El cuadro **Mover** le informa de que la cuenta se movió correctamente. Haga clic en **OK**.

También puede asociar varias cuentas a una entidad. Seleccione las cuentas y, después, haga clic en **Mover**. En el cuadro Mover cuentas, seleccione la entidad a la que desea mover la cuenta y, a continuación, haga clic en **Guardar**. El cuadro Mover cuentas le pide que confirme que desea mover las cuentas. Haga clic en **Sí** y, luego, en **Aceptar**.

Para ver un tutorial en vídeo acerca de cómo crear una jerarquía de entidades de costo, consulte [Creating a Cost Entity Hierarchy in Cloudyn](https://youtu.be/dAd9G7u0FmU) (Creación de una jerarquía de entidades de costo en Cloudyn).

Los usuarios con Contrato Enterprise de Azure, pueden ver un tutorial en vídeo acerca de cómo asociar cuentas y suscripciones a entidades en [Connecting to Azure Resource Manager with Cloudyn](https://youtu.be/oCIwvfBB6kk) (Conexión a Azure Resource Manager con Cloudyn).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear un usuario con acceso de administrador
> * Crear un usuario con acceso de usuario
> * Eliminar un usuario
> * Eliminar o exportar datos personales
> * Crear y administrar entidades


Si aún no ha habilitado el acceso de la API de Azure Resource Manager en sus cuentas, consulte el artículo siguiente.

> [!div class="nextstepaction"]
> [Activación de suscripciones y cuentas de Azure](../../cost-management/activate-subs-accounts.md)
