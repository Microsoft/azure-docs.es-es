---
title: 'Azure Event Hubs: intercambio de eventos mediante distintos protocolos'
description: En este artículo se muestra cómo los consumidores y los productores que utilizan distintos protocolos (AMQP, Apache Kafka y HTTPS) pueden intercambiar eventos cuando se usa Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 6cdc4b9040f314b4ec41f84cc7436f0f2e3d6af6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87002512"
---
# <a name="exchange-events-between-consumers-and-producers-that-use-different-protocols-amqp-kafka-and-https"></a>Intercambio de eventos entre consumidores y productores que utilizan distintos protocolos: AMQP, Kafka y HTTPS
Azure Event Hubs admite tres protocolos para consumidores y productores: AMQP, Kafka y HTTPS. Cada uno de estos protocolos tiene su propia manera de representar un mensaje, por lo que surge la pregunta siguiente: si una aplicación envía eventos a un centro de eventos con un protocolo y los consume con otro, ¿qué aspecto tienen las distintas partes y valores del evento cuando llegan al consumidor? En este artículo se describen los procedimientos recomendados para el productor y el consumidor a fin de garantizar que la aplicación de consumo interprete correctamente los valores de un evento.

La información de este artículo abarca específicamente estos clientes, con las versiones indicadas utilizadas para desarrollar los fragmentos de código:

* Cliente Java de Kafka (versión 1.1.1 en https://www.mvnrepository.com/artifact/org.apache.kafka/kafka-clients)
* Cliente de Microsoft Azure Event Hubs para Java (versión 1.1.0 en https://github.com/Azure/azure-event-hubs-java)
* Cliente de Microsoft Azure Event Hubs para .NET (versión 2.1.0 en https://github.com/Azure/azure-event-hubs-dotnet)
* Microsoft Azure Service Bus (versión 5.0.0 en https://www.nuget.org/packages/WindowsAzure.ServiceBus)
* HTTPS (solo es compatible con productores)

Otros clientes de AMQP pueden comportarse de forma ligeramente diferente. AMQP tiene un sistema de tipos bien definidos, pero los aspectos específicos de serializar los tipos específicos de lenguaje hacia y desde ese sistema de tipos depende del cliente, igual que de cómo el cliente proporciona acceso a las partes de un mensaje AMQP.

## <a name="event-body"></a>Cuerpo del evento
Todos los clientes de Microsoft AMQP representan el cuerpo del evento como un contenedor de bytes sin interpretar. Una aplicación de producción pasa una secuencia de bytes al cliente y una aplicación de consumo recibe esa misma secuencia desde el cliente. La interpretación de la secuencia de bytes se produce dentro del código de aplicación.

Cuando se envía un evento a través de HTTPS, el cuerpo del evento es el contenido publicado, que también se trata como bytes sin interpretar. Es fácil lograr el mismo estado en un productor o un consumidor Kafka empleando los elementos ByteArraySerializer y ByteArrayDeserializer proporcionados, tal como se muestra en el código siguiente:

### <a name="kafka-byte-producer"></a>Productor Kafka byte[]

```java
final Properties properties = new Properties();
// add other properties
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, ByteArraySerializer.class.getName());

final KafkaProducer<Long, byte[]> producer = new KafkaProducer<Long, byte[]>(properties);

final byte[] eventBody = new byte[] { 0x01, 0x02, 0x03, 0x04 };
ProducerRecord<Long, byte[]> pr =
    new ProducerRecord<Long, byte[]>(myTopic, myPartitionId, myTimeStamp, eventBody);
```

### <a name="kafka-byte-consumer"></a>Consumidor Kafka byte[]
```java
final Properties properties = new Properties();
// add other properties
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, ByteArrayDeserializer.class.getName());

final KafkaConsumer<Long, byte[]> consumer = new KafkaConsumer<Long, byte[]>(properties);

ConsumerRecord<Long, byte[]> cr = /* receive event */
// cr.value() is a byte[] with values { 0x01, 0x02, 0x03, 0x04 }
```

Este código crea una canalización de bytes transparente entre las dos mitades de la aplicación y permite al desarrollador de la aplicación serializar y deserializar manualmente en cualquier forma deseada, incluida la toma de decisiones de deserialización en tiempo de ejecución, por ejemplo, según el tipo o la información del remitente en las propiedades del conjunto de usuarios del evento.

Las aplicaciones que tienen un solo tipo de cuerpo de evento fijo pueden utilizar otros serializadores y deserializadores de Kafka para convertir datos de forma transparente. Por ejemplo, considere una aplicación que utiliza JSON. La construcción y la interpretación de la cadena JSON se produce en el nivel de aplicación. En el nivel de Event Hubs, el cuerpo del evento siempre es una cadena, una secuencia de bytes que representa los caracteres de la codificación UTF-8. En este caso, el productor o consumidor Kafka puede aprovechar el elemento StringSerializer o StringDeserializer proporcionado como se muestra en el código siguiente:

### <a name="kafka-utf-8-string-producer"></a>Productor de cadena UTF-8 de Kafka
```java
final Properties properties = new Properties();
// add other properties
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

final KafkaProducer<Long, String> producer = new KafkaProducer<Long, String>(properties);

final String exampleJson = "{\"name\":\"John\", \"number\":9001}";
ProducerRecord<Long, String> pr =
    new ProducerRecord<Long, String>(myTopic, myPartitionId, myTimeStamp, exampleJson);
```

### <a name="kafka-utf-8-string-consumer"></a>Consumidor de cadena UTF-8 de Kafka
```java
final Properties properties = new Properties();
// add other properties
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

final KafkaConsumer<Long, String> consumer = new KafkaConsumer<Long, String>(properties);

ConsumerRecord<Long, Bytes> cr = /* receive event */
final String receivedJson = cr.value();
```

Para el lado AMQP, Java y .NET proporcionan maneras integradas para convertir cadenas a o desde secuencias de bytes UTF-8. Los clientes de Microsoft AMQP representan los eventos como una clase denominada EventData. En los ejemplos siguientes se muestra cómo serializar una cadena UTF-8 en un cuerpo de evento EventData con un productor AMQP y cómo deserializar un cuerpo de evento EventData en una cadena UTF-8 en un consumidor AMQP.

### <a name="java-amqp-utf-8-string-producer"></a>Productor de cadena UTF-8 de Java AMQP
```java
final String exampleJson = "{\"name\":\"John\", \"number\":9001}";
final EventData ed = EventData.create(exampleJson.getBytes(StandardCharsets.UTF_8));
```

### <a name="java-amqp-utf-8-string-consumer"></a>Consumidor de cadena UTF-8 de Java AMQP
```java
EventData ed = /* receive event */
String receivedJson = new String(ed.getBytes(), StandardCharsets.UTF_8);
```

### <a name="c-net-utf-8-string-producer"></a>Productor de cadena UTF-8 de C# .NET
```csharp
string exampleJson = "{\"name\":\"John\", \"number\":9001}";
EventData working = new EventData(Encoding.UTF8.GetBytes(exampleJson));
```

### <a name="c-net-utf-8-string-consumer"></a>Consumidor de cadena UTF-8 de C# .NET
```csharp
EventData ed = /* receive event */

// getting the event body bytes depends on which .NET client is used
byte[] bodyBytes = ed.Body.Array;  // Microsoft Azure Event Hubs Client for .NET
// byte[] bodyBytes = ed.GetBytes(); // Microsoft Azure Service Bus

string receivedJson = Encoding.UTF8.GetString(bodyBytes);
```

Como Kafka es código abierto, el desarrollador de aplicaciones puede inspeccionar la implementación de cualquier serializador o deserializador e implementar código, que produzca o consuma una secuencia de bytes compatible en el lado AMQP.

## <a name="event-user-properties"></a>Propiedades de usuario de eventos

Se pueden establecer y recuperar propiedades del conjunto de usuarios de ambos clientes AMQP (en los clientes de Microsoft AMQP se llaman propiedades) y Kafka (donde se llaman encabezados). Los remitentes HTTPS pueden establecer las propiedades de usuario en un evento proporcionándolas como encabezados HTTP en la operación POST. Sin embargo, Kafka trata los cuerpos de los eventos y los valores de encabezado de los eventos como secuencias de bytes. Mientras que en los clientes AMQP, los valores de propiedades tienen tipos, que se comunican mediante la codificación de los valores de propiedad según el sistema de tipos AMQP.

HTTPS es un caso especial. En el punto de envío, todos los valores de propiedad son texto UTF-8. El servicio Event Hubs hace una interpretación limitada para convertir los valores de propiedad adecuados a enteros con signo de 32 y 64 bits, números de punto flotante de 64 bits y booleanos codificados con AMQP. Cualquier valor de propiedad que no se ajuste a uno de esos tipos se trata como una cadena.

La combinación de estos enfoques para escribir la propiedad significa que un consumidor Kafka ve la secuencia de bytes codificada con AMQP sin formato, incluida la información de tipo de AMQP. Mientras que un consumidor AMQP ve la secuencia de bytes sin tipo enviada por el productor Kafka, que la aplicación debe interpretar.

Para los consumidores Kafka que reciben propiedades de los productores AMQP o HTTPS, utilice la clase AmqpDeserializer, que está modelada según los demás deserializadores del ecosistema de Kafka. Interpreta la información de tipo en las secuencias de bytes codificadas con AMQP para deserializar los bytes de datos en un tipo Java.

Como procedimiento recomendado, incluya una propiedad en los mensajes enviados a través de AMQP o HTTPS. El consumidor Kafka puede usarla para determinar si los valores de encabezado necesitan deserialización de AMQP. El valor de la propiedad no es importante. Solo necesita un nombre conocido que el consumidor Kafka pueda buscar en la lista de encabezados y ajustar su comportamiento en consecuencia.

### <a name="amqp-to-kafka-part-1-create-and-send-an-event-in-c-net-with-properties"></a>AMQP a Kafka, parte 1: crear y enviar un evento en C# (.NET) con propiedades
```csharp
// Create an event with properties "MyStringProperty" and "MyIntegerProperty"
EventData working = new EventData(Encoding.UTF8.GetBytes("an event body"));
working.Properties.Add("MyStringProperty", "hello");
working.Properties.Add("MyIntegerProperty", 1234);

// BEST PRACTICE: include a property which indicates that properties will need AMQP deserialization
working.Properties.Add("AMQPheaders", 0);
```

### <a name="amqp-to-kafka-part-2-use-amqpdeserializer-to-deserialize-those-properties-in-a-kafka-consumer"></a>AMQP a Kafka, parte 2: usar AmqpDeserializer para deserializar las propiedades de un consumidor Kafka
```java
final AmqpDeserializer amqpDeser = new AmqpDeserializer();

ConsumerRecord<Long, Bytes> cr = /* receive event */
final Header[] headers = cr.headers().toArray();

final Header headerNamedMyStringProperty = /* find header with key "MyStringProperty" */
final Header headerNamedMyIntegerProperty = /* find header with key "MyIntegerProperty" */
final Header headerNamedAMQPheaders = /* find header with key "AMQPheaders", or null if not found */

// BEST PRACTICE: detect whether AMQP deserialization is needed
if (headerNamedAMQPheaders != null) {
    // The deserialize() method requires no prior knowledge of a property's type.
    // It returns Object and the application can check the type and perform a cast later.
    Object propertyOfUnknownType = amqpDeser.deserialize("topicname", headerNamedMyStringProperty.value());
    if (propertyOfUnknownType instanceof String) {
        final String propertyString = (String)propertyOfUnknownType;
        // do work here
    }
    propertyOfUnknownType = amqpDeser.deserialize("topicname", headerNamedMyIntegerProperty.value());
    if (propertyOfUnknownType instanceof Integer) {
        final Integer propertyInt = (Integer)propertyOfUnknownType;
        // do work here
    }
} else {
    /* event sent via Kafka, interpret header values the Kafka way */
}
```

Si la aplicación conoce el tipo esperado para una propiedad, existen métodos de deserialización que no requieren una conversión más tarde, pero generan un error si la propiedad no es del tipo esperado.

### <a name="amqp-to-kafka-part-3-a-different-way-of-using-amqpdeserializer-in-a-kafka-consumer"></a>AMQP a Kafka, parte 3: una manera diferente de usar AmqpDeserializer en un consumidor Kafka
```java
// BEST PRACTICE: detect whether AMQP deserialization is needed
if (headerNamedAMQPheaders != null) {
    // Property "MyStringProperty" is expected to be of type string.
    try {
        final String propertyString = amqpDeser.deserializeString(headerNamedMyStringProperty.value());
        // do work here
    }
    catch (IllegalArgumentException e) {
        // property was not a string
    }

    // Property "MyIntegerProperty" is expected to be a signed integer type.
    // The method returns long because long can represent the value range of all AMQP signed integer types.
    try {
        final long propertyLong = amqpDeser.deserializeSignedInteger(headerNamedMyIntegerProperty.value());
        // do work here
    }
    catch (IllegalArgumentException e) {
        // property was not a signed integer
    }
} else {
    /* event sent via Kafka, interpret header values the Kafka way */
}
```

Ir en la otra dirección es más complicado, porque el consumidor AMQP siempre ve los encabezados establecidos por un productor Kafka como bytes sin formato (tipo org.apache.qpid.proton.amqp.Binary para el cliente de Microsoft Azure Event Hubs para Java, o `System.Byte[]` para los clientes .NET AMQP de Microsoft). La ruta de acceso más fácil es usar uno de los serializadores proporcionados con Kafka para generar los bytes para los valores de encabezado en el lado del productor Kafka y, a continuación, escribir un código de deserialización compatible en el lado del consumidor AMQP.

Al igual que para AMQP a Kafka, el procedimiento recomendado es incluir una propiedad en los mensajes enviados a través de Kafka. El consumidor AMQP puede usar la propiedad para determinar si los valores de encabezado necesitan deserialización. El valor de la propiedad no es importante. Solo necesita un nombre conocido que el consumidor AMQP pueda buscar en la lista de encabezados y ajustar su comportamiento en consecuencia. Si no se puede cambiar el productor Kafka, también es posible que la aplicación de consumo compruebe si el valor de la propiedad es de tipo binario o de bytes, e intente la deserialización en función del tipo.

### <a name="kafka-to-amqp-part-1-create-and-send-an-event-from-kafka-with-properties"></a>Kafka a AMQP, parte 1: crear y enviar un evento desde Kafka con propiedades
```java
final String topicName = /* topic name */
final ProducerRecord<Long, String> pr = new ProducerRecord<Long, String>(topicName, /* other arguments */);
final Headers h = pr.headers();

// Set headers using Kafka serializers
IntegerSerializer intSer = new IntegerSerializer();
h.add("MyIntegerProperty", intSer.serialize(topicName, 1234));

LongSerializer longSer = new LongSerializer();
h.add("MyLongProperty", longSer.serialize(topicName, 5555555555L));

ShortSerializer shortSer = new ShortSerializer();
h.add("MyShortProperty", shortSer.serialize(topicName, (short)22222));

FloatSerializer floatSer = new FloatSerializer();
h.add("MyFloatProperty", floatSer.serialize(topicName, 1.125F));

DoubleSerializer doubleSer = new DoubleSerializer();
h.add("MyDoubleProperty", doubleSer.serialize(topicName, Double.MAX_VALUE));

StringSerializer stringSer = new StringSerializer();
h.add("MyStringProperty", stringSer.serialize(topicName, "hello world"));

// BEST PRACTICE: include a property which indicates that properties will need deserialization
h.add("RawHeaders", intSer.serialize(0));
```

### <a name="kafka-to-amqp-part-2-manually-deserialize-those-properties-in-c-net"></a>Kafka a AMQP, parte 2: deserializar manualmente esas propiedades en C# (.NET)
```csharp
EventData ed = /* receive event */

// BEST PRACTICE: detect whether manual deserialization is needed
if (ed.Properties.ContainsKey("RawHeaders"))
{
    // Kafka serializers send bytes in big-endian order, whereas .NET on x86/x64 is little-endian.
    // Therefore it is frequently necessary to reverse the bytes before further deserialization.

    byte[] rawbytes = ed.Properties["MyIntegerProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    int myIntegerProperty = BitConverter.ToInt32(rawbytes, 0);

    rawbytes = ed.Properties["MyLongProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    long myLongProperty = BitConverter.ToInt64(rawbytes, 0);

    rawbytes = ed.Properties["MyShortProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    short myShortProperty = BitConverter.ToInt16(rawbytes, 0);

    rawbytes = ed.Properties["MyFloatProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    float myFloatProperty = BitConverter.ToSingle(rawbytes, 0);

    rawbytes = ed.Properties["MyDoubleProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    double myDoubleProperty = BitConverter.ToDouble(rawbytes, 0);

    rawbytes = ed.Properties["MyStringProperty"] as System.Byte[];
string myStringProperty = Encoding.UTF8.GetString(rawbytes);
}
```

### <a name="kafka-to-amqp-part-3-manually-deserialize-those-properties-in-java"></a>Kafka a AMQP, parte 3: deserializar manualmente esas propiedades en Java
```java
final EventData ed = /* receive event */

// BEST PRACTICE: detect whether manual deserialization is needed
if (ed.getProperties().containsKey("RawHeaders")) {
    byte[] rawbytes =
        ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyIntegerProperty")).getArray();
    int myIntegerProperty = 0;
    for (byte b : rawbytes) {
        myIntegerProperty <<= 8;
        myIntegerProperty |= ((int)b & 0x00FF);
    }

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyLongProperty")).getArray();
    long myLongProperty = 0;
    for (byte b : rawbytes) {
        myLongProperty <<= 8;
        myLongProperty |= ((long)b & 0x00FF);
    }

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyShortProperty")).getArray();
    short myShortProperty = (short)rawbytes[0];
    myShortProperty <<= 8;
    myShortProperty |= ((short)rawbytes[1] & 0x00FF);

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyFloatProperty")).getArray();
    int intbits = 0;
    for (byte b : rawbytes) {
        intbits <<= 8;
        intbits |= ((int)b & 0x00FF);
    }
    float myFloatProperty = Float.intBitsToFloat(intbits);

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyDoubleProperty")).getArray();
    long longbits = 0;
    for (byte b : rawbytes) {
        longbits <<= 8;
        longbits |= ((long)b & 0x00FF);
    }
    double myDoubleProperty = Double.longBitsToDouble(longbits);

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyStringProperty")).getArray();
String myStringProperty = new String(rawbytes, StandardCharsets.UTF_8);
}
```

## <a name="next-steps"></a>Pasos siguientes
En este artículo se muestra cómo hacer streaming a instancias de Event Hubs sin cambiar los clientes del protocolo o ejecutar sus propios clústeres. Para obtener más información acerca de Event Hubs y Event Hubs para Kafka, consulte los artículos siguientes:  

* [Más información sobre Events Hubs](./event-hubs-about.md)
* [Información sobre Event Hubs para Kafka](event-hubs-for-kafka-ecosystem-overview.md)
* [Más ejemplos en GitHub de Event Hubs para Kafka](https://github.com/Azure/azure-event-hubs-for-kafka)
* Use [MirrorMaker](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330) para [transmitir eventos desde Kafka local a una instancia de Event Hubs en la nube](event-hubs-kafka-mirror-maker-tutorial.md).
* Aprenda a transmitir en una instancia de Event Hubs mediante [aplicaciones nativas de Kafka](event-hubs-quickstart-kafka-enabled-event-hubs.md), [Apache Flink](event-hubs-kafka-flink-tutorial.md) o [Akka Streams](event-hubs-kafka-akka-streams-tutorial.md).
