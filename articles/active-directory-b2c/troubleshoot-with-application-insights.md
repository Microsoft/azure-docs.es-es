---
title: Solución de problemas de las directivas personalizadas con Application Insights
titleSuffix: Azure AD B2C
description: Configuración de Application Insights para realizar el seguimiento de la ejecución de las directivas personalizadas.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: troubleshooting
ms.date: 11/04/2019
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 9b5bc3f87296ea1af5de28178df6d8f27c965476
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87116081"
---
# <a name="collect-azure-active-directory-b2c-logs-with-application-insights"></a>Recopilación de registros de Azure Active Directory B2C con Application Insights

En este artículo se proporcionan los pasos para recopilar registros de Active Directory B2C (Azure AD B2C) de forma que pueda diagnosticar problemas con sus directivas personalizadas. Application Insights proporciona un modo de diagnosticar excepciones y visualizar problemas de rendimiento de la aplicación. Azure AD B2C incluye una característica para enviar datos a Application Insights.

Los registros de actividad detallados aquí **SOLO** deben estar habilitados durante el desarrollo de las directivas personalizadas.

> [!WARNING]
> No habilite el modo de desarrollo en producción. Los registros recopilan todas las notificaciones que se envían a los proveedores de identidad y se reciben de estos. Usted, como desarrollador, asume la responsabilidad de los datos personales recopilados en los registros de Application Insights. Estos registros detallados solo se recopilan cuando la directiva se coloca en **MODO DE DESARROLLADOR**.

## <a name="set-up-application-insights"></a>Configuración de Application Insights

Si aún no tiene una, cree una instancia de Application Insights en su suscripción.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene la suscripción de Azure (no el directorio de Azure AD B2C).
1. Seleccione **Crear un recurso** en el panel de navegación izquierdo.
1. Busque y seleccione **Application Insights** y, luego, seleccione **Crear**.
1. Complete el formulario, seleccione **Revisar y crear** y seleccione **Crear**.
1. Cuando la implementación se haya completado, seleccione **Ir al recurso**.
1. En **Configurar**, en el menú de Application Insights, seleccione **Propiedades**.
1. Registre la **CLAVE DE INSTRUMENTACIÓN** para su uso en un paso posterior.

## <a name="configure-the-custom-policy"></a>Configuración de la directiva personalizada

1. Abra el archivo de usuario de confianza (RP), por ejemplo *SignUpOrSignin.xml*.
1. Agregue los siguientes atributos al elemento `<TrustFrameworkPolicy>`:

   ```xml
   DeploymentMode="Development"
   UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights"
   ```

1. Si aún no existe, agregue un nodo secundario `<UserJourneyBehaviors>` al nodo `<RelyingParty>`. Tiene que estar situado inmediatamente después de `<DefaultUserJourney ReferenceId="UserJourney Id" from your extensions policy, or equivalent (for example:SignUpOrSigninWithAAD" />`.
1. Agregue el siguiente nodo como nodo secundario del elemento `<UserJourneyBehaviors>`. Asegúrese de reemplazar `{Your Application Insights Key}` por la **Clave de instrumentación** de Application Insights que registro anteriormente.

    ```xml
    <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="{Your Application Insights Key}" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
    ```

    * `DeveloperMode="true"` indica a Application Insights que acelere la telemetría a través de la canalización de procesamiento. Es bueno para el desarrollo, pero está restringido en volúmenes elevados.
    * `ClientEnabled="true"` envía el script del lado cliente de ApplicationInsights para realizar un seguimiento de la vista de página y de los errores del lado cliente. Puede verlos en la tabla **browserTimings** en el portal de Application Insights. Mediante el establecimiento de `ClientEnabled= "true"`, agrega Application Insights al script de la página y obtiene los intervalos de carga de página y de las llamadas AJAX, los recuentos, los detalles de las excepciones del explorador y los errores de AJAX, así como los recuentos de usuarios y sesiones. Este campo es **opcional** y está establecido en `false` de forma predeterminada.
    * `ServerEnabled="true"` envía el JSON UserJourneyRecorder como evento personalizado a Application Insights.

    Por ejemplo:

    ```xml
    <TrustFrameworkPolicy
      ...
      TenantId="fabrikamb2c.onmicrosoft.com"
      PolicyId="SignUpOrSignInWithAAD"
      DeploymentMode="Development"
      UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights"
    >
    ...
    <RelyingParty>
      <DefaultUserJourney ReferenceId="UserJourney ID from your extensions policy, or equivalent (for example: SignUpOrSigninWithAzureAD)" />
      <UserJourneyBehaviors>
        <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="{Your Application Insights Key}" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
      </UserJourneyBehaviors>
      ...
    </TrustFrameworkPolicy>
    ```

1. Cargue la directiva.

## <a name="see-the-logs-in-application-insights"></a>Visualización de registros en Application Insights

Hay un breve retraso, normalmente inferior a cinco minutos, antes de que pueda ver nuevos registros en Application Insights.

1. Abra el recurso de Application Insights que ha creado en [Azure Portal](https://portal.azure.com).
1. En el menú **Información general**, seleccione **Analytics**.
1. Abra una nueva pestaña en Application Insights.

A continuación se muestra una lista de consultas que puede usar para ver los registros:

| Consultar | Descripción |
|---------------------|--------------------|
`traces` | Ver todos los registros generados por Azure AD B2C |
`traces | where timestamp > ago(1d)` | Ver todos los registros generados por Azure AD B2C para el último día

Las entradas pueden ser largas. Exporte a un archivo CSV para realizar un examen más detallado.

Para más información sobre las consultas, vea [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

## <a name="next-steps"></a>Pasos siguientes

La Comunidad ha desarrollado un visor de recorrido de usuario para ayudar a los desarrolladores de identidad. Lee de la instancia de Application Insights y proporciona una vista bien estructurada de los eventos de recorrido del usuario. Proporciona el código fuente para implementarlo en su propia solución.

Microsoft no admite el reproductor del recorrido del usuario y está disponible estrictamente tal cual.

Aquí puede encontrar la versión del visor que lee los eventos de Application Insights en GitHub:

[Azure-Samples/active-directory-b2c-advanced-policies](https://github.com/Azure-Samples/active-directory-b2c-advanced-policies/tree/master/wingtipgamesb2c/src/WingTipUserJourneyPlayerWebApplication)
