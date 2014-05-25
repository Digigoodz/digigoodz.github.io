---
layout: post
title: Testbaren Code schreiben (Teil 2)
tags: testing php phpunit

---

Im Post [Testbaren Code schreiben]({% post_url 2014-04-21-testbaren-code-schreiben%}) haben wir in ein paar kleinen Schritten einen nicht testbaren Code so modifiziert, dass wir ihn testen können. 

Als Basis benutzen wir das Projekt aus [PHPUnit in 5 Schritten]({% post_url 2014-04-6-phpunit-in-5-schritten %}). Der vollständige Quellcode zu diesem Tutorial findet sich auf [github: PhpUnit LegacyCode Tutorial](https://github.com/Digigoodz/phpunit-legacycode-tutorial)
<!-- ex -->
In das Projekt fügen wir unseren Fake Legacy Code ein den wir als Basis benutzen und Schritt für Schritt, und Test für Test verbessern werden.

## Dependency Injection und Inversion of Control
Um unseren Code testbar zu machen werden wir mit _Dependency Injection_ (DI) arbeiten die dem Prinzip des _Inversion of Control_ (IoC) folgt. Wir injizieren die Abhängigkeiten externer Komponenten die für die korrekte Funktion unserer Klasse notwendig sind über den Konstruktor.

Durch die Injizierung der Abhängigkeiten sorgen wir für eine Entkopplung der Abhängigkeiten, und sind somit in der Lage die einzelnen Komponenten selbst zu kontrollieren.

## Schrittweises Refactoring
Im vorigen Post [Testbaren Code schreiben]({% post_url 2014-04-21-testbaren-code-schreiben%}) habe ich ein paar Abkürzungen genommen und ein paar Kleinigkeiten weggelassen die wir jetzt implementieren werden.

## Test Doubles und Mocks
Wir wollen für unsere Tests nicht die echten Objekte verwenden von der `SomeClass`abhängig ist, deshalb werden wir mit Objekten arbeiten die so tun als ob: Das sind dann unsere _Test Doubles_ oder _Mocks_.

Es gibt unterschiedliche Methoden und Präferenzen um an das Ziel zu kommen. Wir werden aber [Mockery](https://github.com/padraic/mockery) verwenden um unsere Implementierung zu testen.

## Legacy Code
In der Verzeichnisstruktur findet sich unser Legacy Code in 
```
src/Tutorial/LegacyCode
```

Die Testimplementierung findet sich unter
```
src/Project
```

Die Testimplementierung wird im schrittweisen Umbau des Codes ebenfalls verändert und an die neuen Gegebenheiten angepasst.
Die Ausführung der Testimplementierung ergibt folgendes Ergebnis:

```
$ php php src/Project/index.php

Status saved for Customer: 1 with status: active
Customer Status updated
active
```


## Schritt 1
Im ersten Schritt werden wir `new DatabaseLayer();` aus der Funktion `getCustomerStatus` in den Konstruktor verschieben und eine neue Instanzvariable für den DatabaseLayer.

An der Funktionsweise der Klasse hat sich nichts geändert, und alles funktioniert noch wie vor der Anpassung.

## Schritt 2: Konstruktor Injizierung
Wir werden im zweiten Schritt dafür sorgen, dass alle Abhängigkeiten über den Konstruktor injiziert werden, um dann folgendes Ergebnis zu erhalten:

{% highlight php linenos %}
<?php
public function __construct(
    ThirdPartyApi $api, Logger $logger, DatabaseLayer $db) {
    $this->api = $api;
    $this->logger = $logger;
    $this->db = $db;
}
{% endhighlight %}

Wir benutzen hier das Type-Hinting von PHP um sicherzustellen, dass auch die richtigen Objekte an dieser Stelle übergeben werden. 
Wir verzichten erst einmal bewusst auf Type-Hintng mit Interfaces, da wir uns noch in einer frühen Phase befinden und unsere aktuelle Aufgabe das nicht vorsieht.

Wir werden einige Tests schreiben die im späteren Verlauf überflüssig werden und wieder entfernt werden, und die man mit mehr Erfahrung auch nicht zwingend benötigt.

Wir wollen jetzt testen ob unser Konstruktor korrekt funktioniert.


Der erste unvollständige Test (ohne dass die Klasse `SomeClass` modifiziert wurde) sieht so aus:

{% highlight php linenos %}
<?php
    public function testConstructorInitialization()
    {
        $db = \Mockery::mock('DatabaseLayer');
        $logger = \Mockery::mock('Logger');
        $api = \Mockery::mock('ThirdPartyApi');

        $someClass = new \SomeClass($db, $logger, $api);
        // ?  
    }
{% endhighlight %}
 
Wir führen jetzt den Test aus:

```
$ bin/phpunit -c tests/phpunit.xml --strict
…
There was 1 risky test:

1) Tutorial\LegacyCodeTest\SomeClassTest::testConstructorInitialization
This test did not perform any assertions

OK, but incomplete, skipped, or risky tests!
Tests: 1, Assertions: 0, Risky: 1.
```

Uns fehlt an dieser Stelle die Möglichkeit zu verifizieren, ob unsere Implementierung dem Test genügt.
Wie weiter oben schon geschrieben werden wir einige Tests schreiben die später wieder entfernt werden können. Dasselbe gilt auch für ein paar Erweiterungen der Klasse `SomeClass`. Aber hier kann man sehr gut die Parallele zu Hardwaretests ziehen, wo man Messpunkte über temporär angebrachte Pins aus eienr Hardwarekomponente führt.

In unserem Fall werden wir `SomeClass` um die jeweiligen getter-Methoden erweitern.

Wir erwarten, dass wir dieselben Objekte von den gettern bekommen, die wir injiziert haben:
{% highlight php linenos %}
<?php
    public function testConstructorInitialization()
    {
        $db = \Mockery::mock('DatabaseLayer');
        $logger = \Mockery::mock('Logger');
        $api = \Mockery::mock('ThirdPartyApi');

        $someClass = new \SomeClass($db, $logger, $api);
        $this->assertEquals($db, $someClass->getDb());
        $this->assertEquals($logger, $someClass->getLogger());
        $this->assertEquals($api, $someClass->getApi());  
    }
{% endhighlight %}

Der Test schlägt schon bei der ersten Assertion fehl, weil weil das Db Objekt nicht existiert. Die anderen Assertions werden fehlschlagen weil die Objekte nicht identisch sind.
Damit der Test sauber durchläuft passen wir den Konstruktor entsprechend an:

{% highlight php linenos %}
<?php    
    public function __construct(
    DatabaseLayer $db, Logger $logger, ThirdPartyApi $api) {
        $this->db = $db;
        $this->api = $api;
        $this->logger = $logger;
    }
{% endhighlight %}

Durch diesen einfachen Schritt haben wir schon eine Menge erreicht.
Die Abhängigkeiten werden nun im Konstruktor injiziert, und wir können die Funktion `getCustomerStatus` fast schon testen.

### Legacy Code anpassen
Wir müssen unsere Testimplementierung so anpassen, dass wir die Objekte jetzt vor der Instanziierung von `SomeClass` erzeugen und dann im Konstruktor mitgeben.

## Schritt 3: getGustomerStatus Test
Durch die vorbereitenden Schritte können wir `getCustomerStatus` jetzt testen. Wir werden dafür die Abhängigkeiten _mocken_ und schreiben unseren Test der so aussieht:

{% highlight php linenos %}
<?php
    public function testGetCustomerStatusReturnsActive()
    {
        $db = \Mockery::mock('DatabaseLayer');
        $db->shouldReceive('saveCustomerStatus')
            ->once()
            ->with(1, "active");

        $logger = \Mockery::mock('Logger');
        $logger->shouldReceive('log')
            ->with("Customer Status updated");

        $api = \Mockery::mock('ThirdPartyApi');
        $api->shouldReceive('getCustomerStatus')
            ->once()
            ->with(1)
            ->andReturn("active");


        $someClass = new \SomeClass($db, $logger, $api);
        $status = $someClass->getCustomerStatus(1);
        $this->assertEquals("active", $status);
    }
{% endhighlight %}

Wir machen in diesem Test eine Menge. Vor allem validieren wir an dieser Stelle auch sehr viel internes Verhalten über die Mock-Objekte die wir erzeugen.
Das ist ein kontroverses Thema, aber für unser Beispiel wird es benötigt und ich tendiere sehr häufig dazu die meisten Tests mit Mocks zu verwenden.

### Mock Verhalten
Wir erwarten, dass unsere Mocks in bestimmter Art und Weise aufgerufen werden. Dass bestimmte Funktionen mit bestimmten Parametern aufgerufen werden, und bestimmte Ergebnisse geliefert werden.
Diese Mocks werden dann in die zu testende Klasse injiziert und verrichten dort dann anstelle der richtigen Objekte ihre Arbeit.

Wenn wir unseren Test jetzt ausführen erhalten wir folgendes Ergebnis:

```
$ bin/phpunit -c tests/phpunit.xml --strict
…
..

Time: 41 ms, Memory: 4.50Mb

OK (2 tests, 4 assertions)
```

## Fazit
Wir haben 3 Schritte benötigt um unseren Legacy Code testbar(er) zu machen. Die Abhängigkeiten können durch Mocks ersetzt werden und wir haben eine Funktion durch einen ersten Unit Test (nicht vollständig) abgedeckt.

Damit sollte die prinzipielle Vorgehensweise zur schrittweisen Verbesserung von nicht-testbarem Code transparenter geworden sein.

Wichtig ist es immer kleine Schritte zu machen, und nicht zu viel auf einmal ändern zu wollen. Mit mehr Erfahrung lassen sich immer ein paar Dinge zusammenfassen, aber übersichtlicher bleibt es in kleinen Schritten.

