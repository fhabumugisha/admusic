---
description: Lire un flyer d'événement et ajouter l'événement dans db.json
argument-hint: [chemin/vers/image.jpeg]
allowed-tools: Skill(add-event)
---

<objective>
Delegate event extraction from flyer image to the add-event skill for: $ARGUMENTS

This routes to the specialized skill containing extraction rules, JSON template, and formatting conventions for the AD Music event database.
</objective>

<process>
1. Use Skill tool to invoke add-event skill
2. Pass user's request: $ARGUMENTS
3. Let skill handle the full workflow (read image, extract data, verify, insert)
</process>

<success_criteria>
- Skill successfully invoked
- Flyer image read and data extracted
- Event added to db.json after user verification
</success_criteria>
