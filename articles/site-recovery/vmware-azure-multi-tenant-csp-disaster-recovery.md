---
title: Configuración de la recuperación ante desastres de VMware en Azure en un entorno multiinquilino mediante Site Recovery y el programa Proveedor de soluciones en la nube (CSP) | Microsoft Docs
description: Describe cómo configurar la recuperación ante desastres de VMware en un entorno multiinquilino con Azure Site Recovery.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/27/2018
ms.author: mayg
ms.openlocfilehash: b7cf0d35ae610c3f9fe477267e28e5449459148b
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87924587"
---
# <a name="set-up-vmware-disaster-recovery-in-a-multi-tenancy-environment-with-the-cloud-solution-provider-csp-program"></a>Configuración de la recuperación ante desastres de VMware en Azure en un entorno multiinquilino con el programa Proveedor de soluciones en la nube (CSP)

El [programa CSP](https://partner.microsoft.com/cloud-solution-provider) fomenta escenarios de colaboración para servicios en la nube de Microsoft, lo que incluye Office 365, Enterprise Mobility Suite y Microsoft Azure. Con CSP, los asociados tienen una relación completa con los clientes y se convierten en el principal punto de contacto de la relación. Los asociados pueden implementar suscripciones de Azure para los clientes y combinarlas con sus propias ofertas personalizadas de valor añadido.

Con [Azure Site Recovery](site-recovery-overview.md), los asociados pueden administrar la recuperación ante desastres de los clientes directamente mediante CSP. O bien, se puede usar CSP para configurar entornos de Site Recovery y dejar que los clientes administren sus propias necesidades de recuperación ante desastres en modo de autoservicio. En ambos escenarios, los asociados son el enlace entre Site Recovery y sus clientes. Los asociados atienden la relación con los clientes y les facturan por el uso de Site Recovery.

En este artículo se describe cómo los asociados pueden crear y administrar las suscripciones de los inquilinos mediante CSP para un escenario de replicación multiinquilino de VMware.

## <a name="prerequisites"></a>Prerequisites

Para configurar la replicación de VMware debe hacer lo siguiente:

- [Preparar](tutorial-prepare-azure.md) recursos de Azure, que incluyan una suscripción de Azure, una red virtual de Azure y una cuenta de almacenamiento.
- [Preparar](vmware-azure-tutorial-prepare-on-premises.md) las máquinas virtuales y los servidores de VMware locales.
- Para cada inquilino, cree un servidor de administración independiente que se comunique con las máquinas virtuales de inquilino y sus propios servidores vCenter. Solo el asociado debe tener derechos de acceso a este servidor de administración. Más información sobre los [entornos multiinquilino](vmware-azure-multi-tenant-overview.md).

## <a name="create-a-tenant-account"></a>Creación de una cuenta de inquilino

1. Mediante el [Centro de partners de Microsoft](https://partnercenter.microsoft.com/), inicie sesión en su cuenta de CSP.
2. En el menú **Panel**, seleccione **Clientes**.
3. En la página que se abre, haga clic en el botón **Agregar cliente**.
4. En la página **Nuevo cliente**, rellene los detalles de la cuenta del inquilino.

    ![La página Información de la cuenta](./media/vmware-azure-multi-tenant-csp-disaster-recovery/customer-add-filled.png)

5. A continuación, haga clic en **Siguiente: suscripciones**.
6. En la página de selección de suscripciones, seleccione la casilla **Microsoft Azure**. Puede agregar otras suscripciones ahora o en cualquier otro momento.
7. En la página **Revisar**, confirme los detalles del inquilino y, a continuación, haga clic en **Enviar**.
8. Después de haber creado la cuenta del inquilino, aparece una página de confirmación que muestra los detalles de la cuenta predeterminada y la contraseña de esa suscripción. Guarde la información y cambie la contraseña más adelante si es necesario mediante la página de inicio de sesión de Azure Portal.

Puede compartir esta información con el inquilino tal cual, o puede crear y compartir una cuenta independiente si es necesario.

## <a name="access-the-tenant-account"></a>Acceso a la cuenta de inquilino

Puede acceder a la suscripción del inquilino mediante el panel del centro de partners de Microsoft.

1. En la página **Clientes**, haga clic en el nombre de la cuenta del inquilino.
2. En la página **Suscripciones** de la cuenta del inquilino, puede supervisar las suscripciones existentes de la cuenta y agregar otras, según sea necesario.
3. Para administrar las operaciones de recuperación ante desastres del inquilino, seleccione **Todos los recursos (Azure Portal)** . Esto concede acceso a las suscripciones de Azure del inquilino.

    ![El vínculo Todos los recursos](./media/vmware-azure-multi-tenant-csp-disaster-recovery/all-resources-select.png)  

4. Para verificar el acceso, haga clic en el vínculo de Azure Active Directory de la parte superior derecha de Azure Portal.

    ![Vínculo de Azure Active Directory](./media/vmware-azure-multi-tenant-csp-disaster-recovery/aad-admin-display.png)

Ahora puede realizar todas las operaciones de Site Recovery para el inquilino mediante Azure Portal y administrarlas. Para acceder a la suscripción del inquilino mediante CSP para la recuperación ante desastres administrada, siga el proceso descrito anteriormente.

## <a name="assign-tenant-access-to-the-subscription"></a>Asignación del acceso de inquilino a la suscripción

1. Asegúrese de que esté configurada la infraestructura de recuperación ante desastres. Los asociados acceden a las suscripciones del inquilino mediante el portal de CSP, independientemente de si la recuperación ante desastres es administrada o autoservicio. Configure su propio almacén y registre la infraestructura en las suscripciones del inquilino.
2. Proporcione al inquilino la [cuenta que ha creado](#create-a-tenant-account).
3. También puede agregar un nuevo usuario a la suscripción de inquilino mediante el portal de CSP, de la manera siguiente:

    a) Vaya a la página de suscripción de CSP del inquilino y seleccione la opción **Users and licenses** (Usuarios y licencias).

      ![Página de suscripción de CSP del inquilino](./media/vmware-azure-multi-tenant-csp-disaster-recovery/users-and-licences.png)

    b) Ahora cree un nuevo usuario; para ello, escriba los datos pertinentes y seleccione los permisos, o bien cargue la lista de usuarios en un archivo CSV.
    
    c) Después de crear un usuario, vuelva Azure Portal. En la página **Suscripción**, seleccione la correspondiente.

    Seleccione **Control de acceso (IAM)** y después **Asignaciones de roles**.

    (e) Haga clic en **Agregar asignación de roles** para agregar un usuario con el nivel de acceso pertinente. Los usuarios que se crearon mediante el portal de CSP se muestran en la pestaña Asignaciones de roles.

      ![Adición de un usuario](./media/vmware-azure-multi-tenant-csp-disaster-recovery/add-user-subscription.png)

- Para la mayoría de las operaciones de administración, el rol de *Colaborador* es suficiente. Un usuario con este nivel de acceso puede hacerlo todo en una suscripción, excepto cambiar los niveles de acceso (para lo cual se necesita el nivel de acceso de *Propietario*).
- Site Recovery también tiene tres [roles de usuario predefinidos](site-recovery-role-based-linked-access-control.md) que se pueden usar para restringir aún más los niveles de acceso según sea necesario.

## <a name="multi-tenant-environments"></a>Entornos multiinquilino

Existen tres modelos principales multiinquilino:

* **Proveedor de servicios de hospedaje compartido (HSP)** : el asociado posee la infraestructura física y usa los recursos compartidos (vCenter, centros de datos, almacenamiento físico, etc.) para hospedar máquinas virtuales multiinquilino en la misma infraestructura. El asociado puede proporcionar administración de recuperación ante desastres como servicio administrado, o bien el inquilino puede ser propietario de la recuperación ante desastres como solución de autoservicio.

* **Proveedor de servicios de hospedaje dedicados**: el asociado posee la infraestructura física, pero usa recursos dedicados (varios vCenters, almacenes de datos físicos, etc.) para hospedar las máquinas virtuales de cada inquilino en infraestructuras diferentes. El asociado puede proporcionar administración de recuperación ante desastres como servicio administrado, o bien el inquilino puede poseerla como solución de autoservicio.

* **Proveedor de servicios administrados (MSP)** : aquí el cliente posee la infraestructura física que hospeda las máquinas virtuales y el asociado proporciona la habilitación y la administración de la recuperación ante desastres.

Mediante la configuración de suscripciones de los inquilinos como se describe en este artículo, puede empezar a habilitar rápidamente a los clientes en todos los modelos multiinquilino pertinentes. Puede aprender más acerca de los diferentes modelos de multiinquilino y la habilitación de controles de acceso locales [aquí](vmware-azure-multi-tenant-overview.md).

## <a name="next-steps"></a>Pasos siguientes
- Más información el [control de acceso basado en rol de Azure (Azure RBAC)](site-recovery-role-based-linked-access-control.md) para administrar implementaciones de Azure Site Recovery.
- [Más información](vmware-azure-architecture.md) sobre la arquitectura de replicación de VMware a Azure.
- [Revise el tutorial](vmware-azure-tutorial.md) de replicación de máquinas virtuales VMware en Azure.
Más información sobre [entornos multiinquilino](vmware-azure-multi-tenant-overview.md) para la replicación de máquinas virtuales VMware en Azure.
