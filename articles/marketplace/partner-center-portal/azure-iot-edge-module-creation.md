---
title: Creación de una oferta de módulo Azure IoT Edge con el Centro de partners en Azure Marketplace
description: Aprenda a crear, configurar y publicar una oferta de módulo IoT Edge en Azure Marketplace mediante el Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: keferna
ms.author: keferna
ms.date: 07/22/2020
ms.openlocfilehash: 779fc1cfc617e18eee4d84dcfd007f10ddb31f96
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87808718"
---
# <a name="create-an-iot-edge-module-offer"></a>Creación de una oferta de módulo de IoT Edge

En este artículo se explica cómo crear y publicar una entrada de oferta de módulo IoT Edge para Azure Marketplace. Antes de comenzar, [cree una cuenta de Marketplace comercial en el Centro de partners](create-account.md), si aún no lo ha hecho. Asegúrese de que la cuenta está inscrita en el programa Marketplace comercial.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Información general**.
3. En la página Información general, seleccione **+ Nueva oferta** > **Módulo IoT Edge**.

    ![Muestra el menú de navegación izquierdo.](./media/new-offer-iot-edge.png)

> [!IMPORTANT]
> Después de publicar una oferta, las modificaciones que se realicen en el Centro de partners solo aparecen en los escaparates cuando la oferta se vuelve a publicar. Asegúrese de volver a realizar la publicación siempre después de realizar cambios.

### <a name="offer-id-and-alias"></a>Identificador y alias de la oferta

Escriba un **Identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

- Se muestra a los clientes en la dirección web de la oferta de Marketplace y en las plantillas de Azure Resource Manager, si procede.
- Use solo letras minúsculas y números. Puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres. Por ejemplo, si escribe **test-offer-1**, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
- El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

- Este nombre no se usa en Marketplace y es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.
- No se puede cambiar después de seleccionar **Crear**.

Seleccione **Crear** para generar la oferta y continuar.

## <a name="offer-overview"></a>Información general de la oferta

La página **Información general de la oferta** muestra una representación visual de los pasos necesarios para publicar esta oferta (tanto completados como siguientes), así como el tiempo necesario que debe tardar cada uno en completarse.

En esta página se incluyen vínculos para realizar operaciones en esta oferta en función de la selección que realice. Por ejemplo:

- Si la oferta es un borrador: [Eliminación de una oferta borrador](update-existing-offer.md#delete-a-draft-offer)
- Si la oferta está activa: [Detención de la venta de una oferta](update-existing-offer.md#stop-selling-an-offer-or-plan)
- Si la oferta está en versión preliminar: [Publicación](publishing-status.md#publisher-approval)
- Si no ha completado el cierre de sesión del anunciante: [Cancelación de la publicación.](update-existing-offer.md#cancel-publishing)

## <a name="offer-setup"></a>Configuración de la oferta

Siga estos pasos para configurar la oferta.

### <a name="customer-leads"></a>Clientes potenciales

Al publicar la oferta en Marketplace con el Centro de partners, puede conectarla a un sistema de administración de relaciones con clientes (CRM). Esto le permite recibir información de contacto del cliente en cuanto alguien expresa interés en el producto o lo usa.

1. Seleccione un destino de clientes potenciales al que quiera que enviemos los clientes potenciales. El Centro de partners admite los siguientes sistemas CRM:

    - [Dynamics 365](commercial-marketplace-lead-management-instructions-dynamics.md) for Customer Engagement
    - [Marketo](commercial-marketplace-lead-management-instructions-marketo.md)
    - [Salesforce](commercial-marketplace-lead-management-instructions-salesforce.md)

    > [!NOTE]
    > Si su sistema CRM no aparece en la lista anterior, use [Tabla de Azure](commercial-marketplace-lead-management-instructions-azure-table.md) o [Punto de conexión HTTPS](commercial-marketplace-lead-management-instructions-https.md) para almacenar los datos del cliente potencial y, a continuación, exporte los datos al sistema CRM.

2. Conecte su oferta al destino de clientes potenciales cuando la publique en el Centro de partners.
3. Compruebe que la conexión al destino de clientes potenciales está configurada correctamente. Después de publicarla en el Centro de partners, se valida la conexión y se le envía un cliente potencial de prueba. Mientras obtiene una vista previa de la oferta antes de publicarla, también puede probar la conexión de los clientes potenciales tratando de adquirir la oferta en el entorno de versión preliminar.
4. Asegúrese de que la conexión con el destino de clientes potenciales permanece actualizada para no perder ningún cliente potencial.

Estos son algunos recursos adicionales de administración de clientes potenciales:

- [Introducción a la administración de clientes potenciales](commercial-marketplace-get-customer-leads.md)
- [Preguntas frecuentes de la administración de clientes potenciales](../lead-management-for-cloud-marketplace.md#frequently-asked-questions)
- [Errores comunes de la configuración de clientes potenciales](../lead-management-for-cloud-marketplace.md#publishing-config-errors)
- [Introducción a la administración de clientes potenciales](https://assetsprod.microsoft.com/mpn/cloud-marketplace-lead-management.pdf) en formato PDF (asegúrese de que el bloqueador de elementos emergentes está desactivado).

Seleccione **Guardar borrador** antes de continuar.

### <a name="properties"></a>Propiedades

Esta página le permite definir las categorías que se usan para agrupar su oferta en Marketplace y los contratos legales que dan soporte a su oferta.

#### <a name="category"></a>Category

Seleccione categorías y subcategorías para colocar la oferta en las áreas de búsqueda adecuadas del marketplace. En la descripción de la oferta, asegúrese de indicar la forma en que la oferta admite estas categorías. Seleccione:

- Al menos una y un máximo de dos categorías, incluidas una categoría principal y una secundaria (opcional).
- Hasta dos subcategorías para cada categoría principal o secundaria. Si no hay ninguna subcategoría aplicable a la oferta, seleccione **No aplicable**.

Consulte la lista completa de categorías y subcategorías en [Procedimientos recomendados para la publicación de ofertas](../gtm-offer-listing-best-practices.md). En el marketplace, los módulos IoT Edge aparecen siempre en la categoría  **Internet de las cosas** > **Módulo IoT Edge** .

#### <a name="legal"></a>Información legal

Debe proporcionar los términos y condiciones de la oferta. Tiene dos opciones:

- Usar el contrato estándar para Marketplace comercial de Microsoft.
- Proporcionar sus propios términos y condiciones.

##### <a name="standard-contract-for-the-microsoft-commercial-marketplace"></a>Contrato estándar para Marketplace comercial de Microsoft

Ofrecemos una plantilla de contrato estándar para facilitar las transacciones en Marketplace comercial. Puede optar por ofrecer su solución con el contrato estándar, que los clientes solo deben consultar y aceptar una vez. Esta es una buena opción si no desea crear términos y condiciones personalizados.

Para obtener más información sobre el contrato estándar, consulte [Contrato estándar para Marketplace comercial de Microsoft](../standard-contract.md). También puede descargar el [contrato estándar](https://go.microsoft.com/fwlink/?linkid=2041178) en formato PDF (asegúrese de que el bloqueador de elementos emergentes está desactivado).

Para usar el contrato estándar, active la casilla **Use the Standard Contract for Microsoft's commercial marketplace** (Usar el contrato estándar para Marketplace comercial de Microsoft) y, a continuación, haga clic en **Aceptar**.

> [!NOTE]
> Después de publicar una oferta con el contrato estándar para Marketplace comercial de Microsoft, no puede usar sus propios términos y condiciones personalizados. Puede ofrecer su solución con el Contrato estándar o con sus propios términos y condiciones.

![Muestra el uso de la casilla del contrato estándar para Marketplace comercial de Microsoft.](media//iot-edge-module-standard-contract-checkbox.png)

##### <a name="your-own-terms-and-conditions"></a>Términos y condiciones propios

Si quiere proporcionar sus propios términos y condiciones personalizados, especifíquelos en el cuadro **Términos y condiciones**. Puede escribir un número ilimitado de caracteres de texto en este cuadro. Los clientes deben aceptar estos términos para poder probar la oferta.

Seleccione **Guardar borrador** antes de pasar a la siguiente sección, Descripción de la oferta.

## <a name="offer-listing"></a>Descripción de la oferta

Aquí puede definir los detalles de la oferta que se muestran en Marketplace. Esto incluye el nombre de la oferta, la descripción, las imágenes, etc. Asegúrese de seguir las directivas que se detallan en la página de directivas de Microsoft mientras configura esta oferta.

> [!NOTE]
> No es necesario que los detalles de la oferta estén en inglés si la descripción de la oferta comienza con la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]." También es puede proporcionar un vínculo al contenido de la oferta en un idioma distinto al utilizado en la descripción de la oferta.

### <a name="name"></a>Nombre

El nombre que se escribe aquí se muestra como el título de la oferta. Este campo se rellena previamente con el texto que escribió en el cuadro **Alias de la oferta** al crear la oferta. Puede cambiar este nombre más adelante.

El nombre:

- Puede contener marcas comerciales y, además, puede incluir símbolos de marca comercial o copyright.
- No puede tener más de 50 caracteres.
- No puede incluir emojis.

### <a name="search-results-summary"></a>Resumen de los resultados de la búsqueda

Proporcione una breve descripción de la oferta. Puede tener una longitud de hasta 100 caracteres y se usa en los resultados de la búsqueda de Marketplace.

### <a name="long-summary"></a>Resumen largo

Proporcione una descripción más detallada de la oferta. Puede tener una longitud de hasta 256 caracteres y se usa en los resultados de la búsqueda de Marketplace.

### <a name="description"></a>Descripción

[!INCLUDE [Long description-1](./includes/long-description-1.md)]

Las ofertas de módulos IoT Edge deben incluir un párrafo de requisitos mínimos de hardware en la parte inferior de la descripción, de la siguiente manera:

- Requisitos mínimos de hardware: sistema operativo Linux x64 y arm32, 1 GB de RAM y 500 MB de almacenamiento

[!INCLUDE [Long description-2](./includes/long-description-2.md)]

[!INCLUDE [Long description-3](./includes/long-description-3.md)]

#### <a name="privacy-policy-url"></a>Dirección URL de la directiva de privacidad

Escriba la dirección web de la directiva de privacidad de su organización. Usted es responsable de garantizar que la oferta cumpla con las leyes y reglamentos de privacidad. También es responsable de publicar una directiva de privacidad válida en su sitio web.

#### <a name="useful-links"></a>Vínculos útiles

Proporcione documentos en línea complementarios sobre su oferta. Puede agregar hasta 25 vínculos. Para agregar un vínculo, seleccione **+ Agregar un vínculo** y, a continuación, rellene los campos siguientes:

- **Título**: los clientes verán el título en la página de detalles de la oferta.
- **Vínculo (URL)** : escriba un vínculo para que los clientes vean el documento en línea. El vínculo debe comenzar con `http://` o `https://`.

Asegúrese de agregar al menos un vínculo a la documentación y un vínculo a los dispositivos IoT Edge compatibles desde el  [catálogo de dispositivos IoT de Azure](https://catalog.azureiotsolutions.com/).

### <a name="contact-information"></a>Información de contacto

Debe proporcionar el nombre, el correo electrónico y el número de teléfono de un **contacto de soporte técnico** y un **contacto de ingeniería**. Esta información no se muestra a los clientes. Está disponible para Microsoft y se puede proporcionar a los asociados del programa Proveedor de soluciones en la nube (CSP).

- Contacto de soporte técnico (se requiere): Para preguntas generales de soporte técnico.
- Contacto de ingeniería (se requiere): Para preguntas técnicas y problemas de certificación.
- Contacto del programa CSP (opcional): Para preguntas de revendedores relacionadas con el programa CSP.

En la sección **Contacto de soporte técnico**, proporcione la dirección web del **Sitio web de soporte técnico**, donde los asociados pueden encontrar soporte técnico para su oferta, esté disponible en Azure global, Azure Government o ambos.

En la sección **Contacto del programa CSP**, proporcione el vínculo (**Materiales de marketing del programa CSP**), donde los asociados del programa CSP pueden encontrar materiales de marketing para su oferta.

#### <a name="additional-marketplace-listing-resources"></a>Recursos adicionales de la descripción de Marketplace

Para obtener más información sobre la creación de listas de ofertas, consulte [Procedimientos recomendados para la publicación de ofertas](../gtm-offer-listing-best-practices.md).

### <a name="marketplace-images"></a>Imágenes de Marketplace

Proporcione logotipos e imágenes para usarlos con la oferta. Todas las imágenes deben estar en formato PNG. Las imágenes borrosas se rechazarán.

>[!Note]
>Si tiene un problema al cargar archivos, asegúrese de que la red local no bloquee el servicio https://upload.xboxlive.com que usa el Centro de partners.

#### <a name="store-logos"></a>Logotipos de Store

Proporcione un archivo PNG para el logotipo de tamaño **grande** (entre 216 x 216 y 350 x 350 píxeles). El Centro de partners lo utilizará para crear un logotipo de tamaño **pequeño** (48 x 48 píxeles) y un logotipo de tamaño **medio** (90 x 90 píxeles). Opcionalmente, puede reemplazarlos por imágenes diferentes.

Se requieren los tres tamaños de logotipo para su uso en distintos lugares de la lista:

- **Pequeño (48 x 48)**
- **Medio (90 x 90)**
- **Grande** (entre 216 x 216 y 350 x 350)

[!INCLUDE [Logo suggestions](./includes/graphics-suggestions.md)]

#### <a name="screenshots-optional"></a>Capturas de pantalla (opcional)

Agregue hasta cinco capturas de pantalla que muestren el funcionamiento de la oferta. Cada una debe tener un tamaño de 1280 x 720 píxeles y el formato PNG.

#### <a name="videos-optional"></a>Vídeos (opcional)

Agregue hasta cinco vídeos que muestren la oferta. Escriba el nombre del vídeo, su dirección web y una imagen en miniatura en formato PNG del vídeo con un tamaño de 1280 x 720 píxeles.

#### <a name="marketplace--examples"></a>Ejemplos de marketplace

Este es un ejemplo de cómo aparece la información de la oferta en Azure Marketplace:

:::image type="content" source="media/example-iot-azure-marketplace-offer.png" alt-text="Ilustración de cómo aparece esta oferta en Azure Marketplace.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo grande
2. Categorías
3. Dirección de soporte técnico (vínculo)
4. Términos y condiciones
5. Dirección de la directiva de privacidad (vínculo)
6. Nombre
7. Resumen
8. Descripción
9. Vínculos útiles
10. Capturas de pantallas o vídeos

<br>Este es un ejemplo de cómo aparece la información de la oferta en los resultados de la búsqueda de Azure Marketplace:

:::image type="content" source="media/example-iot-azure-marketplace-offer-search-results.png" alt-text="Ilustración de cómo aparece esta oferta en los resultados de la búsqueda de Azure Marketplace.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo pequeño
2. Nombre de la oferta
3. Resumen de los resultados de la búsqueda

<br>Este es un ejemplo de cómo aparece la información de la oferta en Azure Portal:

:::image type="content" source="media/example-iot-azure-portal-offer.png" alt-text="Ilustración de cómo aparece esta oferta en Azure Portal.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Nombre
2. Descripción
3. Vínculos útiles
4. Capturas de pantalla

<br>Este es un ejemplo de cómo aparece la información de la oferta en los resultados de la búsqueda de Azure Portal:

:::image type="content" source="media/example-iot-azure-portal-offer-search-results.png" alt-text="Ilustración de cómo aparece esta oferta en los resultados de la búsqueda de Azure Portal.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo pequeño
2. Nombre de la oferta
3. Resumen de los resultados de la búsqueda

<br>Seleccione **Guardar borrador** antes de continuar con la siguiente sección, Versión preliminar.

## <a name="preview"></a>Versión preliminar

La **pestaña Versión preliminar** permite elegir un **público preliminar** limitado para validar la oferta antes de ponerla a disposición de un público más amplio de Marketplace.

> [!IMPORTANT]
> Después de ver la oferta en Versión preliminar, debe seleccionar **Transmitir** para publicar su oferta para el público.

Especifique el público de versión preliminar mediante los GUI de identificador de suscripción de Azure con una descripción adicional de cada uno. Los clientes no pueden ver ninguno de estos campos.

> [!NOTE]
> Puede encontrar el id. de suscripción de Azure en la página Suscripciones de Azure Portal.

Agregue al menos un id. de suscripción de Azure, ya sea de forma individual (hasta 10) o mediante la carga de un archivo CSV (hasta 100). Al agregar estos id. de suscripción, puede definir quién puede obtener una vista previa de la oferta antes de publicarla. Aunque la oferta ya esté publicada, puede definir un público preliminar para probar los cambios o las actualizaciones pertinentes.

Seleccione **Guardar borrador** antes de continuar con la siguiente sección, Información general del plan.

### <a name="plan-overview"></a>Información general del plan

Esta pestaña le permite ofrecer diversas opciones de planes dentro de la misma oferta en el Centro de partners. Los planes (anteriormente denominados SKU) pueden diferir en cuanto a las nubes disponibles, como las nubes globales, las nubes gubernamentales y la imagen de referencia del plan. Para publicar su oferta en Marketplace, debe configurar al menos un plan.

Después de crear sus planes, la pestaña **Información general del plan** muestra lo siguiente:

- Nombres de los planes
- Modelo de precios
- Regiones de Azure (globales o gubernamentales)
- Estado de publicación actual
- Acciones disponibles

Las acciones disponibles en Información general del plan varían en función del estado actual del plan del plan. Incluyen:

- **Eliminar borrador**: si el estado del plan es un borrador.
- **Detener el plan de venta**: si el estado el plan es publicado.

#### <a name="create-new-plan"></a>Creación de un nuevo plan

Seleccione **Crear nuevo plan**. Aparece el cuadro de diálogo **Nuevo plan**.

En el cuadro **Id. de plan**, cree un identificador de plan único para cada plan de esta oferta. Este identificador será visible para los clientes en la dirección web del producto. Use solo letras minúsculas y números, guiones o caracteres de subrayado, y un máximo de 50 caracteres.

En el cuadro **Nombre del plan**, escriba un nombre para este plan. Los clientes verán este nombre al decidir que plan van a seleccionar en su oferta. Cree un nombre único para cada plan de esta oferta. Por ejemplo, puede usar el nombre de oferta **Windows Server** con los planes **Windows Server 2016** y **Windows Server 2019**.

> [!NOTE]
> El id. de oferta no se puede cambiar después de seleccionar **Crear**.

Seleccione **Crear**.

### <a name="plan-setup"></a>Configuración del plan

Esta pestaña le permite configurar las nubes en las que está disponible el plan. Sus respuestas de esta pestaña afectan a los campos que se muestran en otras pestañas.

#### <a name="azure-regions"></a>Regiones de Azure

Todos los planes de las ofertas del módulo de IoT Edge están disponibles automáticamente en **Azure Global**.  Los clientes pueden usar su plan en todas las regiones globales de Azure que usan el marketplace. Para obtener más información, consulte [Disponibilidad geográfica y soporte técnico de moneda](../marketplace-geo-availability-currencies.md).

Seleccione la opción [Azure Government](../../azure-government/documentation-government-welcome.md) para que la solución aparezca aquí. Es una nube de la comunidad gubernamental con acceso controlado para los clientes de agencias gubernamentales tribales, locales, estatales o federales de Estados Unidos, así como para los asociados aptos para abastecerlos. Como publicador, es responsable de los controles de cumplimiento, las medidas de seguridad y los procedimientos recomendados para esta comunidad en la nube. Azure Government usa redes y centros de datos aislados físicamente (ubicados solo en Estados Unidos). Antes de la [publicación](../../azure-government/documentation-government-manage-marketplace-partners.md) en Azure Government, pruebe y valide la solución en esa área, ya que los resultados pueden ser diferentes. Para almacenar y realizar una copia intermedia de la solución, solicite una cuenta de prueba desde [Prueba de Microsoft Azure Government](https://azure.microsoft.com/global-infrastructure/government/request/).

> [!NOTE]
> Una vez que el plan esté publicado y disponible en una región específica, no se puede quitar esa región.

#### <a name="azure-government-certifications"></a>Certificaciones de Azure Government

Esta opción es visible únicamente si **Azure Government** está seleccionado en **Regiones de Azure**.

Los servicios de Azure Government controlan datos que están sujetos a determinados reglamentos y requisitos gubernamentales. Por ejemplo, FedRAMP, NIST 800.171 (DIB), ITAR, IRS 1075, DoD L4 y CJIS. Para dar a conocer sus certificaciones de estos programas, puede proporcionar hasta 100 vínculos que las describan. Pueden ser vínculos a sus listas en el programa directamente o a su propio sitio web. Estos vínculos solo son visibles para los clientes de Azure Government.

## <a name="plan-listing"></a>Lista del plan

Esta pestaña muestra información específica de cada plan diferente de la misma oferta.

### <a name="plan-name"></a>Nombre del plan

Se rellena previamente con el nombre que asignó al plan cuando lo creó. Puede cambiar este nombre según sea necesario. Puede tener una longitud máxima de 50 caracteres. Este nombre aparece como el título de este plan en Azure Marketplace y Azure Portal. Se usa como el nombre de módulo predeterminado cuando el plan está listo para usarse.

### <a name="plan-summary"></a>Resumen del plan

Proporcione un breve resumen del plan (no de la oferta). Este resumen aparece en los resultados de la búsqueda de Azure Marketplace y puede contener hasta 100 caracteres.

### <a name="plan-description"></a>Description del plan

Describa lo que hace que este plan sea único, así como las diferencias entre los planes de la oferta. No describa la oferta, solo el plan. Esta descripción aparecerá en Azure Marketplace y en Azure Portal en la página Descripción de la oferta. Puede ser el mismo contenido que proporcionó en el resumen del plan y contener hasta 2000 caracteres.

Después de completar estos campos seleccione **Guardar borrador**.

#### <a name="plan-examples"></a>Ejemplos de plan

A continuación se muestra un ejemplo de los detalles del plan de Azure Marketplace (los precios mostrados son meramente ilustrativos y no pretenden reflejar los costos reales):

:::image type="content" source="media/example-iot-azure-marketplace-plan.png" alt-text="Ilustración de los detalles del plan de Azure Marketplace.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Nombre de la oferta
2. Nombre del plan
3. Description del plan

<br>A continuación se muestra un ejemplo de los detalles del plan en Azure Portal (los precios mostrados son meramente ilustrativos y no pretenden reflejar los costos reales):

:::image type="content" source="media/example-iot-azure-marketplace-plan-details.png" alt-text="Ilustración de los detalles del plan en Azure Portal.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Nombre de la oferta
2. Nombre del plan
3. Description del plan

## <a name="availability"></a>Disponibilidad

Si desea ocultar la oferta publicada para que los clientes no puedan buscarla, examinarla ni comprarla en Marketplace, active la casilla **Ocultación del plan** en la pestaña Disponibilidad.

Este campo se utiliza normalmente cuando:

- La oferta está pensada para usarse solo indirectamente cuando se hace referencia a esta a través de otra aplicación.
- La oferta no debe adquirirse individualmente.
- El plan se usó para las pruebas iniciales y ya no es relevante.
- El plan se ha usado para las ofertas temporales o estacionales y ya no debe ofrecerse.

## <a name="technical-configuration"></a>Configuración técnica

El tipo de oferta **Módulo IoT Edge** es un tipo específico de contenedor que se ejecuta en un dispositivo IoT Edge. En la pestaña **Configuración técnica**, proporcionará información de referencia para el repositorio de imágenes de contenedor en [Azure Container Registry](https://azure.microsoft.com/services/container-registry/), junto con los valores de configuración que permiten a los clientes usar el módulo fácilmente.

Una vez publicada la oferta, la imagen de contenedor de IoT Edge se copia en Azure Marketplace en un registro de contenedor público específico. Todas las solicitudes de los usuarios de Azure que usan su módulo se atienden desde el registro de contenedor público de Azure Marketplace, no desde su registro de contenedor privado.

Puede elegir como destino varias plataformas y proporcionar varias versiones a la imagen de contenedor del módulo por medio de etiquetas. Para obtener más información sobre etiquetas y control de versiones, consulte [Preparar los recursos técnicos del módulo IoT Edge](create-iot-edge-module-asset.md).

### <a name="image-repository-details"></a>Detalles del repositorio de imágenes

Proporcionará la siguiente información en la pestaña **Detalles del repositorio de imágenes**.

**Seleccionar el origen de la imagen**: seleccione la opción **Registro de Azure Container Registry**.

**Id. de suscripción a Azure**: proporcione el identificador de suscripción en el que se indica el uso de recursos y los servicios se facturan por la instancia de Azure Container Registry que incluye la imagen de contenedor. Puede encontrar este identificador en la [página Suscripciones](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) de Azure Portal.

**Nombre del grupo de recursos de Azure**: proporcione el nombre del [grupo de recursos](../../azure-resource-manager/management/manage-resource-groups-portal.md) que contiene la instancia de Azure Container Registry con la imagen de contenedor. El grupo de recursos debe ser accesible en el identificador de suscripción (anterior). Puede encontrar el nombre en la página [Grupos de recursos](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceGroups) en Azure Portal.

**Nombre de Azure Container Registry**: proporcione el nombre de la instancia de [Azure Container Registry](../../container-registry/container-registry-intro.md) que tiene su imagen de contenedor. El registro de contenedor debe estar presente en el grupo de recursos de Azure que proporcionó anteriormente. Proporcione solo el nombre del registro, no el nombre completo del servidor de inicio de sesión. Asegúrese de omitir la parte **azurecr.io** del nombre. Puede encontrar el nombre del registro en la [página Registros de contenedor](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.ContainerRegistry%2Fregistries) en Azure Portal.

**Nombre de usuario de administrador de Azure Container Registry**: proporcione el [nombre de usuario de administrador](../../container-registry/container-registry-authentication.md#admin-account) asociado a la instancia de Azure Container Registry que tiene su imagen de contenedor. El nombre de usuario y la contraseña son necesarios para asegurarse de que la empresa tiene acceso al registro. Para obtener el nombre de usuario y la contraseña de administrador, establezca la propiedad **admin-enabled** en **True** mediante la Interfaz de la línea de comandos (CLI) de Azure. Opcionalmente, puede establecer **Usuario administrador** en **Habilitar** en Azure Portal.

:::image type="content" source="media/example-iot-update-container-registry.png" alt-text="Muestra el cuadro de diálogo Actualizar registro de contenedor.":::

#### <a name="call-out-description"></a>Descripción destacada

1. Usuario administrador

<br>**Contraseña de Azure Container Registry**: proporcione la contraseña del nombre de usuario administrador asociado a la instancia de Azure Container Registry que tiene su imagen de contenedor. El nombre de usuario y la contraseña son necesarios para asegurarse de que la empresa tiene acceso al registro. Puede obtener la contraseña de Azure Portal a través de **Container Registry** > **Claves de acceso**, o bien de la CLI de Azure con el [comando show.](https://docs.microsoft.com/cli/azure/acr/credential?view=azure-cli-latest#az-acr-credential-show)

:::image type="content" source="media/example-iot-access-keys.png" alt-text="Muestra la pantalla de la clave de acceso en Azure Portal.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Claves de acceso
2. Nombre de usuario
3. Contraseña

**Nombre del repositorio en Azure Container Registry**. Proporcione el nombre del repositorio de Azure Container Registry que tiene su imagen. Especifique el nombre del repositorio cuando inserte la imagen en el registro. Para encontrar el nombre del repositorio, puede ir a [Container Registry](https://azure.microsoft.com/services/container-registry/) > **página Repositorios**. Para obtener más información, consulte [Visualización de los repositorios de registros de contenedor en Azure Portal](../../container-registry/container-registry-repositories.md). Observe que una vez establecido el nombre, no se puede cambiar. Use un nombre único para cada oferta de su cuenta.

### <a name="image-tags-for-new-versions-of-your-offer"></a>Etiquetas de imagen para las nuevas versiones de la oferta

Los clientes deben poder obtener actualizaciones automáticamente de Azure Marketplace cuando publique una actualización. Si no quieren realizar la actualización, deben poder mantener una versión específica de la imagen. Para hacerlo, puede agregar nuevas etiquetas de imagen cada vez que realice una actualización en la imagen.

**Etiqueta de imagen**. Este campo debe incluir una etiqueta **más reciente** que apunte a la versión más reciente de la imagen en todas las plataformas admitidas. Debe incluir al menos una etiqueta de versión (por ejemplo, que empiece por xx.xx.xx, donde xx es un número). Los clientes deben usar [etiquetas de manifiesto](https://github.com/estesp/manifest-tool) para dirigirse a varias plataformas. Todas las etiquetas a las que hace referencia una etiqueta de manifiesto también se deben agregar para que podamos cargarlas. Todas las etiquetas de manifiesto (excepto la etiqueta más reciente) deben comenzar por X.Y- o X.Y.Z-, donde X, Y, Z son números enteros. Por ejemplo, si una etiqueta más reciente apunta a 1.0.1-linux-x64, 1.0.1-linux-arm32 y 1.0.1-windows-arm32, estas seis etiquetas deben agregarse a este campo. Para obtener más información sobre etiquetas y control de versiones, consulte [Preparar los recursos técnicos del módulo IoT Edge.](create-iot-edge-module-asset.md)

### <a name="default-deployment-settings-optional"></a>Configuración de implementación predeterminada (opcional)

Defina la configuración más común para implementar el módulo IoT Edge. Para optimizar las implementaciones de los clientes, permítales que inicien su módulo IoT Edge directamente con esta configuración predeterminada.

**Rutas predeterminadas**. El centro de IoT Edge administra la comunicación entre los módulos, IoT Hub y los dispositivos. Puede definir rutas de entrada y salida de datos entre módulos e IoT Hub, lo que le ofrece flexibilidad para enviar mensajes donde deben ir sin necesidad de servicios adicionales para procesar los mensajes o escribir código adicional. Las rutas se construyen mediante pares de nombre-valor. Puede definir hasta cinco nombres de ruta predeterminados, cada uno de hasta 512 caracteres de longitud.

Asegúrese de usar la [sintaxis de ruta](../../iot-edge/module-composition.md#declare-routes) correcta en el valor de ruta (normalmente definida como FROM/message/* INTO $upstream). Esto significa que todos los mensajes enviados por cualquier módulo se envían a su instancia de IoT Hub. Para hacer referencia al módulo, use el nombre de módulo predeterminado, que será el **Nombre de la oferta** sin espacios ni caracteres especiales. Para hacer referencia a otros módulos aún no conocidos, use la convención <FROM_MODULE_NAME> para que los clientes sepan que deben actualizar esta información. Para obtener más información sobre las rutas de IoT Edge, consulte [Declaración de rutas](../../iot-edge/module-composition.md#declare-routes).

Por ejemplo, si el módulo ContosoModule escucha las entradas de ContosoInput y emite los datos en ContosoOutput, tiene sentido definir las dos rutas predeterminadas siguientes:

- Nombre 1: ToContosoModule
- Valor 1: FROM /messages/modules/<FROM_MODULE_NAME>/outputs/* INTO BrokeredEndpoint("/modules/ContosoModule/inputs/ContosoInput")
- Nombre 2: FromContosoModuleToCloud
- Valor 2: FROM /messages/modules/ContonsoModule/outputs/ContosoOutput INTO $upstream

**Propiedades deseadas del módulo gemelo predeterminado**. Un módulo gemelo es un documento JSON que se encuentra en IoT Hub y que almacena la información de estado de una instancia del módulo, incluidas las propiedades deseadas. Las propiedades deseadas se usan junto con las propiedades notificadas para sincronizar la configuración o las condiciones del módulo. El back-end de solución puede establecer propiedades deseadas, y el módulo puede leerlas. El módulo también puede recibir notificaciones de cambios en las propiedades deseadas. Las propiedades deseadas se crean usando hasta cinco pares nombre-valor, y cada valor predeterminado debe tener menos de 512 caracteres. Puede definir hasta cinco propiedades deseadas gemelas nombre-valor. Los valores de las propiedades deseadas gemelas deben ser un JSON válido, sin escape, sin matrices y con una jerarquía anidada máxima de cuatro niveles. En un escenario donde un parámetro necesario para un valor predeterminado no tiene sentido (por ejemplo, la dirección IP del servidor de un cliente), puede agregar un parámetro como valor predeterminado. Para más información sobre las propiedades deseadas gemelas, consulte [Definición o actualización de las propiedades deseadas](../../iot-edge/module-composition.md#define-or-update-desired-properties).

Por ejemplo, si un módulo admite una frecuencia de actualización configurable de forma dinámica mediante propiedades deseadas gemelas, tiene sentido definir la siguiente propiedad deseada gemela predeterminada:

- Nombre 1: RefreshRate
- Valor 1: 60

**Variables de entorno predeterminadas**. Las variables de entorno proporcionan información complementaria a un módulo que facilita el proceso de configuración. Las variables de entorno se crean mediante pares nombre-valor. Cada nombre y valor de la variable de entorno predeterminada debe tener menos de 512 caracteres, y puede definir hasta cinco. Si un parámetro necesario para un valor predeterminado no tiene sentido (por ejemplo, la dirección IP del servidor de un cliente), puede agregar un parámetro como valor predeterminado.

Por ejemplo, si un módulo exige aceptar los términos de uso para iniciarse, puede definir la variable de entorno siguiente:

- Nombre 1: ACCEPT_EULA
- Valor 1: Y

**Opciones de creación del contenedor predeterminadas**. Las opciones de creación del contenedor dirigen la creación del contenedor de Docker del módulo IoT Edge. IoT Edge admite las opciones de creación de contenedor de la API del motor de Docker. Consulte todas las opciones en [Lista de contenedores.](https://docs.docker.com/engine/api/v1.30/#operation/ContainerList) El campo Opciones de creación debe ser JSON válido, sin escape y de menos de 512 caracteres.

Por ejemplo, si un módulo exige el enlace de puertos, defina las opciones de creación siguientes:

"HostConfig":{"PortBindings":{"5012/tcp":[{"HostPort":"5012"}]}

## <a name="review-and-publish"></a>Revisión y publicación

Una vez que haya completado todas las secciones necesarias de la oferta, puede enviarla para su revisión y publicación.

En la esquina superior derecha del portal, seleccione **Revisar y publicar**.

En la página de revisión puede ver el estado de publicación:

- Ver el estado de finalización de cada sección de la oferta. No puede realizar la publicación hasta que todas las secciones de la oferta estén marcadas como completas.
    - **No iniciada**: significa que la sección no se ha iniciado y se debe completar.
    - **Incompleta**: significa que la sección tiene errores que deben corregirse o que requiere más información. Vea las secciones anteriores de este documento para obtener instrucciones.
    - **Completa**: la sección tiene todos los datos necesarios y no contiene errores. Para poder enviar la oferta, todas sus secciones deben estar en estado Completa.
- Proporcione instrucciones de prueba al equipo de certificación para asegurarse de que la oferta se ha probado correctamente. Además, proporcione cualquier nota complementaria que resulte útil para entender su oferta.

Para enviar la oferta para su publicación, seleccione **Publicar**.

Le enviaremos un correo electrónico para avisarle cuando haya una versión preliminar de la oferta disponible para su revisión y aprobación. Para ofrecer la oferta al público, vaya al Centro de partners y seleccione **Publicar**.

## <a name="next-steps"></a>Pasos siguientes

- [Actualización de una oferta existente en Marketplace comercial](https://docs.microsoft.com//azure/marketplace/partner-center-portal/update-existing-offer)
