# Worlds, locations and lore in SillyTavern

You want two things:

1. **Go somewhere:** type something like "go to the Victoria Housekeeping building" and have the characters who belong there show up, some of them at random.
2. **Talk to one character:** chat one-on-one and still have the world's lore apply.

SillyTavern can do both with three built-in features. You don't need any extensions:

| Feature | What it does for you |
|---|---|
| **World Info / Lorebooks** | Holds the world, location and faction lore. Entries switch on when a keyword shows up in the chat. |
| **Group Chats** | A "world" is a group that holds every character in that world. |
| **Slash commands + Quick Replies (STscript)** | One-click buttons that "move" you by muting and unmuting group members. |

I checked command and field names against the SillyTavern docs and the
v1.16 source. If something differs in your version, type `/help slash` in any
chat to see the commands your install has.

---

## 1. Put the world in a lorebook, not on the cards

A lorebook is a list of entries. Each entry has trigger **keys** and **content**.
When a key appears in recent messages, the content is added to the prompt.
Options worth knowing:

- **Constant (blue circle):** always on. Use it for the core facts of the world.
- **Normal (green circle):** fires on keywords. Use it for locations, factions, people and terms.
- **Sticky N:** stays active for N messages after it fires. For locations this
  matters a lot: the scene stays at "the Victoria office" after you stop saying its name.
- **Cooldown / Delay:** blocks an entry for N messages after it fires, or until the chat has N messages.
- **Inclusion Group:** entries with the same group label compete, and only one gets in.
  Give every location the same group (`location`) so two locations can't be active at once.
- **Probability %:** the chance an entry fires once its key matches. Use it for
  random flavor such as weather, rumors or a passing NPC.
- **Character Filter:** limits an entry to certain characters or tags, or excludes them.
- **Position "@D":** adds the entry at a depth in the chat. Depth 2 to 4 is good for rules the model must not forget.
- **Decorators:** start an entry's content with `@@activate` to force it on, or `@@dont_activate` to force it off.

### Where to attach the lorebook (this is what keeps lore in one-on-one chats)

| Binding | How to set it | When it applies |
|---|---|---|
| **Global** | World Info panel → choose it in the global selector | Every chat |
| **Character** | Character panel → 🌐 globe. **Shift+click** the globe (or More… → Link World Info) to add *extra* books | Whenever that character is speaking |
| **Chat** | Character panel → Chat Lore button | Only that chat |
| **Persona** | Persona panel → Persona Lore | Whenever you use that persona |

**Recommendation:** link the world book as an *extra* lorebook on every
character from that world (Shift+click the globe). Then:
- A one-on-one chat with Ellen still knows New Eridu, Hollows, Victoria Housekeeping and so on.
- Characters from other worlds don't get this lore.

> **Group chat catch:** in the default *Swap character cards* mode,
> SillyTavern loads only the **current speaker's** character lorebook (I confirmed this in
> `getCharacterLore()` in the source). If only Lycaon has the world book linked, it
> disappears from the prompt when Ellen speaks. Link the book to every member, or
> make it the group chat's **Chat Lore**.

### Starter file

`worlds/new-eridu.lorebook.json` imports directly. Go to World Info → Import →
pick the file. It contains:

- 2 constant entries: a world overview, and a narration rule ("only characters present at the location act").
- Location entries (`[LOCATION] …`) with **sticky 8**. Each one says who is
  usually present and who is often around. "Victoria Housekeeping" is filled in, and "Sixth Street" is a template.
- A faction entry, short character entries (so characters can *talk about* each other
  even when they aren't in the scene), and lore terms (Hollows, Ethereals, Proxy, Bangboo).

The lore text is a short summary I wrote from memory. Check it and expand it.
If you add more locations, put them all in the same inclusion group, for example `location`.

---

## 2. One group per world = "going to places"

1. Create a **Group Chat** called, say, *New Eridu*. Add every character from that world.
2. Set **Reply order** to **Natural Order**. Characters who are named reply first.
   After that it goes by each card's *Talkativeness*, then a random member speaks.
   This gives you the randomness for free.
   - Pooled Order also works: a random member who hasn't spoken since your last message.
3. Keep **Swap character cards** (the default). *Join* mode merges every card and
   tends to blur the characters.
4. **Muting** (the struck-out speech bubble on a member) means "not in this scene".
   Muted members never get picked to reply. Your location buttons mute and unmute them.
5. Optional: a **Narrator** character (a card whose description is "You narrate
   the scene and voice background NPCs in New Eridu…"). Leave it unmuted at every location
   for shopkeepers and random passers-by who don't have cards.

### Location buttons (Quick Replies)

Extensions → Quick Replies → create a set called `Places` and turn it on for the
group chat. Add one Quick Reply per location. Member names must match the card names
**exactly**.

**Victoria Housekeeping:** Lycaon is always there, and each maid has a 50% chance of being there:

```stscript
/member-disable Ellen Joe |
/member-disable Corin Wickes |
/member-disable Rina |
/member-enable Von Lycaon |
/rand from=0 to=1 round=round | /setvar key=r |
/if left={{getvar::r}} right=1 rule=eq {: /member-enable Ellen Joe :} |
/rand from=0 to=1 round=round | /setvar key=r |
/if left={{getvar::r}} right=1 rule=eq {: /member-enable Corin Wickes :} |
/rand from=0 to=1 round=round | /setvar key=r |
/if left={{getvar::r}} right=1 rule=eq {: /member-enable Rina :} |
/setvar key=location Victoria Housekeeping |
/sys *You push open the doors of the Victoria Housekeeping office.* |
/trigger
```

How it works:
- The `/sys` line contains "Victoria Housekeeping", so the location lorebook entry fires and stays on for 8 messages.
- `/trigger` asks one of the present members to react.

**Talk privately with one character in the group:** mute everyone except them:

```stscript
/member-disable Von Lycaon | /member-disable Corin Wickes | /member-disable Rina |
/member-enable Ellen Joe |
/sys *You find Ellen alone.* | /trigger Ellen Joe
```

A Quick Reply that asks where you want to go:

```stscript
/input Where to? |
/setvar key=location {{pipe}} |
/sys *You head to {{getvar::location}}.* |
/trigger
```

Use this for places that have lore entries but no fixed cast. Natural Order
decides who answers.

**Typing it in plain chat:** with location entries and the "only present characters act"
rule, even a normal message like *"I go to the Victoria Housekeeping building"* fires
the location entry. Muting is what makes absent characters actually stop
replying. If you want plain-text commands to *also* mute and unmute
automatically, set an entry's **Automation ID** to a Quick Reply with the same ID.
SillyTavern then runs that script whenever the entry fires.

### Commands I confirmed (from v1.16 source)

| Command | Aliases | Purpose |
|---|---|---|
| `/member-enable <name or index>` | `/enable` | Unmute a member |
| `/member-disable <name or index>` | `/disable` | Mute a member |
| `/member-add <name>` / `/member-remove` | `/addmember` | Change group membership |
| `/member-get`, `/member-count`, `/member-peek` | | Inspect members |
| `/trigger [name or index]` | | Make a member (or the next by order) reply |
| `/sendas name=X text` | | Post a message as a character |
| `/sys text` | | Neutral narrator message |
| `/inject id=X position=chat depth=N text` | | Add a hidden instruction at a depth (for example "Current location: …") |
| `/world name` / `/world state=off name` | | Turn a *global* lorebook on or off |
| `/getchatbook`, `/getcharbook`, `/getpersonabook`, `/getglobalbooks` | | Get bound lorebook names |
| `/wi-set-timed-effect`, `/wi-get-timed-effect` | | Control sticky and cooldown from scripts |
| `/rand`, `/setvar`, `/getvar`, `/if`, `/times` | | Logic for random casts |

Macros: `{{random::a::b::c}}` picks a new value every time it's evaluated.
`{{pick::a::b::c}}` picks once and stays fixed for that chat. Don't put
`{{random}}` in lorebook content to choose who's present, because it changes on every message.
Do the random part in the Quick Reply instead, as above.

---

## 3. Several worlds side by side

- One lorebook per world (`new-eridu`, `other-world`, …). Link each one only to its own characters.
- One group chat per world.
- If a character crosses between worlds, link both books to that character, or use
  the **Character Filter** on individual entries.
- For lorebooks with many entries, use **Scan Depth** of about 4 to 6 and a **Context %** budget of about 20 to 30.
  Turn on **Recursive Scan** so that a location entry naming "Lycaon" also pulls in Lycaon's entry.

## Sources
- World Info: https://docs.sillytavern.app/usage/core-concepts/worldinfo/
- Group chats: https://docs.sillytavern.app/usage/core-concepts/groupchats/
- STscript: https://docs.sillytavern.app/usage/st-script/
- Source checked: `public/scripts/slash-commands.js`, `world-info.js` in the `sillytavern@1.16.0` npm package
