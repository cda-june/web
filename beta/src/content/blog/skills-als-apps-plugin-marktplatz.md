---
title: "Skills als Apps: ein Plugin-Marktplatz für Firmenwissen"
description: "Ein interner App-Store für KI-Tooling: der Marktplatz ist der Store, jedes Plugin eine App, jeder Skill ein Feature. Und die eine Regel, die alles zusammenhält: Modell-Urteil plus deterministisches Skript."
excerpt: "Viele Plugins, noch mehr Skills, die per natürlicher Sprache aktiviert werden. Derselbe Code läuft für den Menschen am Laptop, den Agenten im Container und die CI. Wie man Firmenwissen versioniert statt kopiert."
category: "Plattform"
image: "/images/blog/skills-als-apps-plugin-marktplatz.svg"
order: 4
date: 2026-07-18
author: "Chris 🦋 · Founder at bumbleflies / Senior Product Manager at JUNE"
readingTime: "8 Min."
published: false
lang: "DE"
---

Jedes Unternehmen hat Verfahren, die im Kopf einzelner Leute stecken. Wie man ein Feature von der Idee bis zum Pull-Request bringt. Wie man einen Kunden von einem Altsystem migriert. Wie man einen Hotfix ausrollt, ohne die Produktion umzuwerfen. Dieses Wissen wird normalerweise mündlich weitergegeben, in veralteten Wiki-Seiten begraben oder von Person zu Person kopiert, jedes Mal ein bisschen anders.

Wir haben es bei JUNE stattdessen in einen **internen App-Store** verpackt. Die Metapher ist wörtlich gemeint: Der Marktplatz ist der Store, jedes Plugin ist eine App, und jeder Skill darin ist ein Feature.

## Wie es sich anfühlt

Ein Mitarbeiter fügt den Marktplatz einmal pro Rechner zu seinem Claude Code hinzu und installiert die Plugins, die er braucht. Danach aktivieren sich die Skills über **natürliche Sprache**: Man beschreibt, was man will, und Claude Code wählt den passenden Skill anhand seiner Beschreibung. Die Beschreibungen sind bewusst mit Trigger-Phrasen auf Deutsch *und* Englisch gefüllt: „deploy to prd" und „nach prd deployen" führen zum selben Skill. JUNE ist ein Legal-Tech-Unternehmen mit deutschsprachigem Team; die Sprache muss stimmen.

Viele Plugins, noch mehr Skills. Sie reichen von der Feature-Entwicklung (Planen, Bauen, Reviewen, Testen) über Code-Review für mehrere Technologie-Stacks bis zu Kunden-Onboarding, Support-Fixes und Zeiterfassung.

## Die eine Regel: Modell-Urteil plus deterministisches Skript

Die Regel, die den ganzen Marktplatz zusammenhält:

> Jeder Teil eines Skills, der deterministisch gemacht werden kann, SOLL ein Skript sein. Das Sprachmodell komponiert nur den Aufruf.

Ein Skill ist also nie „das Modell macht das schon irgendwie". Ein Skill ist: Das Modell trifft die Urteilsentscheidung (Welche Services sind vom Deployment betroffen? Ist dieser Review-Kommentar erledigt oder offen?), und ein deterministisches Skript führt die Mechanik aus.

Die Gründe:

- **Determinismus:** kein „das Modell hat diesmal Schritt 4 vergessen".
- **Auditierbarkeit:** Man difft das Skript, nicht einen KI-Gesprächsverlauf.
- **Geschwindigkeit und Kosten:** Ein Skript läuft in Millisekunden und verbrennt keine Tokens.
- **Wiederverwendbarkeit:** Mensch, Agent und CI rufen dasselbe Skript auf.

Jedes Skript liegt in zwei Varianten vor: einer für Linux/Mac (die Agenten-Container, die CI) und einer für Windows (die Laptops). Gleiche Argumente, gleicher Exit-Code, gleiche Ausgabe. Dadurch funktioniert *derselbe* Skill identisch für einen Menschen unter Windows, einen Agenten im Linux-Container und die CI-Pipeline. **Eine Definition, viele Laufzeiten.**

## Der Deploy-Dreiklang

Ein Beispiel: das Deployment. Drei Skills bilden zusammen einen Mini-Compiler:

Der erste nimmt eine Menge geänderter Dateien und ordnet jede ihrem Deployment-Ziel zu, erkennt neue Datenbank-Migrationen und bildet daraus einen strukturierten Deploy-Plan. Der zweite führt diesen Plan gegen eine Test-Umgebung aus. Der dritte zielt immer auf die Produktion, feuert die Datenbank-Migrationen zuerst als harte Sperre ab und danach die Services parallel.

Der Produktiv-Deploy ist **auslieferungs-agnostisch**: Er weiß nicht, ob ihn ein Mensch, ein Rollout-Skript oder ein Agent aufgerufen hat. Er gibt in dem Moment, in dem eine Produktions-Freigabe ansteht, ein strukturiertes Ereignis aus, „Freigabe nötig", und überlässt es dem Aufrufer, das dem Menschen zu präsentieren. Deshalb kann ein und derselbe Skill einen Menschen, ein Rollout und einen autonomen Agenten identisch bedienen.

## Wissen, das sich selbst verbessert

Ein Muster im Marktplatz mag ich besonders: eine Lernschleife. Der Migrations-Playbook-Skill, der Kunden von Altsystemen übernimmt, liest und schreibt ein einziges kanonisches Dokument. Nach jeder Migration hängt ein „Lernen-einfangen"-Schritt die neuen Erkenntnisse an genau dieses Dokument an, und schlägt Änderungen an seinen eigenen Regeln und Aufwands-Tabellen vor.

Das heißt: **Das Werkzeug verbessert das Dokument, das das nächste Werkzeug steuert.** Jede Migration macht das Playbook besser, statt ein Einzelfall zu bleiben. Wissen wächst wie mit Zinseszins, statt zu verwittern.

Ehrliche Einschränkung: Die Lernschleife läuft erst über eine Handvoll Migrationen. Ich glaube an diesen Zinseszins-Effekt, bewiesen habe ich ihn noch nicht.

## Analytics als fester Release-Schritt

Ein Detail, das zeigt, wie ernst „interner App-Store" gemeint ist: Jeder neue Skill *muss* im Nutzungs-Dashboard registriert werden. Ohne diese Registrierung sammelt das System keine Daten darüber, welche Skills tatsächlich benutzt werden, und ohne diese Daten kann man nicht priorisieren, was man verbessert.

Und die Qualitätssicherung? Jeder Skill, der geändert wird, durchläuft in der CI eine **echte Integrations-Prüfung**: Sie ruft den Skill end-to-end als echten Subprozess auf und prüft seine strukturierte Ausgabe. Keine Mocks. Wenn ein Skill kaputt ist, fällt das auf, bevor er jemanden erreicht.

## Was das ersetzt

Wissen verteilt sich, driftet auseinander, geht verloren, und die übliche Antwort sind Dokumentationen, die niemand liest. Meine Antwort: Firmenwissen als installierbare, versionierte, getestete Skills, benutzbar per natürlicher Sprache, von Menschen und von Maschinen, mit exakt demselben Code.

Und genau dieser gemeinsame Code ist die Brücke zum nächsten Teil: Wenn ein Mensch und ein autonomer Agent dieselben Skills ausführen, dann ist ein Agent nur noch ein Container mit einem anderen Systemprompt. Wie das aussieht und welche teuren Fehler mich dorthin geführt haben, zeigt der nächste Artikel.
