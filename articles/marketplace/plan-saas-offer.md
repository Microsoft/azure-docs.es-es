---
title: Planeamiento de una oferta de SaaS en el marketplace comercial de Microsoft
description: Cómo planear una nueva oferta de software como servicio (SaaS) para publicarla o venderla en Microsoft AppSource, Azure Marketplace o mediante el programa del Proveedor de soluciones en la nube (CSP) por medio del programa del marketplace comercial del Centro de partners de Microsoft.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 08/30/2020
ms.openlocfilehash: 60ca3723831b1d47e5c20f292394386df08ebdd1
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89380513"
---
# <a name="how-to-plan-a-saas-offer-for-the-commercial-marketplace"></a>Planeamiento de una oferta de SaaS en el marketplace comercial

En este artículo se explican los distintos requisitos y opciones para publicar una oferta de software como servicio (SaaS) en el marketplace comercial de Microsoft. Las ofertas de SaaS permiten ofrecer soluciones de software, y conceder licencias para ellas, a los clientes mediante una suscripción en línea en lugar de la instalación local en equipos individuales. Este artículo le ayudará a preparar la oferta para publicarla en el marketplace comercial con el Centro de partners.

## <a name="listing-options"></a>Opciones de publicación

Cuando prepare la publicación de una nueva oferta de SaaS, debe decidir la opción de _publicación_. Esta opción determina la información adicional que deberá proporcionar cuando cree la oferta en el Centro de partners. La opción de publicación se define en la página **Configuración de la oferta**, según se explica en [Creación de una oferta de SaaS en el marketplace comercial](create-new-saas-offer.md).

En la tabla siguiente se muestran las opciones de publicación de las ofertas de SaaS en el marketplace comercial.

| Opción de publicación | Proceso de transacción |
| ------------ | ------------- |
| Ponerse en contacto conmigo | El cliente se pone en contacto directamente a partir de la información de la descripción.``*`` |
| Evaluación gratuita | Al cliente se le redirige a la dirección URL de destino mediante Azure Active Directory (Azure AD).``*`` |
| Obténgalo ahora (de forma gratuita) | Al cliente se le redirige a la dirección URL de destino a través de Azure AD.``*`` |
| Venta mediante Microsoft  | Las ofertas vendidas a través de Microsoft se denominan ofertas _procesables_. Una oferta procesable es aquella en la que Microsoft facilita el intercambio de dinero por una licencia de software en nombre del editor. Microsoft factura las ofertas de SaaS con el modelo de precios elegido y administra las transacciones de los clientes en su nombre. Tenga en cuenta que las cuotas de uso de la infraestructura de Azure se le facturan al asociado directamente. Al elegir el modelo de precio, es necesario considerar los costos de la infraestructura. Este aspecto se explica con más detalle a continuación en [Facturación de SaaS](#saas-billing).  |
|||

``*``Los editores son responsables de todos los aspectos de la transacción de las licencias de software, incluidas, entre otras cosas, el pedido, la entrega, la medición, la facturación, el pago y el cobro.

Para más información sobre estas opciones de publicación, consulte [Capacidades de transacción de marketplace comercial](marketplace-commercial-transaction-capabilities-and-considerations.md).

Una vez publicada la oferta, la opción de publicación elegida para ella aparece en forma de un botón en la esquina superior izquierda de la página de descripción de la oferta. Por ejemplo, en la siguiente captura de pantalla se muestra una página de descripción de la oferta en Azure Marketplace con los botones **Ponerse en contacto conmigo** y **Versión de prueba**.

![Se muestra la descripción de una oferta en la tienda en línea.](./media/listing-options.png)

## <a name="technical-requirements"></a>Requisitos técnicos

Los requisitos técnicos varían en función de la opción de publicación que elija para su oferta.

La opción _Ponerse en contacto conmigo_ no tiene requisitos técnicos. Tiene la opción de conectar un sistema CRM para administrar clientes potenciales, que se describe en la sección [Clientes potenciales](#customer-leads), más adelante en este artículo.

Las opciones de publicación _Obtener ahora (gratis)_ , _Evaluación gratuita_ y _Venta mediante Microsoft_ presentan los siguientes requisitos técnicos:

- La aplicación de SaaS debe ser una solución para varios inquilinos.
- Puede permitir la autenticación de los usuarios tanto con cuentas de Microsoft (MSA) como de [Azure Active Directory (Azure AD)](https://azure.microsoft.com/services/active-directory/).
- Debe crear una página de aterrizaje. Después de que un usuario compre la oferta, se le dirigirá a la página de aterrizaje para facilitar el aprovisionamiento o la configuración adicionales que sean necesarios. Puede encontrar instrucciones sobre la creación de la página de aterrizaje en estos artículos:
  - [Creación de la página de aterrizaje de su oferta de SaaS comercializable en el marketplace comercial](azure-ad-transactable-saas-landing-page.md)
  - [Creación de la página de aterrizaje de su oferta de SaaS gratuita o de evaluación en el marketplace comercial](azure-ad-free-or-trial-landing-page.md)

Estos requisitos técnicos adicionales se aplican solo a la opción de publicación _Venta mediante Microsoft_ (procesable):

- se requiere Azure AD con la autenticación y la administración de identidades de inicio de sesión único (SSO). Para más información, consulte [Azure AD y ofertas de SaaS comercializables en el marketplace comercial](azure-ad-saas.md).
- Debe usar las [API de cumplimiento de SaaS](./partner-center-portal/pc-saas-fulfillment-api-v2.md) para la integración con Azure Marketplace y Microsoft AppSource. Debe exponer servicios que puedan interactuar con la suscripción de SaaS para crear, actualizar y eliminar una cuenta de usuario y un plan de servicio. Los cambios importantes en la API deben admitirse dentro de un plazo de 24 horas. Los cambios no importantes en la API se publicarán de forma periódica. En la documentación de las [API](./partner-center-portal/pc-saas-fulfillment-api-v2.md) hay diagramas y explicaciones detalladas que describen el uso de los campos recopilados.
- Debe crear al menos un plan para la oferta. El precio del plan se basa en el modelo de precios seleccionado antes de la publicación: _tarifa plana_ o _por usuario_. Se proporcionan más detalles sobre los [planes](#plans) más adelante en este artículo.
- El cliente puede cancelar su oferta en cualquier momento.

### <a name="technical-information"></a>Información técnica

Si va a crear una oferta procesable, deberá recopilar la siguiente información para la página **Configuración técnica**. Si en lugar de crear una oferta procesable decide procesar las transacciones de forma independiente, omita esta sección y vaya a [Versiones de prueba](#test-drives).

- **Dirección URL de la página de aterrizaje**: la dirección URL del sitio de SaaS (por ejemplo, `https://contoso.com/signup`) a la que se dirigirá a los usuarios después de adquirir la oferta en el marketplace comercial, que desencadena el proceso de configuración de la suscripción de SaaS recién creada. Esta dirección URL recibirá un token que se puede usar para llamar a las API de entrega y obtener los detalles de aprovisionamiento de la página de registro interactivo.

  Se llamará a esta dirección URL con el parámetro del token de identificación de compra de marketplace, que identifica de forma única la compra de SaaS del cliente específico. Este token se debe intercambiar por los detalles de la suscripción de SaaS correspondiente mediante la [API de resolución](./partner-center-portal/pc-saas-fulfillment-api-v2.md#resolve-a-purchased-subscription). Tanto estos detalles como otros que desee recopilar deben usarse como parte de una página web interactiva del cliente integrada en su experiencia para completar el registro y activar su compra. En esta página, el usuario debe registrarse mediante la autenticación con un solo clic con Azure Active Directory (Azure AD).

  Esta dirección URL también se llamará con el parámetro del token de identificación de compra de marketplace cuando el cliente inicie la experiencia de SaaS administrada desde Azure Portal o el Centro de administración de M365. Ambos flujos deben administrarse: la primera vez que se proporciona el token después de la compra de un nuevo cliente y cuando se proporciona de nuevo para un cliente existente que administra su solución de SaaS.

    La página de aterrizaje que configure debe estar en funcionamiento de forma ininterrumpida. Esta es la única vía por la que se le notificarán nuevas compras de las ofertas de SaaS realizadas en el marketplace comercial o las solicitudes de configuración de una suscripción activa a una oferta.

- **Webhook de conexión**: para todos los eventos asincrónicos que Microsoft tiene que enviarle (por ejemplo, si la suscripción a SaaS se ha cancelado), le pedimos que nos proporcione una URL de webhook de conexión. Se llamará a esta dirección URL para notificarle sobre el evento.

  El webhook que proporcione debe estar en funcionamiento de forma ininterrumpida, ya que esta es la única vía por la que se le notificarán las actualizaciones de las suscripciones de SaaS de sus clientes adquiridas a través del marketplace comercial.

  > [!NOTE]
  > Dentro de Azure Portal, es necesario que cree una [aplicación de Azure Active Directory (Azure AD)](../active-directory/develop/howto-create-service-principal-portal.md) de un único inquilino para permitir el uso de un identificador de AzureApp para autenticar la conexión entre los dos servicios. Para encontrar el [identificador de inquilino](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in), vaya a su instancia de Azure Active Directory, seleccione **Propiedades** y busque el número del identificador de directorio que se muestra. Por ejemplo, `50c464d3-4930-494c-963c-1e951d15360e`.

- **Id. de inquilino de Azure Active Directory**: (también conocido como identificador de directorio). Dentro de Azure Portal, es necesario [registrar una aplicación de Azure Active Directory (AD)](../active-directory/develop/howto-create-service-principal-portal.md) para poder agregarla a la lista de control de acceso (ACL) de la API a fin de garantizar que está autorizado para llamarla. Para encontrar el identificador de inquilino de la aplicación de Azure Active Directory (AD), vaya a la hoja [Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) de Azure Active Directory. En la columna **Nombre para mostrar**, seleccione la aplicación. Luego, busque el número de **Id. de directorio (inquilino)** que se muestra (por ejemplo, `50c464d3-4930-494c-963c-1e951d15360e`).

- **Id. de aplicación de Azure Active Directory**: también necesitará el [identificador de la aplicación](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in). Para obtener su valor, vaya a la hoja [Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) de Azure Active Directory. En la columna **Nombre para mostrar**, seleccione la aplicación. Luego, busque el número de identificador de la aplicación (cliente) que se muestra (por ejemplo, `50c464d3-4930-494c-963c-1e951d15360e`).

  El identificador de la aplicación de Azure AD está asociado a su identificador de editor de la cuenta del Centro de partners. El mismo identificador de aplicación debe usarse con todas las ofertas de esa cuenta.

  > [!NOTE]
  > Si el publicador tiene dos o más cuentas diferentes en el Centro de partners, deben usarse dos o más id. de la aplicación de Azure AD diferentes, uno para cada cuenta. Cada cuenta de partner del Centro de partners debe usar un identificador de aplicación de Azure AD único para todas las ofertas de SaaS publicadas a través de esta cuenta.

## <a name="test-drives"></a>Versiones de prueba
Se puede elegir habilitar una versión de prueba de la aplicación SaaS. Las versiones de prueba proporcionan a los clientes acceso a un entorno preconfigurado durante un número fijo de horas. Aunque puede habilitar las opciones de publicación en cualquier versión de prueba, esta característica conlleva requisitos adicionales. Para más información sobre las versiones de prueba, consulte [¿Qué es una versión de prueba?](what-is-test-drive.md). Para información sobre la configuración de diferentes tipos de versiones de prueba, consulte [Configuración técnica de la versión de prueba](test-drive-technical-configuration.md).

> [!TIP]
> Una versión de prueba es diferente de una [evaluación gratuita](plans-pricing.md#free-trials). Puede ofrecer una versión de prueba, una evaluación gratuita o ambas. Las dos proporcionan a los clientes la solución durante un período fijo. Sin embargo, una versión de prueba también incluye un práctico recorrido autoguiado por las principales características y ventajas del producto demostradas en un escenario de implementación real.

## <a name="customer-leads"></a>Clientes potenciales

Debe conectar su oferta al sistema de administración de relaciones con clientes (CRM) para recopilar información del cliente. El cliente le pedirá permiso para compartir su información. Estos datos del cliente, junto con el nombre y el identificador de la oferta, además de la tienda en línea donde la encontró, se envían al sistema CRM que haya configurado. El marketplace comercial admite una gran variedad de sistemas CRM, junto con la opción de usar una tabla de Azure o de configurar un punto de conexión HTTPS con Power Automate.

Puede agregar o modificar una conexión CRM en cualquier momento durante o después de la creación de la oferta. Se pueden encontrar instrucciones detalladas en [Administración de clientes potenciales del marketplace comercial](lead-management-for-cloud-marketplace.md).

## <a name="selecting-an-online-store"></a>Selección de una tienda en línea

Al publicar una oferta de SaaS, se mostrará en Microsoft AppSource, Azure Marketplace o ambos. Cada tienda en línea atiende los requisitos únicos de cada cliente. AppSource es para soluciones empresariales y Azure Marketplace para soluciones de TI. El tipo de oferta, las funcionalidades de transacción y la categoría determinarán dónde se publicará la oferta. Las categorías y subcategorías se asignan a cada tienda en línea en función del público de destino. Para información detallada sobre cómo seleccionar una tienda en línea, consulte [Selección de una tienda en línea](determine-your-listing-type.md#selecting-an-online-store).

## <a name="legal-contracts"></a>Contratos legales

Para simplificar el proceso de adquisición para los clientes y reducir la complejidad legal para los proveedores de software, Microsoft ofrece un contrato estándar que se puede usar con las ofertas del marketplace comercial. Al ofrecer el software bajo el contrato estándar, los clientes solo tienen que leerlo y aceptarlo, sin necesidad de crear términos y condiciones personalizados.

Si decide usar el contrato estándar, tiene la opción de agregarle términos de modificación universales y hasta 10 modificaciones personalizadas. También puede usar sus propios términos y condiciones en lugar del contrato estándar. Estos detalles se administrarán en la página **Propiedades**. Para más información, consulte [Contrato estándar para el marketplace comercial de Microsoft](standard-contract.md).

> [!NOTE]
> Después de publicar una oferta con el contrato estándar en el marketplace comercial, no puede usar sus propios términos y condiciones personalizados. Solo puede elegir una de las dos opciones. O bien ofrecer su solución bajo el contrato estándar o según sus propios términos y condiciones. Si quiere modificar los términos del contrato estándar, puede hacerlo a través de las modificaciones al contrato estándar.

## <a name="offer-listing-details"></a>Detalles de la descripción de la oferta

Al [crear una oferta de SaaS](create-new-saas-offer.md) en el Centro de partners, insertará texto, imágenes, vídeos opcionales y otros detalles en la página **Descripción de la oferta**. Esta es la información que verán los clientes cuando descubran la descripción de su oferta en el marketplace comercial, como se muestra en el ejemplo siguiente.

:::image type="content" source="./media/example-saas-1.png" alt-text="Ilustración de cómo aparece esta oferta en Microsoft AppSource.":::

**Descripciones destacadas**

1. Logotipo
2. Categorías
3. Industrias
4. Dirección de soporte técnico (vínculo)
5. Términos de uso
6. Directiva de privacidad
7. Nombre de la oferta
8. Resumen
9. Descripción
10. Capturas de pantallas o vídeos
11. Documentos

En el ejemplo siguiente se muestra la descripción de una oferta en Azure Portal.

![Se muestra la descripción de una oferta en Azure Portal.](./media/example-managed-services.png)

**Descripciones de llamadas**

1. Título
1. Descripción
1. Vínculos útiles
1. Capturas de pantalla

> [!NOTE]
> No es necesario que el contenido de la descripción de la oferta esté en inglés si comienza con la frase "Esta aplicación solo está disponible en [idioma distinto del inglés]".

Para facilitar la creación de la oferta, prepare algunos de estos elementos con antelación. Los elementos siguientes son obligatorios a menos que se indique lo contrario.

- **Name**: este nombre aparecerá como título de la descripción de la oferta en el marketplace comercial. El nombre puede ser una marca comercial. No puede contener emojis (a menos que sean símbolos de marca comercial y copyright), y no puede tener más de 50 caracteres.
- **Resumen de los resultados de la búsqueda**: describa el propósito o la función de la oferta en una sola frase de 100 caracteres como máximo y sin saltos de línea. Este resumen se usa en los resultados de búsqueda de las ofertas del marketplace comercial.
- **Descripción**: esta descripción se mostrará en la información general de las ofertas del marketplace comercial. Considere la posibilidad de incluir una propuesta de valor, los beneficios principales, los compradores previstos, las asociaciones de categoría o sector, las oportunidades de compra en la aplicación, todas las divulgaciones necesarias y un vínculo para obtener más información.
    
    Este cuadro de texto tiene controles de editor de texto enriquecido que puede utilizar para que su descripción sea más atractiva. También puede usar etiquetas HTML para dar formato a la descripción. En este cuadro puede escribir hasta 3000 caracteres de texto, incluido el marcado HTML. Consulte [Escribir una excelente descripción de la aplicación](https://docs.microsoft.com/windows/uwp/publish/write-a-great-app-description) para encontrar más sugerencias.

- **Instrucciones de inicio** Si opta por vender su oferta a través de Microsoft (oferta procesable), este campo es obligatorio. Estas son instrucciones para ayudar a los clientes a conectarse a su oferta de SaaS. Puede agregar hasta 3000 caracteres de texto y vínculos a documentación en línea más detallada.
- **Palabras clave de búsqueda** (opcional): escriba al menos tres palabras clave de búsqueda que los clientes puedan usar para buscar su oferta en los marketplaces. No es necesario incluir el **nombre** y la **descripción** de la oferta: ese texto se incluye automáticamente en la búsqueda.
- **Vínculo a la directiva de privacidad**: la dirección URL que lleva a la directiva de privacidad de su empresa. Debe proporcionar una directiva de privacidad válida y usted es responsable de garantizar que la aplicación cumple con las leyes y normativas de privacidad.
- **Información de contacto**: debe designar los siguientes contactos de la organización:
  - **Contacto de soporte técnico**: proporcione el nombre, el teléfono y el correo electrónico de los asociados de Microsoft que se usarán cuando los clientes abran vales. También debe incluir la dirección URL del sitio web de soporte técnico.
  - **Contacto de ingeniería**: proporcione el nombre, el teléfono y el correo electrónico que Microsoft usará directamente cuando haya problemas con la oferta. Esta información de contacto no aparece en el marketplace.
  - **Contacto del programa CSP** (opcional): proporcione el nombre, el teléfono y el correo electrónico si participa en el programa CSP, de modo que dichos asociados puedan ponerse en contacto con usted con cualquier pregunta. También puede incluir una dirección URL a sus materiales de marketing.
- **Vínculos útiles** (opcional): puede proporcionar vínculos a varios recursos para los usuarios de su oferta. Por ejemplo, foros, preguntas más frecuentes y notas de la versión.
- **Documentos relacionados**: puede proporcionar hasta tres documentos orientados al cliente, como notas del producto, folletos, listas de comprobación o presentaciones de PowerPoint.
- **Elementos multimedia (logotipos)** : Proporcione un archivo PNG para el logotipo de tamaño **Grande**. El Centro de partners lo usará para crear un logotipo **Pequeño** y un logotipo **Mediano**. Opcionalmente, puede reemplazarlos por imágenes diferentes más adelante.

   - Grande (de 216 x 216 a 350 x 350 píxeles, obligatorio)
   - Mediano (90 x 90 píxeles, opcional)
   - Pequeño (48 x 48 píxeles, opcional)

  Estos logotipos se usan en distintos lugares de las tiendas en línea:

  -  El logotipo pequeño aparece en los resultados de la búsqueda de Azure Marketplace, y en la página principal y en la página de resultados de búsqueda de Microsoft AppSource.
  -  El logotipo mediano aparece cuando se crea un recurso en Microsoft Azure.
  -  El logotipo grande aparece en la página de descripción de la oferta de Azure Marketplace y Microsoft AppSource.

- **Elementos multimedia (capturas de pantalla)** : debe agregar entre una y cinco capturas de pantallas que muestren el funcionamiento de la oferta, con los siguientes requisitos:
  - 1280 x 720 píxeles
  - Archivo .png
  - Se debe incluir un título
- **Elementos multimedia (vídeos)** (opcional): puede agregar hasta cuatro vídeos que muestren su oferta, con los siguientes requisitos:
  - Nombre
  - Dirección URL: solo se debe hospedar en YouTube o en Vimeo.
  - Miniatura: archivo .png de 1280 x 720

> [!Note]
> La oferta debe cumplir las [directivas de certificación del marketplace comercial](https://docs.microsoft.com/legal/marketplace/certification-policies#100-general) generales y las [directivas de software como servicio](https://docs.microsoft.com/legal/marketplace/certification-policies#1000-software-as-a-service-saas) que se van a publicar en el marketplace comercial.

## <a name="preview-audience"></a>Público preliminar
Una audiencia preliminar puede tener acceso a la oferta antes de publicarla en las tiendas en línea a fin de probar la funcionalidad completa. En la página **Audiencia preliminar**, puede definir una audiencia preliminar limitada. Esta opción no está disponible si elige procesar las transacciones de forma independiente en lugar de vender su oferta mediante Microsoft. En este caso, puede omitir esta sección e ir a [Oportunidades de venta adicionales](#additional-sales-opportunities).

> [!NOTE]
> Una audiencia preliminar no es lo mismo que un plan privado. Un plan privado es el que solo está disponible para una audiencia determinada de su elección. Esto le permite negociar un plan personalizado con clientes específicos. Para más información, vea la siguiente sección. Planes

Puede enviar invitaciones a direcciones de correo electrónico de cuentas de Microsoft (MSA) o de Azure Active Directory (Azure AD). Agregue hasta 10 direcciones de correo electrónico manualmente o importe hasta 20 con un archivo .csv. Aunque la oferta ya esté publicada, puede definir una audiencia preliminar para probar los cambios o las actualizaciones que realice en ella.

## <a name="plans"></a>Planes

Las ofertas procesables requieren al menos un plan. Un plan define el ámbito y los límites de la solución, así como los precios asociados. Puede crear varios planes para que su oferta brinde a sus clientes diferentes opciones técnicas y de precios. Si elige procesar las transacciones de forma independiente en lugar de crear una oferta procesable, la página **Planes** no es visible. En este caso, omita esta sección y vaya a [Oportunidades de venta adicionales](#additional-sales-opportunities).

Consulte [Planes y precios de ofertas de marketplace comercial](plans-pricing.md) para obtener instrucciones generales sobre los planes, incluidos los modelos de precios, las evaluaciones gratuitas y los planes privados. En las secciones siguientes se describe información adicional específica de las ofertas de SaaS.

### <a name="saas-pricing-models"></a>Modelos de precios de SaaS

Las ofertas de SaaS pueden usar dos modelos de precios con cada plan: _tarifa plana_ o _por usuario_. Todos los planes de la misma oferta deben estar asociados al mismo modelo de precios. Por ejemplo, una oferta no puede tener un plan de tarifa plana y un plan por usuario.

**Tarifa plana**: habilite el acceso a la oferta con unA única tarifa plana mensual o anual. A veces esto se denomina precios basados en puesto. Con este modelo de precios, puede definir opcionalmente planes de uso medido que emplean la API del servicio de medición del marketplace para cobrar a los clientes por el uso que no está cubierto por la tarifa plana. Para más información sobre la facturación del uso medido, consulte [Facturación según uso con el servicio de medición de marketplace](./partner-center-portal/saas-metered-billing.md). También debe usar esta opción si el comportamiento del servicio SaaS presenta ráfagas.

**Por usuario**: habilite el acceso a su oferta con un precio basado en el número de usuarios que pueden acceder a la oferta u ocupar los puestos. Con este modelo basado en el usuario, puede establecer el número mínimo y máximo de usuarios admitidos por el plan. Además, puede crear varios planes para configurar diferentes puntos de precio en función del número de usuarios. Estos campos son opcionales. Si no se seleccionan, se interpretará que el número de usuarios no tiene límite (el mínimo es 1 y el máximo es tantos como el sistema pueda admitir). Estos campos se pueden editar como parte de una actualización del plan.

> [!IMPORTANT]
> Una vez publicada la oferta, no se puede cambiar el modelo de precios. Además, todos los planes de la misma oferta deben compartir el mismo modelo de precios.

### <a name="saas-billing"></a>Facturación de SaaS

En el caso de aplicaciones SaaS que se ejecutan en la suscripción de Azure (del editor), el uso de la infraestructura se factura directamente; los clientes no ven las tarifas de uso de la infraestructura real. Debe agrupar las tarifas de uso de la infraestructura de Azure en el precio de la licencia de software para compensar el costo de la infraestructura que implementó para ejecutar la solución.

Las ofertas de aplicaciones SaaS que se venden a través de Microsoft admiten facturación mensual o anual según una cuota fija, por usuario o cargos de consumo por medio del [servicio de facturación medido](./partner-center-portal/saas-metered-billing.md). El marketplace comercial funciona según un modelo de agencia, en el que los editores establecen los precios y Microsoft factura a los clientes y paga los ingresos al editor, a la vez que retiene la cuota correspondiente a la agencia.

Este es un ejemplo de desglose de costos y pagos para demostrar el modelo de agencia. En este ejemplo, Microsoft factura USD 100 al cliente por su licencia de software y paga USD 80 al editor.

| Costo de licencia | 100 USD al mes |
| ------------ | ------------- |
| Costo de uso de Azure (D1/1 núcleo) | Facturado directamente al publicador, no al cliente |
| Microsoft factura al cliente | 100,00 USD al mes (el anunciante debe contar con los costos de paso a través o en que se incurran en el precio de la licencia) |
| **Microsoft factura** | **100 USD al mes** |
| Microsoft le paga el 80 % del costo de licencia<br>`*` En el caso de aplicaciones SaaS que cumplen los requisitos, Microsoft paga el 90 % del costo de la licencia| 80,00 USD al mes<br>``*`` 90,00 USD al mes |
|||

**`*` Tarifa reducida del servicio Marketplace**: en determinadas ofertas de SaaS que haya publicado en nuestro marketplace comercial, Microsoft reducirá su tarifa del servicio Marketplace del 20 % (como se describe en el Acuerdo de publicador de Microsoft) al 10 %. Para que su oferta sea apta, Microsoft debe haber designado sus ofertas como incentivadas para realizar una venta conjunta por IP de Azure. Para recibir el precio reducido del servicio Marketplace durante el mes, se deben cumplir los criterios de idoneidad al menos cinco (5) días hábiles antes del final de cada mes natural. Los honorarios del servicio Marketplace reducidos también se aplican a máquinas virtuales y aplicaciones administradas con incentivos para realizar una venta conjunta por IP de Azure, así como a cualquier otra oferta de IaaS comercializable cualificada que esté disponible a través del marketplace comercial.

## <a name="additional-sales-opportunities"></a>Oportunidades de venta adicionales

Puede optar por participar en los canales de marketing y ventas respaldados por Microsoft. Al crear la oferta en el Centro de partners, verá dos pestañas hacia el final del proceso:

- **Resell through CSPs** (Revender a través de los CSP): use esta opción para permitir que los asociados de los proveedores de soluciones en la nube (CSP) de Microsoft revendan la solución como parte de una oferta agrupada. Para más información, consulte [Programa del Proveedor de soluciones en la nube](cloud-solution-providers.md).

- **Venta conjunta con Microsoft**: esta opción permite que los equipos de ventas de Microsoft consideren la solución idónea para la venta conjunta de IP al evaluar las necesidades de los clientes. Consulte [Opción de venta conjunta en el Centro de partners](./partner-center-portal/commercial-marketplace-co-sell.md) para información detallada sobre cómo preparar la oferta para la evaluación.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una oferta de SaaS en el marketplace comercial](create-new-saas-offer.md)
- [Procedimientos recomendados para la publicación de ofertas](gtm-offer-listing-best-practices.md)
