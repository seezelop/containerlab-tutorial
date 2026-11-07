# Containerlab desde cero: configuración reproducible con `exec`

Esta guía muestra cómo hacer que una topología de Containerlab configure automáticamente direcciones IPv4 durante el despliegue.

El objetivo es evitar configuraciones manuales posteriores al `deploy` y hacer que el laboratorio pueda reconstruirse siempre con el mismo resultado.

---

# 1. Topología inicial

Partimos de dos nodos Linux conectados entre sí:

```text
node01 -------- node02
```

Archivo:

```text
topologies/event.clab.yml
```

Contenido inicial:

```yaml
name: event-network

topology:
  nodes:
    node01:
      kind: linux
      image: alpine:3.20

    node02:
      kind: linux
      image: alpine:3.20

  links:
    - endpoints: ["node01:eth1", "node02:eth1"]
```

---

# 2. Problema de la configuración manual

Si las direcciones IPv4 se agregan manualmente después del despliegue con comandos como:

```bash
docker exec clab-event-network-node01   ip addr add 10.100.0.1/30 dev eth1
```

y:

```bash
docker exec clab-event-network-node02   ip addr add 10.100.0.2/30 dev eth1
```

esas configuraciones desaparecen cuando los contenedores se destruyen.

Por ejemplo:

```bash
sudo containerlab destroy -t topologies/event.clab.yml
```

Después de volver a desplegar:

```bash
sudo containerlab deploy -t topologies/event.clab.yml
```

las interfaces `eth1` vuelven a existir, pero las direcciones configuradas manualmente ya no están presentes.

---

# 3. Agregar configuración automática con `exec`

Modificar `topologies/event.clab.yml`:

```yaml
name: event-network

topology:
  nodes:
    node01:
      kind: linux
      image: alpine:3.20
      exec:
        - ip addr add 10.100.0.1/30 dev eth1

    node02:
      kind: linux
      image: alpine:3.20
      exec:
        - ip addr add 10.100.0.2/30 dev eth1

  links:
    - endpoints: ["node01:eth1", "node02:eth1"]
```

La sección:

```yaml
exec:
  - ip addr add 10.100.0.1/30 dev eth1
```

indica que Containerlab debe ejecutar ese comando dentro de `node01` durante el despliegue.

En `node02` ocurre lo mismo:

```yaml
exec:
  - ip addr add 10.100.0.2/30 dev eth1
```

---

# 4. Direccionamiento utilizado

Se utilizará la red:

```text
10.100.0.0/30
```

Asignación:

| Nodo | Interfaz | Dirección |
|---|---|---|
| node01 | eth1 | 10.100.0.1/30 |
| node02 | eth1 | 10.100.0.2/30 |

La topología queda:

```text
        10.100.0.0/30

node01 ---------------- node02
  eth1                    eth1
10.100.0.1              10.100.0.2
```

---

# 5. Destruir una instancia anterior

Si el laboratorio ya está desplegado, eliminarlo:

```bash
sudo containerlab destroy -t topologies/event.clab.yml
```

Verificar:

```bash
docker ps
```

Los contenedores:

```text
clab-event-network-node01
clab-event-network-node02
```

ya no deberían aparecer.

---

# 6. Desplegar nuevamente

Ejecutar:

```bash
sudo containerlab deploy -t topologies/event.clab.yml
```

En este caso no es necesario ejecutar manualmente comandos `ip addr add`.

Las direcciones IPv4 se configuran automáticamente mediante las instrucciones declaradas en `exec`.

---

# 7. Verificar la dirección de `node01`

Ejecutar:

```bash
docker exec clab-event-network-node01   ip addr show eth1
```

Debe aparecer:

```text
10.100.0.1/30
```

---

# 8. Verificar la dirección de `node02`

Ejecutar:

```bash
docker exec clab-event-network-node02   ip addr show eth1
```

Debe aparecer:

```text
10.100.0.2/30
```

---

# 9. Probar conectividad

Desde `node01` hacia `node02`:

```bash
docker exec clab-event-network-node01   ping -c 3 10.100.0.2
```

Resultado esperado:

```text
3 packets transmitted
3 packets received
0% packet loss
```

Desde `node02` hacia `node01`:

```bash
docker exec clab-event-network-node02   ping -c 3 10.100.0.1
```

Si ambos responden correctamente, las direcciones configuradas mediante `exec` están funcionando.

---

# 10. Comprobar la reproducibilidad

Destruir nuevamente el laboratorio:

```bash
sudo containerlab destroy -t topologies/event.clab.yml
```

Volver a desplegar:

```bash
sudo containerlab deploy -t topologies/event.clab.yml
```

Sin ejecutar ninguna configuración manual, comprobar otra vez:

```bash
docker exec clab-event-network-node01   ip addr show eth1
```

```bash
docker exec clab-event-network-node02   ip addr show eth1
```

Las direcciones deben volver a aparecer:

```text
node01 -> 10.100.0.1/30
node02 -> 10.100.0.2/30
```

Volver a probar conectividad:

```bash
docker exec clab-event-network-node01   ping -c 3 10.100.0.2
```

---

# 11. Resultado

La configuración queda declarada en el archivo YAML y puede reconstruirse automáticamente con cada despliegue.

Flujo:

```text
topologies/event.clab.yml
      ↓
containerlab deploy
      ↓
creación de nodos
      ↓
creación del enlace
      ↓
ejecución de exec
      ↓
configuración IPv4
      ↓
conectividad
```

Validaciones realizadas:

```text
Topología YAML           OK
Nodos Linux              OK
Enlace eth1              OK
Configuración automática OK
Direccionamiento IPv4    OK
Conectividad ICMP        OK
Recreación del lab       OK
```