---
title: Configuración de un laboratorio para usar Puerta de enlace de Escritorio remoto en Azure DevTest Labs
description: Obtenga información sobre cómo configurar un laboratorio en Azure DevTest Labs con una puerta de enlace de Escritorio remoto para garantizar el acceso seguro a las máquinas virtuales de laboratorio sin tener que exponer el puerto RDP.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: bc45a0c2953f8f84289fa01d4af72bf98544bd7f
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288085"
---
# <a name="configure-your-lab-in-azure-devtest-labs-to-use-a-remote-desktop-gateway"></a>Configuración del laboratorio de Azure DevTest Labs para usar una puerta de enlace de Escritorio remoto
En Azure DevTest Labs, puede configurar una puerta de enlace de Escritorio remoto para el laboratorio con el fin de garantizar el acceso seguro a las máquinas virtuales (VM) del laboratorio sin tener que exponer el puerto RDP. El laboratorio proporciona un lugar central para que los usuarios del laboratorio vean todas las máquinas virtuales a las que tienen acceso y se conecten a ellas. El botón **Connect** (Conectar) de la página **Virtual Machine** (Máquina Virtual) crea un archivo RDP específico de la máquina que se puede abrir para conectarse a ella. Puede personalizar y proteger aún más la conexión RDP si conecta el laboratorio a una puerta de enlace de Escritorio remoto. 

Este enfoque es más seguro porque el usuario del laboratorio se autentica directamente en la máquina de puerta de enlace, o bien puede usar las credenciales de la empresa en un equipo de puerta de enlace unido al dominio para conectarse a sus máquinas. El laboratorio también admite la autenticación de token en el equipo de puerta de enlace que permite a los usuarios conectarse a sus máquinas virtuales de laboratorio sin exponer el puerto RDP a Internet. Este artículo le guía a través de un ejemplo sobre cómo configurar un laboratorio en el que se usa la autenticación de token para conectarse a las máquinas del laboratorio.

## <a name="architecture-of-the-solution"></a>Arquitectura de la solución

![Arquitectura de la solución](./media/configure-lab-remote-desktop-gateway/architecture.png)

1. La acción [Obtener contenido del archivo RDP](/rest/api/dtl/virtualmachines/getrdpfilecontents) se llama al seleccionar el botón **Connect** (Conectar).1. 
1. La acción Obtener contenido del archivo RDP invoca `https://{gateway-hostname}/api/host/{lab-machine-name}/port/{port-number}` para solicitar un token de autenticación.
    1. `{gateway-hostname}` es el nombre de host de puerta de enlace especificado en la página **Configuración del laboratorio** del laboratorio en Azure Portal. 
    1. `{lab-machine-name}` es el nombre de la máquina a la que se intenta conectar.
    1. `{port-number}` es el puerto en el que se debe realizar la conexión. Normalmente este puerto es el 3389. Si en la máquina virtual del laboratorio se usa la característica [IP compartida](devtest-lab-shared-ip.md) de DevTest Labs, el puerto será otro.
1. La puerta de enlace de Escritorio remoto traslada la llamada desde `https://{gateway-hostname}/api/host/{lab-machine-name}/port/{port-number}` a una función de Azure para generar el token de autenticación. El servicio DevTest Labs incluye de forma automática la clave de función en el encabezado de solicitud. La clave de función se guardará en el almacén de claves del laboratorio. El nombre de ese secreto que se va a mostrar como **secreto del token de puerta de enlace** en la página **Configuración del laboratorio** del laboratorio.
1. Se espera que la función de Azure devuelva un token para la autenticación de token basada en certificados en la máquina de puerta de enlace.  
1. Después, la acción Obtener contenido del archivo RDP devuelve el archivo RDP completo, incluida la información de autenticación.
1. Puede abrir el archivo RDP con el programa de conexión de RDP que prefiera. Recuerde que no todos los programas de conexión de RDP admiten la autenticación de token. El token de autenticación tiene una fecha de expiración, establecida por la aplicación de función. Establezca la conexión con la máquina virtual del laboratorio antes de que expire el token.
1. Una vez que la máquina de puerta de enlace de Escritorio remoto autentique el token en el archivo RDP, la conexión se reenvía a la máquina del laboratorio.

### <a name="solution-requirements"></a>Requisitos de la solución
Para trabajar con la característica de autenticación de token de DevTest Labs, hay varios requisitos de configuración para las máquinas de puerta de enlace, los servicios de nombres de dominio (DNS) y las funciones.

### <a name="requirements-for-remote-desktop-gateway-machines"></a>Requisitos para máquinas de puerta de enlace de Escritorio remoto
- En la máquina de puerta de enlace se debe instalar un certificado TLS/SSL para controlar el tráfico HTTPS. El certificado debe coincidir con el nombre de dominio completo (FQDN) del equilibrador de carga para la granja de servidores de puerta de enlace, o bien el FQDN de la propia máquina, si solo hay una. Los certificados TLS/SSL de comodín no funcionan.  
- Un certificado de firma instalado en las máquinas de puerta de enlace. Para crear un certificado de firma, use el script [Create-SigningCertificate.ps1](https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/tools/Create-SigningCertificate.ps1).
- Instale el módulo [Autenticación conectable](https://code.msdn.microsoft.com/windowsdesktop/Remote-Desktop-Gateway-517d6273) que admite la autenticación de token para la puerta de enlace de Escritorio remoto. Un ejemplo de este tipo de módulo es `RDGatewayFedAuth.msi`, que se incluye con las [imágenes de System Center Virtual Machine Manager (VMM)](/system-center/vmm/install-console?view=sc-vmm-1807). Para más información sobre System Center, vea la [documentación de System Center](/system-center/) y la [información sobre precios](https://www.microsoft.com/cloud-platform/system-center-pricing).  
- El servidor de puerta de enlace puede controlar las solicitudes realizadas a `https://{gateway-hostname}/api/host/{lab-machine-name}/port/{port-number}`.

    El nombre de host de puerta de enlace es el FQDN del equilibrador de carga de la granja de servidores de puerta de enlace, o bien el FQDN de la propia máquina si solo hay una. `{lab-machine-name}` es el nombre de la máquina del laboratorio a la que se intenta conectar, y `{port-number}` es el puerto en el que se va a realizar la conexión.  De forma predeterminada, este puerto es 3389.  Pero si en la máquina virtual se usa la característica [IP compartida](devtest-lab-shared-ip.md) de DevTest Labs, el puerto será otro.
- Se puede usar el módulo de [solicitud de enrutamiento de la aplicación](/iis/extensions/planning-for-arr/using-the-application-request-routing-module) para Internet Information Server (IIS) para redirigir las solicitudes a `https://{gateway-hostname}/api/host/{lab-machine-name}/port/{port-number}` a la función de Azure, que controla la solicitud para obtener un token para la autenticación.


## <a name="requirements-for-azure-function"></a>Requisitos de la función de Azure
La función de Azure controla la solicitud con el formato de `https://{function-app-uri}/app/host/{lab-machine-name}/port/{port-number}` y devuelve el token de autenticación en función del mismo certificado de firma instalado en las máquinas de puerta de enlace. `{function-app-uri}` es el URI que se usa para acceder a la función. La clave de función se pasa de forma automática en el encabezado de la solicitud. Para obtener una función de ejemplo, vea [https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/src/RDGatewayAPI/Functions/CreateToken.cs](https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/src/RDGatewayAPI/Functions/CreateToken.cs). 


## <a name="requirements-for-network"></a>Requisitos de red

- El DNS para el FQDN asociado con el certificado TLS/SSL instalado en las máquinas de puerta de enlace debe dirigir el tráfico a la máquina de puerta de enlace o al equilibrador de carga de la granja de servidores de máquina de puerta de enlace.
- Si en la máquina del laboratorio se usan direcciones IP privadas, debe haber una ruta de acceso de red desde la máquina de puerta de enlace a la máquina del laboratorio, ya sea mediante el uso compartido de la misma red virtual o mediante redes virtuales emparejadas.

## <a name="configure-the-lab-to-use-token-authentication"></a>Configuración del laboratorio para usar la autenticación de token 
En esta sección se muestra cómo configurar un laboratorio para usar una máquina de puerta de enlace de Escritorio remoto que admite la autenticación de token. En esta sección no se describe cómo configurar una granja de servidores de puerta de enlace de Escritorio remoto. Para obtener esa información, vea la sección [Ejemplo para crear una puerta de enlace de Escritorio remoto](#sample-to-create-a-remote-desktop-gateway) al final de este artículo. 

Antes de actualizar la configuración del laboratorio, almacene la clave necesaria para ejecutar de forma correcta la función para que devuelva un token de autenticación en el almacén de claves del laboratorio. Puede obtener el valor de clave de función en la página **Administrar** de la función en Azure Portal. Para más información sobre cómo guardar un secreto en un almacén de claves, vea [Incorporación de un secreto a Key Vault](../key-vault/secrets/quick-create-portal.md#add-a-secret-to-key-vault). Guarde el nombre del secreto para usarlo más tarde.

Para buscar el identificador del almacén de claves del laboratorio, ejecute el siguiente comando de la CLI de Azure: 

```azurecli
az resource show --name {lab-name} --resource-type 'Microsoft.DevTestLab/labs' --resource-group {lab-resource-group-name} --query properties.vaultName
```

Siga estos pasos para configurar el laboratorio para usar la autenticación de token:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.
1. En la lista de laboratorios, seleccione el **suyo**.
1. En la página del laboratorio, seleccione **Configuración y directivas**.
1. En el menú de la izquierda, en la sección **Configuración**, seleccione **Configuración del laboratorio**.
1. En la sección **Escritorio remoto**, escriba el nombre de dominio completo (FQDN) o la dirección IP de la máquina de puerta de enlace de servicios de Escritorio remoto o de la granja de servidores en el campo **Nombre de host de puerta de enlace**. Este valor debe coincidir con el FQDN del certificado TLS/SSL que se usa en las máquinas de puerta de enlace.

    ![Opciones de Escritorio remoto en la configuración del laboratorio](./media/configure-lab-remote-desktop-gateway/remote-desktop-options-in-lab-settings.png)
1. En la sección **Escritorio remoto**, para el secreto de **token de puerta de enlace**, escriba el nombre del secreto que ha creado antes. Este valor no es la propia clave de función, sino el nombre del secreto en el almacén de claves del laboratorio que contiene la clave de función.

    ![Secreto del token de puerta de enlace en la configuración del laboratorio](./media/configure-lab-remote-desktop-gateway/gateway-token-secret.png)
1. **Guarde** los cambios.

    > [!NOTE] 
    > Al hacer clic en **Guardar**, acepta los [términos de licencia de la puerta de enlace de Escritorio remoto](https://www.microsoft.com/licensing/product-licensing/products). Para más información sobre la puerta de enlace remota, vea [Bienvenido a Servicios de Escritorio remoto](https://aka.ms/rds) e [Implementación del entorno de Escritorio remoto](/windows-server/remote/remote-desktop-services/rds-deploy-infrastructure).


Si prefiere configurar el laboratorio mediante la automatización, vea [Set-DevTestLabGateway.ps1](https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/tools/Set-DevTestLabGateway.ps1) para obtener un script de PowerShell de ejemplo para establecer las opciones **nombre de host de puerta de enlace** y **secreto del token de puerta de enlace**. En el [repositorio de GitHub de Azure DevTest Labs](https://github.com/Azure/azure-devtestlab) también se proporciona una plantilla de Azure Resource Manager que permite crear o actualizar un laboratorio con las opciones de **nombre de host de puerta de enlace** y **secreto del token de puerta de enlace**.

## <a name="configure-network-security-group"></a>Configuración del grupo de seguridad de red
Para proteger aún más el laboratorio, se puede agregar un grupo de seguridad de red (NSG) a la red virtual que usan las máquinas virtuales del laboratorio. Para obtener instrucciones sobre cómo configurar un NSG, vea [Creación, modificación o eliminación de un grupo de seguridad de red](../virtual-network/manage-network-security-group.md).

Este es un NSG de ejemplo que solo permite que el tráfico que primero atraviesa la puerta de enlace llegue a las máquinas del laboratorio. El origen de esta regla es la dirección IP de la máquina de puerta de enlace única, o bien la dirección IP del equilibrador de carga situado delante de las máquinas de puerta de enlace.

![Grupo de seguridad de red: reglas](./media/configure-lab-remote-desktop-gateway/network-security-group-rules.png)

## <a name="sample-to-create-a-remote-desktop-gateway"></a>Ejemplo para crear una puerta de enlace de Escritorio remoto

> [!NOTE] 
> Al usar las plantillas de ejemplo, acepta los [términos de licencia de la puerta de enlace de Escritorio remoto](https://www.microsoft.com/licensing/product-licensing/products). Para más información sobre la puerta de enlace remota, vea [Bienvenido a Servicios de Escritorio remoto](https://aka.ms/rds) e [Implementación del entorno de Escritorio remoto](/windows-server/remote/remote-desktop-services/rds-deploy-infrastructure).

En el [repositorio de GitHub de Azure DevTest Labs](https://github.com/Azure/azure-devtestlab) se proporcionan algunos ejemplos para facilitar la configuración de los recursos necesarios para usar la autenticación de token y la puerta de enlace de Escritorio remoto con DevTest Labs. En estos ejemplos se incluyen plantillas de Azure Resource Manager para máquinas de puerta de enlace, la configuración del laboratorio y la aplicación de función.

Siga estos pasos para configurar una solución de ejemplo para la granja de servidores de puerta de enlace de Escritorio remoto.

1. Cree un certificado de firma.  Ejecute [Create-SigningCertificate.ps1](https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/tools/Create-SigningCertificate.ps1). Guarde la huella digital, la contraseña y la codificación Base64 del certificado que se ha creado.
2. Obtenga un certificado TLS/SSL. El nombre de dominio completo asociado con el certificado TLS/SSL debe ser para el dominio que controla. Guarde la huella digital, la contraseña y la codificación Base64 de este certificado. Para obtener la huella digital con PowerShell, use los comandos siguientes.

    ```powershell
    $cer = New-Object System.Security.Cryptography.X509Certificates.X509Certificate;
    $cer.Import(‘path-to-certificate’);
    $hash = $cer.GetCertHashString()
    ```

    Para obtener la codificación Base64 con PowerShell, use el comando siguiente.

    ```powershell
    [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes(‘path-to-certificate’))
    ```
3. Descargue los archivos de [https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/GatewaySample/arm/gateway](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/GatewaySample/arm/gateway).

    La plantilla requiere acceso a otras plantillas de Resource Manager y recursos relacionados en el mismo URI base. Copie todos los archivos de [https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/arm/gateway](https://github.com/Azure/azure-devtestlab/blob/master/samples/DevTestLabs/GatewaySample/arm/gateway) y RDGatewayFedAuth.msi en un contenedor de blobs en una cuenta de almacenamiento.  
4. Implemente **azuredeploy.json** desde [https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/GatewaySample/arm/gateway](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/GatewaySample/arm/gateway). La plantilla toma los parámetros siguientes:
    - adminUsername: requerido.  Nombre de usuario administrador para las máquinas de puerta de enlace.
    - adminPassword: requerido. Contraseña de la cuenta de administrador para las máquinas de puerta de enlace.
    - instanceCount: número de máquinas de puerta de enlace que se van a crear.  
    - alwaysOn: indica si la aplicación de Azure Functions que se ha creado se debe mantener en un estado activo o no. Mantener la aplicación de Azure Functions evitará retrasos cuando los usuarios intentan conectarse por primera vez a su máquina virtual de laboratorio, pero tiene implicaciones de costo.  
    - tokenLifetime: el período de tiempo de validez del token creado. El formato es HH:MM:SS.
    - sslCertificate: la codificación Base64 del certificado TLS/SSL para la máquina de puerta de enlace.
    - sslCertificatePassword: la contraseña del certificado TLS/SSL para la máquina de puerta de enlace.
    - sslCertificateThumbprint: la huella digital del certificado para la identificación en el almacén de certificados local del certificado TLS/SSL.
    - signCertificate: la codificación Base64 del certificado de firma para la máquina de puerta de enlace.
    - signCertificatePassword: la contraseña del certificado de firma para la máquina de puerta de enlace.
    - signCertificateThumbprint: la huella digital del certificado para la identificación en el almacén de certificados local del certificado de firma.
    - _artifactsLocation: ubicación del URI donde se pueden encontrar todos los recursos complementarios. Este valor debe ser un URI completo, no una ruta de acceso relativa.
    - _artifactsLocationSasToken: el token de firma de acceso compartido (SAS) que se usa para acceder a los recursos complementarios, si la ubicación es una cuenta de almacenamiento de Azure.

    La plantilla se puede implementar mediante la CLI de Azure con el comando siguiente:

    ```azurecli
    az group deployment create --resource-group {resource-group} --template-file azuredeploy.json --parameters @azuredeploy.parameters.json -–parameters _artifactsLocation="{storage-account-endpoint}/{container-name}" -–parameters _artifactsLocationSasToken = "?{sas-token}"
    ```

    Estas son las descripciones de los parámetros:

    - {storage-account-endpoint} se puede obtener mediante la ejecución de `az storage account show --name {storage-acct-name} --query primaryEndpoints.blob`.  {storage-acct-name} es el nombre de la cuenta de almacenamiento que contiene los archivos que se han cargado.  
    - {container-name} es el nombre del contenedor en {storage-acct-name} que contiene los archivos que se han cargado.  
    - {sas-token} se puede obtener mediante la ejecución de `az storage container generate-sas --name {container-name} --account-name {storage-acct-name} --https-only –permissions drlw –expiry {utc-expiration-date}`. 
        - {storage-acct-name} es el nombre de la cuenta de almacenamiento que contiene los archivos que se han cargado.  
        - {container-name} es el nombre del contenedor en {storage-acct-name} que contiene los archivos que se han cargado.  
        - {utc-expiration-date} es la fecha (en formato UTC) a la que caducará el token de SAS y ya no se podrá usar para acceder a la cuenta de almacenamiento.

    Registre los valores de gatewayFQDN y gatewayIP desde la salida de la implementación de la plantilla. También tendrá que guardar el valor de la clave de la función recién creada, que se puede encontrar en la pestaña [Configuración de Function App](../azure-functions/functions-how-to-use-azure-function-app-settings.md).
5. Configure DNS para dirigir el FQDN de ese certificado TLS/SSL a la dirección IP de gatewayIP del paso anterior.

    Después de crear la granja de servidores de puerta de enlace de Escritorio remoto y de realizar las actualizaciones de DNS correspondientes, está listo para usarse en un laboratorio de DevTest Labs. Las opciones de **nombre de host de puerta de enlace** y **secreto del token de puerta de enlace** se deben configurar para usar las máquinas de puerta de enlace que se hayan implementado. 

    > [!NOTE]
    > Si en la máquina del laboratorio se usan direcciones IP privadas, debe haber una ruta de acceso de red desde la máquina de puerta de enlace a la máquina del laboratorio, ya sea mediante el uso compartido de la misma red virtual o mediante una red virtual emparejada.

    Una vez que se hayan configurado la puerta de enlace y el laboratorio, el archivo de conexión que se crea cuando el usuario del laboratorio hace clic en **Conectar** incluirá automáticamente la información necesaria para conectarse mediante la autenticación de token.     

## <a name="next-steps"></a>Pasos siguientes
Vea el artículo siguiente para más información sobre Servicios de Escritorio remoto: [Documentación de Servicios de Escritorio remoto](/windows-server/remote/remote-desktop-services/Welcome-to-rds)
