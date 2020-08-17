---
title: Envío y recepción de mensajes X12 para B2B
description: Intercambio de mensajes X12 para escenarios de integración empresarial B2B en Azure Logic Apps con Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 04/29/2020
ms.openlocfilehash: 61f3f2af61bc24f76d061de672a3eaacd54f7f0e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87066125"
---
# <a name="exchange-x12-messages-for-b2b-enterprise-integration-in-azure-logic-apps-with-enterprise-integration-pack"></a>Intercambio de mensajes X12 para la integración empresarial B2B en Azure Logic Apps con Enterprise Integration Pack

Para trabajar con mensajes X12 en Azure Logic Apps, puede usar el conector X12, que proporciona los desencadenadores y acciones necesarios para administrar las comunicaciones X12. Para obtener información sobre los mensajes EDIFACT, consulte [Intercambio de mensajes EDIFACT](logic-apps-enterprise-integration-edifact.md).

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si aún no tiene ninguna suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* La aplicación lógica desde la que quiere usar el conector X12 y un desencadenador que inicie el flujo de trabajo de la aplicación lógica. El conector X12 proporciona únicamente las acciones, no los desencadenadores. Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) e [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* Una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) asociada con la suscripción a Azure y vinculada a las aplicaciones lógicas en las que planea usar el conector X12. Tanto la cuenta de integración como la aplicación de lógica deben existir en la misma ubicación o región de Azure.

* Al menos dos [entidades](../logic-apps/logic-apps-enterprise-integration-partners.md) que ya haya definido en su cuenta de integración mediante el calificador de identidad X12.

* Los [esquemas](../logic-apps/logic-apps-enterprise-integration-schemas.md) que se usarán para la validación XML que ya ha agregado a su cuenta de integración. Si está trabajando con esquemas de ley de transferencia y responsabilidad de seguros de salud (HIPAA), consulte [esquemas de HIPAA](#hipaa-schemas).

* Para poder usar el conector X12, debe crear un [contrato](../logic-apps/logic-apps-enterprise-integration-agreements.md) X12 entre las entidades y almacenarlo en la cuenta de integración. Si está trabajando con esquemas de ley de transferencia y responsabilidad de seguros de salud (HIPAA), necesita agregar una sección `schemaReferences` a su acuerdo. Para más información, consulte los [esquemas de HIPAA](#hipaa-schemas).

<a name="receive-settings"></a>

## <a name="receive-settings"></a>Receive Settings (Configuración de recepción)

Una vez definidas las propiedades del contrato, puede configurar cómo este identifica y controla los mensajes entrantes que recibe de su asociado mediante este contrato.

1. En **Agregar**, seleccione **Configuración de recepción**.

1. Configure estas propiedades en función del contrato con el asociado con el que intercambia mensajes. La **Configuración de recepción** se organiza en estas secciones:

   * [Identificadores](#inbound-identifiers)
   * [Reconocimiento](#inbound-acknowledgement)
   * [Esquemas](#inbound-schemas)
   * [Sobres](#inbound-envelopes)
   * [Números de control](#inbound-control-numbers)
   * [Validaciones](#inbound-validations)
   * [Configuración interna](#inbound-internal-settings)

   Para las descripciones de las propiedades, consulte las tablas de esta sección.

1. Cuando acabe, asegúrese seleccionar **Aceptar** para guardar la configuración.

<a name="inbound-identifiers"></a>

### <a name="receive-settings---identifiers"></a>Configuración de recepción: identificadores

![Propiedades de identificador para mensajes entrantes](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-identifiers.png)

| Propiedad | Descripción |
|----------|-------------|
| **ISA1 (calificador de autorización)** | Valor del calificador de autorización que quiere usar. El valor predeterminado es **00 - No existe información de autorización**. <p>**Nota**: Si selecciona otros valores, especifique un valor para la propiedad **ISA2**. |
| **ISA2** | Valor de la información de autorización que se usará cuando la propiedad **ISA1** no esté definida como **00 - No existe información de autorización**. Este valor de propiedad debe tener un carácter alfanumérico como mínimo y 10 como máximo. |
| **ISA3 (calificador de seguridad)** | Valor del calificador de seguridad que quiere usar. El valor predeterminado es **00 - No existe información de seguridad**. <p>**Nota**: Si selecciona otros valores, especifique un valor para la propiedad **ISA4**. |
| **ISA4** | Valor de la información de seguridad que se usará cuando la propiedad **ISA3** no esté definida como **00 - No existe información de seguridad**. Este valor de propiedad debe tener un carácter alfanumérico como mínimo y 10 como máximo. |
|||

<a name="inbound-acknowledgement"></a>

### <a name="receive-settings---acknowledgement"></a>Configuración de recepción: confirmación

![Confirmación para mensajes entrantes](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-acknowledgement.png)

| Propiedad | Descripción |
|----------|-------------|
| **TA1 esperada** | Devuelva una confirmación técnica (TA1) al remitente del intercambio. |
| **FA esperada** | Devuelva una confirmación funcional (FA) al remitente del intercambio. <p>Para la propiedad **Versión de FA**, en función de la versión de esquema, seleccione la confirmación 997 o 999. <p>Para activar la generación de bucles AK2 en confirmaciones funcionales para conjuntos de transacciones aceptados, seleccione **Incluir bucle AK2/IK2**. |
||||

<a name="inbound-schemas"></a>

### <a name="receive-settings---schemas"></a>Configuración de recepción: esquemas

![Esquemas para mensajes entrantes](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-schemas.png)

En esta sección, seleccione un [esquema](../logic-apps/logic-apps-enterprise-integration-schemas.md) de la [cuenta de integración](./logic-apps-enterprise-integration-create-integration-account.md) para cada tipo de transacción (ST01) y aplicación de remitentes (GS02). La canalización de recepción de EDI desensambla el mensaje entrante haciendo coincidir los valores y el esquema que defina en esta sección con los valores de ST01 y GS02 del mensaje entrante y con el esquema del mensaje entrante. Después de completar cada fila, aparece una fila vacía automáticamente.

| Propiedad | Descripción |
|----------|-------------|
| **Versión** | Versión X12 para el esquema. |
| **Tipo de transacción (ST01)** | Tipo de transacción |
| **Aplicación de remitentes (GS02)** | Aplicación de remitentes |
| **Esquema** | Archivo de esquema que quiere usar |
|||

<a name="inbound-envelopes"></a>

### <a name="receive-settings---envelopes"></a>Configuración de recepción: sobres

![Separadores que se usarán en conjuntos de transacciones para mensajes entrantes](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-envelopes.png)

| Propiedad | Descripción |
|----------|-------------|
| **Uso de ISA11** | Separador que se usará en un conjunto de transacciones: <p>- **Identificador estándar**: Use un punto (.) para la notación decimal, en lugar de la notación decimal del documento entrante de la canalización de recepción de EDI. <p>- **Separador de repeticiones**: Especifique el separador para las repeticiones de un elemento de datos simple o una estructura de datos repetida. Por ejemplo, normalmente se utiliza el acento circunflejo (^) como separador de repeticiones. En los esquemas HIPAA, solo se puede usar el acento circunflejo. |
|||

<a name="inbound-control-numbers"></a>

### <a name="receive-settings---control-numbers"></a>Configuración de recepción: números de control

![Control de duplicados de número de control para mensajes entrantes](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-control-numbers.png) 

| Propiedad | Descripción |
|----------|-------------|
| **No permitir duplicados en números de control de intercambio** | Bloquee los intercambios duplicados. Compruebe el número de control de intercambio (ISA13) para el número de control de intercambio recibido. Si se detecta una coincidencia, la canalización de recepción de EDI no procesa el intercambio. <p><p>Para especificar el número de días para realizar la comprobación, especifique un valor para la propiedad **Comprobar si hay ISA13 duplicados todos los (días)** . |
| **No permitir duplicados en números de control de grupo** | Bloquee los intercambios que tienen números de control de grupo duplicados. |
| **No permitir duplicados en números de control de conjuntos de transacciones** | Bloquee los intercambios que tengan números de control de conjunto de transacciones duplicados. |
|||

<a name="inbound-validations"></a>

### <a name="receive-settings---validations"></a>Configuración de recepción: validaciones

![Validaciones para mensajes entrantes](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-validations.png)

La fila **predeterminada** muestra las reglas de validación que se usan para un tipo de mensaje de EDI. Si quiere definir otras reglas, seleccione cada uno de los cuadros en los que quiera definir el conjunto de reglas como **true**. Después de completar cada fila, aparece una fila vacía automáticamente.

| Propiedad | Descripción |
|----------|-------------|
| **Tipo de mensaje** | Tipo de mensaje de EDI |
| **Validación de EDI** | Realiza una validación de EDI en los tipos de datos según lo definido en las propiedades de EDI del esquema, las restricciones de longitud, los elementos de datos vacíos y los separadores finales. |
| **Validación extendida** | Si el tipo de datos no es EDI, lo que se valida es el requisito de elemento de datos, así como las repeticiones permitidas, las enumeraciones y la validación de la longitud del elemento de datos (mín. o máx.). |
| **Permitir ceros iniciales/finales** | Conserve los caracteres de espacio y cero iniciales o finales adicionales. No los quite. |
| **Recortar ceros iniciales y finales** | Quite cualquier carácter de espacio y cero inicial o final. |
| **Directiva de separador final** | Genere separadores finales. <p>- **No permitido**: Prohíbe los delimitadores y separadores finales en el intercambio de entrada. Si el intercambio contiene delimitadores y separadores finales, se declara no válido. <p>- **Opcional**: acepta intercambios con o sin delimitadores y separadores finales. <p>- **Obligatorio**: El intercambio de entrada debe contener delimitadores y separadores finales. |
|||

<a name="inbound-internal-settings"></a>

### <a name="receive-settings---internal-settings"></a>Configuración de recepción: configuración interna

![Configuración interna para mensajes de entrada](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-internal-settings.png)

| Propiedad | Descripción |
|----------|-------------|
| **Convertir formato decimal implícito Nn en valor numérico de base 10** | Convierte un número EDI especificado con el formato "Nn" en un valor numérico de base 10. |
| **Crear etiquetas XML vacías si se permiten los separadores finales** | Hace que el remitente del intercambio incluya etiquetas XML vacías para los separadores finales. |
| **Dividir intercambio como conjuntos de transacciones: suspender conjuntos de transacciones en caso de error** | Analiza todos los conjuntos de transacciones de un intercambio en un documento XML independiente aplicando el sobre adecuado al conjunto de transacciones. Solo suspende aquellas transacciones en las que se produce un error en la validación. |
| **Dividir intercambio como conjuntos de transacciones: suspender intercambio en caso de error** | Analiza todos los conjuntos de transacciones que hay en un intercambio en un documento XML independiente aplicando el sobre adecuado. Suspende todo el intercambio cuando uno o varios conjuntos de transacciones no superan la validación. |
| **Conservar intercambio: suspender conjuntos de transacciones en caso de error** | Deja intacto el intercambio y crea un documento XML para todo el intercambio por lotes. Suspende solo los conjuntos de transacciones que no superan la validación, pero sigue procesando el resto. |
| **Conservar intercambio: suspender intercambio en caso de error** |Deja intacto el intercambio y crea un documento XML para todo el intercambio por lotes. Suspende todo el intercambio cuando uno o varios conjuntos de transacciones del intercambio no superan la validación. |
|||

<a name="send-settings"></a>

## <a name="send-settings"></a>Send Settings (Configuración de envío)

Una vez definidas las propiedades del contrato, puede configurar cómo este identifica y controla los mensajes de salida que envía a su asociado mediante este contrato.

1. En **Agregar**, seleccione **Send Settings** (Configuración de envío).

1. Configure estas propiedades en función del contrato con el asociado con el que intercambia mensajes. Para las descripciones de las propiedades, consulte las tablas de esta sección.

   La **Configuración de envío** se organiza en estas secciones:

   * [Identificadores](#outbound-identifiers)
   * [Reconocimiento](#outbound-acknowledgement)
   * [Esquemas](#outbound-schemas)
   * [Sobres](#outbound-envelopes)
   * [Número de versión de control](#outbound-control-version-number)
   * [Números de control](#outbound-control-numbers)
   * [Juegos de caracteres y separadores](#outbound-character-sets-separators)
   * [Validación](#outbound-validation)

1. Cuando acabe, asegúrese seleccionar **Aceptar** para guardar la configuración.

<a name="outbound-identifiers"></a>

### <a name="send-settings---identifiers"></a>Configuración de envío: identificadores

![Propiedades de identificador para mensajes de salida](./media/logic-apps-enterprise-integration-x12/x12-send-settings-identifiers.png)

| Propiedad | Descripción |
|----------|-------------|
| **ISA1 (calificador de autorización)** | Valor del calificador de autorización que quiere usar. El valor predeterminado es **00 - No existe información de autorización**. <p>**Nota**: Si selecciona otros valores, especifique un valor para la propiedad **ISA2**. |
| **ISA2** | Valor de la información de autorización que se usará cuando la propiedad **ISA1** no esté definida como **00 - No existe información de autorización**. Este valor de propiedad debe tener un carácter alfanumérico como mínimo y 10 como máximo. |
| **ISA3 (calificador de seguridad)** | Valor del calificador de seguridad que quiere usar. El valor predeterminado es **00 - No existe información de seguridad**. <p>**Nota**: Si selecciona otros valores, especifique un valor para la propiedad **ISA4**. |
| **ISA4** | Valor de la información de seguridad que se usará cuando la propiedad **ISA3** no esté definida como **00 - No existe información de seguridad**. Este valor de propiedad debe tener un carácter alfanumérico como mínimo y 10 como máximo. |
|||

<a name="outbound-acknowledgement"></a>

### <a name="send-settings---acknowledgement"></a>Configuración de envío: confirmación

![Propiedades de confirmación para mensajes de salida](./media/logic-apps-enterprise-integration-x12/x12-send-settings-acknowledgement.png)

| Propiedad | Descripción |
|----------|-------------|
| **TA1 esperada** | Devuelva una confirmación técnica (TA1) al remitente del intercambio. <p>Esta configuración especifica que el asociado del host que envía el mensaje solicita una confirmación del asociado invitado en el contrato. El asociado del host espera estas confirmaciones en función de la configuración de recepción del contrato. |
| **FA esperada** | Devuelva una confirmación funcional (FA) al remitente del intercambio. Para la propiedad **Versión de FA**, en función de la versión de esquema, seleccione la confirmación 997 o 999. <p>Esta configuración especifica que el asociado del host que envía el mensaje solicita una confirmación del asociado invitado en el contrato. El asociado del host espera estas confirmaciones en función de la configuración de recepción del contrato. |
|||

<a name="outbound-schemas"></a>

### <a name="send-settings---schemas"></a>Configuración de envío: esquemas

![Esquemas para mensajes de salida](./media/logic-apps-enterprise-integration-x12/x12-send-settings-schemas.png)

En esta sección, seleccione un [esquema](../logic-apps/logic-apps-enterprise-integration-schemas.md) de la [cuenta de integración](./logic-apps-enterprise-integration-create-integration-account.md) para cada tipo de transacción (ST01). Después de completar cada fila, aparece una fila vacía automáticamente.

| Propiedad | Descripción |
|----------|-------------|
| **Versión** | Versión X12 para el esquema. |
| **Tipo de transacción (ST01)** | Tipo de transacción para el esquema |
| **Esquema** | Archivo de esquema que quiere usar. Si selecciona primero el esquema, la versión y el tipo de transacción se definen automáticamente. |
|||

<a name="outbound-envelopes"></a>

### <a name="send-settings---envelopes"></a>Configuración de envío: sobres

![Separadores de un conjunto de transacciones que se usarán para los mensajes de salida](./media/logic-apps-enterprise-integration-x12/x12-send-settings-envelopes.png)

| Propiedad | Descripción |
|----------|-------------|
| **Uso de ISA11** | Separador que se usará en un conjunto de transacciones: <p>- **Identificador estándar**: Use un punto (.) para la notación decimal, en lugar de la notación decimal del documento de salida de la canalización de envío de EDI. <p>- **Separador de repeticiones**: Especifique el separador para las repeticiones de un elemento de datos simple o una estructura de datos repetida. Por ejemplo, normalmente se utiliza el acento circunflejo (^) como separador de repeticiones. En los esquemas HIPAA, solo se puede usar el acento circunflejo. |
|||

<a name="outbound-control-version-number"></a>

### <a name="send-settings---control-version-number"></a>Configuración de envío: número de versión de control

![Número de versión de control para los mensajes de salida](./media/logic-apps-enterprise-integration-x12/x12-send-settings-control-version-number.png)

En esta sección, seleccione un [esquema](../logic-apps/logic-apps-enterprise-integration-schemas.md) de la [cuenta de integración](./logic-apps-enterprise-integration-create-integration-account.md) para cada intercambio. Después de completar cada fila, aparece una fila vacía automáticamente.

| Propiedad | Descripción |
|----------|-------------|
| **Número de versión de control (ISA12)** | Versión del estándar X12 |
| **Indicador de uso (ISA15)** | Contexto de un intercambio, que pueden ser datos de **Prueba**, de **Information** o de **Producción** |
| **Esquema** | Esquema que se usará para generar los segmentos GS y ST para un intercambio X12 codificado que se envía a la canalización de envío de EDI. |
| **GS1** | Opcional: seleccione el código funcional. |
| **GS2** | Opcional: especifique el emisor de la aplicación. |
| **GS3** | Opcional: especifique el receptor de la aplicación. |
| **GS4** | Opcional: seleccione **CCYYMMDD** o **YYMMDD**. |
| **GS5** | Opcional: seleccione **HHMM**, **HHMMSS** o **HHMMSSdd**. |
| **GS7** | Opcional, seleccione un valor para la agencia responsable. |
| **GS8** | Opcional, especifique la versión del documento de esquema. |
|||

<a name="outbound-control-numbers"></a>

### <a name="send-settings---control-numbers"></a>Configuración de envío: números de control

![Números de control de los mensajes salientes](./media/logic-apps-enterprise-integration-x12/x12-send-settings-control-numbers.png)

| Propiedad | Descripción |
|----------|-------------|
| **Número de control de intercambio (ISA13)** | Intervalo de valores para el número de control de intercambio, que puede tener un valor mínimo de 1 y un máximo de 999999999 |
| **Número de control de grupo (GS06)** | Intervalo de valores para el número de control de grupo, que puede tener un valor mínimo de 1 y un máximo de 999999999 |
| **Número de control de conjunto de transacciones (ST02)** | Intervalo de valores para el número de control del conjunto de transacciones, que puede tener un valor mínimo de 1 y un máximo de 999999999 <p>- **Prefijo**: opcional (valor alfanumérico) <br>- **Sufijo**: opcional (valor alfanumérico) |
|||

<a name="outbound-character-sets-separators"></a>

### <a name="send-settings---character-sets-and-separators"></a>Configuración de envío (Juegos de caracteres y separadores)

![Delimitadores para tipos de mensajes en mensajes salientes](./media/logic-apps-enterprise-integration-x12/x12-send-settings-character-sets-separators.png)

La fila **Predeterminado** muestra el juego de caracteres que se usa como delimitador para un esquema de mensaje. Si no quiere usar el juego de caracteres **Predeterminado**, puede especificar un conjunto de delimitadores diferente para cada tipo de mensaje. Después de completar cada fila, aparece una fila vacía automáticamente.

> [!TIP]
> Para proporcionar valores de caracteres especiales, edite el contrato como JSON y proporcione el valor ASCII para el carácter especial.

| Propiedad | Descripción |
|----------|-------------|
| **Juego de caracteres que se va a usar** | El juego de caracteres X12, que es **Básico**, **Extendido** o **UTF8**. |
| **Esquema** | Esquema que quiere usar. Después de seleccionar el esquema, elija el juego de caracteres que quiera usar, en función de las siguientes descripciones. |
| **Tipo de entrada** | Tipo de entrada para el juego de caracteres |
| **Separador de componentes** | Carácter único que separa los elementos de datos compuestos |
| **Separador de elementos de datos** | Carácter único que separa los elementos de datos sencillos dentro de los datos compuestos |
| **Separador de caracteres de reemplazo** | Un carácter de reemplazo que reemplaza todos los caracteres separadores en los datos de carga al generar el mensaje X12 de salida. |
| **Terminador de segmento** | Carácter único que indica el final de un segmento EDI |
| **Sufijo** | Carácter que se usa con el identificador del segmento. Si especifica un sufijo, el elemento de datos del terminador de segmento puede estar vacío. Si el terminador de segmentos se deja vacío, debe designar un sufijo. |
|||

<a name="outbound-validation"></a>

### <a name="send-settings---validation"></a>Configuración de envío: validación

![Propiedades de validación para mensajes de salida](./media/logic-apps-enterprise-integration-x12/x12-send-settings-validation.png) 

La fila **predeterminada** muestra las reglas de validación que se usan para un tipo de mensaje de EDI. Si quiere definir otras reglas, seleccione cada uno de los cuadros en los que quiera definir el conjunto de reglas como **true**. Después de completar cada fila, aparece una fila vacía automáticamente.

| Propiedad | Descripción |
|----------|-------------|
| **Tipo de mensaje** | Tipo de mensaje de EDI |
| **Validación de EDI** | Realiza una validación de EDI en los tipos de datos según lo definido en las propiedades de EDI del esquema, las restricciones de longitud, los elementos de datos vacíos y los separadores finales. |
| **Validación extendida** | Si el tipo de datos no es EDI, lo que se valida es el requisito de elemento de datos, así como las repeticiones permitidas, las enumeraciones y la validación de la longitud del elemento de datos (mín. o máx.). |
| **Permitir ceros iniciales/finales** | Conserve los caracteres de espacio y cero iniciales o finales adicionales. No los quite. |
| **Recortar ceros iniciales y finales** | Quite cualquier carácter de espacio y cero inicial o final. |
| **Directiva de separador final** | Genere separadores finales. <p>- **No permitido**: Prohíbe los delimitadores y separadores finales en el intercambio de salida. Si el intercambio contiene delimitadores y separadores finales, se declara no válido. <p>- **Opcional**: envía intercambios con o sin delimitadores y separadores finales. <p>- **Obligatorio**: el intercambio de salida debe contener delimitadores y separadores finales. |
|||

<a name="hipaa-schemas"></a>

## <a name="hipaa-schemas-and-message-types"></a>Esquemas HIPAA y tipos de mensaje

Cuando trabaje con esquemas HIPAA y los tipos de mensaje 277 u 837, deberá seguir algunos pasos adicionales. Los [números de versión del documento (GS8)](#outbound-control-version-number) para estos tipos de mensajes tienen más de 9 caracteres; por ejemplo, "005010X222A1". Además, algunos números de versión de documento se asignan a tipos de mensaje variante. Si no hace referencia al tipo de mensaje correcto en el esquema y en el acuerdo, recibirá este mensaje de error:

`"The message has an unknown document type and did not resolve to any of the existing schemas configured in the agreement."`

En esta tabla se enumeran los tipos de mensaje afectados, cualquier variante y los números de versión de documento que se asignan a esos tipos de mensaje:

| Tipo de mensaje o variante |  Descripción | Número de versión del documento (GS8) |
|-------------------------|--------------|-------------------------------|
| 277 | Notificación de estado de información de asistencia sanitaria | 005010X212 |
| 837_I | Reclamación de asistencia sanitaria institucional | 004010X096A1 <br>005010X223A1 <br>005010X223A2 |
| 837_D | Reclamación de asistencia sanitaria dental | 004010X097A1 <br>005010X224A1 <br>005010X224A2 |
| 837_P | Reclamación de asistencia sanitaria profesional | 004010X098A1 <br>005010X222 <br>005010X222A1 |
|||

También debe deshabilitar la validación de EDI al usar estos números de versión de documento porque se produce un error que indica que la longitud de caracteres no es válida.

Para especificar estos números de versión del documento y tipos de mensaje, siga estos pasos:

1. En el esquema HIPAA, reemplace el tipo de mensaje actual por el tipo de mensaje variante para el número de versión del documento que quiere usar.

   Por ejemplo, supongamos que quiere usar el número de versión del documento `005010X222A1` con el tipo de mensaje `837`. En el esquema, reemplace cada valor `"X12_00501_837"` por el valor `"X12_00501_837_P"` en su lugar.

   Para actualizar el esquema, siga estos pasos:

   1. En Azure Portal, vaya a la cuenta de integración. Busque y descargue su esquema. Reemplace el tipo de mensaje y cambie el nombre del archivo de esquema, y cargue el esquema revisado en la cuenta de integración. Para más información, vea [Editar esquemas](../logic-apps/logic-apps-enterprise-integration-schemas.md#edit-schemas).

   1. En la configuración de mensajes del contrato, seleccione el esquema revisado.

1. En el objeto `schemaReferences` del contrato, agregue otra entrada que especifique el tipo de mensaje variante que coincida con el número de versión del documento.

   Por ejemplo, supongamos que quiere usar el número de versión del documento `005010X222A1` para el tipo de mensaje `837`. El contrato tiene una sección `schemaReferences` con estas propiedades y valores:

   ```json
   "schemaReferences": [
      {
         "messageId": "837",
         "schemaVersion": "00501",
         "schemaName": "X12_00501_837"
      }
   ]
   ```

   En esta sección `schemaReferences`, agregue otra entrada que tenga estos valores:

   * `"messageId": "837_P"`
   * `"schemaVersion": "00501"`
   * `"schemaName": "X12_00501_837_P"`

   Una vez que haya terminado, la sección `schemaReferences` debe tener un aspecto parecido a este:

   ```json
   "schemaReferences": [
      {
         "messageId": "837",
         "schemaVersion": "00501",
         "schemaName": "X12_00501_837"
      },
      {
         "messageId": "837_P",
         "schemaVersion": "00501",
         "schemaName": "X12_00501_837_P"
      }
   ]
   ```

1. En la configuración de mensajes del contrato, deshabilite la validación de EDI desactivando la casilla **Validación EDI** para cada tipo de mensaje o para todos si usa los valores **predeterminados** .

   ![Deshabilitar la validación de todos los tipos de mensaje o de cada uno de ellos](./media/logic-apps-enterprise-integration-x12/x12-disable-validation.png) 

## <a name="connector-reference"></a>Referencia de conectores

Si necesita más detalles técnicos sobre este conector, como las acciones y los límites que se describen en el archivo de Swagger del conector, consulte la [página de referencia del conector](/connectors/x12/).

> [!NOTE]
> En el caso de las aplicaciones lógicas de un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión con la etiqueta ISE de este conector usa los [límites de mensajes B2B del ISE](../logic-apps/logic-apps-limits-and-config.md#b2b-protocol-limits).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre otros [conectores para Logic Apps](../connectors/apis-list.md)
