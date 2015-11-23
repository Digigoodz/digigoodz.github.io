---
layout: post
title: Wie man das nginx resolver Problem lokaler Domains mit dnsmasq löst
tags: dns dnsmasq nginx
excerpt: "Neulich hatte ich mit einem Projekt zu tun in dem das _Frontend_ über __proxy_pass__ in der [nginx](http://nginx.org/) Konfiguration das
          _Backend_ gekapselt hat.
          Das ist keine Besonderheit, aber eine interessante Aufgabenstellung wenn man das ganze lokal auf seinem Mac nachstellen will."
meta:
  description: Was macht man wenn man mit nginx und proxy_pass lokale Domains benutzen will oer muss? dnsmasq ist die Lösung. Wie dnsmasq dafür genutzt werden kann zeigt der Post.
---

Neulich hatte ich mit einem Projekt zu tun in dem das _Frontend_ über __proxy_pass__ in der [nginx](http://nginx.org/) Konfiguration das 
_Backend_ gekapselt hat.
Das ist keine Besonderheit, aber eine interessante Aufgabenstellung wenn man das ganze lokal auf seinem Mac nachstellen will.  
Jeder der seine Webprojekte zuallererst lokal entwickelt weiß dass er für seine virtuellen Hosts einen Eintrag in der `/etc/hosts` einfügen muss.
Das ist schnell gemacht und funktioniert problemlos. Für die lokale Entwicklung ist das zu einfach gedacht und ohne richtige Namensauflösung nicht möglich.
Wie wir das Problem mit [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) lösen erkläre ich in diesem Post.


## Die Standardkonfiguration

### /etc/hosts

Als erstes schauen wir uns unsere Beispielkonfiguration in der `/etc/hosts` Datei an:

```
127.0.0.1 web.frontend.dev
127.0.0.1 web.backend.dev
```

Damit ist der erste Schritt getan zur Auflösung der Domainnamen.


### Webserver Konfiguration

Jetzt kommen wir zu dem Teil wo das Frontend alle Aufrufe nach außen kapselt und die Backendaufrufe an das Backend proxied.
Unsere Frontend Konfiguration sieht dann so aus __frontend.conf__:

```
server {
    listen *:80;
    server_name "web.frontend.dev";
    
    resolver 127.0.0.1;
    
    root "/data/web.frontend.dev/public";
    index index.html;
    
    location / {
        try_files $uri $uri/
    }
    
    location ~ ^/api/(.*)$ {
        add_header 'Allow' 'GET, DELETE, HEAD, PUT, POST';
        proxy_pass http://web.backend.dev/$1;
    }
}
```

Die Backend Konfiguration __backend.conf__:

```
server {
    listen *:80;
    server_name "web.backend.dev";
    
    root "/data/web.backend.dev/public";
    index index.php;
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param APPLICATION_NAME myapp;
        fastcgi_param APPLICATION_ENV development;
        fastcgi_read_timeout 90;
    }
}
```

Das Problem an der Stelle ist, dass es so nicht funktioniert. Nginx kann die Namen aus der /etc/hosts Datei nicht auflösen.

In __frontend.conf__ haben wir mit `resolver 127.0.0.1;` einen Nameserver definiert den wir nicht haben. Wünschenswert wäre
an der Stelle dass man mit `resolver /etc/hosts` direkt die Datei auswählen kann, aber die Funktionalität ist nicht vorhanden
und das ist auch kein Problem.  
Wir werden __dnsmasq__ als Hilfsmittel verwenden.


## dnsmasq mit /etc/hosts
_dnsmasq_ ist ein sehr leichtgewichtiger DNS- und DHCP-Server der alle Namensanfragen entgegen nimmt, versucht die Auflösung der Namen anhand
der `/etc/hosts` Datei auszuführen, und andernfalls unbekannte Anfragen weiterleitet und das Ergebnis cached.
Dieser Ansatz bietet einen kleinen Nachteil, aber dazu später mehr.

### Installation
Für die Installation benutzen wir [homebrew](http://brew.sh/)

```
$ brew install dnsmasq
...
To configure dnsmasq, copy the example configuration to /usr/local/etc/dnsmasq.conf
and edit to taste.

  cp /usr/local/opt/dnsmasq/dnsmasq.conf.example /usr/local/etc/dnsmasq.conf

To have launchd start dnsmasq at startup:
  sudo cp -fv /usr/local/opt/dnsmasq/*.plist /Library/LaunchDaemons
  sudo chown root /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
Then to load dnsmasq now:
  sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
```

Folgt man den Anweisungen ist man auch schon fast fertig. Die Standard Konfiguration ist im ersten Schritt ausreichend, sprich wir müssen nichts anpassen.

Das einzige was noch fehlt ist die Anpassung der `/etc/resolv.conf`. Hier muss die IP des neuen Nameservers an erster Stelle eingetragen werden - in unserem Fall ist das die 127.0.0.1.
Das machen wir nicht direkt - da bei jedem Bootvorgang diese Datei neu geschrieben wird, sondern über 
_Systemsteuerung -> Netzwerk -> Erweitert_ und sollte dann so aussehen:

![Netzwerksetting]({{ site.url }}/assets/img/network-setting-advanced-dns.png)

Damit ist _dnsmasq_ an erster Stelle für die Namensauflösung und an der zweiten Stelle der DHCP Server meines Netzwerks, bzw. hier steht dann der deines Netzwerks.

### Validierung
Um jetzt zu testen ob auch alles so funktioniert wie wir uns das vorstellen benutzen wir __dig__, ein Kommandozeilen Tool 
um Informationen eines DNS Servers abzurufen.

Aus der manpage von dig:
> dig (domain information groper) is a flexible tool for interrogating DNS name servers.

```
$ dig web.frontend.dev
; <<>> DiG 9.8.3-P1 <<>> web.frontend.dev
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18609
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;web.frontend.dev.		IN	A

;; ANSWER SECTION:
web.frontend.dev.	0	IN	A	127.0.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Nov 18 14:00:50 2015
;; MSG SIZE  rcvd: 50
```

Dadurch sind wir in der Lage unser initiales Problem zu lösen. Wir haben unter 127.0.0.1 unseren eigenen DNS Server laufen der in der Lage ist unsere lokalen
Domains aufzulösen und einen beispielhaften Aufruf `http://web.frontend.dev/api/info` an unser Backend weiterzureichen.

### Nachteil
Ein Nachteil darf hier auch nicht verschwiegen werden. Für _jede neue Domain_ die wir einfügen müssen wir den _dnsmaq Service neu starten_, weil eine Änderung in
der `/etc/hosts` leider nicht automatisch erkannt wird.

## dnsmasq mit default Adresse
Um den Nachteil zu umgehen bei jeder neuen Domain die wir für unsere Entwicklung benutzen auch dnsmasq neu zu starten schauen wir in das Konfigurationsfile `/usr/local/etc/dnsmasq.conf`.
Dort finden wir diese Sektion:

```
# Add domains which you want to force to an IP address here.
# The example below send any host in double-click.net to a local
# web-server.
#address=/double-click.net/127.0.0.1
```

Wenn wir an dieser Stelle diese Anpassung machen

```
# Add domains which you want to force to an IP address here.
# The example below send any host in double-click.net to a local
# web-server.
#address=/double-click.net/127.0.0.1
address=/.dev/127.0.0.1
```
dann wird __dnsmasq__ _jede Adresse_ die auf _.dev_ endet mit _127.0.0.1_ beantworten.
Dafür muss __dnsmasq__ einmal neu gestartet werden damit die Konfigurationsanpassung übernommen wird:

```
$ sudo launchctl unload /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist 
$ sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
```

Die Validierung wieder mit _dig_ und einem beliebigen Domainnamen mit einer _.dev_ TLD (Top Level Domain):

```
$ dig dummy.dev
; <<>> DiG 9.8.3-P1 <<>> dummy.dev
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38797
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;dummy.dev.			IN	A

;; ANSWER SECTION:
dummy.dev.		0	IN	A	127.0.0.1

;; Query time: 1 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Nov 18 14:30:24 2015
;; MSG SIZE  rcvd: 43
```

Damit ist es jetzt nicht mehr notwendig für die .dev Domänen eine Anpassung der /etc/hosts Datei vorzunehmen.

## Fazit
Wenn wir lokal entwickeln ist es im Regelfall ausreichend wenn wir unsere lokalen Domänen in die `/etc/hosts` Datei eintragen.

__dnsmasq__ kann uns in Fällen in denen wir eine wirkliche Namensauflösung benötigen sehr einfach helfen. Es ist leicht zu installieren 
und im einfachsten Fall entfällt eine aufwändige Konfiguration.

Wir sind in der Lage lokal Serverbeziehungen so abzubilden wie wir es auch in einem Produktiv-Umfeld machen würden, und erhalten zusätzlich noch
einen DNS Cache für Namensauflösungen. Und wir können auch all zu lästige Ads damit wegfiltern wenn wir wollen.