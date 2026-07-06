## Was Sind Cookies

### Was sind Cookies und für was sie eigentlich gedacht waren
- Cookies speichern Daten oder Content wie z.B einen Warenkorb auf dem eigenen Computer ab 
	- Zweck Speicher sparen auf dem Server - spart Geld

### Wie Cookies funktionieren
- Das Internet bzw. das HTTP Protokoll ist "statelss", d.h jede Anfrage wird vergessen
- Mit Cookies kann man dieses Problem umgehen
- Bei einer Anmeldung oder einem Login werden die Daten überprüft im Backend und anschließend wird der Inhalt + Cookie an den Browser übermittelt
- Dieser Cookie wird dann auf dem Computer gespeichert mit einer ID
- Möchte man sich jetzt wieder anmelden erkennt der Server anhand des Cookies und der ID den Nutzer wieder
- Cookies können auch Einstellungen oder andere Konfigs speichern, sodass sie Internet "bestehen" bleiben

- **Grenzen von Cookies**
	- max. 300 Cookies in einem Bowser
	- Größe max. 4096 bytes
	- **eigentlich** nicht Domain überschreitend (also gilt nicht für andere Webseiten)

### Wie Cookies uns verfolgen
- **Eigentlich** kann ein Cookie der auf Internetseite A generiert wurde nicht bei Internetseite B auftreten 
- Jedoch kann Internetseite B auf bestimmte Weise auf Internetseite A verweisen (z.B Like Button)
	- **Folge** -> Internetseite B muss mit Internetseite A kommunizieren und verwendet dabei den Cookie Internetseite A -> jetzt Internetseite A Informationen über einen 

### Ziel von Cookies
- Daten sammeln um gezielte Werbung zu schalten
- Datenverkauf
- Oder das das Internet Login nicht vergisst -> Session Cookies

### Wie man sich gegen Cookies schützen kann
- Geeignete Browser -> Safari, Brave
- Browser Extensions
- Europäisches Recht -> GDPR -> General Data Protection Regulation
	- Gesetz das Internetseiten verpflichtet  ihre Cookies offen zu legen