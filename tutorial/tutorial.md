# Mlog tutorial outline

Strictly V8. Should be accessible even to people who are new to programming.

## Mlog basics

* Editing code
  * In-game vs. out-of-game 
    * Labels for jumps
    * Comments
  * Literals
    * Numbers, including limitations
    * Strings
  * Variables
* Basic control structures
  * Instruction flow, implicit loops
  * Conditional execution
    * Jump conditions
  * Loops
  * `select`
* Printing
  * Coloring (`[red]`)
  * Icons (`print ":mega:"`)
  * `printchar`
  * `format`
  * `printflush`
* `op` instruction
  * Basic operations
  * Double <-> long conversion, 53 bits of storage

# Blocks

* Linked blocks
  * Named links
  * `getlink`
  * Dynamic linking
* External memory
  * Indexed access: `read`/`write`
  * Numerical values
  * Object/number conversion
  * `lookup` and `sensor @id`
* Controlling blocks
  * `enabled`
    * Nonstandard effects (sorters etc)
  * `config`, `color`
  * `shoot`, `shootp`: later, when discussing attack/defence logic
* `sensor` basics

# Units

* Binding units
  * Single unit, no flags   
  * `@unit`
  * storing and reusing unit references
* Moving units
  * `move`
  * `approach`, `within`
  * waiting for units to arrive
* Mining and moving items
* Blocks: `getBlock`, `build`, `deconstruct`
* Payload

# Functions

# Arrays

# Drawing

* Displays
  * Display transformations
* Colors: literals, built-ins, `packcolor`, `unpackcolor`
* Text printing

# Attack/defense logic

Radar, shoot, shootp

# User input

Using turrets to read player input 

# Processor access

# Data strings
