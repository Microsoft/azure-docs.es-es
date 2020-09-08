---
title: Relocalización general
description: Aprenda a usar la relocalización general para buscar delimitadores cercanos.
author: bucurb
manager: dacoghl
services: azure-spatial-anchors
ms.author: bobuc
ms.date: 09/18/2019
ms.topic: conceptual
ms.service: azure-spatial-anchors
ms.custom: devx-track-csharp
ms.openlocfilehash: 02f5c570b7400266ac648eaa257fb76d26819a40
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89003191"
---
# <a name="coarse-relocalization"></a>Relocalización general

La relocalización general es una característica que proporciona una respuesta inicial a la pregunta: *¿Dónde está mi dispositivo ahora/qué contenido debo observar?* La respuesta no es precisa, pero tiene el formato: *Está cerca de estos delimitadores, intente localizar uno de ellos*.

La relocalización general funciona al asociar varias lecturas de sensores en el dispositivo con la creación y la consulta de los delimitadores. En el caso de los escenarios exteriores, los datos del sensor suelen ser la posición GPS (sistema de posición global) del dispositivo. Cuando el GPS no está disponible o no es confiable (por ejemplo, en interiores), los datos del sensor están formados por los puntos de acceso Wi-Fi y las balizas Bluetooth al alcance. Todos los datos recopilados del sensor contribuyen al mantenimiento de un índice espacial que Azure Spatial Anchors usa para determinar rápidamente los delimitadores que se encuentran a una distancia máxima de unos 100 metros del dispositivo.

La búsqueda rápida de los delimitadores habilitados por la relocalización general simplifica el desarrollo de aplicaciones respaldadas por colecciones a escala mundial de, por ejemplo, millones de delimitadores distribuidos geográficamente. La complejidad de la administración de delimitadores está oculta, lo que le permite concentrarse más en la increíble lógica de su aplicación. Azure Spatial Anchors se encarga de que todo el trabajo pesado de delimitador se realice en segundo plano.

## <a name="collected-sensor-data"></a>Datos del sensor recopilados

Los datos de sensor que puede enviar al servicio de delimitador es uno de los siguientes:

* Posición del GPS: latitud, longitud, altitud.
* Intensidad de la señal de puntos de acceso WiFi en el rango.
* Intensidad de la señal de las balizas Bluetooth en el rango.

En general, su aplicación tendrá que adquirir permisos específicos del dispositivo para acceder a los datos de GPS, WiFi o BLE. Además, algunos de los datos de sensor anteriores no están disponibles por diseño en determinadas plataformas. Para tener en cuenta estas situaciones, la recopilación de datos del sensor es opcional y está desactivada de forma predeterminada.

## <a name="set-up-the-sensor-data-collection"></a>Configurar la recopilación de datos del sensor

Comencemos por crear un proveedor de huellas digitales de sensor y hacer que la sesión sea consciente de ello:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
// Create the sensor fingerprint provider
sensorProvider = new PlatformLocationProvider();

// Create and configure the session
cloudSpatialAnchorSession = new CloudSpatialAnchorSession();

// Inform the session it can access sensor data from your provider
cloudSpatialAnchorSession.LocationProvider = sensorProvider;
```

# <a name="objc"></a>[ObjC](#tab/objc)

```objc
// Create the sensor fingerprint provider
ASAPlatformLocationProvider *sensorProvider;
sensorProvider = [[ASAPlatformLocationProvider alloc] init];

// Create and configure the session
cloudSpatialAnchorSession = [[ASACloudSpatialAnchorSession alloc] init];

// Inform the session it can access sensor data from your provider
cloudSpatialAnchorSession.locationProvider = sensorProvider;
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
// Create the sensor fingerprint provider
var sensorProvider: ASAPlatformLocationProvider?
sensorProvider = ASAPlatformLocationProvider()

// Create and configure the session
cloudSpatialAnchorSession = ASACloudSpatialAnchorSession()

// Inform the session it can access sensor data from your provider
cloudSpatialAnchorSession!.locationProvider = sensorProvider
```

# <a name="java"></a>[Java](#tab/java)

```java
// Create the sensor fingerprint provider
PlatformLocationProvider sensorProvider = new PlatformLocationProvider();

// Create and configure the session
cloudSpatialAnchorSession = new CloudSpatialAnchorSession();

// Inform the session it can access sensor data from your provider
cloudSpatialAnchorSession.setLocationProvider(sensorProvider);
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
// Create the sensor fingerprint provider
std::shared_ptr<PlatformLocationProvider> sensorProvider;
sensorProvider = std::make_shared<PlatformLocationProvider>();

// Create and configure the session
cloudSpatialAnchorSession = std::make_shared<CloudSpatialAnchorSession>();

// Inform the session it can access sensor data from your provider
cloudSpatialAnchorSession->LocationProvider(sensorProvider);
```

# <a name="c-winrt"></a>[C++ WinRT](#tab/cppwinrt)
```cpp
// Create the sensor fingerprint provider
PlatformLocationProvider sensorProvider = PlatformLocationProvider();

// Create and configure the session
cloudSpatialAnchorSession = CloudSpatialAnchorSession();

// Inform the session it can access sensor data from your provider
cloudSpatialAnchorSession.LocationProvider(sensorProvider);
```
---

A continuación, deberá decidir qué sensores desea usar para la relocalización general. Esta decisión es específica de la aplicación que está desarrollando, pero las recomendaciones de la tabla siguiente deben proporcionarle un buen punto de partida:

|                 | Interiores | Exteriores |
|-----------------|---------|----------|
| **GPS**         | Off | Por |
| **WiFi**        | Por | Activado (opcional) |
| **Balizas de BLE** | Activado (opcional con advertencias, consulte a continuación) | Off |

### <a name="enabling-gps"></a>Habilitar GPS

Supongamos que la aplicación ya tiene permiso para acceder a la posición GPS del dispositivo, usted puede configurar los Spatial Anchors de Azure para usarlos:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
sensorProvider.Sensors.GeoLocationEnabled = true;
```

# <a name="objc"></a>[ObjC](#tab/objc)

```objc
ASASensorCapabilities *sensors = locationProvider.sensors;
sensors.geoLocationEnabled = true;
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
let sensors = locationProvider?.sensors
sensors.geoLocationEnabled = true
```

# <a name="java"></a>[Java](#tab/java)

```java
SensorCapabilities sensors = sensorProvider.getSensors();
sensors.setGeoLocationEnabled(true);
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
const std::shared_ptr<SensorCapabilities>& sensors = sensorProvider->Sensors();
sensors->GeoLocationEnabled(true);
```

# <a name="c-winrt"></a>[C++ WinRT](#tab/cppwinrt)

```cpp
SensorCapabilities sensors = sensorProvider.Sensors()
sensors.GeoLocationEnabled(true);
```

---

Al usar el GPS en la aplicación, tenga en cuenta que las lecturas proporcionadas por el hardware suelen ser:

* frecuencia asincrónica y baja (menos de 1 Hz).
* no confiable/ruidoso (en promedio de la desviación estándar de 7-m).

En general, tanto el sistema operativo del dispositivo como los Spatial Anchors de Azure realizarán algún filtrado y extrapolación en la señal GPS sin procesar en un intento de mitigar estos problemas. Este procesamiento adicional requiere un tiempo adicional para la convergencia, por lo que para obtener mejores resultados debe intentar:

* Crear el proveedor de huellas digitales del sensor lo antes posible en su aplicación.
* Mantener el proveedor de huellas digitales del sensor activo entre varias sesiones.
* Compartir el proveedor de huellas digitales del sensor entre varias sesiones.

Si tiene previsto usar el proveedor de huellas digitales del sensor fuera de una sesión de delimitador, asegúrese de iniciarlo antes de solicitar las estimaciones del sensor. Por ejemplo, el código siguiente se encargará de actualizar la posición del dispositivo en el mapa en tiempo real:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
// Game about to start, start tracking the sensors
sensorProvider.Start();

// Game loop
while (m_isRunning)
{
    // Get the GPS estimate
    GeoLocation geoPose = sensorProvider.GetLocationEstimate();

    // Paint it on the map
    drawCircle(
        x: geoPose.Longitude,
        y: geoPose.Latitude,
        radius: geoPose.HorizontalError);
}

// Game ended, no need to track the sensors anymore
sensorProvider.Stop();
```

# <a name="objc"></a>[ObjC](#tab/objc)

```objc
// Game about to start, start tracking the sensors
[sensorProvider start];

// Game loop
while (m_isRunning)
{
    // Get the GPS estimate
    ASAGeoLocation *geoPose = [sensorProvider getLocationEstimate];

    // Paint it on the map
    drawCircle(geoPose.longitude, geoPose.latitude, geoPose.horizontalError);
}

// Game ended, no need to track the sensors anymore
[sensorProvider stop];
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
// Game about to start, start tracking the sensors
sensorProvider?.start()

// Game loop
while m_isRunning
{
    // Get the GPS estimate
    var geoPose: ASAGeoLocation?
    geoPose = sensorProvider?.getLocationEstimate()

    // Paint it on the map
    drawCircle(geoPose.longitude, geoPose.latitude, geoPose.horizontalError)
}

// Game ended, no need to track the sensors anymore
sensorProvider?.stop()
```

# <a name="java"></a>[Java](#tab/java)

```java
// Game about to start, start tracking the sensors
sensorProvider.start();

// Game loop
while (m_isRunning)
{
    // Get the GPS estimate
    GeoLocation geoPose = sensorProvider.getLocationEstimate();

    // Paint it on the map
    drawCircle(geoPose.getLongitude(), geoPose.getLatitude(), geoPose.getHorizontalError());
}

// Game ended, no need to track the sensors anymore
sensorProvider.stop();
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
// Game about to start, start tracking the sensors
sensorProvider->Start();

// Game loop
while (m_isRunning)
{
    // Get the GPS estimate
    std::shared_ptr<GeoLocation> geoPose = sensorProvider->GetLocationEstimate();

    // Paint it on the map
    drawCircle(geoPose->Longitude(), geoPose->Latitude(), geoPose->HorizontalError());
}

// Game ended, no need to track the sensors anymore
sensorProvider->Stop();
```

# <a name="c-winrt"></a>[C++ WinRT](#tab/cppwinrt)

```cpp
// Game about to start, start tracking the sensors
sensorProvider.Start();

// Game loop
while (m_isRunning)
{
    // Get the GPS estimate
    GeoLocation geoPose = sensorProvider.GetLocationEstimate();

    // Paint it on the map
    drawCircle(geoPose.Longitude(), geoPose.Latitude(), geoPose.HorizontalError());
}

// Game ended, no need to track the sensors anymore
sensorProvider.Stop();
```

---

### <a name="enabling-wifi"></a>Habilitar WiFi

Supongamos que la aplicación ya tiene permiso para acceder al estado de WiFi del dispositivo, usted puede configurar los Spatial Anchors de Azure para usarlos:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
sensorProvider.Sensors.WifiEnabled = true;
```

# <a name="objc"></a>[ObjC](#tab/objc)

```objc
ASASensorCapabilities *sensors = locationProvider.sensors;
sensors.wifiEnabled = true;
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
let sensors = locationProvider?.sensors
sensors.wifiEnabled = true
```

# <a name="java"></a>[Java](#tab/java)

```java
SensorCapabilities sensors = sensorProvider.getSensors();
sensors.setWifiEnabled(true);
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
const std::shared_ptr<SensorCapabilities>& sensors = sensorProvider->Sensors();
sensors->WifiEnabled(true);
```

# <a name="c-winrt"></a>[C++ WinRT](#tab/cppwinrt)

```cpp
SensorCapabilities sensors = sensorProvider.Sensors()
sensors.WifiEnabled(true);
```

---

Al usar el WiFi en la aplicación, tenga en cuenta que las lecturas proporcionadas por el hardware suelen ser:

* frecuencia asincrónica y baja (menos de 0.1 Hz).
* posible limitación al nivel de sistema operativo.
* no confiable/ruidoso (en promedio de la desviación estándar de 3-m).

Los Spatial Anchors de Azure intentarán crear un mapa de intensidad de señal WiFi filtrado durante una sesión para intentar mitigar estos problemas. Para obtener mejores resultados, debe intentar:

* crear la sesión antes de colocar el primer delimitador.
* mantener la sesión activa lo máximo posible (es decir, cree todos los delimitadores y la consulta en una sesión).

### <a name="enabling-bluetooth-beacons"></a>Habilite las balizas Bluetooth

Supongamos que la aplicación ya tiene permiso para acceder al estado de Bluetooth del dispositivo, usted puede configurar los Spatial Anchors de Azure para usarlos:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
sensorProvider.Sensors.BluetoothEnabled = true;
```

# <a name="objc"></a>[ObjC](#tab/objc)

```objc
ASASensorCapabilities *sensors = locationProvider.sensors;
sensors.bluetoothEnabled = true;
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
let sensors = locationProvider?.sensors
sensors.bluetoothEnabled = true
```

# <a name="java"></a>[Java](#tab/java)

```java
SensorCapabilities sensors = sensorProvider.getSensors();
sensors.setBluetoothEnabled(true);
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
const std::shared_ptr<SensorCapabilities>& sensors = sensorProvider->Sensors();
sensors->BluetoothEnabled(true);
```

# <a name="c-winrt"></a>[C++ WinRT](#tab/cppwinrt)

```cpp
SensorCapabilities sensors = sensorProvider.Sensors();
sensors.BluetoothEnabled(true);
```

---

Las balizas suelen ser dispositivos versátiles en los que se puede configurar todo, incluidos los UUID y las direcciones MAC. Esta flexibilidad puede ser problemática para Azure Spatial Anchors, que considera que las balizas se identifican de forma única por sus UUID. Si no se garantiza esta singularidad, lo más probable es que se produzcan agujeros de gusano espaciales. Para obtener mejores resultados, debe intentar:

* Asignar UUID únicos a las balizas.
* implementarlos, normalmente en un patrón normal, como una cuadrícula.
* Pasar la lista de UUID de baliza única al proveedor de huellas digitales del sensor:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
sensorProvider.Sensors.KnownBeaconProximityUuids = new[]
{
    "22e38f1a-c1b3-452b-b5ce-fdb0f39535c1",
    "a63819b9-8b7b-436d-88ec-ea5d8db2acb0",
    . . .
};
```

# <a name="objc"></a>[ObjC](#tab/objc)

```objc
NSArray *uuids = @[@"22e38f1a-c1b3-452b-b5ce-fdb0f39535c1", @"a63819b9-8b7b-436d-88ec-ea5d8db2acb0"];

ASASensorCapabilities *sensors = locationProvider.sensors;
sensors.knownBeaconProximityUuids = uuids;
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
let uuids = [String]()
uuids.append("22e38f1a-c1b3-452b-b5ce-fdb0f39535c1")
uuids.append("a63819b9-8b7b-436d-88ec-ea5d8db2acb0")

let sensors = locationProvider?.sensors
sensors.knownBeaconProximityUuids = uuids
```

# <a name="java"></a>[Java](#tab/java)

```java
String uuids[] = new String[2];
uuids[0] = "22e38f1a-c1b3-452b-b5ce-fdb0f39535c1";
uuids[1] = "a63819b9-8b7b-436d-88ec-ea5d8db2acb0";

SensorCapabilities sensors = sensorProvider.getSensors();
sensors.setKnownBeaconProximityUuids(uuids);
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
std::vector<std::string> uuids;
uuids.push_back("22e38f1a-c1b3-452b-b5ce-fdb0f39535c1");
uuids.push_back("a63819b9-8b7b-436d-88ec-ea5d8db2acb0");

const std::shared_ptr<SensorCapabilities>& sensors = sensorProvider->Sensors();
sensors->KnownBeaconProximityUuids(uuids);
```

# <a name="c-winrt"></a>[C++ WinRT](#tab/cppwinrt)

```cpp
std::vector<winrt::hstring> uuids;
uuids.emplace_back("22e38f1a-c1b3-452b-b5ce-fdb0f39535c1");
uuids.emplace_back("a63819b9-8b7b-436d-88ec-ea5d8db2acb0");

SensorCapabilities sensors = sensorProvider.Sensors();
sensors.KnownBeaconProximityUuids(uuids);
```

---

Azure Spatial Anchors solo realizará un seguimiento de las balizas Bluetooth que se encuentran en la lista de UUID conocidos de proximidad de las balizas. Las balizas maliciosas programadas para tener UUID en la lista de permitidos aún pueden afectar negativamente a la calidad del servicio. Por ese motivo, solo debe usar balizas en espacios seleccionados en los que pueda controlar su implementación.

## <a name="querying-with-sensor-data"></a>Consulta con datos de sensor

Una vez que haya creado los delimitadores con los datos del sensor asociados, puede empezar a recuperarlos con las lecturas del sensor que el dispositivo ha proporcionado. Ya no es necesario proporcionar el servicio con una lista de los anclajes conocidos que espera encontrar; en su lugar, simplemente deje que el servicio conozca la ubicación del dispositivo, tal y como lo notifican los sensores incorporados. Después, Azure Spatial Anchors descifrará el conjunto de delimitadores cerca del dispositivo e intentará relacionarlos visualmente.

Para que las consultas utilicen los datos del sensor, empiece por crear un criterio de proximidad de dispositivo:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
NearDeviceCriteria nearDeviceCriteria = new NearDeviceCriteria();

// Choose a maximum exploration distance between your device and the returned anchors
nearDeviceCriteria.DistanceInMeters = 5;

// Cap the number of anchors returned
nearDeviceCriteria.MaxResultCount = 25;

anchorLocateCriteria = new AnchorLocateCriteria();
anchorLocateCriteria.NearDevice = nearDeviceCriteria;
```

# <a name="objc"></a>[ObjC](#tab/objc)

```objc
ASANearDeviceCriteria *nearDeviceCriteria = [[ASANearDeviceCriteria alloc] init];

// Choose a maximum exploration distance between your device and the returned anchors
nearDeviceCriteria.distanceInMeters = 5.0f;

// Cap the number of anchors returned
nearDeviceCriteria.maxResultCount = 25;

ASAAnchorLocateCriteria *anchorLocateCriteria = [[ASAAnchorLocateCriteria alloc] init];
anchorLocateCriteria.nearDevice = nearDeviceCriteria;
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
let nearDeviceCriteria = ASANearDeviceCriteria()!

// Choose a maximum exploration distance between your device and the returned anchors
nearDeviceCriteria.distanceInMeters = 5.0

// Cap the number of anchors returned
nearDeviceCriteria.maxResultCount = 25

let anchorLocateCriteria = ASAAnchorLocateCriteria()!
anchorLocateCriteria.nearDevice = nearDeviceCriteria
```

# <a name="java"></a>[Java](#tab/java)

```java
NearDeviceCriteria nearDeviceCriteria = new NearDeviceCriteria();

// Choose a maximum exploration distance between your device and the returned anchors
nearDeviceCriteria.setDistanceInMeters(5.0f);

// Cap the number of anchors returned
nearDeviceCriteria.setMaxResultCount(25);

AnchorLocateCriteria anchorLocateCriteria = new AnchorLocateCriteria();
anchorLocateCriteria.setNearDevice(nearDeviceCriteria);
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
auto nearDeviceCriteria = std::make_shared<NearDeviceCriteria>();

// Choose a maximum exploration distance between your device and the returned anchors
nearDeviceCriteria->DistanceInMeters(5.0f);

// Cap the number of anchors returned
nearDeviceCriteria->MaxResultCount(25);

auto anchorLocateCriteria = std::make_shared<AnchorLocateCriteria>();
anchorLocateCriteria->NearDevice(nearDeviceCriteria);
```

# <a name="c-winrt"></a>[C++ WinRT](#tab/cppwinrt)

```cpp
NearDeviceCriteria nearDeviceCriteria = NearDeviceCriteria();

// Choose a maximum exploration distance between your device and the returned anchors
nearDeviceCriteria.DistanceInMeters(5.0f);

// Cap the number of anchors returned
nearDeviceCriteria.MaxResultCount(25);

// Set the session's locate criteria
anchorLocateCriteria = AnchorLocateCriteria();
anchorLocateCriteria.NearDevice(nearDeviceCriteria);
```

---

El parámetro `DistanceInMeters` controla hasta qué punto exploraremos el gráfico delimitador para recuperar el contenido. Supongamos, por ejemplo, que ha rellenado algún espacio con delimitadores a una densidad constante de 2 cada medidor. Además, la cámara del dispositivo está observando un solo delimitador y el servicio lo ha encontrado correctamente. Lo más probable es que esté interesado en recuperar todos los anclajes que haya colocado cerca en lugar del único delimitador que está observando actualmente. Suponiendo que los delimitadores que ha colocado están conectados en un gráfico, el servicio puede recuperar todos los delimitadores cercanos automáticamente siguiendo los bordes del gráfico. `DistanceInMeters` controla la cantidad de recorrido del gráfico realizado, se le darán todos los delimitadores conectados al que ha localizado, que están más cerca de `DistanceInMeters`.

Tenga en cuenta que los valores grandes de `MaxResultCount` pueden afectar de manera negativa al rendimiento. Establézcalo en un valor razonable para su aplicación.

Por último, debe indicar a la sesión que use la búsqueda basada en sensor:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
cloudSpatialAnchorSession.CreateWatcher(anchorLocateCriteria);
```

# <a name="objc"></a>[ObjC](#tab/objc)

```objc
[cloudSpatialAnchorSession createWatcher:anchorLocateCriteria];
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
cloudSpatialAnchorSession!.createWatcher(anchorLocateCriteria)
```

# <a name="java"></a>[Java](#tab/java)

```java
cloudSpatialAnchorSession.createWatcher(anchorLocateCriteria);
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
cloudSpatialAnchorSession->CreateWatcher(anchorLocateCriteria);
```

# <a name="c-winrt"></a>[C++ WinRT](#tab/cppwinrt)

```cpp
cloudSpatialAnchorSession.CreateWatcher(anchorLocateCriteria);
```

---

## <a name="expected-results"></a>Resultados esperados

Los dispositivos GPS de nivel de consumidor suelen ser bastante imprecisos. Un estudio de [Zandenbergen y Barbeau (2011)][6] informa que la precisión media de los teléfonos móviles con GPS asistido (A-GPS) es de alrededor de 7 metros, ¡un valor bastante grande para ser ignorado! Para tener en cuenta estos errores de medición, el servicio trata los delimitadores como distribuciones de probabilidad en el espacio GPS. Como tal, un delimitador es ahora la región del espacio que más probablemente (es decir, con más del 95% de confianza) contiene su posición GPS verdadera y desconocida.

La misma razón se aplica cuando se consulta con GPS. El dispositivo se representa como otra región de confianza espacial en torno a su verdadera posición GPS desconocida. Descubrir los delimitadores cercanos se traduce en simplemente encontrar los delimitadores con regiones de confianza *lo suficientemente cercanas* a la región de confianza del dispositivo, como se ilustra en la imagen siguiente:

![Selección de candidatos de delimitador con GPS](media/coarse-reloc-gps-separation-distance.png)

La precisión de la señal del GPS, tanto en la creación del delimitador como en las consultas, tiene una gran influencia sobre el conjunto de delimitadores devueltos. Por el contrario, las consultas basadas en WiFi o balizas considerarán todos los delimitadores que tienen al menos un punto de acceso o señalización en común con la consulta. En ese sentido, el resultado de una consulta basada en WiFi o balizas está determinada principalmente por el intervalo físico de los puntos de acceso, las señalizaciones y los obstáculos ambientales.

En la siguiente tabla se calcula el espacio de búsqueda esperado para cada tipo de sensor:

| Sensor      | Radio del espacio de búsqueda (aproximado) | Detalles |
|-------------|:-------:|---------|
| GPS         | 20 m - 30 m | Determinado por la incertidumbre del GPS entre otros factores. Los números informados se estiman para la precisión media de GPS de los teléfonos móviles con A-GPS, que es de 7 metros. |
| Wi-Fi        | 50 m - 100 m | Determinado por el intervalo de puntos de acceso inalámbrico. Depende de la frecuencia, la intensidad del transmisor, los obstáculos físicos, las interferencias, etc. |
| Las balizas de BLE |  70 m | Determinado por el intervalo de la señalización. Depende de la frecuencia, la intensidad de transmisión, los obstáculos físicos, las interferencias, etc. |

## <a name="per-platform-support"></a>Soporte por plataforma

En la tabla siguiente se resumen los datos de sensor recopilados en cada una de las plataformas admitidas, junto con las advertencias específicas de la plataforma:

|                 | HoloLens | Android | iOS |
|-----------------|----------|---------|-----|
| **GPS**         | N/D | Compatible con las API [LocationManager][3] (tanto GPS como NETWORK) | Compatible con las API [CLLocationManager][4] |
| **WiFi**        | Compatible con una frecuencia de aproximadamente un escaneo cada 3 segundos | Compatible. Comenzando con el nivel 28 de API, los escaneos WiFi se aceleran a 4 llamadas cada 2 minutos. Desde Android 10, la limitación se puede deshabilitar desde el menú de configuración del Desarrollador. Para más información, consulte la [documentación de Android][5]. | N/A: sin API pública |
| **Balizas de BLE** | Limitado a [Eddystone][1] y [iBeacon][2] | Limitado a [Eddystone][1] y [iBeacon][2] | Limitado a [Eddystone][1] y [iBeacon][2] |

## <a name="next-steps"></a>Pasos siguientes

Use la relocalización general en una aplicación.

> [!div class="nextstepaction"]
> [Unity](../how-tos/set-up-coarse-reloc-unity.md)

> [!div class="nextstepaction"]
> [Objective-C](../how-tos/set-up-coarse-reloc-objc.md)

> [!div class="nextstepaction"]
> [Swift](../how-tos/set-up-coarse-reloc-swift.md)

> [!div class="nextstepaction"]
> [Java](../how-tos/set-up-coarse-reloc-java.md)

> [!div class="nextstepaction"]
> [C++/NDK](../how-tos/set-up-coarse-reloc-cpp-ndk.md)

> [!div class="nextstepaction"]
> [C++/WinRT](../how-tos/set-up-coarse-reloc-cpp-winrt.md)

<!-- Reference links in article -->
[1]: https://developers.google.com/beacons/eddystone
[2]: https://developer.apple.com/ibeacon/
[3]: https://developer.android.com/reference/android/location/LocationManager
[4]: https://developer.apple.com/documentation/corelocation/cllocationmanager?language=objc
[5]: https://developer.android.com/guide/topics/connectivity/wifi-scan
[6]: https://www.cambridge.org/core/journals/journal-of-navigation/article/positional-accuracy-of-assisted-gps-data-from-highsensitivity-gpsenabled-mobile-phones/E1EE20CD1A301C537BEE8EC66766B0A9
