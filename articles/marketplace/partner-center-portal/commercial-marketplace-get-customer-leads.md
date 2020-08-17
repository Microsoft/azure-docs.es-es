---
title: Administración de clientes potenciales en el marketplace comercial de Microsoft
description: Aprenda a generar y recibir clientes potenciales a partir de sus ofertas de Microsoft AppSource y Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: keferna
ms.author: keferna
ms.date: 03/30/2020
ms.openlocfilehash: 9ee433f226b37c8ffd6ad466cca7cbd844d53524
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86535984"
---
# <a name="customer-leads-from-your-commercial-marketplace-offer"></a>Clientes potenciales a partir de la oferta en el marketplace comercial

Los clientes potenciales son clientes interesados en sus ofertas o que las implementan desde [Microsoft AppSource](https://appsource.microsoft.com) y [Azure Marketplace](https://azuremarketplace.microsoft.com). Puede recibir clientes potenciales después de publicar la oferta en el marketplace comercial. En este artículo se explican los siguientes conceptos relacionados con la administración de clientes potenciales:

* El modo en que su oferta de marketplace comercial genera clientes potenciales para garantizar que no pierde oportunidades de negocio. 
* Cómo conectar su sistema de administración de relaciones con clientes (CRM) a su oferta para que pueda administrar los clientes potenciales en una ubicación central.
* Los datos de clientes potenciales que le enviamos para que pueda hacer un seguimiento de los clientes que se comunican con usted.

## <a name="generate-customer-leads"></a>Generar clientes potenciales

Estos son los lugares en los que se genera un cliente potencial:

- Un cliente acepta compartir su información tras seleccionar **Ponerse en contacto conmigo** en el marketplace comercial. Este es un cliente potencial de *interés inicial*. Compartimos información con usted sobre el cliente que ha expresado interés en obtener su producto. El cliente potencial se encuentra en la parte superior del embudo que desemboca en la adquisición.

    ![Contacto de Dynamics 365](./media/commercial-marketplace-get-customer-leads/dynamics-365-contact-me.png)

- Un cliente selecciona **Obténgalo ahora** (o selecciona **Crear** en [Azure Portal](https://portal.azure.com/)) para conseguir la oferta. Este es un cliente potencial *activo*. Compartimos información con usted sobre el cliente que ha empezado a implementar su producto.

    ![Botón Obténgalo ahora de SQL](./media/commercial-marketplace-get-customer-leads/sql-get-it-now.png)

    ![Botón Crear de Windows Server](./media/commercial-marketplace-get-customer-leads/windows-server-create.png)

- Un cliente selecciona **Versión de prueba** o **Prueba gratuita** para probar su oferta. Las versiones de prueba o evaluaciones gratuitas representan vías rápidas para compartir su negocio al instante con clientes potenciales sin barreras de entrada.

    ![Botón Versión de prueba de Dynamics 365](./media/commercial-marketplace-get-customer-leads/dynamics-365-test-drive.png)

    ![Botón Prueba gratuita de Dynamics 365](./media/commercial-marketplace-get-customer-leads/dynamics-365-free-trial.png)

## <a name="connect-to-your-crm-system"></a>Conectarse a su sistema CRM

[!INCLUDE [Test drive content](./includes/connect-lead-management.md)]

## <a name="understand-lead-data"></a>Descripción de los datos de clientes potenciales

Todos los clientes potenciales que recibe durante el proceso de adquisición de clientes tienen datos en campos específicos. El primer campo donde buscar es `LeadSource`, que sigue este formato: **Origen-Acción** | **Oferta**.

**Orígenes**: El valor de este campo se rellena en función del Marketplace que generó el cliente potencial. Los valores posibles son `"AzureMarketplace"`, `"AzurePortal"` y `"AppSource (SPZA)"`.

**Acciones**: El valor de este campo se rellena en función de la acción realizada por el cliente en el marketplace que generó el cliente potencial.

Los valores posibles son:

- **"INS"** : significa *instalación*. Esta acción se encuentra en Azure Marketplace o AppSource cuando un cliente adquiere su producto.
- **"PLT"** : significa *prueba dirigida por asociado*. Esta acción se encuentra en AppSource cuando un cliente selecciona la opción **Ponerse en contacto conmigo**.
- **"DNC"** : significa *no ponerse en contacto conmigo*. Esta acción se encuentra en AppSource cuando se solicita a un asociado al que se hace referencia cruzada en la página de la aplicación el establecimiento de contacto. Compartimos una notifican que indica que este cliente tenía una referencia cruzada en la aplicación, pero no es necesario establecer contacto con él.
- **"Create"** : esta acción aparece solo en Azure Portal y se genera cada vez que un cliente adquiere su oferta para su cuenta.
- **"StartTestDrive"** : esta acción solo está destinada a la opción **Versión de prueba** y se genera cuando un cliente inicia su versión de prueba.

**Ofertas**: Es posible que tenga varias ofertas en el marketplace comercial. El valor de este campo se rellena en función de la oferta que generó el cliente potencial. El identificador del publicador y el de la oferta se envían en este campo y son los valores que usted proporcionó al publicar la oferta en el marketplace.

A continuación se muestran los valores en el formato esperado `publisherid.offerid`: 

- `checkpoint.check-point-r77-10sg-byol`
- `bitnami.openedxcypress`
- `docusign.3701c77e-1cfa-4c56-91e6-3ed0b622145`

## <a name="customer-information"></a>Información del usuario

La información del cliente se envía a través de varios campos. En el ejemplo siguiente se muestra la información del cliente que se encuentra en un cliente potencial:

- Nombre: John
- Apellidos: Smith
- Correo electrónico: jsmith\@microsoft.com
- Teléfono: 1234567890
- País: US
- Compañía: Microsoft
- Título: CTO

>[!NOTE]
>No todos los datos del ejemplo anterior están siempre disponibles para cada cliente potencial. Puesto que obtendrá clientes potenciales a partir de distintos pasos, tal como se ha mencionado en la sección "Generar clientes potenciales", la mejor manera de administrarlos es desduplicar los registros y personalizar los seguimientos. De este modo, cada cliente obtiene un mensaje adecuado y se entabla una relación única.

## <a name="best-practices-for-lead-management"></a>Procedimientos recomendados para la administración de clientes potenciales

Aquí le presentamos algunas recomendaciones para generar clientes potenciales a lo largo del ciclo de ventas:

- **Proceso**: defina un proceso de ventas claro, con hitos, análisis y una clara propiedad del equipo.
- **Calificación**: defina los requisitos previos, que indican si un cliente potencial se ha calificado por completo. Asegúrese de que los representantes de ventas o marketing califiquen a los clientes potenciales con cuidado antes de hacerlos pasar por el proceso de ventas completo.
- **Seguimiento**: no olvide hacer un seguimiento en 24 horas. Incluya al cliente potencial en la herramienta CRM de su elección inmediatamente después de que el cliente implemente la versión de prueba. Envíele un correo electrónico mientras aún tenga el recuerdo reciente. Solicite la programación de una llamada telefónica para comprender mejor si el producto es una buena solución para su problema. Cabe esperar que la transacción típica requiera muchas llamadas de seguimiento.
- **Consolidación**: consolide a los clientes potenciales para encaminarse hacia un aumento del margen de ganancias. Póngase en contacto con ellos, pero no los bombardee. Le recomendamos que les envíe un correo electrónico varias veces antes de cerrarlos: no abandone al primer intento. Recuerde que estos clientes conocieron directamente el producto y emplearon un tiempo en una evaluación gratuita. Son posibles interesados.

## <a name="common-questions-about-lead-management"></a>Preguntas habituales sobre la administración de clientes potenciales

### <a name="where-can-i-get-help-in-setting-up-my-lead-destination"></a>¿Dónde puedo obtener ayuda para la configuración de mi destino de clientes potenciales?

Siga los pasos descritos en la sección [Conectarse a su sistema CRM](#connect-to-your-crm-system) o envíe una incidencia de soporte técnico desde [Ayuda y soporte técnico del Centro de partners](https://aka.ms/marketplacepublishersupport). A continuación, seleccione **Creación de ofertas** > **El tipo de oferta** > **Configuración de administración de clientes potenciales**.

### <a name="am-i-required-to-configure-a-lead-destination-in-order-to-publish-an-offer-in-the-commercial-marketplace"></a>¿Tengo que configurar un destino de clientes potenciales para poder publicar una oferta en el marketplace comercial?

La respuesta depende del tipo de oferta que publique. El software como servicio (SaaS) y Dynamics 365 Customer Engagement usan **Contacto** para mostrar todas las ofertas de Dynamics 365 Finance y Operations, todas las ofertas de Dynamics 365 Business Central y todas las ofertas de servicio de consultoría. Como resultado, requieren una conexión a un destino de cliente potencial. Si el tipo de oferta no aparece en la lista, no se requiere ninguna conexión a un destino de cliente potencial. Es recomendable configurar un destino de clientes potenciales para no perder oportunidades de negocio.

### <a name="how-can-i-find-the-test-lead"></a>¿Cómo puedo encontrar al cliente potencial de prueba?

Busque `"MSFT_TEST"` en el destino del cliente potencial. A continuación se incluye un cliente potencial de muestra de Microsoft. Tenga en cuenta que el formato del cliente potencial varía en función del destino del cliente potencial.

```
{
    "UserDetails": {
      "FirstName": "MSFT_TEST_636573304831318844",
      "LastName": "MSFT_TEST_636573304831318844",
      "Email": "MSFT_TEST_636573304831318844@test.com",
      "Phone": "1234567890",
      "Country": "US",
      "Company": "MSFT_TEST_636573304831318844",
      "Title": "MSFT_TEST_636573304831318844"
    },
    "LeadSource": "AzureMarketplace",
    "ActionCode": "INS",
    "OfferTitle": "Contoso Test"
    "Description": "MSFT_TEST_636573304831318844"
}
```

### <a name="i-have-a-live-offer-but-why-am-i-not-seeing-any-leads"></a>Tengo una oferta activa. ¿Por qué no veo ningún cliente potencial?

Asegúrese de que la conexión con el destino de clientes potenciales sea válida. Le enviaremos un cliente potencial de muestra después de seleccionar **Publicar** en la oferta en el Centro de partners. Si ve el cliente potencial de prueba, la conexión es válida. También puede probar la conexión de clientes potenciales intentando obtener la vista previa de la oferta durante el paso de vista previa. Seleccione **Obténgalo ahora**, **Contacto** o **Prueba gratuita** en la lista del marketplace comercial.

Además, asegúrese de buscar los datos correctos. El contenido de la sección [Descripción de los datos de clientes potenciales](#understand-lead-data) de este artículo describe los datos de clientes potenciales que se envían al destino del cliente potencial.

### <a name="i-configured-azure-blob-storage-as-my-lead-destination-but-why-dont-i-see-the-lead"></a>He configurado Azure Blob Storage como destino de mis clientes potenciales. ¿Por qué no veo el cliente potencial?

Azure Blob Storage ya no se admite como destino de clientes potenciales, por lo que le faltan los clientes potenciales que haya generado su oferta. Cambie a cualquiera de las otras [opciones de destino de clientes potenciales](./commercial-marketplace-get-customer-leads.md). 

### <a name="i-received-an-email-from-the-commercial-marketplace-but-why-cant-i-find-the-lead-in-my-crm"></a>He recibido un correo electrónico del marketplace comercial. ¿Por qué no encuentro al cliente potencial en mi CRM?

Es posible que el dominio de correo electrónico del usuario final sea .edu. Por motivos de privacidad, no se pasa información personal procedente del dominio .edu. Envíe una incidencia de soporte técnico mediante la [Ayuda y soporte técnico del Centro de partners](https://aka.ms/marketplacepublishersupport).

### <a name="i-configured-an-azure-table-as-my-lead-destination-how-can-i-view-the-leads"></a>He configurado una tabla de Azure como destino de mi cliente potencial. ¿Cómo puedo ver los clientes potenciales?

Puede tener acceso a los datos de clientes potenciales almacenados en la tabla de Azure desde Azure Portal. También puede descargar e instalar el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) gratuitamente para ver los datos de las tablas de la cuenta de Azure Storage.

### <a name="i-configured-an-azure-table-as-my-lead-destination-can-i-get-notified-whenever-a-new-commercial-marketplace-lead-is-sent"></a>He configurado una tabla de Azure como destino de mi cliente potencial. ¿Puedo recibir una notificación cuando se envíe un nuevo cliente potencial del marketplace comercial?

Sí. Siga las instrucciones de [Configuración de la administración de clientes potenciales mediante una tabla de Azure](./commercial-marketplace-lead-management-instructions-azure-table.md) para configurar un flujo de Microsoft que envía un correo electrónico si se agrega un cliente potencial a la tabla de Azure.

### <a name="i-configured-salesforce-as-my-lead-destination-but-why-cant-i-find-the-leads"></a>He configurado Salesforce como destino de clientes potenciales. ¿Por qué no los encuentro?

Compruebe si el formulario de web a cliente potencial es un campo obligatorio en función de la lista desplegable. En caso afirmativo, cámbielo por un campo de texto no obligatorio.

### <a name="there-was-an-issue-with-my-lead-destination-and-i-missed-some-leads-can-i-have-them-sent-to-me-in-an-email"></a>Hubo un problema con el destino de clientes potenciales y perdí algunos clientes potenciales. ¿Pueden enviármelos en un correo electrónico?

Debido a las directivas de información personal, no podemos compartir la información de clientes potenciales mediante correo electrónico no protegido.

### <a name="i-configured-an-azure-table-as-my-lead-destination-how-much-will-it-cost"></a>He configurado una tabla de Azure como destino de mi cliente potencial. ¿Cuánto costará?

Hay pocos datos de generación de clientes potenciales. Es menor que 1 GB para casi todos los editores. El costo depende del número de clientes potenciales recibidos. Por ejemplo, si se reciben 1000 clientes potenciales en un mes, el costo es de unos 50 céntimos. Para obtener más información sobre los precios de almacenamiento, consulte [Información general sobre los precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/).

Si no se responde a su pregunta, póngase en contacto con el Soporte técnico de Microsoft desde la [Ayuda y soporte técnico del Centro de Partners](https://aka.ms/marketplacepublishersupport). A continuación, seleccione **Creación de ofertas** > **El tipo de oferta** > **Configuración de administración de clientes potenciales**.

### <a name="im-receiving-email-notifications-when-new-customer-leads-are-received-how-can-i-configure-someone-else-to-receive-these-emails"></a>Recibo notificaciones por correo electrónico cuando se reciben nuevos clientes potenciales. ¿Cómo puedo configurar a otra persona para que reciba estos mensajes de correo?

Acceda a la oferta en el Centro de partners y vaya a la página **Configuración de la oferta** > **Administración de clientes potenciales** > **Editar**. Actualice las direcciones de correo electrónico en el campo **Correo electrónico de contacto**.

## <a name="next-steps"></a>Pasos siguientes

Una vez implementada la configuración técnica, incorpore estos clientes potenciales a la estrategia actual de ventas y marketing, así como a los procesos operativos. Estamos interesados en comprender mejor el proceso de ventas global, y queremos trabajar en estrecha colaboración con usted para conseguir clientes potenciales de alta calidad y una cantidad suficiente de datos que le permitan tener éxito. Agradecemos sus comentarios acerca de cómo podemos optimizar y mejorar los clientes potenciales que le enviamos con datos adicionales para obtener los mejores resultados de estos clientes. Díganos si está interesado en [ofrecer comentarios](mailto:AzureMarketOnboard@microsoft.com) y sugerencias para ayudar a que su equipo de ventas tenga mayor éxito con los clientes potenciales del marketplace comercial.
