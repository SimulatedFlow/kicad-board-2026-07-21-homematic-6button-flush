```json
{
  "board_name": "Homematic-6Button-Flush",
  "one_liner": "Batteriebetriebener 868-MHz-Funk-Wandtaster (6 Taster, 3 links/3 rechts) fuer Homematic/BidCoS auf STM32G0-Basis mit steckbarem CC1101-Modul und CR2032-Knopfzelle — reiner Funk-SENDER (AskSinPP-Richtung), keine Netz-/Bus-Verbindung.",
  "market_gap": "Offener, handloetbarer DIY-Funktaster fuer Homematic. Passt hinter Standard-55er-Schalterrahmen (Gira/Busch/JUNG-Style) in eine UP-Dose. Verkauf unbestueckt/teilbestueckt.",
  "confidence": "medium",
  "price_eur": 22,
  "target_enclosure": "Standard-Unterputzdose Oe60mm; Platine rund Oe50mm, hinter 55er-Wippen-/Tasterrahmen; CC1101-Steckmodul + CR2032-Halter auf der Rueckseite (Bau-Tiefe UP-tauglich).",
  "injection_notes": "keine"
}
```

## GELOESTE SPEC-ENTSCHEIDUNGEN (Silvan 2026-07-26, MVP aus SPEC.md Kap.5)
Die offenen Punkte der SPEC sind jetzt FEST entschieden — baue GENAU diese Variante:
- **MCU:** STM32G0 (STM32G030F6Px, TSSOP-20). Klein, guenstig, Low-Power — Empfehlung der SPEC.
- **Rolle / Strom:** **Batterie-Sender (CR2032, 3 V)**. Reiner 868-MHz-Funk-Sender. **KEINE** Netzspannung, **KEIN** RS485-Bus, **KEIN** Netzteil an Bord. Das umgeht Netz-Isolation, das proprietaere HmIP-Wired-Protokoll und die UP-Netzspeisung vollstaendig (sichere UP-Variante).
- **Funk:** **CC1101 868-MHz-Modul**, steckbar ueber 1x8-Stiftleiste (Standard-CC1101-Modul-Pinout, AskSinPP-kompatibel). Das Modul traegt den Transceiver.
- **Antenne:** **λ/4-Draht (~86 mm) fuer 868 MHz.** Auf der Traegerplatine: ANT-Loetpad + rundum Antennen-Keepout (alle 4 Lagen kupferfrei), damit die Planes die Antenne nicht verstimmen.
- **Bedienung:** 6 Taster (3 links / 3 rechts) + 1 Status-LED. Taster nutzen die internen Pull-ups des STM32 (keine externen Pull-up-Widerstaende noetig).
- Sauberes, verkaufbares Board im Bluepill-Familien-Stil (unbestueckt/teilbestueckt; SMD ok; 4-Lagen mit Innen-GND-/Power-Plane).

## ⚠️ KRITISCHE BUILD-HINWEISE (ZUERST LESEN — dieses Board ist KEIN Standard-Bluepill-Carrier)
Rundes, **zweiseitig bestuecktes SMD-Board mit HF-Steckmodul**. Wichtig fuer einen DRC-sauberen, autoroutbaren Bau:
1. **Umriss RUND Oe 50,0 mm** (Zentrum 0,0), **NICHT** das 100×80-Rechteck mit 4× M3. Dazu **2× M2-Bohrungen** (Ø 2,2 mm) links/rechts fuer die Prototyp-Montage.
2. **ZWEI SEITEN, feste Zuordnung** (wie unten gelistet): **Front = F.Cu** (die 6 Taster + Status-LED, zeigen zur Schalterblende), **Rueckseite = B.Cu** (STM32, CR2032-Halter, CC1101-Steckmodul, Kondensatoren, SWD). Diese Seiten-Zuordnung nach dem Platzieren beibehalten.
3. **Falls die Pipeline `suggest_placement` (apply=true) erzwingt:** das stapelt evtl. alles in Reihen auf F.Cu und ausserhalb des Kreises — das ist hier FALSCH. Nach `suggest_placement` die kritischen Bauteile mit `move_component` ZURUECK auf ihre Seite/Koordinaten setzen (Taster 3 links/3 rechts auf F.Cu; STM32/BT1/Modul/SWD auf B.Cu innerhalb Oe50) und per `get_board_2d_view` visuell kontrollieren.
4. **ANTENNEN-KEEPOUT (Funkreichweite!):** Halbkreis-/Rechteck-Zone am oberen Rand (um das ANT-Pad bei X=0, Y=+21 mm, ca. 16×8 mm) auf **ALLEN 4 Lagen kupferfrei** — keine Tracks, keine GND-/Power-Plane-Fuellung. Die GND-Plane (In1.Cu) und die +3V0-Plane (In2.Cu) muessen diese Zone AUSSPAREN. Das CC1101-Modul (J_RF) so platzieren, dass sein Antennen-Ende zu dieser Zone zeigt.
5. **PLANE-VIAS (kritisch — haeufigster Grund fuer `unconnected` bei SMD-Boards!):** SMD-Pads liegen nur auf EINER Aussenlage und verbinden sich NICHT automatisch mit den Innen-Planes. Setze **VOR dem Signal-Routing neben JEDES GND-Pad ein Via zur GND-Plane (In1.Cu)** und neben JEDES **+3V0-Pad ein Via zur +3V0-Plane (In2.Cu)**. Danach `drc` pruefen: es duerfen KEINE GND-/+3V0-Pads mehr `unconnected` sein — ERST DANN die restlichen Signale routen.
6. Nur **~16 Bauteile** und ~17 Signalnetze (Taster, SPI zum Modul, SWD, LED) → nach korrekter Platzierung + Plane-Vias sicher autoroutbar (nur Signale auf F/B, GND/+3V0 ueber die Planes).

## BUILD-PROMPT

### Projekt-Spezifikation: Homematic-6Button-Flush (868-MHz-Funk-Wandtaster, Batterie)

Baue ein rundes, 4-lagiges UP-Tastermodul: **STM32G030** liest 6 Taster, treibt eine Status-LED und spricht per SPI ein **steckbares CC1101-868-MHz-Modul** an (Funk-Sender, AskSinPP/BidCoS-Richtung). Versorgung aus einer **CR2032-Knopfzelle (3 V)**. Reiner Sender — keine Netz-/Busverbindung.

---

### 1. Harte DFM- & Routing-Vorgaben
- **Bauteilanzahl:** ~16 Komponenten (uebersichtlich, garantiert autoroutbar).
- **Leiterbahnen & Abstaende:**
  - Signale (Taster, SPI, SWD, LED): Breite ≥ 0,3 mm, Clearance ≥ 0,25 mm.
  - Power (+3V0, GND): Breite ≥ 0,4 mm (Hauptversorgung ueber Planes).
- **Lagen-Stackup (4 Lagen, PFLICHT):**
  - **F.Cu (Front):** Signale + SMD-Pads der 6 Taster und der Status-LED.
  - **In1.Cu (Innen 1):** durchgehende **GND-Plane** (Netz `GND`), Antennen-Keepout ausgespart.
  - **In2.Cu (Innen 2):** durchgehende **+3V0-Plane** (Netz `+3V0`), Antennen-Keepout ausgespart.
  - **B.Cu (Rueckseite):** Signale + SMD-Pads von STM32, Kondensatoren, CR2032-Halter, CC1101-Modul-Header, SWD-Header.

---

### 2. Mechanik & Geometrie
- **Platinenform:** kreisrund, **Ø 50,0 mm** exakt (Zentrum X=0, Y=0). Passt hinter einen 55er-Schalterrahmen in eine UP-Dose Ø60 mm.
- **Befestigung:** 2× M2-Bohrungen (Ø 2,2 mm) bei `X=-22,0 mm, Y=0` und `X=+22,0 mm, Y=0` (je 1,0 mm kupferfreier Keepout-Ring).
- **ANT-Pad + Antennen-Keepout:** ANT-Loetpad bei `X=0, Y=+21,0 mm` (fuer den ~86 mm langen λ/4-Draht). Darum ein Keepout-Areal ca. 16×8 mm (oberer Rand) — auf **allen 4 Lagen** kupfer-/plane-frei.

---

### 3. Bestueckungsliste (BOM) & Seiten-Zuordnung

#### FRONT (F.Cu) — Bedienseite, zeigt zur Schalterblende
1. **SW1 (Taster links-oben):** SMD-Taster — *Symbol: `Switch:SW_Push`, Footprint: `Button_Switch_SMD:SW_SPST_B3U-1000P`*. Position ca. `X=-14, Y=+13`.
2. **SW2 (Taster links-mitte):** wie SW1. Position ca. `X=-14, Y=0`.
3. **SW3 (Taster links-unten):** wie SW1. Position ca. `X=-14, Y=-13`.
4. **SW4 (Taster rechts-oben):** wie SW1. Position ca. `X=+14, Y=+13`.
5. **SW5 (Taster rechts-mitte):** wie SW1. Position ca. `X=+14, Y=0`.
6. **SW6 (Taster rechts-unten):** wie SW1. Position ca. `X=+14, Y=-13`.
7. **LED1 (Status-LED):** *Symbol: `Device:LED`, Footprint: `LED_SMD:LED_0805_2012Metric`*. Zentral bei `X=0, Y=+6` (Lichtleiter zur Front).

#### RUECKSEITE (B.Cu) — Elektronik, im Inneren der UP-Dose
8. **U1 (STM32G030F6Px):** *Symbol: `MCU_ST_STM32G0:STM32G030F6Px`, Footprint: `Package_SO:TSSOP-20_4.4x6.5mm_P0.65mm`*. Zentral bei `X=0, Y=-4`.
9. **BT1 (CR2032-Knopfzellenhalter, 3 V):** *Symbol: `Device:Battery_Cell`, Footprint: `Battery:BatteryHolder_Keystone_3002_1x2032`*. Bei `X=0, Y=-12` (grosses Bauteil ~20 mm — Platz freihalten).
10. **J_RF (CC1101-868-MHz-Modul, steckbar):** 1×8-Stiftleiste — *Symbol: `Connector_Generic:Conn_01x08`, Footprint: `Connector_PinHeader_2.54mm:PinHeader_1x08_P2.54mm_Vertical`*. Nahe Oberkante bei `X=0, Y=+13`, Antennen-Ende Richtung ANT-Keepout (Y+).
11. **J_SWD (Programmier-/Debug-Header):** 1×5-Stiftleiste — *Symbol: `Connector_Generic:Conn_01x05`, Footprint: `Connector_PinHeader_2.54mm:PinHeader_1x05_P2.54mm_Vertical`*. Am Rand bei `X=-16, Y=-6`.
12. **C1 (100 nF, VDD-Entkopplung STM32):** *`Device:C`, `Capacitor_SMD:C_0603_1608Metric`*. Direkt an U1 VDD.
13. **C2 (100 nF, VCC-Entkopplung CC1101-Modul):** *`Device:C`, `Capacitor_SMD:C_0603_1608Metric`*. Nahe J_RF Pin 2.
14. **C3 (10 µF, Puffer an der Knopfzelle):** *`Device:C`, `Capacitor_SMD:C_0805_2012Metric`*. Nahe BT1 (+3V0/GND).
15. **R_LED (1 kΩ, Vorwiderstand Status-LED):** *`Device:R`, `Resistor_SMD:R_0603_1608Metric`*. Nahe LED1.
16. **ANT (λ/4-Draht-Loetpad, 868 MHz):** *Symbol: `Connector:TestPoint`, Footprint: `TestPoint:TestPoint_Pad_D2.0mm`*. Bei `X=0, Y=+21` im Antennen-Keepout (eigenes Netz `ANT`, Einzelpad fuer den ~86 mm λ/4-Draht).

*Taster brauchen KEINE externen Pull-ups (interne Pull-ups des STM32 nutzen). BOOT0/NRST bleiben auf Default (kein externer R/C noetig).*

---

### 4. Verbindungs- & Netzliste (Schematic-Leitfaden)
**Versorgung:**
- `+3V0` (Pluspol CR2032, In2.Cu-Plane): BT1 `+` → U1 `VDD`, C1, C2, C3, J_RF Pin 2 (VCC), J_SWD Pin 1 (VREF).
- `GND` (In1.Cu-Plane): BT1 `-`, U1 `VSS`, C1/C2/C3-GND, LED1-Kathode-Seite ueber R? (nein, siehe LED), J_RF Pin 1, J_SWD Pin 5, jeweils die zweite Seite der Taster SW1–SW6.

**Taster (interne Pull-ups, schalten GPIO gegen GND):**
- `BTN1..BTN6`: je ein Pin von SW1..SW6 → freier STM32-GPIO (z. B. `PA0, PA1, PA2, PA3, PA4, PA5`); der jeweils andere Taster-Pin → `GND`.

**Status-LED:**
- `LED_CTRL`: STM32-GPIO (z. B. `PA6`) → R_LED.
- `LED_A`: R_LED → LED1 Anode; LED1 Kathode → `GND`.

**CC1101-Modul (Standard-Modul-Pinout auf J_RF 1×8, SPI1):**
- J_RF Pin 1 = `GND`, Pin 2 = `+3V0`, Pin 3 = `RF_GDO0` (Interrupt, an STM32-GPIO z. B. `PB0`), Pin 4 = `RF_CSN` (an z. B. `PA8`), Pin 5 = `SPI_SCK` (an SPI1 SCK, z. B. `PB3`), Pin 6 = `SPI_MOSI` (SI, z. B. `PB5`), Pin 7 = `SPI_MISO` (SO, z. B. `PB4`), Pin 8 = `RF_GDO2` (an freien GPIO z. B. `PB1`).

**SWD (J_SWD 1×5):**
- Pin 1 = `+3V0`, Pin 2 = `SWDIO` (STM32 `PA13`), Pin 3 = `SWCLK` (STM32 `PA14`), Pin 4 = `NRST` (STM32 `NRST`), Pin 5 = `GND`.

**Antenne:**
- `ANT`: Einzel-Loetpad (ANT) — traegt den externen ~86 mm λ/4-Draht (868 MHz). Eigenes Netz, keine weitere Verbindung auf der Platine (das CC1101-Modul traegt seinen HF-Ausgang selbst; ANT-Pad ist die dokumentierte λ/4-Draht-Option). Im Antennen-Keepout platziert.

*Hinweis: Die konkreten STM32-Pin-Zuordnungen sind Vorschlaege — jeder freie GPIO ist elektrisch/DRC-gleichwertig; wichtig ist, dass alle Netze sauber verbunden sind.*

---

### 5. Arbeitsauftraege fuer die MCP-Werkzeuge (Server 'kicad')
1. **Projekt anlegen:** KiCad-Projekt `Homematic-6Button-Flush`.
2. **Schaltplan:** obige Netzliste sauber umsetzen, jedem Symbol den exakt genannten Footprint zuweisen, globale Labels fuer `+3V0`, `GND`, SPI, SWD, BTN1..6.
3. **PCB-Layout:**
   - Umriss (`Edge.Cuts`) als **Kreis Ø 50,0 mm**, Zentrum 0,0; 2× M2-Loecher bei X=±22, Y=0.
   - **Antennen-Keepout** (16×8 mm um X=0,Y=+21) auf allen 4 Lagen anlegen (Planes sparen ihn aus).
   - Bauteile nach der Front/Rueckseiten-Aufteilung platzieren (Taster/LED auf F.Cu; STM32/BT1/Modul/SWD/Cs auf B.Cu). Falls `suggest_placement` alles umwirft: kritische Teile per `move_component` zuruecksetzen, dann `get_board_2d_view` visuell pruefen (alle Bauteile IM Kreis, kein Bauteil bei 0,0 gestapelt, sichtbarer Routing-Abstand).
4. **4-Lagen-Stackup + Planes:** F.Cu/In1.Cu(GND)/In2.Cu(+3V0)/B.Cu. `add_zone` GND ueber die ganze Flaeche auf In1.Cu, `add_zone` +3V0 auf In2.Cu — **beide sparen den Antennen-Keepout aus**.
5. **Plane-Vias (PFLICHT vor dem Routing):** neben jedes GND-SMD-Pad ein Via zur In1.Cu, neben jedes +3V0-SMD-Pad ein Via zur In2.Cu. `drc` pruefen → keine GND-/+3V0-Pads mehr `unconnected`.
6. **Routing:** NUR mit MCP-Operation `autoroute` (Freerouting synchron). Nur Signale auf F.Cu/B.Cu; GND/+3V0 kommen ueber die Planes+Vias. Keine Signale auf In1/In2. KEIN `java -jar`/Bash/Hintergrundprozess — auf `autoroute` warten, bis es zurueckkehrt.
7. **DRC & Speichern:** `drc` laufen lassen, Fehler beheben (Ziel 0 severity=error), Board SPEICHERN (die geroutete `.kicad_pcb` MUSS auf Platte). Gerber/BOM/PDF/Render NICHT selbst exportieren — das macht die Pipeline (kicad-cli) danach.

*Fuehre den Bau autonom durch und fasse am Ende ehrlich zusammen: Umriss ok? ALLE Bauteile platziert (kein 0,0)? Anzahl Routing-Segmente? DRC-Status (severity=error)? .kicad_pcb gespeichert?*
