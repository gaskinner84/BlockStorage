---

copyright:
  years: 2018, 2019
lastupdated: "2019-02-05"

keywords:

subcollection: BlockStorage

---
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Conexión a los LUN iSCSI en CloudLinux
{: #mountingCloudLinux}

Siga estas instrucciones para instalar el LUN de iSCSI con la multivía de acceso en el release 6.10 de CloudLinux Server.

Antes de empezar, asegúrese de que el host que está accediendo al volumen de {{site.data.keyword.blockstoragefull}} se haya autorizado previamente a través del [{{site.data.keyword.slportal}} ![Icono de enlace externo](../../icons/launch-glyph.svg "Icono de enlace externo")](https://control.softlayer.com/){:new_window}.
{:tip}

1. Inicie la sesión en el [{{site.data.keyword.slportal}} ![Icono de enlace externo](../../icons/launch-glyph.svg "Icono de enlace externo")](https://control.softlayer.com/){:new_window}.
2. En la página de listado de {{site.data.keyword.blockstorageshort}}, localice el nuevo volumen y pulse **Acciones**.
3. Pulse **Autorizar host**.
4. En la lista, seleccione el host o los hosts que pueden acceder al volumen y pulse **Enviar**.
5. Anote el nombre calificado iSCSI (IQN) del host, nombre de usuario, contraseña y dirección de destino.

De manera alternativa, puede autorizar el host a través de la SLCLI.
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
{:codeblock}

Es mejor ejecutar el tráfico de almacenamiento en una VLAN, que omita el cortafuegos. La ejecución del tráfico de almacenamiento a través de cortafuegos de software aumenta la latencia y afecta negativamente al rendimiento del almacenamiento.
{:important}

## Montaje de volúmenes de {{site.data.keyword.blockstorageshort}}
{: #mountingCloudLin}

1. Instale los programas de utilidad multivía e iSCSI en el host y actívelos.
   ```
   yum install iscsi-initiator-utils
   ```
   {: pre}

   ```
   yum install multipath-tools

   ```
   {: pre}

   ```
   chkconfig multipathd on
   ```
   {: pre}

   ```
   chkconfig iscsid on
   ```
   {: pre}

2. Cree o edite los archivos de configuración.
   - Actualice '/etc/multipath.conf'. <br/>**Nota**: Todos los datos de la lista negra deben ser específicos de su sistema.
     ```
     defaults {
        user_friendly_names no
        flush_on_last_del       yes
        queue_without_daemon    no
        dev_loss_tmo            infinity
        fast_io_fail_tmo        5
     }
     blacklist {
        wwid "SAdaptec*"
   devnode "^hd[a-z]"
   devnode "^(ram|raw|loop|fd|md|dm-|sr|scd|st)[0-9]*"
        devnode "^cciss.*"
   }
   devices {
     device {
        vendor "NETAPP"
   product "LUN"
   path_grouping_policy group_by_prio
   features "3 queue_if_no_path pg_init_retries 50"
   prio "alua"
   path_checker tur
   failback immediate
   path_selector "round-robin 0"
   hardware_handler "1 alua"
   rr_weight uniform
   rr_min_io 128
   }
     }
     ```
     {: codeblock}

   - Actualice los valores de CHAP `/etc/iscsi/iscsid.conf` añadiendo el nombre de usuario, la contraseña.

     ```
     iscsid.startup = /etc/rc.d/init.d/iscsid force-start
     node.startup = automatic
     node.leading_login = No
     node.session.auth.authmethod = CHAP
     node.session.auth.username = <USER NAME VALUE FROM PORTAL>
     node.session.auth.password = <PASSWORD VALUE FROM PORTAL>
     discovery.sendtargets.auth.authmethod = CHAP
     discovery.sendtargets.auth.username = <USER NAME VALUE FROM PORTAL>
     discovery.sendtargets.auth.password = <PASSWORD VALUE FROM PORTAL>
     ```
     {: codeblock}

     Utilice mayúsculas para los nombres de CHAP. Deje los otros valores de CHAP que se han comentado. El almacenamiento de {{site.data.keyword.BluSoftlayer_full}} utiliza únicamente autenticación unidireccional. No habilite Mutual CHAP.
     {:important}


3. Reinicie los servicios `iscsi` y `multipathd`.
   ```
   /etc/init.d/iscsi restart   
   ```
   {: pre}

   ```
   /etc/init.d/multipathd restart   
   ```
   {: pre}

4. Descubra el dispositivo utilizando la dirección IP de destino obtenida desde el {{site.data.keyword.slportal}}.

     A. Ejecute el descubrimiento en la matriz de iSCSI.
       ```
       iscsiadm -m discovery -t sendtargets -p <valor-ip-del-Portal-SL>
       ```
       {: pre}

        Salida de ejemplo
       ```
       # iscsiadm -m discovery -t sendtargets -p 161.26.98.105
       161.26.98.105:3260,1026 iqn.1992-08.com.netapp:stfdal1002
       161.26.98.108:3260,1029 iqn.1992-08.com.netapp:stfdal1002
       ```

     B. Establezca el host para que inicie sesión automáticamente en la matriz de iSCSI.
       ```
       iscsiadm -m node -L automatic
       ```
       {: pre}

5. Verifique que el host ha iniciado sesión en la matriz de iSCSI y que ha mantenido sus sesiones.
   ```
   iscsiadm -m session
   ```
   {: pre}

   Salida de ejemplo
   ```
   tcp: [1] 161.26.98.105:3260,1026 iqn.1992-08.com.netapp:stfdal1002 (non-flash)
   tcp: [2] 161.26.98.108:3260,1029 iqn.1992-08.com.netapp:stfdal1002 (non-flash)
   ```


6. Verifique que el dispositivo está conectado.
   ```
   fdisk -l
   ```
   {: pre}

   Salida de ejemplo
   ```
   Disk /dev/sda: 999.7 GB, 999653638144 bytes
   255 heads, 63 sectors/track, 121534 cylinders
   Units = cylinders of 16065 * 512 = 8225280 bytes
   Sector size (logical/physical): 512 bytes / 512 bytes
   I/O size (minimum/optimal): 262144 bytes / 262144 bytes
   Disk identifier: 0x00040f06

   Device Boot      Start         End      Blocks   Id  System
   /dev/sda1   *           1          33      262144   83  Linux
   Partition 1 does not end on cylinder boundary.
   /dev/sda2              33         164     1048576   82  Linux swap / Solaris
   Partition 2 does not end on cylinder boundary.
   /dev/sda3             164      121535   974912512   83  Linux

   Disk /dev/sdb: 21.5 GB, 21474836480 bytes
   64 heads, 32 sectors/track, 20480 cylinders
   Units = cylinders of 2048 * 512 = 1048576 bytes
   Sector size (logical/physical): 512 bytes / 4096 bytes
   I/O size (minimum/optimal): 4096 bytes / 65536 bytes
   Disk identifier: 0x00000000

   Disk /dev/sdc: 21.5 GB, 21474836480 bytes
   64 heads, 32 sectors/track, 20480 cylinders
   Units = cylinders of 2048 * 512 = 1048576 bytes
   Sector size (logical/physical): 512 bytes / 4096 bytes
   I/O size (minimum/optimal): 4096 bytes / 65536 bytes
   Disk identifier: 0x00000000
   ```

   El volumen ahora está montado y es accesible en el host.

7. Verifique si MPIO se ha configurado correctamente listando los dispositivos. Si la configuración es correcta, sólo se mostrarán dos dispositivos NETAPP.

   ```
   # multipath -l
   ```
   {: pre}

   Salida de ejemplo
   ```
   root@server:~# multipath -l
   3600a098038304454515d4b6a5a444e35 dm-0 NETAPP,LUN C-Mode
   size=20G features='3 queue_if_no_path pg_init_retries 50' hwhandler='1 alua' wp=rw
   |-+- policy='round-robin 0' prio=50 status=active
   | `- 1:0:0:1 sdb 8:16 active ready running
   `-+- policy='round-robin 0' prio=10 status=enabled
   `- 2:0:0:1 sdc 8:32 active ready running
   ```
