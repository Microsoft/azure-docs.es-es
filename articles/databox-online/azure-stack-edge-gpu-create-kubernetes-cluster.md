---
title: Creación y administración de un clúster de Kubernetes en un dispositivo de Azure Stack Edge Pro con GPU | Microsoft Docs
description: Se describe cómo crear y administrar un clúster de Kubernetes en un dispositivo de Azure Stack Edge Pro con GPU mediante la interfaz de Windows PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: cb783e5da7364f38944ce31ce49a6a6529658fe3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903217"
---
# <a name="connect-to-and-manage-a-kubernetes-cluster-via-kubectl-on-your-azure-stack-edge-pro-gpu-device"></a>Conexión y administración de un clúster de Kubernetes mediante kubectl en un dispositivo de Azure Stack Edge Pro con GPU

En un dispositivo Azure Stack Edge Pro, se crea un clúster de Kubernetes al configurar el rol de proceso. Una vez creado el clúster de Kubernetes, puede conectarse al clúster y administrarlo localmente desde una máquina cliente a través de una herramienta nativa, como *kubectl*.

En este artículo se describe cómo conectarse a un clúster de Kubernetes en un dispositivo Azure Stack Edge Pro y, después, administrarlo mediante *kubectl*. 


## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que:

1. Tiene acceso a un dispositivo Azure Stack Edge Pro.

2. Ha activado un dispositivo Azure Stack Edge Pro como se describe en [Activación de Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md).

3. Ha habilitado el rol de proceso en el dispositivo. También se ha creado un clúster de Kubernetes en el dispositivo al configurar el proceso en el dispositivo según las instrucciones de [Configuración del proceso en un dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-configure-compute.md).

4. Tiene acceso a un sistema cliente de Windows que ejecuta PowerShell 5.0 o posterior para acceder al dispositivo. También puede tener otro cliente con un [sistema operativo compatible](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device). 

5. Tiene el punto de conexión de la API de Kubernetes de la página **Dispositivo** de la interfaz de usuario web local. Para obtener más información, consulte las instrucciones en [Obtención de los puntos de conexión de Kubernetes](azure-stack-edge-gpu-deploy-configure-compute.md#get-kubernetes-endpoints).


## <a name="connect-to-powershell-interface"></a>Conexión a la interfaz de PowerShell

Una vez creado el clúster de Kubernetes, puede acceder a este clúster para crear espacios de nombres y usuarios, y asignar usuarios a los espacios de nombres. Para esto, tendrá que conectarse a la interfaz de PowerShell del dispositivo. Siga estos pasos en el cliente de Windows que ejecuta PowerShell.

[!INCLUDE [Connect to admin runspace](../../includes/azure-stack-edge-gateway-connect-minishell.md)]


## <a name="configure-cluster-access-via-rbac"></a>Configuración del acceso al clúster mediante RBAC

Una vez creado el clúster de Kubernetes, puede usar *kubectl* a través de cmdline para acceder al clúster. 

En este enfoque, creará un espacio de nombres y un usuario. Luego, asocie el usuario al espacio de nombres. También debe obtener el archivo *config* que le permita usar un cliente de Kubernetes para comunicarse directamente con el clúster de Kubernetes que creó sin tener que conectarse a la interfaz de PowerShell del dispositivo Azure Stack Edge Pro.

1. Cree un espacio de nombres. Escriba:

    `New-HcsKubernetesNamespace -Namespace <string>` 

    > [!NOTE]
    > En el caso del espacio de nombres y los nombres de usuario, se aplican las [convenciones de nomenclatura del subdominio DNS](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names).

    Este es una salida de ejemplo:

    `[10.100.10.10]: PS> New-HcsKubernetesNamespace -Namespace "myasetest1"`

2. Cree un usuario y obtenga un archivo de configuración. Escriba:

    `New-HcsKubernetesUser -UserName <string>`

    > [!NOTE]
    > No se puede usar *aseuser* como nombre de usuario, ya que está reservado para un usuario predeterminado asociado al espacio de nombres de IoT para Azure Stack Edge Pro.

    Este es un ejemplo de resultado del archivo de configuración:
   
    ```powershell
    [10.100.10.10]: PS> New-HcsKubernetesUser -UserName "aseuser1"
        apiVersion: v1
        clusters:
        - cluster:
            certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01ERXlPVEUyTlRFeE4xb1hEVE13TURFeU5qRTJOVEV4TjFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTXNpCkdZUHB0U1VWcDhKZEdVcHE1MVBURWhsZm8wWkk3YXFBdUlrOHZWdUFCZHJQK1FBSzFxcEN1di93NjIwbUtpZ0QKak1aT3Q4QkREREppWHF6UDZRZm5Oc0U2VXBHMnh0YnYrcTZHV2R5K0t6WkxMbXlwWGY3VjlzZEJnejVKVDNvYQpIdzFja2NTUklHSlV3UWxTbklNaHJUS3JUNDZFUUp3d282TmlNUzZMZDZieVk3WkUrTGg3OS9aNEhLanhTRmhMClc5ZG8veThZR3FXUDZmZTFmMmVmSkhUeGtwR05HZE1UVjNuOFlCZ0pSRzdrNjh0N2MrZ1NhbUlVWVJpTUNSNFAKYlFxcFpscWYvV2REZEJHOFh6aDJ0M1l4SkVIMm00T0Z1cSsvUitMYm95aHdKbmNMdVJ5OEpNZWlwTEQ3UlN0QwpZTDNNR0EzN2JieTRyYm4zVzg4Q0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFNbzFwWlBtQzV1cmRPZUJhSWQ4eEQzRkxCMG8KTlErbXBXMWpDd0ZtY3h6dUtlWmRsNXc2N0tuS2JTcDR0TXo1cXg3bUtSc0UxcnBoWkh2VHlKUXg1ZFk2ZE1Kdgp5d2FQZjBpT05TNlU2cC9INE12U1dJaEtJZ1FuTnE1dDh4TjJCNnZpQW41RmZoRkx6WEQrUlZGSm42cnovWkZnCmV6MHpxTkNKYmcvelFucFROcmQ2cnFFRHpoSVFZOVdYVWQycFh3ZXRqUXJpMkpZamh4NmtEcTVoRkZTM0FLUnIKOWlQTVQxaWNkR1NUMFVvM1hIZ1k2ck45WGp3MHFrY2I0Sy83UlVVWlRvS3dKamROR3lNTnpad000L2puR0p5SwpQTE9ycU5Ddlkvb0lkVEM5eVZVY3VRbXVlR0VqT20xUnN1RDFHYVE0RTZwakppVWJpMVdrajJ1bFhOWT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
            server: https://10.128.47.90:6443
            name: kubernetes
        contexts:
        - context:
            cluster: kubernetes
            user: aseuser1
            name: aseuser1@kubernetes
        current-context: aseuser1@kubernetes
        kind: Config
        preferences: {}
        users:
        - name: aseuser1
            user:
            client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMwVENDQWJtZ0F3SUJBZ0lJWlFXcjY2cGFWSm93RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBeE1qa3hOalV4TVRkYUZ3MHlNVEF4TWpneU1qVTJNVGRhTUJNeApFVEFQQmdOVkJBTVRDR0Z6WlhWelpYSXhNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDCkFRRUF4R3RDaXJ2cVhGYm5wVmtaYlVPRWQ3cXg2UzNVZ092MlhHRHNKM2VYWXN0bUxQVjMrQnlBcWwyN211L1AKaWdmaWt6MG9QSW1iYmtvcVJkamlYckxFZnk0N3dHcEhzdUhUOHNLY2tHTnJleFE2bXhaZ29xaU1nL2FuMUpMdwpiOFEvVnlQaWdVdUt6eVBseEhUZmlmSVM5MzR1VnZVZUc0dzlMRjAyZ2s2Nitpc0ZtanhsVmhseWRMNlc2UmZTCjl0OGpNMEFkdEpJL0xNbE13RHJJRVdFKzM4WDVNelJhQkJYNnlzNDFWSkZxekcwdW14dHdxN2pGOXp1UTE4ekIKalRZaDl3OWVKcDJwS2Fvak5tNE9SSDh4SzVSaUhocjJ2anFJWXkxRDd2WDh0b0U1K05HNmxHZjh5L1NvQnNRbQpmOG9vL1k3SEZmQXVGdlN6WUc1RUlQTFM4UUlEQVFBQm95Y3dKVEFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEClZSMGxCQXd3Q2dZSUt3WUJCUVVIQXdJd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFNR1BxY0YzS1BCbHZ0K24KN1NOcGE3anhWYkhZVGxyNTgwVWxzek93WEwwVnVPUUlCYmN2djIzZk9HNkhDZlQ0bWxBU0JRWVNZcmpLMjJTTwpTWld4cjNQUDlhVzNHajkxc0ttSnc1ZUF1WFhQbUJpK1RWQzBvY0ZLaEQvZ0o1aC93YnBaVndpVjVyRWE5Kzc2CnhNcFAzRld6dG5tT1hPaEl6UFNlR3B4YWpwQXd3ZXd4QU0yb0xGRFZFcy9XTFFMODJZM3NFcE93NVNaSVJJNXMKUHhMUTVnV1ZPM2x2SXcwZ3IrdkJlanZSOUZKaWVuTWFRdGdjSVgyRmpDaDBRMHVYRkdsTVNXWEljbjRLRTR0TApQSFFMalRSVUwyVnRXcW1YZ1RBM3RzN01DcGNRTFdPZFJUYkpSejZCbkc1aXVwcDdOSlFvYW9YcWpNVk5DVDZCCllYMEd0Skk9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
            client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBeEd0Q2lydnFYRmJucFZrWmJVT0VkN3F4NlMzVWdPdjJYR0RzSjNlWFlzdG1MUFYzCitCeUFxbDI3bXUvUGlnZmlrejBvUEltYmJrb3FSZGppWHJMRWZ5NDd3R3BIc3VIVDhzS2NrR05yZXhRNm14WmcKb3FpTWcvYW4xSkx3YjhRL1Z5UGlnVXVLenlQbHhIVGZpZklTOTM0dVZ2VWVHNHc5TEYwMmdrNjYraXNGbWp4bApWaGx5ZEw2VzZSZlM5dDhqTTBBZHRKSS9MTWxNd0RySUVXRSszOFg1TXpSYUJCWDZ5czQxVkpGcXpHMHVteHR3CnE3akY5enVRMTh6QmpUWWg5dzllSnAycEthb2pObTRPUkg4eEs1UmlIaHIydmpxSVl5MUQ3dlg4dG9FNStORzYKbEdmOHkvU29Cc1FtZjhvby9ZN0hGZkF1RnZTellHNUVJUExTOFFJREFRQUJBb0lCQUVVSUVXM2kxMTQycU5raQo5RjNEWWZZV1pscTJZYjRoc0FjTmhWSGxwUTN5d0dsQ3FEUktDQ3BZSVF3MkJqSFR6WnpEM0xWU0E0K0NmMUxuCkE4QVdnaHJVcStsWE1QVzhpcG9DTGJaTlNzUUord0x3bld2dFl0MHFQaGZtd0p2M1UrK1RUQkwyOHNVVUw3ZVkKLzh0aWlhbno3ZU5mNklIMENyZmgxcnQ3WWhsemtRd1hBVHNScVJja0dMaTgrdGN5WnVzdGFhbENUSzBGRTdCaQpBUGE5a2w1SG56eCs4TTcvNWladHkwTUIxYWpWMnlGblBkUmlKSFVCb1AxVVV0QUthYjVZU0RvNllkZ2pIUTRHCjNWN1l1YWZobnVFMXA0VVIvUkloVVdjRlVVaTFBOFpZMFdnd1BDTmhnMWpQZU5vb2Y1UHpRbEY1OTRBREVwUUYKOFR2bG92RUNnWUVBOWZZbUxyY0tlQ0JiMTFoQVhoTi91Z1RTbU5xNnpFL1pPSWl6M0xwckdjRDhvWDdCVW9GcgplelkxbktSS2tkczE5OERnVjlQZUhuNzllQTRoMjM5RkIwNFFhMUJBdUVMRzRsdHJ3VlNxaFBENUR6YkcrSEhSCnJtYThVMEpUSmVVS0tJVjRUUGxlTzFtK2tjbkRJVXY1ckpwZDVXU3RvcUhXdk9RZkEvRUF0VlVDZ1lFQXpHOTcKTitCZVVvbFNiREttVUNGdTdPZGhYSXJYR3RnSEorZ2JOMDlnSHRURG5PY0IxZ1NzNkpZa1FPQU9qbWFxK05lRAp5SUF1NytheWlFRmpyT2tzTGhkSTREUXNkWFZveFFGVko1V1JwWlk3UTVRaFZpYUR2enR4NDlzSDlKSkplM2U0Cnl3NWdpNGkxKy90MnY2eWRKcWdNQ0xxOHlEdFRrcE9PSitkbkp5MENnWUJwZ3lpcURaZU9KTU9CUTdpSkl2QSsKQ21lVmJ1K0hTaEd6TU9HSHBPamc2V3IybEh1Mk94S3lqblM5TjdWTmtLNDhGQitwVFpnUm1RUi9CZ0Q4T2tLUQplYXFOZnFYazViQ1AxZ3dKcVpwazRVTFdoZmNoQ1NLY0lESlZ2VFFTSTRrU0RQK29kYWs0Nkt6WnVhWGRtTXdJCmdVZ2FhZkFhdmpaeVhhSDRmT0NDNlFLQmdHVXJCaDh3dVh5KzJEc1RGWnF4OE9McjNoS2Q0clUyRXRSODJIc1cKbk1xbEgraVZxU0x3VFdFTWJBUnUzTVU3cVlCYnBxdWlRNWdVNG1UcmR4Z3FpK0tEUTEwd2RJL3IrbDBEdTlCTApCRGlkajlaeGg4M0tZWWhSTXBzLzJULys1TDVsRU4zcnozczl2RkZtcisxS3pycENqeklDdDBtZmtrd0hHV0pGCjhaWkJBb0dBVXB3aUIrcWlHbkpxU1FtZHNSZFVabGFBaTRpbGhaa01RYTRHem95ZFQ3OTVHTm44ZThBRjd3WHMKTGpyYjdEV1FwakdCMnZpUlkySUZBVmIyKzZsdDlwOVJRMTZnSmxpNU5ZRXVvQWRoWXBsVWdBZGFHWHNGNHdabwo3SHFHTHBGdmUxVU5Gb0dQdkxpWUNrUFVYdGduQ3dNb0R2SEpKNzVYMXl6ckh6cmxUS1k9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==  
        
    [10.100.10.10]: PS>
    ```
    

3. Un archivo de configuración se muestra en texto sin formato. Copie este archivo y guárdelo como archivo *config*. 

    > [!IMPORTANT]
    > No guarde el archivo de configuración como archivo *.txt*, guarde el archivo sin ninguna extensión de archivo.

4. El archivo de configuración debe residir en la carpeta `.kube` del perfil de usuario en la máquina local. Copie el archivo en esa carpeta del perfil de usuario.

    ![Ubicación del archivo de configuración en el cliente](media/azure-stack-edge-j-series-create-kubernetes-cluster/location-config-file.png)

5. Asocie el espacio de nombres con el usuario que ha creado. Escriba:

    `Grant-HcsKubernetesNamespaceAccess -Namespace <string> -UserName <string>`

    Este es una salida de ejemplo:

    `[10.100.10.10]: PS>Grant-HcsKubernetesNamespaceAccess -Namespace "myasetest1" -UserName "aseuser1"`

    Una vez que tenga el archivo de configuración, no necesita acceso físico al clúster. Si el cliente puede hacer ping a la dirección IP del dispositivo Azure Stack Edge Pro, debería poder dirigir el clúster mediante comandos de *kubectl*.

6. Inicie una nueva sesión de PowerShell en el cliente. No es necesario estar conectado a la interfaz del dispositivo. Ahora puede instalar `kubectl` en el cliente con el siguiente comando:

    ```powershell
    PS C:\windows\system32> curl https://storage.googleapis.com/kubernetes-release/release/v1.15.2/bin/windows/amd64/kubectl.exe -O kubectl.exe
    
    PS C:\windows\system32>
    ```
    Por ejemplo, si el nodo maestro de Kubernetes ejecutaba la versión 1.15.2, instale la versión 1.15.2 en el cliente.

    > [!IMPORTANT]
    > Descargue un cliente que no esté distanciado más de una versión secundaria de la del maestro. Sin embargo, la versión del cliente puede superar a la del maestro en hasta una versión secundaria. Por ejemplo, un maestro v1.3 debe funcionar con nodos v1.1, v1.2 y v1.3, y debe funcionar con clientes v1.2, v1.3 y v1.4. Para obtener información sobre la versión del cliente de Kubernetes, consulte [Kubernetes version and version skew support policy](https://kubernetes.io/docs/setup/release/version-skew-policy/#supported-version-skew) (Versión de Kubernetes y directiva de soporte de asimetría de versiones). Para más información sobre la versión del servidor de Kubernetes en Azure Stack Edge Pro, vaya a Obtención de la versión del servidor de Kubernetes.<!-- insert link-->
    > En ocasiones, `kubectl` está preinstalado en el sistema si se ejecuta Cliente Docker para Windows u otras herramientas. Es importante descargar la versión específica de `kubectl` como se indica en esta sección para que funcione con este clúster de Kubernetes. 

    La instalación puede tardar varios minutos.

7. Compruebe que la versión instalada sea la que ha descargado. Debe especificar la ruta de acceso absoluta a la ubicación en la que se instaló `kubectl.exe` en el sistema.
    
    ```powershell
    PS C:\Users\myuser> C:\windows\system32\kubectl.exe version
    Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:23:26Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"windows/amd64"}
    Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.1", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T09:09:21Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
    PS C:\Users\myuser>
    ``` 
    
    Para obtener más información sobre los comandos de `kubectl` que se usan para administrar el clúster de Kubernetes, vaya a [Overview of kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) (Información general de kubectl).

8. Agregue una entrada DNS al archivo de hosts en el sistema. 

    1. Ejecute el Bloc de notas como administrador y abra el archivo `hosts` ubicado en `C:\windows\system32\drivers\etc\hosts`. 
    2. Use la información que guardó de la página **Dispositivo** en la interfaz de usuario local en el paso anterior para crear la entrada en el archivo de hosts. 

        Por ejemplo, copie este punto de conexión `https://compute.asedevice.microsoftdatabox.com/[10.100.10.10]` para crear la siguiente entrada con la dirección IP y el dominio DNS del dispositivo: 

        `10.100.10.10 compute.asedevice.microsoftdatabox.com`

9. Para comprobar que puede conectarse a los pods de Kubernetes, escriba:
    
    ```powershell
    PS C:\Users\myuser> kubectl get pods -n "myasetest1"
    No resources found.
    PS C:\Users\myuser>
    ```
Ahora puede implementar las aplicaciones en el espacio de nombres y, a continuación, ver esas aplicaciones y sus registros.

> [!IMPORTANT]   
> Hay muchos comandos que no se podrán ejecutar, por ejemplo, los comandos que requieren tener acceso de administrador. Solo puede realizar las operaciones permitidas en el espacio de nombres.


## <a name="remove-kubernetes-cluster"></a>Eliminación del clúster de Kubernetes

Para quitar el clúster de Kubernetes, deberá quitar la configuración de proceso.

Para obtener instrucciones detalladas, vaya a [Eliminación de la configuración de proceso](azure-stack-edge-j-series-manage-compute.md#remove-compute-configuration).
   

## <a name="next-steps"></a>Pasos siguientes

- [Implementación de una aplicación sin estado en Azure Stack Edge Pro](azure-stack-edge-j-series-deploy-stateless-application-kubernetes.md).
