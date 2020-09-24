# Security

## Security MySQL - Server (Standalone) (if it reachable through the internet)

  * keine anonymen Nutzer -> mysql_secure_installation 
    * mysql_secure_installation 
  * Sehr restriktives Anlegen von Benutzern
    * Benutzer nur von bestimmten IP's mit Zugriff zu bestimmter Datenbank/mehrere Datenbanken 
    * keine Wildcard benutzer, 
      * z.B. user@'%' 
      * oder: 'tabl%'@'%'
    * erlaubt:
      * user@'10.10.11.%'
  * Firewall absichern (nur Nutzer/Applikationen von bestimmten IP's dürfen auf den Server zugreifen 
    * nur Zugriff von den Applikations-Servern (IP-Adressen) 
    
```
# 10.10.11.124 - MySQL-Server 
# 10.10.11.120 - App-Server 
iptables -A INPUT -p tcp -s 10.10.11.120 --sport 1024:65535 -d 10.10.11.124 --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp -s 10.10.11.124 --sport 3306 -d 10.10.11.120 --dport 1024:65535 -m state --state ESTABLISHED -j ACCEPT
```

  * Absicherung durch SELinux (welche Files dürfen angefasst werden, welche Ports, welche weiteren Dienste) 
  
## Security MaxScale / MySQL Galera Cluster 

  * Galera Cluster niemals in der DMZ 
    * weil: es für das Galera Cluster keine Sicherheitsmechanismen gibt. 
    * einziger Ausweg (aber nicht vertretbar) : Bewaffnet mit Firewall - Regeln bis unter die Zähne 
  * MaxScale 
    * Nur den 3306-Port für Anfragen von dedizierten App-Servern öffnen, die in der DMZ stehen 
    * keine WebUI nach draussen
    * keine anderen Port nach draussen auf.
  * Achtung: readwritesplit kann nicht mit wildcard-Benutzern arbeiten: z.B. user@'%'
    * Wildcards ip's gehen aber: user@'10.10.11.%'
