---
title: Solución de problemas de implementación de OpenShift Container Platform 3.11
description: Solución de problemas de implementación de OpenShift Container Platform 3.11.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/14/2019
ms.author: haroldw
ms.custom: devx-track-ansible
ms.openlocfilehash: 9595627e9d7ca2de577aa83ebba3dd58d69e6750
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373562"
---
# <a name="troubleshoot-openshift-container-platform-311-deployment-in-azure"></a>Solución de problemas de implementación de OpenShift Container Platform 3.11

Si el clúster de OpenShift no se implementa correctamente, Azure Portal proporcionará una salida de error. La salida puede ser difícil de leer, lo que hace difícil identificar el problema. Examine rápidamente esta salida en busca de un código de salida 3, 4 o 5. El siguiente contenido proporciona información sobre estos tres códigos de salida:

- Código de salida 3: el nombre de usuario de suscripción de Red Hat/contraseña o id. de organización/clave de activación es incorrecto.
- Código de salida 4: el id. de grupo de Red Hat es incorrecto o no hay derechos disponibles.
- Código de salida 5: no se puede aprovisionar el volumen de grupo fino de Docker.

Para los demás códigos de salida, conéctese a los hosts mediante ssh para ver los archivos de registro.

**OpenShift Container Platform 3.11**

Conéctese mediante SSH al host del cuaderno de estrategias de Ansible. Para la plantilla o la oferta de Marketplace, use el host de tipo bastión. Desde el host de tipo bastión puede acceder mediante SSH a todos los demás nodos del clúster (master, infraestructura, CNS, proceso). Deberá ser el usuario root para ver los archivos de registro. El usuario root está deshabilitado para el acceso mediante SSH de forma predeterminada, por lo que no debe utilizarlo para conectarse mediante SSH a otros nodos.

**OKD**

Conéctese mediante SSH al host del cuaderno de estrategias de Ansible. Para la plantilla de OKD (versión 3.9 y anteriores), use el host master-0. Para la plantilla de OKD (versión 3.10 y versiones posterior), use el host de tipo bastión. Desde el host del cuaderno de estrategias de Ansible puede acceder mediante SSH a todos los demás nodos del clúster (master, infraestructura, CNS, proceso). Deberá ser el usuario root (sudo su -) para ver los archivos de registro. El usuario root está deshabilitado para el acceso mediante SSH de forma predeterminada, por lo que no debe utilizarlo para conectarse mediante SSH a otros nodos.

## <a name="log-files"></a>Archivos de registro

Los archivos de registro (stderr y stdout) para los scripts de preparación del host se encuentran en `/var/lib/waagent/custom-script/download/0` en todos los hosts. Si se produjo un error durante la preparación del host, puede ver estos archivos de registro para determinar el error.

Si los scripts de preparación se han ejecutado correctamente, deberá examinar los archivos de registro del directorio `/var/lib/waagent/custom-script/download/1` del host del cuaderno de estrategias de Ansible. Si se produjo el error durante la instalación actual de OpenShift, el archivo stdout mostrará el error. Utilice esta información para ponerse en contacto con soporte técnico para más ayuda.

Salida de ejemplo

```json
TASK [openshift_storage_glusterfs : Load heketi topology] **********************
fatal: [mycluster-master-0]: FAILED! => {"changed": true, "cmd": ["oc", "--config=/tmp/openshift-glusterfs-ansible-IbhnUM/admin.kubeconfig", "rsh", "--namespace=glusterfs", "deploy-heketi-storage-1-d9xl5", "heketi-cli", "-s", "http://localhost:8080", "--user", "admin", "--secret", "VuoJURT0/96E42Vv8+XHfsFpSS8R20rH1OiMs3OqARQ=", "topology", "load", "--json=/tmp/openshift-glusterfs-ansible-IbhnUM/topology.json", "2>&1"], "delta": "0:00:21.477831", "end": "2018-05-20 02:49:11.912899", "failed": true, "failed_when_result": true, "rc": 0, "start": "2018-05-20 02:48:50.435068", "stderr": "", "stderr_lines": [], "stdout": "Creating cluster ... ID: 794b285745b1c5d7089e1c5729ec7cd2\n\tAllowing file volumes on cluster.\n\tAllowing block volumes on cluster.\n\tCreating node mycluster-cns-0 ... ID: 45f1a3bfc20a4196e59ebb567e0e02b4\n\t\tAdding device /dev/sdd ... OK\n\t\tAdding device /dev/sde ... OK\n\t\tAdding device /dev/sdf ... OK\n\tCreating node mycluster-cns-1 ... ID: 596f80d7bbd78a1ea548930f23135131\n\t\tAdding device /dev/sdc ... Unable to add device: Unable to execute command on glusterfs-storage-4zc42:   Device /dev/sdc excluded by a filter.\n\t\tAdding device /dev/sde ... OK\n\t\tAdding device /dev/sdd ... OK\n\tCreating node mycluster-cns-2 ... ID: 42c0170aa2799559747622acceba2e3f\n\t\tAdding device /dev/sde ... OK\n\t\tAdding device /dev/sdf ... OK\n\t\tAdding device /dev/sdd ... OK", "stdout_lines": ["Creating cluster ... ID: 794b285745b1c5d7089e1c5729ec7cd2", "\tAllowing file volumes on cluster.", "\tAllowing block volumes on cluster.", "\tCreating node mycluster-cns-0 ... ID: 45f1a3bfc20a4196e59ebb567e0e02b4", "\t\tAdding device /dev/sdd ... OK", "\t\tAdding device /dev/sde ... OK", "\t\tAdding device /dev/sdf ... OK", "\tCreating node mycluster-cns-1 ... ID: 596f80d7bbd78a1ea548930f23135131", "\t\tAdding device /dev/sdc ... Unable to add device: Unable to execute command on glusterfs-storage-4zc42:   Device /dev/sdc excluded by a filter.", "\t\tAdding device /dev/sde ... OK", "\t\tAdding device /dev/sdd ... OK", "\tCreating node mycluster-cns-2 ... ID: 42c0170aa2799559747622acceba2e3f", "\t\tAdding device /dev/sde ... OK", "\t\tAdding device /dev/sdf ... OK", "\t\tAdding device /dev/sdd ... OK"]}

PLAY RECAP *********************************************************************
mycluster-cns-0       : ok=146  changed=57   unreachable=0    failed=0   
mycluster-cns-1       : ok=146  changed=57   unreachable=0    failed=0   
mycluster-cns-2       : ok=146  changed=57   unreachable=0    failed=0   
mycluster-infra-0     : ok=143  changed=55   unreachable=0    failed=0   
mycluster-infra-1     : ok=143  changed=55   unreachable=0    failed=0   
mycluster-infra-2     : ok=143  changed=55   unreachable=0    failed=0   
mycluster-master-0    : ok=502  changed=198  unreachable=0    failed=1   
mycluster-master-1    : ok=348  changed=140  unreachable=0    failed=0   
mycluster-master-2    : ok=348  changed=140  unreachable=0    failed=0   
mycluster-node-0      : ok=143  changed=55   unreachable=0    failed=0   
mycluster-node-1      : ok=143  changed=55   unreachable=0    failed=0   
localhost                  : ok=13   changed=0    unreachable=0    failed=0   

INSTALLER STATUS ***************************************************************
Initialization             : Complete (0:00:39)
Health Check               : Complete (0:00:24)
etcd Install               : Complete (0:01:24)
Master Install             : Complete (0:14:59)
Master Additional Install  : Complete (0:01:10)
Node Install               : Complete (0:10:58)
GlusterFS Install          : In Progress (0:03:33)
    This phase can be restarted by running: playbooks/openshift-glusterfs/config.yml

Failure summary:

  1. Hosts:    mycluster-master-0
     Play:     Configure GlusterFS
     Task:     Load heketi topology
     Message:  Failed without returning a message.
```

Los errores más comunes durante la instalación son:

1. La clave privada tiene frase de contraseña
2. El secreto de Key Vault con la clave privada no se ha creado correctamente
3. Las credenciales de la entidad de servicio se especificaron incorrectamente
4. La entidad de servicio no tiene acceso de colaborador al grupo de recursos

### <a name="private-key-has-a-passphrase"></a>La clave privada tiene una frase de contraseña

Verá un error que indica que se denegó el permiso de acceso mediante SSH. Conéctese mediante SSH al host del cuaderno de estrategias de Ansible para buscar una frase de contraseña de la clave privada.

### <a name="key-vault-secret-with-private-key-wasnt-created-correctly"></a>El secreto de Key Vault con la clave privada no se ha creado correctamente

La clave privada se copia en el host del cuaderno de estrategias de Ansible: ~/.ssh/id_rsa. Confirme que este archivo es correcto. Para comprobarlo, abra una sesión de SSH en uno de los nodos del clúster desde el host del cuaderno de estrategias de Ansible.

### <a name="service-principal-credentials-were-entered-incorrectly"></a>Las credenciales de la entidad de servicio se especificaron incorrectamente

Al proporcionar la entrada a la plantilla o la oferta de Marketplace, se proporcionó la información incorrecta. Asegúrese de que usa el identificador de aplicación (clientId) y la contraseña (clientSecret) correctas para la entidad de servicio. Puede verificarlo con el siguiente comando de la CLI de Azure.

```azurecli
az login --service-principal -u <client id> -p <client secret> -t <tenant id>
```

### <a name="service-principal-doesnt-have-contributor-access-to-the-resource-group"></a>La entidad de servicio no tiene acceso de colaborador al grupo de recursos

Si está habilitado el proveedor en la nube de Azure, la entidad de servicio utilizada debe tener acceso de colaborador al grupo de recursos. Puede verificarlo con el siguiente comando de la CLI de Azure.

```azurecli
az group update -g <openshift resource group> --set tags.sptest=test
```

## <a name="additional-tools"></a>Herramientas adicionales

Para algunos errores, también puede usar los comandos siguientes para extraer más información:

1. systemctl status \<service>
2. journalctl -xe
