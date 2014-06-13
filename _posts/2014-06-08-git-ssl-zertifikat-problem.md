---
layout: post
title: git SSL Zertifkats Problem
tags: quicktip git mac
excerpt: "Vor kurzem hatte ich mit einem nicht unbekanntem Problem beim clonen von git Projekten zu kämpfen: Das SSL Zertifikat von https://github.com konnte nicht verifiziert werden."
lastmod: 13-06-2014
meta:
  description: Was machen wenn das git clone wegen eine SSL Zertifikatsproblems nicht mehr funktioniert? Zwei Varianten die das Problem lösen.
---
Vor kurzem hatte ich mit einem nicht unbekanntem Problem beim clonen von git Projekten zu kämpfen: Das SSL Zertifikat von https://github.com konnte nicht verifiziert werden. 

Bei mir möglicherweise hervorgerufen durch ein Update meiner git Version - so genau kann ich das nicht sagen.

Eine Suche im Netz brachte verschiedenste Lösungen mit sich. Die Lösung die für mich funktioniert hat und die ich persönlich für die Beste halte ist wie folgt:

## Variante 1
1. Prüfen ob die Datei `/usr/local/etc/gitconfig` existiert
2. Prüfen ob der Eintrag für `sslCAInfo` gesetzt ist
3. Wenn nicht, dann den Eintrag setzen auf

    ```
    [http]
    sslCAInfo=/usr/local/etc/openssl/cert.pem
    ```
4. Prüfen ob das git SSL Problem weiterhin besteht. Wenn ja, dann weiter zu Variante 2.


## Variante 2

1. Download der Zertifikate von `http://curl.haxx.se/ca/cacert.pem` (<a href="http://curl.haxx.se/">http://curl.haxx.se/</a>)
2. Wahlweise ersetzt man die alten unvollständigen Zerifikate unter `/usr/local/etc/openssl/cert.pem` oder kopiert sich die Zertifikate in ein gesondertes Verzeichnis, z.B.  `/usr/local/gitcerts/cacert.pem`
3. Wenn noch nicht vorhanden, erzeugen einer Systemweiten gitconfig: `touch /usr/local/etc/gitconfig`
4. Mit dem bevorzugten Editor folgendes hinzufügen (je nach Ort der Zertifikate):

    ```
    [http]
    sslCAInfo=/usr/local/gitcerts/cacert.pem
    ```
    
Damit sollte das SSL Verifizierungsproblem behoben sein.
