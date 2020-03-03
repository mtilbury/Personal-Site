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

## Projectile Component
My goal was to create a component for instantiating projectiles and firing them off at regular intervals. I was specifically assigned this task to be used for the Gungnir enemy, but I figured I should make the script as generic as possible so it could be reused on other enemies/obstacles that use projectiles. I read the Confluence documentation for the accepted enemy designs so I know that they can probably reuse my code. After reading the Gungnir design spec, I spoke with Nikhil about which mechanics from the enemy I would be implementing. He said that it was really only the projectile component. It seemed like a lot of the details from the spec (e.g. the Gungnir stops firing when the player approaches) weren't going to be implemented—at least not in this sprint.

<div class="image-container text-center">
<video width="400" style="margin: none" alt="Three enemies shooting basic projectiles at regular intervals." loop autoplay>
  <source src="/assets/images/preAlphaFinal/HorizontalProjectiles.mp4" type="video/mp4">
</video>
<p class="figure-caption">Three test enemies fire projectiles at regular intervals.</p>
</div>

Based on what I had learned last sprint, I used Editor Attributes and serialized properties to keep the code easy to understand and use. There are fields for `projectilePrefab`, `projectileDirection`, `projectileSpeed`, and `firingInterval`. I used a coroutine to handle the cooldown. Everything went smoothly and the code was merged without issue.

<div class="image-container text-center">
<video width="400" style="margin: none" alt="A Gungnir fires projectiles and jumps to match the player's height." loop autoplay>
  <source src="/assets/images/preAlphaFinal/Gungnir.mp4" type="video/mp4">
</video>
<p class="figure-caption">A Gungnir shoots projectiles and jumps to match the player's height.</p>
</div>

## Fixed EnemyAggro Bug
I wrote the `EnemyAggro` component last sprint, but I realized that a pretty serious bug had made its way through the code review. If the player used their sword anywhere within the enemy's `EnemyAggro` collider/trigger, the enemy would take damage. You could kill every enemy without even approaching them. As soon as I discovered the bug, I had a good idea of what was causing it. The `EnemyAggro` component adds a `CircleCollider2D` trigger to the enemy. However, the enemy also already has a `BoxCollider2D` trigger which acts as its hitbox. The code that determines when an enemy takes damage is located in `PrototypeEnemy.cs` and uses `OnTriggerStay2D` and `CompareTag("PlayerAttack")`. The EnemyAggro collider essentially just expands the enemy's hitbox.

<div class="image-container text-center">
<video width="400" style="margin: none" alt="The player hits an enemy with their sword despite not making visual contact." loop autoplay>
  <source src="/assets/images/preAlphaFinal/AggroBug.mp4" type="video/mp4">
</video>
<p class="figure-caption">The player hits an enemy with their sword despite not making visual contact.</p>
</div>

While I was confident I had found the source of the bug, fixing it proved more challenging. My first thought was to move the `EnemyAggro` component onto a child of the enemy's GameObject. Each enemy already has a child which stores its visual components, so I decided to move the `EnemyAggro` component there. However, the bug still persisted. I understood that the enemy was checking if a PlayerWeapon-tagged object entered a trigger on the enemy, but now the larger aggro trigger wasn't on the enemy—it was on a child of the enemy. Why was it still counting as part of the enemy's hitbox?
<div>
<code class="highlight" style="margin-left: auto; margin-right: auto; width: 200px; display: block">
<pre>
LAB_EnemyPrototype
├── LAB_Enemy (Patrol)
│   └── Visuals
├── Left Bound
└── Right Bound
</pre>
</code>
<p class="figure-caption text-center" style="max-width: 600px; margin-left: auto; margin-right: auto">An enemy's hierarchical structure. The EnemyAggro component was put on the Visuals GameObject, but the bug persisted.</p>
</div>

I found a helpful post on the Unity forums from someone having the exact same issue I was having: [Trigger in child object calls OnTriggerEnter in parent object](https://answers.unity.com/questions/410711/trigger-in-child-object-calls-ontriggerenter-in-pa.html). It pointed me to the [RigidBody documentation](https://docs.unity3d.com/Manual/class-Rigidbody.html), specifically the portion describing "Compound Colliders." Colliders on child GameObjects actually contribute to the parent's collider. This allows for approximating complicated and expensive mesh colliders. It's an interesting use case, but it wasn't what I wanted for the enemy; the aggro range has to be completely separate from its hitbox. 

<div>
<code class="highlight" style="margin-left: auto; margin-right: auto; width: 200px; display: block">
<pre>
LAB_EnemyPrototype
├── LAB_Enemy (Patrol)
│   └── Visuals
├── EnemyAggroRange
├── Left Bound
└── Right Bound
</pre>
</code>
<p class="figure-caption text-center" style="max-width: 600px; margin-left: auto; margin-right: auto">The new enemy hierarchy includes an EnemyAggroRange object. Since it's not a child of the LAB_Enemy, it does not merely contribute to its hitbox collider.</p>
</div>

One answer on the forum post suggested putting the collider on a sibling object rather than a child object. Then, a follow script could be used to update the AggroRange's position to the enemy's each frame. This worked, but it wasn't the elegant solution I was hoping for. Reorganzing the `EnemyAggroRange` into its own prefab and adding an `EnemyToFollow` property would break existing code. I knew that other people were already using my aggro script in their work, so this was something I was trying to avoid. Nevertheless, I couldn't find a way to fix the bug without breaking the existing interface I provided so I had to commit the changes.

<div class="image-container text-center">
<video width="400" style="margin: none" alt="The player swings their sword within the enemy's aggro radius, but they do not take damage." loop autoplay>
  <source src="/assets/images/preAlphaFinal/AggroFix.mp4" type="video/mp4">
</video>
<p class="figure-caption">The enemy no longer takes damage when the player uses the sword within the aggro radius.</p>
</div>

#### Confluence Documentation

## Playtest Feedback
At the February 22<sup>nd</sup> studio meeting, a survey was released for any playtesting feedback we had. While this isn't directly related to the programming I did this sprint, I spent a lot of time writing my thoughts on the current build of the game and recording gameplay footage to support my thoughts. 

## Enemy Death Component