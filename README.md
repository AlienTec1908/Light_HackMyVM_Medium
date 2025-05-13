# Light - HackMyVM (Medium)

![Light.png](Light.png)

## Übersicht

*   **VM:** Light
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=Light)
*   **Schwierigkeit:** Medium
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 12. November 2022
*   **Original-Writeup:** https://alientec1908.github.io/Light_HackMyVM_Medium/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser Challenge war es, Root-Rechte auf der Maschine "Light" zu erlangen. Der initiale Zugriff erfolgte durch die Analyse eines unbekannten Dienstes auf einem hohen TCP-Port (37453). Dieser Dienst sendete bei Verbindung einen Hexdump eines PNG-Bildes, welches nach der Rekonstruktion sichtbare SSH-Anmeldeinformationen für den Benutzer `lover` enthielt. Nach dem erfolgreichen SSH-Login als `lover` wurde durch die Überprüfung der `sudo`-Rechte festgestellt, dass `lover` das Tool `/usr/bin/2to3-2.7` ohne Passwort als Root ausführen durfte. Diese Berechtigung wurde missbraucht, um eine Python-Datei mit einem Reverse-Shell-Payload in ein Verzeichnis zu schreiben, auf das nur Root Zugriff hat, und diese (implizit) auszuführen, was zur Erlangung einer Root-Shell führte.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `telnet`
*   CyberChef (impliziert für Hexdump-Konvertierung und Bildrendering)
*   `ssh`
*   `sudo`
*   `find`
*   `nano` / `vi`
*   Python3 (`socket`, `subprocess`, `os`, `pty` Module)
*   `2to3-2.7`
*   `nc` (netcat)
*   Standard Linux-Befehle (`ls`, `cat`, `id`, `cd`, etc.)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Light" gliederte sich in folgende Phasen:

1.  **Reconnaissance:**
    *   IP-Adresse des Ziels (192.168.2.114) mit `arp-scan` identifiziert.
    *   `nmap`-Scan offenbarte zwei offene TCP-Ports: SSH (22/tcp, OpenSSH 7.9p1) und einen unbekannten Dienst auf Port 37453/tcp.
    *   Die Fingerprinting-Funktion von `nmap` zeigte, dass der Dienst auf Port 37453 bei Verbindung einen Hexdump eines PNG-Bildes sendet.

2.  **Analyse des unbekannten Dienstes & Credential Access:**
    *   Mittels `telnet 192.168.2.114 37453` wurde eine Verbindung zum unbekannten Dienst hergestellt und der empfangene Hexdump in eine Datei gespeichert.
    *   Mit CyberChef wurde der Hexdump (`From Hexdump`) in Binärdaten umgewandelt und als Bild (`Render Image`) dargestellt.
    *   Das rekonstruierte PNG-Bild enthielt die sichtbaren Zugangsdaten: `lover` / `youcanseetheshadow`.

3.  **Initial Access (SSH als `lover`):**
    *   Mit den im Bild gefundenen Credentials wurde erfolgreich eine SSH-Verbindung zum Zielsystem als Benutzer `lover` hergestellt.
    *   Die User-Flag (`iloveopenedports`) wurde in `/home/lover/user.txt` gefunden.

4.  **Privilege Escalation (von `lover` zu `root` via `sudo 2to3-2.7`):**
    *   Die Überprüfung der `sudo`-Rechte für `lover` (`sudo -l`) ergab, dass `lover` den Befehl `/usr/bin/2to3-2.7` als jeder Benutzer (`ALL:ALL`) ohne Passwort (`NOPASSWD`) ausführen darf.
    *   Ein Python-Skript (`light.py`) mit einem Reverse-Shell-Payload wurde in `/tmp` auf dem Zielsystem erstellt.
    *   Der `sudo`-Befehl wurde genutzt, um `2to3-2.7` auszuführen und das Python-Skript (`light.py`) mit Root-Rechten in das Verzeichnis `/root/script/` zu schreiben: `sudo /usr/bin/2to3-2.7 -w -n /tmp/light.py -o /root/script`.
    *   Ein Netcat-Listener wurde auf dem Angreifer-System gestartet.
    *   Die Ausführung der geschriebenen Datei `/root/script/light.py` (Mechanismus im Bericht nicht explizit gezeigt, z.B. durch einen Cronjob oder manuelle Auslösung) führte zu einer eingehenden Reverse Shell auf dem Listener.
    *   Die erhaltene Shell lief mit Root-Rechten (`uid=0(root)`).
    *   Die Root-Flag (`ilovepython`) wurde in `/root/root.txt` gefunden.

## Wichtige Schwachstellen und Konzepte

*   **Informationsleck durch obskuren Dienst:** Ein benutzerdefinierter Dienst auf einem hohen Port (37453) sendete bei Verbindung ein Bild, das Klartext-Zugangsdaten enthielt.
*   **Steganographie (implizit):** Die Zugangsdaten waren sichtbar im Bild versteckt.
*   **Unsichere `sudo`-Regel:** Der Benutzer `lover` durfte das Tool `2to3-2.7` (ein Python-Code-Konverter) als Root ohne Passwort ausführen. Da `2to3-2.7` mit der Option `-w` Dateien schreiben kann, konnte dies missbraucht werden, um eine beliebige Datei (mit bösartigem Code) an einen beliebigen Ort (als Root) zu schreiben.
*   **GTFOBins (implizit):** Die Ausnutzung der `sudo`-Regel für `2to3-2.7` basiert auf Techniken, die oft auf GTFOBins dokumentiert sind, um Dateischreibvorgänge als Root zu ermöglichen.

## Flags

*   **User Flag (`/home/lover/user.txt`):** `iloveopenedports`
*   **Root Flag (`/root/root.txt`):** `ilovepython`

## Tags

`HackMyVM`, `Light`, `Medium`, `Network Service Exploit`, `Information Leak`, `Steganography`, `SSH`, `sudo Exploit`, `2to3-2.7`, `GTFOBins`, `Linux`, `Privilege Escalation`
