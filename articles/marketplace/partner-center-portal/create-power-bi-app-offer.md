---
title: Creación de una oferta de aplicación de Power BI para Microsoft AppSource
description: Obtenga información sobre cómo crear y publicar una oferta de aplicación de Power BI para Microsoft AppSource.
author: navits09
ms.author: navits
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 07/22/2020
ms.openlocfilehash: f011f2703286cd7eb4e19588ae1ae1fe9319500f
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89646685"
---
# <a name="create-a-power-bi-app-offer"></a>Creación de una oferta de aplicación de Power BI

En este artículo se describe cómo crear y publicar una oferta de aplicación de Power BI para Microsoft[AppSource](https://appsource.microsoft.com/).

Antes de comenzar, [cree una cuenta de Marketplace comercial en el Centro de partners](create-account.md), si aún no lo ha hecho. Asegúrese de que la cuenta está inscrita en el programa Marketplace comercial.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el menú de navegación de la izquierda, seleccione **Marketplace comercial** > **Información general**.
3. En la página de información general, seleccione **+ Nueva oferta** > **Aplicación de servicio Power BI**.

   ![Muestra el menú de navegación izquierdo.](./media/new-offer-power-bi-app.png)

> [!NOTE]
> Después de publicar una oferta, las modificaciones que se realicen en el Centro de partners solo aparecen en las tiendas en línea cuando la oferta se vuelve a publicar. Asegúrese de volver a realizar la publicación siempre después de realizar cambios.

> [!IMPORTANT]
> Si no se muestra ni está habilitada la opción **Aplicación de servicio Power BI Service App**, la cuenta no tiene permiso para crear este tipo de oferta. Compruebe que cumple todos los [requisitos](create-power-bi-app-overview.md) para este tipo de oferta, incluido el registro para una cuenta de desarrollador.

## <a name="new-offer"></a>Nueva oferta

Escriba un **Identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

- Se muestra a los clientes en la dirección web de la oferta de Marketplace y en las plantillas de Azure Resource Manager, si procede.
- Use solo letras minúsculas y números. Puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres. Por ejemplo, si escribe **test-offer-1**, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
- El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

- Este nombre no se usa en Marketplace y es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.
- El alias de la oferta no se puede cambiar después de seleccionar **Crear**.

Seleccione **Crear** para generar la oferta y continuar.

## <a name="offer-overview"></a>Información general de la oferta

Esta página se muestra una representación visual de los pasos necesarios para publicar esta oferta (tanto completados como siguientes), así como el tiempo necesario que debe tardar cada uno en completarse.

Se incluyen vínculos para realizar operaciones en esta oferta en función de la selección que realice. Por ejemplo:

- Si la oferta es un borrador: [Eliminación de una oferta borrador](update-existing-offer.md#delete-a-draft-offer)
- Si la oferta está activa: [Detención de la venta de una oferta](update-existing-offer.md#stop-selling-an-offer-or-plan)
- Si la oferta está en versión preliminar: [Publicar](publishing-status.md#publisher-approval)
- Si no ha completado el cierre de sesión del anunciante: [Cancelación de la publicación](update-existing-offer.md#cancel-publishing)

## <a name="offer-setup"></a>Configuración de la oferta

### <a name="customer-leads"></a>Clientes potenciales

Al publicar la oferta en Marketplace con el Centro de partners, debe conectarla a un sistema de administración de relaciones con clientes (CRM). Esto le permite recibir información de contacto del cliente en cuanto alguien expresa interés en el producto o lo usa.

1. Seleccione un destino de clientes potenciales al que quiera que enviemos los clientes potenciales. El Centro de partners admite los siguientes sistemas CRM:

    - [Dynamics 365](commercial-marketplace-lead-management-instructions-dynamics.md) for Customer Engagement
    - [Marketo](commercial-marketplace-lead-management-instructions-marketo.md)
    - [Salesforce](commercial-marketplace-lead-management-instructions-salesforce.md)

    > [!NOTE]
    > Si su sistema CRM no aparece en la lista anterior, use [Tabla de Azure](commercial-marketplace-lead-management-instructions-azure-table.md) o [Punto de conexión HTTPS](commercial-marketplace-lead-management-instructions-https.md) para almacenar los datos del cliente potencial. A continuación, exporte los datos al sistema CRM.

2. Conecte su oferta al destino de clientes potenciales cuando la publique en el Centro de partners.
3. Compruebe que la conexión al destino de clientes potenciales está configurada correctamente. Después de publicarla en el Centro de partners, se valida la conexión y se le envía un cliente potencial de prueba. Mientras obtiene una vista previa de la oferta antes de publicarla, también puede probar la conexión de los clientes potenciales tratando de adquirir la oferta en el entorno de versión preliminar.
4. Asegúrese de que la conexión con el destino de clientes potenciales permanece actualizada para no perder ningún cliente potencial.

Estos son algunos recursos adicionales de administración de clientes potenciales:

- [Introducción a la administración de clientes potenciales](commercial-marketplace-get-customer-leads.md)
- [Preguntas frecuentes de la administración de clientes potenciales](../lead-management-for-cloud-marketplace.md#frequently-asked-questions)
- [Errores comunes de la configuración de clientes potenciales](../lead-management-for-cloud-marketplace.md#publishing-config-errors)
- [Introducción a la administración de clientes potenciales](https://assetsprod.microsoft.com/mpn/cloud-marketplace-lead-management.pdf) en formato PDF (asegúrese de que el bloqueador de elementos emergentes está desactivado).

Seleccione **Guardar borrador** antes de continuar.

## <a name="properties"></a>Propiedades

Esta página permite definir las categorías y los sectores que se usan para agrupar su oferta en Marketplace, la versión de la aplicación y los contratos legales que respaldan su oferta.

### <a name="category"></a>Category

Seleccione categorías y subcategorías para colocar la oferta en las áreas de búsqueda adecuadas del marketplace. En la descripción de la oferta, asegúrese de indicar la forma en que la oferta admite estas categorías. Seleccione:

- Al menos una y un máximo de dos categorías, incluidas una categoría principal y una secundaria (opcional).
- Hasta dos subcategorías para cada categoría principal o secundaria. Si no hay ninguna subcategoría aplicable a la oferta, seleccione **No aplicable**.

Vea la lista completa de categorías y subcategorías en [Procedimientos recomendados para la publicación de ofertas](../gtm-offer-listing-best-practices.md).

### <a name="industry"></a>Sector

[!INCLUDE [Industry Taxonomy](./includes/industry-taxonomy.md)]

### <a name="legal"></a>Información legal

#### <a name="terms-and-conditions"></a>Términos y condiciones

Si quiere proporcionar sus propios términos y condiciones personalizados, escriba hasta 10 000 caracteres en el cuadro **Términos y condiciones**. Si los términos y condiciones requieren una descripción más larga, escriba un solo vínculo web en el que se puedan encontrar. A los clientes les aparecerá como un vínculo activo.

Los clientes deben aceptar estos términos para poder probar la oferta.

Seleccione **Guardar borrador** antes de pasar a la siguiente sección, Descripción de la oferta.

## <a name="offer-listing"></a>Descripción de la oferta

Aquí puede definir los detalles de la oferta que se muestran en Marketplace. Esto incluye el nombre de la oferta, la descripción, las imágenes, etc.

### <a name="language"></a>Idioma

Seleccione el idioma en el que se mostrará la oferta. La única opción disponible actualmente es **Inglés (Estados Unidos)** .

Tendrá que definir los detalles de Marketplace (como nombre de la oferta, descripción e imágenes) para cada idioma y mercado. Seleccione el nombre de idioma o mercado para proporcionar esta información.

> [!NOTE]
> No es necesario que los detalles de la oferta estén en inglés si la descripción de la oferta comienza con la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]." También es puede proporcionar un vínculo al contenido de la oferta en un idioma distinto al utilizado en la descripción de la oferta.

A continuación se muestra un ejemplo de cómo aparece la información de la oferta en Microsoft AppSource (los precios mostrados son meramente ilustrativos y no pretenden reflejar los costos reales):

:::image type="content" source="media/example-power-bi-app.png" alt-text="Ilustración de cómo aparece esta oferta en Microsoft AppSource.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo
2. Productos
3. Categorías
4. Industrias
5. Dirección de soporte técnico (vínculo)
6. Términos de uso
7. Directiva de privacidad
8. Nombre de la oferta
9. Resumen
10. Descripción
11. Capturas de pantallas o vídeos

### <a name="name"></a>Nombre

El nombre que se escribe aquí se muestra como el título de la oferta. Este campo se rellena previamente con el texto que escribió en el cuadro **Alias de la oferta** al crear la oferta. Puede cambiar este nombre más adelante.

El nombre:

- Puede contener marcas comerciales y, además, puede incluir símbolos de marca comercial o copyright.
- No puede tener más de 50 caracteres.
- No puede incluir emojis.

### <a name="search-results-summary"></a>Resumen de los resultados de la búsqueda

Proporcione una breve descripción de la oferta. Puede tener una longitud de hasta 100 caracteres y se usa en los resultados de la búsqueda de Marketplace.

### <a name="description"></a>Descripción

[!INCLUDE [Long description-1](./includes/long-description-1.md)]

[!INCLUDE [Long description-2](./includes/long-description-2.md)]

[!INCLUDE [Long description-3](./includes/long-description-3.md)]

### <a name="search-keywords"></a>Palabras clave de búsqueda

Escriba un máximo de tres palabras clave de búsqueda opcionales para ayudar a los clientes a buscar su oferta en Marketplace. Para obtener mejores resultados, intente usar también estas palabras clave en la descripción.

### <a name="helpprivacy-web-addresses"></a>Direcciones web de ayuda/privacidad

Proporcione vínculos para ayudar a los clientes a comprender mejor su oferta.

#### <a name="help-link"></a>Vínculo de ayuda

Escriba la dirección web en la que los clientes pueden obtener más información sobre su oferta.

#### <a name="privacy-policy-url"></a>Dirección URL de la directiva de privacidad

Escriba la dirección web de la directiva de privacidad de su organización. Usted es responsable de garantizar que la oferta cumpla con las leyes y los reglamentos de privacidad. También es responsable de publicar una directiva de privacidad válida en su sitio web.

### <a name="contact-information"></a>Información de contacto

Debe proporcionar el nombre, el correo electrónico y el número de teléfono de un **contacto de soporte técnico** y un **contacto de ingeniería**. Esta información no se muestra a los clientes. Está disponible para Microsoft y se puede proporcionar a los asociados del programa Proveedor de soluciones en la nube (CSP).

- Contacto de soporte técnico (se requiere): Para preguntas generales de soporte técnico.
- Contacto de ingeniería (se requiere): Para preguntas técnicas y problemas de certificación.
- Contacto del programa CSP (opcional): Para preguntas de revendedores relacionadas con el programa CSP.

En la sección **Contacto de soporte técnico**, proporcione la dirección web del **sitio web de soporte técnico** en el que los asociados pueden encontrar soporte técnico para la oferta.

### <a name="supporting-documents"></a>Documentos relacionados

Proporcione al menos uno y hasta tres documentos de marketing relacionados en formato PDF. Por ejemplo, notas del producto, folletos, listas de comprobación o presentaciones.

### <a name="marketplace-images"></a>Imágenes de Marketplace

Proporcione logotipos e imágenes para usarlos con la oferta. Todas las imágenes deben estar en formato PNG. Las imágenes borrosas se rechazarán.

[!INCLUDE [logotips](../includes/graphics-suggestions.md)]

>[!NOTE]
>Si tiene un problema al cargar archivos, asegúrese de que la red local no bloquee el servicio `https://upload.xboxlive.com` que usa el Centro de partners.

#### <a name="store-logos"></a>Logotipos de Store

Proporcione un archivo PNG para el logotipo de tamaño **Grande**. El Centro de partners lo usará para crear un logotipo **Pequeño**. Opcionalmente, puede reemplazarlo por una imagen diferente.

- **Grande** (de 216 x 216 a 350 x 350 píxeles, obligatorio)
- **Pequeño** (48 x 48 píxeles, opcional)

Estos logotipos se usan en distintos lugares de la publicación:

[!INCLUDE [logos-appsource-only](../includes/logos-appsource-only.md)]

[!INCLUDE [logotips](../includes/graphics-suggestions.md)]

#### <a name="screenshots"></a>Capturas de pantalla

Agregue al menos una y hasta cinco capturas de pantalla que muestren el funcionamiento de la oferta. Cada una debe tener un tamaño de 1280 x 720 píxeles y el formato PNG.

#### <a name="videos-optional"></a>Vídeos (opcional)

Agregue hasta cinco vídeos que muestren la oferta. Escriba el nombre del vídeo, su dirección web y una imagen en miniatura en formato PNG del vídeo con un tamaño de 1280 x 720 píxeles.

#### <a name="additional-marketplace-listing-resources"></a>Recursos adicionales de la descripción de Marketplace

Para más información sobre la creación de listas de ofertas, vea [Procedimientos recomendados para la publicación de ofertas](../gtm-offer-listing-best-practices.md).

## <a name="technical-configuration"></a>Configuración técnica

Promocione su aplicación del servicio Power BI en producción y proporcione el vínculo del instalador de la aplicación de Power BI que permite a los clientes instalar la aplicación. Para más información, consulte [Publicación de aplicaciones con paneles e informes en Power BI](https://docs.microsoft.com/power-bi/service-create-distribute-apps).

## <a name="supplemental-content"></a>Contenido complementario

Proporcione información adicional sobre su oferta, lo que nos ayudará a validarla. Esta información no se muestra a los clientes ni se publica en Marketplace.

### <a name="validation-assets"></a>Recursos de validación

De manera opcional, agregue instrucciones (hasta 3000 caracteres) para ayudar al equipo de validación de Microsoft a configurar, conectar y probar la aplicación. Incluya opciones de configuración típicas, cuentas, parámetros u otra información que pueda usarse para probar la opción de conexión de datos. Esta información solo está visible para el equipo de validación y solo se usa para validación.

## <a name="review-and-publish"></a>Revisión y publicación

Una vez que haya completado todas las secciones necesarias de la oferta, puede enviar la oferta para su revisión y publicación.

En la esquina superior derecha del portal, seleccione **Revisar y publicar**.

En la página de revisión, puede:

- Ver el estado de finalización de cada sección de la oferta. No puede realizar la publicación hasta que todas las secciones de la oferta estén marcadas como completas.
  - **No iniciada**: significa que la sección no se ha iniciado y se debe completar.
  - **Incompleta**: significa que la sección tiene errores que deben corregirse o que requiere más información. Vea las secciones anteriores de este documento para obtener instrucciones.
  - **Completa**: la sección tiene todos los datos necesarios y no contiene errores. Para poder enviar la oferta, todas sus secciones deben estar en estado Completa.
- Proporcione instrucciones de prueba al equipo de certificación para asegurarse de que la aplicación se ha probado correctamente. Además, proporcione cualquier nota complementaria que resulte útil para entender su oferta.

Para enviar la oferta para su publicación, seleccione **Publicar**.

Le enviaremos un correo electrónico para avisarle cuando haya una versión preliminar de la oferta disponible para su revisión y aprobación. Para ofrecer la oferta al público, vaya al Centro de partners y seleccione **Publicar**.
