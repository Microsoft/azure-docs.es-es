---
title: 'Asistentes de voz en Windows: guías de diseño'
titleSuffix: Azure Cognitive Services
description: Guías de procedimientos recomendados para diseñar una experiencia de agente de voz.
services: cognitive-services
author: cfogg6
manager: spencer.king
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/1/2020
ms.author: adamwa
ms.openlocfilehash: 948fbcd57514f4ef77483b05c60324e867e0e3ed
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89293649"
---
# <a name="design-assistant-experiences-for-windows-10"></a>Diseño del uso del asistente en Windows 10

Los asistentes de voz desarrollados en Windows 10 deben implementar las siguientes guías de experiencia del usuario para que la experiencia de la activación por voz en Windows 10 sea la mejor posible. En este documento se guiará a los desarrolladores para que conozcan el trabajo necesario para que un asistente de voz se integre con el shell de Windows 10.

## <a name="contents"></a>Contenido

- [Resumen de las vistas de activación por voz que se admiten en Windows 10](#summary-of-voice-activation-views-supported-in-windows-10)
- [Resumen de los requisitos](#requirements-summary)
- [Procedimientos recomendados para mejorar la escucha](#best-practices-for-good-listening-experiences)
- [Guía de diseño para la activación por voz en la aplicación](#design-guidance-for-in-app-voice-activation)
- [Guía de diseño para la activación por voz con el dispositivo bloqueado](#design-guidance-for-voice-activation-above-lock)
- [Guía de diseño de la vista previa de la activación por voz](#design-guidance-for-voice-activation-preview)

## <a name="summary-of-voice-activation-views-supported-in-windows-10"></a>Resumen de las vistas de activación por voz que se admiten en Windows 10

Windows 10 infiere una experiencia de activación para el contexto del usuario que se basa en el contexto del dispositivo. En la siguiente tabla de resumen, se proporciona información general de las distintas vistas disponibles cuando la pantalla está encendida.

| Vista (disponibilidad) | Contexto del dispositivo | Objetivo del usuario | Aparece cuando | Necesidades de diseño |
| --- | --- | --- | --- | --- |
| **En la aplicación (19H1)** | Dispositivo sin bloquear, el asistente tiene el foco. | Interactuar con la aplicación del asistente. | El asistente procesa la solicitud en la aplicación. | Experiencia principal de escucha de la vista en la aplicación. |
| **Con el dispositivo bloqueado (19H2)** | Con el dispositivo bloqueado, sin autenticar. | Interactuar con el asistente, pero a distancia. | El sistema está bloqueado y el asistente solicita su activación. | Objetos visuales en pantalla completa para la interfaz de usuario de campo lejano. Implementar directivas de descarte para no desbloquear el bloqueo. |
| **Vista previa de la activación por voz (20H1)** | Con el dispositivo sin bloquear, el asistente no tiene el foco. | Interactuar con el asistente, pero de forma menos intrusiva. | El sistema está sin bloquear y el asistente solicita la activación en segundo plano. | Lienzo mínimo. Cambiar el tamaño o entregar a la vista principal de la aplicación según sea necesario. |

## <a name="requirements-summary"></a>Resumen de los requisitos

Se requiere un esfuerzo mínimo para acceder a las distintas experiencias. No obstante, los asistentes necesitan implementar la guía de diseño correcta para cada vista. La tabla siguiente es una lista en la que encontrará los requisitos que deben cumplirse.

| **Vista de la activación por voz** | **Resumen de los requisitos del asistente** |
| --- | --- |
| **En la aplicación** | <ul><li>Procesar la solicitud en la aplicación.</li><li>Proporciona indicadores en la interfaz de usuario para los distintos estados de la escucha.</li><li>La interfaz de usuario se adapta a medida que cambia el tamaño de las ventanas.</li></ul> |
| **Con el dispositivo bloqueado** | <ul><li>Detectar el estado de bloqueo y solicitar la activación.</li><li>No proporcionar una experiencia de usuario que sea siempre persistente, ya que bloquearía el acceso a la pantalla de bloqueo de Windows.</li><li>Proporcionar objetos visuales a pantalla completa y una experiencia en la que la voz va primero.</li><li>Tener en cuenta la siguiente guía para el descarte.</li><li>Observar las siguientes consideraciones acerca de la privacidad y seguridad.</li></ul> |
| **Vista previa de la activación por voz** | <ul><li>Detectar el estado de desbloqueo y solicitar la activación en segundo plano.</li><li>Dibujar una experiencia de usuario de escucha mínima en el panel de vista previa.</li><li>Dibujar una X de cierre en la parte superior derecha y descartar automáticamente el audio del streaming y detenerlo al presionarla.</li><li>Cambie el tamaño o la entrega a la vista principal de la aplicación del asistente cuando sea necesario para proporcionar respuestas.</li></ul> |

## <a name="best-practices-for-good-listening-experiences"></a>Procedimientos recomendados para mejorar la escucha.

Los asistentes deben crear una experiencia de escucha que proporcione comentarios críticos para que el usuario pueda conocer el estado del asistente. A continuación, se muestran algunos de los posibles estados que hay que tener en cuenta al crear una experiencia de asistente. Estas son solo algunas sugerencias posibles, no es una guía que haya seguir forzosamente.

- El asistente está disponible para la entrada de voz.
- El asistente está en proceso de activación (una palabra clave o presionar el botón del micrófono).
- El asistente está transmitiendo activamente audio a la nube del asistente.
- El asistente está listo para que el usuario empiece a hablar.
- El ayudante oye que se están diciendo palabras.
- El asistente entiende que el usuario ha terminado de hablar.
- El asistente está procesando y preparando una respuesta.
- El ayudante está respondiendo.

Aunque los estados cambien rápidamente, merece la pena considerar la posibilidad de usar la experiencia del usuario para los mismos, ya que las duraciones varían en todo el ecosistema de Windows. Una parte de la solución pueden ser tanto una indicación visual como sonidos de campanillas, también llamados &quot;avisos sonoros&quot;. Del mismo modo, un conjunto de tarjetas visuales y descripciones sonoras es una buena opción de respuesta.

## <a name="design-guidance-for-in-app-voice-activation"></a>Guía de diseño para la activación por voz en la aplicación

Cuando la aplicación del asistente tiene el foco, es indudable que la intención del usuario es interactuar con ella, por lo que es la vista principal de la aplicación la que debe controlar todas las experiencias de activación por voz, Vista cuyo tamaño puede cambiar el usuario. Para ayudar a explicar las interacciones del shell del asistente, en el resto de este documento se usa el ejemplo concreto de un asistente de servicios financieros llamado Contoso. Tanto en este diagrama como en los siguientes, lo que el usuario diga aparecerá a la izquierda en los bocadillos habituales de los tebeos, mientras que las respuestas del asistente se verán a la derecha.

**Vista en la aplicación. Estado inicial cuando comienza la activación por voz:** 
![Captura de pantalla del asistente por voz en Windows antes de la activación](media/voice-assistants/windows_voice_assistant/initial_state.png)

**Vista en la aplicación. Después de la activación por voz correcta, comienza la experiencia de escucha:** ![Captura de pantalla del asistente de voz en Windows mientras el asistente de voz está escuchando](media/voice-assistants/windows_voice_assistant/listening.png)

**Vista en la aplicación. Todas las respuestas permanecen en la experiencia de la aplicación.** ![Captura de pantalla del asistente por voz en Windows cuando el asistente responde](media/voice-assistants/windows_voice_assistant/response.png)

## <a name="design-guidance-for-voice-activation-above-lock"></a>Guía de diseño para la activación por voz con el dispositivo bloqueado

Disponible con 19H2; los asistentes basados en la plataforma de activación por voz de Windows están disponibles para responder con el dispositivo bloqueado.

### <a name="customer-opt-in"></a>Participación del usuario

La activación de voz con el dispositivo bloqueado está siempre deshabilitada de forma predeterminada. Para poder usarla, los usuarios deben dirigirse a la configuración de Windows > Privacidad > Activación por voz. Para más información sobre la supervisión y la solicitud de esta opción de configuración, consulte la [guía de implementación de con el dispositivo bloqueado](windows-voice-assistants-implementation-guide.md#detecting-above-lock-activation-user-preference).

### <a name="not-a-lock-screen-replacement"></a>No es un reemplazo de la pantalla de bloqueo

Aunque las notificaciones u otros puntos de integración de la pantalla de bloqueo de la aplicación estándar siguen estando disponibles para el asistente, la pantalla de bloqueo de Windows siempre define la experiencia inicial del usuario, hasta que se produce una activación por voz. Una vez que se detecta la activación por voz, la aplicación del asistente aparece temporalmente por encima de la pantalla de bloqueo. Para evitar la confusión del cliente, cuando la aplicación del asistente se activa con el dispositivo bloqueado, no debe presentar nunca una interfaz de usuario para solicitar ningún tipo de credenciales o inicio de sesión.

![Captura de pantalla de una pantalla de bloqueo de Windows](media/voice-assistants/windows_voice_assistant/lock_screen1.png)

### <a name="above-lock-experience-following-voice-activation"></a>Experiencia con el dispositivo bloqueado tras la activación por voz

Cuando la pantalla está encendida, la aplicación del asistente está en pantalla completa, sin una barra de título encima de la pantalla de bloqueo. Los objetos visuales de mayor tamaño una mayor potencia en las descripciones de voz y la interfaz de voz principal permiten escenarios en los que el usuario esté demasiado lejos para leer la interfaz de usuario o tenga las manos ocupadas en otra tarea (que no tenga ninguna relación con el equipo).

Cuando la pantalla está apagada, la aplicación del asistente puede reproducir un aviso sonoro para indicar que el asistente se está activando y proporcionar una experiencia en la que se use solo la voz.

![Captura de pantalla del asistente por voz con el dispositivo bloqueado](media/voice-assistants/windows_voice_assistant/above_lock_listening.png)

### <a name="dismissal-policies"></a>Directivas de descarte

El asistente debe implementar la guía para el descarte en esta sección para facilitar el inicio de sesión de los usuarios la próxima vez que quieran usar su PC con Windows. A continuación, se muestran varios requisitos específicos que el asistente debe implementar:

- **Todos los lienzos del asistente que se muestran con el dispositivo bloqueado deben contener una X** en la parte superior derecha para descartar el asistente.
- **Si se presiona cualquier tecla, también se debe descartar la aplicación del asistente**. La entrada mediante teclado es la señal de una aplicación de bloqueo tradicional de que el cliente quiere iniciar sesión. Por lo tanto, ni las entradas de texto ni las que se realicen mediante el teclado deberían dirigirse a la aplicación. En su lugar, la aplicación deberá descartarse automáticamente si se detecta alguna entrada mediante el teclado, para que el usuario pueda iniciar sesión fácilmente en su dispositivo.
- **Si la pantalla se apaga, la aplicación debe descartarse automáticamente.** Así se garantiza que la próxima vez que el usuario use su equipo, la pantalla de inicio de sesión estará lista.
- Si la aplicación está &quot;en uso&quot;, puede continuar con el dispositivo bloqueado. &quot;En uso&quot; significa que se produce cualquier entrada o salida. Por ejemplo, se puede seguir transmitiendo música o vídeo aunque la aplicación esté bloqueada. Se permite usar tanto &quot;Continuar&quot; como otros pasos de diálogo de varios turnos para que la aplicación funcione con el dispositivo bloqueado.
- Los **detalles de la implementación al descartar la aplicación** se pueden encontrar [en la guía de la implementación con la aplicación bloqueada](windows-voice-assistants-implementation-guide.md#closing-the-application).

![Captura de pantalla del asistente por voz en Windows antes de la activación](media/voice-assistants/windows_voice_assistant/above_lock_response.png)

![Captura de pantalla del asistente por voz en Windows antes de la activación](media/voice-assistants/windows_voice_assistant/lock_screen2.png)

### <a name="privacy-amp-security-considerations-above-lock"></a>Consideraciones acerca de la privacidad y seguridad con el dispositivo bloqueado

Muchos equipos son portables, por lo que el usuario no siempre los tiene a mano. Puede que el equipo se haya quedado en la habitación del hotel, en el asiento del avión o en áreas de trabajo a las que otras personas tengan acceso físico. Si los asistentes que se activan con el dispositivo bloqueado no están preparados, este puede sufrir un tipo de ataque llamado &quot;[Evil Maid](https://en.wikipedia.org/wiki/Evil_maid_attack)&quot;.

Por consiguiente, los asistentes deben seguir las instrucciones de esta sección por seguridad. La interacción con el dispositivo bloqueado se produce cuando el usuario de Windows no ha realizado la autenticación. Esto significa que, en general, **la entrada en el asistente también debería tratarse como no autenticada**.

- Los asistentes deberán **implementar una lista de aptitudes permitidas para identificar aquellas que a las que sea seguro** acceder con el dispositivo bloqueado.
- Las tecnologías de identificación del altavoz pueden jugar un papel importante para reducir ciertos riesgos, pero la identificación del altavoz no es un sustituto apropiado de la autenticación de Windows.
- La lista de aptitudes permitidas debería incluir tres clases de acciones o aptitudes:

| **Clase de acción** | **Descripción** | **Ejemplos (esta lista no está completa)** |
| --- | --- | --- |
| Protección sin autenticación | Información de uso general o control y comandos básicos de la aplicación. | &quot;¿Qué hora es?&quot;, &quot;Reproducir la siguiente pista&quot; |
| Protección con identificación de altavoz | Riesgo de suplantación, se revela información personal. | &quot;¿Con quién es mi próxima cita?&quot;, &quot;Revisar la lista de la compra&quot;, &quot;Responder a la llamada&quot; |
| Protección solo después de la autenticación de Windows | Acciones de riesgo elevado que los atacantes podrían usar para causar daños al usuario. | &quot;Ir al supermercado&quot;, &quot;Eliminar mi cita (importante)&quot;, &quot;Enviar un mensaje de texto (agresivo)&quot;, &quot;Lanzar una página web (execrable)&quot; |

En el caso de Contoso, la información general en torno a los títulos de valores está segura sin autenticación. Es probable que la información específica del usuario, como el número de acciones que se poseen, esté segura con la identificación de altavoz. Sin embargo, no se debería permitir la compra o venta de títulos sin la autenticación de Windows.

Para aumentar la seguridad, **Windows siempre bloqueará tanto los vínculos web como otros inicios entre aplicaciones hasta que el usuario inicie sesión.** Como último recurso, Microsoft se reserva el derecho de quitar cualquier aplicación de la lista de asistentes habilitados permitidos si no se resuelve un problema de seguridad grave a tiempo.

## <a name="design-guidance-for-voice-activation-preview"></a>Guía de diseño de la vista previa de la activación por voz

Si el dispositivo está sin bloquear, cuando la aplicación del asistente _no_ tiene el foco, Windows proporciona una interfaz de usuario para la activación por voz menos intrusiva, para que el usuario se encuentre a gusto. Esto sucede especialmente en el caso de activaciones falsas, ya que supondrían un gran problema si iniciaran toda la aplicación. La idea es que cada asistente tenga otra página principal en el shell, el icono de la barra de tareas del asistente. Cuando se produce la solicitud de activación en segundo plano, aparece una pequeña vista encima del icono de la barra de tareas del asistente. Los asistentes deben proporcionar una pequeña experiencia de escucha en este lienzo. Después de procesar las solicitudes, los asistentes pueden elegir cambiar el tamaño de la vista para mostrar una respuesta en contexto, o bien entregar la vista principal de su aplicación, para que muestre objetos visuales mayores con más detalle.

- Para que el tamaño sea el menor posible, la vista previa no tiene barra de título, por lo que **el asistente debe dibujar una X en la parte superior derecha para que los usuarios puedan descartar la vista**. Consulte el apartado en el que se indica cómo [cerrar la aplicación](windows-voice-assistants-implementation-guide.md#closing-the-application) para conocer las API específicas a las que se llama cuando se hace clic en el botón de descarte.
- Para poder usar las vistas previas de la activación por voz, los asistentes pueden invitar a los usuarios a anclar el asistente a la barra de tareas la primera vez que se ejecute.

**Vista previa de la activación por voz: estado inicial**

El asistente de Contoso tiene una página principal en la barra de tareas: su icono circular.

![Captura de pantalla del asistente por voz en Windows antes de la activación como un icono de la barra de tareas](media/voice-assistants/windows_voice_assistant/pre_compact_view.png)

**A medida que la activación progresa**, el asistente solicita la activación en segundo plano. El asistente recibe un pequeño panel de vista previa (con un ancho predeterminado de 408 y un alto de 248). Si la activación por voz del lado servidor determina que la señal era un falso positivo, esta vista puede descartarse para que la interrupción sea mínima.

![Captura de pantalla del asistente de voz de Windows en forma de vista compacta durante la comprobación de la activación](media/voice-assistants/windows_voice_assistant/compact_view_activating.png)

**Cuando se confirma la activación final**, el asistente presenta su experiencia de usuario de escucha. El asistente debe dibujar siempre una X de descarte en la parte superior derecha de la vista previa de la activación por voz.

![Captura de pantalla del asistente por voz de Windows durante la escucha en vista compacta](media/voice-assistants/windows_voice_assistant/compact_view_listening.png)

Las **respuestas rápidas** se pueden mostrar en la vista previa de la activación por voz. TryResizeView permitirá a los asistentes solicitar distintos tamaños.

![Captura de pantalla del asistente por voz de Windows durante la respuesta en vista compacta](media/voice-assistants/windows_voice_assistant/compact_view_response.png)

**Entrega**. En cualquier momento, el asistente puede volver a la vista principal de la aplicación para proporcionar más información, o bien mostrar un diálogo o respuestas que requieran más espacio en la pantalla. Para más información acerca de la implementación, consulte la sección sobre la [transición de la vista compacta a la vista completa](windows-voice-assistants-implementation-guide.md#transition-from-compact-view-to-full-view).

![Capturas de pantallas del asistente por voz en Windows antes y después de expandir la vista compacta](media/voice-assistants/windows_voice_assistant/compact_transition.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Introducción al desarrollo del asistente por voz](how-to-windows-voice-assistants-get-started.md)