+++
title  = "Ubiquiti UniFi AP AC LR – Einrichten, Resetten & per SSH verwalten"
date   = 2025-06-15T09:30:00+02:00
tags   = ["Ubiquiti", "UniFi", "WLAN", "SSH", "Testlab"]
categories = ["Netzwerkgrundlagen"]
draft  = true           # ← nach dem Kontrolllesen auf false setzen
description = "Schritt-für-Schritt-Anleitung: Factory-Reset, Adoption, SSH-Befehle und separates Testnetz für den UniFi AP AC LR."
+++

## 1 · Überblick

Der **UniFi AP AC LR** ist ein beliebter Long-Range-Access-Point für Lab- und Kleinbüro-Setups.  
In diesem Guide erfährst du …

* wie du den AP **factory-resettst** (Hardware-Reset & SSH),
* ihn **im Controller oder UniFi Mobile** adoptierst,
* ein **isoliertes Test-WLAN (VLAN 20)** plus internes Lab-Netz (VLAN 30) erstellst,
* und über **SSH-Befehle** Wartung oder Debugging durchführst.

![UniFi AC LR](https://static-assets.ui.com/f/unifi-ap-ac-lr.png)

---

## 2 · Werksreset

| Methode | Schritte | Hinweis |
|---------|----------|---------|
| **Hardware-Button** | 1 Steckernetzteil ab → an<br>2 Reset-Knopf mit Paperclip **> 10 s** halten<br>3 LED blinkt weiß → blau → aus | setzt alles zurück, DHCP wird erwartet |
| **SSH** | ```ssh ubnt@192.168.1.20```<br>Passwort = `ubnt` (Default)<br>```mca-cli``` → ```set-default``` | nur möglich, wenn Zugangsdaten noch Standard sind |

> **Tipp** : Nach dem Reset holt sich der AP per DHCP eine neue Adresse. CLI-Scanner wie `arp -a` oder UniFi-Mobile-App zeigen sie an.

---

## 3 · Adoption im UniFi-Controller

1. **Controller-Version ≥ 7.5** bereitstellen (Docker, CloudKey o. ä.).  
2. AP antwortet auf Layer-2-Broadcast und taucht als *Pending Adoption* auf.  
3. **Adopt** → ggf. Benutzernamen/Passwort *ubnt/ubnt* angeben.  
4. Firmware-Update startet automatisch → LED wird blau = fertig.

### Alternative : UniFi-Mobile-App  
Öffne *Devices → „+“ → Scan QR-Code* auf der Rückseite.

---

## 4 · Separates Test-Netz (VLAN 20) & internes Lab (VLAN 30)

| Ziel | Einstellung im Controller |
|------|---------------------------|
| **Create Network** | `Settings › Networks › Add New`<br>Type = Corporate, VLAN = 20, Gateway = z.B. `192.168.20.1/24` |
| **WLAN TestLab** | `Settings › WiFi › Add New`<br>SSID = `TestLab-20`, VLAN = 20 |
| **Intern Lab** | Network: VLAN 30, Gateway `192.168.30.1/24`<br>WLAN = `Lab-30`, VLAN = 30 |
| **Firewall Isolation** | `Settings › Firewall & Security`<br>Rule = **Block** `LAN_IN` **VLAN 20 → VLAN 30/Corpo` |  

> Für reine Lab-Tests genügt oft ein *„Allow all → Internet, Block → Corporate“*.

---

## 5 · Nützliche SSH-Befehle

```bash
# Anmelden (Standard nach Reset)
ssh ubnt@<IP>

# Auf Controller "inform" (falls Adoption hängen bleibt)
mca-cli
set-inform http://controller.fqdn:8080/inform

# Echtzeit-Log der Radios
logread -f | grep wlan

# Versionsinfo & Boarddaten
info

# Hidden: Sendeleistung testen (LR hat max 22 dBm @ 2.4 GHz)
iwconfig wlan0 txpower