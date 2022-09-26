---
title: "Xmas Tree Lights"
description: "Arduino-gesteuerte LEDs"
date: 2022-06-07
tags: ["Arduino","LED", "Programmieren"]
---

Matt Parker vom Youtube-Channel **standupmaths** hatte vor einiger Zeit gezeigt, wie er seinen Weihnachtsbaum mit einzeln addressierbaren LEDs bestückt hatte, um diese dann als „3D Monitor“ benutzen zu können.
Hier das Video für den Kontext:

{{< youtube TvlpIojusBE >}}

Das musste ich auch haben.
Daher hatte ich mir zum nächsten Weihnachten 400 einzeln adressierbare LEDs vom Typ WS2812B bestellt und damit den Weihnachtsbaum geschmückt.
Für die Ansteuerung der LEDs habe ich dabei einen Arduino Nano 33 BLE verwendet, der über WLAN verfügt und angesteuert werden kann.

## Endergebnis
Das fertige Ergebnis sah dann so aus:

{{< twitter Abufari 1474744723692793858 >}}

## Bestimmung der 3D Koordinaten der LEDs

### Aufnahme der Fotos

Um im 3D Raum ein stimmiges Bild erzeugen zu können, muss man die Koordinaten jeder einzelnen LED kennen.
Das ist auch die größte Schwierigkeit an diesem Projekt.

Matt Parker hatte dabei die Idee, jeweils immer nur eine LED nach der anderen anzumachen und davon ein Foto zu machen, sodass man eine 2D Projektion der LEDs aus einer Richtung bekommt und die Zuordnung zu jeder LED kennt.
Wenn man das für mindestens zwei, besser vier Seiten macht, kann man daraus die 3D Koordinaten der LEDs zurückrechnen.
Er hatte dafür die Webcam des Laptops genommen und mit einem Pythonskript seinen Raspberry Pi angesteuert.

{{< katex >}}
Ich wollte mir etwas mehr Auflösung gönnen und die Fotos mit dem Handy machen.
Das Problem dabei ist die Synchronisation zwischen Handy und Arduino, wenn ich nicht gerade jedes Mal manuell den Auslöser drücken möchte (es sind insgesamt immerhin \\(400 \cdot 4 = 1600 \\) Fotos).

Dafür habe ich einen kleinen Server auf dem Arduino aufgesetzt, der über eine kleine API zur nächsten LED schaltet.
Somit schalten die LEDs nicht mit einer fixen Zeitspanne weiter sondern nur per HTTP Request.
Auf dem Handy habe ich dann einen kleinen
<a href="https://www.icloud.com/shortcuts/ad814d56b8564ad2b0cfd200306038c6" target=_blank >iOS Kurzbefehl</a>
erstellt, der den HTTP Request ausführt und den Kameraauslöser betätigt.
Ich hatte dafür die Kamera-App „Halide“ benutzt, weil ich dachte, dass die Auswertung mit RAW-Bildern einfacher wird, rückblickend hätte die normale Kamera-App aber problemlos ausgereicht.
Der Kurzbefehl besteht aus dieser Abfolge:
![iOS Kurzbefehl](ChristmasTree_Kurzbefehl.png "Kurzbefehl")

Für jede Seite des Baums müssen alle LEDs durchgeschalten werden, also insgesamt 400 mal.
Im Loop wird der GET Request mit dem Endpunkt [**N**]ext des Arduinos aufgerufen, der auf die nächste LED schaltet.
Das Foto wird aufgenommen und ein eine kleine Pause von 4 Sekunden eingelegt, um der Kamera-App genug Zeit zum schießen und abspeichern des Fotos zu geben.
Nachdem alle 400 Fotos aufgezeichnet wurden, wurde der Baum um 90° gedreht das Ganze dreimal wiederholt.

Zuerst wollte ich die Position der LEDs in den Bildern automatisiert über die Suche nach dem hellsten Punkt im Bild machen, in der Hälfte der Fälle werden die LEDs aber von Teilen des Baums überdeckt und man bekommt falsche Ergebnisse.
Weniger zeitaufwendig war dann letztendlich, die Position der LED im Bild manuell zu bestimmen.

### Bestimmung der LED Position in den Bildern
Hier hilft ein kleines Skript mit der OpenCV Library.
Dieses geht jedes Bild durch und man klickt einfach auf die Position, an der sich die LED befindet.
Da sich die LEDs ja entlang der Lichterkette bewegen, kann man die Position auch sehr gut abschätzen, selbst wenn man sie nicht direkt sehen kann.
Und da die Position am Ende aus den vier Richtungen gemittelt wird, sind kleine Abweichungen auch nicht so dramatisch.

Das Skript sieht dabei im Wesentlichen so aus:

{{< gist Abufari 886e530e07fa724207d836394684feaa >}}



Es gibt eine Liste an `filenames`, die nacheinander abgearbeitet wird.
Bei einem Klick werden die Mauskoordinaten in Pixeln in der `getCoordinate()`-Funktion empfangen und abgespeichert, womit man eine Liste an allen Koordinaten erhält.
Das ist immer noch etwas Arbeit, war aber deutlich einfacher, als irgendein clevers Skript zu schreiben, zumal man diesen Schritt nur ein einziges Mal machen muss (solange einem die Lichterkette nicht verrutscht).

### Berechnung der Koordinaten

Somit hat man nun vier Dateien mit Koordinaten in Pixeln von den vier Seiten des Tannenbaums.
Daraus lassen sich dann die 3D Koordinaten bestimmen, wobei ich ich die Pixelkoordinaten auf den Bereich eines `int16`\`s von etwa \\(-30000\thinspace --\thinspace +30000\\) gemapped habe.
Damit wird auch die \\(0\\) ungefähr in die Mitte des Baums gesetzt.

Den Code hätte man sicherlich hübscher gestalten können, aber es funktioniert.
Am Ende schreibe ich die Koordinaten gleich in einem Format, welches als Array im Arduino C-Code eingelesen werden kann:
{{< gist Abufari 11f715f6ad09de240bc17bd735fe8134 >}}

Und so sieht die Rekonstruktion der Koordinaten aus.
Die Baumform ist auf jeden Fall erkennbar:
<img src="locations.gif" />
