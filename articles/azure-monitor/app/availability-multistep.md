---
title: 'Supervisión con pruebas web de varios pasos: Azure Application Insights'
description: Configuración de pruebas web de varios pasos para supervisar las aplicaciones web con Azure Application Insights
ms.topic: conceptual
ms.date: 05/26/2020
ms.openlocfilehash: 2df8b7450423c901665090608da83f68b43b30e5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87024783"
---
# <a name="multi-step-web-tests"></a>Pruebas web de varios pasos

Puede supervisar una secuencia registrada de direcciones URL e interacciones con un sitio web a través de pruebas web de varios pasos. En este artículo se proporciona una guía paso a paso del proceso de creación de una prueba web de varios pasos con Visual Studio Enterprise.

> [!NOTE]
> Las pruebas web de varios pasos dependen de los archivos de pruebas web de Visual Studio. Se [anunció](https://devblogs.microsoft.com/devops/cloud-based-load-testing-service-eol/) que Visual Studio 2019 será la última versión con la funcionalidad de prueba web. Es importante comprender que, aunque no se agregarán nuevas características, la funcionalidad de pruebas web de Visual Studio 2019 todavía se admite y se seguirá admitiendo durante el ciclo de vida de soporte técnico del producto. El equipo de productos de Azure Monitor ha abordado las preguntas sobre el futuro de las pruebas de disponibilidad de varios pasos [aquí](https://github.com/MicrosoftDocs/azure-docs/issues/26050#issuecomment-468814101).  
> </br>
> **No se admiten** las pruebas web de varios pasos en la nube de[ Azure Government](../../azure-government/index.yml).


## <a name="pre-requisites"></a>Requisitos previos

* Visual Studio 2017 Enterprise posterior.
* Herramientas de rendimiento web y pruebas de carga de Visual Studio.

Para encontrar el requisito de las herramientas de prueba. Inicie el **Instalador de Visual Studio** > **Componentes individuales** > **Depuración y pruebas** > **Herramientas de rendimiento web y pruebas de carga**.

![Captura de pantalla de la interfaz de usuario del instalador de Visual Studio con los componentes individuales seleccionados con una casilla situada junto al elemento de las herramientas de rendimiento Web y pruebas de carga](./media/availability-multistep/web-performance-load-testing.png)

> [!NOTE]
> Las pruebas web de varios pasos tienen costes adicionales asociados. Para más información, consulte la [guía oficial de precios](https://azure.microsoft.com/pricing/details/application-insights/).

## <a name="record-a-multi-step-web-test"></a>Registro de una prueba web de varios pasos 

> [!WARNING]
> Ya no se recomienda usar la grabadora de varios pasos. La grabadora se desarrolló para páginas HTML estáticas con interacciones básicas y no proporciona una experiencia funcional para páginas web modernas.

Para obtener instrucciones sobre cómo crear pruebas web de Visual Studio, consulte la [documentación oficial de Visual Studio 2019](/visualstudio/test/how-to-create-a-web-service-test?view=vs-2019).

## <a name="upload-the-web-test"></a>Carga de la prueba web

1. En el portal de Application Insights, en el panel Disponibilidad, seleccione **Crear prueba** > **Tipo de prueba** > **Prueba web de varios pasos**.

2. Establezca las ubicaciones de prueba, la frecuencia y los parámetros de alerta.

### <a name="frequency--location"></a>Frecuencia y ubicación

|Configuración| Explicación
|----|----|----|
|**Frecuencia de prueba**| establece la frecuencia con que se ejecuta la prueba desde cada ubicación de prueba. Con una frecuencia predeterminada de cinco minutos y cinco ubicaciones de prueba, el sitio se prueba, de media, cada minuto.|
|**Ubicaciones de prueba**| Son los lugares desde donde nuestros servidores envían solicitudes web a la dirección URL. **El número mínimo de ubicaciones de prueba recomendadas es cinco** con el fin de asegurarse de que puede distinguir los problemas del sitio web de los problemas de la red. Puede seleccionar hasta 16 ubicaciones.

### <a name="success-criteria"></a>Criterios de éxito

|Configuración| Explicación
|----|----|----|
| **Tiempo de espera de prueba** |reduzca este valor para recibir una alerta sobre las respuestas lentas. La prueba se considera un error si no se han recibido respuestas de su sitio dentro de este período. Si seleccionó **Analizar solicitudes dependientes**, todas las imágenes, archivos de estilo, scripts y otros recursos dependientes se deben haber recibido durante este período.|
| **Respuesta HTTP** | el código de estado devuelto que se considera correcto. 200 es el código que indica que se ha devuelto una página web normal.|
| **Coincidencia de contenido** | Una cadena, como "Bienvenido". Probamos que se produce una coincidencia exacta entre mayúsculas y minúsculas en todas las respuestas. Debe ser una cadena sin formato, sin caracteres comodín. No se olvide de que si el contenido cambia, es posible que tenga que actualizarla. **En la coincidencia de contenido solo se admiten caracteres en inglés** |

### <a name="alerts"></a>Alertas

|Configuración| Explicación
|----|----|----|
|**Casi en tiempo real (versión preliminar)** | Se recomienda usar alertas casi en tiempo real. La configuración de este tipo de alertas se realiza después de crear la prueba de disponibilidad.  |
|**Clásico** | Ya no se recomienda usar alertas clásicas para nuevas pruebas de disponibilidad.|
|**Umbral de la ubicación de la alerta**|se recomienda un mínimo de 3/5 ubicaciones. La relación óptima entre el umbral de ubicación de la alerta y el número de ubicaciones de prueba es **umbral de ubicación de la alerta** = **número de ubicaciones de prueba - 2, con un mínimo de cinco ubicaciones de prueba.**|

## <a name="configuration"></a>Configuración

### <a name="plugging-time-and-random-numbers-into-your-test"></a>Inserción de números de tiempo y aleatorios en la prueba

Suponga que está probando una herramienta que obtiene datos que dependen del tiempo, como acciones de una fuente externa. Cuando se graba la prueba web, debe utilizar horas específicas, pero las establece como parámetros de la prueba, StartTime y EndTime.

![Captura de pantalla de mi fantástica aplicación de cotizaciones](./media/availability-multistep/app-insights-72webtest-parameters.png)

Al ejecutar la prueba, le gustaría que EndTime fuera siempre la hora actual y que StartTime fuera 15 minutos menos.

El complemento de hora y fecha de prueba web proporciona la manera de controlar los tiempos de parametrización.

1. Agregue un complemento de prueba de web a cada valor variable que desee. En la barra de herramientas de pruebas web, elija **Agregar complemento de prueba web**.
    
    ![Adición del complemento de prueba web](./media/availability-multistep/app-insights-72webtest-plugin-name.png)
    
    En este ejemplo, usamos dos instancias del complemento Date Time Plug-in. Una instancia es para "hace 15 minutos" y otra para "ahora".

2. Abra las propiedades de cada complemento. Asígnele un nombre y configúrelo para utilizar la hora actual. En cada uno de ellos, establezca Añadir minutos = -15.

    ![Parámetros de contexto](./media/availability-multistep/app-insights-72webtest-plugin-parameters.png)

3. En los parámetros de prueba en la web, utilice {{plug-in name}} para hacer referencia al nombre de un complemento.

    ![StartTime](./media/availability-multistep/app-insights-72webtest-plugins.png)

Ahora puede cargar la prueba en el portal. Utilizará los valores dinámicos en cada ejecución de la prueba.

### <a name="dealing-with-sign-in"></a>Tratamiento del inicio de sesión

Si los usuarios inician sesión en la aplicación, tiene varias opciones para simular el inicio de sesión a fin de poder probar páginas detrás del inicio de sesión. El enfoque que utilice dependerá del tipo de seguridad proporcionada por la aplicación.

En todos los casos, debe crear una cuenta en su aplicación para realizar pruebas. Si es posible, restrinja los permisos de esta cuenta de prueba para que sea totalmente imposible que las pruebas web afecten a usuarios reales.

**Nombre de usuario y la contraseña simples**: grabar una prueba web de la manera habitual. Elimine las cookies en primer lugar.

**Autenticación SAML**

|Nombre de propiedad| Descripción|
|----|-----|
| URI de audiencia | El URI de audiencia para el token SAML.  Se trata del URI para Access Control Service (ACS), incluido el nombre de host y el espacio de nombres de ACS. |
| Contraseña de certificado | La contraseña del certificado de cliente que concederá acceso a la clave privada insertada. |
| Certificado de cliente  | El valor del certificado de cliente con una clave privada en formato codificado en Base64. |
| Identificador de nombre | El identificador de nombre del token. |
| No después de | El intervalo de tiempo durante el cual será válido el token.  El valor predeterminado es 5 minutos. |
| No antes de | El intervalo de tiempo durante el cual será válido un token creado en el pasado (para abordar el desfase horario).  El valor predeterminado es (negativo) 5 minutos. |
| Nombre de parámetro de contexto de destino | El parámetro de contexto que recibirá la aserción generada. |


**Secreto del cliente**: si la aplicación tiene una ruta de inicio de sesión que implica un secreto de cliente, úsela. Azure Active Directory (AAD) es un ejemplo de un servicio que proporciona inicio de sesión del secreto de cliente. En AAD, el secreto de cliente es la clave de la aplicación.

Esta es una prueba web de ejemplo de una aplicación web de Azure que usa una clave de aplicación:

![Captura de pantalla de ejemplo](./media/availability-multistep/client-secret.png)

Obtener el token de AAD mediante el secreto de cliente (AppKey).
Extraer el token de portador de la respuesta.
Llamar a la API mediante el token de portador del encabezado de la autorización.
Asegúrese de que la prueba web sea un cliente real (es decir, tiene su propia aplicación en AAD) y use su clientId + appkey. El servicio que se prueba también tiene su propia aplicación en AAD: el identificador URI de esta aplicación, appID, se refleja en la prueba web en el campo "resource".

### <a name="open-authentication"></a>Autenticación abierta
Un ejemplo de autenticación abierta es el iniciar sesión con una cuenta de Microsoft o Google. Muchas aplicaciones que utilizan OAuth proporcionan la alternativa del secreto de cliente, por lo que la primera táctica sería investigar dicha posibilidad.

Si la prueba debe iniciar sesión con OAuth, el enfoque general es el siguiente:

Utilice una herramienta como Fiddler para examinar el tráfico entre el explorador web, el sitio de autenticación y la aplicación.
Realice dos o más inicios de sesión mediante distintas máquinas o exploradores o en intervalos largos de tiempo (para permitir que los tokens expiren).
Mediante la comparación de diferentes sesiones, identifique el token pasado desde el sitio de autenticación que, a continuación, se pasa al servidor de aplicaciones después de iniciar sesión.
Grabe una prueba web con Visual Studio.
Parametrice los tokens, estableciendo el parámetro cuando se devuelve el token desde el autenticador y utilizándolo en la consulta al sitio. (Visual Studio intenta parametrizar la prueba pero no parametriza correctamente los tokens).

## <a name="troubleshooting"></a>Solución de problemas

[Artículo de solución de problemas](troubleshoot-availability.md) dedicado.

## <a name="next-steps"></a>Pasos siguientes

* [Alertas de disponibilidad](availability-alerts.md)
* [Pruebas web de ping de URL](monitor-web-app-availability.md)
