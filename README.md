# Topologia3_IKEv1_GRE
https://www.youtube.com/watch?v=EGQntt083sI

Instituto Tecnologico
Documentacion Tecnica - Configuracion de VPN
VPN Site-to-Site IPSec IKEv1
Con Tunel GRE (GRE over IPsec)
Estudiante: Darwing Manuel Pena Reyes
Matricula: 2024-2690
Asignatura: Ethical Hacking 2
Fecha: Julio 2026
 1. Objetivo de la VPN
Establecer una conexion VPN site-to-site utilizando un tunel GRE para encapsular el trafico entre dos LANs remotas, protegido posteriormente por IPSec en modo transporte. Esta combinacion permite transportar trafico que IPSec puro no soporta de forma nativa (como multicast o protocolos de enrutamiento dinamico), aprovechando GRE como capa de encapsulamiento generico y IPSec como capa de cifrado.
2. Topologia de red
Topologia identica a las anteriores (R2 y R3 como peers, LAN y switch en cada extremo, router ISP intermedio), pero en este caso se crea un tunel GRE clasico (Tunnel50) entre las IPs publicas de R2 y R3, y ese tunel GRE completo se protege mediante un IPSec profile en modo transporte (ya que GRE ya realiza su propio encapsulamiento IP, no es necesario un segundo nivel de encapsulamiento tunel de IPSec).
2.1 Interfaces utilizadas
Dispositivo / Interfaz	Rol
R2 - F0/0	WAN, tunnel source hacia ISP
R2 - G2/0	LAN, hacia switch y PC1
R2 - Tunnel50	Tunel GRE protegido por IPSec
R3 - F1/0	WAN, tunnel source hacia ISP
R3 - G2/0	LAN, hacia switch y PC2
R3 - Tunnel50	Tunel GRE protegido por IPSec
<img width="857" height="554" alt="image" src="https://github.com/user-attachments/assets/27919c2c-0a08-43e3-9f3d-793122e01d08" />


2.2 Direccionamiento IP
Segmento	Red / IP
LAN R2	20.24.34.0/24 - Gateway .90 - PC1 .91
LAN R3	20.24.35.0/24 - Gateway .90 - PC2 .91
Enlace ISP-R2	172.16.34.0/30
Enlace ISP-R3	172.16.35.0/30
Tunnel GRE (Tunnel50)	10.34.34.0/30 (R2 .1 / R3 .2)


Explicacion: Captura general de la topologia armada en GNS3, mostrando todos los dispositivos, sus interfaces conectadas, y el recuadro con nombre y matricula del estudiante.
3. Parametros utilizados
Parametro	Valor configurado
Version IKE	IKEv1 (ISAKMP)
Metodo de autenticacion	Pre-shared key (CiscoVPN123)
Cifrado Fase 1	AES 256
Hash Fase 1	SHA256
Grupo Diffie-Hellman	Grupo 14
Transform-set (Fase 2)	esp-aes 256 + esp-sha256-hmac
Modo IPSec	Transport (GRE ya encapsula)
Protocolo transportado por IPSec	GRE (protocolo IP 47)

4. Configuracion y explicacion (capturas de pantalla)
Interfaz de tunel GRE protegida por IPSec
interface Tunnel50
 ip address 10.34.34.1 255.255.255.252
 tunnel source f0/0
 tunnel destination 172.16.35.2
 tunnel protection ipsec profile PROF-GRE

Explicacion: Se define un tunel GRE clasico (sin 'tunnel mode ipsec ipv4' como en la VTI), y se protege con un IPSec profile. Nota clave: al no especificar 'tunnel mode', Cisco IOS usa GRE por defecto en vez de un modo IPSec nativo.
Transform-set en modo transporte
crypto ipsec transform-set TS-GRE esp-aes 256 esp-sha256-hmac
 mode transport

Explicacion: Se usa modo transport en vez de tunnel, porque el paquete que IPSec va a proteger ya es un paquete GRE (que a su vez encapsula el trafico IP original). Anadir un segundo encapsulamiento tunel séria redundante.
5. Evidencia de funcionamiento
Identidades protegidas en la SA de IPSec
local  ident: (172.16.34.1/255.255.255.255/47/0)
remote ident: (172.16.35.2/255.255.255.255/47/0)
in use settings ={Transport, }
El protocolo 47 en los identificadores confirma que IPSec esta protegiendo especificamente el trafico GRE entre las IPs publicas de los routers, no el trafico de las LANs directamente.
Traceroute desde PC1
trace 20.24.35.91
 1   20.24.34.90
 2   10.34.34.2
 3   *20.24.35.91 (Destination port unreachable)
El tunel GRE (10.34.34.2) aparece como salto visible, confirmando que es una VPN route-based con una interfaz de tunel real, igual que en la VTI, pero encapsulando trafico en GRE antes de cifrarlo.
6. Conclusion
La VPN GRE sobre IPSec IKEv1 quedo funcional y verificada. El uso de modo transporte (en vez de tunnel) y la identificacion del protocolo 47 en las SAs confirma que IPSec esta protegiendo el encapsulamiento GRE, brindando una base flexible para transportar trafico adicional como multicast o rutas dinamicas en escenarios mas complejos.
