# Homematic 6-Taster-Unterputzmodul — Spezifikation (Entwurf)

**Stand:** 2026-07-21 · Idee von Silvan. Ziel: offenes DIY-Tastermodul, das in eine Standard-UP-Dose passt
und **wahlweise Homematic-Funk (868 MHz)** oder **Homematic IP Wired (RS485)** kann.

## 1. Formfaktor & Mechanik
- **Platine rund, Ø 49–50 mm** (passt in UP-Dose Ø60 mm, hinter 55er-Schalterrahmen Gira/Busch/JUNG-Style).
- **6 Taster: 3 links / 3 rechts**, als Wippen-/Doppeltaster-Cluster hinter dem Rahmen.
  - Umsetzung: 6× kurzhubige SMD/THT-Taster + gedruckte/gedrehte Stößel, ODER 6 Taster hinter einer
    2-fach-3-Wippen-Blende. Raster/Position an ein konkretes 55er-Wippen-Design anpassen.
- **Bauhöhe < 24 mm** inkl. Funkmodul auf der Rückseite (UP-Dose-Tiefe beachten).
- Pro Taster optional **Status-LED** (Lichtleiter zur Front) — 6× LED, über MCU/Charlieplexing oder Portexpander.

## 2. Zwei Bestückungsvarianten (gleiche Platine, unterschiedlich bestückt)
### Variante FUNK (868 MHz, klassisch Homematic/BidCoS-kompatibel)
- **CC1101-Funkmodul** (868,3 MHz) auf der **Rückseite** (SMD-Modul oder bedrahtbares Steckmodul).
  → BidCoS ist CC1101-basiert; Firmware z. B. Richtung AskSinPP (Arduino-Lib für selbstgebaute HM-Geräte).
- **Antenne:** λ/4-Draht (~86 mm) oder Helix — Platz in/hinter der Dose einplanen.
- **Stromversorgung** ist die Kernfrage (UP ohne Neutralleiter üblich):
  - a) **Batterie** (CR2032/AAA) → nur als Sender sinnvoll, MCU im Deep-Sleep, Wake per Tasterdruck.
  - b) **Kleines AC/DC-Netzteil** wenn N vorhanden (Hutschienen-frei, aber Platz/Isolationsabstände!).
  - → Empfehlung Prototyp: **Batterie-Sender** (einfachste, sicherste UP-Variante).

### Variante WIRED (Homematic IP Wired, RS485-Bus)
- **RS485-Transceiver** (MAX485/THVD1424) — HmIP Wired ist ein RS485-basierter 2-Draht-Bus mit Speisung.
- **Busspeisung** (typ. ~24 V) → **Buck-Regler auf 3,3 V** an Bord (kein Batterie-Problem).
- Bus-Klemme/Stecker klein (kein 5,08er Phoenix — UP-Platz begrenzt; JST/Federklemme).
- **Hinweis:** Das offizielle HmIP-Wired-Protokoll ist proprietär/nicht offen dokumentiert → realistischer
  Weg ist ein **generischer RS485-Modbus/eigenes Protokoll** an eine CCU/Bridge, ODER Reverse-Engineering.
  Für den ersten Bau: Bus **elektrisch** HmIP-Wired-kompatibel (RS485 + 24 V), Protokoll = eigenes/Modbus zur
  eigenen Bridge (Pi/CCU-Add-on). Ehrliche Bewertung in README aufnehmen (kein „offiziell HmIP" behaupten).

## 3. MCU-Kandidaten
- **STM32G0** (z. B. STM32G030/G031, klein, günstig, Low-Power) — bevorzugt für UP + Batterie.
- Alternativ **STM32F103** (Bluepill-Ökosystem, mehr Referenzcode), aber größer/stromhungriger.
- 6 Taster + 6 LEDs → GPIO reicht; sonst **I²C-Portexpander (PCF8575)**.

## 4. Offene Punkte / Risiken (vor Bau klären)
1. **UP-Stromversorgung** (Batterie vs. Netzteil vs. Busspeisung) — bestimmt Platinenlayout maßgeblich.
2. **HmIP-Wired-Protokoll proprietär** → ehrliche Positionierung „RS485-kompatibel, eigenes Protokoll".
3. **Funk-Zulassung/Antenne** — 868 MHz SRD-Band, Duty-Cycle beachten (nur Sender = unkritisch).
4. **Mechanik** an ein konkretes 55er-Wippen-System koppeln (Stößel-Position).
5. Sicherheitsabstände falls 230-V-Netzteil an Bord (Kriech-/Luftstrecken, besser galv. getrennt/Batterie).

## 5. Empfohlener erster Bau (MVP)
**Runde Ø50-Platine, STM32G0, 6 Taster + 6 LEDs, CC1101 auf Rückseite, Batterie (CR2032), λ/4-Draht.**
→ Reiner **Funk-Sender** (AskSinPP-Richtung), keine Netz-/Bus-Komplikation. WIRED-Variante als 2. Ausbaustufe
mit RS485 + Buck, sobald Funk-MVP steht. Danach in die KiCad-Board-Agent-Pipeline zum Auto-Bau geben.
