---
title: Creación de una nueva oferta de servicio administrado en el Marketplace comercial de Microsoft
description: Cómo crear una oferta de servicio administrado para la lista de Azure Marketplace mediante el portal de Marketplace comercial en el Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: Microsoft-BradleyWright
ms.author: brwrigh
ms.date: 07/07/2020
ms.openlocfilehash: ae079fce02f58d6c6457d7fe6401526f4002662f
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87798195"
---
# <a name="create-a-managed-service-offer"></a>Creación de una oferta de servicio administrado

Servicio administrado ofrece ayuda para habilitar escenarios de [Azure Lighthouse](../../lighthouse/overview.md). Cuando un cliente acepta una oferta de servicio administrado, puede incorporar recursos para la [administración de recursos delegados de Azure](../../lighthouse/concepts/azure-delegated-resource-management.md). Antes de comenzar, [cree una cuenta de Marketplace comercial en el Centro de partners](create-account.md), si aún no lo ha hecho. Asegúrese de que la cuenta está inscrita en el programa de marketplace comercial.

Debe tener un [nivel de competencia de plataforma en la nube Silver o Gold](https://partner.microsoft.com/membership/cloud-platform-competency) o ser un [MSP experto de Azure](https://partner.microsoft.com/membership/azure-expert-msp) para publicar una oferta de servicio administrado.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Información general**.
3. En la página Información general, seleccione **+ Nueva oferta** > **Servicio administrado**.

    ![Muestra el menú de navegación izquierdo.](./media/new-offer-managed-service.png)

>[!NOTE]
>Después de publicar una oferta, las modificaciones que se realicen en el Centro de partners solo aparecen en los escaparates cuando la oferta se vuelve a publicar. Asegúrese de volver a realizar la publicación siempre después de realizar cambios.

## <a name="new-offer"></a>Nueva oferta

Escriba un **Identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

* Se muestra a los clientes en la dirección web de la oferta de Marketplace y en las plantillas de Azure Resource Manager, si procede.
* Use solo letras minúsculas y números. Puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres. Por ejemplo, si escribe **test-offer-1**, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
* El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

* Este nombre no se usa en Marketplace y es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.
* El alias de la oferta no se puede cambiar después de seleccionar **Crear**.

Seleccione **Crear** para generar la oferta y continuar.

## <a name="offer-setup"></a>Configuración de la oferta

### <a name="customer-leads"></a>Clientes potenciales

[!INCLUDE [Connect lead management](./includes/connect-lead-management.md)]

Según las [directivas de certificación de servicios administrados](https://docs.microsoft.com/legal/marketplace/certification-policies#700-managed-services), se requiere un **Destino de clientes potenciales**. Con esta acción se creará un registro en el sistema CRM cada vez que un cliente implemente la oferta.

Para obtener más información, consulte [Introducción a la administración de clientes potenciales](./commercial-marketplace-get-customer-leads.md).

Seleccione **Guardar borrador** antes de continuar.

## <a name="properties"></a>Propiedades

Esta página le permite definir las categorías que se usan para agrupar su oferta en el marketplace y los contratos legales que respaldan la oferta.

### <a name="category"></a>Category

Seleccione un mínimo de una y un máximo de cinco categorías, que se utilizarán para colocar la oferta en las áreas de búsqueda de Marketplace adecuadas. Asegúrese de indicar la forma en que la oferta da soporte a estas categorías en la descripción de la oferta.

### <a name="terms-and-conditions"></a>Términos y condiciones

Proporcione sus propios términos y condiciones legales en el campo **Términos y condiciones**. También puede proporcionar la dirección URL donde se pueden encontrar los términos y condiciones. Los clientes deberán aceptar estos términos para poder probar la oferta.

Seleccione **Guardar borrador** antes de continuar.

## <a name="offer-listing"></a>Descripción de la oferta

Esta página le permite definir los detalles del marketplace (como el nombre de la oferta, la descripción y las imágenes) para su oferta.

> [!NOTE]
> No es necesario que la oferta (por ejemplo, la descripción, los documentos, las capturas de pantallas y los términos de uso) esté en inglés si la descripción de la oferta comienza por la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]". También es aceptable proporcionar una *dirección URL de vínculo útil* para ofrecer contenido en un idioma distinto del que se usa en el contenido del anuncio de la oferta.

Este es un ejemplo de cómo aparece la información de la oferta en Azure Portal:

:::image type="content" source="media/example-managed-services.png" alt-text="Ilustración de cómo aparece esta oferta en Azure Portal.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Título
2. Descripción
3. Vínculos útiles
4. Capturas de pantalla

### <a name="name"></a>Nombre

El nombre que escriba aquí se mostrará a los clientes como el título de la descripción de la oferta. Este campo se ha rellenado previamente con el texto introducido para **Alias de la oferta** cuando creó la oferta, pero puede cambiar este valor. Este nombre puede contener marcas comerciales (y puede incluir símbolos de marca comercial o copyright). El nombre no puede tener más de 50 caracteres ni incluir emojis.

### <a name="search-results-summary"></a>Resumen de los resultados de la búsqueda

Especifique una descripción breve de la oferta (100 caracteres como máximo), que se va a usar en los resultados de la búsqueda de Marketplace.

### <a name="long-summary"></a>Resumen largo

Proporcione una descripción más larga de la oferta (hasta 256 caracteres). Este resumen largo también puede usarse en los resultados de la búsqueda de Marketplace.

### <a name="description"></a>Descripción

[!INCLUDE [Long description-1](./includes/long-description-1.md)]

[!INCLUDE [Long description-2](./includes/long-description-2.md)]

[!INCLUDE [Long description-3](./includes/long-description-3.md)]

### <a name="privacy-policy-link"></a>Vínculo de la directiva de privacidad

Escriba la dirección URL a la directiva de privacidad de la organización (hospedada en su sitio). Usted es el responsable no solo de garantizar que la aplicación cumple la normativa y la legislación en lo tocante a la privacidad, sino también de proporcionar una directiva de privacidad válida.

### <a name="useful-links"></a>Vínculos útiles

Proporcione documentos en línea complementarios opcionales sobre su solución. Para agregar más vínculos útiles, haga clic en **+ Agregar un vínculo**.

### <a name="contact-information"></a>Información de contacto

En esta sección, debe proporcionar el nombre, el correo electrónico y el número de teléfono de un **contacto de soporte técnico** y un **contacto de ingeniería**. Esta información no se muestra a los clientes, pero estará disponible para Microsoft y puede proporcionarse a los asociados de CSP.

### <a name="support-urls"></a>Direcciones URL de soporte técnico

Si tiene sitios web de soporte técnico para **clientes globales de Azure** o **clientes de Azure Government**, proporcione estas direcciones URL aquí.

### <a name="marketplace-images"></a>Imágenes de Marketplace

En esta sección, puede proporcionar logotipos e imágenes que se usarán al mostrar la oferta al cliente. Todas las imágenes deben estar en formato .PNG.

>[!NOTE]
>Si tiene un problema al cargar archivos, asegúrese de que la red local no bloquee el servicio https://upload.xboxlive.com que usa el Centro de partners.

#### <a name="store-logos"></a>Logotipos de Store

Proporcione un archivo PNG para el logotipo de tamaño **grande** (entre 216 x 216 y 350 x 350 píxeles). El Centro de partners lo utilizará para crear un logotipo de tamaño **pequeño** (48 x 48 píxeles) y un logotipo de tamaño **medio** (90 x 90 píxeles). Opcionalmente, puede reemplazarlos por imágenes diferentes.

Se requieren los tres tamaños de logotipo para su uso en distintos lugares de la lista:

- **Pequeño (48 x 48)**
- **Medio (90 x 90)**
- **Grande** (entre 216 x 216 y 350 x 350)

[!INCLUDE [Logo suggestions](./includes/graphics-suggestions.md)]

#### <a name="screenshots"></a>Capturas de pantalla

Agregue hasta cinco capturas de pantalla que muestren el funcionamiento de la oferta. Todas las capturas de pantallas deben tener 1280 x 720 píxeles.

#### <a name="videos"></a>Vídeos

Opcionalmente, puede agregar hasta cinco vídeos que muestren la oferta. Estos vídeos se deben hospedar en YouTube o Vimeo. En cada uno de ellos, escriba el nombre del vídeo, su dirección URL y una imagen en miniatura del vídeo (1280 x 720 píxeles).

#### <a name="additional-marketplace-listing-resources"></a>Recursos adicionales de la descripción de Marketplace

- [Procedimientos recomendados la descripción de ofertas en Marketplace](../gtm-offer-listing-best-practices.md)

Seleccione **Guardar borrador** antes de continuar.

## <a name="preview"></a>Versión preliminar

Antes de publicar su oferta en la oferta de Marketplace más amplia, primero deberá ponerla a disposición de un público preliminar limitado. Esto le permite confirmar cómo se muestra la oferta en Azure Marketplace antes de ponerla a disposición de los clientes. Los equipos de soporte técnico y de ingeniería de Microsoft también podrán ver su oferta durante este período de versión preliminar.

Para definir el público preliminar, escriba los identificadores de suscripción de Azure en la sección **Público preliminar**. Puede escribir hasta 10 identificadores de suscripción manualmente o cargar un archivo .csv con hasta 100 identificadores de suscripción.

Los clientes asociados con estas suscripciones podrán ver la oferta en Azure Marketplace antes de que se publique. Asegúrese de incluir sus propias suscripciones aquí para poder obtener la versión preliminar de la oferta.

Seleccione **Guardar borrador** antes de continuar.

## <a name="plan-overview"></a>Información general del plan

Cada oferta tiene que tener uno o más planes (anteriormente denominados SKU). Puede agregar varios planes para admitir diferentes conjuntos de características a precios diferentes o para personalizar un plan específico para un público limitado de clientes específicos. Los clientes pueden ver los planes disponibles para ellos en la oferta principal.

En la página **Información general del plan**, seleccione **+ Crear nuevo plan**. A continuación, escriba un **Id. de plan** y un **nombre de plan**. Ambos valores solo puede contener caracteres alfanuméricos en minúsculas, guiones y guiones bajos, con un máximo de 50 caracteres. Estos valores pueden ser visibles para los clientes y no se pueden cambiar después de publicar la oferta.

Seleccione **Crear** una vez que haya escrito estos valores para seguir trabajando en el plan. Hay tres secciones para completar: **Lista del plan**, **Precios y disponibilidad** y **Configuración técnica**.

### <a name="plan-listing"></a>Lista del plan

En primer lugar, proporcione un **Resumen de resultados de búsqueda** para el plan. Se trata una descripción breve del plan (100 caracteres como máximo), que se va a usar en los resultados de la búsqueda de Marketplace.

A continuación, escriba una **descripción** que proporcione una explicación más detallada del plan.

### <a name="pricing-and-availability"></a>Precios y disponibilidad

Actualmente, solo hay un modelo de precios que se puede usar para la oferta de servicio administrado: **Traiga su propia licencia (BYOL)** . Esto significa que facturará a sus clientes directamente por los costos relacionados con esta oferta y Microsoft no le aplicará ningún cargo.

La sección **Visibilidad del plan** le permite indicar si este plan debe ser [privado](../../marketplace/private-offers.md). Si deja el cuadro **Este plan es privado** desactivado, el plan no se restringirá a clientes específicos (o a un número determinado de clientes).

> [!NOTE]
> Los planes privados no son compatibles con las suscripciones que se establecen a través de un revendedor del programa Proveedor de soluciones en la nube (CSP).

Para que este plan esté disponible solo para clientes específicos, seleccione **Sí**. Al hacerlo, tendrá que identificar a los clientes proporcionando sus identificadores de suscripción. Se pueden especificar de uno en uno (hasta 10 suscripciones) o mediante la carga de un archivo. csv (hasta 10.000 suscripciones en todos los planes). Asegúrese de incluir sus propias suscripciones aquí para poder probar y validar la oferta.

> [!IMPORTANT]
> Una vez publicado un plan como público, no puede cambiarlo a privado. Para controlar qué clientes pueden aceptar su oferta y delegar recursos, use un plan privado. Con un plan público, no puede restringir la disponibilidad a determinados clientes ni a un determinado número de clientes, aunque puede dejar de vender el plan por completo si decide hacerlo. Puede [quitar el acceso a una delegación](../../lighthouse/how-to/remove-delegation.md) después de que un cliente acepte una oferta solo si incluyó una **autorización** con la **definición de rol** establecida en [Rol para eliminar la asignación de registros de servicios administrados](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) al publicar la oferta. También puede ponerse en contacto con el cliente y solicitarle [que quite el acceso](../../lighthouse/how-to/view-manage-service-providers.md#add-or-remove-service-provider-offers).

### <a name="technical-configuration"></a>Configuración técnica

Esta sección del plan crea un manifiesto con información de autorización para administrar los recursos del cliente. Esta información es necesaria para habilitar la [administración de recursos delegados de Azure](../../lighthouse/concepts/azure-delegated-resource-management.md).

Asegúrese de revisar los [inquilinos, roles y usuarios de los escenarios de Azure Lighthouse](../../lighthouse/concepts/tenants-users-roles.md#best-practices-for-defining-users-and-roles) para comprender qué roles se admiten y cuáles son los procedimientos recomendados para definir las autorizaciones.

> [!NOTE]
> Como se indicó anteriormente, los usuarios y los roles en las entradas de **Autorización** se aplicarán a todos los clientes que compren el plan. Si quiere limitar el acceso a un cliente específico, deberá publicar un plan privado para uso exclusivo.

#### <a name="manifest"></a>Manifest

En primer lugar, proporcione una **Versión** para el manifiesto. Use el formato *n.n.n* (por ejemplo, 1.2.5).

A continuación, escriba su **Id. de inquilino**. Se trata de un GUID asociado con el identificador del inquilino de Azure Active Directory (Azure AD) de la organización; es decir, el inquilino de administración desde el que obtendrá acceso a los recursos de sus clientes. Si no lo tiene a mano, para buscarlo, mantenga el mouse sobre el nombre de la cuenta en la parte superior derecha de Azure Portal o seleccione **Cambiar directorio**.

Si publica una nueva versión de la oferta y necesita crear un manifiesto actualizado, seleccione **+ Nuevo manifiesto**. Asegúrese de aumentar el número de versión de la versión anterior del manifiesto.

#### <a name="authorization"></a>Authorization

Las autorizaciones definen las entidades del inquilino de administración que pueden tener acceso a los recursos y las suscripciones para los clientes que compran el plan. A cada una de estas entidades se le asigna un rol integrado que concede niveles de acceso específicos.

Puede crear hasta veinte autorizaciones para cada plan.

> [!TIP]
> En la mayoría de los casos, querrá asignar roles a una entidad de servicio o un grupo de usuarios de Azure AD, en lugar de a una serie de cuentas de usuario individuales. Esto le permite agregar o quitar el acceso de usuarios individuales sin tener que actualizar y volver a publicar el plan cuando cambien los requisitos de acceso. Al asignar roles a grupos de Azure AD, [asegúrese de que el **tipo de grupo** sea **Seguridad** y no **Office 365**](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md). Para recomendaciones adicionales, consulte [Inquilinos, roles y usuarios en escenarios de Azure Lighthouse](../../lighthouse/concepts/tenants-users-roles.md).

Para cada **Autorización**, deberá proporcionar la siguiente información. Puede seleccionar **+ Agregar autorización** tantas veces como sea necesario para agregar más definiciones de roles y usuarios.

- **Id. de objeto de Azure AD**: identificador de Azure AD de un usuario, un grupo de usuarios o una aplicación al que se concederán determinados permisos (según se define en la definición de roles) para los recursos de los clientes.
- **Nombre para mostrar de objeto de Azure AD**: nombre descriptivo para ayudar al cliente a entender el propósito de esta autorización. El cliente verá este nombre al delegar recursos.
- **Role Definition**: seleccione uno de los roles integrados de Azure AD disponibles en la lista. Este rol determinará los permisos que el usuario del campo **Id. de objeto de Azure AD** tendrá en los recursos de los clientes. Para obtener descripciones de estos roles, consulte [Roles integrados](../../role-based-access-control/built-in-roles.md) y [Soporte de roles para Azure Lighthouse](../../lighthouse/concepts/tenants-users-roles.md#role-support-for-azure-lighthouse).
  > [!NOTE]
  > A medida que se agreguen nuevos roles integrados a Azure, estarán disponibles aquí, aunque pueden retrasarse un poco en aparecer.
- **Roles asignables**: esta opción solo aparece si ha seleccionado Administrador de acceso de usuario en **Definición de roles** para esta autorización. En ese caso, debe agregar uno o varios roles asignables aquí. El usuario del campo **Id. de objeto de Azure AD** podrá asignar estos roles a [identidades administradas](../../active-directory/managed-identities-azure-resources/overview.md), algo necesario para [implementar directivas que pueden corregirse](../../lighthouse/how-to/deploy-policy-remediation.md). Tenga en cuenta que no se aplicará a este usuario ningún otro permiso asociado normalmente al rol Administrador de acceso de usuario.

> [!TIP]
> Para asegurarse de que puede [quitar el acceso a una delegación](../../lighthouse/how-to/remove-delegation.md), si es necesario, incluya una **autorización** con la **definición de rol** establecida en [Rol para eliminar la asignación de registros de servicios administrados](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role). Si este rol no está asignado, solo un usuario puede quitar los recursos delegados del inquilino del cliente.

Una vez que haya completado todas las secciones del plan, puede seleccionar **+ Crear nuevo plan** tantas veces como necesite para crear planes adicionales. Cuando finalice, seleccione **Guardar**.

Seleccione **Guardar borrador** antes de continuar.

## <a name="publish"></a>Publicar

### <a name="submit-offer-to-preview"></a>Envío de la oferta para tener una versión preliminar

Una vez que haya completado todas las secciones necesarias de la oferta, seleccione **Revisar y publicar** en la esquina superior derecha del portal.

Si es la primera vez que publica esta oferta, puede:

- Ver el estado de finalización de cada sección de la oferta.
  - **No iniciada**: la sección no se ha modificado y se debe completar.
  - **Incompleta**: la sección tiene errores que deben corregirse o se requiere más información. Vuelva a las secciones y actualícelas.
  - **Completa**: la sección está completa, se han proporcionado todos los datos necesarios y no hay ningún error. Para poder enviar la oferta, todas sus secciones deben estar en estado Completa.
- En la sección **Notas para la certificación**, envíe instrucciones para la realización de pruebas al equipo de certificación para asegurarse de que la aplicación se prueba correctamente, además de todas las notas adicionales que pueden resultar útiles para conocer la aplicación.
- Enviar la oferta para su publicación, para lo que debe seleccionar **Enviar**. Le enviaremos un correo electrónico cuando haya una versión preliminar de la oferta disponible para su revisión y aprobación. Vuelva al Centro de partners y seleccionar **Transmitir** la oferta para publicar la oferta para el público (o si es una oferta privada, al público privado).

### <a name="customer-experience-and-offer-management"></a>Experiencia del cliente y administración de ofertas

Cuando un cliente implemente su oferta, podrá delegar uno o varios grupos de recursos o suscripciones para la [administración de recursos delegada de Azure](../../lighthouse/concepts/azure-delegated-resource-management.md). Para obtener más información acerca de este proceso, consulte [Proceso de incorporación del cliente](../../lighthouse/how-to/publish-managed-services-offers.md#the-customer-onboarding-process).

Puede [publicar una versión actualizada de la oferta](update-existing-offer.md) en cualquier momento. Por ejemplo, puede querer agregar una nueva definición de roles a una oferta publicada previamente. Al hacerlo, los clientes que ya hayan agregado la oferta verán un icono en la página [**Proveedores de servicios**](../../lighthouse/how-to/view-manage-service-providers.md) de Azure Portal que les informa que hay disponible una actualización. Cada cliente podrá revisar los cambios y decidir si quiere actualizar a la nueva versión.

## <a name="next-steps"></a>Pasos siguientes

- [Actualización de una oferta existente en Marketplace comercial](./update-existing-offer.md)
- [Más información acerca de Azure Lighthouse](../../lighthouse/overview.md)
