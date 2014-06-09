---
layout: post
title: Testbaren Code schreiben
tags: testing php phpunit
---
In einem früheren Artikel [TDD oder der Weg ist das Ziel]({% post_url 2014-03-23-tdd-oder-der-weg-ist-das-ziel %}) habe ich die Frage gestellt, woran es liegen könnte dass augenscheinlich so wenig Unit Testing gemacht wird.

Ich habe noch mal selber darüber nachgedacht wie es bei mir war.
Da gab es einige Punkte, aber der wichtigste:

<p class="personal-info">Ich wußte einfach nicht wirklich wie ich starten soll (mit Legacy-Code), und wie ich Code schreibe der (automatisiert) testbar ist.</p>

Die wohl am häufigsten getroffene Aussage dazu ist, man soll Tests für den neuen Code schreiben, und da wo es sinnvoll ist auch für den alten Code.

Die Kernaussage ist sicherlich korrekt. Das Problem daran ist aber, dass sie nicht auf das _"wie"_ eingeht. Und wenn man mit Unit Testing anfangen möchte ist diese Aussage nicht hilfreich.

Von daher nehmen wir jetzt den Weg eines Anwendungsbeispiels um ein Ergebnis zu erzielen: _Testbarer Code_.



## Das Legacy Code Problem
Wenn man in einem projektgetriebenen Umfeld arbeitet, dann ist das A&O in Budget und Time zu sein. Es kommt eher weniger auf die elegante Lösung im Code an, als vielmehr darauf was am Ende nach außen sichtbar ist. 
Das ist eine kurzsichtige - wenn auch weit verbreitete - Denkweise, die früher oder später zu Problemen führen wird. Das ist aber ein Thema für einen anderen Post.

Nehmen wir ein einfaches Beispiel:

{% highlight php linenos %}
<?php
require 'thirdpartyapi.php';
require 'logger.php';
require 'db.php';

class SomeClass
{
    private $api;
    private $logger;
    
    public function __construct()
    {
        $this->api = new ThirdPartyApi();
        $this->logger = new Logger();
    }
    
    public function getCustomerStatus($customerId)
    {
       $db = new DatabaseLayer();
       
       $status = $this->api->getCustomerStatus($customerId);
       $db->saveCustomerStatus($customerId, $status);
       $this->logger->log("Customer Status updated");
       
       return $status;
    }
}
{% endhighlight %}

Anhand des Beispiels kann man sagen: Auf Unit Test Ebene ist das nicht testbar.

Was sind die Probleme?

1. Objekte werden innerhalb des Konstruktors erzeugt
2. Ein Objekt wird innerhalb einer Funktion erzeugt
3. Die Funktion `getCustomerStatus` macht mehr als der Methodenname aussagt
4. Wir haben eine Abhängigkeit zu irgendeiner anderen Schnittstelle
5. Wir haben eine Abhängigkeit zu einer Datenbank (schreibend)
6. Wir haben eine Abhängigkeit zu einem Logger

So wenig Code, und so viele Probleme - wenn man die Klasse testen möchte.


## Umdenken, in kleinen Schritten
Wir werden ein paar kleine, sehr einfache und überschaubare Schritte vornehmen:

1. Identifizieren der lokalen Instanziierung
2. Entkoppeln der unverzichtbaren Abhängigkeiten durch Konstruktor-Injizierung, internes Refactoring
3. Refactoring der Verwendung von `SomeClass`
4. Manueller Test (wenn wir ein echtes Projekt haben, hier ist es überschaubar)


### Schritt 1, Abhängigkeiten identifizieren
Wir schauen uns an wo überall ein `new` im Code verwendet wird. Das sind die Stellen die uns hier die (Test-)Probleme bereiten:

1. Der Konstruktor (Zeile 13 u. 14)
2. die Funktion `getCustomerStatus` (Zeile 19)

### Schritt 2, Konstruktor Injizierung
Wir übergeben all Objekte die innerhalb der Klasse erzeugt werden und zwingend notwenig sind als Parameter im Konstruktor:

{% highlight php %}
<?php
// …
class SomeClass
{
    private $api;
    private $logger;
    private $db; // neu
    
    public function __construct($api, $logger, $db)
    {
        $this->api = $api;
        $this->logger = $logger;
        $this->db  = $db;
    }
    
    public function getCustomerStatus($customerId)
    {
       // entfällt: $db = new DatabaseLayer();       
       $status = $this->api->getCustomerStatus($customerId);
       $this->db->saveCustomerStatus($customerId, $status);
       $this->logger->log("Customer Status updated");
       
       return $status;
    }
}   
{% endhighlight %}

__Ergebnis__: Wir haben jetzt mit zwei sehr kleinen Anpassungen dafür gesorgt das die vorher nicht testbare Klasse jetzt (problemlos) testbar ist.

Warum das so ist sehen wir am Ende des Posts.

### Schritt 3, Refactoring bei der Verwendung von SomeClass
Dadurch, dass wir jetzt die Erzeugung aller Abhängigkeiten nicht mehr innerhalb der Klasse `SomeClass` haben, müssen wir jetzt schauen wo `SomeClass` verwendet wird, und dort entsprechende Anpassungen vornehmen.

Z.B. könnte irgendwo in unserem fiktiven Projekt so etwas stehen:

{% highlight php %}
<?php
    // … input validation, etc.
    
    $someClass = new SomeClass();
    
    $status = $someClass->getCustomerStatus($customerId);
    
    // … 
{% endhighlight %}

Wir müssen in diesem Fall diese Anpassung vornehmen:

{% highlight php %}
<?php
    // … input validation, etc.
    
    $api = new ThirdPartyApi();
    $logger = new Logger();
    $db = new DatabaseLayer();
    
    $someClass = new SomeClass($api, $logger, $db);
    
    $status = $someClass->getCustomerStatus($customerId);
    
    // … 
{% endhighlight %}

Auf den ersten Blick sieht das komplizierter und umständlicher aus, lässt sich aber durch die Verwendung einer Factory die uns `SomeClass` mit allen Abhängigkeiten an einer zentralen Stelle erzeugt wieder vereinfachen.

#### Schritt 4, Manueller Test
Jetzt würde der Punkt kommen, an dem man _noch_ manuell testet, ob die Applikation noch so funktioniert wie vor der Anpassung.
Wir haben bislang nur einen ersten kleinen, aber sehr wichtigen Schritt getan in Richtung testbarer Code.



## Fazit
Wir haben noch nicht alle Probleme gelöst, aber wir haben dafür gesorgt, dass wir jetzt die Kontrolle haben, und unsere umgebaute Klasse jetzt testen können.

Was wir erreicht haben:

1. <s>Objekte werden innerhalb des Konstruktors erzeugt</s>
2. <s>Ein Objekt wird innerhalb einer Funktion erzeugt</s>
3. Die Funktion `getCustomerStatus` macht mehr als der Methodenname aussagt
4. <s>Wir haben eine Abhängigkeit zu irgendeiner anderen Schnittstelle</s>
5. <s>Wir haben eine Abhängigkeit zu einer Datenbank (schreibend)</s>
6. <s>Wir haben eine Abhängigkeit zu einem Logger</s>

Die Punkte 1 und 2 sind offensichtlich. Die Punkte __4-6__ vielleicht nicht.
Die __Abhängigkeit__ an sich ist immer noch da, __aber wir kontrollieren sie__.
Punkt 3 vernachlässigen wir in diesem Beispiel.


Wir sind jetzt in der Lage der Klasse `SomeClass` die Objekte für die Api, Logger und Datenbank zu übergeben, und selbst zu definieren wie sich diese Objekte in einem Testfall verhalten sollen.

Wie das zu bewerkstelligen ist wird Inhalt eines neuen Posts.



