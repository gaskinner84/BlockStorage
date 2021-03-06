---

copyright:
  years: 2014, 2019
lastupdated: "2019-02-05"

keywords:

subcollection: BlockStorage

---
{:new_window: target="_blank"}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Gestión de {{site.data.keyword.blockstorageshort}}
{: #managingstorage}

Puede gestionar los volúmenes de {{site.data.keyword.blockstoragefull}} mediante el [{{site.data.keyword.slportal}} ![Icono de enlace externo](../../icons/launch-glyph.svg "Icono de enlace externo")](https://control.softlayer.com/){:new_window}.

## Visualización de los detalles de LUN de {{site.data.keyword.blockstorageshort}}

Puede ver un resumen de la información clave para el LUN de almacenamiento seleccionado, incluidas las prestaciones adicionales de instantánea y réplica que se añadieron al almacenamiento.

1. Pulse **Almacenamiento**, **{{site.data.keyword.blockstorageshort}}**.
2. Pulse el Nombre de LUN adecuado en la lista.

De manera alternativa, puede utilizar el mandato siguiente en la CLI SL.
```
# slcli block volume-detail --help
Uso: slcli block volume-detail [OPCIONES] ID_VOLUMEN

Opciones:
  -h, --help  Mostrar este mensaje y salir.
```

## Autorización de hosts para acceder a {{site.data.keyword.blockstorageshort}}

Los hosts "autorizados" son hosts a los que se les ha otorgado acceso a un LUN particular. Sin la autorización del host, no puede acceder ni utilizar el almacenamiento de su sistema. Autorizar a un host a que acceda a LUN genera el nombre de usuario, la contraseña y el nombre calificado iSCSI (IQN), que se necesitan para montar la conexión iSCSI de E/S de multivía de acceso (MPIO).

Puede autorizar y conectar hosts que estén ubicados en el mismo centro de datos que su almacenamiento. Puede tener varias cuentas, pero no puede autorizar a un host de una cuenta a que acceda a su almacenamiento en otra cuenta.
{:important}

1. Pulse **Almacenamiento** -> **{{site.data.keyword.blockstorageshort}}**, y pulse el Nombre de LUN.
2. Desplácese a la sección **Hosts autorizados** de la página.
3. En la parte derecha, pulse **Autorizar host**. Seleccione los hosts que pueden acceder a ese LUN determinado.

De manera alternativa, puede utilizar el mandato siguiente en la CLI SL.
```
# slcli block access-authorize --help
Uso: slcli block access-authorize [OPCIONES] ID_VOLUMEN

Opciones:
  -h, --hardware-id TEXTO    El id de un SoftLayer_Hardware que se va a autorizar
  -v, --virtual-id TEXTO     El id de un SoftLayer_Virtual_Guest que se va a autorizar
  -i, --ip-address-id TEXTO  El id de una SoftLayer_Network_Subnet_IpAddress
                            que se va a autorizar
  --ip-address TEXTO         Una dirección IP que se va a autorizar
  --help                    Mostrar este mensaje y salir.
```

## Visualización de la lista de hosts que están autorizados para acceder a un LUN de {{site.data.keyword.blockstorageshort}}

1. Pulse **Almacenamiento** -> **{{site.data.keyword.blockstorageshort}}**, y pulse el Nombre de LUN.
2. Desplácese hasta la sección **Hosts autorizados**.

Allí puede ver la lista de hosts, que actualmente tienen autorización para acceder al LUN. También puede ver la información de autenticación necesarias para realizar una conexión: nombre de usuario, contraseña y host de IQN. La dirección de destino aparece listada en la página **Detalle de almacenamiento**. Para NFS, la dirección de destino se describe como un nombre de DNS y, para iSCSI, es la dirección IP de Descubrir portal de destino.

De manera alternativa, puede utilizar el mandato siguiente en la CLI SL.
```
# slcli block access-list --help
Uso: slcli block access-list [OPCIONES] ID_VOLUMEN

Opciones:
  --sortby TEXTO  Columna por la que se debe ordenar
  --columns TEXTO Columnas que se deben visualizar. Opciones: id, name, type,
                  private_ip_address, source_subnet, host_iqn, username,
                  password, allowed_host_id
  -h, --help      Mostrar este mensaje y salir.
```

## Visualización del {{site.data.keyword.blockstorageshort}} al cual un host está autorizado

Puede ver los LUN a los cuales un host tiene acceso, incluida la información necesaria para realizar una conexión: nombre de LUN, tipo de almacenamiento, dirección de destino, capacidad y ubicación:

1. Pulse **Dispositivos** -> **Lista de dispositivos** en el [{{site.data.keyword.slportal}} ![Icono de enlace externo](../../icons/launch-glyph.svg "Icono de enlace externo")](http://control.softlayer.com/){:new_window} y pulse sobre el dispositivo adecuado.
2. Seleccione el separador **Almacenamiento**.

Se le presentará una lista de los LUN de almacenamiento a los cuales este host tiene acceso. La lista está agrupada por tipo de almacenamiento (bloque, archivo, otros). Puede autorizar más almacenamiento o puede eliminar el acceso pulsando **Acciones**.

## Montaje y desmontaje de {{site.data.keyword.blockstorageshort}}

En función del sistema operativo del host, siga las instrucciones adecuadas.

- [Conexión a LUN en Linux](/docs/infrastructure/BlockStorage?topic=BlockStorage-mountingLinux)
- [Conexión a LUN en CloudLinux](/docs/infrastructure/BlockStorage?topic=BlockStorage-mountingCloudLinux)
- [Conexión a LUN en Microsoft Windows](/docs/infrastructure/BlockStorage?topic=BlockStorage-mountingWindows)
- [Configuración de almacenamiento en bloque para la copia de seguridad con cPanel](/docs/infrastructure/BlockStorage?topic=BlockStorage-cPanelBackups)
- [Configuración de almacenamiento en bloque para la copia de seguridad con Plesk](/docs/infrastructure/BlockStorage?topic=BlockStorage-PleskBackups)


## Revocación del acceso de un host a {{site.data.keyword.blockstorageshort}}

Si quiere detener el acceso desde un host a un LUN de almacenamiento determinado, puede revocar el acceso. Al revocar el acceso, la conexión de host se descarta del LUN. El sistema operativo y las aplicaciones de ese host ya no se pueden comunicar con el LUN.

Para evitar problemas del lado del host, desmonte el LUN de almacenamiento desde el sistema operativo antes de revocar el acceso para evitar perder unidades o corrupción de datos.
{:important}

Puede revocar el acceso desde la **Lista de dispositivos** o desde la **Vista de almacenamiento**.

### Revocación del acceso de la lista de dispositivos

1. Pulse **Dispositivos**, **Lista de dispositivos** desde el [{{site.data.keyword.slportal}} ![Icono de enlace externo](../../icons/launch-glyph.svg "Icono de enlace externo")](https://control.softlayer.com/){:new_window} y efectúe una doble pulsación sobre el dispositivo adecuado.
2. Seleccione el separador **Almacenamiento**.
3. Se le presentará una lista de los LUN de almacenamiento a los cuales este host tiene acceso. La lista está agrupada por tipo de almacenamiento (bloque, archivo, otros). Junto al nombre de LUN, seleccione **Acción** y pulse Revocar acceso**.
4. Confirme que desea revocar el acceso al LUN porque la acción no puede deshacerse. Pulse **Sí** para revocar el acceso al LUN o **No** para cancelar la acción.

Si desea desconectar varios LUN desde un host específico, debe repetir la acción Revocar acceso para cada LUN.
{:tip}


### Revocación del acceso de la vista de almacenamiento

1. Pulse **Almacenamiento**, **{{site.data.keyword.blockstorageshort}}** y seleccione el LUN desde el cual desea revocar el acceso.
2. Desplácese hasta la sección **Hosts autorizados**.
3. Pulse **Acciones** junto al host cuyo acceso se va a revocar y seleccione **Revocar acceso**.
4. Confirme que desea revocar el acceso al LUN porque la acción no puede deshacerse. Pulse **Sí** para revocar el acceso al LUN o **No** para cancelar la acción.

Si desea desconectar varios hosts de un LUN específico, debe repetir la acción Revocar acceso para cada host.
{:tip}

### Revocación del acceso a través de la CLI SL.

De manera alternativa, puede utilizar el mandato siguiente en la CLI SL.
```
# slcli block access-revoke --help
Uso: slcli block access-revoke [OPCIONES] ID_VOLUMEN

Opciones:
  -h, --hardware-id TEXTO    El id de un SoftLayer_Hardware cuya autorización se va a revocar
  -v, --virtual-id TEXTO     El id de un SoftLayer_Virtual_Guest cuya autorización se va a revocar
  -i, --ip-address-id TEXTO  El id de una SoftLayer_Network_Subnet_IpAddress
                            cuya autorización se va a revocar
  --ip-address TEXTO         Una dirección IP cuya autorización se va a revocar
  --help                    Mostrar este mensaje y salir.
```

## Cancelación de un LUN de almacenamiento

Si ya no necesita un LUN específico, puede cancelarlo en cualquier momento.

Para cancelar un LUN de almacenamiento, es necesario revocar el acceso de los hosts en primer lugar.
{:important}

1. Pulse **Almacenamiento**, **{{site.data.keyword.blockstorageshort}}**.
2. Pulse **Acciones** para el LUN que se va a cancelar y seleccione **Cancelar {{site.data.keyword.blockstorageshort}}**.
3. Confirme si desea cancelar el LUN inmediatamente o en la fecha de aniversario en la que se suministró el LUN.

   Si selecciona la opción de cancelar el LUN en su fecha de aniversario, puede anular la solicitud de cancelación antes de su fecha de aniversario.
   {:tip}
4. Pulse **Continuar** o **Cerrar**.
5. Marque el recuadro de selección **Acuse de recibo** y pulse **Confirmar**.

De manera alternativa, puede utilizar el mandato siguiente en la CLI SL.
```
# slcli block volume-cancel --help
Uso: slcli block volume-cancel [OPCIONES] ID_VOLUMEN

Opciones:
  --reason TEXTO Una razón para la cancelación (opcional)
  --immediate    Cancelar el volumen de almacenamiento de bloques inmediatamente en lugar de
                 hacerlo en el aniversario de facturación
  -h, --help     Mostrar este mensaje y salir.
```
