## Router
### 1. Configurar la VM para que actue como router
1. Configura la interfaz de red para conectarse a Internet (puede ser Ethernet o WiFi)
2. identificar si existen interfaces de red instaladas
```bash
ip a
```
3. Activa el enrutamiento IP, esto permite que la máquina Debian enrute el tráfico entre las interfaces de red.
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```
4. Settear ips?
5. Usando iptables `sudo apt install iptables` configurar el forwarding the paquetes, estos comandos permiten que los paquetes se reenvíen entre las interfaces de red.
```bash
sudo iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
```
6. Configura el masquerading (NAT), esto permite que los paquetes salientes de la red local se enmascaren con la dirección IP de la interfaz de entrada.
```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
7. Guarda la configuración del firewall para asegurarte de que se cargue al reiniciar, es necesario tener instalado `iptables-persistent`
```bash
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```
8. Reiniciar el servicio de red
```bash
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```
9. Para verificar la configuracion de network
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

### 3. Las configuraciones de iptables se deberán cargar automáticamente al iniciar las VMs.