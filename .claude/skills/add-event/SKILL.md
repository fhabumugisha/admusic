---
name: add-event
description: Read an event flyer image, extract event information (title, date, location, times, description), and add the event to db.json with the standardized format. Use when the user provides a flyer image and wants to add an event to the AD Music database.
---

<objective>
Extract event information from a flyer image and insert a properly formatted event entry into db.json for the AD Music event management system.
</objective>

<context>
AD Music uses a single `db.json` file served via My JSON Server as a REST API. Events have a strict schema with standardized formatting rules documented in CLAUDE.md. The `dateDisplay` field is shown as-is in the mobile app and must follow the standard French format.
</context>

<quick_start>
<workflow>

1. **Read the flyer image** using the Read tool on the image file path provided by the user
2. **Ensure the flyer image is in the repo root** with the correct name `DDMMYYYY.jpeg` (based on the event date). If the image is not already at the repo root (e.g. it's in Downloads or another folder), copy it to the repo root with the correct name.
3. **Read db.json** to determine the next sequential `id` (max existing + 1)
4. **Extract information** from the flyer:
   - Title (event name)
   - Location (venue name + full address)
   - Date
   - Start time / end time
   - Description (artists, DJs, dress code, practical info)
   - Whether it's an AD Music event (look for AD Music logo or mention)
5. **Build the JSON object** following the template and rules below
6. **Present extracted data** to the user for verification, flagging any uncertain or missing fields
7. **After user approval**, insert the event at the end of the `events` array in db.json
8. **Local pre-push verification** — Before committing, verify locally:
   - Validate that `db.json` is valid JSON (parse it)
   - Verify that `db.json` is under 10KB (My JSON Server limit)
   - Launch `npx json-server db.json --port 3456` in background
   - Fetch `http://localhost:3456/events/{NEW_ID}` to confirm the new event is served correctly
   - Stop the background server
   - If any check fails, fix the issue before proceeding to commit/push
9. **Commit and push** the changes (flyer image + db.json)
10. **Post-push verification** — Before confirming to the user, verify:
   - **API check**: Fetch `https://my-json-server.typicode.com/fhabumugisha/admusic/events/{NEW_ID}` using WebFetch and confirm the event is returned correctly
   - **Cover image check**: Fetch `https://fhabumugisha.github.io/admusic/DDMMYYYY.jpeg` using WebFetch and confirm the image loads (no 404)
   - If the API returns a 507 error, `db.json` exceeds the 10KB limit — archive older events into `db-archive.json`, commit, push, and re-verify
   - Report the verification results to the user
11. **Mobile screenshot verification** — After post-push verification succeeds:
   - Wait ~30 seconds for GitHub Pages and My JSON Server to propagate updates
   - Use the MCP Playwright tools to take a mobile screenshot:
     1. Navigate to `https://admusicapp-fd5d0.web.app/tabs/home` using the Playwright MCP `browser_navigate` tool
     2. Wait for the dynamic content to load (the MCP handles waiting natively, no hardcoded timeout needed)
     3. Take a screenshot and save it to the `deployed/` folder at the repo root using filename `DDMMYYYY.png` (event date). Example: `deployed/06122025.png`
   - Display the screenshot to the user for visual confirmation that the new event appears correctly
   - Close the browser session when done

</workflow>
</quick_start>

<formatting_rules>

**dateDisplay format**: `Jour DD mois YYYY à HH:MM`

- Day of week in French, capitalized: Lundi, Mardi, Mercredi, Jeudi, Vendredi, Samedi, Dimanche
- Day of month, 2 digits: 01, 06, 14, 31
- Month in French, lowercase: janvier, février, mars, avril, mai, juin, juillet, août, septembre, octobre, novembre, décembre
- 4-digit year
- `à HH:MM` only if start time is known

Examples:
- `Samedi 06 décembre 2025 à 21:00`
- `Vendredi 31 octobre 2025 à 22:00`
- `Dimanche 12 février 2023` (no time known)

**Image naming**: `DDMMYYYY.jpeg` (always this format, always `.jpeg`)

**Cover URL**: `https://fhabumugisha.github.io/admusic/DDMMYYYY.jpeg`

</formatting_rules>

<json_template>
```json
{
  "admusicEvent": true,
  "id": <next_sequential_id>,
  "title": "TITRE EN MAJUSCULES",
  "description": "Description with artists, DJs, dress code, practical info",
  "location": "VENUE NAME, ADDRESS, POSTAL CODE CITY",
  "date": "YYYY-MM-DDT23:59:59.000Z",
  "dateDisplay": "Samedi 06 décembre 2025 à 21:00",
  "heureDebut": "YYYY-MM-DDThh:mm:00.000Z",
  "heureFin": "YYYY-MM-(DD+1)T05:00:00.000Z",
  "cover": "https://fhabumugisha.github.io/admusic/DDMMYYYY.jpeg",
  "booklink": "https://linktr.ee/intladmusic"
}
```
</json_template>

<field_rules>

| Field | Rule |
|---|---|
| `id` | Max existing id in db.json + 1 |
| `title` | Main title from flyer, UPPERCASE |
| `location` | Full address UPPERCASE if available |
| `date` | Event date as `YYYY-MM-DDT23:59:59.000Z` |
| `dateDisplay` | Standardized French format (see formatting_rules) |
| `heureDebut` | ISO 8601 UTC. If not visible on flyer, ask the user |
| `heureFin` | Typically next day at `05:00:00.000Z` |
| `cover` | `https://fhabumugisha.github.io/admusic/DDMMYYYY.jpeg` using event date |
| `booklink` | Always `https://linktr.ee/intladmusic` |
| `admusicEvent` | `true` if AD Music logo/mention visible, otherwise ask user |
| `description` | Optional. Include artists, DJs, dress code, practical details |

</field_rules>

<verification_output>
After extracting data, present it to the user in this format:

```
Extracted event data:

- Title: ...
- Location: ...
- Date: ...
- Start time: ... (or "not found - please provide")
- End time: ... (or "default: 05:00 next day")
- Description: ...
- Booking link: https://linktr.ee/intladmusic
- AD Music event: true/false (or "uncertain - please confirm")
- Cover image: DDMMYYYY.jpeg

Missing/uncertain fields:
- [list any fields that couldn't be clearly read from the flyer]

Shall I add this event to db.json?
```
</verification_output>

<anti_patterns>
- Do NOT insert into db.json without user confirmation
- Do NOT guess dates or times - flag them as uncertain if unclear
- Do NOT use markdown headings (#) in the flyer reading output
- Do NOT forget to check the current max id in db.json before assigning
- Do NOT use `YYYYMMDD.jpg` format for new images - always `DDMMYYYY.jpeg`
</anti_patterns>

<success_criteria>
- Flyer image was read and information extracted
- Flyer image is placed at repo root as `DDMMYYYY.jpeg` (copied from source location if needed)
- All required fields populated (id, title, location, date, dateDisplay, cover, admusicEvent)
- dateDisplay follows the standardized French format exactly
- User confirmed the extracted data before insertion
- Event inserted at end of events array in db.json with correct JSON formatting
- db.json remains valid JSON after insertion
- db.json is under 10KB (My JSON Server limit). If over, archive old events to `db-archive.json`
- After push, API returns the new event at `GET /events/{id}`
- After push, cover image is accessible at its GitHub Pages URL
- Mobile screenshot taken and shown to user confirming the new event is visible
</success_criteria>
</output>