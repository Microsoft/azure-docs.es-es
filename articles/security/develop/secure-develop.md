---
title: Desarrollo de aplicaciones seguras en Microsoft Azure
description: En este artículo se describen los procedimientos recomendados a tener en cuenta durante las fases de implementación y comprobación de su proyecto de aplicación web.
author: TerryLanfear
manager: barbkess
ms.author: terrylan
ms.date: 06/12/2019
ms.topic: article
ms.service: security
ms.subservice: security-develop
services: azure
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.openlocfilehash: 7818ae36c785311466d2fb26ce45dcf50983145d
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87283493"
---
# <a name="develop-secure-applications-on-azure"></a>Desarrollo de aplicaciones seguras en Azure
En este artículo se presentan las actividades de seguridad y los controles que debe tener en cuenta al desarrollar aplicaciones para la nube. Se abarcan los conceptos y preguntas de seguridad que se deben tener en cuenta durante las fases de implementación y comprobación del [ciclo de vida de desarrollo de seguridad (SDL)](https://msdn.microsoft.com/library/windows/desktop/84aed186-1d75-4366-8e61-8d258746bopq.aspx) de Microsoft. El objetivo es ayudarle a definir las actividades y los servicios de Azure que puede usar para desarrollar una aplicación más segura.

En este artículo se tratan las siguientes fases del SDL:

- Implementación
- Comprobación

## <a name="implementation"></a>Implementación
El asunto principal de la fase de implementación es establecer los procedimientos recomendados para la prevención temprana y detectar y quitar los problemas de seguridad del código.
Suponga que su aplicación se usará de maneras inesperadas. Este pensamiento le ayudará a protegerse contra el uso indebido (intencional o accidental) de la aplicación.

### <a name="perform-code-reviews"></a>Realización de revisiones de código

Antes de insertar el código en el repositorio, realice [revisiones de código](https://docs.microsoft.com/azure/devops/learn/devops-at-microsoft/code-reviews-not-primarily-finding-bugs) para aumentar la calidad general del código y reducir el riesgo de que se produzcan errores. Puede usar [Visual Studio](https://docs.microsoft.com/azure/devops/repos/tfvc/get-code-reviewed-vs?view=vsts) para administrar el proceso de revisión de código.

### <a name="perform-static-code-analysis"></a>Realización de análisis de código estático

El [análisis de código estático](https://owasp.org/www-community/controls/Static_Code_Analysis) (también conocido como *análisis de código fuente*) normalmente se realiza como parte de la revisión de código. El análisis de código estático se refiere normalmente a la ejecución de herramientas de análisis de código estático para encontrar posibles vulnerabilidades en el código que no se está ejecutando mediante técnicas como la [comprobación de objetos corruptos](https://en.wikipedia.org/wiki/Taint_checking) y el [análisis de flujo de datos](https://en.wikipedia.org/wiki/Data-flow_analysis).

Azure Marketplace ofrece [herramientas de desarrollo](https://azuremarketplace.microsoft.com/marketplace/apps/category/developer-tools?page=1&search=code%20review) que realizan análisis de código estático y le ayudan con las revisiones de código.

### <a name="validate-and-sanitize-every-input-for-your-application"></a>Validación y saneamiento de todas las entradas de la aplicación

Trate todas las entradas como no confiable para proteger la aplicación de las vulnerabilidades más comunes en las aplicaciones web. Los datos que no son de confianza son un vehículo para los ataques por inyección. Las entradas para su aplicación incluyen parámetros en la dirección URL, entradas del usuario, datos de la base de datos o de una API, y todos los valores que se pasan a la aplicación que un usuario tiene la capacidad de manipular. Una aplicación debe [validar](https://owasp.org/www-project-proactive-controls/v3/en/c5-validate-inputs) que los datos son sintáctica y semánticamente válidos antes de que la aplicación use los datos de alguna forma (como mostrarlos de vuelta al usuario).

Valide las entradas al principio del flujo de datos para asegurarse de que solo los datos con un formato correcto entren en el flujo de trabajo. No querrá que los datos con formato incorrecto se conserven en la base de datos o desencadenen un error de funcionamiento en un componente de nivel inferior.

La incorporación de listas negras y blancas son dos enfoques generales validar la sintaxis de las entradas:

  - Con las listas negras se intenta comprobar que una entrada de usuario concreta no incluya contenido "malintencionado conocido".

  - Con las listas blancas se intenta comprobar que una entrada de usuario concreta coincida con un conjunto de entradas "válidas conocidas". La creación de listas blancas basada en caracteres es un tipo de lista blanca en que una aplicación comprueba si la entrada del usuario solo contiene caracteres "válidos conocidos" o si la entrada coincide con un formato conocido.
    Por ejemplo, esto puede implicar la comprobación de si un nombre de usuario contiene solo caracteres alfanuméricos o si contiene exactamente dos números.

La creación de listas blancas es el método preferido para la creación de software seguro.
Las listas negras son propensas a errores, ya que es imposible pensar en una lista completa de entradas potencialmente erróneas.

Estas tareas deben realizarse en el servidor, no en el lado cliente (o en el servidor y en el lado cliente).

### <a name="verify-your-applications-outputs"></a>Verificación de las salidas de la aplicación

Cualquier salida que muestre visualmente o dentro de un documento siempre debe codificarse e incluir caracteres de escape. El [proceso de escape](https://owasp.org/www-community/Injection_Theory#Escaping_.28aka_Output_Encoding.29), también conocido como *codificación de salida*, se usa para ayudar a garantizar que los datos que no son de confianza no sean un vehículo para los ataques por inyección de código. El proceso de escape, combinado con la validación de datos, proporciona defensas por niveles para aumentar la seguridad del sistema como un todo.

El proceso de escape garantiza que todo se muestra como *salida.* El proceso de escape también permite que el intérprete sepa que los datos no está diseñados para ejecutarse, y esto impide que los ataques funcionen. Es otra técnica de ataque común denominada *ataque de scripts entre sitios*.

Si usa un marco web de terceros, puede comprobar las opciones de codificación de salida en los sitios web mediante la [hoja de referencia rápida de OWASP para prevención de ataques de scripts entre sitios](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.md).

### <a name="use-parameterized-queries-when-you-contact-the-database"></a>Uso de consultas con parámetros al comunicarse con la base de datos

Nunca debe crear una consulta de base de datos en línea "sobre la marcha" dentro del código para enviarse directamente a la base de datos. El código malintencionado que se inserte en la aplicación podría ocasionar que alguien robe, borre o modifique su base de datos. La aplicación también se puede usar para ejecutar comandos malintencionados de sistema operativo en el sistema que hospeda a su base de datos.

En su lugar, utilice consultas con parámetros o procedimientos almacenados. Cuando use consultas con parámetros, puede invocar el procedimiento desde el código de forma segura y pasarle una cadena sin preocuparse de que se procese como parte de la instrucción de consulta.

### <a name="remove-standard-server-headers"></a>Eliminación de encabezados de servidor estándar

Los encabezados como Server, X-Powered-By y X-AspNet-Version revelan información sobre el servidor y las tecnologías subyacentes. Se recomienda que elimine estos encabezados para evitar la creación de huellas digitales en la aplicación.
Consulte [Removing standard server headers on Azure websites](https://azure.microsoft.com/blog/removing-standard-server-headers-on-windows-azure-web-sites/) (Eliminación de encabezados de servidor estándar de los sitios web de Azure).

### <a name="segregate-your-production-data"></a>Separación de los datos de producción

Los datos de producción o datos "reales" no se deben usar para el desarrollo, las pruebas o cualquier otro fin distinto al establecido por la empresa. Se debe usar un conjunto de datos enmascarado ([anonimizado](https://en.wikipedia.org/wiki/Data_anonymization)) para todo el proceso de desarrollo y pruebas.

Esto significa que menos personas tienen acceso a los datos reales, lo que reduce la superficie expuesta a ataques. También significa que menos empleados ven datos personales, lo que elimina posibles vulneraciones de seguridad en la confidencialidad.

### <a name="implement-a-strong-password-policy"></a>Implementación de una directiva de contraseñas seguras

Para defenderse contra ataques por fuerza bruta y adivinación por diccionario, debe implementar una directiva de contraseñas seguras que garantice que los usuarios crean contraseñas complejas (por ejemplo, con una longitud mínima de 12 caracteres, que requiera caracteres alfanuméricos y especiales).

Puede usar un marco de identidad para crear y aplicar directivas de contraseñas. Azure Active Directory B2C le ayuda con la administración de contraseñas al incluir [directivas integradas](../../active-directory-b2c/tutorial-create-user-flows.md#create-a-password-reset-user-flow), [autoservicio de restablecimiento de contraseñas](../../active-directory-b2c/user-flow-self-service-password-reset.md) y mucho más.

Para defenderse contra ataques en cuentas predeterminadas, compruebe que todas las claves y contraseñas sean reemplazables y que se generen o reemplacen después de instalar recursos.

Si la aplicación debe generar contraseñas automáticamente, asegúrese de que las contraseñas generadas sean aleatorias y tengan una alta entropía.

### <a name="validate-file-uploads"></a>Validación de cargas de archivos

Si la aplicación permite [cargar archivos](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload), considere la posibilidad de tomar medidas para esta actividad de riesgo. El primer paso en muchos ataques es insertar código malintencionado en un sistema que está sufriendo un ataque. El uso de la carga de archivos ayuda al atacante a llevarlo a cabo. OWASP ofrece soluciones para validar un archivo a fin de garantizar que el archivo que va a cargar es seguro.

La protección antimalware ayuda a identificar y eliminar virus, spyware y otro software malintencionado. Puede instalar [Microsoft Antimalware](../fundamentals/antimalware.md) o una solución de protección de puntos de conexión de un asociado de Microsoft ([Trend Micro](https://www.trendmicro.com/azure/), [Broadcom](https://www.broadcom.com/products), [McAfee](https://www.mcafee.com/us/products.aspx), [Windows Defender](https://docs.microsoft.com/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-in-windows-10) y [Endpoint Protection](https://docs.microsoft.com/configmgr/protect/deploy-use/endpoint-protection)).

[Microsoft Antimalware](../fundamentals/antimalware.md) incluye características como la protección en tiempo real, los análisis programados, la corrección de malware, las actualizaciones de firmas, las actualizaciones del motor, los ejemplos de informes y la colección de eventos de exclusión. Puede integrar soluciones de asociados y Microsoft Antimalware con [Azure Security Center](../../security-center/security-center-partner-integration.md) para facilitar la implementación y la integración de detecciones (alertas e incidentes).

### <a name="dont-cache-sensitive-content"></a>No almacenar en caché el contenido confidencial

No almacene en la caché del explorador el contenido confidencial. Los exploradores pueden almacenar información para la memoria caché y el historial. Los archivos almacenados en caché se guardan en una carpeta, como la carpeta Archivos temporales de Internet en el caso de Internet Explorer. Si se vuelve a hacer referencia a estas páginas, el explorador las muestra desde su memoria caché. Si se muestra información confidencial al usuario (como su dirección, datos de la tarjeta de crédito, número del seguro social o nombre de usuario), es posible que quede almacenada en la memoria caché del explorador y se pueda volver a recuperar mediante un examen de la memoria caché del explorador o simplemente al presionar el botón **Atrás** del explorador.

## <a name="verification"></a>Comprobación
La fase de comprobación supone un esfuerzo integral para asegurarse de que el código cumple los principios de seguridad y privacidad que se establecieron en las fases anteriores.

### <a name="find-and-fix-vulnerabilities-in-your-application-dependencies"></a>Búsqueda y corrección de vulnerabilidades en las dependencias de la aplicación

Analice la aplicación y sus bibliotecas dependientes para identificar los componentes vulnerables conocidos. Entre los productos que están disponibles para realizar este análisis se incluyen [OWASP Dependency Check](https://www.owasp.org/index.php/OWASP_Dependency_Check),[Snyk](https://snyk.io/) y [Black Duck](https://www.blackducksoftware.com/).

El examen de vulnerabilidades con tecnología de [Tinfoil Security](https://www.tinfoilsecurity.com/) está disponible para Azure App Service Web Apps. [El examen de Tinfoil Security a través de App Service](https://azure.microsoft.com/blog/web-vulnerability-scanning-for-azure-app-service-powered-by-tinfoil-security/) ofrece a los desarrolladores y administradores un medio rápido, integrado y económico para detectar y resolver vulnerabilidades antes de que un individuo malintencionado pueda sacar provecho de ellas.

> [!NOTE]
> También puede [integrar Tinfoil Security con Azure AD](../../active-directory/saas-apps/tinfoil-security-tutorial.md). La integración de Tinfoil Security con Azure AD proporciona las siguientes ventajas:
>  - En Azure AD, puede controlar quién tiene acceso a Tinfoil Security.
>  - Los usuarios pueden iniciar sesión automáticamente en Tinfoil Security (inicio de sesión único) con sus cuentas de Azure AD.
>  - Puede administrar sus cuentas en una única ubicación central: Azure Portal.

### <a name="test-your-application-in-an-operating-state"></a>Prueba de la aplicación en estado operativo

Las pruebas de seguridad de la aplicación dinámicas (DAST) son un proceso para probar una aplicación en un estado operativo a fin de encontrar vulnerabilidades de seguridad. Las herramientas DAST analizan los programas mientras se ejecutan para buscar vulnerabilidades de seguridad como daños en la memoria, configuraciones de servidor no seguras, ataques de script entre sitios, problemas de privilegios de usuario, inyección de código SQL y otros problemas de seguridad críticos.

Las pruebas DAST son distintas de las pruebas de seguridad de la aplicación estáticas (SAST). Las herramientas SAST analizan código fuente o versiones compiladas del código cuando no se está ejecutando para encontrar defectos de seguridad.

Realice pruebas DAST preferentemente con la ayuda de un profesional de seguridad (un [evaluador de penetración](../fundamentals/pen-testing.md) o asesor de vulnerabilidades). Si no está disponible un profesional de seguridad, puede realizar usted mismo una prueba DAST con un escáner de proxy web y algo de entrenamiento. Conecte un escáner DAST desde el principio para asegurarse de que no inserta problemas de seguridad obvios en el código. Consulte el sitio de [OWASP](https://owasp.org/www-community/Vulnerability_Scanning_Tools) para obtener una lista de escáneres de vulnerabilidad para aplicaciones web.

### <a name="perform-fuzz-testing"></a>Realización de pruebas de vulnerabilidades

En las [pruebas de vulnerabilidades](https://cloudblogs.microsoft.com/microsoftsecure/2007/09/20/fuzz-testing-at-microsoft-and-the-triage-process/), se induce un error de programa al introducir deliberadamente datos aleatorios o con formato incorrecto a una aplicación. La inducción de errores en los programas ayudan a revelar los posibles problemas de seguridad antes del lanzamiento de la aplicación.

[Security Risk Detection](https://www.microsoft.com/en-us/security-risk-detection/) es el servicio de pruebas de vulnerabilidades exclusivo de Microsoft para buscar errores críticos para la seguridad en el software.

### <a name="conduct-attack-surface-review"></a>Realice una revisión de la superficie expuesta a ataques

La revisión de la superficie expuesta a ataques después de completar el código ayuda a garantizar que se hayan tomado en cuenta los cambios al diseño o implementación de una aplicación o sistema. Ayuda a garantizar que los nuevos vectores de ataque que se crearon como resultado de los cambios, incluidos los modelos de amenazas, se revisen y mitiguen.

Puede crear una imagen de la superficie expuesta a ataques si realiza un examen de la aplicación. Microsoft ofrece una herramienta de análisis de la superficie expuesta a ataques denominada [Analizador de superficie expuesta a ataques](https://www.microsoft.com/download/details.aspx?id=24487). Puede elegir entre muchas herramientas y servicios de análisis dinámicos comerciales para pruebas y vulnerabilidades, incluidas[OWASP Zed Attack Proxy Project](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project), [Arachni](http://arachni-scanner.com/), [Skipfish](https://code.google.com/p/skipfish/) y [w3af](http://w3af.sourceforge.net/). Estas herramientas de examen rastrean la aplicación y asignan los elementos de la aplicación que son accesibles a través de la web. También puede buscar en Azure Marketplace otras [herramientas de desarrollo](https://azuremarketplace.microsoft.com/marketplace/apps/category/developer-tools?page=1) similares.

### <a name="perform-security-penetration-testing"></a>Realizar pruebas de penetración de seguridad

Asegurarse de que su aplicación es segura es tan importante como probar cualquier otra funcionalidad. Convierta las [pruebas de penetración](../fundamentals/pen-testing.md) en una parte estándar del proceso de compilación e implementación. Programe pruebas de seguridad y exámenes de vulnerabilidades periódicos en las aplicaciones, y supervise si hay puertos abiertos, ataques y puntos de conexión.

### <a name="run-security-verification-tests"></a>Ejecución de pruebas de comprobación de la seguridad

[Secure DevOps Kit para Azure](https://azsk.azurewebsites.net/index.html) (AzSK) contiene SVT para varios servicios de la plataforma Azure. Ejecute estas SVT periódicamente para asegurarse de que su suscripción a Azure y los distintos recursos que componen la aplicación están en un estado seguro. También puede automatizar estas pruebas mediante el uso de la característica de extensiones de implementación continua e integración continua (CI/CD) de AzSK, lo que hace que las SVT estén disponibles como una extensión de Visual Studio.

## <a name="next-steps"></a>Pasos siguientes
En los siguientes artículos, se recomiendan controles de seguridad y actividades que pueden ayudarle a diseñar e implementar aplicaciones seguras.

- [Diseño de aplicaciones seguras](secure-design.md)
- [Implementación de aplicaciones seguras](secure-deploy.md)
