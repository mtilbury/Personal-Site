---
layout: post
author: Michael Tilbury
title: Pre-Alpha Final Sprint
snippet: The main tasks I worked on this sprint were creating a projectile component for the Gungnir enemy and writing a component to handle enemy death. I also fixed a few bugs that popped up from my work last sprint, wrote documentation for the EnemyAggro script, and got to do some thorough playtesting.
---
## At a Glance
The main tasks I tackled this sprint were the creation of a component that fires projectiles for a specific new enemy, as well as a more generic component that will handle enemy deaths. Aside from these tasks, I also got to work on writing some documentation, fixing somg bugs, and submitting some playtesting feedback based on my experience playing the latest build.
* Shield Enemy Prototype was merged into the game
  * Moved out of _Development and renamed
* Created PrototypeRegularProjectile script
  * Fires projectiles with specified direction, speed, and periodicity
  * Will be used for the Gungnir enemy
* Fix EnemyAggro range bug
  * Refactored EnemyAggro into a prefab
  * Wrote confluence documentation on its new usage
* Submitted detailed playtest feedback
  * I had a lot of feedback based on my experience playing the latest build regarding bugs and potential UX improvements
* Wrote enemy death script
  * Runs when any enemy dies. Despawns it, shows a corpse, and plays a particle effect