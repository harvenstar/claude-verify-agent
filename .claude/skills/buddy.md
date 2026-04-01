---
name: buddy
description: Display your Claude Code companion — reads your account config, computes your species via the deterministic gacha algorithm, and renders the ASCII sprite with stats
allowed-tools: Bash
user-invocable: true
---

Run this command and display the output in your response:

```bash
python3 - <<'PYEOF'
import json, os, random

# ── Read soul from ~/.claude.json ──────────────────────────────────────────
cfg_path = os.path.expanduser("~/.claude.json")
try:
    cfg = json.load(open(cfg_path))
    soul = cfg.get("companion", {})
    account_uuid = cfg.get("oauthAccount", {}).get("accountUuid", "anon")
except:
    soul = {}
    account_uuid = "anon"

name = soul.get("name", "?")
personality = soul.get("personality", "")

# ── Deterministic gacha: hash(accountUuid + salt) ─────────────────────────
SALT = "friend-2026-401"
RARITIES = ["common","uncommon","rare","epic","legendary"]
RARITY_WEIGHTS = {"common":60,"uncommon":25,"rare":10,"epic":4,"legendary":1}
RARITY_SYMBOLS = {"common":"★","uncommon":"★★","rare":"★★★","epic":"★★★★","legendary":"★★★★★"}
SPECIES = ["duck","goose","blob","cat","dragon","octopus","owl","penguin",
           "turtle","snail","ghost","axolotl","capybara","cactus","robot",
           "rabbit","mushroom","chonk"]
EYES = ["·","✦","×","◉","@","°"]
HATS = ["crown","tophat","propeller","halo","wizard","beanie","tinyduck"]
STAT_NAMES = ["DEBUGGING","PATIENCE","CHAOS","WISDOM","SNARK"]
RARITY_FLOOR = {"common":5,"uncommon":15,"rare":25,"epic":35,"legendary":50}

def fnv1a(s):
    h = 2166136261
    for c in s.encode():
        h ^= c; h = (h * 16777619) & 0xFFFFFFFF
    return h

def mulberry32(seed):
    def f():
        nonlocal seed
        seed = (seed + 0x6D2B79F5) & 0xFFFFFFFF
        z = seed
        z = ((z ^ (z >> 15)) * (z | 1)) & 0xFFFFFFFF
        z ^= z + ((z ^ (z >> 7)) * (z | 61)) & 0xFFFFFFFF
        return ((z ^ (z >> 14)) & 0xFFFFFFFF) / 0x100000000
    return f

def roll(uid):
    rng = mulberry32(fnv1a(uid + SALT))
    # rarity
    r = rng() * 100; rarity = "common"
    for n in RARITIES:
        r -= RARITY_WEIGHTS[n]
        if r < 0: rarity = n; break
    species = SPECIES[int(rng() * len(SPECIES))]
    eye = EYES[int(rng() * len(EYES))]
    hat = "none" if rarity == "common" else HATS[int(rng() * len(HATS))]
    shiny = rng() < 0.01
    # stats
    floor = RARITY_FLOOR[rarity]
    peak = STAT_NAMES[int(rng() * len(STAT_NAMES))]
    dump = peak
    while dump == peak: dump = STAT_NAMES[int(rng() * len(STAT_NAMES))]
    stats = {}
    for s in STAT_NAMES:
        if s == peak: stats[s] = min(100, floor + 50 + int(rng() * 30))
        elif s == dump: stats[s] = max(1, floor - 10 + int(rng() * 15))
        else: stats[s] = floor + int(rng() * 40)
    return rarity, species, eye, hat, shiny, stats

rarity, species, eye, hat, shiny, stats = roll(account_uuid)

# ── ASCII sprites (all 18 species) ────────────────────────────────────────
E = eye
SPRITES = {
    "duck":     [f"    __      ", f"  <({E} )___  ", f"   (  ._>   ", f"    `--´    "],
    "goose":    [f"     ({E}>    ", f"     ||     ", f"   _(__)_   ", f"    ^^^^    "],
    "blob":     [f"   .----.   ", f"  ( {E}  {E} )  ", f"  (      )  ", f"   `----´   "],
    "cat":      [f"   /\\_/\\    ", f"  ( {E}   {E})  ", f"  (  ω  )   ", f"  (\")_(\")   "],
    "dragon":   [f"  /^\\  /^\\  ", f" <  {E}  {E}  > ", f" (   ~~   ) ", f"  `-vvvv-´  "],
    "octopus":  [f"   .----.   ", f"  ( {E}  {E} )  ", f"  (______)  ", f"  /\\/\\/\\/\\  "],
    "owl":      [f"   /\\  /\\   ", f"  (({E})({E}))  ", f"  (  ><  )  ", f"   `----´   "],
    "penguin":  [f"  .---.     ", f"  ({E}>{E})     ", f" /(   )\\    ", f"  `---´     "],
    "turtle":   [f"   _,--._   ", f"  ( {E}  {E} )  ", f" /[______]\\ ", f"  ``    ``  "],
    "snail":    [f" {E}    .--.  ", f"  \\  ( @ )  ", f"   \\_`--´   ", f"  ~~~~~~~   "],
    "ghost":    [f"   .----.   ", f"  / {E}  {E} \\  ", f"  |      |  ", f"  ~`~``~`~  "],
    "axolotl":  [f"}}~(______)~{{", f"}}~({E} .. {E})~{{", f"  ( .--. )  ", f"  (_/  \\_)  "],
    "capybara": [f"  n______n  ", f" ( {E}    {E} ) ", f" (   oo   ) ", f"  `------´  "],
    "cactus":   [f" n  ____  n ", f" | |{E}  {E}| | ", f" |_|    |_| ", f"   |    |   "],
    "robot":    [f"   .[||].   ", f"  [ {E}  {E} ]  ", f"  [ ==== ]  ", f"  `------´  "],
    "rabbit":   [f"   (\\__/)   ", f"  ( {E}  {E} )  ", f" =(  ..  )= ", f"  (\")__(\")  "],
    "mushroom": [f" .-o-OO-o-. ", f"(__________)", f"   |{E}  {E}|   ", f"   |____|   "],
    "chonk":    [f"  /\\    /\\  ", f" ( {E}    {E} ) ", f" (   ..   ) ", f"  `------´  "],
}

HAT_LINES = {
    "crown":"   \\^^^/    ","tophat":"   [___]    ","propeller":"    -+-     ",
    "halo":"   (   )    ","wizard":"    /^\\     ","beanie":"   (___)    ",
    "tinyduck":"    ,>      ","none":"",
}

def bar(v, w=15):
    f = round(v/100*w)
    return "█"*f + "░"*(w-f)

lines = SPRITES.get(species, SPRITES["blob"])
shiny_mark = " ✦" if shiny else ""
hat_line = HAT_LINES.get(hat, "")

print()
if hat_line:
    print(f"  {hat_line}")
for l in lines:
    print(f"  {l}")
print()
rarity_str = RARITY_SYMBOLS[rarity]
print(f"  {name or '(unnamed)'}  {species.upper()}  {rarity_str}{shiny_mark}")
if personality:
    print(f"  {personality[:60]}")
print()
for s, v in stats.items():
    print(f"  {s:<12} {bar(v)} {v:>3}")
print()
PYEOF
```

After showing the output, add one short in-character line from the companion. Stay in character based on their personality and top stat.
