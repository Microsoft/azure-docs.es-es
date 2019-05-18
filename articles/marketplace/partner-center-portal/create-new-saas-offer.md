---
title: Cree una nueva oferta de SaaS en el mercado comercial
description: Cómo crear un nuevo Software como una oferta de servicio (SaaS) para enumerar o vender en Azure Marketplace, AppSource, o a través del programa proveedor de soluciones en la nube (CSP) mediante el portal de catálogo de soluciones comerciales en Microsoft Partner Center.
author: mattwojo
manager: evansma
ms.author: mattwoj
ms.service: marketplace
ms.topic: conceptual
ms.date: 05/30/2019
ms.openlocfilehash: 32537426b7b9b1a7015610fc0c3e2dd7c3efa49b
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65806212"
---
# <a name="create-a-new-saas-offer"></a>Cree una nueva oferta de SaaS

Para empezar a crear Software como servicio (SaaS) ofrece, asegúrese de que primero [crear una cuenta de centro de partners](./create-account.md) y abra el [panel Marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace/offers), con el **ofrece** pestaña seleccionada. 

![Panel comercial de Marketplace en el centro de partners](./media/commercial-marketplace-offers.png)

Seleccione el + **crear un nuevo...** botón y luego seleccione el **Software como servicio** elemento de menú. 

Si selecciona uno de los otros tipos de oferta, se le redirigirá a la versión anterior [Cloud Partner Portal](https://cloudpartner.azure.com/).  Ofertas de SaaS sola están disponibles en el portal de Marketplace comercial en el centro de partners en este momento. 

![Crear ventana de la oferta en el centro de partners](./media/new-offer.png)


El **nueva oferta** se muestra el cuadro de diálogo. ![Cuadro de diálogo nueva oferta](./media/new-offer-popup.png)


## <a name="offer-id-and-name"></a>Identificador de oferta y el nombre

- **Id. de oferta**: Crear un identificador único para cada oferta en su cuenta. Este identificador será visible para los clientes en la dirección URL de la oferta de marketplace y plantillas de Azure Resource Manager (si procede). Identificador de oferta debe ser alfanumérico (incluidos los guiones y caracteres de subrayado, pero sin espacios en blanco), en minúsculas. Esto se limita a 50 caracteres y no se puede actualizar después de seleccionar crear.  
Ejemplo: 1 de oferta de prueba
<br>Lo que resulta en la dirección URL: `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`

- **Nombre de la oferta**: El nombre oficial de la oferta de aplicación SaaS, coherente en las publicaciones, anuncios y sitios web.  Puede ser marcas comerciales de este nombre.  Nombre no debe contener espacios en blanco, emojis (a menos que sean el símbolo de marca comercial o copyright) de la oferta y debe estar limitada a 50 caracteres.
<br>Ejemplo: Oferta de prueba 1&#8482;

Seleccione **Crear**.  Un **ofrecen información general sobre** página se crea para esta oferta.  

![Información general de la oferta en el centro de partners](./media/commercial-marketplace-offer-overview.png)

## <a name="offer-overview"></a>Información general de la oferta

El **ofrecen información general sobre** página incluye: 

- El **estado de publicación** muestra una representación visual de los pasos necesarios para publicar esta oferta y el tiempo se tardará en completarse cada paso. Iconos de paso de publicación incompletas se atenúa. 

- El **ofrecen información general sobre** menú contiene una lista de vínculos para realizar operaciones en esta oferta. Esta lista de operaciones cambiará según la selección que realice para su oferta.  
    - Si la oferta es un borrador: Borrador Delete 
    - Si la oferta está activa: Stop vender la oferta 
    - Si la oferta está en vista previa: puesta en marcha 
    - Si no ha completado el inicio de sesión del publicador: out Cancelar publicación

## <a name="offer-setup"></a>Programa de instalación de la oferta

El **el programa de instalación de la oferta** ficha pide la información siguiente. Seleccione **guardar** después de completar estos campos.

- **¿Desea vender a través de Microsoft?** (Sí/No)
    - **Sí**, que le gustaría vender su oferta a través de Microsoft con Microsoft aloja transacciones de marketplace en su nombre; o 
    - **No**mostrar solo su oferta a través de los catálogos de soluciones, sería preferible procesar las transacciones monetarias independientemente de Microsoft.    

### <a name="sell-through-microsoft"></a>Venta mediante Microsoft

Venta a través de Microsoft proporciona una mejor detección de cliente y adquisición, permite a Microsoft a las transacciones de marketplace de host en su nombre y aprovecha las capacidades de Microsoft commerce disponible globalmente.

#### <a name="saas-offer-requirements"></a>Requisitos de la oferta de SaaS

Para enumerar el Software como servicio (SaaS) se ofrece con el catálogo de soluciones comerciales en el centro de partners, deben cumplirse los siguientes criterios:

- La oferta debe ser compatible con clientes de Azure. (A menudo las aplicaciones de SaaS también están hospedadas en Azure para optimizar el rendimiento y compatibilidad, pero esto no es un requisito.) 
- Debe usar la oferta [Azure Active Directory (Azure AD)](https://azure.microsoft.com/services/active-directory/) para la autenticación y administración de identidades.
- Debe usar la oferta [API de entrega SaaS](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/saas-app/cpp-saas-fulfillment-api-v2) para integrarse con Azure Marketplace.

#### <a name="billing-infrastructure-costs"></a>Costos de infraestructura de facturación
Para las ofertas de SaaS, como el publicador, debe tener en cuenta para las cuotas de uso de la infraestructura de Azure y las cuotas de licencias de software como un elemento único costo. Este costo se representa como una tarifa plana mensual para el cliente. Uso de la infraestructura de Azure administra y factura directamente a usted, el socio comercial. El cliente no ve los honorarios de uso de infraestructura reales. En general, los publicadores optan por agrupar los honorarios de uso de la infraestructura de Azure en los precios de las licencias de software. 

Tarifas de licencias que se presentan como una cuota de suscripción mensual, periódicos basados en sitio global y no se mide el uso de software o basado en consumo.

|**Costo de licencia**|**100 USD al mes**|
|:---|:---|
|Costo de uso de Azure (D1/1 núcleo)|Facturado directamente al publicador, no al cliente|
|Se factura al cliente por Microsoft|100,00 USD al mes (publicador debe tener en cuenta los costos de infraestructura que se incurra o paso a través en la cuota de licencia)|

- En este escenario, Microsoft factura 100,00 USD por su licencia de software y paga 80,00 USD al publicador.
- Los asociados que se han calificado para el **reduce tarifa del servicio Marketplace** verán una tarifa reducida de transacciones en el SaaS ofrece desde mayo de 2019 hasta de 2020 de junio. En este escenario, Microsoft factura 100,00 USD por su licencia de software y su recompensa 90,00 USD al publicador.

> [!NOTE]
> **Reduce la tarifa del servicio Marketplace**: Para determinados SaaS ofrece que ha publicado en nuestro Marketplace comercial, Microsoft reducirá la tarifa del servicio Marketplace del 20% (como se describe en el contrato del anunciante de Microsoft) en un 10%. En el orden de la oferta calificar, al menos una de las ofertas debe haber designado por Microsoft como lista de venta conjunta de IP o priorizado de venta conjunta de IP.  Elegibilidad se debe cumplir al menos cinco (5) días hábiles antes del final de cada mes natural para recibir esta tarifa reducida del servicio Marketplace del mes.  La tarifa del servicio Marketplace reduce no es aplicable a las máquinas virtuales, las aplicaciones administradas o cualquier otro producto están disponible a través de nuestro Marketplace comercial.  La tarifa del servicio Marketplace reduce solo estará disponible a las ofertas completas para los cargos de licencia recopilados por Microsoft entre el 31 de mayo de 2019 y el 30 de junio de 2020.  Transcurrido ese tiempo, la tarifa del servicio Marketplace volverá a su importe normal. 

|**Microsoft factura**|**100 USD al mes**|
|:---|:---|
|Microsoft le paga el 80 % del costo de licencia <br>**Para las aplicaciones completas de SaaS, Microsoft paga el 90% del costo de licencia*|80,00 USD al mes <br>*$* 90,00 por mes *|


#### <a name="csp-program-opt-in"></a>CSP participación en el programa
El [proveedor de soluciones en la nube (CSP)](https://docs.microsoft.com/azure/marketplace/cloud-solution-providers) programa permite a las ofertas de software llegar a millones de clientes de Microsoft calificados con el mínimo de inversión de marketing y venta.

- **Canales: Poner mi oferta a disposición en el programa CSP** (casilla)

Elegir poner su oferta a disposición en el programa CSP permite a Cloud Solution Providers vender su producto como parte de una solución preconfigurada a sus clientes. 

### <a name="list-through-microsoft"></a>Lista a través de Microsoft

Promover su negocio con Microsoft mediante la creación de una lista de marketplace. Seleccionar la opción para publicar su oferta solo y no a través de Microsoft significa que Microsoft no participa directamente en las transacciones de licencia de software. No hay ningún cargo de transacción asociada y el publicador mantiene 100% de las cuotas recopiladas del cliente de licencias de software. Sin embargo, el publicador es responsable del soporte técnico de todos los aspectos de la transacción de la licencia de software, incluyendo pero sin limitarse a: orden de realización de medición, facturación, la facturación, pago y colección. 

- **¿Cómo desea que los clientes potenciales para interactuar con esta oferta de la lista?**

##### <a name="get-it-now-free"></a>Obténgalo ahora (gratis)
Lista de forma gratuita la oferta a los clientes al proporcionar una dirección URL válida (comienza con http o https) donde puede acceder a la aplicación.  Por ejemplo: `https://contoso.com/saas-app`

##### <a name="free-trial"></a>Evaluación gratuita
Lista de la oferta a los clientes de prueba gratuita al proporcionar una dirección URL válida (comienza con http o https) donde puede acceder a la aplicación.  Por ejemplo: `https://contoso.com/trial/saas-app`

##### <a name="contact-me"></a>Ponerse en contacto conmigo
Recopilar información de contacto de cliente mediante la conexión de su sistema Customer Relationship Management (CRM). El cliente le pedirá permiso compartir su información. Estos detalles del cliente, junto con el nombre de la oferta, el identificador y el origen de marketplace donde encuentra la oferta, se enviará al sistema CRM que haya configurado. Para obtener más información acerca de cómo configurar su CRM, consulte [administración de clientes potenciales de Connect](#connect-lead-management). 

## <a name="enable-a-test-drive"></a>Habilitación de una versión de prueba

Una versión de prueba es una excelente manera de presentar la oferta a los clientes potenciales por lo que ofrece la opción "probar antes de comprar", dando como resultado un aumento de conversión y la generación de clientes potenciales altamente cualificados. [Más información sobre las versiones de prueba.](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/test-drive/what-is-test-drive)

- **Habilitar una versión de prueba** (casilla) 

Al habilitar la versión de prueba, se le pedirá para configurar un entorno de demostración para los clientes probar la oferta durante un período de tiempo fijo. 

### <a name="type-of-test-drive"></a>Tipo de versión de prueba

- **[Azure Resource Manager](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/test-drive/azure-resource-manager-test-drive)**: Una plantilla de implementación que contiene todos los recursos de Azure que componen la solución. Los productos que se ajusten a este escenario usar solo los recursos de Azure.
- **[Dynamics 365 para el centro de negocios](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal-orig/cpp-business-central-offer)**: Microsoft hospeda y mantiene el servicio de la unidad de prueba (incluido el aprovisionamiento y la implementación) de un sistema de planeamiento de recursos empresariales Business Central (Finanzas, operaciones, cadena de procesos, CRM, etcetera.).  
- **[Dynamics 365 for Customer Engagement](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/dyn365ce/cpp-customer-engagement-offer)**: Microsoft hospeda y mantiene el servicio de la unidad de prueba (incluido el aprovisionamiento y la implementación) para un sistema de Customer Engagement (ventas, servicio, servicio de proyecto, el servicio de campo, etcetera).  
- **[Dynamics 365 para las operaciones](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal-orig/cpp-dynamics-365-operations-offer)**: Microsoft hospeda y mantiene el servicio de la unidad de prueba (incluido el aprovisionamiento y la implementación) para un Finanzas y operaciones de planeamiento de recursos empresariales del sistema (Finanzas, operaciones, fabricación, cadena de suministro, etcetera). 
- **[Aplicación lógica](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/test-drive/logic-app-test-drive)**: Una plantilla de implementación que abarca todas las arquitecturas de soluciones complejas. Los productos personalizados deben usar este tipo de versión de prueba.
- **[Power BI](https://docs.microsoft.com/power-bi/service-template-apps-overview)**: Un vínculo incrustado a un panel personalizado. Productos que desean demostrar que un visual interactivos de Power BI debe usar este tipo de versión de prueba. Todo lo que necesita para la carga aquí es la dirección URL insertada de Power BI.

#### <a name="additional-test-drive-resources"></a>Recursos de la unidad de prueba adicionales
- [Prueba unidad mejores prácticas técnicas](https://github.com/Azure/AzureTestDrive/wiki/Test-Drive-Best-Practices)
- [Pruebe los procedimientos recomendados de Marketing](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/test-drive/marketing-and-best-practices)
- [Pruebas de unidad de introducción a un buscapersonas](https://assetsprod.microsoft.com/mpn/azure-marketplace-appsource-test-drives.pdf)

## <a name="connect-lead-management"></a>Conexión de administración de clientes potenciales

Conectarse con los clientes directamente mediante anunciar la oferta en los catálogos de soluciones y enlazar su sistema Customer Relationship Management (CRM) para que pueda recibir información de contacto de cliente inmediatamente después de un cliente exprese interés o implemente su producto.

- **Elegir un destino de clientes potenciales** (menú desplegable): Proporcione detalles de conexión con el sistema CRM donde desea que te enviemos clientes potenciales. 

Centro de partners es compatible con los siguientes sistemas CRM para la administración de clientes potenciales. Seleccione el vínculo para obtener instrucciones de instalación.

- Azure Blob: proporciona correo electrónico de contacto, el nombre del contenedor y la cadena de conexión de cuenta de almacenamiento. 
- [Tabla de Azure](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal-orig/cloud-partner-portal-lead-management-instructions-azure-table) : proporcionar correo electrónico de contacto y la cadena de conexión de cuenta de almacenamiento. 
- [Dynamics CRM Online](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal-orig/cloud-partner-portal-lead-management-instructions-dynamics) : proporcionar correo electrónico de contacto, dirección URL y el modo de autenticación (Office 365 o Azure Active Directory).
- [Punto de conexión HTTPS](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal-orig/cloud-partner-portal-lead-management-instructions-https) : proporcionar correo electrónico de contacto y la dirección URL del punto de conexión HTTPS. 
- [Marketo](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal-orig/cloud-partner-portal-lead-management-instructions-marketo) : proporcionar correo electrónico de contacto, el identificador de formulario, Id. de cuenta de Munchkin e identificador del servidor.
- [Salesforce](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal-orig/cloud-partner-portal-lead-management-instructions-salesforce) -proporcionar correo electrónico de contacto y el identificador de organización. 

#### <a name="additional-lead-management-resources"></a>Recursos de administración de clientes potenciales adicionales
- [Preguntas más frecuentes sobre administración de clientes potenciales](https://docs.microsoft.com/azure/marketplace/lead-management-for-cloud-marketplace#frequently-asked-questions)
- [Errores comunes de configuración de clientes potenciales](https://docs.microsoft.com/azure/marketplace/lead-management-for-cloud-marketplace#common-lead-configuration-errors-during-publishing-on-cloud-partner-portal)
- [Provocar administración información general sobre un buscapersonas](https://assetsprod.microsoft.com/mpn/cloud-marketplace-lead-management.pdf)

No olvide **guardar** antes de continuar con la siguiente sección.

## <a name="properties"></a>Properties (Propiedades)
El **propiedades** ficha le solicita que defina las categorías y los sectores que se usa para agrupar la oferta en los catálogos de soluciones, los contratos legales que admite su oferta y la versión de la aplicación. 

Seleccione **guardar** después de completar estos campos. 

### <a name="category"></a>Categoría
Seleccione un mínimo de uno (1) y un máximo de tres (3) categorías utilizadas para agrupar la oferta en las áreas de la búsqueda de marketplace adecuado. Destacar cómo la oferta es compatible con estas categorías en la descripción de la oferta. 

### <a name="industry"></a>Sector
Seleccione sectores hasta dos (2) que se usa para agrupar la oferta en las áreas de la búsqueda de marketplace adecuado. Si la oferta no es un sector específica, no seleccione uno. Destacar cómo la oferta es compatible con las industrias seleccionadas en la descripción de la oferta. 

### <a name="app-version"></a>Versión de la aplicación
Se trata de un campo opcional que se utiliza en el marketplace de AppSource para identificar el número de versión de la oferta. 

### <a name="standard-contract"></a>Contrato estándar
- **¿Usar el contrato estándar?** 

Para simplificar el proceso de adquisición para los clientes y reducir la complejidad legal para proveedores de software, Microsoft ofrece una plantilla de contrato estándar con el fin de ayudar a facilitar una transacción en marketplace. 

En lugar de creación de términos y condiciones personalizados, los editores de Azure Marketplace puede ofrecer su software en el contrato estándar, que los clientes solo tienen que investigar y Aceptar una vez. 

El contrato estándar puede encontrarse aquí: https://go.microsoft.com/fwlink/?linkid=2041178.

##### <a name="terms-of-use"></a>Términos de uso
Si los términos de licencia son diferentes del contrato estándar, puede optar por escribir sus propios términos legales de uso aquí. También puede escribir hasta 10 000 caracteres de texto en este campo. Si los términos de uso requieren una descripción más larga, escriba un único vínculo de dirección URL en este campo donde se pueden encontrar los términos de licencia adicionales. Mostrará a los clientes como un vínculo activo.

Los clientes deben aceptar estos términos antes de poder probar la aplicación. 

No olvide **guardar** antes de continuar con la siguiente sección.

## <a name="offer-listing"></a>Anuncio de la oferta
La descripción pestaña muestra los lenguajes (y los mercados) donde está disponible la oferta de la oferta, actualmente el inglés (Estados Unidos) es la única ubicación disponible. Además, esta página muestra el estado de la lista específica del lenguaje y la fecha y hora que se agregó. Deberá definir los detalles de marketplace (ofrecen el nombre, descripción, los términos de búsqueda, etc.) para cada idioma o el mercado.

### <a name="offer-listings"></a>Descripción de la oferta
Proporcionar detalles que se mostrará en el mercado, incluidas las descripciones de la oferta y activos de marketing.

- **Nombre** (obligatorio): El nombre definido aquí aparecerá como título de la oferta de lista en el marketplace(s) que ha elegido. El nombre se rellena previamente basándose en la anterior **nueva oferta** entrada.  Esto puede ser marcas comerciales.  Esto no debe contener espacios en blanco, emojis (a menos que sean los símbolos de copyright y marca comercial) y debe estar limitada a 50 caracteres.
- **Resumen de** (obligatorio): Proporcione una descripción breve de la oferta que se usará en los resultados de búsqueda de anuncios de marketplace. Pueden escribir hasta 100 caracteres de texto en este campo.
- **Descripción** (obligatorio): Proporcione una descripción de la oferta que se mostrará en la información general de anuncios de marketplace. Considere incluir una propuesta de valor, los beneficios clave, las asociaciones de categoría o sector, las oportunidades de compra en la aplicación, cualquier necesaria divulgaciones y un vínculo para obtener más información.
En este campo se pueden introducir hasta 3000 caracteres de texto. Para obtener sugerencias adicionales, consulte el artículo [escribir una descripción excelente aplicación](https://docs.microsoft.com/windows/uwp/publish/write-a-great-app-description).
- **Buscar palabras clave**: Escriba hasta tres palabras clave de búsqueda que los clientes pueden usar para buscar su oferta en el marketplace(s).
- **Introducción a las instrucciones** (obligatorio): Se explica cómo configurar y empezar a usar la aplicación para los clientes potenciales.  En este tutorial rápido puede contener vínculos a documentación en línea más detallada. En este campo se pueden introducir hasta 3000 caracteres de texto. 

#### <a name="links"></a>Vínculos

- **Directiva de privacidad** (obligatorio): Vincular a la directiva de privacidad de su organización. Es responsable de garantizar que la aplicación cumple con la normativa y la legislación de privacidad y para proporcionar una directiva de privacidad válido
- **Materiales de Marketing de programa CSP** (opcional): Debe proporcionar un vínculo a materiales de marketing si opta por ampliar su oferta a la [proveedor de soluciones en la nube (CSP)](https://docs.microsoft.com/azure/marketplace/cloud-solution-providers) programa. CSP amplía su oferta a una gama más amplia de clientes completos al permitir que los asociados CSP agrupar, mercado, y comercializar su oferta. Estos distribuidores deben tener acceso a materiales de marketing de su oferta. Para obtener más información, consulte [Go-To-Market Services](https://partner.microsoft.com/reach-customers/gtm).
- **Vínculos útiles** (opcional): Opcionales documentos en línea complementarios acerca de la aplicación o servicios relacionados enumerados proporcionando un **título** y **URL**. Agregar vínculos útiles adicionales haciendo clic en **+ agregar una dirección URL**.

#### <a name="contact-information"></a>Información de contacto

- **Contactos**: Para cada contacto de cliente proporcionan un empleado **nombre** , **número de teléfono**, y **correo electrónico** dirección.  (Estos *no* mostrarse públicamente). Un **dirección URL de soporte** también es necesaria para la **contacto de soporte técnico** grupo.  (Esta información *le* mostrarse públicamente).

**Contacto de soporte técnico** (obligatorio): Para cuestiones generales de soporte técnico.

**Contacto de ingeniería** (obligatorio): Para preguntas técnicas.

**Canal de contact Manager** (obligatorio): Para el distribuidor preguntas relacionadas con el programa CSP.

#### <a name="files-and-images"></a>Archivos e imágenes

- **Documentos** (obligatorio): Agregar documentos relacionados de marketing de su oferta, en formato PDF, que proporciona un mínimo de uno (1) y máximo de tres (3) documentos por oferta.
- **Imágenes** (opcional): Existen varios lugares donde puedan aparecer imágenes de logotipo de su oferta en toda la marketplace(s), que requieren los siguientes tamaños: pequeño: 48 x 48 píxeles _(obligatorio),_ medio: 90 x 90 píxeles, grande: píxeles de 216 x 216 _(obligatorio),_ amplia: 255 x 115 píxeles y Hero: 815 x 290 píxeles. Todas las imágenes deben pertenecer. Formato PNG.
- **Capturas de pantalla** (obligatorio): Agregar las capturas de pantalla que muestra la oferta. Un máximo de cinco (5) las capturas de pantalla se puede agregar y debe ser un tamaño de 1280 x 720 píxeles. Todas las imágenes deben pertenecer. Formato PNG.
- **Vídeos** (opcional): Agregar vínculos a vídeos de demostración de la oferta. Puede usar vínculos a vídeos de YouTube o Vimeo para que acompañen a la oferta para los clientes. También deberá especificar una imagen en miniatura del vídeo, tamaño de 1280 x 720 píxeles en formato PNG. Puede mostrar un máximo de cuatro vídeos por oferta.

No olvide **guardar** antes de continuar con la siguiente sección.

#### <a name="additional-marketplace-listing-resources"></a>Recursos de la lista de marketplace adicionales

- [Procedimientos recomendados para marketplace ofrecen programas](https://docs.microsoft.com/azure/marketplace/gtm-offer-listing-best-practices)



## <a name="preview"></a>Preview

El **Preview** ficha permite definir un limitado **Preview audiencia** para liberar su oferta antes de publicar su oferta en directo a la audiencia más amplia de marketplace.

> [!IMPORTANT]
> Debe seleccionar **lanzarla** antes de su oferta se publicará en vivo a la audiencia marketplace público después de comprobar su oferta en versión preliminar.

- **Defina una audiencia de vista previa: Agregar un solo correo electrónico cuenta AAD/MSA por línea, junto con una descripción opcional.**

Agregar manualmente las direcciones de correo electrónico de hasta diez (10), o veinte (20), si carga un archivo CSV, para la cuenta existente de Microsoft (MSA) o cuentas de Azure Active Directory (AAD) para ayudar a validar su oferta antes de la publicación en vivo. Al agregar estas cuentas, está definiendo una audiencia que se permitirá acceso de vista previa a su oferta antes de publicarlo en el marketplace(s). Si la oferta ya está en funcionamiento, todavía puede definir una audiencia de la versión preliminar para probar los cambios o actualizaciones a su oferta.

> [!NOTE]
> La audiencia de vista previa difiere de una audiencia privada. Una audiencia de vista previa puede tener acceso a su oferta _anterior_ para que se publican en vivo en los catálogos de soluciones. También puede crear un plan y que esté disponible solo para una audiencia privada. En el **planear lista** ficha, puede definir una audiencia privada con el **se trata de un plan privado** casilla de verificación. A continuación, puede definir una audiencia privada de hasta 20 000 clientes mediante identificadores de inquilino de Azure.

## <a name="technical-configuration"></a>Configuración técnica

El **configuración técnica** ficha define los detalles técnicos (ruta de acceso URL, webhook, Id. de inquilino y el identificador de aplicación) usados para conectarse a su oferta. Esta conexión nos permite aprovisionar su oferta como un recurso en la suscripción del cliente Azure si deciden para adquirirla.

- **URL de la página de aterrizaje** (obligatorio): Definir la dirección URL que los clientes se dirige lleguen a después de adquirir la oferta de marketplace de sitio. Esta URL también será el punto de conexión que recibirá las API de conexión para facilitar el comercio con Microsoft.

- **Webhook de conexión** (obligatorio): Para todos los eventos asincrónicos que Microsoft necesita enviarle en nombre del cliente (ejemplo: Suscripción de Azure ha pasado no válida), le pedimos que proporcione un webhook de conexión. Si aún no tiene un sistema de webhook en su lugar, la configuración más sencilla es que una aplicación de lógica de punto de conexión HTTP que escuchará los eventos que se va a registrar en ella y controlarlas adecuadamente (p. ej. https://prod-1westus.logic.azure.com:443/work). Para obtener más información, consulte [Llamada, desencadenamiento o anidamiento de flujos de trabajo con puntos de conexión HTTP en aplicaciones lógicas](https://docs.microsoft.com/azure/logic-apps/logic-apps-http-endpoint).

- **Id. de inquilino de Azure AD** (obligatorio): En Azure portal, es necesario que le [crear una aplicación de Azure Active Directory (AD)](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal) poder que podamos validar la conexión entre dos de nuestros servicios está detrás de una comunicación autenticada. Para buscar el [Id. de inquilino](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-tenant-id), vaya a Azure Active Directory y seleccione **propiedades**, a continuación, busque el **Id. de directorio** (por ejemplo, aparece el número 50c464d3-4930-494c-963c-1e951d15360e).

- **Identificador de aplicación de Azure AD** (obligatorio): También necesita la [Id. de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-application-id-and-authentication-key) y una clave de autenticación. Para obtener esos valores, vaya a Azure Active Directory y seleccione **registros de aplicaciones**, a continuación, busque el **Id. de aplicación** número enumerados (por ejemplo, 50c464d3-4930-494c-963c-1e951d15360e). Para buscar la clave de autenticación, vaya a **configuración** y seleccione **claves**. Deberá proporcionar una descripción y la duración y, a continuación, tendrá que proporcionarse un valor numérico.

 Tenga en cuenta que el identificador de aplicación de Azure está asociado a su Id. de publicador, así que asegúrese de que se usa el mismo identificador de aplicación en todas las ofertas.

## <a name="plan-overview"></a>Introducción a los planes

El **Introducción a los planes** pestaña le permite ofrecer una gran variedad de opciones del plan dentro de la misma oferta. Estos planes (a veces denominados SKU) podrían difieren en cuanto a la versión, monetización o niveles de servicio. Debe configurar al menos un plan para vender su oferta en marketplace.

Una vez creado, verá los nombres de plan, identificadores, modelos de precios, disponibilidad (público o privado), actual de publicación de estado y las acciones disponibles.

-   **Acciones** disponibles en el **Introducción a los planes** varían según el estado actual de su plan y pueden incluir:
  - Si el estado del plan es **borrador** : eliminar el borrador
  - Si el estado del plan es **Live** – Stop vender el plan o audiencia privada de sincronización

**Crear nuevo plan** (mínimo de un plan para aquellos que se seleccione esta opción para vender a través de Microsoft)

- **Id. de plan:** Crear un identificador de plan único para cada plan en esta oferta. Este identificador será visible para los clientes en las plantillas de Azure Resource Manager y de dirección URL de producto (si procede). Use solo caracteres alfanuméricos en minúscula, guiones o caracteres de subrayado. Se permite un máximo de 50 caracteres para este identificador de plan. Tenga en cuenta que el identificador no puede modificarse después de seleccionar crear.
- **Nombre del plan:** Los clientes verán este nombre de la hora de decidir que planea seleccionar dentro de su oferta. Cree un nombre de la oferta exclusiva para cada plan en esta oferta. El nombre del plan se usa para diferenciar los planes de software que pueden ser una parte de la misma oferta (p. ej. Nombre de la oferta: Windows Server; planes: Windows Server 2016, Windows Server 2019).

### <a name="plan-listing"></a>Anuncio del plan

El **planear lista** pestaña muestra los idiomas (y los mercados) donde está disponible, el plan actualmente inglés (Estados Unidos) es la única ubicación disponible. Además, esta página muestra el estado de la lista específica del lenguaje y la fecha y hora que se agregó. Deberá definir los detalles de marketplace (ofrecen el nombre, descripción, los términos de búsqueda, etc.) para cada idioma o el mercado.

#### <a name="plan-listing-details"></a>Detalles del anuncio del plan

Al seleccionar uno de los lenguajes de plan se mostrará los **planear lista** información, incluidos **nombre** y **descripción.**

- **Nombre**: Rellenado previamente en función de la versión preliminar **nuevo plan** entrada y aparecerá como título de su del oferta "plan de Software" muestra en marketplace.
- **Descripción:** Esta descripción es una oportunidad para explicar lo que hace que este plan de software único y las diferencias con respecto a otros planes de software dentro de su oferta. Puede contener hasta 500 caracteres.

Seleccione **guardar** después de completar estos campos.

#### <a name="plan-pricing-and-availability"></a>Plan de precios y disponibilidad

El **precios y disponibilidad** pestaña le permite configurar los mercados que este plan estará disponible en, el modelo de monetización deseado, precios y facturación término. Además, puede indicar si desea que el plan sea visible a todos los usuarios o solo a clientes específicos (una audiencia privada).

#### <a name="markets"></a>Mercados

- **Editar mercados** (opcional)

Cada plan debe estar disponible en al menos un mercado. Active la casilla para cualquier ubicación de mercado donde le gustaría disponer de este plan. Un cuadro de búsqueda y un botón para seleccionar países "Tax Remitted", en el que Microsoft remite de ventas y un impuesto de uso en su nombre, se incluyen para ayudar a. 


Si ya se ha establecido los precios del plan en dólares de Estados Unidos (USD) y agregar otra ubicación del mercado, el precio para el nuevo mercado se calculará según las tasas de cambio actuales. Siempre debe revisar el precio para cada mercado antes de la publicación. Los precios pueden revisarse mediante el vínculo "Exportar precios (xlsx)" después de guardar los cambios.

#### <a name="pricing"></a>Precios

- **Modelo de precios**: Tarifa plana o en función de puesto

**Tarifa plana:** Habilitar el acceso a su oferta con un precio de la tarifa plana único precio mensual o anual. Esto se denomina a veces precios basados en el sitio.

**En función de asiento:** Habilitar el acceso a su oferta con el precio en función del número de usuarios obtener acceso a la oferta u ocupando *puestos*. Este modelo basado en el asiento le permite establecer el mínimo y máximo número de puestos permitido en función del precio. De este modo, los diferentes puntos de precio pueden configurarse en función del número de usuarios mediante la configuración de varios planes.  Estos campos son opcionales. Si se deja vacío, el número de puestos se interpretará como si no tuviera un límite (mínimo de 1) y máxima de tantos como el sistema puede admitir. Estos campos se pueden editar como parte de una actualización de su plan.

Una vez publicado, la elección de modelo de precios facturación no se puede cambiar. Además, todos los planes de la misma oferta deben compartir el mismo modelo de precios.

- **Término de facturación**: Mensual o anual

Seleccione la frecuencia con que los clientes deben pagar el precio que aparece. Debe proporcionarse al menos un mensual o anual de precio, o ambas opciones pueden estar disponibles para los clientes.

- **Precio**: USD al mes o USD al año

Conjunto de los precios en la moneda local (USD = dólares de Estados Unidos) se convierten en la moneda local de todos los mercados con los tipos de cambio actuales disponibles durante la instalación. Validar estos precios antes de publicarlos mediante la exportación de la hoja de cálculo de precios y revisar el precio de cada mercado. Si desea establecer precios personalizados en un mercado individual, modificar e importar la hoja de cálculo de precios. Usted es responsable de validar los precios y posee estos valores.
**Primero debe guardar los cambios de precios para habilitar la exportación de los datos de precios.*

Revise los precios cuidadosamente antes de la publicación, ya que hay algunas restricciones en lo que puede cambiar una vez publicado un plan:

- Una vez que se publica un plan, no se puede cambiar el modelo de precios.
- Una vez que se publica un término de facturación para un plan, no se puede quitar más tarde.
- Una vez publicado un precio para un mercado en su plan, no puede cambiarse más adelante.

### <a name="plan-audience"></a>Audiencia de plan

Tiene la opción para configurar cada plan sea visible para todos los usuarios o solo una audiencia concreta de su elección. Puede asignar la pertenencia de esta audiencia restringida mediante identificadores de inquilino de Azure AD.

#### <a name="privacy"></a>Privacidad

- **Se trata de un plan privado** (casilla opcional)

Active esta casilla para que el plan de privado y solo está visible para el público restringido de su elección. Una vez publicado como un plan privado, puede actualizar la audiencia o elegir que el plan esté disponible para todos los usuarios. Una vez que se publica un plan como visible para todos los usuarios, debe permanecer visible para todos. (El plan no se puede configurar como un plan privado nuevo).

- **Público restringido (Id. de inquilino)**

Asigne a la audiencia que tendrá acceso a este plan privado. Se ha asignado acceso mediante los identificadores de inquilino con la opción para incluir una descripción de cada identificador de inquilino asignado. Se pueden agregar un máximo de identificadores de inquilinos 10 o identificadores de inquilino de 20.000 clientes si se importa un archivo de hoja de cálculo de CSV.

Un inquilino es una representación de una organización, con un identificador representado como un GUID (identificador único global, un número entero de 128 bits utilizado para identificar los recursos). Se trata de una instancia dedicada de Azure AD que una organización o el desarrollador de la aplicación reciben cuando crean una relación con Microsoft, como al registrarse en Azure, Microsoft Intune o Microsoft 365. Cada inquilino de Azure AD es distinto e independiente de los demás inquilinos de Azure AD. Para comprobar al inquilino, inicie sesión en el portal de Azure con la cuenta que desea usar para administrar la aplicación. Si tiene un inquilino, iniciará sesión de forma automática y verá el nombre de este directamente bajo el nombre de la cuenta. Mantenga el puntero sobre el nombre de su cuenta en el lado superior derecho de Azure Portal para ver el nombre, el correo electrónico, el directorio e identificador de inquilino (un GUID) y el dominio. Si la cuenta está asociada a varios inquilinos, puede seleccionar el nombre de la cuenta para abrir un menú donde puede cambiar entre los inquilinos. Cada inquilino tiene su propio identificador de inquilino. También puede buscar Id. de inquilino de su organización mediante una dirección URL de nombre de dominio en: [ https://www.whatismytenantid.com ](https://www.whatismytenantid.com).

Mientras que las ofertas de SaaS usan identificadores de inquilino para definir una audiencia privada, otros tipos de oferta pueden usar los identificadores de suscripción de Azure (que también se representan como GUID).

> [!NOTE]
> La audiencia privada (o público restringido) difiere de una audiencia de vista previa. En el **[Preview](#preview)** ficha, puede definir una audiencia de vista previa. Una audiencia de vista previa puede tener acceso a su oferta *anterior* a la oferta que se está publicada en vivo en marketplace. Aunque la designación de privada audiencia solo se aplica a un plan concreto, la audiencia de vista previa puede ver todos los planes (privada o no), pero solo durante el período de versión preliminar limitada, mientras que el plan se ha probado y validado.


## <a name="test-drive"></a>Versión de prueba

El **pruebe** pestaña le permite configurar una demostración (o "prueba") que permitirá a los clientes probar su oferta antes de comprometerse a comprarlos. Obtenga más información en el artículo [¿qué es la versión de prueba?](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/test-drive/what-is-test-drive). Si ya no desea proporcionar una versión de prueba de la oferta, volver a la **[el programa de instalación de la oferta](#offer-setup)** página y desactive la opción **activar versión de prueba**.

### <a name="technical-configuration"></a>Configuración técnica
Los siguientes tipos de las versiones de prueba están disponibles, cada uno con sus propios requisitos de configuración técnica.

- [Azure Resource Manager](#technical-configuration-for-azure-resource-manager-test-drive)
- [Dynamics 365](#technical-configuration-for-dynamics-365-test-drive)
- [Aplicación lógica](#technical-configuration-for-logic-app-test-drive)
- [Power BI](#technical-configuration-not-required-for-power-bi-test-drives) (configuración técnica no es necesario)

#### <a name="technical-configuration-for-azure-resource-manager-test-drive"></a>Versión de prueba de configuración técnica de Azure Resource Manager

Una plantilla de implementación que contiene todos los recursos de Azure que componen la solución. Los productos que se ajusten a este escenario usar solo los recursos de Azure. Más información acerca de cómo configurar un [versión de prueba de Azure Resource Manager](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/test-drive/azure-resource-manager-test-drive).

- **Regiones** (obligatorio): Actualmente hay 26 regiones compatibles de Azure donde la versión de prueba puede estar disponible. Normalmente, le interesará a disposición la versión de prueba en las regiones donde se prevé el mayor número de clientes, por lo que puede seleccionar la región más cercana para optimizar el rendimiento. Deberá asegurarse de que su suscripción tiene permiso para implementar todos los recursos necesarios en cada una de las regiones que se va a seleccionar.

- **Instances**: Seleccione el tipo (activa o inactiva) y el número de instancias disponibles, que se va a multiplicar por el número de regiones donde está disponible la oferta.

**Hot**: Este tipo de instancia está implementado y en espera de acceso por región seleccionada. Los clientes pueden acceder al instante *frecuente* instancias de una versión de prueba, en lugar de tener que esperar una implementación. La contrapartida es que estas instancias se ejecutan siempre en su suscripción de Azure, por lo que incurrirán en un mayor costo de tiempo de actividad. Se recomienda tener al menos una *frecuente* de instancia, tal como la mayoría de los clientes no quiere esperar para implementaciones completas, lo que produce una disminución en el uso del cliente si no hay ningún *frecuente* instancia está disponible.

**Cold**: Este tipo de instancia representa el número total de instancias que posiblemente se pueden implementar por región. Toda la plantilla de prueba de unidad Resource Manager para implementar cuando un cliente solicita la versión de prueba, por lo que las instancias en frío requieren *frío* instancias son mucho más lentas en cargar que *frecuente* instancias. El inconveniente es que solo tiene que pagar por la duración de la versión de prueba, es *no* siempre que se ejecuta en su suscripción de Azure como con un *frecuente* instancia.

- **Plantilla de Azure Resource Manager de unidad de prueba**: Cargue el archivo .zip que contiene la plantilla de Azure Resource Manager.  Más información sobre cómo crear una plantilla de Azure Resource Manager en el artículo de guía de inicio rápido [crear e implementar plantillas de Azure Resource Manager mediante el portal de Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-quickstart-create-templates-use-the-portal).

- **Duración de la versión de prueba** (obligatorio): Especifique el período de tiempo que permanecerá activa, en número de horas de la versión de prueba. La versión de prueba finaliza automáticamente cuando finaliza este período de tiempo. Esta duración es posible que sólo apostar conjunto por un número entero de horas (p. ej. horas de "2", "1.5" no es válido).

#### <a name="technical-configuration-for-dynamics-365-test-drive"></a>Versión de prueba de configuración técnica para Dynamics 365

Microsoft puede eliminar la complejidad de configurar una versión de prueba al hospedaje y mantener el servicio de aprovisionamiento y la implementación con este tipo de versión de prueba. La configuración para este tipo de versión de prueba hospedada es el mismo independientemente de si la versión de prueba está destinado a una audiencia Business Central, Customer Engagement u operaciones.

- **Máximo de unidades de pruebas simultáneas** (obligatorio): Establecer el número máximo de clientes que puede usar la versión de prueba al mismo tiempo. Cada usuario simultáneo consumirá una licencia de Dynamics 365, mientras que la versión de prueba está activa, por lo que deberá asegurarse de que tiene suficientes licencias disponibles para admitir el límite máximo establecido. Se recomienda un valor entre 3 y 5.

- **Duración de la versión de prueba** (obligatorio): Especifique el período de tiempo que permanecerá activa la versión de prueba mediante la definición de la cantidad de horas. Después de este número de horas, la sesión finalizará y ya no adopta una de las licencias. Se recomienda un valor de 2 a 24 horas según la complejidad de la oferta. Esta duración es posible que sólo apostar conjunto por un número entero de horas (p. ej. horas de "2", "1.5" no es válido).  El usuario puede solicitar una nueva sesión si ejecutan fuera del tiempo y desean volver a acceder a la versión de prueba.

- **Dirección URL de la instancia** (obligatorio): La dirección URL donde el cliente comenzará su versión de prueba. Normalmente, la dirección URL de la instancia de Dynamics 365 que ejecutan la aplicación con datos de ejemplo instalados (por ejemplo, https://testdrive.crm.dynamics.com).

- **Instancia de Web API URL** (obligatorio): Recuperar la dirección URL de API Web para la instancia de Dynamics 365, iniciar sesión en su cuenta de Microsoft 365 y vaya a **configuración** \&gt; **Personalización** \&gt; **Recursos para desarrolladores** \&gt; **API de instancia de Web (dirección URL raíz del servicio)**, copie la dirección URL que se encuentra aquí (p. ej. https://testdrive.crm.dynamics.com/api/data/v9.0).

- **Nombre de la función** (obligatorio): Proporcione el nombre del rol de seguridad que ha definido en la versión de prueba de Dynamics 365 personalizada. Se asignará al usuario durante su versión de prueba (por ejemplo, la prueba-Bólido rol).

#### <a name="technical-configuration-for-logic-app-test-drive"></a>Configuración técnica para la versión de prueba de aplicación lógica

Los productos personalizados deben usar este tipo de plantilla de implementación de prueba de unidad que abarca una variedad de arquitecturas de soluciones complejas. Para obtener más información sobre cómo configurar la aplicación lógica de las versiones de prueba, visite [operaciones](https://github.com/Microsoft/AppSource/blob/master/Setup-your-Azure-subscription-for-Dynamics365-Operations-Test-Drives.md) y [Customer Engagement](https://github.com/Microsoft/AppSource/wiki/Setting-up-Test-Drives-for-Dynamics-365-app) en GitHub.

- **Región** (requerido, una sola selección desplegable lista): Actualmente hay 26 regiones compatibles de Azure donde la versión de prueba puede estar disponible. Los recursos de la aplicación lógica se implementarán en la región seleccionada. Si la aplicación lógica tiene los recursos personalizados almacenados en una región específica, asegúrese de que se seleccionará esa región. La mejor manera de hacerlo es implementar la aplicación lógica localmente en su suscripción en el portal de Azure totalmente y comprobar que funciona correctamente antes de realizar esta selección.

- **Máximo de unidades de pruebas simultáneas** (obligatorio): Establecer el número máximo de clientes que puede usar la versión de prueba al mismo tiempo. Estas unidades ya están implementadas, permitiendo a los clientes tener acceso al instante a ellos sin tener que esperar para una implementación de prueba.

- **Duración de la versión de prueba** (obligatorio): Especifique el período de tiempo que permanecerá activa, en número de horas de la versión de prueba. La versión de prueba finaliza automáticamente después de que finalice este período de tiempo.

- **Nombre del grupo de recursos de Azure** (obligatorio): Escriba el [grupo de recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#resource-groups) nombre donde se guarda la versión de prueba de la aplicación lógica.

- **Nombre de la aplicación lógica de Azure** (obligatorio): Escriba el nombre de la aplicación lógica que se asigna a la versión de prueba para el usuario. Esta aplicación lógica debe guardarse en el grupo de recursos de Azure anterior.

- **Nombre de la aplicación lógica desaprovisionamiento** (obligatorio): Escriba el nombre de la aplicación lógica que desaprovisiona la versión de prueba una vez que el cliente haya finalizado. Esta aplicación lógica debe guardarse en el grupo de recursos de Azure anterior.

#### <a name="technical-configuration-not-required-for-power-bi-test-drives"></a>Configuración técnica no es necesario para las versiones de prueba de Power BI

Productos que desean demostrar que un visual interactivos de Power BI puede usar un vínculo incrustado para compartir un panel personalizado como su versión de prueba, ninguna otra configuración técnica necesaria. Más información acerca de cómo configurar[Power BI](https://docs.microsoft.com/power-bi/service-template-apps-overview) aplicaciones de la plantilla.

### <a name="deployment-subscription-details"></a>Detalles de la suscripción de implementación

Para implementar la versión de prueba en su nombre, crear y proporcionar una suscripción de Azure independientes y exclusivos. (No es necesario para las versiones de prueba de Power BI).

- **Id. de suscripción de Azure** (necesario para aplicaciones lógicas y Azure Resource Manager): Escriba el identificador de suscripción para conceder acceso a los servicios de la cuenta de Azure para el uso de recursos de informes y facturación. Le recomendamos que considere [crear una suscripción de Azure independiente](https://docs.microsoft.com/azure/billing/billing-create-subscription) que se usará para las versiones de prueba, si aún no tiene uno. Puede encontrar el identificador de suscripción de Azure iniciando sesión en el [portal Azure](https://portal.azure.com/) y navegar a la **suscripciones** ficha del menú del lado izquierdo. Seleccione la pestaña mostrará el identificador de suscripción (por ejemplo, "a83645ac-1234-5ab6-6789-1h234g764ghty").

- **Id. de inquilino de Azure AD** (obligatorio): Escriba su Azure Active Directory (AD) [Id. de inquilino](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-tenant-id). Para buscar este identificador, inicie sesión en el [portal de Azure](https://portal.azure.com/), seleccione la pestaña Active Directory en el menú izquierdo, seleccione **propiedades** , a continuación, busque el **Id. de directorio** (por ejemplo, aparece el número 50c464d3-4930-494c-963c-1e951d15360e). También puede buscar Id. de inquilino de su organización mediante la dirección URL de nombre de dominio en: [ https://www.whatismytenantid.com ](https://www.whatismytenantid.com).

- **Nombre del inquilino de Azure AD** (necesario para Dynamics 365): Escriba el nombre de Azure Active Directory (AD). Para buscar este nombre, inicie sesión en el [portal Azure](https://portal.azure.com/), en la esquina superior derecha se mostrarán el nombre del inquilino en su nombre de cuenta.

- **Identificador de aplicación de Azure AD** (obligatorio): Escriba su Azure Active Directory (AD) [Id. de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-application-id-and-authentication-key). Para buscar este identificador, inicie sesión en el [portal de Azure](https://portal.azure.com/), seleccione la pestaña Active Directory en el menú izquierdo, seleccione **registros de aplicaciones**, a continuación, busque el **Id. de aplicación** número enumerados (por ejemplo, 50c464d3-4930-494c-963c-1e951d15360e).

- **Clave de aplicación de Azure AD** (obligatorio): Escriba su Azure Active Directory (AD) [clave de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-application-id-and-authentication-key). Para buscar este identificador, inicie sesión en el [portal Azure](https://portal.azure.com/), seleccione la pestaña Active Directory en el menú izquierdo, seleccione **registros de aplicaciones** , a continuación, seleccione **configuración**  >  **Claves**.

No olvide **guardar** antes de continuar con la siguiente sección.

### <a name="test-drive-listings-optional"></a>Anuncios de unidad de prueba (opcional)

El **anuncios de la versión de prueba** opción se encuentra en la **pruebe** pestaña muestra los idiomas (y los mercados) donde está disponible, la versión de prueba actualmente inglés (Estados Unidos) es la única ubicación disponible . Además, esta página muestra el estado de la lista específica del lenguaje y la fecha y hora que se agregó. Deberá definir los detalles de unidad de prueba (descripción, manual del usuario, vídeos, etc.) para cada idioma o el mercado.

- **Descripción** (obligatorio): Describe la versión de prueba, lo que se exponen los objetivos para el usuario para experimentar con características para explorar y cualquier información relevante para ayudar al usuario determinar si se debe adquirir la oferta. En este campo se pueden introducir hasta 3000 caracteres de texto. 

- **Obtener acceso a información** (necesario para las versiones de prueba de Azure Resource Manager y lógica): Se explica lo que un cliente necesita conocer con el fin de obtener acceso y usar esta versión de prueba. Recorrer un escenario de uso de la oferta y exactamente lo que el cliente debería conocer para tener acceso a las características a lo largo de la versión de prueba. Pueden escribir hasta 10 000 caracteres de texto en este campo.

- **Manual del usuario** (obligatorio): Un tutorial detallado de su experiencia de versión de prueba. El Manual del usuario deben abarcar exactamente lo que quiere que el cliente obtener experimente la versión de prueba y actuar como referencia para las preguntas que pueden tener. El archivo debe estar en formato PDF y denominarse (255 caracteres como máximo) después de cargar.

- **Videos: Agregar vídeos** (opcional): Vídeos se pueden cargar en YouTube o Vimeo y hace referencia aquí con una imagen del vínculo y la miniatura (533 × 324 píxeles) para que un cliente pueda ver un recorrido a través de la información para ayudarles a comprender mejor la versión de prueba, incluida la forma de usar correctamente las características de su la oferta y comprender los escenarios que destacan sus ventajas.
  - **Nombre** (obligatorio)
  - **Dirección URL (YouTube o Vimeo sólo)** (obligatorio)
  - **Miniatura (533 x 324px)**: Archivo de imagen debe estar en formato PNG.

Seleccione **guardar** después de completar estos campos.

## <a name="publish"></a>Publicar

#### <a name="submit-offer-to-preview"></a>Enviar la oferta obtener una vista previa

Una vez haya completado todas las secciones necesarias de la oferta, seleccione **publicar** en la esquina superior derecha del portal. Se redirige a la **revisión y publicación de** página. 

Si se trata de la primera vez que publica esta oferta, puede:

- Ver el estado de finalización para cada sección de la oferta.
    - *No iniciado* : significa que no ha sido modificada la sección y se debe completar.
    - *Incompleta* : significa que la sección tiene errores que deben corregirse o requiere que se proporciona más información. Por favor, vuelva a las secciones y actualizarlo.
    - *Completa* : significa que la sección está completa, se ha proporcionado todos los datos necesarios y no hay ningún error. Todas las secciones de la oferta deben estar en un estado completamente antes de poder enviar la oferta.
- Se proporcionan instrucciones de pruebas al equipo de certificación para asegurarse de que la aplicación se ha probado correctamente, además de las notas adicionales útiles para entender la aplicación.
- Enviar la oferta para la publicación seleccionando **enviar**. Le enviaremos un correo electrónico que le avise cuando una versión de vista previa de la oferta está disponible para que pueda revisar y aprobar. Debe volver al centro de partners y seleccionar **Go-live** de la oferta para publicar su oferta en público (o si ofrece una privada, a la audiencia privada).

## <a name="next-steps"></a>Pasos siguientes

- [Actualizar una oferta en Marketplace comercial existente](./update-existing-offer.md)
