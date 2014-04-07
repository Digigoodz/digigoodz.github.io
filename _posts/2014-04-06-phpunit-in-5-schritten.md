---
layout: post
title: PHPUnit in 5 Schritten
tags: php phpunit tutorial
excerpt: "In diesem Tutorial zeige ich wie man von Null beginnt um ins Unit Testing einzusteigen.

Am Ende dieses Projektes steht zwar nur ein sehr simpler Unit Test, aber das wichtigste an diesem Tutorial ist der Weg dorthin."

---
In diesem Tutorial zeige ich wie man von Null beginnt um ins Unit Testing einzusteigen.

Am Ende dieses Projektes steht zwar nur ein sehr simpler Unit Test, aber das wichtigste an diesem Tutorial ist der Weg dorthin.


#### Schritt 1, composer
Im ersten Schritt werden wir <a href="http://getcomposer.org" target="_blank">composer</a> installieren und eine entsprechende Konfigurationsdatei `composer.json` erstellen.

Die Zusammenfassung von der Composer Webseite:
> Composer is a tool for dependency management in PHP. It allows you to declare the dependent libraries your project needs and it will install them in your project for you.

Ein großartiges Werkzeug, dass es uns noch einfacher macht PHPUnit (und eine Menge anderer Dinge) schnell und unkompliziert in das eigene Projekt einzubinden und nutzbar zu machen.

Wir legen uns als erstes ein leeres Verzeichnis an in das wir im Anschluss wechseln:

```
$ mkdir phpunit-tutorial
$ cd phpunit-tutorial
```

Danach installieren wir composer. Am einfachsten lässt sich composer so installieren: 

```
$ curl -sS https://getcomposer.org/installer | php
```


#### Schritt 2, composer Konfigurationsfile

Danach erstellen wir uns ein `composer.json` File um PHPUnit zu installieren.  
Ein absolut minimales File würde so aussehen:

```
{
    "require-dev": {
        "phpunit/phpunit": "3.7.*"
    }
}
```


<p class="personal-info">Eine Angabe eines Repositories ist nicht notwendig, da <a href="http://packagist.org">packagist</a> als Default-Repository unter der Haube verwendet wird.</p>

Damit hätten wir dann schon alles zusammen was wir brauchen um PHPUnit zu nutzen.

Unser `composer.json` File fällt detaillierter aus um folgendes zu erreichen:

* prüfen ob unsere PHP Version mindestens 5.3 oder größer ist (Namespaces)
* installiert PHPUnit (in unserer Dev-Umgebung)
* erzeugt uns ein `bin` Verzeichnis mit einem symbolischen Link auf den `phpunit` command-line runner
* erzeugt die composer Autoloader Konfiguration für den _Tutorial Namespace_

{% highlight json %}
{
  "name": "digigoodz/phpunit-quickstart",
  "description": "This is the companion project of the tutorial how to start using PHPUnit in a few simple steps.",
  "license": "MIT",
  "type": "project",
  "authors": [
      {
          "name": "Frank Lucht"
      }
    ],
  "require": {
      "php": ">=5.3"
  },
  "require-dev": {
      "phpunit/phpunit": "3.7.*"
  },
  "config": {
      "bin-dir": "bin/"
  },
  "autoload": {
      "psr-0": {
          "Tutorial\\": "src/"
      }
  }
}

{% endhighlight %}



#### Schritt 3, composer install
Um die Abhängigkeiten zu installieren wird composer jetzt ausgeführt:

```
$ php composer.phar install
```

Es dauert einen Augenblick alle Pakete herunterzuladen und wenn alles funktioniert hat sieht das Verzeichnis so aus:

```
$ ls -l
bin/
composer.json
composer.lock
composer.phar
vendor/
```

#### Zwischenstand
Wir haben jetzt die Grundlage geschaffen um gleich mit der Konfiguration für das Unit Testing zu beginnen.  
Wenn wir jetzt PHPUnit einmal ausführen sieht das Ergebnis so aus:

```
$ bin/phpunit --version
PHPUnit 3.7.32 by Sebastian Bergmann.
```

#### Schritt 4, PHPUnit Setup
Als erstes legen wir uns die Verzeichnisse an in denen wir unsere PHPUnit Konfiguration und unsere Unit Tests liegen werden.


```
$ mkdir -p test/Tutorial
```

In `test` werden wir die Konfigurationen ablegen und in `Tutorial` werden wir die Unit Tests ablegen.

Ein guter Startpunkt für die Konfiguration findet sich im Vendor Verzeichnis von PHPUnit: `vendor/phpunit/phpunit/phpunit.xml.dist`

Für unsere Zwecke erzeugen uns in unserem Testverzeichnis eine neue `phpunit.xml` Datei mit folgendem Inhalt:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:noNamespaceSchemaLocation="../vendor/phpunit/phpunit/phpunit.xsd"
  bootstrap="./bootstrap.php"
  backupGlobals="false"
  verbose="true">
  <testsuites>
    <testsuite name="Tutorial Testsuite">
      <directory suffix=".php">Tutorial</directory>
    </testsuite>
  </testsuites>
</phpunit>
{% endhighlight %}

Im `test` Verzeichnis legen wir jetzt noch eine `bootstrap.php` an:

{% highlight php %}
<?php
error_reporting(E_ALL | E_STRICT);

ini_set('display_startup_errors', 1);
ini_set('display_errors', 1);

include __DIR__ . '/../vendor/autoload.php';
{% endhighlight %}

Die `bootstrap.php` wird vor Ausführung der Unit Tests aufgerufen um ein paar PHP Settings für das Error Reporting vorzunehmen und um das Autoloading von composer einzubinden.

#### Schritt 5, Der Test
Wir sind jetzt soweit unsere zu testende Klasse und den dazugehörigen Unit Test zu schreiben.
Wir werden composer für das Autoloading nach [PSR-0](http://www.php-fig.org/psr/psr-0/) nutzen.

Dazu legen wir uns ein neues Verzeichnis an:

```
mkdir -p src/Tutorial
```

Als nächstes erzeugen wir in unserem neuen Verzeichnis die Datei `Tutorial.php` mit folgendem Inhalt:

{% highlight php %}
<?php
namespace Tutorial;

class Tutorial
{
    public function saluteYou()
    {
        return "Hello World!";
    }
}
{% endhighlight %}


Unser Testverzeichnis ist aus dem vorherigen Schritt vorbereitet.

Wir erstellen jetzt in dem vorhandenen Testverzeichnis `test/Tutorial` die Testdatei `TutorialTest.php` mit diesem Inhalt:

{% highlight php %}
<?php
namespace Tests;

use Tutorial\Tutorial;

class TutorialTest extends \PHPUnit_Framework_TestCase
{
    public function testTutorialClassShouldSaluteYou()
    {
        $expected = "Hello World!";

        $tutorial = new Tutorial();
        $this->assertEquals($expected, $tutorial->saluteYou());
    }
}
{% endhighlight %}

Wir führen jetzt PHPUnit aus um unsere Annahme, dass die Funktion `saluteYou` uns `Hello World` liefert, zu verifizieren:


```
$ bin/phpunit -c test/phpunit.xml                                                                                                                     
PHPUnit 3.7.32 by Sebastian Bergmann.

Configuration read from /phpunit-tutorial/test/phpunit.xml
.
Time: 24 ms, Memory: 3.25Mb

OK (1 test, 1 assertion)
```

#### Fazit
Es ist durchaus mit einem gewissen Aufwand verbunden wenn man, wie wir, bei Null anfängt. Aber dieser erste initiale Schritt ist unsere Grundlage für weitere Tests und nachfolgende Projekte.

Bei den nächsten Schritten hat man schon die notwendigen Dateien parat um sie wiederzuverwenden und den neuen Gegebenheiten anzupassen.

Wer sich die Tipparbeit sparen will, kann sich das vollständige Projekt von Github clonen: [https://github.com/Digigoodz/phpunit-quickstart](https://github.com/Digigoodz/phpunit-quickstart)

```
$ git clone https://github.com/Digigoodz/phpunit-quickstart .
```







