---
title: "Das Nervensystem: Event-Automatisierung und wie man ein Sprachmodell ehrlich hält"
description: "Wie eine ständig laufende Automatisierungssäule aus Support-Mails klassifizierte Tickets macht und ein mehrstufiger Datenschutz-Filter, der zeigt, was „vertraue dem Modell nicht, verifiziere mit Code\" praktisch heißt."
excerpt: "Viele Workflows, eine Menge Verarbeitungsschritte, kein Mensch in der Schleife. Ein selbstlernender Ticket-Router und ein Datenschutz-Filter, der das Sprachmodell schreiben lässt, aber ihm kein Wort glaubt."
category: "Automatisierung"
image: "/images/blog/nervensystem-n8n-automatisierung.svg"
order: 3
date: 2026-07-15
author: "Chris 🦋 · Founder at bumbleflies / Senior Product Manager at JUNE"
readingTime: "9 Min."
published: false
lang: "DE"
---

Wenn die Fundamente, Tickets und Chat, das Skelett des Systems sind, dann ist die Automatisierungssäule das Nervensystem: immer wach, ereignisgetrieben, ohne Menschen in der Schleife. Sie reagiert auf jede Veränderung an einem Ticket und steuert daraus die anderen Systeme.

Wir haben diese Säule bei JUNE mit n8n gebaut, einer Open-Source-Automatisierungsplattform. Viele Workflows, eine Menge Verarbeitungsschritte. Sie verwandelt eine Support-Mail in ein klassifiziertes Ticket, ein Call-Recording in eine strukturierte Aufgabenliste, einen Kommentar in fertige Release-Notes. Zwei dieser Workflows verdienen einen genaueren Blick, weil hinter ihnen zwei Prinzipien stehen, die den Rest des Stacks erklären.

## Erstens: Der Code ist die Wahrheit, nicht die Handarbeit

Die prägende Entscheidung dieser Säule: Für jeden nichttrivialen Workflow ist die exportierte Konfiguration zwar die Quelle der Wahrheit, aber sie wird nicht von Hand geschrieben. Ein kleines Python-Skript *erzeugt* sie.

Warum? Weil das Hand-Editieren großer Workflow-Definitionen immer wieder dieselben Fehler produziert: falsche Knoten-IDs, kaputte Verbindungs-Arrays, Typ-Verwechslungen. Ein Generator-Skript macht diese Fehler nicht. Beide, Generator und erzeugte Konfiguration, liegen im Git. Das ist dieselbe Philosophie, die den ganzen Stack durchzieht: **Wo etwas deterministisch sein kann, soll es ein Skript sein.**

## Zweitens: ein Router, der aus menschlichen Korrekturen lernt

Der Support-Workflow ist ein selbstverbessernder Kreislauf aus zwei Teilen.

Der erste Teil fängt jede neue Support-Konversation ab und legt daraus ein Ticket an. Dann klassifiziert ein Sprachmodell das Ticket: In welche Liste gehört es? Die Klassifikation läuft als **Few-Shot-Prompt** (ein Prompt mit Beispielen statt festen Regeln): Das Modell bekommt Beispiele vergangener Tickets mit der Liste, in die sie einsortiert wurden. Ist es sich hinreichend sicher, verschiebt es das Ticket automatisch. Ist es unsicher, bleibt das Ticket im Eingang liegen.

Der zweite Teil schließt den Kreis: Immer wenn ein *Mensch* ein Ticket manuell aus dem Eingang verschiebt, wird genau diese Korrektur als neues Beispiel gespeichert. Die Trainingsdaten des Routers *sind* das Protokoll der menschlichen Korrekturen. Es gibt keinen separaten Labeling-Schritt. Am ersten Tag, mit leerer Beispiel-Tabelle, überspringt das System das Modell einfach und lässt alles im Eingang, und lernt ab der ersten manuellen Verschiebung.

**Die Korrekturen, die das Team ohnehin jeden Tag macht, sind die Trainingsdaten.** Man muss sie nur einfangen.

Ein Wert darin ist ehrliches Raten: die Sicherheitsschwelle, ab der das Modell selbst verschieben darf. Ich habe sie nach Gefühl gewählt, nicht durch Tuning. Bisher hat sie gehalten. Ob sie richtig sitzt oder ob das nur Glück war, weiß ich bis heute nicht.

Und das Ganze ist an jeder Verzweigung fehlertolerant entworfen: Schlägt die Klassifikation fehl, liegt das Ticket ja bereits im Eingang und ist dort sicher aufgehoben. Nichts geht verloren, nur weil das Modell mal patzt.

## Das Musterbeispiel: der Datenschutz-Filter

Wenn mich jemand fragt, wie man ein Sprachmodell in Produktion sicher macht, zeige ich diesen Workflow.

JUNE generiert kundensichtbare Release-Notes automatisch aus internen Tickets. Interne Tickets sind für Kolleg:innen geschrieben, nicht für die Öffentlichkeit: Sie enthalten Details, die in einer Release-Note nichts zu suchen haben. Deshalb liegt zwischen Ticket und Note von Anfang an eine deterministische Grenze, und nicht bloß eine Anweisung im Prompt.

Die naive Lösung wäre: dem Modell im Prompt sagen „nenne keine Namen". Ich tue das auch: Der Prompt enthält ein hartes Verbot mit Beispielen. **Aber ich vertraue dem Prompt nicht.** Der Ablauf ist deshalb gestaffelt:

1. **Das Modell schreibt** die Release-Note, mit der Anweisung, alles zu generalisieren.
2. **Ein deterministischer Filter prüft** das Ergebnis nach: Regex-Suchen nach E-Mails, URLs, Telefonnummern, IDs. Plus eine Heuristik, die großgeschriebene Wörter außerhalb des Satzanfangs, die nicht auf einer kleinen Erlaubnisliste stehen, als wahrscheinliche Namen markiert.
3. **Schlägt der Filter an, redigiert ein zweites Modell** die markierten Stellen: Es soll jeden markierten Begriff entfernen oder verallgemeinern.
4. **Derselbe Filter läuft ein zweites Mal.**
5. **Löst er *immer noch* aus, verweigert der Workflow hart** und postet stattdessen einen Kommentar: „Sanitizer konnte sensible Inhalte nicht entfernen, bitte manuell umschreiben."

**Modell schreibt → Code prüft → Modell korrigiert → Code prüft erneut → im Zweifel harte Verweigerung.** Das Sprachmodell wird eingesetzt, wo es glänzt (flüssig formulieren, verallgemeinern), aber der eingezäunte Bereich ist eng, und die Grenze ist deterministischer Code, kein weiteres Modell.

Der Filter-Code ist übrigens bewusst an zwei Stellen wortgleich dupliziert (identischer Code an beiden Stellen, statt in eine gemeinsame Funktion ausgelagert), mit dem Kommentar „halte diesen Block mit dem anderen synchron". Manchmal ist Redundanz die richtige Entscheidung.

## Der rote Faden

Der Router wird aus menschlichen Korrekturen klüger; der Filter glaubt dem Modell kein einziges Wort. Diese zwei Muster tragen die ganze Säule. Deshalb läuft sie nachts ohne Menschen in der Schleife.

Im nächsten Teil geht es um die Säule darüber: den Marktplatz, der Firmenwissen in installierbare Skills verpackt, die „Apps", die Menschen *und* Agenten benutzen.
