# Containerlab desde cero: Open vSwitch y VLANs

Esta guía muestra cómo integrar **Open vSwitch (OVS)** con Containerlab para crear un switch virtual de Capa 2, conectar hosts Linux y utilizar VLANs para separar dominios de broadcast.

---

# 1. ¿Qué es Open vSwitch?

**Open vSwitch (OVS)** es un switch virtual open source para Linux.

Permite trabajar con conceptos de Capa 2 como:

- bridges;
- puertos;
- switching Ethernet;
- aprendizaje de direcciones MAC;
- VLANs.

En este laboratorio se utilizará OVS como switch virtual para conectar dos hosts Linux creados con Containerlab.

---

# 2. Verificar Open vSwitch

Comprobar si OVS está instalado:

```bash
ovs-vsctl --version
```

Si el comando no existe, en Ubuntu puede instalarse con:

```bash
sudo apt update
sudo apt install openvswitch-switch
```

Después verificar nuevamente:

```bash
ovs-vsctl --version
```

---

# 3. Crear el bridge OVS

Crear un bridge llamado:

```text
sw1
```

Ejecutar:

```bash
sudo ovs-vsctl add-br sw1
```

Verificar:

```bash
sudo ovs-vsctl show
```

Debería aparecer:

```text
Bridge sw1
```

También puede verificarse desde Linux:

```bash
ip link show sw1
```

---

# 4. Crear la topología

Crear el archivo:

```text
topologies/ovs-basico.clab.yml
```

Contenido:

```yaml
name: ovs-basico

topology:
  nodes:
    host1:
      kind: linux
      image: alpine:3.20

    host2:
      kind: linux
      image: alpine:3.20

    sw1:
      kind: ovs-bridge

  links:
    - endpoints: ["host1:eth1", "sw1:eth1"]
    - endpoints: ["host2:eth1", "sw1:eth2"]
```

La topología será:

```text
host1 ---- sw1 ---- host2
```

A nivel de interfaces:

```text
host1:eth1 ---- sw1:eth1
host2:eth1 ---- sw1:eth2
```

---

# 5. Desplegar la topología

Ejecutar:

```bash
sudo containerlab deploy -t topologies/ovs-basico.clab.yml
```

Inspeccionar:

```bash
sudo containerlab inspect -t topologies/ovs-basico.clab.yml
```

Los hosts Linux deberían aparecer en ejecución.

---

# 6. Verificar los puertos de OVS

Consultar el estado de Open vSwitch:

```bash
sudo ovs-vsctl show
```

Listar los puertos asociados con `sw1`:

```bash
sudo ovs-vsctl list-ports sw1
```

Resultado esperado:

```text
eth1
eth2
```

Esto indica que Containerlab conectó ambos enlaces al bridge OVS.

---

# 7. Configurar direcciones IPv4

Se utilizará la red:

```text
10.200.0.0/24
```

Asignación:

| Host | Interfaz | Dirección |
|---|---|---|
| host1 | eth1 | 10.200.0.1/24 |
| host2 | eth1 | 10.200.0.2/24 |

Configurar `host1`:

```bash
docker exec clab-ovs-basico-host1   ip addr add 10.200.0.1/24 dev eth1
```

Configurar `host2`:

```bash
docker exec clab-ovs-basico-host2   ip addr add 10.200.0.2/24 dev eth1
```

---

# 8. Verificar las direcciones

En `host1`:

```bash
docker exec clab-ovs-basico-host1   ip addr show eth1
```

Debe aparecer:

```text
10.200.0.1/24
```

En `host2`:

```bash
docker exec clab-ovs-basico-host2   ip addr show eth1
```

Debe aparecer:

```text
10.200.0.2/24
```

---

# 9. Probar switching básico

Desde `host1`:

```bash
docker exec clab-ovs-basico-host1   ping -c 3 10.200.0.2
```

Resultado esperado:

```text
3 packets transmitted
3 packets received
0% packet loss
```

Si el ping funciona, ambos hosts pueden comunicarse a través de `sw1`.

---

# 10. Consultar la tabla MAC

Open vSwitch mantiene una Forwarding Database o **FDB**.

Consultar:

```bash
sudo ovs-appctl fdb/show sw1
```

Después del tráfico generado por el ping deberían aparecer direcciones MAC aprendidas en los puertos del switch.

Conceptualmente:

```text
MAC de host1 -> eth1
MAC de host2 -> eth2
```

---

# 11. Consultar las MAC de los hosts

En `host1`:

```bash
docker exec clab-ovs-basico-host1   ip link show eth1
```

En `host2`:

```bash
docker exec clab-ovs-basico-host2   ip link show eth1
```

La dirección MAC aparece después de:

```text
link/ether
```

Estas direcciones pueden compararse con las entradas mostradas por:

```bash
sudo ovs-appctl fdb/show sw1
```

---

# 12. Crear separación mediante VLANs

Asignar `host1` a VLAN 10:

```bash
sudo ovs-vsctl set port eth1 tag=10
```

Asignar `host2` a VLAN 20:

```bash
sudo ovs-vsctl set port eth2 tag=20
```

La separación lógica queda:

```text
VLAN 10                VLAN 20

host1                  host2
  |                      |
eth1                   eth2
  |                      |
  +-------- sw1 ---------+
```

Aunque ambos hosts estén conectados al mismo switch físico o virtual, pertenecen a dominios de Capa 2 diferentes.

---

# 13. Verificar las VLANs

Ejecutar:

```bash
sudo ovs-vsctl show
```

Deberían aparecer configuraciones similares a:

```text
Port eth1
    tag: 10

Port eth2
    tag: 20
```

También pueden consultarse los puertos individualmente:

```bash
sudo ovs-vsctl list port eth1
```

```bash
sudo ovs-vsctl list port eth2
```

---

# 14. Comprobar el aislamiento

Manteniendo:

```text
host1 -> 10.200.0.1/24
host2 -> 10.200.0.2/24
```

probar:

```bash
docker exec clab-ovs-basico-host1   ping -c 3 10.200.0.2
```

El ping debería fallar.

La razón es que:

```text
host1 -> VLAN 10
host2 -> VLAN 20
```

Los hosts pertenecen a dominios de Capa 2 distintos.

Sin routing entre VLANs no existe comunicación entre ellos.

---

# 15. Colocar ambos hosts en la misma VLAN

Cambiar `eth2` a VLAN 10:

```bash
sudo ovs-vsctl set port eth2 tag=10
```

Ahora:

```text
host1 -> VLAN 10
host2 -> VLAN 10
```

Verificar:

```bash
sudo ovs-vsctl show
```

Deberían aparecer:

```text
Port eth1
    tag: 10

Port eth2
    tag: 10
```

---

# 16. Probar conectividad nuevamente

Desde `host1`:

```bash
docker exec clab-ovs-basico-host1   ping -c 3 10.200.0.2
```

El ping debería volver a funcionar.

Comparación:

```text
MISMA VLAN

host1 VLAN 10 <----> host2 VLAN 10
             OK
```

```text
VLAN DIFERENTE

host1 VLAN 10   X   host2 VLAN 20
            sin comunicación
```

---

# 17. Destruir el laboratorio

Cuando ya no sea necesario mantener la topología activa:

```bash
sudo containerlab destroy -t topologies/ovs-basico.clab.yml
```

Esto elimina los nodos y enlaces creados por Containerlab.

El bridge OVS `sw1` fue creado manualmente en el host, por lo que puede seguir existiendo después del `destroy`.

Comprobar:

```bash
sudo ovs-vsctl show
```

Si se desea eliminar también el bridge:

```bash
sudo ovs-vsctl del-br sw1
```

Verificar nuevamente:

```bash
sudo ovs-vsctl show
```

---

# 18. Resultado

Validaciones realizadas:

```text
Open vSwitch             OK
Bridge sw1               OK
Integración Containerlab OK
Puertos eth1 / eth2      OK
Direccionamiento IPv4    OK
Switching de Capa 2      OK
Aprendizaje MAC / FDB    OK
VLAN 10                  OK
VLAN 20                  OK
Aislamiento entre VLANs  OK
Misma VLAN               OK
Conectividad ICMP        OK
```