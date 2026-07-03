# Datenlöschung & Website-Recht


## Allgemeine Informationen


### Rollenverteilung
- **Incident Lead:** Lucas
- **Recovery Engineer:** Valentin
- **Legal & Communication Officer:** Nicola
- **Presenter / Moderator:** Amelie


### Unternehmen
- **Name:** STVDV (Strafvolzugsdatenverantwortlich)
- **Branche:** Strafverfolgung / Justiz
- **IT-Setup:** Datenbank mit Server


## Bearbeitung des Auftrags


### 1 Incident Summary
- **Was ist passiert?** Ein interner User löscht absichtlich Daten und Backups. Es fehlen Immutable Backups, Logging und ein 4-Augen-Prinzip. Dies wurde beim Restore-Test herausgefunden. 
- **Welche Systme/Daten sind betroffen?** Unsere Air Gap Datenbank.
- **Warum ist das kritisch?** Vergehen verfallen, aufgrund mangelnder Beweise. Zudem ist unser RPO = 0.


### 2 Recovery Plan


#### RPO / RTO defnieren
- RPO = 0
- RTO = 24h


#### Restore-Schritte
1. Zugang von User, welcher Daten gelöscht hat, sperren
2. Fälle anfragen
3. Fälle neu ins System speisen
4. 4 Augen Prinzip einführen
5. Immutable Backups machen


#### Prioritätenliste
System ist nicht ausgefallen, es fehlen nur Daten. Zurückgewinnung der Daten in folgender Priorisierung:
* **Priorität 1**: Fälle, welche in den nächsten 2 Tagen besprochen werden
* **Priorität 2**: Fälle, welche in einer Woche besprochen werden
* **Priorität 3**: Fälle, welche einen Termin in mehr als 2 Wochen haben
* **Priorität 4**: Fälle, welche noch keinen Termin haben


### 3 Öffentliches Statement
**Medienmitteilung der StVDV AG, Zürich**
Am 1. Juli 2026 kam es bei der StVDV zu einem internen Sicherheitsvorfall, bei dem Daten  gelöscht wurden. Die betroffenen Systeme wurden umgehend isoliert; die Wiederherstellung mit analogen Daten der entsprechenden Stellen läuft und wird möglichst zeitnah abgeschlossen. Nach aktuellem Kenntnisstand wurden **keine Daten entwendet oder veröffentlicht** – es handelt sich um eine Löschung, nicht um einen Datenabfluss. Der Eidgenössische Datenschutz- und Öffentlichkeitsbeauftragte (EDÖB) sowie die betroffenen kantonalen Stellen wurden informiert, eine Strafanzeige ist eingereicht. Die angeschlossenen Anstalten arbeiten vorübergehend mit etablierten Notfallprozessen; betroffene Behörden müssen nichts unternehmen und werden direkt durch unseren Support kontaktiert. Wir informieren laufend unter [stvdv.ch/status](www.stvdv.ch/status).


### Website Legal Pack


#### Impressum
StVDV AG

[info@stvdv.ch](info@stvdv.ch)

Max Müsterli

Zürich, Schweiz

Bezirksgericht Uster

#### Disclaimer
Die StVDV AG verwahrt die Strafvollzugsdaten mit grösster Sorgfalt und nach anerkannten Sicherheitsstandards (u. a. unveränderliche Sicherungskopien, Protokollierung, Vier-Augen-Prinzip bei kritischen Operationen). Ein unterbruchsfreier Betrieb kann dennoch nicht garantiert werden; geplante Wartungsfenster werden den angeschlossenen Behörden mindestens 5 Arbeitstage im Voraus angekündigt. Für Schäden aus leichter Fahrlässigkeit sowie für Ausfälle infolge höherer Gewalt oder vorsätzlicher Handlungen Dritter wird die Haftung im gesetzlich zulässigen Rahmen ausgeschlossen; die Haftung für Absicht und grobe Fahrlässigkeit der StVDV AG bleibt vorbehalten. Bei Datenverlust beschränkt sich die Pflicht der StVDV AG auf die Wiederherstellung aus der letzten verfügbaren Sicherung gemäss vereinbartem RPO.


#### AGB
- **Klausel 1 – Verfügbarkeit und Wartung:** Die StVDV AG gewährleistet eine Verfügbarkeit des Vollzugsportals von 99,5 % im Jahresmittel, gemessen ausserhalb angekündigter Wartungsfenster. Wartungsfenster werden mindestens 5 Arbeitstage im Voraus angekündigt und wo möglich in betriebsarme Randzeiten gelegt. Bei sicherheitsrelevanten Notfällen sind ausserordentliche Unterbrüche zulässig; die Kundschaft wird unverzüglich informiert.
- **Klausel 2 – Datensicherung und Datenverlust:** Die StVDV AG erstellt täglich Sicherungskopien (RPO 24 Stunden), bewahrt diese versioniert während 90 Tagen auf (Retention) und hält mindestens eine unveränderliche, räumlich getrennte Kopie vor (Immutable Backup, 3-2-1-Regel). Lösch- und Änderungsoperationen an Sicherungen unterliegen dem Vier-Augen-Prinzip und werden vollständig protokolliert. Bei Datenverlust beschränkt sich der Anspruch der Kundin auf die Wiederherstellung des letzten gesicherten Standes; weitergehende Ansprüche bestehen nur bei Absicht oder grober Fahrlässigkeit der StVDV AG.
- **Klausel 3 – Pflichten der Nutzenden:** Die Kundin (Angehörige des Justizministerium) stellt sicher, dass nur autorisierte Fachpersonen Zugang erhalten, persönliche Konten mit starken Passwörtern und Zwei-Faktor-Authentifizierung geschützt sind und Konten nicht geteilt werden. Austritte und Rollenwechsel von Mitarbeitenden sind der StVDV AG innert 24 Stunden zu melden, damit Berechtigungen entzogen werden können. Festgestellte Unregelmässigkeiten oder Verdacht auf Missbrauch sind unverzüglich dem Support zu melden.


### 5 Vergleich CH vs DE/EU (Tabelle)
| Thema | Schweiz | Deutschland/EU | Bedeutung für unseren Fall |
| ----- | ------- | -------------- | -------------------------- |
| Impressumspflicht | Keine allgemeine Impressumspflicht; nur bei kommerziellen Online-Angeboten Anbieterkennzeichnung nach Art. 3 Abs. 1 lit. s UWG | Strenge Impressumspflicht nach § 5 DDG (früher TMG) für praktisch jede geschäftsmässige Website; Verstösse sind abmahnfähig | Als kommerzielle Anbieterin führen wir ein vollständiges Impressum; falls je Kunden in DE/EU bedient werden, ist es ohnehin Pflicht |
| Disclaimer / Haftung | Haftung nach OR; Freizeichnung für leichte Fahrlässigkeit möglich, für Absicht und grobe Fahrlässigkeit nichtig (Art. 100 OR) | Ähnliche Grenzen über AGB-Inhaltskontrolle (§§ 305 ff., 309 BGB); pauschale Haftungsausschlüsse häufig unwirksam | Für die vorsätzliche Insider-Tat können wir die eigene Haftung nicht einfach wegbedingen – der Disclaimer schützt nur begrenzt, entscheidend sind die technischen Massnahmen |
| AGB / Gerichtsstand | Gerichtsstandsvereinbarung im B2B-Verhältnis zulässig; Schranke: missbräuchliche AGB ggü. Konsumenten (Art. 8 UWG) | Strenge AGB-Inhaltskontrolle; bei Verbrauchern gilt zwingend der Gerichtsstand an deren Wohnsitz | Unsere Kunden sind Behörden (B2B) → Gerichtsstand Zürich in den AGB ist zulässig und sinnvoll |
