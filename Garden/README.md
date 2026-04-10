# __Game Prototype__

![sunseed_banner](/PortfolioBilder/gardensettlers_day.png)



## _A Brief Project Description_
Together with my supervisor at inDirection Games, I have spent nearly six months developing a prototype for a game in which players build a civilization in a pixel-art landscape.

[Webpage]((https://indirectiongames.com)) <br>

## _My Roles in the Project_

What I have worked on continuously / My role:

Developed the game prototype further by iterating on feedback regarding how the core mechanics function. One example is how the characters initially worked independently, but now collaborate when building and gathering resources, which enhances the player experience and makes the game feel more like a living village. Another core mechanic is the energy system: it was previously based on happiness, but has now been redesigned to use energy levels, creating a more realistic simulation of how the characters work, including sleep cycles and fatigue.

Worked on implementing traits: each character has random attributes that affect different aspects of gameplay. For example, Speedy increases movement speed, Night Owl allows characters to work longer hours, and Learner gains experience faster than others.

Created some pixel art, including simple icons and visual effects.

# Contributions 

### Traits
	
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="/PortfolioGifs/Frog.gif" alt="juice1" width="800" height="auto"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; |
|:---:|


> *In the GIFs, you can see the frog grabbing an enemy and eating it.*

<details>
  <summary>Frog!</summary>

#### The Idea
The concept was to make an objective that assists you in combat.

#### The Logic
To have the frog assist you in combat you have to water it which is a core mechanic in Sun Seed. After you have watered it to the required number it starts shooting out its tounge at the nearest "Enemy" tagged object. The frog launches its toungue as a linerenderer, grabs enemy, eats it (destroys enemy) and it resets the timer.

<br>

*Click the dropdown arrows below to see the `code`!* <br>

<details>
<summary>Show GnomeTrait.cs</summary>

 ```cs
namespace Gardens.Data;

public enum GnomeTrait
{
    None = 0,
    Foodie = 1,    // Eats more often.
    Learner = 2,   // Earns more XP.
    Speedy = 3,    // Increases speed.
    // Worker = 4, // Works more often.
    Lazy = 4,      // Works less often.                
    Swimmer = 5,   // Swims a lot faster.            
    Calm = 6,      // Less fear effect.
    Nightowl = 7,  // Stays up longer.
    Energetic = 8, // Energy lasts longer.
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




