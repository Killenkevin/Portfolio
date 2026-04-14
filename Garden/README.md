# __Game Prototype__

![sunseed_banner](/PortfolioBilder/gardensettlers_day.png)



## _A Brief Project Description_
Together with my supervisor at inDirection Games, I have spent nearly six months developing a prototype for a game in which players build a civilization in a pixel-art landscape.

<br>

## _My Roles in the Project_

What I have worked on continuously / My role:

Developed the game prototype further by iterating on feedback regarding how the core mechanics function. One example is how the characters initially worked independently, but now collaborate when building and gathering resources, which enhances the player experience and makes the game feel more like a living village. Another core mechanic is the energy system: it was previously based on happiness, but has now been redesigned to use energy levels, creating a more realistic simulation of how the characters work, including sleep cycles and fatigue.

Worked on implementing traits: each character has random attributes that affect different aspects of gameplay. For example, Speedy increases movement speed, Night Owl allows characters to work longer hours, and Learner gains experience faster than others.

Created some pixel art, including simple icons and visual effects.

# Examples

### Trait effects
	

<details>
  <summary>Traits</summary>

#### The Idea
Peronalize each resident with different positive or/and negative effects

#### The Logic
When a resident moves into your village they get assigned a trait at random. Each level up grants an additional trait.

<br>

*Click the dropdown arrows below to see the `code`!* <br>

<details>
<summary>Show GnomeTrait.cs</summary>

 ```cs
namespace Gardens.Data;

public enum GnomeTrait
{
    None = 0,
    /// <summary>
    /// Eats more often.
    /// </summary>
    Foodie = 1,
    /// <summary>
    /// Earns more XP.
    /// </summary>
    Learner = 2,
    /// <summary>
    /// Increases speed a bit.
    /// </summary>
    Speedy = 3,
    /// <summary>
    /// Works more often.
    /// </summary>
    //Worker = 4,
    /// <summary>
    /// Works less often.
    /// </summary>
    Lazy = 4,
    /// <summary>
    /// Swims a lot faster.
    /// </summary>
    Swimmer = 5,
    /// <summary>
    /// Less fear effect.
    /// </summary>
    Calm = 6,
    Nightowl = 7,           // Not in effect
    /// <summary>
    /// Energy lasts longer.
    /// </summary>
    Energetic = 8,
    Max = Energetic,
}




```
</details>

<details>
<summary>Show TraitEffectService.cs</summary>

 ```cs
using Gardens.Data;
using Gardens.Domain.Game.Model.Entity;
using System;

namespace Gardens.Domain.Game.Service
{
    public sealed class TraitEffectService
    {
        public TraitEffectService()
        {
        }

        public float EnergyDrainMultiplier(WorldEntity keeper)
        {
            if (!keeper.IsGnome) return 1f;
            if (keeper.Traits.Contains(GnomeTrait.Lazy)) return 1.25f;
            return keeper.Traits.Contains(GnomeTrait.Energetic) ? 0.75f : 1f;
        }

        public float MovementSpeedMultiplier(WorldEntity keeper)
        {
            if (!keeper.IsGnome) return 1f;
            return keeper.Traits.Contains(GnomeTrait.Speedy) ? 1.2f : 1f;
        }

        public float WorkSpeedMultiplier(WorldEntity keeper)
        {
            if (!keeper.IsGnome) return 1f;
            if (keeper.Traits.Contains(GnomeTrait.Lazy))
            {
              /*  if (keeper.Traits.Contains(GnomeTrait.Worker))
                {
                    // Cancel out lazy
                    return 1f;
                } */
                return 0.75f;
            }
            /*if (keeper.Traits.Contains(GnomeTrait.Worker))
            {
                return 1.5f;
            } */
            return 1f;
        }

        public float HungerRateMultiplier(WorldEntity keeper)
        {
            if (!keeper.IsGnome) return 1f;
            return keeper.Traits.Contains(GnomeTrait.Foodie) ? 1.5f : 1f;
        }

        public float SwimSpeedMultiplier(WorldEntity keeper)
        {
            if (!keeper.IsGnome) return 1f;
            return keeper.Traits.Contains(GnomeTrait.Swimmer) ? 2.00f : 1f;
        }

        public float LearningMultiplier(WorldEntity keeper)
        {
            if (!keeper.IsGnome) return 1f;
            return keeper.Traits.Contains(GnomeTrait.Learner) ? 1.5f : 1f;
        }

        public float StressResistanceMultiplier(WorldEntity keeper)
        {
            if (!keeper.IsGnome) return 1f;
            return keeper.Traits.Contains(GnomeTrait.Calm) ? 0.5f : 1f;
        }

        public float FoodHappinessMultiplier(WorldEntity keeper)
        {
            if (!keeper.IsGnome) return 1f;
            return keeper.Traits.Contains(GnomeTrait.Foodie) ? 1.25f : 1f;
        }

        public bool IsActiveAtNight(WorldEntity keeper, int currentHour)
        {
            if (!keeper.IsGnome) return false;
            return keeper.Traits.Contains(GnomeTrait.Nightowl) && (currentHour >= 20 || currentHour <= 8);
        }

        // NOTE: Not used right now
        public TimeSpan ModifyTimer(WorldEntity keeper, TimeSpan baseTimer, TimerType type)
        {
            if (!keeper.IsGnome) return baseTimer;

            return type switch
            {
                _ => baseTimer
            };
        }

        public enum TimerType
        {
            Work,
            Sick,
            Home,
        }
    }
}


```
</details>

</details>




