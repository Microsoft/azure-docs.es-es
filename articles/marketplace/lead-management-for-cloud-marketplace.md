---
title: Administración de clientes potenciales del marketplace comercial | Azure Marketplace y AppSource
description: Información general sobre diversos temas relacionados con la publicación de ofertas y los artefactos técnicos en Azure Marketplace y AppSource
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 04/14/2020
ms.openlocfilehash: b08324654a046b5a2296f6e533107c1b26a0e1fc
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88606883"
---
# <a name="lead-management-for-the-commercial-marketplace"></a>Administración de clientes potenciales del marketplace comercial

Los clientes son el centro de cualquier negocio bien llevado. En la transformación de las adquisiciones de producto de hoy en día, los vendedores tienen que centrarse en conectar directamente con los clientes y generar una relación. Este es el motivo por el que generar clientes potenciales de alta calidad es una herramienta esencial para su ciclo de ventas. Tras publicar su oferta en [Cloud Partner Portal](https://partner.microsoft.com/), hay herramientas que le permiten recibir la información de contacto de clientes mediante programación inmediatamente después de que un cliente exprese interés o implemente el producto en Marketplace. 

## <a name="what-are-leads-in-the-marketplace"></a>¿Qué son los clientes potenciales en Marketplace?

Los clientes potenciales son los clientes que están interesados en sus productos de Marketplace o que están implementándolos desde allí. Sin importar si el producto está publicado en Azure Marketplace o en AppSource, podrá recibir clientes potenciales de los clientes una vez que la función esté configurada correctamente desde su CRM en las publicaciones de Centro de partners. 

## <a name="how-to-connect-your-crm-system-with-partner-center"></a>Conexión del sistema de CRM con Centro de partners

Para empezar a recibir clientes potenciales, el conector de administración de clientes potenciales en Centro de partners está diseñado para conectarse fácilmente con la información de CRM a una lista de sistemas de CRM disponibles. Ahora puede aprovechar fácilmente los clientes potenciales generados por Marketplace sin un importante esfuerzo de ingeniería por integrarse con un sistema externo.

Estas son las instrucciones paso a paso sobre cómo conectar cada uno de los destinos posibles de clientes potenciales:

**Dynamics CRM Online**: consulte [Configurar la administración de clientes potenciales para Dynamics 365 for Customer Engagement](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md) para obtener instrucciones sobre cómo configurar Dynamics CRM Online para obtener clientes potenciales.

**Marketo**: consulte [Configuración de la administración de clientes potenciales en Marketo](./partner-center-portal/commercial-marketplace-lead-management-instructions-marketo.md) para obtener instrucciones sobre cómo establecer la configuración de clientes potenciales de Marketo para obtener clientes potenciales.

**Salesforce**: consulte [Configuración de la administración de clientes potenciales en Salesforce](./partner-center-portal/commercial-marketplace-lead-management-instructions-salesforce.md) para obtener instrucciones sobre cómo configurar su instancia de Salesforce para obtener clientes potenciales.

**Tabla de Azure**: consulte [Configuración de la administración de clientes potenciales mediante una tabla de Azure](./partner-center-portal/commercial-marketplace-lead-management-instructions-azure-table.md) para obtener instrucciones sobre cómo configurar su cuenta de almacenamiento de Azure para obtener clientes potenciales en una tabla de Azure.

**Punto de conexión HTTPS**: consulte [Configuración de la administración de clientes potenciales mediante un punto de conexión HTTPS](./partner-center-portal/commercial-marketplace-lead-management-instructions-https.md) para obtener instrucciones sobre cómo configurar su punto de conexión HTTPS para obtener clientes potenciales.

Una vez que haya configurado correctamente el destino de clientes potenciales y haya presionado Publicar en su oferta, validaremos la conexión y le enviaremos un cliente potencial de prueba. Si está viendo la oferta antes de publicarla en Internet, también puede probar la conexión de los clientes potenciales tratando de adquirir la oferta en el entorno de versión preliminar. Es importante asegurarse de que la configuración de los clientes potenciales se mantenga actualizada para no perder ninguno de estos clientes; así pues, asegúrese de actualizar estas conexiones siempre que cambie algo por su parte.

### <a name="what-are-the-next-steps"></a>¿Cuáles son los siguientes pasos?

Una vez que esté implementada la configuración técnica, debería incorporar estos clientes potenciales a la estrategia actual de ventas y marketing y los procesos operativos. Estamos interesados en recibir una mejor perspectiva sobre el proceso de ventas global, y deseamos trabajar en estrecha colaboración con usted para conseguir clientes potenciales de alta calidad y una cantidad suficiente de datos que le permitan tener éxito. Agradecemos sus comentarios acerca de cómo podemos optimizar y mejorar los clientes potenciales que le enviamos con datos adicionales para obtener los mejores resultados de estos clientes. Díganos si está interesado en ofrecer comentarios y sugerencias para ayudar a que su equipo de ventas tenga mayor éxito con los clientes potenciales de Marketplace.

## <a name="common-lead-configuration-errors-during-publishing-in-partner-center"></a><a id="publishing-config-errors"></a> Errores comunes de configuración de clientes potenciales durante la publicación en Centro de partners

**No se pudo guardar el cliente potencial en Dynamics CRM. Compruebe la configuración de la cuenta de Dynamics CRM. LastCRMError: no se puede iniciar sesión en Dynamics CRM; LastCRMException:** 

> Si ha seleccionado la autenticación de O365, compruebe si la cuenta de usuario y la contraseña son válidas. Si ha seleccionado AAD, compruebe si el identificador del inquilino, el identificador de la aplicación y la clave secreta de la aplicación coinciden con lo que ha configurado en AAD. Siga las instrucciones que encontrará [aquí](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md). Si el nombre de usuario y la contraseña de la cuenta son válidos, asegúrese de que tiene acceso a Dynamics 365 y una licencia asignada (es decir, los pasos 11 al 15 si usa Azure Active Directory o la configuración de seguridad si usa un usuario de Office). 

**No se pudo guardar el cliente potencial en Dynamics CRM. El usuario no tiene permisos de creación para el atributo leadsourcecode en la entidad de cliente potencial.** 

> La aplicación o el usuario no tienen roles de seguridad para el sistema de escritura de clientes potenciales de Microsoft Marketplace. Siga los pasos 11 a 15 que se indican [aquí](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md) si usa Azure Active Directory o la configuración de seguridad si usa un usuario de Office.

**No se pudo guardar el cliente potencial en Dynamics CRM mediante AAD. Excepción: No se ha podido encontrar el inquilino. Esta instancia puede ocurrir si no hay suscripciones activas para el inquilino.**  

> El identificador de directorio proporcionado en la sección de administración de clientes potenciales no es un directorio válido. Obtenga el identificador de directorio basándose en las instrucciones que aparecen en el paso 2 (en Azure Active Directory, a partir de [aquí](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md)).

**No se pudo guardar el cliente potencial en Dynamics CRM. LastCRMError: error de SecLib::RetrievePrivilegeForUser: no hay ningún rol asignado al usuario.**  

> Resolución: asigne el rol de seguridad al sistema de escritura de clientes potenciales de Microsoft Marketplace. Siga las instrucciones que se proporcionan [aquí](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md) en la configuración de seguridad.

**No se pudo guardar el cliente potencial en Dynamics CRM mediante AAD. Excepción: No se encontró la aplicación con el identificador en el directorio.** 

> El identificador de aplicación proporcionado en la sección de administración de clientes potenciales no es un directorio válido. Obtenga el identificador de directorio basándose en las instrucciones que aparecen en el paso 8 (en Azure Active Directory, a partir de [aquí](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md)). 

**No se pudo guardar el cliente potencial en Dynamics CRM mediante AAD. Excepción: El identificador del inquilino solicitado no es válido y el formato de dominio externo no es válido** 

> El identificador de directorio proporcionado en la sección de administración de clientes potenciales no es un directorio válido. Obtenga el identificador de directorio basándose en las instrucciones que aparecen en el paso 2 (en Azure Active Directory, a partir de [aquí](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md)). 

**No se pudo guardar el cliente potencial en Dynamics CRM mediante AAD. Excepción: Error al validar las credenciales: se ha proporcionado un secreto de cliente no válido.** 

> Resolución: Inicie sesión en Azure Portal y compruebe que la clave de aplicación coincida con lo que hay en Centro de partners. Obtenga la contraseña basándose en las instrucciones que aparecen en el paso 10 (en Azure Active Directory, a partir de [aquí](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md)). 

**No se pudo guardar el cliente potencial en Dynamics CRM. LastCRMError: Se agotó el tiempo de espera del canal de solicitud mientras se esperaba una respuesta después de 00:02:00. Aumente el valor de tiempo de espera que se pasa a la llamada a la solicitud o aumente el valor de SendTimeout en el enlace. El tiempo asignado a esta operación puede haber sido una fracción de un tiempo de espera mayor.**  

> Resolución: Inicie sesión en Centro de partners, compruebe los detalles del escaparate electrónico >> Destino de clientes potenciales >> Dirección URL y compruebe que la instancia de Dynamic CRM sea válida.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**¿Qué son los clientes potenciales y por qué son importantes para mí como anunciante en Marketplace?** 

Los clientes potenciales son los clientes que están implementando sus productos desde Marketplace. Independientemente de que el producto aparezca en [Azure Marketplace](https://azuremarketplace.microsoft.com) o [AppSource](https://appsource.microsoft.com/), podrá recibir a los clientes potenciales que estén interesados en su producto si ha configurado el destino de clientes potenciales en su oferta.  

**¿Dónde puedo obtener ayuda para la configuración de mi destino de clientes potenciales?** 

Puede encontrar documentación en [Obtención de clientes potenciales](./partner-center-portal/commercial-marketplace-get-customer-leads.md) o enviar una incidencia de soporte técnico desde [Ayuda y soporte técnico](https://aka.ms/marketplacepublishersupport). Seleccione el tipo de oferta y la administración de clientes potenciales. 

**¿Debo configurar un destino de clientes potenciales para poder publicar una oferta en Marketplace?**

Sí, si va a publicar una aplicación SaaS de contacto o servicios de consultoría.  

**¿Cómo puedo confirmar que la configuración de cliente potencial es correcta?**

Después de configurar su oferta y el destino de clientes potenciales, publique su oferta. En el paso de validación de clientes potenciales, Marketplace enviará un cliente potencial de prueba al destino de clientes potenciales configurado en su oferta. 

**¿Cómo puedo encontrar al cliente potencial de prueba?**

Busque "MSFT_TEST" en el destino de clientes potenciales; a continuación, le mostramos los datos de un cliente potencial de prueba de ejemplo: 

```text
company = MSFT_TEST_636573304831318844 

country = US 

description = MSFT_TEST_636573304831318844 

email = MSFT_TEST_636573304831318844@test.com

encoding = UTF-8 

encoding = UTF-8 

first_name = MSFT_TEST_636573304831318844 

last_name = MSFT_TEST_636573304831318844 

lead_source = MSFT_TEST_636573304831318844-MSFT_TEST_636573304831318844|\<Offer Name> 

oid = 00Do0000000ZHog 

phone = 1234567890 

title = MSFT_TEST_636573304831318844 
```

**Tengo una oferta activa, pero no veo a los clientes potenciales.**

Cada cliente potencial tendrá datos que se pasen en campos al destino de clientes potenciales seleccionado; los clientes potenciales se incluirán en este formato: **Acción de origen | Oferta**

- *Orígenes:* :
  - AzureMarketplace
  - AzurePortal
  - TestDrive  
  - SPZA (acrónimo de AppSource)

- *Acciones:*
  - "INS": representa "instalación". Se genera en Azure Marketplace o AppSource cada vez que un cliente presiona el botón para adquirir su producto.
  - "PLT": representa "prueba dirigida por asociado". Se genera en AppSource cada vez que un cliente presiona el botón Ponerse en contacto conmigo.
  - "DNC": representa "no ponerse en contacto conmigo". Se genera en AppSource cada vez que se solicita a un socio al que se hace referencia cruzada en la página de la aplicación el establecimiento de contacto. En este momento, compartimos la advertencia de que este contacto tenía una referencia cruzada en la aplicación, pero no es necesario establecer contacto con él.
  - "Crear": esta acción aparece solo en Azure Portal y se genera cada vez que un cliente adquiere su oferta para su cuenta.
  - "StartTestDrive": esta acción está destinada solo a las versiones de prueba y se genera cada vez que un cliente inicia su versión de prueba.

- *Ofertas:*
  - "checkpoint.check-point-r77-10sg-byol",
  - "bitnami.openedxcypress",
  - "docusign.3701c77e-1cfa-4c56-91e6-3ed0b622145a"

*Estos son los datos de ejemplo de la información del cliente*

```json
{ 
"FirstName":"John",
"LastName":"Smith",
"Email":"jsmith@microsoft.com",
"Phone":"1234567890",
"Country":"US",
"Company":"Microsoft",
"Title":"CTO"
}
```

Obtenga más detalles en [Lead Info](./partner-center-portal/commercial-marketplace-get-customer-leads.md) (Información de cliente potencial). 

**He configurado el blob de Azure como destino de cliente potencial, ¿por qué no veo al cliente potencial?** 

El cliente potencial se escribe solo cuando se selecciona el almacenamiento de blobs de Azure como destino de clientes potenciales. Cambie a la tabla de Azure para recibir a los clientes potenciales en tiempo real.

**He recibido un correo electrónico desde Marketplace, ¿por qué no puedo encontrar al cliente potencial en mi instancia de CRM?**  

Es posible que el dominio de correo electrónico del usuario final sea .edu. Por motivos de privacidad, no se pasan los datos de información de identificación personal provenientes del dominio .edu. Envíe una incidencia de soporte técnico desde [Ayuda y soporte técnico](https://aka.ms/marketplacepublishersupport).

**He configurado la tabla o el blob de Azure como destino de cliente potencial, ¿cómo puedo ver a los clientes potenciales?** 

Puede tener acceso a la tabla o blob desde Azure Portal, o puede descargar e instalar el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) gratuitamente para ver las tablas o blobs de su cuenta de Azure Storage. 

**He configurado la tabla de Azure como destino del cliente potencial, ¿puedo recibir una notificación cada vez que Marketplace envíe un nuevo cliente potencial?** 

Sí, siga las instrucciones para configurar la combinación de función y tabla de Azure en la documentación que se proporciona [aquí](./partner-center-portal/commercial-marketplace-lead-management-instructions-azure-table.md). 

**He configurado Salesforce como destino de cliente potencial, ¿por qué no puedo encontrar a los clientes potenciales?**

Compruebe si el campo de web en el formulario para clientes potenciales es un campo obligatorio de tipo lista desplegable. En caso afirmativo, cámbielo por un campo de texto no obligatorio.  
 
**Hubo un problema con el destino de clientes potenciales y perdí algunos clientes potenciales. ¿Pueden enviármelos en un correo electrónico?** 

Debido a las directivas de privacidad, no podemos compartir la información de clientes potenciales a través de correo electrónico no protegido. 

**He configurado Azure Storage (blobs o tablas) como el destino de clientes potenciales, ¿cuánto va a costar?** 

Los datos de generación de clientes potenciales son pocos (menos de 1 GB para casi todos los anunciantes). El costo dependerá del número de clientes potenciales recibido; si se reciben 1000 clientes en un mes, el costo aproximado sería de 50 céntimos de USD. 
