# Beloved - HackMyVM Writeup

![Beloved VM Icon](Beloved.png)

Dieses Repository enthält das Writeup für die HackMyVM-Maschine "Beloved" (Schwierigkeitsgrad: Easy), erstellt von DarkSpirit. Ziel war es, initialen Zugriff auf die virtuelle Maschine zu erlangen und die Berechtigungen bis zum Root-Benutzer zu eskalieren.

## VM-Informationen

*   **VM Name:** Beloved
*   **Plattform:** HackMyVM
*   **Autor der VM:** DarkSpirit
*   **Schwierigkeitsgrad:** Easy
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Beloved](https://hackmyvm.eu/machines/machine.php?vm=Beloved)

## Writeup-Informationen

*   **Autor des Writeups:** Ben C.
*   **Datum des Berichts:** 09. Juni 2021
*   **Link zum Original-Writeup (GitHub Pages):** [https://alientec1908.github.io/Beloved_HackMyVM_Easy/](https://alientec1908.github.io/Beloved_HackMyVM_Easy/)

## Kurzübersicht des Angriffspfads

Der Angriff auf die Beloved-Maschine umfasste die folgenden Schritte:

1.  **Reconnaissance:**
    *   Identifizierung der Ziel-IP (`192.168.2.128` unter dem Hostnamen `beloved.hmv`) und offener Ports (vermutlich 80 für HTTP/Nginx und 22 für SSH) – initiale Scans fehlen im direkten Log, werden aber impliziert.
    *   Die Webseite auf Port 80 wurde als WordPress-Installation (qdPM Theme) identifiziert.
2.  **Vulnerability Identification (wpDiscuz) & Initial Access (www-data):**
    *   `wpscan` (impliziert) identifizierte das Plugin `wpDiscuz`.
    *   `searchsploit` fand mehrere Exploits für `wpDiscuz 7.0.4`, darunter den Python-Exploit `49967.py` (Unauthenticated RCE).
    *   Der Exploit `49967.py` wurde ausgeführt und ermöglichte das Auslesen der `wp-config.php`, wodurch Datenbank-Zugangsdaten (`wordpress_user:secure_password_2021!!!`) offengelegt wurden.
    *   Der Exploit ermöglichte (impliziert) Remote Code Execution und führte zu einer Shell als Benutzer `www-data`.
3.  **Privilege Escalation (www-data zu Beloved):**
    *   `sudo -l` für `www-data` zeigte: `(beloved) NOPASSWD: /usr/local/bin/nokogiri`.
    *   Nokogiri (ein Ruby-Tool) wurde mit `sudo -u beloved /usr/local/bin/nokogiri /tmp/log.txt` gestartet, was eine interaktive Ruby-Shell (`irb`) öffnete.
    *   Innerhalb der `irb`-Shell wurde mit `system '/bin/bash'` eine Bash-Shell als Benutzer `beloved` erlangt.
4.  **Privilege Escalation (Beloved zu Root):**
    *   Als `beloved` wurde im Verzeichnis `/opt` eine Schwachstelle bei den Dateiberechtigungen ausgenutzt. Durch Ausführen von `touch ref` und `touch -- --reference=ref` konnte der Benutzer `beloved` den Besitz der Datei `/opt/id_rsa` (ein privater SSH-Schlüssel von Root) übernehmen.
    *   Der private SSH-Schlüssel von Root wurde aus `/opt/id_rsa` ausgelesen.
    *   Mit dem extrahierten privaten Schlüssel wurde erfolgreich eine SSH-Verbindung als `root` zum Zielsystem aufgebaut.
5.  **Flags:**
    *   Die User-Flag (`/home/beloved/user.txt`) wurde als `beloved` gelesen.
    *   Die Root-Flag (`/root/root.txt`) wurde als `root` gelesen.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `curl`
*   `wpscan`
*   `searchsploit`
*   `python3`
*   `vi`
*   `touch`
*   `sudo`
*   `nokogiri` (ausgeführt via `irb`)
*   `ls`
*   `cat`
*   `ssh`
*   `id`
*   `system` (Ruby-Befehl)
*   `bash`
*   `cd`

## Identifizierte Schwachstellen (Zusammenfassung)

*   **Veraltetes WordPress-Plugin `wpDiscuz` (Version 7.0.4):** Anfällig für Unauthenticated Remote Code Execution (RCE) / Arbitrary File Upload, was den initialen Zugriff als `www-data` und das Auslesen der `wp-config.php` ermöglichte.
*   **Unsichere `sudo`-Konfiguration (www-data zu Beloved):** Der Benutzer `www-data` konnte `/usr/local/bin/nokogiri` ohne Passwort als Benutzer `beloved` ausführen. Dies ermöglichte die Eskalation durch Starten einer Ruby-Shell und anschließender Bash-Shell.
*   **Unsichere Dateiberechtigungen / Exploit im Verzeichnis `/opt`:** Eine nicht näher spezifizierte Schwachstelle oder eine unsichere Konfiguration der Berechtigungen im Verzeichnis `/opt` erlaubte es dem Benutzer `beloved`, den Besitz des privaten SSH-Schlüssels von Root (`/opt/id_rsa`) zu übernehmen.

## Flags

*   **User Flag (`/home/beloved/user.txt`):** `020588f87676a40236192c324c1a57fc`
*   **Root Flag (`/root/root.txt`):** `d585a3099ec825ec1c086b50ce8ff7d3`

---

**Wichtiger Hinweis:** Dieses Dokument und die darin enthaltenen Informationen dienen ausschließlich zu Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Techniken und Werkzeuge sollten nur in legalen und autorisierten Umgebungen (z.B. eigenen Testlaboren, CTFs oder mit expliziter Genehmigung) angewendet werden. Das unbefugte Eindringen in fremde Computersysteme ist eine Straftat und kann rechtliche Konsequenzen nach sich ziehen.

---
*Bericht von Ben C. - Cyber Security Reports*
