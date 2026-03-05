# Changes to implement in index.html

Read the full file first. Then implement ALL of the following:

## 1. COMBAT SYSTEM FIX (most critical)
The melee combat formula must be exactly:
- Attacker rolls: ATK stat + d6()
- Defender rolls: DEF stat + d6()
- If attackRoll > defenderRoll: damage = attackRoll - defenderRoll (subtract from HP)
- If attackRoll <= defenderRoll: miss, 0 damage

Find performAttack() (or equivalent combat function) and enforce this formula.

ALLY PROTECTION:
- Entities on the same team NEVER attack each other
- Player NEVER attacks recruited allies (isAlly=true entities)
- Recruited allies NEVER attack player
- Check team membership before any attack: if attacker.team === target.team, skip
- Player's recruited allies have team set to player's team

UNWINNABLE FIGHT FIX:
- Add a combat cooldown per entity: after attacking, entity waits 1.5 seconds before attacking again
- If two entities have been fighting for more than 8 seconds with neither dying, one of them retreats (moves away for 3 seconds)
- Track entity.lastAttackTime and entity.combatStartTime

## 2. RANGED ATTACK CRASH FIX
In attemptRangedAttack() and the projectile update, the crash happens when target dies mid-flight. Fix:
- Before accessing p.target properties, always check: if (!p.target || !p.target.alive) { p.life = 0; continue; }
- In the projectile hit code, wrap everything in try/catch
- Also check that kills++ and checkGoals() won't throw if player is null
- Ranged key = F (already set), Recruit key = R (already set)

## 3. EDGE STUCK FIX
In moveEntitySmooth(), clamp entities to MAP_SIZE - 2 on all sides (not 1.5):
  entity.x = Math.max(1, Math.min(MAP_SIZE - 2, entity.x + ...))
  entity.y = Math.max(1, Math.min(MAP_SIZE - 2, entity.y + ...))

Also in the unstuck system: if entity is within 1 tile of any edge, push it toward center.

## 4. CASTLES (new feature)
Add 4-6 'castle' tiles or structures on the map. A castle is a 3x3 block of 'wall' tiles with a 'castle_core' tile at center (or just mark specific tiles as castle zones).

Better approach: Create an array `castles = []` where each castle has:
  { x, y, team: -1 (neutral), captureProgress: 0, captureTeam: -1, spawnTimer: 0, id }

A castle is captured when a single entity stands on its center tile (within 1 tile) for 3 continuous seconds (captureProgress counts up). When captured:
- castle.team = capturing entity's team
- addLog message: '[Name] has captured Castle #N for team [TEAMNAME]!'
- Spawn a new entity on that team at the castle location

Every 10 seconds, each captured castle spawns a new entity of its controlling team.
Visually: draw a flag/banner above the castle in the team's color. Neutral castles have a gray flag.
Draw castles as large 3x3 isometric structures with tower-like appearance.

## 5. ITEMS EVERY 10 SECONDS + AI GATHERS + DROP ON DEATH
- Every 10 seconds (in gameTick), scatter 2-3 new items on random walkable tiles
- AI pathfinding: if no enemies nearby (>5 tiles), AI moves toward nearest item if within 6 tiles
- Each entity has inventory: entity.inventory = [] (array of item objects)
- When AI picks up item, push to entity.inventory, apply stat boost
- When entity dies: drop all inventory items at death location (push back to items array)
- Max inventory per entity: 3 items

## 6. FOOD ITEMS
- New item type: food items that restore 1 HP when picked up (instead of stat boost)
- Food items: '🍎 Apple', '🍖 Meat', '🍞 Bread', '🧀 Cheese', '🍇 Grapes'
- Food items appear on or adjacent to 'forest' tiles (trees)
- Spawn 3-5 food items during map generation near forest areas
- Every 15 seconds, spawn 1-2 more food items near forest tiles
- When picked up: restore 1 HP (not above maxHp), show '+1 HP' floating text in green

## 7. WATER DAMAGE
- In gameTick (every 500ms = 0.5s), check if player is standing on water or lava tile
- Water: deal 0.5 damage per tick (= 1 HP/second)  
- Lava: deal 1 damage per tick (= 2 HP/second)
- Show red floating text '-0.5' or '-1' above player when taking water/lava damage
- AI entities also take water/lava damage (same rate) - this encourages them to move off water
- Play a soft hurt sound when taking water damage

## 8. FLYING MOBS
Create flying mob types. A flying entity has entity.flying = true.

Flying mob emoji pool: ['🦅 Eagle Rider','🐦 Storm Raven','🦉 Night Owl','🦋 Chaos Moth','🐝 Killer Bee','🦆 War Duck','🕊️ Death Dove','🦜 Hex Parrot','🐛 Sky Wyrm','🦗 Blade Locust']

Flying behavior:
- IGNORE all terrain collision (walls, water, lava - they fly over everything)
- Movement pattern: sinusoidal/swooping flight path
  entity.flyPhase = random angle, entity.flyAmplitude = 0.5 + random()*1.5
  dx/dy includes: Math.sin(entity.flyPhase) * entity.flyAmplitude as a perpendicular oscillation
  entity.flyPhase += dt * (0.5 + entity.stats.cmd * 0.3) // CMD controls oscillation speed
- ATK stat controls turn speed: how fast they can redirect toward target
  entity.flightAngle = current direction angle
  Turn rate = entity.stats.atk * 45 degrees/second (max 270 deg/s at ATK 6)
- CMD stat controls acceleration: how fast they speed up
- Flying entities move 20% faster than ground (speed mult = 1.2)

Visually: draw flying entities slightly higher (offset sy by -25px), add shadow circle on ground below them, draw wings (two arc shapes on sides of entity).

10% of newly spawned AI should be flying types.

## 9. END-GAME NARRATIVE GENERATOR
When the player dies (and after the death overlay appears), generate a narrative paragraph.

Track these game events in an array: `gameEvents = []`
Each event: { time, type, actor, target, detail }
Event types: 'kill', 'castle_captured', 'item_collected', 'recruit', 'ranged_kill', 'player_death', 'team_wipe'

When player dies, call generateNarrative() which:
1. Sorts events by time
2. Picks 5-8 most dramatic events (kills, castle captures, dramatic moments)
3. Uses template strings to compose a paragraph like:
   "The battle of the arena began with [player] the [class] joining the fray for team [TEAM]. 
    In the early moments, [dramatic event 1]. [Event 2]. [Event 3]. 
    After [time] seconds of brutal combat, [player] fell to [killer]. 
    As the dust settled, team [WINNER] stood victorious with [N] warriors still standing."

Display this narrative in the death overlay, styled nicely with a scroll/parchment look.
Add a 'COPY NARRATIVE' button that copies it to clipboard.

## 10. MORE ITEM SPAWNS IN WORLD
Besides food under trees, ensure:
- Items appear on road tiles occasionally (merchants dropped them)
- Rare 'golden' items appear: boost stat by 2 instead of 1, shown with ✨ prefix
- Item variety: add 'experience tome' items that boost ALL stats by 0.5 (round up) shown as 📚

## 11. UI IMPROVEMENTS
- In left panel after class name, show: 'Team: [banner emoji] [TEAMNAME]' in team color
- In top bar, show team kill counts as small colored numbers
- Show water damage warning: flash red vignette on screen edges when standing in water/lava
- Floating text for water damage should be in blue (water) or orange (lava)

## IMPLEMENTATION NOTES
- Keep all existing systems (ranged F key, recruit R key, healing every 6 ticks, smooth movement, terrain speed multipliers, viewCX/viewCY stable camera)
- Preserve the 4-team system if already implemented, otherwise add it
- The unstuck system should also work for flying entities (they can't get stuck on terrain, but can get stuck in corners; for flying just pick random direction)
- Test that recruit (R key) still works: CMD+d6 vs enemy INT+d6, success = joins player team

After all changes, commit and push:
cd /Users/semitallc/Projects/rpg-arena && git add -A && git commit -m 'Castles, flying mobs, water damage, food, narrative, combat fix, edge fix' && git push

When completely done: openclaw system event --text 'RPG Arena v3 done: castles/flying/narrative/water damage/food' --mode now
