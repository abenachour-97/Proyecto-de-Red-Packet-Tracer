# 🌐 Topología Lógica — Trabajo final de grado ASIR

Red de un recinto de exposiciones con **2 pabellones**, simulada en **Cisco Packet Tracer**.
Este documento explica, de forma sencilla, cómo está organizada la red y cómo se ha configurado.

---

## 📌 ¿Qué se ha hecho?

Se ha montado una red **segmentada en VLANs**, con un router que permite (o no) que las VLANs se hablen entre sí. La idea es simple: **cada tipo de usuario tiene su propia red**, aunque compartan el mismo switch físico.

- ✅ 5 VLANs separadas por tipo de usuario
- ✅ 4 switches (1 core + 3 de acceso)
- ✅ 1 router con *Router-on-a-Stick* para comunicar las VLANs
- ✅ PCs de prueba en cada VLAN para verificar con `ping`

---

## 🗺️ Estructura de la red

La red tiene **3 zonas físicas**, cada una con su switch de acceso:

| Zona | Qué hay | Switch |
|------|---------|--------|
| 🏢 Pabellón 1 — Planta baja | Stands, cafetera, sala de presentaciones 3, demo, networking | `SW-PAB1` |
| 🏛️ Pabellón 1 — Oficina (planta superior) | Sala de reuniones, escritorios, zona directivos | `SW-OFICINA` |
| 🏢 Pabellón 2 | Stands, salas de presentaciones 1 y 2, cafetera, networking, entrada principal | `SW-PAB2` |

### Jerarquía de conexiones

<img width="637" height="646" alt="image" src="https://github.com/user-attachments/assets/f8280307-2acd-42ab-8f9b-0bcc80aac73a" />


## 🧩 VLANs

Las VLANs separan el tráfico por tipo de usuario. **Por defecto no pueden comunicarse entre sí**; solo lo hacen a través del router.

| VLAN | Nombre | Quién la usa | Red | Gateway |
|:----:|--------|--------------|-----|---------|
| **10** | Stands | Todos los stands | `192.168.10.0/24` | `192.168.10.1` |
| **20** | Organizacion | Oficina (planta superior) | `192.168.20.0/24` | `192.168.20.1` |
| **30** | Presentaciones | Salas de presentaciones | `192.168.30.0/24` | `192.168.30.1` |
| **40** | Invitados | Cafeteras + zonas de networking | `192.168.40.0/24` | `192.168.40.1` |
| **50** | Gestion | Switches y router (administración) | `192.168.50.0/24` | `192.168.50.1` |

### ¿Por qué se separan así?

- **Stands (10):** son terceros (expositores). Van aislados para que no vean equipos internos.
- **Organización (20):** la oficina maneja datos sensibles; necesita su propia red protegida.
- **Presentaciones (30):** las salas necesitan buena conectividad sin mezclarse con el resto.
- **Invitados (40):** tráfico público (wifi/cafetería). Es la zona menos confiable.
- **Gestión (50):** solo para administrar los equipos de red, separada del tráfico de usuarios.

---

## 💻 Direccionamiento de los PCs

| PC | VLAN | IP | Máscara | Gateway |
|----|:----:|----|---------|---------|
| PC-Stand-1 | 10 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC-Stand-2 | 10 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| PC-Stand-3 | 10 | 192.168.10.12 | 255.255.255.0 | 192.168.10.1 |
| PC-Oficina-1 | 20 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| PC-Oficina-2 | 20 | 192.168.20.11 | 255.255.255.0 | 192.168.20.1 |
| PC-Pres-1 | 30 | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| PC-Pres-2 | 30 | 192.168.30.11 | 255.255.255.0 | 192.168.30.1 |
| PC-Invitado-1 | 40 | 192.168.40.10 | 255.255.255.0 | 192.168.40.1 |
| PC-Invitado-2 | 40 | 192.168.40.11 | 255.255.255.0 | 192.168.40.1 |

---

## 🔧 ¿Cómo funciona? (resumen simple)

1. **VLANs:** se crean las 5 VLANs en los 4 switches.
2. **Puertos *access*:** cada puerto donde se conecta un PC se asigna a su VLAN.
3. **Puertos *trunk*:** los enlaces entre switches (y hacia el router) llevan el tráfico de **todas** las VLANs a la vez.
4. **Router-on-a-Stick:** el router usa **una sola interfaz física** dividida en **subinterfaces** (una por VLAN). Cada subinterfaz actúa como *gateway* de su VLAN y permite el tráfico entre ellas.

### Dispositivos usados

| Dispositivo | Modelo | Cantidad |
|-------------|--------|:--------:|
| Router | Cisco 2911 | 1 |
| Switch Core | Cisco 3560 / 2960 | 1 |
| Switch de acceso | Cisco 2960 | 3 |
| PCs de prueba | Generic PC | 9+ |

---

## ⌨️ Comandos principales

<details>
<summary><b>Crear VLANs (en los 4 switches)</b></summary>

```
enable
configure terminal
vlan 10
 name Stands
vlan 20
 name Organizacion
vlan 30
 name Presentaciones
vlan 40
 name Invitados
vlan 50
 name Gestion
end
```
</details>

<details>
<summary><b>Puerto access (hacia un PC)</b></summary>

```
interface FastEthernet 0/1
 switchport mode access
 switchport access vlan 10
```
</details>

<details>
<summary><b>Puerto trunk (entre switches / hacia el router)</b></summary>

```
interface GigabitEthernet 0/1
 switchport mode trunk
```
</details>

<details>
<summary><b>Router-on-a-Stick</b></summary>

```
interface GigabitEthernet 0/0
 no shutdown

interface GigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface GigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0

interface GigabitEthernet 0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0

interface GigabitEthernet 0/0.50
 encapsulation dot1Q 50
 ip address 192.168.50.1 255.255.255.0
```
</details>

---

## ✅ Verificación

Desde `PC-Stand-1` (`192.168.10.10`) → *Desktop → Command Prompt*:

| Prueba | Comando | Resultado esperado |
|--------|---------|--------------------|
| Gateway de su VLAN | `ping 192.168.10.1` | ✅ Responde |
| Misma VLAN | `ping 192.168.10.11` | ✅ Responde |
| Otra VLAN (Oficina) | `ping 192.168.20.10` | ✅ Responde vía router |
| Otra VLAN (Presentaciones) | `ping 192.168.30.10` | ✅ Responde vía router |

### 🛠️ Si algo falla

| Síntoma | Causa probable | Qué revisar |
|---------|----------------|-------------|
| Falla el ping en la misma VLAN | Puerto access mal configurado | `switchport access vlan X` |
| Falla el ping entre VLANs | Subinterfaces del router | `encapsulation dot1Q` e IPs |
| Falla todo | Trunk mal configurado | `switchport mode trunk` en el enlace switch–router |

---

## 📸 Capturas

<img width="1055" height="601" alt="image" src="https://github.com/user-attachments/assets/d41927f3-de89-45eb-895c-509a6d3b8ab1" />
<img width="835" height="582" alt="image" src="https://github.com/user-attachments/assets/66cd05d7-c06c-4014-91b3-8755e92aee50" />
<img width="1260" height="598" alt="image" src="https://github.com/user-attachments/assets/737c10b2-c22b-454f-9ba4-ece38cc8f1ef" />

## 📸 Pabellones:
Planos de las Plantas:
<img width="941" height="627" alt="image" src="https://github.com/user-attachments/assets/659131f1-1349-42ee-9f27-2125232979c3" />
<img width="929" height="423" alt="image" src="https://github.com/user-attachments/assets/37b11fc1-1a75-486b-b1d7-c68152facdc6" />
<img width="1353" height="674" alt="image" src="https://github.com/user-attachments/assets/5b8f03b1-088e-4052-a4ee-4737c95f1289" />

## RACKS:
<img width="717" height="660" alt="image" src="https://github.com/user-attachments/assets/a138fd96-e64e-4a06-93c0-665802bcfafa" />
<img width="739" height="641" alt="image" src="https://github.com/user-attachments/assets/8fbdacae-cff5-4974-9b90-17c6982f4b9c" />
<img width="741" height="655" alt="image" src="https://github.com/user-attachments/assets/0c559c04-d7ab-44f5-90c5-a3b080018379" />





