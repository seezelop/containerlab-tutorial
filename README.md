# Containerlab Tutorial

Guía práctica e introductoria para aprender **Containerlab** construyendo laboratorios de red de forma progresiva.

El repositorio comienza con una topología mínima de dos nodos y avanza paso a paso hacia escenarios con:

- FRRouting;
- Open vSwitch;
- VLANs;
- trunks 802.1Q;
- routing entre VLANs;
- arquitectura de core y distribución;
- segmentación de usuarios y servicios;
- firewall y políticas con nftables.

La intención no es reproducir una red empresarial completa, sino aprender los fundamentos necesarios para construir laboratorios más avanzados con Containerlab.

---

## Objetivo

El proyecto busca responder una pregunta simple:

> ¿Cómo pasar de una topología básica en Containerlab a una red segmentada con routing, switching y políticas de seguridad?

Cada laboratorio agrega un concepto nuevo sobre una base previamente validada.

```text
2 nodos
   ↓
configuración reproducible
   ↓
routing
   ↓
switching
   ↓
VLANs
   ↓
trunks
   ↓
inter-VLAN routing
   ↓
core y distribución
   ↓
servicios y segmentación
   ↓
firewall y políticas
```

---

## Tecnologías utilizadas

- Containerlab
- Docker
- Alpine Linux
- FRRouting (FRR)
- Open vSwitch (OVS)
- Linux networking
- nftables
- Git / GitHub

---

## Requisitos

La guía fue desarrollada sobre Linux.

Se necesita tener disponibles:

```text
Docker
Containerlab
Open vSwitch
Git
```

Comandos útiles para verificar el entorno:

```bash
docker --version
containerlab version
ovs-vsctl --version
git --version
```

> Algunos laboratorios requieren privilegios `sudo` para crear bridges OVS o manipular interfaces del host.

---

## Estructura del repositorio

```text
containerlab-tutorial/
├── docs/
│   ├── 01-containerlab-primera-topologia.md
│   ├── 02-configuracion-reproducible-con-exec.md
│   ├── 03-frrouting-basico.md
│   ├── 04-open-vswitch-vlans.md
│   ├── 05-8021q-trunks.md
│   ├── 06-routing-entre-vlans.md
│   ├── 07-core-y-distribucion.md
│   ├── 08-servicios-y-segmentacion.md
│   ├── 09-firewall-y-politicas.md
│   └── 10-conclusiones-y-proximos-pasos.md
├── images/
├── topologies/
└── .gitignore
```

### `docs/`

Contiene la explicación paso a paso de cada laboratorio.

### `topologies/`

Contiene los archivos `.clab.yml` utilizados por Containerlab.

### `images/`

Contiene los diagramas que acompañan la documentación.

---

## Recorrido de la guía

| # | Tema | Conceptos principales |
|---|---|---|
| 01 | [Primera topología](docs/01-containerlab-primera-topologia.md) | Containerlab, nodos Linux, links, deploy, inspect |
| 02 | [Configuración reproducible con exec](docs/02-configuracion-reproducible-con-exec.md) | Automatización básica de configuración |
| 03 | [FRRouting básico](docs/03-frrouting-basico.md) | FRR, vtysh, interfaces, persistencia |
| 04 | [Open vSwitch y VLANs](docs/04-open-vswitch-vlans.md) | OVS, switching, puertos access |
| 05 | [Trunks 802.1Q](docs/05-8021q-trunks.md) | VLAN tagging, trunks entre switches |
| 06 | [Routing entre VLANs](docs/06-routing-entre-vlans.md) | Router-on-a-stick, subinterfaces |
| 07 | [Core y distribución](docs/07-core-y-distribucion.md) | Separación de funciones de red |
| 08 | [Servicios y segmentación](docs/08-servicios-y-segmentacion.md) | VLAN de servicios, segmentación lógica |
| 09 | [Firewall y políticas](docs/09-firewall-y-politicas.md) | nftables, forwarding, ACCEPT/DROP |
| 10 | [Conclusiones y próximos pasos](docs/10-conclusiones-y-proximos-pasos.md) | Resumen, límites y evoluciones |

---

## Cómo utilizar los laboratorios

Clonar el repositorio:

```bash
git clone https://github.com/seezelop/containerlab-tutorial.git
cd containerlab-tutorial
```

Cada laboratorio indica los pasos necesarios antes de desplegar la topología.

Ejemplo:

```bash
sudo containerlab deploy -t topologies/ovs-trunk.clab.yml
```

Inspeccionar:

```bash
sudo containerlab inspect -t topologies/ovs-trunk.clab.yml
```

Destruir:

```bash
sudo containerlab destroy -t topologies/ovs-trunk.clab.yml
```

> Es recomendable seguir los laboratorios en orden. Algunos conceptos posteriores reutilizan conocimientos introducidos en ejercicios anteriores.

---

## Metodología de trabajo

La guía sigue deliberadamente este ciclo:

```text
crear
  ↓
desplegar
  ↓
inspeccionar
  ↓
configurar
  ↓
validar
  ↓
agregar complejidad
  ↓
volver a validar
```

La conectividad se comprueba antes de agregar nuevas capas, por ejemplo antes de introducir reglas de firewall.

Esto ayuda a separar problemas de:

```text
topología
switching
routing
segmentación
políticas
```

---

## Troubleshooting

Durante los laboratorios aparecen situaciones reales de troubleshooting, entre ellas:

- interfaces OVS residuales;
- bridges que permanecen después de destruir una topología;
- pares `veth` residuales;
- rutas de management creadas por Containerlab;
- pérdida de configuración VLAN después de recrear un bridge;
- nombres de interfaces Linux demasiado largos;
- diferencias entre configuración persistente y configuración temporal dentro de un contenedor.

Cada caso se documenta dentro del laboratorio donde aparece.

---

## Segmentación y políticas

Uno de los objetivos de la serie es diferenciar dos conceptos:

```text
Segmentación -> separa redes
Firewall     -> decide qué tráfico puede atravesarlas
```

En los últimos laboratorios se llega a un escenario donde:

```text
VLAN 10 -> Servicios : ACCEPT
VLAN 20 -> Servicios : DROP
```

El resultado se valida tanto mediante conectividad como mediante los contadores de `nftables`.

---

## Alcance

Este repositorio es una **guía introductoria**.

No aborda en profundidad:

- automatización con Python;
- CI/CD con GitHub Actions;
- OSPF o BGP avanzados;
- redundancia;
- NAT y escenarios completos de Internet;
- IDS/IPS;
- NAC;
- 802.1X;
- RADIUS;
- observabilidad;
- imágenes de sistemas operativos de fabricantes.

Son posibles evoluciones del proyecto, pero quedan fuera del alcance de esta serie.

---

## Vendors y posibles evoluciones

Containerlab también puede utilizarse para construir laboratorios con imágenes y sistemas de distintos fabricantes.

Posibles extensiones incluyen:

```text
Nokia
Cisco
Juniper
```

También se puede evolucionar el repositorio incorporando:

```text
Python
GitHub Actions
validaciones automáticas
generación de configuraciones
captura de tráfico
monitoreo
pruebas de seguridad
```

---

## Estado del proyecto

La serie introductoria está compuesta por **9 laboratorios prácticos** más un documento final de conclusiones.

```text
01 -> 09   Laboratorios
10         Conclusiones y próximos pasos
```

---

## Uso

Los laboratorios están pensados para fines educativos y de experimentación en entornos controlados.

No deben interpretarse como configuraciones listas para entornos de producción.