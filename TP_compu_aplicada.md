## Router
### 1. Configurar la VM para que actue como router
1. identificar si existen interfaces de red instaladas
```bash
ip a
```
2. Activa el enrutamiento IP, esto permite que la máquina Debian enrute el tráfico entre las interfaces de red.
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```
3. Settear ips?
4. Usando iptables `sudo apt install iptables` configurar el forwarding the paquetes, estos comandos permiten que los paquetes se reenvíen entre las interfaces de red.
```bash
sudo iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
```
5. Configura el masquerading (NAT), esto permite que los paquetes salientes de la red local se enmascaren con la dirección IP de la interfaz de entrada.
```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
6. Guarda la configuración del firewall para asegurarte de que se cargue al reiniciar, es necesario tener instalado `iptables-persistent`
```bash
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```
7. Reiniciar el servicio de red
```bash
sudo systemctl restart networking
```
8. Para verificar la configuracion de network
```bash
ip a
```

### 2. Las políticas por defecto de las 3 cadenas de la tabla FILTER deben ser DROP
1. Inspecciona el archivo `/etc/iptables/rules.v4`
```bash
sudo nano /etc/iptables/rules.v4
```
2. Busca las líneas que definen las políticas por defecto para las cadenas INPUT, OUTPUT y FORWARD. Deberían verse así:
```
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
```
3. Cambia `ACCEPT` por `DROP` y deberia quedar asi
```
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT DROP [0:0]
```
4. Guarda el archivo y reinicia el servicio de `iptables`
```bash
sudo systemctl restart iptables
```

### 4. El tráfico desde/hacia la interfaz loopback deberá ser posible.
1. Añade reglas para permitir el tráfico en la interfaz loopback
```bash
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT
```
2. Guarda la configuracion en persistente
```bash
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```
3. Reinicia el servicio de red
```bash
sudo systemctl restart networking
```
4. para verificar las configuraciones
```bash
sudo iptables -L
```
### 5. Las únicas VMs que podrán acceder vía SSH a esta VM serán las del sector de IT
1. Abre el archivo de configuración SSH
```bash
sudo nano /etc/ssh/sshd_config
```
2. Buscar la linea q comienza con `#PermitRootLogin` y descomentarla
3. Agrega una linea abajo con los IPs de las maquina que permitiras acceso
```
AllowUsers root@<IP>
```
### 6. Se permitira iniciar conexiones desde la red LAN hacia la red WAN (Navegar por internet), no al reves. Para probar este punto, considere crear una ruta estatica en una maquina en la WAN simulada.
1. Para configurar las reglas de iptables de trafico saliente, esto elimina las reglas existentes
```bash
sudo iptables -F
sudo iptables -X
sudo iptables -t nat -F
sudo iptables -t nat -X
sudo iptables -t mangle -F
sudo iptables -t mangle -X
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```
2. Reiniciar el servicio de red
```bash
sudo systemctl restart networking
```