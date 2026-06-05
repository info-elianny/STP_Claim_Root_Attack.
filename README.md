# 🛡️ Ataque STP Claim Root

📹 [Video demostración](https://youtu.be/1ulKlf1zeLc) | 📂 [Repositorio](https://github.com/info-elianny/STP_Claim_Root_Attack)

---

## 📌 Objetivo del Laboratorio

Demostrar cómo un dispositivo puede intentar convertirse en el **Root Bridge** de una red que utiliza STP, con el fin de observar los cambios que se producen en la topología y comprender el impacto que esto puede tener en el funcionamiento de las comunicaciones.

### Topología de Red
![Topología de Red](https://i.postimg.cc/5N4fcsyj/stp-1.png)

---

## 🎯 Objetivo del Script

Capturar una trama STP legítima de la red, modificarla y reenviarla con una **prioridad inferior** para intentar que el equipo atacante sea reconocido como el Root Bridge de la topología, permitiendo observar cómo este cambio afecta el funcionamiento y la ruta del tráfico dentro de la red.

---

## ⚙️ Requisitos

### Hardware y Software

- Un equipo que actuará como **atacante** (Kali Linux)
- Un **switch** con Spanning Tree Protocol (STP) habilitado
- Al menos un equipo conectado al switch para generar tráfico normal
- Todos los dispositivos en la **misma red local**
- **Python 3** instalado
- Biblioteca **Scapy** instalada
- Permisos de **administrador (root)** para ejecutar el script

### Instalación de dependencias

```bash
pip install scapy
```

---

## 🔧 Parámetros Utilizados

| Parámetro | Descripción |
|-----------|-------------|
| Interfaz de red | Adaptador desde el cual se realizará el ataque |
| Cantidad de BPDUs | Número de tramas BPDU modificadas a enviar |
| Intervalo entre envíos | Tiempo en segundos entre cada BPDU falsa |
| BPDU capturada | Trama STP legítima usada como base para la modificación |
| Prioridad del Root Bridge | Valor configurado para presentarse como switch con mayor prioridad |
| MAC del Root Bridge | MAC configurada para anunciar al atacante como Root Bridge |
| Costo de ruta (Path Cost) | Valor ajustado para indicar la ruta más favorable al Root Bridge falso |

---

## 🚀 Cómo Ejecutar el Script

```bash
sudo python3 stp_claim_root.py
```

> ⚠️ **Debe ejecutarse con permisos root (sudo)**

---

## 📋 Funcionamiento del Script

**Paso 1:** El script comprueba que el usuario tenga permisos de administrador (root), ya que la captura y el envío de tramas de red requieren privilegios elevados.

**Paso 2:** Se solicita la **interfaz de red**, la **cantidad de BPDUs** a enviar y el **intervalo de tiempo** entre cada envío.

**Paso 3:** El script escucha el tráfico de la red hasta capturar una **BPDU válida** proveniente de un switch con STP habilitado, la cual servirá como base para la modificación.

**Paso 4:** Se crea una copia de la BPDU y se modifican sus campos para anunciar al atacante como el **Root Bridge**, asignándole una prioridad superior a la del resto de los dispositivos.

**Paso 5:** El script envía repetidamente las **BPDUs modificadas** para intentar que los switches reconozcan al atacante como el nuevo Root Bridge.

**Paso 6:** Durante la ejecución, se muestra en pantalla la cantidad de BPDUs enviadas y los cambios producidos en el switch objetivo.

---

## 📸 Capturas de Pantalla

### "show spanning-tree" antes del ataque
![STP antes del ataque](https://i.postimg.cc/FKQPqBFq/STP-CAP-1.png)

### Ataque corriendo
![Ataque corriendo](https://i.postimg.cc/sDkKQNWX/STP-CAP-2.png)

![Ataque corriendo 2](https://i.postimg.cc/DzFg54Bz/STP-CAP-2-2.png)

### "show spanning-tree" después del ataque
![STP después del ataque](https://i.postimg.cc/fLqfw2ZC/STP-CAP-3.png)

---

## 🌐 Documentación de la Red

| Dispositivo | Interfaz | Dirección IP | Máscara de Red |
|-------------|----------|--------------|----------------|
| R-1 | E0/0 | 6.6.1.1 | 255.255.255.0 |
| SW-1 | ---- | ----- | ----- |
| Kali Linux (Atacante) | e1 | 6.6.1.11 | 255.255.255.0 |
| Windows 7 (Víctima) | e0 | No se usó | No se usó |
| Cloud | net | 192.168.206.135 | 255.255.255.0 |

---

## 🛡️ Contramedidas para Mitigar el Ataque

### 1. BPDU Guard
Función aplicada en los puertos de acceso (donde se conectan dispositivos finales). Cuando un puerto con BPDU Guard habilitado recibe un paquete BPDU, el switch deshabilita inmediatamente ese puerto poniéndolo en estado **err-disabled**, evitando que un atacante envíe BPDUs maliciosas para proclamarse Root Bridge.

```
SW(config-if)# spanning-tree bpduguard enable
```

![BPDU Guard resultado](https://i.postimg.cc/G3B54Yf7/STP-M-1.png)

### 2. STP Port Security (Prioridad manual del Root Bridge)
Asignar manualmente la **prioridad más baja posible** al switch legítimo, garantizando que ningún otro dispositivo pueda ganar la elección STP.

```
SW(config)# spanning-tree vlan 1 priority 0
```

![STP Port Security resultado](https://i.postimg.cc/cLcDpcK4/STP-M-2.png)

### 3. Root Guard
Se aplica en puertos donde no se desea que aparezca un Root Bridge. Si recibe una BPDU superior, el puerto pasa a estado **root-inconsistent** y deja de reenviar tráfico, protegiendo al Root Bridge legítimo.

```
SW(config-if)# spanning-tree guard root
```

![Root Guard resultado](https://i.postimg.cc/B6RNTYzg/STP-M-3.png)

### 4. BPDU Filtering
Bloquea el envío y recepción de BPDUs en puertos específicos, evitando que dispositivos no autorizados participen en la negociación STP.

```
SW(config-if)# spanning-tree bpdufilter enable
```

![BPDU Filtering resultado](https://i.postimg.cc/8krZpGJS/STP-M-4.png)

---

> ⚠️ **Este script es únicamente con fines educativos y de investigación en entornos controlados.**  
> ⚠️ **BY: Elianny**
