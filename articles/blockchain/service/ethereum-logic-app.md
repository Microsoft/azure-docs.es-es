---
title: 'Uso del conector Ethereum Blockchain con Azure Logic Apps: Azure Blockchain Service'
description: Uso del conector Ethereum Blockchain con Azure Logic Apps para desencadenar funciones de contrato inteligente y responder a eventos de contrato inteligente.
ms.date: 10/14/2019
ms.topic: how-to
ms.reviewer: chrisseg
ms.openlocfilehash: 61dbda7cd7f486c7a8d838084875b34803833502
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077036"
---
# <a name="use-the-ethereum-blockchain-connector-with-azure-logic-apps"></a>Uso del conector Ethereum Blockchain con Azure Logic Apps

Use el [conector Ethereum Blockchain](/connectors/blockchainethereum/) con [Azure Logic Apps](../../logic-apps/index.yml) para realizar acciones y responder a eventos de contrato inteligente. Supongamos que desea crear un microservicio basado en REST que devuelva información de un libro de contabilidad de cadena de bloques. Mediante el uso de una aplicación lógica, puede aceptar solicitudes HTTP que consulten la información almacenada en un libro de contabilidad de cadena de bloques.

## <a name="prerequisites"></a>Requisitos previos

Complete el requisito previo opcional de [Inicio rápido: Uso de Visual Studio Code para conectarse a una red del consorcio de Azure Blockchain Service](connect-vscode.md). El inicio rápido le guía por la instalación de [Azure Blockchain Development Kit para Ethereum](https://marketplace.visualstudio.com/items?itemName=AzBlockchain.azure-blockchain) y la configuración del entorno de desarrollo de la cadena de bloques.

## <a name="create-a-logic-app"></a>Creación de una aplicación lógica

Azure Logic Apps le ayuda a programar y automatizar procesos empresariales y flujos de trabajo cuando necesita integrar sistemas y servicios. En primer lugar, debe crear una lógica que use el conector Ethereum Blockchain.

1. En [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso** > **Integración** > **Logic App**.
1. En **Crear aplicación lógica**, indique los detalles sobre dónde va a crear la aplicación lógica. Seleccione **Crear** cuando haya terminado.

    Para más información sobre la creación de aplicaciones lógicas, consulte [Creación de flujos de trabajo automatizados con Azure Logic Apps](../../logic-apps/quickstart-create-first-logic-app-workflow.md).

1. Después de que Azure implemente la aplicación, seleccione el recurso de la aplicación lógica.
1. En el Diseñador de aplicaciones lógicas, en **Plantillas**, seleccione **Aplicación lógica en blanco**.

Cada aplicación lógica debe comenzar con un desencadenador, que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador se activa, el motor de Logic Apps crea una instancia de aplicación lógica que inicia y ejecuta el flujo de trabajo.

El conector Ethereum Blockchain incluye un desencadenador y varias acciones. El desencadenador o la acción que se usa depende del escenario.

Si el flujo de trabajo:

* Se desencadena cuando se produce un evento en la cadena de bloques: [Uso del desencadenador de eventos](#use-the-event-trigger).
* Consulta o implementa un contrato inteligente: [Uso de acciones](#use-actions).
* Sigue un escenario común: [Generación de un flujo de trabajo mediante el kit de desarrollo](#generate-a-workflow).

## <a name="use-the-event-trigger"></a>Uso del desencadenador de eventos

Use desencadenadores de eventos de Ethereum Blockchain cuando quiera que se ejecute una aplicación lógica después de que se produzca un evento del contrato inteligente. Por ejemplo, puede que desee enviar un correo electrónico cuando se llame a una función del contrato inteligente.

1. En el Diseñador de aplicaciones lógicas, seleccione el conector Ethereum Blockchain.
1. En la pestaña **Triggers** (Desencadenadores), seleccione **When a smart contract event occurs** (Cuando se produce un evento de contrato inteligente).
1. Cambie o [cree una conexión de API](#create-an-api-connection) a Azure Blockchain Service.
1. Escriba los detalles sobre el contrato inteligente en el que desea buscar eventos.

    ![Diseñador de aplicaciones lógicas con las propiedades del desencadenador de eventos](./media/ethereum-logic-app/event-properties.png)

    | Propiedad | Descripción |
    |----------|-------------|
    | **Contract ABI** (ABI del contrato) | La interfaz binaria de aplicaciones (ABI) del contrato define las interfaces del contrato inteligente. Para más información, consulte [Obtención de ABI del contrato](#get-the-contract-abi). |
    | **Smart contract address** (Dirección del contrato inteligente) | La dirección del contrato es la dirección de destino del contrato inteligente en Ethereum Blockchain. Para más información, consulte [Obtención de la dirección del contrato](#get-the-contract-address). |
    | **Event Name** (Nombre del evento) | Seleccione un evento de contrato inteligente para buscarlo. El evento desencadena la aplicación lógica. |
    | **Interval** (Intervalo) y **Frequency** (Frecuencia) | Seleccione la frecuencia con la que desea buscar el evento. |

1. Seleccione **Guardar**.

Para completar la aplicación lógica, puede agregar un nuevo paso que realice una acción basada en el desencadenador de eventos de Ethereum Blockchain. Por ejemplo, enviar un correo electrónico.

## <a name="use-actions"></a>Uso de acciones

Use las acciones de Ethereum Blockchain cuando quiera que una aplicación lógica realice una acción en el libro de contabilidad de cadena de bloques. Por ejemplo, puede que desee crear un microservicio basado en REST que llame a una función del contrato inteligente cuando se realice una solicitud HTTP a una aplicación lógica.

Las acciones del conector requieren un desencadenador. Puede usar una acción del conector Ethereum Blockchain como paso siguiente después de un desencadenador, como un desencadenador de solicitud HTTP para un microservicio.

1. En el Diseñador de aplicaciones lógicas, seleccione **New Step** (Nuevo paso) después de un desencadenador.
1. Seleccione el conector Ethereum Blockchain.
1. En la pestaña **Actions** (Acciones), seleccione una de las acciones disponibles.

    ![Diseñador de aplicaciones lógicas con las propiedades de las acciones](./media/ethereum-logic-app/action-properties.png)

1. Cambie o [cree una conexión de API](#create-an-api-connection) a Azure Blockchain Service.
1. En función de la acción que elija, proporcione los detalles siguientes sobre la función del contrato inteligente.

    | Propiedad | Descripción |
    |----------|-------------|
    | **Contract ABI** (ABI del contrato) | La ABI del contrato define las interfaces del contrato inteligente. Para más información, consulte [Obtención de ABI del contrato](#get-the-contract-abi). |
    | **Contract Bytecode** (Código de bytes del contrato) | Código de bytes del contrato inteligente compilado. Para más información, consulte [Obtención del código de bytes del contrato](#get-the-contract-bytecode). |
    | **Smart contract address** (Dirección del contrato inteligente) | La dirección del contrato es la dirección de destino del contrato inteligente en Ethereum Blockchain. Para más información, consulte [Obtención de la dirección del contrato](#get-the-contract-address). |
    | **Smart contract function name** (Nombre de la función del contrato inteligente) | Seleccione el nombre de la función del contrato inteligente para la acción. La lista se rellena a partir de los detalles de la ABI del contrato. |

    Después de seleccionar un nombre de función del contrato inteligente, puede ver los campos obligatorios para los parámetros de la función. Escriba los valores o el contenido dinámico necesario para su escenario.

Ahora puede usar la aplicación lógica. Cuando se desencadena el evento de la aplicación lógica, se ejecuta la acción de Ethereum Blockchain. Por ejemplo, un desencadenador de solicitud HTTP ejecuta una acción de Ethereum Blockchain para consultar un valor de estado del contrato inteligente. Esta consulta da como resultado una respuesta HTTP que devuelve el valor.

## <a name="generate-a-workflow"></a>Generación de un flujo de trabajo

La extensión Azure Blockchain Development Kit for Ethereum de Visual Studio Code puede generar flujos de trabajo de aplicación lógica para escenarios comunes. Hay cuatro escenarios disponibles:

* Publicación de datos en una instancia de Azure SQL Database
* Publicación de eventos en una instancia de Azure Event Grid o de Azure Service Bus
* Publicación de informes
* Microservicio basado en REST

 Azure Blockchain Development Kit usa Truffle para simplificar el desarrollo de la cadena de bloques. Para generar una aplicación lógica basada en un contrato inteligente, necesita una solución de Truffle para el contrato inteligente. También necesita una conexión a la red del consorcio de Azure Blockchain Service. Para más información, consulte [Inicio rápido: Uso de Visual Studio Code para conectarse a una red del consorcio de Azure Blockchain Service](connect-vscode.md).

Por ejemplo, los pasos siguientes generan una aplicación lógica de un microservicio basado en REST que se basa en el contrato inteligente **HelloBlockchain** del inicio rápido:

1. En la barra lateral del explorador de Visual Studio Code, expanda la carpeta **contracts** de la solución.
1. Haga clic con el botón derecho en **HelloBlockchain.sol** y seleccione **Generate microservices for smart contracts** (Generar microservicios para contratos inteligentes) en el menú.

    ![Panel de Visual Studio Code con la selección Generate Microservices for Smart Contracts (Generar microservicios para contratos inteligentes)](./media/ethereum-logic-app/generate-logic-app.png)

1. En la paleta de comandos, seleccione **Aplicación lógica**.
1. Escriba la **dirección del contrato**. Para más información, consulte [Obtención de la dirección del contrato](#get-the-contract-address).
1. Seleccione la suscripción de Azure y el grupo de recursos para la aplicación lógica.

    Los archivos de configuración y código de la aplicación lógica se generan en el directorio **generatedLogicApp**.

1. Consulte el directorio **generatedLogicApp/HelloBlockchain**. Hay un archivo JSON de aplicación lógica para cada función, evento y propiedad del contrato inteligente.
1. Abra el archivo **generatedLogicApp/HelloBlockchain/Service/property.RequestMessage.logicapp.json** y copie el contenido.

    ![Archivo JSON con código para copiar](./media/ethereum-logic-app/requestmessage.png)

1. En la aplicación lógica, seleccione **Vista de código de la aplicación lógica**. Reemplace el archivo JSON existente por el archivo JSON de la aplicación lógica generado.

    ![Vista del código de la aplicación lógica con nuevo código de aplicación reemplazado](./media/ethereum-logic-app/code-view.png)

1. Seleccione **Diseñador** para cambiar a la vista de diseñador.
1. La aplicación lógica incluye los pasos básicos para el escenario. Sin embargo, debe actualizar los detalles de configuración para el conector Ethereum Blockchain.
1. Seleccione el paso **Conexiones** y cambie o [cree una conexión de API](#create-an-api-connection) a Azure Blockchain Service.

    ![Vista del diseñador con la selección Conexiones](./media/ethereum-logic-app/microservice-logic-app.png)

1. Ahora puede usar la aplicación lógica. Para probar el microservicio basado en REST, emita una solicitud HTTP POST a la dirección URL de solicitud de la aplicación lógica. Copie el contenido de **Dirección URL de HTTP POST** del paso **Cuando se recibe una solicitud HTTP**.

    ![Panel del Diseñador de aplicaciones lógicas con la dirección URL de HTTP POST](./media/ethereum-logic-app/post-url.png)

1. Use cURL para crear una solicitud HTTP POST. Reemplace el texto del marcador de posición *\<HTTP POST URL\>* por la dirección URL del paso anterior.

    ``` bash
    curl -d "{}" -H "Content-Type: application/json" -X POST "<HTTP POST URL>"
    ```

    El comando cURL devuelve una respuesta de la aplicación lógica. En este caso, la respuesta es la salida de la función **RequestMessage** del contrato inteligente.

    ![Salida de código de la función RequestMessage del contrato inteligente](./media/ethereum-logic-app/curl.png)

Para más información sobre el uso del kit de desarrollo, consulte la [página de la wiki sobre Azure Blockchain Development Kit para Ethereum](https://github.com/Microsoft/vscode-azure-blockchain-ethereum/wiki).

## <a name="create-an-api-connection"></a>Creación de una conexión de API

El conector Ethereum Blockchain requiere una conexión de API a una cadena de bloques. Puede usar el conector de API para varias aplicaciones lógicas. Algunas propiedades son necesarias y otras dependen del escenario.

> [!IMPORTANT]
> Para crear transacciones en una cadena de bloques, se necesita una clave privada o la dirección y la contraseña de una cuenta. Solo se precisa una forma de autenticación. No es necesario proporcionar a la vez la clave privada y los detalles de la cuenta. La consulta de contratos no requiere una transacción. Si está usando acciones que consultan el estado del contrato, no es necesario usar la clave privada ni la dirección y la contraseña de la cuenta.

Para ayudarle a configurar una conexión a un miembro de Azure Blockchain Service, en la siguiente lista se muestran las propiedades que puede necesitar en función de su escenario.

| Propiedad | Descripción |
|----------|-------------|
|**Nombre de la conexión** | Nombre de la conexión de API. Necesario. |
|**Ethereum RPC endpoint** (Punto de conexión RPC de Ethereum) | Dirección HTTP del nodo de transacción de Azure Blockchain Service. Necesario. Para más información, consulte [Obtención del punto de conexión de RPC](#get-the-rpc-endpoint). |
|**Clave privada** | Clave privada de la cuenta de Ethereum. Se requiere la clave privada o la dirección y contraseña de la cuenta para las transacciones. Para más información, consulte [Obtención de la clave privada](#get-the-private-key). |
|**Dirección de la cuenta** | Dirección de la cuenta de miembro de Azure Blockchain Service. Se requiere la clave privada o la dirección y contraseña de la cuenta para las transacciones. Para más información, consulte [Obtención de la dirección de la cuenta](#get-the-account-address). |
|**Contraseña de la cuenta** | La contraseña de la cuenta se establece al crear el miembro. Para información sobre el restablecimiento de la contraseña, consulte [Cuenta de Ethereum](consortium.md#ethereum-account).|

## <a name="get-the-rpc-endpoint"></a>Obtención del punto de conexión de RPC

Para conectarse a una red de cadena de bloques se necesita la dirección del punto de conexión RPC de Azure Blockchain Service. Puede obtener la dirección del punto de conexión mediante Azure Blockchain Development Kit para Ethereum o mediante Azure Portal.

**Para usar el kit de desarrollo**:

1. En Visual Studio Code, en **Azure Blockchain Service**, haga clic con el botón derecho en el consorcio.
1. Seleccione **Copy RPC Endpoint Address** (Copiar dirección del punto de conexión RPC).

    ![Panel de Visual Studio Code que muestra el consorcio con la opción Copy RPC Endpoint Address (Copiar dirección del punto de conexión RPC) seleccionada](./media/ethereum-logic-app/devkit-rpc.png)

    El punto de conexión RPC se copia en el portapapeles.

**Para utilizar Azure Portal**:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya al miembro de Azure Blockchain Service. Seleccione **Nodos de transacción** y el vínculo al nodo de transacción predeterminado.

    ![Página Transaction nodes (Nodos de transacción) con la selección (nodo predeterminado)](./media/ethereum-logic-app/transaction-nodes.png)

1. Seleccione **Cadenas de conexión** > **Claves de acceso**.
1. Copie la dirección del punto de conexión de **HTTPS (Access key 1)** (HTTPS [clave de acceso 1]) o **HTTPS (Access key 2)** (HTTPS [clave de acceso 2]).

    ![Azure Portal con las claves de acceso de la cadena de conexión](./media/ethereum-logic-app/connection-string.png)

    El punto de conexión RPC es la dirección URL HTTPS, que incluye la dirección y la clave de acceso del nodo de transacción del miembro de Azure Blockchain Service.

## <a name="get-the-private-key"></a>Obtención de la clave privada

Puede usar la clave privada de la cuenta de Ethereum para autenticarse al enviar una transacción a la cadena de bloques. Las claves pública y privada de la cuenta de Ethereum se generan a partir de un mnemotécnico de 12 palabras. Azure Blockchain Development Kit para Ethereum genera un mnemotécnico al conectarse a un miembro del consorcio de Azure Blockchain Service. Puede obtener la dirección del punto de conexión mediante la extensión del kit de desarrollo.

1. En Visual Studio Code, abra la paleta de comandos (F1).
1. Seleccione **Azure Blockchain: Retrieve private key** (Azure Blockchain: Recuperar la clave privada).
1. Seleccione el mnemotécnico que guardó al conectarse al miembro del consorcio.

    ![Paleta de comandos con una opción para seleccionar el mnemotécnico](./media/ethereum-logic-app/private-key.png)

    La clave privada se copia en el portapapeles.

## <a name="get-the-account-address"></a>Obtención de la dirección de la cuenta

Puede usar la cuenta y la contraseña del miembro para autenticarse al enviar una transacción a la cadena de bloques. La contraseña se establece al crear el miembro.

1. En Azure Portal, vaya a la página de información general de Azure Blockchain Service.
1. Copie la dirección de **Member account** (Cuenta del miembro).

    ![Página de información general con la dirección de la cuenta de miembro](./media/ethereum-logic-app/member-account.png)

Para más información sobre la dirección y la contraseña de la cuenta, consulte [Cuenta de Ethereum](consortium.md#ethereum-account).

## <a name="get-the-contract-abi"></a>Obtención de ABI del contrato

La ABI del contrato define las interfaces del contrato inteligente. Describe cómo interactuar con el contrato inteligente. Puede obtener la ABI del contrato mediante Azure Blockchain Development Kit para Ethereum. También puede obtenerla del archivo de metadatos del contrato creado por el compilador de Solidity.

**Para usar el kit de desarrollo**:

Si ha usado el kit de desarrollo o Truffle para compilar el contrato inteligente, puede usar la extensión para copiar la ABI del contrato en el portapapeles.

1. En el panel del explorador de Visual Studio Code, expanda la carpeta **build/contracts** del proyecto de Solidity.
1. Haga clic con el botón derecho en el archivo JSON de metadatos del contrato. El nombre de archivo es el nombre del contrato inteligente seguido de la extensión **.json**.
1. Seleccione **Copy contract ABI** (Copiar ABI del contrato).

    ![Panel de Visual Studio Code con la selección Copy contract ABI (Copiar ABI del contrato)](./media/ethereum-logic-app/abi-devkit.png)

    La ABI del contrato se copia en el portapapeles.

**Para usar el archivo de metadatos del contrato**:

1. Abra el archivo de metadatos del contrato incluido en la carpeta **build/contracts** del proyecto de Solidity. El nombre de archivo es el nombre del contrato inteligente seguido de la extensión **.json**.
1. Busque la sección **abi** en el archivo JSON.
1. Copie la matriz JSON de **abi**.

    ![Código de ABI en el archivo de metadatos del contrato](./media/ethereum-logic-app/abi-metadata.png)

## <a name="get-the-contract-bytecode"></a>Obtención del código de bytes del contrato

El código de bytes del contrato es el contrato inteligente compilado ejecutado por la máquina virtual de Ethereum. Puede obtener el código de bytes del contrato mediante Azure Blockchain Development Kit para Ethereum. También puede obtenerlo desde el compilador de Solidity.

**Para usar el kit de desarrollo**:

Si ha usado el kit de desarrollo o Truffle para compilar el contrato inteligente, puede usar la extensión para copiar el código de bytes del contrato en el portapapeles.

1. En el panel del explorador de Visual Studio Code, expanda la carpeta **build/contracts** del proyecto de Solidity.
1. Haga clic con el botón derecho en el archivo JSON de metadatos del contrato. El nombre de archivo es el nombre del contrato inteligente seguido de la extensión **.json**.
1. Seleccione **Copy Contract Bytecode** (Copiar código de bytes del contrato).

    ![Panel de Visual Studio Code con la selección Copy Contract Bytecode (Copiar código de bytes del contrato)](./media/ethereum-logic-app/bytecode-devkit.png)

    El código de bytes del contrato se copia en el portapapeles.

**Para usar el archivo de metadatos del contrato**:

1. Abra el archivo de metadatos del contrato incluido en la carpeta **build/contracts** del proyecto de Solidity. El nombre de archivo es el nombre del contrato inteligente seguido de la extensión **.json**.
1. Busque el elemento **bytecode** en el archivo JSON.
1. Copie el valor de **bytecode**.

    ![Panel de Visual Studio Code con código de bytes en los metadatos](./media/ethereum-logic-app/bytecode-metadata.png)

**Para usar el compilador de Solidity**:

Use el comando `solc --bin <smart contract>.sol` para generar el código de bytes del contrato.

## <a name="get-the-contract-address"></a>Obtención de la dirección del contrato

La dirección del contrato es la dirección de destino del contrato inteligente en Ethereum Blockchain. Esta dirección se usa para enviar una transacción o para consultar el estado de un contrato inteligente. Puede obtener la dirección del contrato a partir de la salida de la migración de Truffle o mediante el archivo de metadatos del contrato.

**Para usar la salida de la migración de Truffle**:

Truffle muestra la dirección del contrato después de la implementación del contrato inteligente. Copie la **dirección del contrato** de la salida.

![Salida de la migración de Truffle con la dirección del contrato en Visual Studio Code](./media/ethereum-logic-app/contract-address-truffle.png)

**Para usar el archivo de metadatos del contrato**:

1. Abra el archivo de metadatos del contrato incluido en la carpeta **build/contracts** del proyecto de Solidity. El nombre de archivo es el nombre del contrato inteligente seguido de la extensión **.json**.
1. Busque la sección **networks** en el archivo JSON.
1. Las redes privadas se identifican mediante un identificador de red de entero. Busque el valor address en la sección de red.
1. Copie el valor de **address**.

![Metadatos con el valor de la dirección en Visual Studio Code](./media/ethereum-logic-app/contract-address-metadata.png)

## <a name="next-steps"></a>Pasos siguientes

Vea escenarios comunes en el vídeo [Doing more with Logic Apps](https://channel9.msdn.com/Shows/Blocktalk/Doing-more-with-Logic-Apps?term=logic%20apps%20blockchain&lang-en=true) (Hacer más con las aplicaciones lógicas).
