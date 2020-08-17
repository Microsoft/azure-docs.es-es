---
title: Mensajes EDIFACT para la integración B2B
description: Intercambio de mensajes EDIFACT en formato EDI para la integración empresarial B2B en Azure Logic Apps con Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 04/22/2020
ms.openlocfilehash: dfe48a1348929bbe8d4af6793d94d52514df9d74
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87001543"
---
# <a name="exchange-edifact-messages-for-b2b-enterprise-integration-in-azure-logic-apps-with-enterprise-integration-pack"></a>Intercambio de mensajes EDIFACT para integración empresarial B2B en Azure Logic Apps con Enterprise Integration Pack

Para poder intercambiar mensajes EDIFACT para Azure Logic Apps, debe crear un contrato EDIFACT y almacenarlo en la cuenta de integración. Estos son los pasos para crear un contrato EDIFACT.

> [!NOTE]
> En esta página se tratan las características de EDIFACT para Azure Logic Apps. Para obtener más información, consulte [X12](logic-apps-enterprise-integration-x12.md).

## <a name="before-you-start"></a>Antes de comenzar

Esto es lo que necesita:

* Una [cuenta de integración](logic-apps-enterprise-integration-create-integration-account.md) que ya esté definida y asociada a su suscripción de Azure  
* Al menos dos [asociados](logic-apps-enterprise-integration-partners.md) que ya estén definidos en su cuenta de integración.

> [!NOTE]
> Al crear un contrato, el contenido de los mensajes que envíe al asociado o reciba de este debe coincidir con el tipo de contrato.

Cuando haya [creado una cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) y [agregado los asociados](logic-apps-enterprise-integration-partners.md), siga estos pasos para generar un contrato EDIFACT.

## <a name="create-an-edifact-agreement"></a>Creación de un acuerdo EDIFACT 

1. Inicie sesión en [Azure Portal](https://portal.azure.com "Azure Portal"). 

2. En el menú principal de Azure, seleccione **Todos los servicios**. En el cuadro de búsqueda, escriba "integración" y, después, seleccione **Cuentas de integración**.

   ![Búsqueda de la cuenta de integración](./media/logic-apps-enterprise-integration-edifact/edifact-0.png)

   > [!TIP]
   > Si no aparece **Todos los servicios**, expanda el menú. En la parte superior del menú contraído, seleccione **Mostrar etiquetas de texto**.

3. En la hoja **Cuentas de integración**, seleccione la cuenta de integración donde se vaya a crear el contrato.

   ![Selección de la cuenta de integración donde desea crear el contrato](./media/logic-apps-enterprise-integration-edifact/edifact-1-4.png)

4. Elija **Acuerdos**. Si no tiene un icono Acuerdos, agréguelo.   

   ![Elección del icono "Acuerdos"](./media/logic-apps-enterprise-integration-edifact/edifact-1-5.png)

5. En la página Acuerdos, elija **Agregar**.

   ![Elección de "Agregar"](./media/logic-apps-enterprise-integration-edifact/edifact-agreement-2.png)

6. En **Agregar**, escriba un valor en **Nombre** para el contrato. Para el **Tipo de contrato**, seleccione **EDIFACT**. Seleccione valores en **Asociado del host**, **Identidad del host**, **Asociado invitado** e **Identidad del invitado** para el contrato.

   ![Especificación de los detalles del contrato](./media/logic-apps-enterprise-integration-edifact/edifact-1.png)

   | Propiedad | Descripción |
   | --- | --- |
   | Nombre |Nombre del contrato |
   | Tipo de contrato | Debe ser EDIFACT |
   | Host Partner (Partner anfitrión) |Un contrato tiene asociado un partner anfitrión e invitado. El asociado del host representa la organización que configura el contrato. |
   | Host Identity (Identidad anfitriona) |Un identificador del asociado del host. |
   | Guest Partner (Partner invitado) |Un contrato tiene asociado un partner anfitrión e invitado. El partner invitado representa la organización que está haciendo negocios con el partner anfitrión. |
   | Guest Identity |Un identificador del asociado invitado. |
   | Receive Settings (Configuración de recepción) |Estas propiedades se aplican a todos los mensajes que recibe un contrato. |
   | Send Settings (Configuración de envío) |Estas propiedades se aplican a todos los mensajes que envía un contrato. |
   ||| 

## <a name="configure-how-your-agreement-handles-received-messages"></a>Configuración de la forma en que su contrato controla los mensajes recibidos

Ahora que ha establecido las propiedades del contrato, puede configurar cómo este identifica y controla los mensajes entrantes recibidos del asociado a través de este contrato.

> [!IMPORTANT]
> El conector EDIFACT solo admite caracteres UTF-8.
> Si la salida contiene caracteres inesperados, compruebe que los mensajes EDIFACT usan el juego de caracteres UTF-8.

1. En **Agregar**, seleccione **Configuración de recepción**.
Configure estas propiedades en función del contrato con el asociado con el que intercambia mensajes. Para las descripciones de las propiedades, consulte las tablas de esta sección.

   La **configuración de recepción** se organiza en las siguientes secciones: Identificadores, Confirmación, Esquemas, Números de control, Validaciones y Configuración interna.

   ![Selección de "Configuración de recepción"](./media/logic-apps-enterprise-integration-edifact/edifact-2.png)  

2. Cuando haya terminado, asegúrese de guardar la configuración con **Aceptar**.

Ahora el contrato está preparado para controlar los mensajes entrantes que cumplan la configuración seleccionada.

### <a name="identifiers"></a>Identificadores

| Propiedad | Descripción |
| --- | --- |
| UNB6.1 (contraseña de referencia del destinatario) |Escriba un valor alfanumérico de entre 1 y 14 caracteres. |
| UNB6.2 (calificador de referencia del destinatario) |Escriba un valor alfanumérico con un carácter como mínimo y dos como máximo. |

### <a name="acknowledgments"></a>Agradecimientos

| Propiedad | Descripción |
| --- | --- |
| Receipt of Message (CONTRL) (Recepción del mensaje [CONTRL]) |Active esta casilla para devolver una confirmación técnica (CONTRL) al remitente del intercambio. Estas confirmaciones se envían al remitente del intercambio según la configuración de envío del acuerdo. |
| Acknowledgement (CONTRL) (Confirmación [CONTRL]) |Active esta casilla para devolver una confirmación funcional (CONTRL) al remitente del intercambio. La confirmación se envía al remitente del intercambio basándose en la configuración de envío del acuerdo. |

### <a name="schemas"></a>Esquemas

| Propiedad | Descripción |
| --- | --- |
| UNH2.1 (TYPE) (UNH2.1 [TIPO]) |Seleccione un tipo de conjunto de transacciones. |
| UNH2.2 (VERSION) (UNH2.2 [VERSIÓN]) |Escriba el número de versión de mensaje. (Como mínimo un carácter; tres caracteres como máximo). |
| UNH2.3 (RELEASE) (UNH2.3 [LANZAMIENTO]) |Escriba el número de versión de mensaje. (Como mínimo un carácter; tres caracteres como máximo). |
| UNH2.5 (ASSOCIATED ASSIGNED CODE) (UNH2.5 [CÓDIGO ASIGNADO ASOCIADO]) |Escriba el código asignado. (Seis caracteres como máximo. Debe ser alfanumérico). |
| UNG2.1 (APP SENDER ID) (UNG2.1 [ID. DE REMITENTE DE APLICACIÓN]) |Escriba un valor alfanumérico con un carácter como mínimo y 35 como máximo. |
| UNG2.2 (APP SENDER CODE QUALIFIER) (UNG2.2 [CALIFICADOR DE CÓDIGO DE REMITENTE DE APLICACIÓN]) |Escriba un valor alfanumérico con un máximo de cuatro caracteres. |
| SCHEMA |Seleccione el esquema cargado anteriormente que desee utilizar desde la cuenta de integración asociada. |

### <a name="control-numbers"></a>Números de control

| Propiedad | Descripción |
| --- | --- |
| Disallow Interchange Control Number duplicates (No permitir duplicados del número de control de intercambio) |Seleccione esta propiedad para bloquear los intercambios duplicados. Si la selecciona, la acción de descodificación de EDIFACT comprueba que el número de control de intercambio (UNB5) del intercambio recibido no coincide con el número de control de intercambio. Si se detecta una coincidencia, el intercambio no se procesa. |
| Check for duplicate UNB5 every (days) (Comprobar UNB5 duplicados cada [días]) |Si elige no permitir números de control de intercambio duplicados, puede especificar el número de días en que se realizará la comprobación; para ello, indique el valor pertinente para esta configuración. |
| Disallow Group control number duplicates (No permitir duplicados del número de control de grupo) |Seleccione esta propiedad para bloquear los intercambios con números de control de grupo duplicados (UNG5). |
| Disallow Transaction set control number duplicates (No permitir duplicados del número de control del conjunto de transacciones) |Seleccione esta propiedad para bloquear los intercambios con números de control de conjuntos de transacciones duplicados (UNH1). |
| EDIFACT Acknowledgement Control Number (Número de control de confirmación EDIFACT) |Para designar los números de referencia del conjunto de transacciones que se utilizará en las confirmaciones, escriba un valor para el prefijo, un intervalo de números de referencia y un sufijo. |

### <a name="validation"></a>Validación

Cuando haya completado cada fila de validación, se agrega otra automáticamente. Si no se especifica ninguna regla, la validación utiliza la fila "Predeterminado".

| Propiedad | Descripción |
| --- | --- |
| Tipo de mensaje |Seleccione el tipo de mensaje EDI. |
| EDI Validation (Validación de EDI) |Realiza una validación de EDI en los tipos de datos según lo definido en las propiedades de EDI del esquema, las restricciones de longitud, los elementos de datos vacíos y los separadores finales. |
| Extended Validation (Validación extendida) |Si el tipo de datos no es EDI, lo que se valida es el requisito de elemento de datos, así como las repeticiones permitidas, las enumeraciones y la validación de la longitud del elemento de datos (mín./máx.). |
| Allow Leading/Trailing Zeroes (Permitir ceros a la izquierda/finales) |Conserve los caracteres de espacio y cero iniciales o finales adicionales. No los quite. |
| Recortar ceros iniciales y finales |Quite los caracteres de espacio y cero iniciales o finales. |
| Trailing Separator Policy (Directiva de separador final) |Genere separadores finales. <p>Seleccione **No permitido** para prohibir los delimitadores y separadores finales en el intercambio recibido. Si el intercambio contiene delimitadores y separadores finales, se declara no válido. <p>Seleccione **Opcional** para aceptar intercambios con o sin delimitadores y separadores finales. <p>Seleccione **Obligatorio** cuando el intercambio deba tener delimitadores y separadores finales. |

### <a name="internal-settings"></a>Configuración interna

| Propiedad | Descripción |
| --- | --- |
| Create empty XML tags if trailing separators are allowed (Crear etiquetas XML vacías si se permiten separadores finales) |Seleccione esta casilla para que el remitente del intercambio incluya etiquetas XML vacías para los separadores finales. |
| Dividir intercambio como conjuntos de transacciones: suspender conjuntos de transacciones en caso de error|Analiza todos los conjuntos de transacciones de un intercambio en un documento XML independiente aplicando el sobre adecuado al conjunto de transacciones. Suspende solo los conjuntos de transacciones que no superen la validación. |
| Dividir intercambio como conjuntos de transacciones: suspender intercambio en caso de error|Analiza todos los conjuntos de transacciones de un intercambio en un documento XML independiente aplicando el sobre adecuado. Suspende todo el intercambio cuando uno o varios conjuntos de transacciones no superan la validación. | 
| Conservar intercambio: suspender conjuntos de transacciones en caso de error |Deja intacto el intercambio y crea un documento XML para todo el intercambio por lotes. Suspende solo los conjuntos de transacciones que no superan la validación, sin dejar de procesar el resto. |
| Conservar intercambio: suspender intercambio en caso de error |Deja intacto el intercambio y crea un documento XML para todo el intercambio por lotes. Suspende todo el intercambio cuando uno o varios conjuntos de transacciones no superan la validación. |

## <a name="configure-how-your-agreement-sends-messages"></a>Configuración de la forma en que su contrato envía mensajes

Puede configurar cómo este contrato identifica y administra los mensajes que se envían a los asociados a través de él.

1.  En **Agregar**, seleccione **Send Settings** (Configuración de envío).
Configure estas propiedades en función del contrato con el asociado con el que intercambia mensajes. Para las descripciones de las propiedades, consulte las tablas de esta sección.

    La **configuración de envío** se organiza en las siguientes secciones: Identificadores, Confirmación, Esquemas, Sobres, Juegos de caracteres y separadores, Números de control y Validaciones.

    !["Send Settings" (Configuración de envío)](./media/logic-apps-enterprise-integration-edifact/edifact-3.png)    

2. Cuando haya terminado, asegúrese de guardar la configuración con **Aceptar**.

Ahora el contrato está preparado para controlar los mensajes salientes que cumplan la configuración seleccionada.

### <a name="identifiers"></a>Identificadores

| Propiedad | Descripción |
| --- | --- |
| UNB1.2 (Syntax version) (UNB1.2 [versión de sintaxis]) |Seleccione un valor entre **1** y **4**. |
| UNB2.3 (Sender Reverse Routing Address) (UNB2.3 [dirección de enrutamiento inverso del remitente]) |Escriba un valor alfanumérico con un carácter como mínimo y 14 como máximo. |
| UNB3.3 (Recipient Reverse Routing Address) (UNB3.3 [dirección de enrutamiento inverso del destinatario]) |Escriba un valor alfanumérico con un carácter como mínimo y 14 como máximo. |
| UNB6.1 (contraseña de referencia del destinatario) |Escriba un valor alfanumérico con un carácter como mínimo y 14 caracteres como máximo. |
| UNB6.2 (calificador de referencia del destinatario) |Escriba un valor alfanumérico con un carácter como mínimo y dos como máximo. |
| UNB7 (Application Reference ID) (UNB7 [Identificador de referencia de solicitud]) |Escriba un valor alfanumérico con un carácter como mínimo y 14 como máximo. |

### <a name="acknowledgment"></a>Acknowledgement (Confirmación)

| Propiedad | Descripción |
| --- | --- |
| Receipt of Message (CONTRL) (Recepción del mensaje [CONTRL]) |Active esta casilla si el asociado del host espera recibir una confirmación técnica (CONTRL). Esta configuración especifica que el asociado hospedado que envía el mensaje solicita una confirmación del asociado invitado. |
| Acknowledgement (CONTRL) (Confirmación [CONTRL]) |Active esta casilla si el asociado hospedado espera recibir una confirmación funcional (CONTRL). Esta configuración especifica que el asociado hospedado que envía el mensaje solicita una confirmación del asociado invitado. |
| Generate SG1/SG4 loop for accepted transaction sets (Generar bucle SG1/SG4 en los conjuntos de transacciones aceptadas) |Si elige solicitar una confirmación funcional, active esta casilla para forzar la generación de bucles SG1/SG4 en confirmaciones CONTRL funcionales para conjuntos de transacciones aceptados. |

### <a name="schemas"></a>Esquemas

| Propiedad | Descripción |
| --- | --- |
| UNH2.1 (TYPE) (UNH2.1 [TIPO]) |Seleccione un tipo de conjunto de transacciones. |
| UNH2.2 (VERSION) (UNH2.2 [VERSIÓN]) |Escriba el número de versión de mensaje. |
| UNH2.3 (RELEASE) (UNH2.3 [LANZAMIENTO]) |Escriba el número de versión de mensaje. |
| SCHEMA |Seleccione el esquema que se utilizará. Los esquemas se encuentran en la cuenta de integración. Para obtener acceso a los esquemas, antes debe vincular su cuenta de integración con su aplicación lógica. |

### <a name="envelopes"></a>Envelopes (Sobres)

| Propiedad | Descripción |
| --- | --- |
| UNB8 (Processing Priority Code) (UNB8 [Código de prioridad de procesamiento]) |Escriba un valor alfabético con un solo carácter. |
| UNB10 (Communication Agreement) (UNB10 [Acuerdo de comunicaciones]) |Escriba un valor alfanumérico con un carácter como mínimo y 40 como máximo. |
| UNB11 (Test Indicator) (UNB11 [Indicador de prueba]) |Active esta casilla para indicar que el intercambio generado son datos de prueba |
| Aplicar segmento UNA (notificación del servicio) |Active esta casilla para generar un segmento UNA para el intercambio que se va a enviar. |
| Aplicar segmentos UNG (encabezado de grupo funcional) |Active esta casilla para crear segmentos de agrupación en el encabezado de grupo funcional en los mensajes enviados al asociado invitado. Los siguientes valores se utilizan para crear los segmentos UNG: <p>En el caso de **UNG1**, escriba un valor alfanumérico con un mínimo de un carácter y un máximo de seis caracteres. <p>En el caso de **UNG2.1**, escriba un valor alfanumérico con un mínimo de un carácter y un máximo de 35 caracteres. <p>En el caso de **UNG2.2**, escriba un valor alfanumérico con un máximo de cuatro caracteres. <p>En el caso de **UNG3.1**, escriba un valor alfanumérico con un mínimo de un carácter y un máximo de 35 caracteres. <p>En el caso de **UNG3.2**, escriba un valor alfanumérico con un máximo de cuatro caracteres. <p>En el caso de **UNG6**, escriba un valor alfanumérico con un mínimo de un carácter y un máximo de tres. <p>En el caso de **UNG7.1**, escriba un valor alfanumérico con un mínimo de un carácter y un máximo de tres caracteres. <p>En el caso de **UNG7.2**, escriba un valor alfanumérico con un mínimo de un carácter y un máximo de tres caracteres. <p>En el caso de **UNG7.3**, escriba un valor alfanumérico con un mínimo de un carácter y un máximo de seis caracteres. <p>En el caso de **UNG8**, escriba un valor alfanumérico con un carácter como mínimo y 14 caracteres como máximo. |

### <a name="character-sets-and-separators"></a>Juegos de caracteres y separadores

Además del juego de caracteres, puede especificar un conjunto de delimitadores diferente para cada tipo de mensaje. Si no se especifica un juego de caracteres en un esquema de mensaje concreto, se usará el juego de caracteres predeterminado.

| Propiedad | Descripción |
| --- | --- |
| UNB1.1 (System Identifier) (UNB1.1 [Identificador del sistema]) |Seleccione el conjunto de caracteres EDIFACT que se aplicará al intercambio de salida. |
| Schema |Seleccione un esquema en la lista desplegable. Después de completar cada fila, se agrega automáticamente una nueva. Para el esquema seleccionado, elija el conjunto de separadores que desee usar, en función de las siguientes descripciones. |
| Tipo de entrada |Seleccione un tipo de entrada en la lista desplegable. |
| Separador de componentes |Para separar elementos de datos compuestos, escriba un solo carácter. |
| Separador de elementos de datos |Para separar elementos de datos simples dentro de elementos de datos compuestos, escriba un solo carácter. |
| Terminador de segmento |Para indicar el final de un segmento EDI, escriba un solo carácter. |
| Sufijo |Seleccione el carácter que se usa con el identificador de segmento. Si designa un sufijo, el elemento de datos del terminador de segmento podrá estar vacío. Si se deja vacío el terminador de segmento, deberá designar un sufijo. |

### <a name="control-numbers"></a>Números de control

| Propiedad | Descripción |
| --- | --- |
| UNB5 (Interchange Control Number) (UNB5 [Número de control de intercambio]) |Escriba un prefijo, un intervalo de valores para el número de control de intercambio y un sufijo. Estos valores se utilizan para generar un intercambio saliente. El prefijo y el sufijo son opcionales; el número de control, obligatorio. El número de control se incrementa para cada nuevo mensaje; el prefijo y el sufijo siguen siendo los mismos. |
| UNG5 (Group Control Number) (UNG5 [Número de control de grupo]) |Escriba un prefijo, un intervalo de valores para el número de control de intercambio y un sufijo. Estos valores se utilizan para generar el número de control de grupo. El prefijo y el sufijo son opcionales; el número de control, obligatorio. El número de control se incrementa para cada nuevo mensaje hasta que se alcanza el valor máximo; el prefijo y el sufijo siguen siendo los mismos. |
| UNH1 (Message Header Reference Number) (UNH1 [Número de referencia de encabezado de mensaje]) |Escriba un prefijo, un intervalo de valores para el número de control de intercambio y un sufijo. Estos valores se utilizan para generar el número de referencia de encabezado del mensaje. El prefijo y el sufijo son opcionales; el número de referencia, obligatorio. El número de referencia se incrementa para cada nuevo mensaje; el prefijo y el sufijo siguen siendo los mismos. |

### <a name="validation"></a>Validación

Cuando haya completado cada fila de validación, se agrega otra automáticamente. Si no se especifica ninguna regla, la validación utiliza la fila "Predeterminado".

| Propiedad | Descripción |
| --- | --- |
| Tipo de mensaje |Seleccione el tipo de mensaje EDI. |
| EDI Validation (Validación de EDI) |Realiza una validación de EDI en los tipos de datos según lo definido en las propiedades de EDI del esquema, las restricciones de longitud, los elementos de datos vacíos y los separadores finales. |
| Extended Validation (Validación extendida) |Si el tipo de datos no es EDI, lo que se valida es el requisito de elemento de datos, así como las repeticiones permitidas, las enumeraciones y la validación de la longitud del elemento de datos (mín./máx.). |
| Allow Leading/Trailing Zeroes (Permitir ceros a la izquierda/finales) |Conserve los caracteres de espacio y cero iniciales o finales adicionales. No los quite. |
| Recortar ceros iniciales y finales |Quite los ceros iniciales o finales. |
| Trailing Separator Policy (Directiva de separador final) |Genere separadores finales. <p>Seleccione **No permitido** para prohibir los delimitadores y separadores finales en el intercambio enviado. Si el intercambio contiene delimitadores y separadores finales, se declara no válido. <p>Seleccione **Opcional** para enviar intercambios con o sin delimitadores y separadores finales. <p>Seleccione **Obligatorio** si el intercambio enviado debe contener delimitadores y separadores finales. |

## <a name="find-your-created-agreement"></a>Búsqueda del contrato creado

1.  Cuando termine de establecer todas las propiedades del contrato, en la página **Agregar**, elija **Aceptar** para terminar de crear el contrato y volver a la cuenta de integración.

    Ahora el contrato recién agregado aparece en la lista **Acuerdos**.

2.  También puede ver los contratos en la información general de la cuenta de integración. En el menú de la cuenta de integración, elija **Información general** y seleccione el icono **Acuerdos**. 

    ![Elección del icono "Acuerdos"](./media/logic-apps-enterprise-integration-edifact/edifact-4.png)   

## <a name="connector-reference"></a>Referencia de conectores

Si necesita más detalles técnicos sobre este conector, como las acciones y los límites que se describen en el archivo de Swagger del conector, consulte la [página de referencia del conector](/connectors/edifact/).

> [!NOTE]
> En el caso de las aplicaciones lógicas de un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión con la etiqueta ISE de este conector usa los [límites de mensajes B2B del ISE](../logic-apps/logic-apps-limits-and-config.md#b2b-protocol-limits).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)
