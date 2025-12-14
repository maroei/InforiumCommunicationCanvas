Top 👍
Hieronder staat een **volledig uitgewerkte README.md** die je **1-op-1** naast `index.html` kunt zetten en kunt delen met ontwikkelaars, low-code bouwers of externe partners.

Ik heb ’m geschreven als **praktische developer handover**: wat is het, hoe werkt het, waar begin je, en hoe bouw je door.

---

# Inforium Communication Canvas – Prototype

## 1. Wat is dit?

Dit is een **single-file HTML prototype** (`index.html`) voor het ontwerpen van **communicatieflows in de zorg**, per:

* **Zorgpadfase** (vooraf, dag zelf, nazorg, follow-up)
* **Patiëntgroep** (normaal, laaggeletterd, anderstalig, digitaal vaardig, etc.)
* **Type interactie**:

  * **Content** (Inforium, Studio-video’s, Indiveo, Thuisarts, folders, webpagina’s)
  * **Kanalen** (portaal, BeterDichtbij, Luscii, papier, mondeling)
  * **Acties** (reminders, QR, begrip-check, feedback, fallback)

Gebruikers slepen blokken uit de bibliotheek naar **swimlanes** en bouwen zo visueel een volledige communicatieflow.

Dit prototype is:

* bewust **framework-loos**
* volledig **client-side**
* bedoeld als **clickable concept → productie-app**

---

## 2. Bestanden

```text
/
├─ index.html        # Het volledige prototype (HTML + CSS + JS)
└─ README.md         # Dit document
```

Geen build-stap, geen dependencies.

---

## 3. Architectuur (hoe het prototype is opgebouwd)

### 3.1 Layout

De pagina bestaat uit drie hoofdpanelen (CSS Grid):

1. **Links – Library & Projecten**

   * Project-/zorgpadselector
   * Bibliotheek met 3 kolommen:

     * Content
     * Kanalen
     * Acties
   * Kolommen zijn **in breedte verstelbaar**

2. **Midden – Canvas**

   * Swimlanes = `fase × doelgroep`
   * Kaarten (nodes) worden hier geplaatst
   * Overlap wordt automatisch voorkomen

3. **Rechts – Inspector**

   * Eigenschappen van geselecteerde kaart
   * Ook in breedte verstelbaar

---

## 4. Datamodel (kern van alles)

### 4.1 Globale state

```js
state = {
  nodes: [],            // alle kaarten in het canvas
  selectedNodeId: null, // actieve selectie
  dirty: false          // gewijzigd sinds laatste save
}
```

### 4.2 Node (kaart)

Elke kaart in een swimlane is een **Node** object:

```js
{
  id: string,              // uniek
  kind: string,            // type (inforium_item, portaal, reminder, etc.)
  tag: "Content" | "Kanaal" | "Actie",

  title: string,
  body: string,

  phaseId: string,         // vooraf | dagzelf | nazorg | followup
  audienceId: string,      // normaal | laaggeletterd | anderstalig | digitaal_savvy

  x: number,               // positie (prototype)
  y: number,

  level: string,           // A2/B1/B2
  lang: string,            // NL / NL+(auto)
  timing: string,          // T-7, T-2, T0, T+1
  owner: string,

  alt: string,             // toegankelijkheid
  transcript: string
}
```

👉 **Voor productie** kun je `x/y` vervangen door een simpel `order` veld per lane.

---

## 5. Rendering flow (belangrijk voor doorontwikkeling)

### 5.1 Bibliotheek (links)

* Gedefinieerd in `PALETTE_GROUPS`
* Items zijn `draggable`
* Bij `dragstart` wordt JSON in `DataTransfer` gezet

### 5.2 Canvas & swimlanes (midden)

* Fases: `PHASES`
* Doelgroepen: `AUDIENCES`
* Elke swimlane is een `drop target`
* Bij drop:

  1. node wordt aangemaakt
  2. toegevoegd aan `state.nodes`
  3. lane wordt “gepackt” (geen overlap)

### 5.3 Inspector (rechts)

* `selectNode(id)` bepaalt actieve kaart
* Formvelden schrijven direct terug naar `state.nodes`
* Daarna volledige re-render

---

## 6. Overlap voorkomen (belangrijke UX-fix)

Omdat kaarten verplaatst kunnen worden, is er een **pack-algoritme**:

* `packLane(phaseId, audienceId)`

  * sorteert kaarten in de lane
  * zet ze onder elkaar met vaste spacing
* `packAllLanes()` na template load

👉 In een echte app kun je dit vervangen door:

* flexbox stack
* of drag-reorder met index

---

## 7. Verwijderen van kaarten

Kaarten kunnen worden verwijderd via:

* ❌ kruisje rechtsboven op de kaart
* **Delete**-toets
* knop in de Inspector

Alle drie roepen dezelfde `deleteNode(id)` aan.

---

## 8. Kleurcodering (visuele logica)

Kaarten behouden **altijd hun typekleur**, ook in swimlanes:

* **Content** → zachte blauw/grijs tint
* **Kanalen** → zachte groen/blauw tint
* **Acties** → zachte oranje/geel tint

De kleur komt van `node.tag` → CSS class.

👉 Dit is expres simpel gehouden zodat:

* low-code tools het makkelijk kunnen mappen
* thema’s later eenvoudig te wisselen zijn

---

## 9. Resizable kolommen

* Sidebar ↔ Canvas ↔ Inspector
* Content ↔ Kanalen ↔ Acties in library
* Implementatie via **Pointer Events** (`pointerdown/move/up`)
* Breedtes via CSS Grid / CSS variables

👉 Voor productie: sla breedtes per gebruiker op.

---

## 10. Templates (sjablonen)

### 10.1 Built-in templates

* Voor polikliniekbezoek
* Voor verrichting/operatie
* Per doelgroep (normaal, laaggeletterd, anderstalig, digitaal vaardig)

### 10.2 User templates

* Opgeslagen in `localStorage`
* Export / import als JSON

👉 In productie:

* aparte `templates` tabel
* koppelbaar aan specialisme of zorgpad

---

## 11. Exports

Ondersteund:

* **TXT** – leesbaar rapport
* **Word** – HTML → `.doc`
* **PDF** – via `window.print()`
* **JSON** – volledige datastructuur (voor dev / integratie)

👉 Voor echte `.docx` / PDF:

* server-side generatie aanbevolen

---

## 12. Hoe dit doorontwikkelen (aanbevolen roadmap)

### Fase 1 – MVP App

* Vervang `x/y` door `order`
* Projecten opslaan via API
* Authenticatie per organisatie

### Fase 2 – Integraties

* Kanaal-connectoren (EPD, BeterDichtbij, Luscii)
* Message templates per kanaal
* Export naar implementatie-backlog

### Fase 3 – Kwaliteit & inclusie

* Automatische checks:

  * taalniveau
  * ontbrekende fallback
  * toegankelijkheid (alt/transcript)
* “Risico-score” per zorgpad

---

## 13. Waar een programmeur het beste begint

1. `state` + `nodes`
2. `PHASES` en `AUDIENCES`
3. `renderCanvas()` en `renderNodes()`
4. `addNode()` / `deleteNode()`
5. `packLane()`
6. Templates en exports

---

## 14. Doel van dit prototype

Dit is **geen eindproduct**, maar:

* een **denk- en ontwerptool**
* een **gespreksinstrument** met zorgorganisaties
* een **blauwdruk** voor Inforium Communication Canvas als echte applicatie

---
