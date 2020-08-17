---
title: Solución de problemas de alertas y notificaciones de Azure Monitor
description: Problemas comunes con las alertas de Azure Monitor y posibles soluciones.
author: ofirmanor
ms.author: ofmanor
ms.topic: reference
ms.date: 03/16/2020
ms.subservice: alerts
ms.openlocfilehash: c3eed095d87d1e1fa1f210b360822ceefdbe0521
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87045240"
---
# <a name="troubleshooting-problems-in-azure-monitor-alerts"></a>Solución de problemas en las alertas de Azure Monitor

En este artículo se describen los problemas comunes en las alertas y notificaciones de Azure Monitor.

Las alertas de Azure Monitor le informan de forma proactiva cuando se detectan condiciones importantes en los datos que se supervisan. Le permiten identificar y solucionar los problemas antes de que los usuarios del sistema puedan verlos. Para más información sobre las alertas, consulte [Información general sobre las alertas en Microsoft Azure](alerts-overview.md).

Si tiene un problema en el que a veces una alerta no se desencadena según lo esperado, consulte los artículos siguientes. Puede ver las alertas "desencadenadas" en Azure Portal.

- [Solución de problemas con las alertas de métricas de Azure Monitor en Microsoft Azure](alerts-troubleshoot-metric.md)  
- [Solución de problemas con las alertas de registro de Azure Monitor en Microsoft Azure](alerts-troubleshoot-metric.md)

Si la alerta se desencadena según lo previsto en Azure Portal, pero no se generan las notificaciones adecuadas, use la información que se encuentra a lo largo de este artículo para solucionar ese problema.

## <a name="action-or-notification-on-my-alert-did-not-work-as-expected"></a>La acción o notificación de mi alerta no funcionó como se esperaba

Si puede ver una alerta desencadenada en Azure Portal, pero tiene un problema con algunas de sus acciones o notificaciones, consulte las secciones siguientes.

## <a name="did-not-receive-expected-email"></a>No se recibió el correo electrónico esperado

Si puede ver una alerta desencadenada en Azure Portal, pero no recibió el correo electrónico que ha configurado para ello, siga estos pasos:

1. **¿Se suprimió el correo electrónico mediante una [regla de acción](alerts-action-rules.md)** ?

    Compruébelo haciendo clic en la alerta desencadenada en el portal y examine la pestaña Historial en busca de los [grupos de acciones](action-groups.md) suprimidos:

    ![Historial de supresiones de reglas de acción de alerta](media/alerts-troubleshoot/history-action-rule.png)

1. **¿Está el tipo de acción "Rol de Azure Resource Manager de correo electrónico"?**

    Esta acción solo examina las asignaciones de roles de Azure Resource Manager que se encuentran en el ámbito de la suscripción y de tipo *Usuario*.  Asegúrese de que ha asignado el rol en el nivel de suscripción y no en el nivel de recurso o de grupo de recursos.

1. **¿El servidor de correo electrónico y el buzón aceptan correos electrónicos externos?**

    Compruebe que no se bloqueen los mensajes de correo electrónico de estas tres direcciones:
      - azure-noreply@microsoft.com  
      - azureemail-noreply@microsoft.com
      - alerts-noreply@mail.windowsazure.com

    Es habitual que las listas de distribución de correo o las listas postales internas bloqueen correos electrónicos de direcciones de correo electrónico externas. Debe permitir el correo de las direcciones de correo electrónico anteriores.  
    Para probarlo, agregue una dirección de correo electrónico del trabajo convencional (no una lista de distribución de correo) al grupo de acciones y compruebe si las alertas llegan a ese correo electrónico.

1. **¿El correo electrónico lo procesaron reglas de la bandeja de entrada o un filtro de correo no deseado?**

    Compruebe que no hay ninguna regla de bandeja de entrada que elimine esos mensajes de correo electrónico o los mueva a otra carpeta. Por ejemplo, las reglas de bandeja de entrada podrían detectar remitentes específicos o palabras concretas en el asunto.

    Compruebe también:

   - la configuración del correo no deseado de su cliente de correo electrónico (como Outlook o Gmail)
      - los límites del remitente, la configuración del correo no deseado y la configuración de la cuarentena del servidor de correo electrónico (como Exchange, Office 365 o G-Suite)
      - la configuración de la aplicación de seguridad de correo electrónico, si existe (como Barracuda o Cisco).

1. **¿Ha cancelado la suscripción al grupo de acciones accidentalmente?**

    Los correos electrónicos de alerta proporcionan un vínculo para cancelar la suscripción al grupo de acciones. Para comprobar si ha cancelado la suscripción a este grupo de acciones accidentalmente, puede realizar una de estas acciones:

    1. Abrir el grupo de acciones en el portal y comprobar la columna Estado:

    ![Columna Estado del grupo de acciones](media/alerts-troubleshoot/action-group-status.png)

    2. Buscar en el correo electrónico la confirmación de cancelación de suscripción:

    ![Cancelación de suscripción al grupo de acciones de alerta](media/alerts-troubleshoot/unsubscribe-action-group.png)

    Para suscribirse de nuevo, use el vínculo del correo electrónico de confirmación de cancelación de suscripción que ha recibido, o bien quite la dirección de correo electrónico del grupo de acciones y luego vuelva a agregarla. 
 
1. **¿Se le ha limitado la tasa debido a que muchos correos electrónicos van a una sola dirección de correo electrónico?**

    El correo electrónico tiene una [tasa limitada](alerts-rate-limiting.md) de no más de cien mensajes de correo electrónico cada hora para cada dirección. Si supera este umbral, se quitan las notificaciones de correo electrónico adicionales.  Compruebe si ha recibido un mensaje que indica que la dirección de correo electrónico tiene limitado el número de correos temporalmente: 
 
   ![Tasa de correo electrónico limitada](media/alerts-troubleshoot/email-paused.png)

   Si desea recibir un gran volumen de notificaciones sin limitación de tasa, considere la posibilidad de usar una acción diferente, como el método webhook, la aplicación lógica, la función de Azure o los runbooks de automatización, ninguna de las cuales tiene una tasa limitada. 

## <a name="did-not-receive-expected-sms-voice-call-or-push-notification"></a>No se recibió el SMS, la llamada de voz o la notificación de inserción esperada

Si puede ver una alerta desencadenada en el portal pero no recibió el SMS, la llamada de voz o la notificación de inserción que ha configurado para ello, siga estos pasos: 

1. **¿Se suprimió la acción mediante una [regla de acción](alerts-action-rules.md)?**

    Compruébelo haciendo clic en la alerta desencadenada en el portal y examine la pestaña Historial en busca de los [grupos de acciones](action-groups.md) suprimidos: 

    ![Historial de supresiones de reglas de acción de alerta](media/alerts-troubleshoot/history-action-rule.png)

   Si no se realizó de manera intencionada, puede modificar, deshabilitar o eliminar la regla de acción.
 
1. **SMS/voz:  ¿es correcto su número de teléfono?**

   Compruebe la acción de SMS por si hubiera errores tipográficos en el código de país o en el número de teléfono.
 
1. **SMS/voz: ¿se le ha limitado la tasa?**

    Los SMS y las llamadas de voz tienen una tasa limitada de no más de una notificación cada cinco minutos por número de teléfono. Si supera este umbral, las notificaciones se quitarán.

      - Llamada de voz: compruebe el historial de llamadas y vea si tiene una llamada diferente de Azure en los cinco minutos anteriores.
      - SMS: compruebe en el historial de SMS si tiene un mensaje que indica que el número de teléfono tiene limitada la tasa.

    Si desea recibir un gran volumen de notificaciones sin limitación de tasa, considere la posibilidad de usar una acción diferente, como el método webhook, la aplicación lógica, la función de Azure o los runbooks de automatización, ninguna de las cuales tiene una tasa limitada. 
 
1. **SMS: ¿ha cancelado la suscripción al grupo de acciones accidentalmente?**

    Abra el historial de SMS y compruebe si ha optado por no recibir la entrega de SMS de este grupo de acciones específico (mediante la respuesta DISABLE nombre_corto_de_grupo_de_acciones) o de todos los grupos de acciones (mediante la respuesta STOP). Para suscribirse de nuevo, envíe el comando SMS pertinente (ENABLE nombre_corto_de_grupo_de_acciones o START) o bien quite la acción de SMS del grupo de acciones y luego vuelva a agregarla.  Para obtener más información, consulte [Comportamiento de las alertas por SMS en los grupos de acciones](alerts-sms-behavior.md).

1. **¿Ha bloqueado accidentalmente las notificaciones en el teléfono?**

   La mayoría de los teléfonos móviles le permiten bloquear llamadas o SMS de números de teléfono o códigos cortos específicos, o bloquear las notificaciones de inserción de aplicaciones concretas (como Azure Mobile App). Para comprobar si ha bloqueado accidentalmente las notificaciones en el teléfono, busque en la documentación correspondiente al sistema operativo y modelo del teléfono, o bien pruebe con un teléfono y número de teléfono diferentes.

## <a name="expected-another-type-of-action-to-trigger-but-it-did-not"></a>Se esperaba otro tipo de acción para desencadenar, pero no tuvo lugar 
   
Si puede ver una alerta desencadenada en el portal, pero su acción configurada no se desencadenó, siga estos pasos:

1. **¿Se suprimió la acción mediante una regla de acción?**

    Compruébelo haciendo clic en la alerta desencadenada en el portal y examine la pestaña Historial en busca de los [grupos de acciones](action-groups.md) suprimidos:

    ![Historial de supresiones de reglas de acción de alerta](media/alerts-troubleshoot/history-action-rule.png)
 
    Si no se realizó de manera intencionada, puede modificar, deshabilitar o eliminar la regla de acción.

1. **¿No se ha desencadenado un webhook?**

    1. **¿Se han bloqueado las direcciones IP de origen?**
    
       Agregue las [direcciones IP](action-groups.md#action-specific-information) desde las que se llama al webhook a su lista de permitidos.

    1. **¿Funciona correctamente el punto de conexión de webhook?**

       Compruebe que la configuración del punto de conexión del webhook es correcta y que el punto de conexión funciona correctamente. Compruebe los registros de webhook o instrumente su código para que pueda investigar (por ejemplo, registre la carga útil de entrada).

    1. **¿Está llamando a Slack o a Microsoft Teams?**  
    Cada uno de estos puntos de conexión espera un formato JSON específico. Siga [estas instrucciones](action-groups-logic-app.md) para configurar una acción de aplicación lógica en su lugar.

    1. **¿El webhook dejó de responder o devolvió errores?** 

        Nuestro tiempo de espera para una respuesta de webhook es de diez segundos. Se volverá a intentar la llamada de webhook hasta dos veces más cuando se devuelvan los siguientes códigos de estado HTTP: 408, 429, 503, 504 o cuando el punto de conexión HTTP no responda. El primer reintento se produce transcurridos 10 segundos. El segundo reintento, y final, se produce una vez transcurridos cien segundos. Si el segundo reintento produce un error, el punto de conexión no se llamará de nuevo durante 30 minutos para ninguno de los grupos de acciones.

## <a name="action-or-notification-happened-more-than-once"></a>La acción o notificación se produjo más de una vez 

Si ha recibido una notificación de una alerta (por ejemplo, un mensaje de correo electrónico o un SMS) más de una vez, o si la acción de la alerta (por ejemplo, un webhook o una función de Azure) se desencadenó varias veces, siga estos pasos: 

1. **¿Es realmente la misma alerta?** 

    En algunos casos, se desencadenan varias alertas similares en torno a la misma hora. Por lo tanto, es posible que parezca que la misma alerta desencadenó sus acciones varias veces. Por ejemplo, una regla de alerta del registro de actividad puede configurarse para desencadenarse cuando se inicia y finaliza un evento (operación correcta o errónea); para ello, no se aplicarían filtros en el campo de estado del evento. 

    Para comprobar si estas acciones o notificaciones provienen de diferentes alertas, examine los detalles de la alerta, como su marca de tiempo y el identificador de la alerta o su identificador de correlación. También puede consultar la lista de alertas desencadenadas en el portal. En ese caso, necesitaría adaptar la lógica de la regla de alerta o configurar el origen de la alerta. 

1. **¿La acción se repite en varios grupos de acciones?** 

    Cuando una alerta se desencadena, cada uno de sus grupos de acciones se procesa de forma independiente. Por lo tanto, si una acción (por ejemplo, una dirección de correo electrónico) aparece en varios grupos de acciones desencadenados, se le llamará una vez por cada grupo de acciones. 

    Para comprobar los grupos de acciones que se han desencadenado, consulte la pestaña del historial de alertas. Allí vería tanto los grupos de acciones definidos en la regla de alerta como los grupos de acciones agregados a la alerta mediante reglas de acción: 

    ![Acción repetida en varios grupos de acciones](media/alerts-troubleshoot/action-repeated-multi-action-groups.png)

## <a name="action-or-notification-has-an-unexpected-content"></a>La acción o notificación tiene un contenido inesperado

Si ha recibido la alerta pero cree que faltan algunos de sus campos o son incorrectos, siga estos pasos: 

1. **¿Ha elegido el formato correcto para la acción?** 

    Cada tipo de acción (correo electrónico, webhook, etc.) tiene dos formatos: el predeterminado (el formato heredado) y el [formato de esquema común más reciente](alerts-common-schema.md). Cuando se crea un grupo de acciones, se especifica el formato que se desea por acción; las distintas acciones de los grupos de acciones pueden tener diferentes formatos. 

    Por ejemplo, para la acción de webhook: 

    ![Opción de esquema de acción de webhook](media/alerts-troubleshoot/webhook.png)

    Compruebe si el formato especificado en el nivel de acción es el esperado. Por ejemplo, puede que haya desarrollado un código que responda a las alertas (webhook, función, aplicación lógica, etc.), que espera un formato, pero más adelante en la acción usted u otra persona especificó un formato diferente.  

    Además, compruebe el formato de carga útil (JSON) para [alertas de registro de actividad](activity-log-alerts-webhook.md), para [alertas de búsqueda de registros](alerts-log-webhook.md) (Application Insights y análisis de registros), para [alertas de métricas](alerts-metric-near-real-time.md#payload-schema), para el [esquema de alerta común](alerts-common-schema-definitions.md) y para las [alertas de métricas clásicas](alerts-webhooks.md) en desuso.

 
1. **Alertas de registros de actividad: ¿La información está disponible en el registro de actividad?** 

    Las [alertas del registro de actividad](activity-log-alerts.md) son alertas basadas en eventos escritos en el registro de actividad de Azure, tales como eventos sobre la creación, actualización o eliminación de recursos de Azure, eventos sobre el estado de los servicios y recursos o hallazgos de Azure Advisor y Azure Policy. Si ha recibido una alerta basada en el registro de actividad pero algunos campos que necesita faltan o son incorrectos, compruebe primero los eventos en el propio registro de actividad. Si el recurso de Azure no escribió los campos que está buscando en su evento de registro de actividad, esos campos no se incluirán en la alerta correspondiente. 

## <a name="action-rule-is-not-working-as-expected"></a>La regla de acción no funciona como se esperaba 

Si puede ver una alerta desencadenada en el portal, pero una regla de acción relacionada no funcionó según lo esperado, siga estos pasos: 

1. **¿Está habilitada la regla de acción?** 

    Compruebe la columna Estado de la regla de acción para ver si el rol de acción correspondiente está habilitado. 

    ![Gráfico](media/alerts-troubleshoot/action-rule-status.png) 

    Si no está habilitada, puede habilitar la regla de acción seleccionándola y haciendo clic en Habilitar. 

1. **¿Se trata de una alerta de estado del servicio?** 

    Las alertas de estado del servicio (servicio de monitor = "Service Health") no se ven afectadas por las reglas de acción. 

1. **¿La regla de acción actuó en la alerta?** 

    Compruebe si la regla de acción ha procesado la alerta haciendo clic en la alerta desencadenada en el portal y examine la pestaña Historial.

    Este es un ejemplo de regla de acción que suprime todos los grupos de acciones: 
 
     ![Historial de supresiones de reglas de acción de alerta](media/alerts-troubleshoot/history-action-rule.png)

    Este es un ejemplo de una regla de acción que agrega otro grupo de acciones:

    ![Acción repetida en varios grupos de acciones](media/alerts-troubleshoot/action-repeated-multi-action-groups.png)
 

1. **¿El filtro y el ámbito de la regla de acción coinciden con la alerta desencadenada?** 

    Si cree que la regla de acción se debe haber desencadenado pero no lo hizo, o que no se debería haber desencadenado pero lo hizo, examine cuidadosamente el ámbito de la regla de acción y las condiciones del filtro comparándolos con las propiedades de la alerta desencadenada. 


## <a name="how-to-find-the-alert-id-of-a-fired-alert"></a>Cómo buscar el identificador de alerta de una alerta desencadenada

Al abrir un caso sobre una alerta desencadenada concreta (por ejemplo, si no recibió su notificación por correo electrónico), deberá proporcionar el identificador de la alerta. 

Para localizarlo, siga estos pasos:

1. En Azure Portal, vaya a la lista de alertas desencadenadas y busque esa alerta específica. Puede usar los filtros para ayudarle a encontrarlo. 

1. Haga clic en la alerta para abrir los detalles de la misma. 

1. Desplácese hacia abajo por los campos de alerta de la primera pestaña (la pestaña de resumen) hasta que lo localice y cópielo. Ese campo también incluye un botón auxiliar "Copiar al Portapapeles" que puede usar.  

    ![Búsqueda del identificador de alerta](media/alerts-troubleshoot/get-alert-id.png)

## <a name="problem-creating-updating-or-deleting-action-rules-in-the-azure-portal"></a>Problema al crear, actualizar o eliminar reglas de acción en Azure Portal

Si ha recibido un error al intentar crear, actualizar o eliminar una [regla de acción](alerts-action-rules.md), siga estos pasos: 

1. **¿Ha recibido un error de permiso?**  

    Debe tener el  [rol integrado Colaborador de supervisión](../../role-based-access-control/built-in-roles.md#monitoring-contributor) o bien los permisos específicos relacionados con las reglas de acción y las alertas.

1. **¿Ha comprobado los parámetros de la regla de acción?**  

    Compruebe la [documentación de la regla de acción](alerts-action-rules.md) o el comando [Set-AzActionRule de PowerShell de la regla de acción](/powershell/module/az.alertsmanagement/set-azactionrule?view=azps-3.5.0). 


## <a name="next-steps"></a>Pasos siguientes
- Si usa una alerta de registro, consulte también [Solución de problemas de alertas de registro en Azure Monitor](./alerts-troubleshoot-log.md).
- Vuelva a [Azure Portal](https://portal.azure.com) para comprobar si ha resuelto el problema con las instrucciones anteriores. 
