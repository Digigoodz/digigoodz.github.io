---
layout: post
title: TDD oder Der Weg ist das Ziel
tags: tdd php phpunit
excerpt: Es gibt viele Blogs über TDD und Unit Testing. Welche Vorteile es mit sich bringt, welche Nachteile es mit sich
         bringt, warum man es machen sollte, oder warum auch nicht.

         Die Entwickler(innen) die ihre Projekte per TDD entwickeln oder Unit Testing praktizieren scheinen immer noch eine Minderheit
         zu sein.
---
Es gibt viele Blogs über <span class="info" title="Test Driven Development">TDD<sup>?</sup></span> und Unit Testing. Welche Vorteile es mit sich bringt, welche Nachteile es mit sich
bringt, warum man es machen sollte, oder warum auch nicht.

Die Entwickler(innen) die ihre Projekte per TDD entwickeln oder Unit Testing praktizieren scheinen immer noch eine Minderheit
zu sein. So schien es mir wenigstens auf der <a href="http://phpconference.com">IPC, Berlin</a> im Juni 2013 oder bei der <a href="http://europe.zendcon.com/">ZendCon Europe, Paris</a> im November 2013.

<p class="personal-info">Woran kann das liegen?</p>

Eine einfache Frage auf die es keine einfache Antwort gibt. Zu vielschichtig sind die Umstände und Einflüsse die einem
Entwickler im täglichen Geschäft zu schaffen machen.

Ich werde in diesem Post meinen Weg von einem gestressten Entwickler zu einem entspannten und überzeugten TDD und Unit
Test Evangelisten aufzeigen.

#### Der Anfang
Ich arbeite seit über 8 Jahren in einem projektgetriebenen Umfeld. Zeit ist Geld, es ist wichtig zu liefern - und das schnell.
Am Anfang waren die Projekte eher klein und überschaubar. Überwiegend One-Man Shows um möglichst viele Aufträge erledigen zu können.

Unit Testing fand in diesem Umfeld nicht statt. Es ging auch ohne, und das eine ganze Zeit lang.

Aber irgendwann fängt es an. Ein schleichender Prozess dessen Auswirkungen sich aufsummiert und irgendwann nicht mehr
verwaltbar ist. Je mehr Projekte erledigt wurden und je umfangreicher die Projekte wurden, umso mehr Wartung, Pflege und
Erweiterungen kamen. Hier mal schnell was erweitert, dort mal was reingehackt - teilweise direkt am offenen Herzen.

Die Anzahl der Bälle die gleichzeitig in der Luft gehalten werden mußten wurde irgendwann absurd.

Die Code Basis wurde langsam aber sicher immer schlimmer. Die Tage wurden länger, die Zeit ein neues Feature zu implementieren
wurde länger - teilweise sogar unmöglich. Ein gefixter Bug hatte auch schon mal neue Fehler zur folge, weil auf dem Fehler Features
aufgesetzt wurden - schön ist was anderes.

Ein Faktor der aber noch schwerer wiegt, und den man erst im Rückblick und mit Abstand erkennt: Man verliert Reputation und Vertrauen.

#### Der Wendepunkt
Irgendwann kommt man an den Punkt, an dem man merkt dass es so nicht weitergehen kann. Man braucht eine Lösung, irgendetwas
das einem hilft dem Chaos Herr zu werden. Es gibt nicht die "Eine" allumfassende Lösung. Für mich und das Team mit
dem ich zusammenarbeite wollten wir eine Lösung finden die uns in Zukunft die Arbeit erleichtern sollte.

Unser Ansatz war: ___Unit Testing und (nach Möglichkeit) Test Driven Development___ - bei allen neuen Projekten.

Wir haben schon längere Zeit mit dieser Option geliebäugelt, aber es gab einige Hindernisse und Probleme die dann doch
immer dafür gesorgt haben es sein zu lassen. Was das war, und wie es gelöst wurde siehst du etwas weiter unten.

#### Wie fängt man an?
Aller Anfang ist bekanntlich schwer. Vor allem bei einem Werkzeug von dem man nicht so richtig weiß wie man am Besten anfängt.
Das Internet ist voller guter und schlechter Beispiele, es gibt Bücher und Leute die sich damit auskennen.

<blockquote cite="Erich Kästner"><p>Es gibt nichts gutes, außer man tut es.</p><cite>Erich Kästner</cite></blockquote>

Wir haben ein Projekt ausgewählt das einen überschaubaren Aufwand hatte, und das für einen Entwickler ausgelegt war.
Der Aufwand bewegte sich in der initialen Schätzung - wenn alles glatt läuft, und die Anforderungen so bleiben - bei 22 Personentagen.
Das Ziel war eine Schnittstelle für eine iOS-, Android- und Web-App bereitzustellen. Die Schnittstelle dient als mittler
zwischen diversen anderen Schnittstellen, und bot somit eine mehr als ausreichende Komplexität.

Was es galt festzustellen:

1. Können wir Unit Testing in adäquater Zeit lernen und umsetzen?
2. Können wir solidere Software bauen, Features einfacher implementieren?
3. Werden wir langsamer?

Hatten wir einen Plan B? Nicht wirklich.

#### Der Weg
<blockquote cite="Johann Wolfgang von Goethe"><p>Es ist nicht genug, zu wissen, man muß auch anwenden; es ist nicht genug, zu wollen, man muß auch tun.</p>
<cite>Johann Wolfgang von Goethe</cite></blockquote>

Alleine die Erwähnung von Unit Tests kann bei dem Einen oder Anderen schon ausreichen in Verteidigungsposition zu gehen.
Ein paar der Aussagen - ob nun Management/Business oder Entwickler - auf die man trifft:

1. Dafür ist keine Zeit
2. Dafür werden wir nicht bezahlt
3. Das ist zu aufwändig
4. Dafür haben wir ne QA
5. Für meine Sachen brauch ich das nicht

Die Fragen und die Aussagen sind durchaus gerechtfertigt. Wie geht man damit um?

Man kann versuchen sich den Mund fusselig zu reden, Studien zu präsentieren, auf die Geschichte in der Softwareentwicklung
verweisen. Das kann funktionieren - je nachdem wer einem gegenüber sitzt - muß aber nicht.

Und nun?

<p class="personal-info">Nicht drüber sprechen, einfach machen!</p>

Da es das erste Projekt war das so entwickelt wurde blieben Probleme nicht aus. Der Anfang war etwas holperig, es war hier
und da schwierig die Tests zu schreiben und Testdaten zu erzeugen.

Die Tests sind nicht aller erste Sahne, aber sie erfüllen ihren Zweck.

#### Das Ergebnis
Das Resultat war für uns wegweisend - es ist jetzt knapp 2 Jahre her.

Welches Ergebnis haben wir mit dem "Test-Projekt" erzielt?

1. __Können wir Unit Testing in adäquater Zeit lernen und umsetzen?__  
   Klares Ja. Wir benutzen <a href="https://github.com/sebastianbergmann/phpunit/">PHPUnit</a>,
   und bleiben damit in der gewohnten Sprache.
   Tests schreiben wurde mit jedem Test, und jedem Tag ein wenig besser.

2. __Können wir solidere Software bauen, Features einfacher implementieren?__  
   Klares Ja. Durch die Testbarkeit war die Kopplung des Codes nicht so starr wie wir es vorher hatten. Mit den Tests wurde
   es deutlich einfacher und sicherer neue Features einzubauen.

3. __Werden wir langsamer?__  
   Der Anfang war langsamer. Die Geschwindigkeit am Ende des Projektes war ungewöhnlich hoch. Neue Features einzubauen machte keine Probleme und sorgte
   nicht für Frust, sondern machte wieder Spaß.

   Wie bei allen Projekten änderten sich die Anforderungen und der Umfang. Das stellte diesmal für uns kein Problem dar. Wir waren
   auch zeitlich im Rahmen - wir waren nicht langsamer.

Aber eine weitere wichtige Frage die nicht gestellt wurde: __Kann man die Defektrate senken?__

Dazu ein paar Zahlen:

- ~3.2 kLOC Business Code (mitlerweile ~5.3kLOC)
- ~3.2 kLOC Test Code (mitlerweile ~4.8kLOC)
- __Bugs__ die in der Entwicklung aufgetreten sind: __6__
- __Minor Bugs__ im Betrieb während der ersten 18 Monate: __&lt;5__
- __Major Bugs__ im Betrieb während der ersten 18 Monate: __0__
- __Neue Features__ während der ersten 18 Monate: __80+__

Das ist ein mehr als hervorragendes Ergebnis für das allererste Projekt das wir so entwickelt haben.

#### Fazit
<blockquote cite="Kent Beck"><p>I'm not a great programmer; I'm just a good programmer with great habits.</p><cite>Kent Beck</cite></blockquote>

Auch wenn die Tests im Rückblick noch nicht die Besten waren, waren sie so gut ein Projekt dieser Qualität auf die Beine zu stellen.

Auf Basis dieses _"Experiments"_ wurden alle weiteren neuen Projekte nur noch mit Unit Tests implementiert. Die hohe Qualität
der Software hat nicht nur Einfluß auf den Stresslevel. Wartung, Betrieb und Erweiterbarkeit der Software gestaltet sich
wesentlich einfacher und sicherer.

Eine niedrige Defektrate bedeutet gleichzeitig auch einen monetären Gewinn. Weniger Fehler, die zusätzlich noch leichter
zu beheben sind bedeuten mehr Zeit für neue Dinge. Die Abwärtsspirale in der wir uns befunden haben hat sich umgekehrt.

Würde ich Unit Testing und TDD weiterempfehlen? __Unbedingt__.

Wartet nicht drauf dass euch das Management oder der Vorgesetzte grünes Licht gibt - handelt selbst. Die Veränderung muß von uns kommen.



<br />
<br />

<h4 class="book-recommendation">Bücherempfehlung</h4>
<div class="books">
<a href="http://www.amazon.de/gp/product/0321146530/ref=as_li_tf_il?ie=UTF8&camp=1638&creative=6742&creativeASIN=0321146530&linkCode=as2&tag=digigoodz-21"><img border="0" src="http://ws-eu.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=0321146530&Format=_SL110_&ID=AsinImage&MarketPlace=DE&ServiceVersion=20070822&WS=1&tag=digigoodz-21" ></a><img src="http://ir-de.amazon-adsystem.com/e/ir?t=digigoodz-21&l=as2&o=3&a=0321146530" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
<br />
<a href="http://www.amazon.de/gp/product/0321146530/ref=as_li_tf_tl?ie=UTF8&camp=1638&creative=6742&creativeASIN=0321146530&linkCode=as2&tag=digigoodz-21">Test Driven Development. By Example (Addison-Wesley Signature)</a><img src="http://ir-de.amazon-adsystem.com/e/ir?t=digigoodz-21&l=as2&o=3&a=0321146530" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
<br />
<br />
<a href="http://www.amazon.de/gp/product/0201485672/ref=as_li_tf_il?ie=UTF8&camp=1638&creative=6742&creativeASIN=0201485672&linkCode=as2&tag=digigoodz-21"><img border="0" src="http://ws-eu.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=0201485672&Format=_SL110_&ID=AsinImage&MarketPlace=DE&ServiceVersion=20070822&WS=1&tag=digigoodz-21" ></a><img src="http://ir-de.amazon-adsystem.com/e/ir?t=digigoodz-21&l=as2&o=3&a=0201485672" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
<br />
<a href="http://www.amazon.de/gp/product/0201485672/ref=as_li_tf_tl?ie=UTF8&camp=1638&creative=6742&creativeASIN=0201485672&linkCode=as2&tag=digigoodz-21">Refactoring: Improving the Design of Existing Code (Object Technology Series)</a><img src="http://ir-de.amazon-adsystem.com/e/ir?t=digigoodz-21&l=as2&o=3&a=0201485672" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
</div>
<br />
<br />
##### Studien
<div class="books">
<a href="http://www.infoq.com/news/2009/03/TDD-Improves-Quality">Empirical Studies Show Test Driven Development Improves Quality</a>
</div>

<br />