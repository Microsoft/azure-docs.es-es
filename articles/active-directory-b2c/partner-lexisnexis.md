---
title: Tutorial para configurar Azure Active Directory B2C con LexisNexis
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo integrar la autenticación de Azure AD B2C con LexisNexis, que es un servicio de validación de identidades y generación de perfiles que se usa para comprobar la identificación del usuario y proporcionar evaluaciones de riesgo completas basadas en el dispositivo del usuario.
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/22/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 6f5b6bfdb523a22fc4dd9593bfec556da7493aa9
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87371199"
---
# <a name="tutorial-for-configuring-lexisnexis-with-azure-active-directory-b2c"></a>Tutorial para configurar LexisNexis con Azure Active Directory B2C

En este tutorial de ejemplo, se proporcionan instrucciones sobre cómo integrar Azure AD B2C con [LexisNexis](https://risk.lexisnexis.com/products/threatmetrix/?utm_source=bingads&utm_medium=ppc&utm_campaign=SEM%7CLNRS%7CUS%7CEN%7CTMX%7CBR%7CBing&utm_term=threat%20metrix&utm_network=o&utm_device=c&msclkid=1e85e32ec18c1ae9bbc1bc2998e026bd). LexisNexis proporciona una variedad de soluciones, que puede encontrar [aquí](https://risk.lexisnexis.com/products/threatmetrix/?utm_source=bingads&utm_medium=ppc&utm_campaign=SEM%7CLNRS%7CUS%7CEN%7CTMX%7CBR%7CBing&utm_term=threat%20metrix&utm_network=o&utm_device=c&msclkid=1e85e32ec18c1ae9bbc1bc2998e026bd). En este tutorial de ejemplo, trataremos la solución de **ThreatMetrix** de LexisNexis. ThreatMetrix es un servicio de validación de identidades y generación de perfiles. Se usa para comprobar la identificación del usuario y proporcionar evaluaciones de riesgo completas basadas en el dispositivo del usuario.

Esta integración realiza la generación de perfiles basándose en algunos fragmentos de información de usuario, que proporciona el usuario durante el flujo de registro. ThreatMetrix determina si se debe permitir que el usuario continúe con el inicio de sesión o no. Se consideran los atributos siguientes en el análisis de riesgos de ThreatMetrix:

- Email
- Número de teléfono
- Información de generación de perfiles recopilada desde el equipo del usuario

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- [Un inquilino de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-create-tenant) vinculado a la suscripción de Azure.

## <a name="scenario-description"></a>Descripción del escenario

La integración de ThreatMetrix incluye los siguientes componentes:

- Azure AD B2C: el servidor de autorización, responsable de comprobar las credenciales del usuario, al que también se le conoce como proveedor de identidades.

- ThreatMetrix: el servicio ThreatMetrix toma las entradas proporcionadas por el usuario y las combina con la información de generación de perfiles recopilada desde la máquina del usuario para comprobar la seguridad de la interacción del usuario.

- API REST personalizada: esta API implementa la integración entre Azure AD B2C y el servicio ThreatMetrix.

En el siguiente diagrama de arquitectura se muestra la implementación.

![Captura de pantalla del diagrama de arquitectura de LexisNexis](media/partner-lexisnexis/lexisnexis-architecture-diagram.png)

|Paso | Descripción |
|:--------------|:-------------|
|1. | El usuario llega a una página de inicio de sesión. El usuario opta por iniciar sesión para crear una nueva cuenta y escribir información en la página. Azure AD B2C recopila los atributos de usuario.
| 2. | Azure AD B2C llama a la API de nivel intermedio y pasa los atributos de usuario.
| 3. | La API de nivel intermedio recopila atributos de usuario y los transforma en un formato que la API de LexisNexis puede consumir. A continuación, los envía a LexisNexis.  
| 4. | LexisNexis consume la información y la procesa para validar la identificación del usuario en función del análisis de riesgos. A continuación, devuelve el resultado a la API de nivel intermedio.
| 5. | La API de nivel intermedio procesa la información y envía la información pertinente de nuevo a Azure AD B2C.
| 6. | Azure AD B2C recibe información de la API de nivel intermedio. Si aparece la respuesta Error, se muestra un mensaje de error al usuario. Si aparece la respuesta Correcto, el usuario se autentica y se le concede acceso.

## <a name="onboard-with-lexisnexis"></a>Incorporación con LexisNexis

1. Para crear una cuenta de LexisNexis, póngase en contacto con [LexisNexis](https://risk.lexisnexis.com/products/threatmetrix/?utm_source=bingads&utm_medium=ppc&utm_campaign=SEM%7CLNRS%7CUS%7CEN%7CTMX%7CBR%7CBing&utm_term=threat%20metrix&utm_network=o&utm_device=c&msclkid=1e85e32ec18c1ae9bbc1bc2998e026bd).

2. Cree una directiva de LexisNexis que cumpla sus requisitos. Use la documentación que está disponible [aquí](https://risk.lexisnexis.com/products/threatmetrix/?utm_source=bingads&utm_medium=ppc&utm_campaign=SEM%7CLNRS%7CUS%7CEN%7CTMX%7CBR%7CBing&utm_term=threat%20metrix&utm_network=o&utm_device=c&msclkid=1e85e32ec18c1ae9bbc1bc2998e026bd).

>[!NOTE]
> El nombre de la directiva se usará más adelante.

Una vez creada la cuenta, recibirá la información que necesita para la configuración de la API. El proceso se describe en las secciones siguientes.

## <a name="configure-azure-ad-b2c-with-lexisnexis"></a>Configuración de Azure AD B2C con LexisNexis

### <a name="part-1---deploy-the-api"></a>Parte 1: implementación de la API

Implemente el código de la API proporcionado en un servicio de Azure. El código se puede publicar desde Visual Studio siguiendo estas [instrucciones](https://docs.microsoft.com/visualstudio/deployment/quickstart-deploy-to-azure?view=vs-2019).

>[!NOTE]
>Necesitará la dirección URL del servicio implementado para configurar Azure AD con la configuración necesaria.

### <a name="part-2---configure-the-api"></a>Parte 2: configuración de la API

La configuración de la aplicación se puede [configurar en App Service en Azure](https://docs.microsoft.com/azure/app-service/configure-common#configure-app-settings).  Con este método, los valores se pueden configurar de forma segura sin registrarlos en un repositorio. Debe especificar la configuración siguiente para la API de REST:

| Configuración de la aplicación | Source | Notas |
| :-------- | :------------| :-----------|
|ThreatMetrix: Url | Configuración de la cuenta de ThreatMetrix |     |
|ThreatMetrix:OrgId | Configuración de la cuenta de ThreatMetrix |     |
|ThreatMetrix:ApiKey |Configuración de la cuenta de ThreatMetrix|  |
|ThreatMetrix: Directiva | Nombre de la directiva creada en ThreatMetrix | |
| BasicAuth:ApiUsername |Defina un nombre de usuario para la API| El nombre de usuario se usará en la configuración de Azure AD B2C.
| BasicAuth:ApiPassword | Defina una contraseña para la API | La contraseña se usará en la configuración de Azure AD B2C.

### <a name="part-3---deploy-the-ui"></a>Parte 3: implementación de la interfaz de usuario

Esta solución utiliza plantillas de interfaz de usuario personalizadas que se cargan mediante Azure AD B2C. Estas plantillas de interfaz de usuario realizan la generación de perfiles que se envía directamente al servicio ThreatMetrix.

Consulte estas [instrucciones](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-ui-customization#custom-page-content-walkthrough) para implementar los archivos de interfaz de usuario incluidos en una cuenta de almacenamiento de blobs. Las instrucciones incluyen la configuración de una cuenta de almacenamiento de blobs, la configuración de CORS y la habilitación del acceso público.

La interfaz de usuario se basa en la plantilla de página azul océano. Todos los vínculos dentro de la interfaz de usuario deben actualizarse para hacer referencia a la ubicación implementada. En la carpeta de la interfaz de usuario, busque y reemplace https://yourblobstorage/blobcontainer por la ubicación implementada.

### <a name="part-4---create-api-policy-keys"></a>Parte 4: creación de claves de directiva de API

Consulte este [documento](https://docs.microsoft.com/azure/active-directory-b2c/secure-rest-api#add-rest-api-username-and-password-policy-keys) y cree dos claves de directiva: una para el nombre de usuario de la API y otra para la contraseña de la API que definió anteriormente.

La directiva de ejemplo utiliza estos nombres de clave:

- B2C_1A_RestApiUsername

- B2C_1A_RestApiPassword

### <a name="part-5---update-the-api-url"></a>Parte 5: actualización de la URL de API

En la directiva de TrustFrameworkExtensions proporcionada, busque el perfil técnico llamado `Rest-LexisNexus-SessionQuery` y actualice el elemento de metadatos `ServiceUrl` con la ubicación de la API implementada anteriormente.

### <a name="part-6---update-ui-url"></a>Parte 6: actualización de la URL de interfaz de usuario

En la directiva de TrustFrameworkExtensions proporcionada, realice la acción de buscar y reemplazar para buscar https://yourblobstorage/blobcontainer/ con la ubicación en la que se implementaron los archivos de interfaz de usuario.

### <a name="part-7---configure-the-azure-ad-b2c-policy"></a>Parte 7: configuración de la directiva de Azure AD B2C

Consulte este [documento](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications#custom-policy-starter-pack) para descargar el [paquete de inicio de LocalAccounts](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts) y configure la directiva para el inquilino de Azure AD B2C.

>[!NOTE]
>Actualice las directivas proporcionadas para que se relacionen con su inquilino específico.

>[!NOTE]
> Como procedimiento recomendado, se sugiere que los clientes agreguen una notificación de consentimiento en la página de colección de atributos. Notifique a los usuarios que la información se enviará a servicios de terceros para la verificación de identidad.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Abra el inquilino de Azure AD B2C y, en Directivas, seleccione **Flujos de usuario**.

2. Seleccione el **flujo de usuario** que creó anteriormente.

3. Seleccione **Ejecutar flujo de usuario** y elija la configuración:

   a. **Aplicación**: seleccione la aplicación registrada (el ejemplo es JWT).

   b. **URL de respuesta**: seleccione la **dirección URL de redireccionamiento**.

   c. Seleccione **Ejecutar flujo de usuario**.

4. Recorra el flujo de registro y cree una cuenta.

5. Cierre sesión.

6. Recorra el flujo de inicio de sesión.  

7. El rompecabezas ThreatMetrix aparecerá después de escribir **continuar**.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-overview)

- [Introducción a las directivas personalizadas en Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications)
