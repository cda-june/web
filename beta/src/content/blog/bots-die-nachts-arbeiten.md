---
title: "Agenten, die arbeiten, während du schläfst: Claude Code als autonomer Daemon"
description: "Autonome Agenten, die einen Team-Chat beobachten, Code implementieren, Pull-Requests öffnen und Hotfixes ausrollen, und die Lektionen, die jede einzelne Leitplanke erklären."
excerpt: "Ein Wort im Chat weckt einen Agenten. Er implementiert, öffnet einen PR, meldet sich zurück. Mehrere Personas aus einem Bausatz. Und die Nacht, in der ein Agent im Leerlauf eine Menge Tokens verbrannte."
category: "Autonomie"
image: "/images/blog/bots-die-nachts-arbeiten.svg"
order: 5
date: 2026-07-22
author: "Chris 🦋 · Founder at bumbleflies / Senior Product Manager at JUNE"
readingTime: "10 Min."
published: false
lang: "DE"
---

Das ist der Artikel, auf den die im ersten Teil zitierte Kundenfrage wirklich zielte: *„Man gibt Feature-Requests textuell ein, und dann laufen Agents los, implementieren das, machen Pull-Requests?"*

Ja. So funktioniert es. Und so haben wir es bei JUNE gebaut.

## Die Grundidee: ein Agent ist Claude Code als Daemon

Ein „Agent" ist nichts anderes als **Claude Code, das als langlebiger Daemon in einem Container läuft**, angetrieben von Chat-Nachrichten statt von einem Menschen am Terminal. Es beobachtet einen Team-Chat-Kanal, und sobald ein Triggerwort fällt, implementiert es Code-Änderungen, öffnet Pull-Requests, arbeitet Review-Kommentare ab, rollt Hotfixes aus oder testet die Anwendung im Browser, alles unbeaufsichtigt.

Der Kern steckt in der Architektur: Es gibt mehrere Personas (einen Entwickler-Agenten, einen Support-Agenten, einen Produktmanagement-Agenten, einen Test-Agenten), aber sie sind **keine getrennten Codebasen.** Sie sind dieselbe Laufzeit, spezialisiert allein durch einen anderen Systemprompt, eine andere Liste installierter Skills und ein paar Umgebungsvariablen.

> „Ein neuer Agent ist einfach ein Container mit anderen Umgebungsvariablen und einem anderen Systemprompt."

Das ist das DRY-Prinzip auf Agenten-Ebene. Eine Verbesserung am gemeinsamen Bausatz erreicht sofort alle.

## Der Auslöser: kein Webhook, sondern ein simpler Poll

Man würde erwarten, dass ein solches System über Webhooks getrieben wird. Tut es nicht. Jeder Agent ist eine **kurz getaktete Poll-Schleife.** Immer wieder fragt er den Chat ab: Gibt es eine neue Nachricht mit dem Triggerwort? Ist die Antwort ja, startet er das Sprachmodell. Ist sie nein, schläft er weiter, ohne einen einzigen Token zu verbrennen. Keine Webhook-Registrierung, die stillschweigend kaputtgeht, keine nach außen offene Schnittstelle. Und um die gefühlte Latenz zu verstecken, gibt es einen hübschen UX-Trick: Noch bevor das Modell überhaupt startet, postet der Poll eine Vorab-Bestätigung, „Ich kümmere mich drum! 🐳", die sich laufend mit dem aktuellen Arbeitsschritt aktualisiert. Der Mensch sieht sofort eine Reaktion, statt auf den ersten Token zu warten.

## Wie es Claude Code kopflos ausführt

Im Kern ruft der Bootstrap Claude Code im **Headless-Modus** (ohne interaktive Bestätigungs-Dialoge) auf, mit übersprungenen Berechtigungs-Abfragen. Der Agent soll ja nicht bei jeder Datei nachfragen. Genau deshalb ist eine der wichtigsten Leitplanken ein **harter Stopp per Hook**: Ein Merge nach `master` oder `main` wird kategorisch verweigert.

> „Autonomes Mergen ist deaktiviert … überlasse den Merge einem Menschen."

Der Agent darf pushen, darf Pull-Requests öffnen, aber der Merge in die Hauptlinie bleibt eine menschliche Entscheidung. Das ist die „Hand am Bremshebel", von der sich das ganze System leiten lässt. Weil die Berechtigungen übersprungen sind, muss dieser Stopp ein *harter* Code-Stopp sein. Eine bloße Prompt-Regel wäre nur ein Ratschlag, den das Modell im Eifer ignorieren könnte.

## Die Lektionen

Fast jede Leitplanke in diesem System lässt sich auf eine konkrete Erfahrung aus dem Betrieb zurückführen. Das ist keine Peinlichkeit, sondern die Methode: **Das System wächst, indem es seine eigenen Fehler in Code gießt.**

**Die Nacht im Leerlauf.** Der Chat-Token eines Agenten war abgelaufen. Der Poll interpretierte das als „es gibt Arbeit" und feuerte immer wieder das Sprachmodell, um das vermeintliche Problem zu „lösen", die ganze Nacht. Am Morgen: eine Menge Token-Kosten für nichts. Die Antwort waren *mehrere* unabhängige Ausgaben-Wächter: ein stiller Token-Refresh, der zuerst versucht, das Problem ohne Modell zu lösen; eine Fehler-Zustandsmaschine, die nach wiederholten Fehlschlägen stark drosselt; und eine Wochenlimit-Markierung. Seither feuert ein abgelaufener Token *nie* das Modell; der Poll überspringt einfach den Tick.

Die Wächter haben das Verbrennen gestoppt, aber sie haben einen eigenen Fehlermodus mitgebracht: Ein Agent, der tatsächlich festhängt, versucht es jetzt nur noch selten erneut. Ist er wirklich kaputt, merkt man es entsprechend spät.

**Die Konfiguration auf dem Netzlaufwerk.** Anfangs lag die Agenten-Konfiguration auf einem persistenten Netzlaufwerk. Dort schlugen Klonen und Zurücksetzen des Git-Repositorys immer wieder fehl und hinterließen korrupte Dateien. Der defekte Ordner ließ sich nicht mehr löschen, und der Agent hing fest. Die Lektion: Die Konfiguration gehört auf flüchtigen lokalen Speicher, der bei jedem Start frisch geklont wird; nur der *Zustand* liegt persistent. Und niemals ein `sleep infinity` im Fehlerfall; lieber sauber beenden und den Container einen frischen Prozess starten lassen.

**Die Selbst-Neustart-Schleife.** Die Agenten lernen dazu: Nach einem Review-Kommentar schreiben sie eine neue Regel in ihre Wissensbasis und pushen sie. Anfangs interpretierte die Deployment-Automatik diesen Push als Konfigurations-Änderung, und startete den Agenten mitten in der Arbeit neu. Nach dem Neustart wiederholte der Agent die Arbeit, lernte, committete, pushte, startete neu … eine unendliche Selbst-Neustart-Schleife. Die Korrektur: Pushes in die Wissensbasis explizit von der Neustart-Logik ausnehmen.

**„Verlasse dich nie auf die Absender-Identität."** Weil der Agent über den Token eines Menschen postet, teilen sich Agent und Mensch einen Anzeigenamen. Ein Fall, in dem der Agent auf seine eigene Statusnachricht reagierte, weil darin das Triggerwort vorkam, führte zur Regel: Alles macht sich an Nachrichten-IDs fest, nie an der Anzeige-Identität.

**„Gemacht zählt erst als gelernt, wenn es geschrieben steht."** Der Test-Agent, der die Anwendung im Browser durchklickt, führt eine eigene Wissensbasis über die Oberfläche des Produkts. Der Leitsatz dahinter: Erfahrung, die nirgends notiert wird, ist verloren. Also schreiben die Agenten ihre Lektionen auf und pushen sie, deploy-neutral, sofort für alle verfügbar.

## Die selbstlernende Säule

Diese Agenten werden über die Monate besser, statt gleich schlecht zu bleiben: Sie schreiben nach jedem Review, nach jeder Korrektur generalisierbare Regeln in eine Wissensbasis und teilen sie. Der Entwickler-Agent lernt Konventionen des Frontends, der Support-Agent lernt die Choreografie eines Rollouts, der Test-Agent lernt die Eigenheiten der Oberfläche. **Die Werkzeuge verbessern die Dokumente, die die Werkzeuge steuern**, derselbe Zinseszins-Effekt wie im Marktplatz.

## Was man daraus mitnimmt

Autonome Agenten in Produktion sind kein Zaubertrick. Sie sind ein sehr gewöhnliches Werkzeug (Claude Code) in einer sehr disziplinierten Umgebung: ein billiger Poll statt fragiler Webhooks, harte Code-Grenzen um die riskanten Aktionen, mehrere unabhängige Kostenwächter und eine Kultur, in der jede Erfahrung zu einer neuen Regel wird.

Der schwierigste Teil ist nicht, den Agenten zum Arbeiten zu bringen. Der schwierigste Teil ist, ihm die Grenzen zu geben, an denen man nachts ruhig schläft.

Im letzten Teil der Serie drehe ich die Perspektive um: weg von den Maschinen, die autonom arbeiten, hin zu einem einzelnen Menschen und dem Cockpit, das dessen Tag mit vielen Agenten orchestriert.
