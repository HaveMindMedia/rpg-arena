Create a complete multiplayer browser-based isometric RPG game. Write all code to a single file: /Users/semitallc/Projects/rpg-arena/index.html

## REQUIREMENTS

### Stats System
- 6 stats: Attack, Defense, Speed, Energy, Intelligence, Command (each 1-6)
- Contest = stat + 1d6 roll
- Character creation: input name, click "Create Character" → stats 1,2,3,4,5,6 randomly shuffled across the 6 stats
- Auto-assign class by highest stat:
  - Attack → Warrior → Berserker Strike (double damage 1x/combat)
  - Defense → Guardian → Fortify (negate all damage 1x/combat)
  - Speed → Scout → Evasion (auto-dodge 1x)
  - Energy → Mage → Energy Burst (AoE damage to all nearby enemies)
  - Intelligence → Sage → Mind Link (+3 to counter-roll)
  - Command → Commander → Rally (nearby allies +2 to next roll)

### Combat System
- Real-time: AI acts every 2000ms
- Attack roll: attacker(ATK + d6) vs defender(DEF + d6); damage = difference + 1 if attacker wins
- Speed stat determines order (higher acts first)
- HP = 10 + Energy*2
- Special ability cooldown reduced by Intelligence
- Scrolling combat log showing dice rolls

### Isometric World (Canvas-based)
- 32x32 tile isometric map rendered on HTML5 canvas
- Tile types with 3D isometric diamond rendering (top face + two side faces):
  - Grass: bright green top (#6dbf67), medium green sides (#4e9948)
  - Stone: light gray top (#b0b0b0), dark gray sides (#787878)
  - Water: animated blue (#4fc3f7) with sine wave animation each frame
  - Forest: green top with brown tree cylinder drawn on top
  - Wall: gray with darker merlons on top edge
  - Dungeon: very dark gray (#303030) with orange glow circle (torch)
- Each tile is 64px wide, 32px tall in isometric projection
- Camera pans to follow player
- Smooth character movement animations between tiles

### Characters (drawn on canvas)
- Isometric box/figure shape colored by class
- Class colors: Warrior=red, Guardian=blue, Scout=yellow, Mage=purple, Sage=cyan, Commander=orange
- Floating health bar above character (green→yellow→red based on HP%)
- Name label above health bar
- Floating damage numbers that rise and fade out
- Death animation: character shrinks and fades

### AI Mobs
- New AI spawns every 15 seconds, max 15 on map
- Same stat generation as players (shuffle 1-6)
- Same class/ability assignment
- Behaviors (checked every 500ms):
  - Wander randomly if no enemies within 5 tiles
  - Chase nearest enemy (player or other AI) if within 5 tiles
  - Attack if adjacent (distance=1)
  - Flee if HP < 20% of max
  - Use special ability if cooldown=0 and enemy in range
  - AIs attack each other when no players nearby
- AI names: pick from adjectives [Fierce, Swift, Dark, Grim, Ancient, Wild] + monsters [Troll, Goblin, Orc, Shade, Wraith, Beast]

### Items
- Scattered on map, auto-pickup when player steps on tile
- Each item boosts one stat by +1 (max 9)
  - Attack: "Iron Sword"/"Battle Axe"/"War Hammer" — draw red diamond icon
  - Defense: "Shield"/"Chain Mail"/"Tower Shield" — blue diamond icon
  - Speed: "Swift Boots"/"Wind Cloak"/"Haste Ring" — yellow diamond icon
  - Energy: "Life Crystal"/"Health Potion"/"Vitality Gem" — green diamond icon
  - Intelligence: "Tome of Knowledge"/"Mystic Orb"/"Sage Staff" — purple diamond icon
  - Command: "Warlord Banner"/"Horn of Command"/"General Badge" — orange diamond icon
- Gentle floating animation (bob up and down)
- Glow effect (radial gradient)

### Multiplayer (BroadcastChannel)
- Use window.BroadcastChannel("rpg-arena") to sync between browser tabs
- Each tab is one player with unique ID (UUID)
- First tab to open becomes host (manages AI spawning, world updates)
- Host broadcasts state every 100ms; non-hosts receive and render other players
- Players appear on each other's screens with their colors

### Goals Panel
- 3 active goals displayed in left panel with progress bars
- Goal types:
  - "Collect [N] items" (N = 3-5)
  - "Defeat [N] enemies" (N = 3-8)
  - "Survive [N] seconds" (N = 60-180)
  - "Reach the dungeon" (special tile)
- On completion: award random stat +1, generate new goal

### World Updates (every 60 seconds)
- Add random terrain patch (3x3 area of new tile type)
- Scatter 3 new items on map
- Log "The world shifts..." in combat log
- Maybe spawn a wall segment or dungeon room

### Sound Effects (Web Audio API, synthesized)
```javascript
// All sounds generated procedurally via AudioContext
function playAttack() { /* 220hz sawtooth, 0.1s decay */ }
function playHit() { /* 110hz, noise burst, 0.15s */ }
function playPickup() { /* 440→880hz sine glide, 0.2s */ }
function playDeath() { /* 440→110hz descending, 0.5s */ }
function playSpawn() { /* sweep oscillator, 0.2s */ }
function playGoalComplete() { /* C4-E4-G4-C5 arpeggio, 0.5s */ }
function playStep() { /* white noise click, 0.05s */ }
function playSpecial() { /* class-specific unique sound */ }
```

### UI Layout
```
+--------------------------------------------------+
| RPG ARENA  | Players: N | Time: MM:SS | Score: N |
+--------+-----------------------------------+------+
| STATS  |                                   | LOG  |
| Class  |       ISOMETRIC CANVAS            |      |
| HP bar |                                   | NEAR |
| ATK: N |                                   | BY   |
| DEF: N |                                   |      |
| SPD: N |                                   |      |
| ENE: N |                                   |      |
| INT: N |                                   |      |
| CMD: N |                                   |      |
+--------+                                   +------+
| GOALS  |                                   |      |
| Goal 1 |                                   |      |
| Goal 2 |                                   |      |
| Goal 3 |                                   |      |
+--------+-----------------------------------+------+
|  [SPECIAL ABILITY - cooldown]    [WAIT]          |
+--------------------------------------------------+
```

### Character Creation Overlay
- Dark overlay on game
- Input field for character name
- "Create Character" button
- Shows what class/ability they'll get (randomly determined after clicking)

### Death/Spectator Mode
- On death: overlay shows "YOU DIED", final stats, score breakdown
- Game CONTINUES running underneath — AIs keep fighting
- "Play Again" button creates new character
- Can watch as spectator while dead

### Scoring
- kills * 10 + items_collected * 5 + goals_completed * 25 + seconds_survived / 10

### Code Structure Requirements
- Single index.html file, all CSS and JS embedded
- Use requestAnimationFrame for 60fps render loop
- Separate game logic tick (setInterval 500ms)
- Clean, commented code
- Handle edge cases (no AudioContext before user gesture, BroadcastChannel fallback for single player)

Make this as visually impressive and playable as possible. Use rich colors, smooth animations. The isometric world should feel alive.

After writing index.html, run: openclaw system event --text "RPG Arena game is ready at /Users/semitallc/Projects/rpg-arena/index.html" --mode now
