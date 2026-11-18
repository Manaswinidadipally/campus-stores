# Campus Stores

Campus Stores is a Product Engineer work sample set at a fictional campus media equipment desk. Work through a club’s screening, from equipment pickup to a partial return and the next morning’s handover. Open **Candidate brief** in the app for the assignment.

## Reviewer guide: where to look

My work is in `app.js` (behaviour), plus a small block in `index.html` and `styles.css` (the "Still out" strip). Nothing else was changed, and there are no new dependencies.

To see it, open `index.html` and press **Restart sample shift**. Then:

1. **Loan register → New issue slip**: issue PR-04, SP-02 and AD-12 under RQ-214 to Kabir Sen, due Fri 8 pm.
2. Press **Move to return time**. In **Loan register**, press **Record return** on the projector and then the speaker slip. Look at the "who has these now?" part of the form and at the **Still out** strip under the shift banner.
3. Set the adapter slip's **Now with** dropdown to Leela Das.
4. Press **Move to next morning**. The adapter is flagged **Overdue**. Press **Record return** on the adapter slip, choose the unit you are actually holding, and confirm.
5. Check **Inventory**, **Requests → RQ-214** (per-item status) and **Desk book** (the automatic "Counter log" entries, in time order).

Code pointers, all in `app.js`: `register()` (holder dropdown and return form), the `data-return` branch of the submit handler (return, unit correction, holders of remaining items), `deskbook()` (time-ordered log), and `paint()` (the "Still out" strip).

## What works well in the original product

- **It stops the wrong equipment leaving the desk.** At pickup, trying to issue the USB-C adapter under the Debate Union request is refused ("outside the approved request"), and trying to issue a second adapter under the Film Society request is refused ("approved quantity… already on loan"). A desk volunteer cannot hand out something the club wasn't approved for by accident.
- **Inventory status can't drift from the loan register.** The Inventory status of each unit is worked out from the open slips, not typed in. The unit in repair (PR-06) is never offered for issue. When I issued and returned slips, Inventory always matched the register.
- **One slip per physical unit** gives each item its own record, which is what makes a partial return possible at all (the projector and speaker can close while the adapter stays open).

## What I changed, and why

The friction I hit during the shift: at 7:30 pm the register knew the adapter slip was open but said it was Kabir's, while the only place saying Leela has it was Kabir's free-text note in the Desk book. At 9 am, with two identical-looking adapters (AD-12 and AD-15), "Record return" closed whichever slip I clicked without asking which unit was in my hand.

1. **Who physically holds each unit.** Every slip now has "Collected by" (who signed for it) and "Now with" (who has it now), editable from a dropdown. Holder changes are logged. *Evidence:* after the 7:30 pm return, I set the adapter slip to Leela Das, and the register, the Still out strip and the request form all said so.
2. **Partial returns update the remaining items.** When I return the projector, the form lists the other items still out on that request and asks who has them. Anything the returner held is set to **Unknown** instead of being left showing the wrong person. *Evidence:* after Kabir returned the projector, the speaker and adapter slips showed "with Unknown" until I confirmed them.
3. **The return form checks the physical unit.** It asks which unit was handed back, with model and bin, so the identical adapters can be told apart. It starts blank when there is more than one possible unit. If a different unit comes back than the one recorded, the slip is corrected and a log entry asks the desk to check the other unit is on its shelf. *Evidence:* AD-12 was recorded, AD-15 came back, and the slip was corrected to AD-15.
4. **A "Still out" strip on every tab.** It lists every unit on loan, who has it, which club, when it is due, and an **Overdue** tag. It is the first thing the next shift sees. *Evidence:* on Saturday morning the strip showed the adapter overdue (due Fri 8 pm) while the Debate Union microphone (due Sat 10 am) was not.
5. **An automatic Desk book log.** Issues, holder changes and returns are written to the Desk book, in the order they happened, next to the manual handover notes. *Evidence:* the Friday 3:45 pm issues appear before Kabir's 7:30 pm message, and the Saturday return appears last.
6. **If nobody is confirmed as the holder, the return form makes you choose who brought it back** (it starts on a blank, required "Select…") instead of defaulting to the first person in the list. "Unknown" is also a valid choice there, for when a unit is dropped back without anyone saying who returned it; the Desk book then logs "returned by Unknown".

## Walkthrough: the same job with my change

- **Fri 3:45 pm.** Kabir collects the kit for Isha. I issue the projector, speaker and adapter to him. Each issue is logged, and the Still out strip shows three Film Society items with Kabir.
- **Fri 7:30 pm.** Kabir returns the projector and speaker. In each return form I confirm the unit and that Kabir brought it back. The remaining items on RQ-214 drop to "with Unknown". Using Kabir's message I set the adapter to Leela Das. I add a handover note. The strip now says `AD-12 … with Leela Das · Film Society · due Fri 8 pm`.
- **Sat 9:00 am.** The morning shift opens the app and sees one Film Society item still out, with Leela, overdue, without reading the Desk book. Leela arrives with an adapter. I press **Record return**, pick the unit she is actually holding from the list (model and bin shown), and confirm. The slip closes, Inventory and RQ-214 show everything returned, and the log records it.

## What I kept unchanged, and why

- **The layout, tabs, styling and storage key.** The change is about information, not a new look, and it keeps existing saved shifts working (a missing holder falls back to whoever collected the item).
- **One slip per physical unit and the issue form with its checks.** These are what stop the wrong equipment going out, so I built on them instead of replacing them.
- **The Outstanding / All slips views, Inventory filters, and manual handover notes.** They work, and staff already know them.
- **"Restart sample shift"**, so the exercise can still be reset to its fictional start.

## One real counter observation to validate the change

**Handover discrepancies per shift change.** At each shift change, the incoming person walks the shelves and the people on the loan list and counts mismatches with the register: units missing or in the wrong bin, a wrong identical unit, or a holder who is "Unknown" or wrong. If the change helps, that count per handover should fall over a few weeks compared with the weeks before. A companion measure is how many seconds it takes the incoming shift to answer "what is still out, and who has it?"

## Limits

**How it was tested.** By hand in a browser, I ran the adapter return (closing slip L-404 as Leela Das) and reloaded the page to check the register and Desk book kept it. Separately, I ran scripted checks of the whole shift in Node with a simulated browser (jsdom): issuing, quantity and type refusals, partial returns and holder updates, returning the other identical adapter, rejecting wrong units, returning as "Unknown", the Desk book order, overdue flags, and saving and reloading state. Those scripts are not in this repo, and they use a simulated browser, so they don't prove real-browser behaviour: for example, the browser's own "required field" check is not exercised, only the app's own validation. Not tested: phone-width layout with real touch, dark theme, keyboard-only use, or a screen reader. Not built: issuing a whole kit in one action, defaulting "Return by" from the request, and adding people outside the five on the list. I used an AI assistant (Claude) to help write and review the code.

## Run the app

Open `index.html` in a modern browser. The app includes its scripts, styles, and icons and can run offline.

To serve it over local HTTP, run this from the repository folder and visit `http://localhost:8000`.

```sh
python3 -m http.server 8000 --bind 127.0.0.1
```

Each browser stores its own sample shift locally when browser storage is available. **Restart sample shift** restores the fictional starting records.

## Source

- `index.html` contains the desk layout, candidate brief, and bundled icons.
- `app.js` contains the sample records and interactions.
- `styles.css` defines the desktop and mobile layouts.

Icon license notices are in [third-party/lucide-LICENSE.txt](third-party/lucide-LICENSE.txt).


## Submit your work

1. [Fork Campus Stores](https://github.com/virinchi-healthcare/campus-stores/fork) into your GitHub account. Complete the candidate brief and push your changes to your public fork. In your README, point reviewers to your work and explain the decision you made.
2. In your fork, open **Commits**, select the final commit you want reviewed, and copy the page URL. It should look like `https://github.com/YOUR-USERNAME/campus-stores/commit/COMMIT-SHA`.
3. [Submit your work through the Google Form](https://docs.google.com/forms/d/e/1FAIpQLScvzoh1R4WS-SkiwAVBQbMvniieJYSoOEfLMyVo5CyjOvYUgQ/viewform). Your **roll number**, **email ID**, and **final commit link from your public fork** are required. Enter your roll number and email ID in the form only.
