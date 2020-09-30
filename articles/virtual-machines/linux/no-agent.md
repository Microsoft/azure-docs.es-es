---
title: Creación de imágenes de Linux sin un agente de aprovisionamiento
description: Cree imágenes de Linux generalizadas sin un agente de aprovisionamiento en Azure.
author: danielsollondon
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 09/01/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: 63bc3caf97e1325c365171ba3f8e6353885d9b68
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322558"
---
# <a name="creating-generalized-images-without-a-provisioning-agent"></a>Creación de imágenes generalizadas sin un agente de aprovisionamiento

Microsoft Azure proporciona agentes de aprovisionamiento para máquinas virtuales Linux en forma de [walinuxagent](https://github.com/Azure/WALinuxAgent) o [cloud-init](https://github.com/canonical/cloud-init) (recomendado). Sin embargo, podría haber un escenario en el que no quiera usar ninguna de estas aplicaciones para el agente de aprovisionamiento, como:

- Su distribución o versión de Linux no admite el Agente Linux/cloud-init.
- Necesita que se establezcan propiedades de máquina virtual específicas, como el nombre de host.

> [!NOTE] 
>
> Si no necesita que se establezca ninguna propiedad ni que se produzca ningún tipo de aprovisionamiento, debería considerar la posibilidad de crear una imagen especializada.

En este artículo se muestra cómo puede configurar la imagen de la máquina virtual para cumplir los requisitos de la plataforma de Azure y establecer el nombre de host, sin instalar un agente de aprovisionamiento.

## <a name="networking-and-reporting-ready"></a>Redes e informes listos

Para que la máquina virtual Linux se comunique con los componentes de Azure, necesitará un cliente DHCP para recuperar una dirección IP del host de la red virtual, así como la resolución DNS y la administración de rutas. La mayoría de las distribuciones se envían con estas utilidades integradas. Entre las herramientas que han probado los proveedores de distribución de Linux en Azure se incluyen `dhclient`, `network-manager`, `systemd-networkd` y otras.

> [!NOTE]
>
> Actualmente, la creación de imágenes generalizadas sin un agente de aprovisionamiento solo admite máquinas virtuales habilitadas para DHCP.

Una vez que se hayan configurado las redes, debe "informar de que está listo". Esto indicará a Azure que la máquina virtual se ha aprovisionado correctamente.

> [!IMPORTANT]
>
> Si no informa a Azure de que está listo, se reiniciará la máquina virtual.

## <a name="demosample"></a>Demostración/ejemplo

En esta demostración se mostrará cómo puede tomar una imagen de Marketplace existente (en este caso, una máquina virtual Debian Buster) y quitar el Agente Linux (walinuxagent), pero también se mostrará cómo crear el proceso más básico para informar a Azure de que la máquina virtual está "lista".

### <a name="create-the-resource-group-and-base-vm"></a>Cree el grupo de recursos y la máquina virtual base:

```bash
$ az group create --location eastus --name demo1
```

Cree la máquina virtual base:

```bash
$ az vm create \
    --resource-group demo1 \
    --name demo1 \
    --location eastus \
    --ssh-key-value <ssh_pub_key_path> \
    --public-ip-address-dns-name demo1 \
    --image "debian:debian-10:10:latest"
```

### <a name="remove-the-image-provisioning-agent"></a>Eliminación del agente de aprovisionamiento de imágenes

Una vez que se aprovisiona la máquina virtual, puede hacer SSH en ella y quitar el Agente Linux:

```bash
$ sudo apt purge -y waagent
$ sudo rm -rf /var/lib/waagent /etc/waagent.conf /var/log/waagent.log
```

### <a name="add-required-code-to-the-vm"></a>Incorporación del código necesario a la máquina virtual

También dentro de la máquina virtual, dado que hemos quitado el Agente Linux de Azure, debemos proporcionar un mecanismo para informar de que estamos listos. 

#### <a name="python-script"></a>Script de Python

```python
import http.client
import sys
from xml.etree import ElementTree

wireserver_ip = '168.63.129.16'
wireserver_conn = http.client.HTTPConnection(wireserver_ip)

print('Retrieving goal state from the Wireserver')
wireserver_conn.request(
    'GET',
    '/machine?comp=goalstate',
    headers={'x-ms-version': '2012-11-30'}
)

resp = wireserver_conn.getresponse()

if resp.status != 200:
    print('Unable to connect with wireserver')
    sys.exit(1)

wireserver_goalstate = resp.read().decode('utf-8')

xml_el = ElementTree.fromstring(wireserver_goalstate)

container_id = xml_el.findtext('Container/ContainerId')
instance_id = xml_el.findtext('Container/RoleInstanceList/RoleInstance/InstanceId')
print(f'ContainerId: {container_id}')
print(f'InstanceId: {instance_id}')

# Construct the XML response we need to send to Wireserver to report ready.
health = ElementTree.Element('Health')
goalstate_incarnation = ElementTree.SubElement(health, 'GoalStateIncarnation')
goalstate_incarnation.text = '1'
container = ElementTree.SubElement(health, 'Container')
container_id_el = ElementTree.SubElement(container, 'ContainerId')
container_id_el.text = container_id
role_instance_list = ElementTree.SubElement(container, 'RoleInstanceList')
role = ElementTree.SubElement(role_instance_list, 'Role')
instance_id_el = ElementTree.SubElement(role, 'InstanceId')
instance_id_el.text = instance_id
health_second = ElementTree.SubElement(role, 'Health')
state = ElementTree.SubElement(health_second, 'State')
state.text = 'Ready'

out_xml = ElementTree.tostring(
    health,
    encoding='unicode',
    method='xml'
)
print('Sending the following data to Wireserver:')
print(out_xml)

wireserver_conn.request(
    'POST',
    '/machine?comp=health',
    headers={
        'x-ms-version': '2012-11-30',
        'Content-Type': 'text/xml;charset=utf-8',
        'x-ms-agent-name': 'custom-provisioning'
    },
    body=out_xml
)

resp = wireserver_conn.getresponse()
print(f'Response: {resp.status} {resp.reason}')

wireserver_conn.close()
```

#### <a name="generic-steps-without-using-python"></a>Pasos genéricos (sin usar Python)

Si la máquina virtual no tiene Python instalado ni tampoco disponible, puede reproducir esta lógica de script anterior mediante programación con los pasos siguientes:

1. Recupere el `ContainerId` y el `InstanceId` analizando la respuesta de WireServer: `curl -X GET -H 'x-ms-version: 2012-11-30' http://$168.63.129.16/machine?comp=goalstate`.

2. Construya los siguientes datos XML, insertando el `ContainerId` y el `InstanceId` analizados del paso anterior:
   ```xml
   <Health>
     <GoalStateIncarnation>1</GoalStateIncarnation>
     <Container>
       <ContainerId>CONTAINER_ID</ContainerId>
       <RoleInstanceList>
         <Role>
           <InstanceId>INSTANCE_ID</InstanceId>
           <Health>
             <State>Ready</State>
           </Health>
         </Role>
       </RoleInstanceList>
     </Container>
   </Health>
   ```

3. Publique estos datos en WireServer: `curl -X POST -H 'x-ms-version: 2012-11-30' -H "x-ms-agent-name: WALinuxAgent" -H "Content-Type: text/xml;charset=utf-8" -d "$REPORT_READY_XML" http://168.63.129.16/machine?comp=health`

### <a name="automating-running-the-code-at-first-boot"></a>Automatización de la ejecución del código en el primer arranque

En esta demostración se usa systemd, que es el sistema de inicialización más habitual en las distribuciones de Linux modernas. Así pues, la forma más sencilla y nativa de garantizar que este mecanismo para informar de que se está listo se ejecute en el momento adecuado es crear una unidad de servicio de systemd. Puede agregar el siguiente archivo de unidad a `/etc/systemd/system` (en este ejemplo, el archivo de unidad se denomina `azure-provisioning.service`):

```
[Unit]
Description=Azure Provisioning

[Service]
Type=oneshot
ExecStart=/usr/bin/python3 /usr/local/azure-provisioning.py
ExecStart=/bin/bash -c "hostnamectl set-hostname $(curl \
    -H 'metadata: true' \
    'http://169.254.169.254/metadata/instance/compute/name?api-version=2019-06-01&format=text')"
ExecStart=/usr/bin/systemctl disable azure-provisioning.service

[Install]
WantedBy=multi-user.target
```

Este servicio de systemd realiza tres acciones para el aprovisionamiento básico:

1. Informa a Azure de que está listo (para indicar que apareció correctamente).
1. Cambia el nombre de la máquina virtual en función del nombre de la máquina virtual proporcionado por el usuario extrayendo estos datos de [Azure Instance Metadata Service (IMDS)](https://docs.microsoft.com/azure/virtual-machines/linux/instance-metadata-service). **Tenga en cuenta** que IMDS también proporciona otros [metadatos de instancia](https://docs.microsoft.com/azure/virtual-machines/linux/instance-metadata-service#accessing-azure-instance-metadata-service), como las claves públicas SSH, por lo que puede establecer más de un nombre de host.
1. Se deshabilita por su cuenta, de modo que solo se ejecute en el primer arranque y no en reinicios posteriores.

Con la unidad en el sistema de archivos, ejecute lo siguiente para su habilitación:

```
$ sudo systemctl enable azure-provisioning.service
```

Ahora la máquina virtual está lista para generalizarse y tener una imagen creada a partir de ella. 

#### <a name="completing-the-preparation-of-the-image"></a>Finalización de la preparación de la imagen

De nuevo en la máquina de desarrollo, ejecute lo siguiente a fin de prepararse para la creación de imágenes a partir de la máquina virtual base:

```
$ az vm deallocate --resource-group demo1 --name demo1
$ az vm generalize --resource-group demo1 --name demo1
```

Asimismo, cree la imagen a partir de esta máquina virtual:

```
$ az image create \
    --resource-group demo1 \
    --source demo1 \
    --location eastus \
    --name demo1img
```

Ahora estamos listos para crear una nueva máquina virtual (o varias máquinas virtuales) a partir de la imagen:

```
$ IMAGE_ID=$(az image show -g demo1 -n demo1img --query id -o tsv)
$ az vm create \
    --resource-group demo12 \
    --name demo12 \
    --location eastus \
    --ssh-key-value <ssh_pub_key_path> \
    --public-ip-address-dns-name demo12 \
    --image "$IMAGE_ID" 
    --enable-agent false
```

> [!NOTE]
>
> Es importante establecer `--enable-agent` en `false`, ya que walinuxagent no existe en esta máquina virtual que se va a crear a partir de la imagen.

Esta máquina virtual debe llevar a cabo el aprovisionamiento correctamente. Al iniciar sesión en la máquina virtual recién aprovisionada, debería poder ver el resultado del servicio de systemd para informar de que se está listo:

```
$ sudo journalctl -u azure-provisioning.service
-- Logs begin at Thu 2020-06-11 20:28:45 UTC, end at Thu 2020-06-11 20:31:24 UTC. --
Jun 11 20:28:49 thstringnopa systemd[1]: Starting Azure Provisioning...
Jun 11 20:28:54 thstringnopa python3[320]: Retrieving goal state from the Wireserver
Jun 11 20:28:54 thstringnopa python3[320]: ContainerId: 7b324f53-983a-43bc-b919-1775d6077608
Jun 11 20:28:54 thstringnopa python3[320]: InstanceId: fbb84507-46cd-4f4e-bd78-a2edaa9d059b._thstringnopa2
Jun 11 20:28:54 thstringnopa python3[320]: Sending the following data to Wireserver:
Jun 11 20:28:54 thstringnopa python3[320]: <Health><GoalStateIncarnation>1</GoalStateIncarnation><Container><ContainerId>7b324f53-983a-43bc-b919-1775d6077608</ContainerId><RoleInstanceList><Role><InstanceId>fbb84507-46cd-4f4e-bd78-a2edaa9d059b._thstringnopa2</InstanceId><Health><State>Ready</State></Health></Role></RoleInstanceList></Container></Health>
Jun 11 20:28:54 thstringnopa python3[320]: Response: 200 OK
Jun 11 20:28:56 thstringnopa bash[472]:   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
Jun 11 20:28:56 thstringnopa bash[472]:                                  Dload  Upload   Total   Spent    Left  Speed
Jun 11 20:28:56 thstringnopa bash[472]: [158B blob data]
Jun 11 20:28:56 thstringnopa2 systemctl[475]: Removed /etc/systemd/system/multi-user.target.wants/azure-provisioning.service.
Jun 11 20:28:56 thstringnopa2 systemd[1]: azure-provisioning.service: Succeeded.
Jun 11 20:28:56 thstringnopa2 systemd[1]: Started Azure Provisioning.
```

## <a name="support"></a>Soporte técnico

Si implementa su propio código o agente de aprovisionamiento, contará con el soporte técnico de este código. El soporte técnico de Microsoft solo investigará aquellas incidencias relacionadas con las interfaces de aprovisionamiento no disponibles. No dejamos de realizar mejoras y cambios en esta área, por lo que debe supervisar los cambios en el Agente Linux de Azure y cloud-init para aprovisionar los cambios de API.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte [Aprovisionamiento de Linux](provisioning.md).
