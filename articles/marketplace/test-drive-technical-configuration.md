---
title: Configuración técnica de las versiones de prueba en Marketplace comercial de Microsoft
description: Más información sobre las versiones de prueba. Las versiones de prueba permiten que nuevos clientes prueben las ofertas antes de confirmar la compra.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 08/13/2019
author: keferna
ms.author: keferna
ms.openlocfilehash: a7afaf0da3bbb021d76c2d109cb385d86fd7c860
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319854"
---
# <a name="test-drive-technical-configuration"></a>Configuración técnica de la versión de prueba

La opción Versión de prueba de Marketplace comercial de Microsoft le permite configurar un paseo práctico y autoguiado por las características clave del producto. Con una versión de prueba, los nuevos clientes pueden probar la oferta antes de confirmar la compra. Para más información, consulte [¿Qué es la versión de prueba?](what-is-test-drive.md)

Si ya no quiere ofrecer una versión de prueba de la oferta, vuelva a la página **Configuración de la oferta** y desactive la casilla **Habilitar versión de prueba**. No todos los tipos de oferta tienen una versión de prueba disponible.

## <a name="azure-resource-manager-test-drive"></a>Versión de prueba de Azure Resource Manager

Es la única opción de versión de prueba para las ofertas de máquina virtual o de aplicación de Azure, y requiere una configuración bastante detallada. Lea las siguientes secciones [Detalles de suscripción de implementación](#deployment-subscription-details) y [Listas de versiones de prueba](#test-drive-listings) y, después, continúe con el tema independiente de [Configuración de versiones de prueba de Azure Resource Manager](azure-resource-manager-test-drive.md).

## <a name="hosted-test-drive"></a>Versión de prueba hospedada

Microsoft puede eliminar la complejidad de configurar una versión de prueba hospedando y manteniendo el aprovisionamiento y la implementación del servicio de mediante este tipo de versión de prueba. La configuración de este tipo de versión de prueba hospedada es la misma, independientemente de si la versión de prueba está destinada a un público de Dynamics 365 Business Central, Dynamics 365 Customer Engagement o Dynamics 365 Operations.

- **Máximo de versiones de prueba simultáneas** (obligatorio): establezca el número máximo de clientes que pueden usar la versión de prueba al mismo tiempo. Cada usuario simultáneo consumirá una licencia de Dynamics 365 mientras la versión de prueba esté activa, por lo que debe asegurarse de tener suficientes licencias disponibles para admitir el límite máximo establecido. Se recomienda un valor entre 3 y 5.

- **Duración de la versión de prueba** (obligatorio): especifique el número de horas que la versión de prueba permanecerá activa. Después de este período, la sesión finalizará y dejará de consumir una de las licencias. Se recomienda un valor que oscile entre 2 y 24 horas, en función de la complejidad de la oferta. Esta duración solo se puede establecer en horas completas (por ejemplo, "2" horas es válido; "1,5" no lo es). El usuario puede solicitar una nueva sesión si se agota el tiempo y desea volver a acceder a la versión de prueba.

- **URL de la instancia** (obligatorio): la dirección URL en la que el cliente comenzará su versión de prueba. Normalmente es la dirección URL de la instancia de Dynamics 365 en la que se ejecuta la aplicación con los datos de ejemplo instalados (por ejemplo, `https://testdrive.crm.dynamics.com`).

- **Dirección URL de API web de la instancia** (obligatorio): recupere la dirección URL de la API web de la instancia de Dynamics 365. Para ello, inicie sesión en su cuenta de Microsoft 365 y vaya a **Configuración** > **Personalización** > **Recursos para desarrolladores** > **API web de instancia (URL raíz de servicio)** . Copie la dirección URL que se encuentra aquí (por ejemplo, `https://testdrive.crm.dynamics.com/api/data/v9.0`).

- **Nombre del rol** (obligatorio): Proporcione el nombre del rol de seguridad que ha definido en la versión de prueba de Dynamics 365 personalizada, que se asignará al usuario durante la versión de prueba (por ejemplo, test-drive-role).

Para obtener ayuda sobre cómo configurar el entorno de Dynamics 365 para la versión de prueba y conceder permiso a AppSource para aprovisionar y desaprovisionar usuarios de la versión de prueba en el inquilino, siga [estas instrucciones](https://github.com/Microsoft/AppSource/blob/patch-1/Microsoft%20Hosted%20Test%20Drive/Setup-your-Azure-subscription-for-Dynamics365-Microsoft-Hosted-Test-Drives.md).

## <a name="logic-app-test-drive"></a>Versión de prueba de aplicación lógica

Microsoft no hospeda este tipo de versión de prueba. Úselo para conectarse con una oferta de Dynamics 365 u otro recurso personalizado, que abarca una variedad de arquitecturas de soluciones complejas. Para más información acerca de cómo configurar versiones de prueba de Logic App, visite [Operations](https://github.com/Microsoft/AppSource/blob/master/Setup-your-Azure-subscription-for-Dynamics365-Operations-Test-Drives.md) y [Customer Engagement](https://github.com/Microsoft/AppSource/wiki/Setting-up-Test-Drives-for-Dynamics-365-app) en GitHub.

- **Regiones** (obligatorio, lista desplegable de selección única): Actualmente hay 26 regiones admitidas de Azure en las que se puede usar la versión de prueba. Los recursos de la aplicación lógica se implementarán en la región que seleccione. Si la aplicación lógica tiene recursos personalizados almacenado en una región concreta, asegúrese de que se selecciona aquí dicha región. La mejor forma es implementar totalmente Logic App de forma local en la suscripción de Azure en el portal y comprobar que funciona correctamente antes de realizar esta selección.

- **Número máximo de versiones de prueba simultáneas** (obligatorio): Establezca el número máximo de clientes que puede usar la versión de prueba al mismo tiempo. Estas versiones de prueba ya están implementadas, lo que permitiendo a los clientes acceder al instante a ellas sin tener que esperar que se implementen.

- **Duración de la versión de prueba** (obligatorio): Especifique la duración en que la versión de prueba permanecerá activa, en número de horas. La versión de prueba termina automáticamente al finalizar este período.

- **Nombre del grupo de recursos de Azure:** (obligatorio): escriba el nombre del [grupo de recursos de Azure](../azure-resource-manager/resource-group-overview.md#resource-groups) en el que se guarda la versión de prueba de la aplicación lógica.

- **Asignar nombre de aplicación lógica** (obligatorio): escriba el nombre de la aplicación lógica que asigna la versión de prueba al usuario. Dicha aplicación lógica debe guardarse en el grupo de recursos de Azure anterior.

- **Desaprovisionar nombre de aplicación lógica** (obligatorio): escriba el nombre de la aplicación lógica que desaprovisiona la versión de prueba una vez que el cliente ha finalizado. Dicha aplicación lógica debe guardarse en el grupo de recursos de Azure anterior.

## <a name="power-bi-test-drive"></a>Versión de prueba de Power BI

Los productos que desean demostrar que un objeto visual interactivo de Power BI puede usar un vínculo insertado para compartir un panel personalizado como versión de prueba, no se necesita ninguna configuración técnica. Todo lo que debe hacer aquí es cargar la dirección URL insertada de Power BI.

Para obtener más información sobre cómo configurar aplicaciones de Power BI, consulte [¿Qué son las aplicaciones de plantilla de Power BI?](https://docs.microsoft.com/power-bi/service-template-apps-overview)

## <a name="deployment-subscription-details"></a>Detalles de suscripción de implementación

Para permitir que Microsoft implemente la versión de prueba en su nombre, cree y proporcione una suscripción de Azure independiente y única (no se requiere para las versiones de prueba de Power BI).

- **Id. de suscripción de Azure** (obligatorio para Azure Resource Manager y aplicaciones lógicas): escriba el identificador de la suscripción para conceder acceso a los servicios de la cuenta de Azure para la generación de informes y la facturación del uso de recursos. Se recomienda considerar la posibilidad de [crear una suscripción de Azure independiente](../cost-management-billing/manage/create-subscription.md) que se use para las versiones de prueba, en caso de que no se tenga ninguna. Para buscar su identificador de la suscripción de Azure inicie sesión en [Azure Portal](https://portal.azure.com/) y vaya a la pestaña **Suscripciones** del menú izquierdo. Al seleccionar la pestaña se mostrará el identificador de la suscripción (por ejemplo, "a83645ac-1234-5ab6-6789-1h234g764ghty").

- **Id. de inquilino de Azure AD** (obligatorio): escriba el [identificador de inquilino](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) de Azure Active Directory (AD). Para buscar dicho identificador, inicie sesión en [Azure Portal](https://portal.azure.com/), seleccione la pestaña Active Directory del menú de la izquierda, seleccione **Propiedades** y busque el número de **identificador de directorio** que aparece (por ejemplo, 50c464d3-4930-494c-963c-1e951d15360e). El ID de inquilino de su organización también lo puede buscar mediante la dirección URL de su nombre de dominio en: [https://www.whatismytenantid.com](https://www.whatismytenantid.com).

- **Nombre del inquilino de Azure AD** (obligatorio para Dynamic 365): Escriba el nombre de inquilino de Azure Active Directory (AD). Para buscar este nombre, inicie sesión en [Azure Portal](https://portal.azure.com/) y en la esquina superior derecha se mostrará su nombre del inquilino, debajo de su nombre de cuenta.

- **Identificador de aplicación de Azure AD** (obligatorio): escriba el [identificador de la aplicación](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) de Azure Active Directory (AD). Para buscarlo, inicie sesión en [Azure Portal](https://portal.azure.com/), seleccione la pestaña Active Directory en menú de la izquierda, elija **Registros de aplicaciones** y busque el número de **Id. de aplicación** que aparece (por ejemplo, `50c464d3-4930-494c-963c-1e951d15360e`).

- **Secreto del cliente de la aplicación Azure AD** (obligatorio): escriba el [secreto del cliente](../active-directory/develop/howto-create-service-principal-portal.md#create-a-new-application-secret) de su aplicación de Azure AD. Para encontrar este valor, inicie sesión en [Azure Portal](https://portal.azure.com/). Seleccione la pestaña **Azure Active Directory** en el menú izquierdo, seleccione **Registros de aplicaciones** y, después, seleccione la versión de prueba de su aplicación. A continuación, seleccione **Certificados y secretos**, seleccione **Nuevo secreto de cliente**, escriba una descripción, seleccione **Nunca** en **Expira** y, después, elija **Agregar**. No olvide copiar el valor. No salga de la página antes de copiar el valor.

## <a name="test-drive-listings"></a>Listas de versiones de prueba

La opción **Anuncios de versión de prueba** de la pestaña **Versión de prueba** del Centro de partners muestra los idiomas (y los mercados) en los que la versión de prueba está disponible; actualmente la única ubicación disponible es inglés (Estados Unidos). Además, esta página muestra el estado de la descripción específica del idioma y la fecha y hora en que se agregó. Deberá definir los detalles de unidad de prueba (descripción, manual del usuario, vídeos, etc.) para cada idioma o el mercado.

- **Descripción** (se requiere): se describe la versión de prueba, lo que se demostrará, los objetivos con los que experimentará el usuario, las características que se van a explorar y cualquier información relevante que ayude al usuario a determinar si debe adquirir la oferta. En este campo se puede escribir hasta 3000 caracteres de texto.

- **Información de acceso** (se requiere para las versiones de prueba de Azure Resource Manager y Logic): se explica todo lo que los clientes necesitan saber para acceder a esta versión de prueba y usarla. se muestra un escenario para usar la oferta y exactamente lo que el cliente debe saber para acceder a las características en la versión de prueba. En este campo se puede escribir hasta 10 000 caracteres de texto.

- **Manual del usuario** (se requiere): un tutorial detallado de la experiencia con la versión de prueba. Debe abarcar exactamente lo que desee que el cliente aprenda al usar la versión de prueba y puede servir como referencia para las preguntas que puedan surgirle. El archivo debe estar en formato PDF y hay que asignarle un nombre (255 caracteres como máximo) después de cargarlo.

- **Vídeos: agregar vídeos** (opcional): aquí se puede hacer referencia a los vídeos hospedados en otras ubicaciones con un vínculo y una imagen en miniatura (533 × 324 píxeles) para que los clientes puedan ver toda la información que pueda ayudarles a conocer mejor la versión de prueba, incluida la forma de usar correctamente las características de la oferta y comprender los escenarios que destacan sus ventajas.
  - **Nombre** (se requiere)
  - **URL** (solo YouTube o Vimeo; se requiere)
  - **Miniatura** (533 x 324 píxeles): la imagen debe estar en formato PNG.

Si está creando la versión de prueba en el Centro de partners, seleccione **Guardar borrador** antes de continuar.

## <a name="next-step"></a>Paso siguiente

- [Actualización de una oferta existente en Marketplace comercial](partner-center-portal/update-existing-offer.md)
