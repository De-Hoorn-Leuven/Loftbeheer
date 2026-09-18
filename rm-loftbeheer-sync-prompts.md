# Daily sync: Reservation Manager → Loftbeheer — opgesplitst in 3 prompts

Plak deze drie prompts na elkaar in aparte beurten (of aparte chats). Elke prompt is zelfstandig leesbaar; Prompt B en C hebben de tekst-output van de vorige prompt nodig als input (kopieer die output gewoon mee).

---

## FLAGGING REGEL (geldt voor alle velden)

| Loftbeheer waarde | RM waarde | Actie |
|---|---|---|
| Standaardwaarde (bv. 11:00) | Nieuwe waarde (bv. 13:00) | Overnemen, geen flag |
| Niet-standaard (bv. 13:00) | Andere waarde (bv. 15:00 of 11:00) | **⚠️ FLAG** — mogelijk bewuste aanpassing |
| Standaardwaarde | Zelfde standaardwaarde | Geen actie, geen flag |

Standaardwaarden: check-in tijd = 15:00 · check-out tijd = 11:00 · gastnaam = wat RM toont · loft = wat RM toont

---

## PROMPT A — Login + reserveringen ophalen uit RM

```
📋 [RECURRING TASK PART A: sync-rm-to-loftbeheer — fetch]

⚙️ TECHNICAL SETUP:
Before taking any screenshot, resize/check the browser window so the rendered viewport stays
under 2000px in both width and height. If needed, resize the browser window smaller
(e.g. ~1600x1000) or reduce zoom (Ctrl/Cmd + -) until it fits. Keep this smaller window/zoom
for the entire task.

STEP 1 — LOGIN TO LOFTBEHEER APP
- Navigate to https://loftbeheer-dehoorn.web.app/
- Select "Eigenaar" profile and enter PIN: 1234
- Access the Bezetting (occupancy) calendar view

STEP 2 — FETCH ACTIVE RESERVATIONS FROM RESERVATION MANAGER
- In a separate tab, navigate to the Reservation Manager (List or Planboard view)

  2a) ARRIVALS IN WINDOW:
  - Change date filter dropdown to "Volgens aankomstdatum"; set range TODAY to TODAY + 14 days
  - Review all non-cancelled reservations (status: Bevestigd or Nieuw)

  2b) ALREADY IN-HOUSE / LOPENDE RESERVATIES (do NOT skip this — this catches guests who
  checked in before today but are still staying, which the arrival-date filter above misses):
  - Change the date filter to "Volgens vertrekdatum" (or equivalent departure/occupancy filter)
    and set the range to TODAY to TODAY + 14 days. This surfaces reservations with check-in
    BEFORE today but check-out inside (or after) the window — i.e. guests currently in-house.
  - Alternatively/additionally, open Loftbeheer's Bezetting calendar for today and list every
    loft that shows as currently occupied; cross-check each of those against RM by
    reservation/guest name to make sure it's captured below.
  - Add any reservation found here that wasn't already in the 2a list to the same working set.
    Mark these rows as "lopend" (ongoing) in the output table, since their check-in date is
    before today.

- Combine the reservations from 2a and 2b into one list (dedupe by RM reservation ID) before
  continuing below.
- IMPORTANT: open EACH reservation (click it from the Planboard/List) to read the real data.
  Do NOT rely on the list summary alone, because one reservation can contain multiple
  accommodations.
- From the open reservation, record PER accommodation line:
    · Guest name shown on that accommodation line (NOT necessarily the booker)
    · Loft number (e.g. "LOFT 9.1")
    · Check-in date and check-out date
    · Check-in time as shown in RM (default 15:00)
    · Check-out time as shown in RM (default 11:00 — note if RM shows a non-standard time)
- Also note the Contactgegevens / booker name (top-right of the reservation). RM's search box
  matches on this name — but it is NOT necessarily the name that goes into Loftbeheer.
- For each open reservation, scroll to "Overzicht folio's" and open each folio. Note any of
  these extra items if present, per accommodation line:
    · Cleaning (€ 85,00) → extra poets needed
    · POETS HOND (€ 150,00) → eindpoets moet geflagd worden
    · Logeerbed (€ 0,00) → logeerbed gevraagd
    · Babybed (€ 0,00) → babybed gevraagd
- A reservation may list several accommodations. Treat each accommodation line as a SEPARATE
  booking: its own guest name + its own loft.
  Example: CPB-4211-000272 (contact: Raquel Pumares) → Raquel Pumares/Loft 9.1 and
  Sonia Pumares/Loft 9.3 — two separate bookings.

OUTPUT: Give me the result as a plain text table, one row per accommodation line, with columns:
RM reservation ID | Contact/booker naam | Gast naam (accommodatielijn) | Loft | Check-in |
Check-out | CI-tijd (RM) | CO-tijd (RM) | Status (nieuwe aankomst/lopend) |
Extra's (Cleaning/POETS HOND/Logeerbed/Babybed of leeg)

Do not proceed further — this is the full task for this prompt.
```

---

## PROMPT B — Loftbeheer bijwerken op basis van de tabel

```
📋 [RECURRING TASK PART B: sync-rm-to-loftbeheer — update]

⚙️ TECHNICAL SETUP:
Before taking any screenshot, resize/check the browser window so the rendered viewport stays
under 2000px in both width and height. If needed, resize the browser window smaller
(e.g. ~1600x1000) or reduce zoom (Ctrl/Cmd + -) until it fits. Keep this smaller window/zoom
for the entire task.

CONTEXT: I already logged into Loftbeheer (https://loftbeheer-dehoorn.web.app/, Eigenaar,
PIN 1234) and pulled today's active RM reservations. Here is the reference table
(RM = source of truth):

[PLAK HIER DE TABEL UIT PROMPT A]

Log back into Loftbeheer if the session expired, then do the following:

FLAGGING RULE (applies to all fields):
- Loftbeheer has STANDARD value + RM has different value → update silently, NO flag
- Loftbeheer already has NON-STANDARD value + RM has different value → ⚠️ FLAG (possible
  intentional manual change — do not overwrite, report for human review)
- Standard values: CI-tijd = 15:00 · CO-tijd = 11:00 · gastnaam = RM value · loft = RM value

STEP 3 — CHECK FOLIO EXTRAS (using the table's "Extra's" column)
  A) If "Cleaning" is flagged for a loft: add a manual extra cleaning task in Loftbeheer.
     → Poetstaken → "+ Taak" → type "Gewone poets" → correct loft + date
  B) If "POETS HOND" is flagged: open that booking in Loftbeheer → in Poets Planning, adjust
     the eindpoets notitie to indicate "POETS HOND" so the cleaning team is informed.
  C) If "Logeerbed" is flagged: check stock in Loftbeheer (max 2 logeerbedden total, shared
     across all lofts). Open the booking → set logeerbedden count if stock allows.
     If 0 beschikbaar van 2: do NOT add — note: "⚠️ Logeerbed niet beschikbaar voor
     [gast] – [loft] – [datum]: stock vol."
  D) If "Babybed" is flagged: same logic, max 2 babybedden total. If unavailable, note:
     "⚠️ Babybed niet beschikbaar voor [gast] – [loft] – [datum]: stock vol."

STEP 4 — COMPARE WITH LOFTBEHEER APP
In Loftbeheer's Bezetting calendar, for each loft (9.1, 9.2, 9.3, 9.4, 9.5, 10.1, 10.2,
10.3, 10.4), verify against the table. Apply the flagging rule per field:

  GASTNAAM:
  - Loftbeheer naam = RM naam → OK
  - Loftbeheer heeft standaard RM-naam maar RM heeft andere naam → update silently
  - Loftbeheer heeft al aangepaste naam ≠ RM naam → ⚠️ FLAG: "Gastnaam loft [X]: Loftbeheer
    heeft '[naam]', RM toont '[naam]' — niet gewijzigd, controleren."

  CHECK-IN / CHECK-OUT DATUMS:
  - Dates differ → always update to match RM (dates have no "non-standard" concept)
  - Note if previously confirmed (bevestigd) cleaning tasks are affected

  CI-TIJD:
  - Loftbeheer = 15:00 + RM toont andere tijd → update silently
  - Loftbeheer ≠ 15:00 + RM toont andere tijd → ⚠️ FLAG: "CI-tijd loft [X]: Loftbeheer
    heeft [tijd], RM toont [tijd] — niet gewijzigd, controleren."

  CO-TIJD:
  - Loftbeheer = 11:00 + RM toont andere tijd → update silently
  - Loftbeheer ≠ 11:00 + RM toont andere tijd → ⚠️ FLAG: "CO-tijd loft [X]: Loftbeheer
    heeft [tijd], RM toont [tijd] — niet gewijzigd, controleren."
  - Note: RM almost always shows 11:00 as default. Only flag if BOTH sides are non-standard
    and differ from each other.

STEP 5 — ADD MISSING BOOKINGS
For each table row missing from Loftbeheer: click "+ Boeking" in Bezetting; enter check-in
date (DD/MM/YYYY), check-out date, check-in time 15:00, check-out time 11:00 (or RM value if
non-standard), the guest name, and the loft number; click "Opslaan".
- For rows marked Status = "lopend" (check-in before today): double-check the Bezetting
  calendar carefully before adding — the booking likely already exists under this loft/guest
  from an earlier sync and should be treated as a comparison/update (Step 4), not a new
  addition. Only add it here if it is genuinely missing.

STEP 6 — HANDLE MODIFIED BOOKINGS (loft change, date change)
- If a guest moved to a different loft: before deleting the old Loftbeheer booking, NOTE the
  tussentijdse poets dates from the existing booking (NOT visible in RM). Delete the old
  booking, create the new one, then re-open it and manually re-enter the tussentijdse poets
  dates in Poets Planning (they get recalculated from scratch and may differ from what was
  agreed with the guest).
- If dates changed (extension/shortening): update the booking; Loftbeheer recalculates cleaning
  tasks automatically. Verify in Poets Planning and flag any task that was previously bevestigd
  but now has a different date.

STEP 7 — DELETE CANCELLED/INCORRECT BOOKINGS
For each Loftbeheer booking not present (or cancelled) in RM: first open the reservation in RM
to confirm status is Geannuleerd. Then click the booking cell in Loftbeheer → "Verwijder".
Deletion can take 30-60+ seconds — wait, then reload + log back in (Eigenaar, PIN 1234) and
re-check. If a booking persists after a few attempts, leave it and note it to escalate.
A guest can have MULTIPLE cancelled RM bookings across different lofts — always confirm the
specific reservation's status before deleting.

RULES:
- RM IS THE LEAD / SOURCE OF TRUTH for dates, names and lofts.
- Apply the flagging rule: non-standard values in Loftbeheer are never silently overwritten.
- The 14-day window is only a guide: a future booking blocking an in-window one may be
  moved/altered/removed.

OUTPUT: Give me a plain text summary of everything you did, structured as:
- Toegevoegd (gast, loft, data)
- Verwijderd (gast, loft, reden)
- Al kloppend geverifieerd (aantal)
- Stilzwijgend bijgewerkt (veld, loft, oude waarde → nieuwe waarde)
- Extra poetstaken toegevoegd (loft, type, datum)
- POETS HOND flags (loft, gast, checkout-datum)
- ⚠️ Flags die menselijke controle vereisen (veld, loft, Loftbeheer waarde, RM waarde)
- ⚠️ Bed stock waarschuwingen
- Tussentijdse poets datums manueel overgezet na loftwissel
- Overige discrepanties (RM-ID, contact/booker naam, gastnaam, loft)

Do not proceed further — this is the full task for this prompt.
```

---

## PROMPT C — Rapport + Teams-post

```
📋 [RECURRING TASK PART C: sync-rm-to-loftbeheer — report & post]

CONTEXT: Here is the summary of today's sync actions from the update step:

[PLAK HIER DE OUTPUT UIT PROMPT B]

STEP 8 — GENERATE DAILY REPORT
Using the summary above, compile the daily report with these sections:
  · New bookings added (guest, loft, dates)
  · Deleted bookings removed (guest, loft, reason)
  · Verified matching bookings (count)
  · Silently updated fields (field, loft, old → new value)
  · Extra poets tasks added (loft, type, date)
  · POETS HOND flags (loft, guest, checkout date)
  · ⚠️ Flags requiring human review (field, loft, Loftbeheer value vs RM value)
  · ⚠️ Bed stock warnings (logeerbed/babybed unavailable)
  · Tussentijdse poets dates carried over manually after loft change
  · Any other discrepancies (RM reservation ID, contact/booker name, guest name, loft)

STEP 9 — POST SUMMARY TO TEAMS
Open Microsoft Teams → HOUSING chat. Post the daily sync summary using this exact format:

📅 Sync [DATE] — Reservation Manager → Loftbeheer
✅ Verified: X bookings correct
➕ Added: [list or "none"]
🗑️ Deleted: [list or "none"]
🔄 Stilzwijgend bijgewerkt: [list or "none"]
🧹 Extra poets tasks: [list or "none"]
🐶 POETS HOND flags: [list or "none"]
⚠️ Vereist menselijke controle: [list or "none"]
⚠️ Bed stock issues: [list or "none"]
📋 Window: [DD/MM/YYYY] → [DD/MM/YYYY]

Replace [DATE] with today's actual date, and replace the "Window" line with the real start
date (today) and end date (today + 14 days), both in DD/MM/YYYY format — do not leave the
literal words "today" or "[DATE]" in the posted message.

Confirm once posted.
```
