---
title: "Ein Cockpit für einen Menschen: den eigenen Tag mit vielen Agenten orchestrieren"
description: "Viele parallele Scanner, die aus Mail, Chat, Tickets, Kalender und sogar der eigenen KI-Historie einen priorisierten Tagesplan machen, mit zwei Versprechen: nichts geht verloren, nichts bleibt stecken."
excerpt: "Die persönliche Seite des Stacks. Ein Meta-Agent, der viele Quellen scannt, jeden losen Faden gegen die Tickets abgleicht und jede Aufgabe eine Nachricht davon entfernt macht, erledigt zu sein."
category: "Produktivität"
image: "/images/blog/cockpit-tag-mit-zehn-agenten.svg"
order: 6
date: 2026-07-25
author: "Chris 🦋 · Founder at bumbleflies / Senior Product Manager at JUNE"
readingTime: "8 Min."
published: false
lang: "DE"
---

Die bisherigen Artikel handelten von Systemen, die für viele arbeiten: das Nervensystem, der Marktplatz, die autonomen Agenten. Dieser letzte Teil dreht die Perspektive um. Er handelt von einem einzelnen Menschen und der Frage, die sich jeder Wissensarbeiter jeden Morgen stellt: *Was ist heute eigentlich wichtig, und was habe ich vergessen?*

Die Antwort ist ein persönlicher Meta-Agent, den ich „Cockpit" nenne. Er zieht jeden Arbeitskontext zusammen (Mail, Chat, Tickets, Video-Calls, Support-Postfach, Kalender, Aktivitätsprotokoll, lokale Verzeichnisse, den Agenten-Bus) und macht daraus einen priorisierten Tagesplan und *eine* nächste Handlung.

## Zwei Versprechen tragen das ganze Design

Alles am Cockpit folgt aus zwei Zusagen:

1. **Nichts geht verloren.** Jeder lose Faden aus jeder Quelle wird gegen die Tickets abgeglichen, und wenn er nirgends erfasst ist, in ein Eingangs-Ticket geroutet.
2. **Nichts bleibt stecken.** Jede aufgetauchte Aufgabe kommt mit einem fertig einfügbaren Fortsetzungs-Befehl, sodass sie „eine Nachricht davon entfernt ist, erledigt zu sein".

## Viele parallele Scanner, die niemals scheitern

Das Herz ist ein Fächer aus **Scannern**, je einer pro Quelle, alle gleichzeitig gestartet. Jeder Scanner bekommt denselben Auftrag und muss ein streng strukturiertes JSON-Ergebnis zurückgeben.

Ein Scanner **scheitert nie.** Wenn eine Quelle nicht erreichbar ist, gibt er keinen Fehler zurück, sondern ein sauberes „nicht verfügbar, Grund: …". Damit kann eine tote Quelle niemals den ganzen Lauf abbrechen. Das ist dieselbe Fehlertoleranz wie im Nervensystem: Das System arbeitet eingeschränkt weiter, statt umzukippen.

Und weil das strukturierte Ergebnis strikt validiert wird, bevor irgendetwas ihm vertraut, kann ein einzelner Scanner, der halluziniert oder Müll liefert, den Plan nicht vergiften. **Vertraue dem Modell nicht, verifiziere mit Code**, auch hier.

## Der Scanner, der die eigene KI-Historie liest

Ein Detail hebt das Cockpit heraus. Einer dieser Scanner liest die **eigene Gesprächshistorie von Claude Code**, die Protokolle der KI-Sitzungen des Menschen. Warum? Weil dort Verpflichtungen liegen, die man mündlich gegenüber der KI eingegangen ist („ich mache später X"), offene Fragen, angefangene Arbeitsschritte. Der Scanner holt diese in Arbeit befindlichen Zusagen zurück an die Oberfläche, damit sie nicht im Sitzungsverlauf versickern.

Ein Agent, der über die Arbeit eines Menschen mit anderen Agenten reflektiert.

Der Historie-Scanner ist der neueste Scanner, und die ehrliche Antwort ist, dass ich noch nicht weiß, ob er eine gute Idee ist oder nur eine seltsame. Er holt Zusagen an die Oberfläche, die ich sonst vergessen würde, aber er holt auch Rauschen hoch. Es ist der Scanner, bei dem ich mir am wenigsten sicher bin.

## „Gelesen heißt nicht erledigt"

Meine liebste Regel im Cockpit stammt, wie so vieles im System, aus einer echten Erfahrung. Der Mail-Scanner listet nicht nur ungelesene, sondern auch *gelesene* Mails. Denn: **Gelesen heißt nicht erledigt.** Eine gelesene Mail, bei der die Gegenseite zuletzt geschrieben hat und die eine Bitte oder eine Lieferung enthält, ist weiterhin offene Arbeit.

Dahinter steht eine konkrete Regression: eine Beispieldatei, gelesen an einem Montag, aber erst Tage später manuell bemerkt, weil „gelesen" fälschlich als „erledigt" galt. Die Regel ist die Lektion aus diesen verlorenen Tagen.

## Blockiert, wartend, als Nächstes

Das Cockpit unterscheidet sauber die Zustände, die die meisten Menschen im Kopf vermischen:

- **Blockiert:** Es fehlen Zugang, Daten oder eine Voraussetzung. Kann hoch priorisiert sein, aber nie „als Nächstes" dran sein.
- **Wartend:** Ich schulde noch eine Nachverfolgung, aber jemand anderes muss zuerst handeln.
- **Nächste Handlung:** der einzige höchstbewertete Faden, der *weder* blockiert *noch* wartend ist.

Diese Unterscheidung ist der Grund, warum die „nächste Handlung" immer wirklich machbar ist. Ein blockierter Punkt drängt sich nicht als To-do auf, das man sowieso nicht angehen kann.

## DRY, sogar hier

Auch das Cockpit folgt dem Prinzip „eine Definition, viele Laufzeiten". Es teilt sich eine Konfiguration und einen gemeinsamen Speicher mit einem leichteren Geschwister-Skill, der in jedem Projekt verfügbar ist. Und seine Scanner rufen dieselben Kommunikations-Skills aus dem Marktplatz auf, die auch die Agenten benutzen. Das Cockpit ist kein Solitär: Es baut auf denselben Fundamenten auf wie der Rest des Systems und liest aus ihnen.

## Wo die Serie landet

Damit ist die Serie wieder bei den Fundamenten, mit denen sie begonnen hat. Das ist die Bedeutung von „ein System": Ein Mensch, ein Agent und ein zeitgesteuerter Job benutzen dasselbe Vokabular, dieselben Tickets, dieselben Skills, nicht weil es elegant aussieht, sondern weil nur so aus einzelnen KI-Tricks ein Betriebssystem wird, das trägt.

Im Cockpit läuft dieses Muster in einem einzigen Morgenplan zusammen. Und wie jede Säule ist es aus Lektionen gebaut statt aus Versprechen: die gelesene Mail, die tagelang liegen blieb; der Scanner, der niemals scheitern darf; die Validierung, die ein halluziniertes Ergebnis einfriedet. Der Plan, den man jeden Morgen sieht, ist der aktuelle Zustand dieser Lektionen.
