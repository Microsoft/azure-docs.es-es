---
title: Creación de una oferta de servicios de consultoría en marketplace comercial de Microsoft
description: En este artículo se explica cómo publicar una oferta de servicios de consultoría en Microsoft AppSource o en Azure Marketplace mediante el Centro de partners.
author: Microsoft-BradleyWright
ms.author: brwrigh
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: 03fa302d2e8aa173419912b99d60734c7a4e0356
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88814271"
---
# <a name="create-a-consulting-service-offer"></a>Creación de una oferta de servicios de consultoría

En este artículo se explica cómo publicar una oferta de servicios de consultoría en [Microsoft AppSource](https://appsource.microsoft.com/) o en [Azure Marketplace](https://azuremarketplace.microsoft.com/). Lista de ofertas de servicios de consultoría basadas en Microsoft [Dynamics 365](https://dynamics.microsoft.com/) y Power Platform en AppSource. Lista de ofertas de servicios de consultoría basadas en Microsoft Azure en Azure Marketplace. Antes de comenzar, [cree una cuenta de marketplace comercial en el Centro de partners](create-account.md), si aún no lo ha hecho. Asegúrese de que la cuenta está inscrita en el programa marketplace comercial.

Antes de crear la oferta, revise los requisitos previos en [Requisitos previos de los servicios de consultoría](consulting-service-prerequisites.md).

## <a name="publishing-benefits"></a>Ventajas de publicación

Ventajas de publicar en Marketplace comercial:

- Promocionar la empresa con la marca de Microsoft.
- Llegar potencialmente a más de 100 millones de usuarios de Office 365 y Dynamics 365 en AppSource y a más de 200 000 organizaciones en Azure Marketplace.
- Recibir clientes potenciales de alta calidad de estos mercados.
- Promocionar sus servicios mediante los equipos de campo y televenta de Microsoft.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Información general**.
3. En la página Información general, seleccione **+ Nueva oferta** > **Servicios de consultoría**.

    ![Muestra el menú de navegación izquierdo.](./media/new-offer-consulting-service.png)

>[!NOTE]
>Después de publicar una oferta, las modificaciones que se realicen en el Centro de partners solo aparecen en las tiendas en línea cuando la oferta se vuelve a publicar. Asegúrese de volver a realizar la publicación siempre después de realizar cambios.

## <a name="new-offer"></a>Nueva oferta

Escriba un **Identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

- Este identificador es visible para los clientes en la dirección web de la oferta de Marketplace.
- Use solo letras minúsculas y números. Puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres. Por ejemplo, si escribe **test-offer-1**, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
- El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

- Este nombre no se usa en Marketplace y es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.
- El alias de la oferta no se puede cambiar después de seleccionar **Crear**.

Seleccione **Crear** para generar la oferta y continuar.

## <a name="offer-setup"></a>Configuración de la oferta

### <a name="connect-lead-management"></a>Conexión de administración de clientes potenciales

Al publicar la oferta en Marketplace con el Centro de partners, _debe_ conectarla a un sistema de administración de relaciones con clientes (CRM) o de automatización de marketing. Esto le permite recibir información de contacto del cliente en cuanto alguien expresa interés en el producto o lo usa.

1. Seleccione **Conectar** para especificar dónde quiere que se envíen los clientes potenciales. El Centro de partners admite los siguientes sistemas:

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
- [Introducción a la administración de clientes potenciales](https://assetsprod.microsoft.com/mpn/cloud-marketplace-lead-management.pdf) en formato PDF (asegúrese de que el bloqueador de elementos emergentes está desactivado)

Seleccione **Guardar borrador** antes de continuar.

### <a name="properties"></a>Propiedades

Esta página le permite establecer el producto principal más relacionado con el servicio de consultoría, establecer un tipo de servicio de consultoría y elegir productos aplicables.

1. Seleccione un **Producto principal** en la lista desplegable.
2. Seleccione un **Tipo de servicio de consultoría** en la lista desplegable:

    - **Evaluación:** una evaluación del entorno del cliente para determinar la aplicabilidad de una solución y proporcionar una estimación de costo y tiempo.
    - **Sesión informativa**: introducción a una solución o un servicio de consultoría para determinar el interés del cliente mediante plataformas, demostraciones y ejemplos de clientes.
    - **Implementación**: una instalación completa que da como resultado una solución que funciona completamente. Limítese a las soluciones que se pueden implementar en dos semanas o menos.
    - **Prueba de concepto**: una implementación de ámbito limitado para determinar si una solución cumple los requisitos del cliente.
    - **Taller**: una involucración interactiva que se lleva a cabo en las instalaciones del cliente. Puede implicar entrenamiento, sesiones informativas, evaluaciones o demostraciones que se crean en los datos o el entorno del cliente.

3. Si ha seleccionado **Azure** como producto principal, seleccione hasta tres **Áreas de solución**. Esto hace que sea más fácil para los clientes de Azure Marketplace encontrar la oferta. Si no ha seleccionado Azure, omita este paso.

    - Análisis
    - Modernización de aplicaciones
    - Archivar
    - AI + Aprendizaje automático
    - Copia de seguridad
    - Datos de gran tamaño
    - Plataforma de datos
    - Administración de centro de datos
    - DevOps
    - Recuperación ante desastres
    - Identidad
    - Internet de las cosas
    - Migración
    - Redes
    - Seguridad
    - Storage

1. Si ha seleccionado **Azure** como producto principal, tiene la opción de seleccionar hasta seis **sectores**. Esto hace que sea más fácil para los clientes de Azure Marketplace encontrar la oferta. Vea la lista completa de sectores en [Procedimientos recomendados para la publicación de ofertas](../gtm-offer-listing-best-practices.md). Si no ha seleccionado Azure, omita este paso.
1. Si ha seleccionado un producto principal *distinto* de Azure, seleccione hasta tres **Productos aplicables**. Esto hace que sea más fácil para los clientes de AppSource encontrar la oferta. Para más información, consulte [Instrucciones sobre listas de servicios de consultoría de Microsoft AppSource](https://go.microsoft.com/fwlink/?LinkId=828734&amp;clcid=0x409) (PDF).
1. Si ha seleccionado un producto principal *distinto* de Azure, tiene la opción de seleccionar hasta dos **sectores** y dos **segmentos de mercado verticales** para cada sector. Esto hace que sea más fácil para los clientes de AppSource encontrar la oferta. Vea la lista completa de sectores y segmentos de mercado verticales en [Procedimientos recomendados para la publicación de ofertas](../gtm-offer-listing-best-practices.md).
1. Agregue hasta tres **Competencias** que su empresa haya conseguido para mostrar en la lista de ofertas de servicios de consultoría. Se requiere al menos una competencia, excepto en el caso de MSP experto de Azure y MSP de Redes de Azure.

Seleccione **Guardar borrador** antes de continuar.

## <a name="offer-listing"></a>Descripción de la oferta

Aquí puede definir los detalles de la oferta que se muestran en Marketplace. Esto incluye el nombre de la oferta, la descripción, las imágenes, etc. Asegúrese de seguir las directivas que se detallan en la [página de directivas de certificación comercial](https://docs.microsoft.com/legal/marketplace/certification-policies#800-consulting-services) al configurar esta oferta.

> [!NOTE]
> No es necesario que los detalles de la oferta estén en inglés si la descripción de la oferta comienza con la frase &quot;Esta aplicación solo está disponible en [idioma distinto del inglés].&quot; También es puede proporcionar un vínculo al contenido de la oferta en un idioma distinto al utilizado en la descripción de la oferta.

A continuación se muestra un ejemplo de cómo aparece la información en Azure Marketplace (los precios mostrados son meramente ilustrativos y no pretenden reflejar los costos reales):

:::image type="content" source="media/example-consulting-service-offer.png" alt-text="Ilustración de cómo aparece esta oferta en Azure Marketplace.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo
2. Price
3. Áreas de solución
4. Industrias
5. Nombre de la oferta
6. Resumen
7. Descripción
8. Capturas de pantallas o vídeos

<br>A continuación se muestra un ejemplo de cómo aparece la información de la oferta en Microsoft AppSource (los precios mostrados son meramente ilustrativos y no pretenden reflejar los costos reales):

:::image type="content" source="media/example-consulting-service-offer-appsource.png" alt-text="Ilustración de cómo aparece esta oferta en Microsoft AppSource.":::

#### <a name="call-out-descriptions"></a>Descripciones destacadas

1. Logotipo
2. Price
3. Productos
4. Industrias
5. Nombre de la oferta
6. Resumen
7. Descripción
8. Capturas de pantallas o vídeos
9. Documentos

### <a name="name"></a>Nombre

El nombre que escriba aquí se muestra como el título de la oferta. Este campo se rellena previamente con el texto que escribió en el cuadro de texto **Alias de la oferta** al crear la oferta. Puede cambiar este nombre más adelante.

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

### <a name="keywords"></a>Palabras clave

Escriba hasta tres palabras clave de búsqueda adecuadas para el producto principal y el servicio de consultoría. Esto facilitará la búsqueda de la oferta.

### <a name="duration"></a>Duration

Establezca la duración esperada de este compromiso con el cliente.

### <a name="contact-information"></a>Información de contacto

Debe proporcionar el nombre, correo electrónico y número de teléfono de un **Contacto principal** y un **Contacto secundario**. Esta información no se muestra a los clientes. Está disponible para Microsoft y se puede proporcionar a los asociados del programa Proveedor de soluciones en la nube (CSP).

### <a name="supporting-documents"></a>Documentos relacionados

Agregue hasta tres (pero al menos uno) documentos PDF para la oferta.

### <a name="marketplace-images"></a>Imágenes de Marketplace

Proporcione logotipos e imágenes para usarlos con la oferta. Todas las imágenes deben estar en formato .PNG. Las imágenes borrosas se rechazarán.

[!INCLUDE [logotips](../includes/graphics-suggestions.md)]

>[!Note]
>Si tiene un problema al cargar archivos, asegúrese de que la red local no bloquee el servicio https://upload.xboxlive.com que usa el Centro de partners.

#### <a name="store-logos"></a>Logotipos de Store

Proporcione un archivo PNG para el logotipo de tamaño **Grande**. El Centro de partners lo usará para crear un logotipo **Pequeño**. Opcionalmente, puede reemplazarlo por una imagen diferente.

- **Grande** (de 216 x 216 a 350 x 350 píxeles, obligatorio)
- **Pequeño** (48 x 48 píxeles, opcional)

Estos logotipos se usan en distintos lugares de la publicación.

[!INCLUDE [Logo tips](../includes/graphics-suggestions.md)]

#### <a name="screenshots-optional"></a>Capturas de pantalla (opcional)

Agregue hasta cinco capturas de pantalla que muestren el funcionamiento de la oferta. Cada una debe tener un tamaño de 1280 x 720 píxeles y el formato PNG.

#### <a name="videos-optional"></a>Vídeos (opcional)

Agregue hasta cuatro vídeos que muestren la oferta. Escriba el nombre del vídeo, su dirección web (URL) y una imagen en miniatura en formato PNG del vídeo con un tamaño de 1280 x 720 píxeles.

Seleccione **Guardar borrador** antes de continuar.

## <a name="pricing-and-availability"></a>Precios y disponibilidad

Aquí podrá definir elementos como los precios, el mercado y una clave privada.

1. **Mercado**: establezca el mercado en el que estará disponible la oferta. Solo puede seleccionar un mercado por oferta.
    1. En el caso de los mercados de Estados Unidos o Canadá, seleccione **Editar Estados** (o **provincias**) para especificar dónde estará disponible la oferta.
2. **Público preliminar**: configure la **Clave oculta** que se usa para establecer la audiencia privada de la oferta.
3. **Precios**: Especifique si la oferta es **Gratuita** o **De pago**.

    > [!NOTE]
    > Las ofertas de servicios de consultoría son solo para la lista. Cualquier transacción se realizará directamente, fuera de Marketplace comercial.

4. Para una oferta de pago, especifique el **Precio y moneda** y si el precio es **Fijo** o **Estimado**. Si es estimado, debe especificar en la descripción los factores que afectan al precio.
5. Seleccione **Guardar borrador** antes de continuar.

## <a name="review-and-publish"></a>Revisión y publicación

Una vez que haya completado todas las secciones necesarias de la oferta, puede enviar la oferta para su revisión y publicación.

1. Cuando esté listo para publicar la oferta del servicio de consultoría, haga clic en **Revisar y publicar**.
2. Revise los detalles de la página de envío final.
3. Si es necesario, escriba una nota al equipo de certificación si cree que alguno de los detalles de la oferta requiere explicación.
4. Cuando esté listo, seleccione **Enviar**.
5. En la página **Información general de la oferta** se muestra la fase de publicación en la que se encuentra la oferta.

Para más información sobre el tiempo que puede esperar que esté la oferta en cada fase de publicación, consulte [Comprobación del estado de publicación de la oferta de Marketplace comercial](publishing-status.md).

## <a name="update-your-existing-consulting-service-offers"></a>Actualización de las ofertas de servicios de consultoría existentes

- [Actualización de una oferta existente en Marketplace comercial](update-existing-offer.md)
