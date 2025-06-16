============================================================
        DYNAMIC HUNGER SYSTEM - DOCUMENTATION
============================================================

This document explains how the automatic hunger drain system
works in your project. It is designed to be a reference
for adjusting and tuning the gameplay feel.


------------------------------------------------------------
    SECTION 1: HOW IT WORKS - THE ENGINE
------------------------------------------------------------

The system is built on a simple but powerful concept using
a single PARALLEL common event.

Think of it like a clock:

1. THE TIMER:
   A variable (#62 HungerTimer) counts up by 1 on every
   single frame of the game. Since the game runs at 60
   frames per second, this is our stopwatch.

2. THE TRIGGER:
   The system constantly asks one question using a
   CONDITIONAL BRANCH:
   
   "Has the Timer reached the target Interval yet?"

   As soon as the answer is "YES", it does two things:
   - It runs the hunger drain script.
   - It IMMEDIATELY resets the Timer back to 0.

This is why the system is efficient and doesn't lag the
game. The complex script only runs for a single frame
every few seconds, not constantly.


------------------------------------------------------------
    SECTION 2: HOW TO ADJUST PARAMETERS
------------------------------------------------------------

To change how the hunger system feels, you only need to
change the values of these specific Variables and Switches.
You can do this at the start of the game or even during
gameplay for dynamic effects.

--- Core Gameplay Loop ---

  VARIABLE #61: HungerDropInterval
    - What it does: This is the most important setting. It
      controls HOW OFTEN hunger drains.
    - How to Adjust: The value is in frames. To get the
      number of seconds, use this formula:
      (Number of Seconds) * 60 = Value
      
      EXAMPLES:
      - For a drop every 5 seconds: 5 * 60 = 300. Set Var 61 to 300.
      - For a drop every 30 seconds: 30 * 60 = 1800. Set Var 61 to 1800.

  VARIABLE #63: BaseHungerDrop
    - What it does: This controls HOW MUCH hunger is lost
      per tick, before any modifiers are applied.
    - How to Adjust: A value of 1 or 2 is a good starting
      point. If you want hunger to drain faster overall,
      increase this number.
      
      *** IMPORTANT: If this is 0, hunger will NEVER drain. ***


--- Food and Buffs ---

  VARIABLE #64: SatietyLevel
    - What it does: Represents the "quality" of the last
      food eaten. Higher satiety slows down hunger drain.
    - How to Adjust: When the player eats food, set this
      variable. For example:
      - A simple apple might set Satiety to 20.
      - A hearty stew might set Satiety to 80.
      The system will handle the rest. This value should
      probably decrease over time with another, slower timer.

  SWITCH #61: WellFedBuff
    - What it does: A simple ON/OFF buff. When ON, it cuts
      the hunger drain rate in half.
    - How to Adjust: Turn this Switch ON when the player
      eats a very high-quality meal or uses a special item.
      Remember to turn it OFF when the buff duration ends.

  SWITCH #62: PoisonedDebuff
    - What it does: A simple ON/OFF debuff. When ON, it
      increases hunger drain by 50%.
    - How to Adjust: Turn this Switch ON when the player
      is hit by a poison effect or eats bad food.


--- States ---

  STATE #61: Starving
    - What it does: This is a status effect on the actor.
      If the party leader has this state, hunger drain
      doubles. This creates a dangerous "death spiral"
      that the player needs to escape.
    - How to Adjust: This is controlled by your
      VariableTriggerSystem plugin, which should apply
      this state when Hunger (Var 56) reaches 0.


--- System Variables (Do Not Touch) ---

  VARIABLE #56: HungerCurrent
    - This is the player's current hunger. The system
      manages this. You should only change it when the
      player eats food (to increase it).

  VARIABLE #62: HungerTimer
    - The system's internal stopwatch. Leave this alone.


------------------------------------------------------------
    SECTION 3: QUICK TROUBLESHOOTING
------------------------------------------------------------

If the system ever behaves strangely, check these two
common issues first.

PROBLEM: Hunger drains instantly to zero.
  - CAUSE: The timer is not resetting.
  - SOLUTION: Open the parallel common event. Make sure the
    `Control Variables: #0062 HungerTimer = 0` command is
    INSIDE the Conditional Branch, right after the Script.

PROBLEM: Hunger isn't draining at all.
  - CAUSE: The drain calculation is resulting in zero.
  - SOLUTION: Check these three things:
    1. Is the master switch for the parallel event ON?
    2. Is Variable #63 (BaseHungerDrop) greater than 0?
    3. Is Variable #61 (HungerDropInterval) not set to a
       super high number?
	   
	   
------------------------------------------------------------
    SECTION 4: HOW TO EDIT THE SCRIPT
------------------------------------------------------------

If you ever need to change which Variables, Switches, or
States this system uses, you will need to edit the
multi-line SCRIPT command inside the parallel common event.

Below is a map of the script, showing exactly which number
corresponds to which game mechanic.


--- SCRIPT MAP ---

To edit, open the `SYS: Hunger System` common event, find
the `Script...` command, and change the numbers as needed.

Original Script Line:
const currentHunger = $gameVariables.value(56);
  - PURPOSE: Your main Hunger variable.
  - EDIT: Change the `56` to your new Hunger variable ID.

Original Script Line:
const baseDrop = $gameVariables.value(63);
  - PURPOSE: The base amount to drop per tick.
  - EDIT: Change the `63` to your new Base Drop variable ID.

Original Script Line:
const satiety = $gameVariables.value(64);
  - PURPOSE: The player's Satiety level from food.
  - EDIT: Change the `64` to your new Satiety variable ID.

Original Script Line:
const wellFed = $gameSwitches.value(61);
  - PURPOSE: The "Well Fed" buff switch.
  - EDIT: Change the `61` to your new buff switch ID.

Original Script Line:
const poisoned = $gameSwitches.value(62);
  - PURPOSE: The "Poisoned" debuff switch.
  - EDIT: Change the `62` to your new debuff switch ID.

Original Script Line:
const starving = $gameParty.leader().isStateAffected(61);
  - PURPOSE: The "Starving" status effect.
  - EDIT: Change the `61` to your new Starving state ID.

Original Script Line:
$gameVariables.setValue(56, newHunger);
  - PURPOSE: This is the final step where the script sets
    the new hunger value.
  - EDIT: This number MUST match the Hunger variable ID you
    set at the very top of the script. Change this `56` to
    your new Hunger variable ID.


--- ADVANCED: Changing Modifier Strength ---

You can also edit the script to change how powerful the
modifiers are. These are found in the "Calculation"
section of the script.

To edit Satiety strength:
  - Find the line: const satietyModifier = 1 - (satiety / 200);
  - The `200` controls the power.
    - A HIGHER number (e.g., 400) makes satiety WEAKER.
    - A LOWER number (e.g., 150) makes satiety STRONGER.

To edit Buff/Debuff strength:
  - Find lines like: if (wellFed) { effectModifier *= 0.5; }
  - The `0.5` is a 50% reduction. Change this to `0.75` for
    a weaker 25% reduction.
  - For the poison line `*= 1.5`, this is a 50% increase.
    Change it to `*= 2.0` to make it double the drain rate.

Always be careful when editing the script, and only change
the numbers unless you are comfortable with Javascript.