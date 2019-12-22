# 1. Hardening Firewallu pomocou ufw

Firewall si nastavíme na Debiane 10 s IP adresou 192.168.0.1 pomocou UFW. 
Je potrebné spraviť základný update a upgrade systému a následne môžeme nainštalovať ufw. 

```
apt update && apt upgrade
apt install ufw
```

Prvé pravidlá, ktoré je potrebné zadefinovať sú defaultné pravidlá. V predvolenom nastavení je ufw nastavené tak, aby zakázalo všetky prichádzajúce pripojenia
a povolilo všetky odchádzajúce. Nastavíme teda ufw späť na defaultné hodnoty, aby sme začali od začiatku. 

```
ufw default deny incoming
ufw default allow outgoing
```

Na povolenie prichádzajúcích SSH spojení použijeme príkaz:

```
ufw allow ssh
```

Takto sa vytvoria pravidlá brány firewall, ktoré povolia všetky pripojenia na porte 22, čo je port, ktorý sa štandardne používa pre SSH. 

Na povolenie ufw je potrebné použiť tento príkaz: 

```
ufw enable
```

Povolíme aj iné pripojenia.

```
ufw allow http
ufw allow https
ufw allow 53
```

Dajú sa povoliť aj špecifické porty alebo ip adresy. Pri určovaní rozsahu portov pomocou UFW musíme špecifikovať protokol (tcp alebo udp), na ktorý by sa pravidlá mali vzťahovať.

```
ufw allow 6000:6007/tcp
ufw allow 6000:6007/udp
ufw allow from 203.0.113.4
ufw allow from 203.0.113.4 to any port 22
```

Zistíme si aké mamé sieťové rozhrania:

```
ip a 
```

Pre rozhranie enp0s3 povolíme HTTP a pre rozhranie enp0s8 povolíme MySQL: 

```
ufw allow in on enp0s3 to any port 80
ufw allow in on enp0s8 to any port 3306
```

Vieme tiež zakázať pripojenia: 

```
ufw deny http
ufw deny from 203.0.113.4
```

Nadefinované pravidlá môžeme aj vymazať. Najprv ale zistíme aké je číslovanie pravidiel: 

```
ufw status numberes
ufw delete 2
ufw delete allow http
ufw delete allow 80
```

Stav ufw si vieme kedykoľvek skontrolovať pomocou príkazu: 

```
ufw status verbose
```

Tiež vieme logovať ufw. Logy nájdeme v súbore /var/logs/ufw:

```
ufw logging on
cat /var/logs/ufw
```



# 2. Hardening SSH servera

Na inštaláciu a konfiguráciu ssh serveru som použila Debian 10, pričom statická IP adresa bola nastavená na 192.168.0.1.
Na to, aby všetko prebehlo v poriadku je potrebné byť prihlásený ako root. 

Najprv spravíme základný update a upgrade systému.

```
apt update && apt upgrade
```
Následne nainštalujeme ssh server a spustíme ho. 

```
apt install openssh-server
systemctl start ssh.service
```

Základný konfiguračný súbor je nano /etc/ssh/sshd_config.
V ňom vieme zmeniť základné parametre: 

* `AllowUsers student1 student2` - povolenie pripojenia len pod niektorými používateľmi
* `AllowGroups skupina1 skupina2` - povolenie pripojenia len konkrétnych skupín
* `Port 2222` - zmena defaultného portu z 22 na 2222
* `PubkeyAuthentication yes` - povolenie prihlasovať sa pomocou verejného kľúča
* `PasswordAuthentication no` - zakázanie prihlasovať sa pomocou hesla, je nutné mať nakopírovaný verejný kľúč v súbore ~/.ssh/authorized_keys


Následne sa vieme vzidalene prihlásiť na tento ssh server napríklad takto:  

`ssh -p 2222 student1@192.168.0.1`
