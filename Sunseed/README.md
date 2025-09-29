# __Vesper__

![vesper_banner](/_Images/vesper_thumb_16_9_v2.png)

```
Developed:  09/2023 - 10/2023
Duration:   7 weeks
Engine:     Unity
Genre:      Skill-based Puzzle Platformer
Team:       3 Programmers, 3 Artist
```

<table>
  <tr>
    <td width="50%"><img src="/_Images/vesper (2).jpg" /></td>
    <td width="50%"><img src="/_Images/vesper (1).jpg" /></td>
  </tr>
  <tr>
    <td width="50%"><img src="/_Images/vesper (3).jpg" /></td>
    <td width="50%"><img src="/_Images/vesper (5).jpg" /></td>
  </tr>
</table>

## _A Brief Game Description_
Vesper, a skill-based puzzle platformer. Play as a formless celestial being that has been trapped in a physical form. Use the ability to change size and find your way home!

[Webpage (itch.io)](https://yrgo-game-creator.itch.io/vesper) <br>
[Trailer](https://youtu.be/9p0B5Zs0Df8)

## _My Roles in the Project_

I co-developed the player controls and mechanics alongside the other programmers, contributed to adding polish and “juice” to enhance the gameplay experience, and took on the role of Scrum Master throughout the project.

My most significant contribution was the development of the majority of the interactive elements within the game world. This was made possible through close collaboration with the game designer and level designer, working together to brainstorm ideas and to bring their vision to life. This collaboration also extended beyond gameplay elements, allowing me to assist with animations, cutscenes, and VFX.

Below you’ll find a list of a few features I created and implemented myself, along with summaries of my code.

# Contributions 

### Juice
	
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="/_GIFs/VesperSquishSquash.gif" alt="juice1" width="300" height="auto"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="/_GIFs/VesperSquishSquash1.gif" alt="juice1" width="300" height="auto"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <br> |
|:---:|


> *In the GIFs, the cube's particle systems have been disabled to highlight the SquishAndSquash.*

<details>
  <summary>Give me the juice!</summary>

#### The Idea
The concept was to make the cube's shape respond dynamically to its environment and movement.

#### The Logic
The squish-squash mechanic works by temporarily scaling the object in a squashing (wider, shorter) or stretching (narrower, taller) manner using Lerp, and then smoothly reverting it back to its original size.
The mechanic relies on a raycast system along with calculations of delta positions, as well as the `Jump()` and `LandingActions()` methods, to determine when to activate the effect.

<br>

*Click the dropdown arrows below to see the `code`!* <br>

<details>
<summary>Show SquishAndSquash.cs</summary>

 ```cs
public class SquishAndSquash : MonoBehaviour
{
    public float squashAmount = 0.2f;
    public float stretchAmount = 0.2f;

    public float squishSquashDuration = 0.2f;
    public float revertScaleDuration = 0.1f;

    private Vector3 originalScale;

    public bool squishAndSquashEnabled;

    void Start()
    {
        squishAndSquashEnabled = true;
        originalScale = transform.localScale;
    }

    public void JumpSquash()
    {
        if (!squishAndSquashEnabled) return;
        StartCoroutine(SquishSquashOverTime(originalScale.x - stretchAmount, originalScale.y + stretchAmount));
    }

    public void LandSquish()
    {
        if (!squishAndSquashEnabled) return;
        StartCoroutine(SquishSquashOverTime(originalScale.x + squashAmount, originalScale.y - squashAmount));
    }

    public void ToggleEnabled(bool boolean)
    {
        squishAndSquashEnabled = boolean;
    }

    IEnumerator SquishSquashOverTime(float targetX, float targetY)
    {
        Vector3 originalSize = transform.localScale;
        Vector3 targetSize = new Vector3(targetX, targetY, originalSize.z);

        float currentTime = 0.0f;

        while (currentTime <= squishSquashDuration)
        {
            transform.localScale = Vector3.Lerp(originalSize, targetSize, currentTime / squishSquashDuration);
            currentTime += Time.deltaTime;
            yield return null;
        }

        transform.localScale = targetSize;

        // Revert back to original size
        StartCoroutine(RevertOverTime(targetSize.x, targetSize.y, originalScale.x, originalScale.y));
    }

    IEnumerator RevertOverTime(float startX, float startY, float targetX, float targetY)
    {
        Vector3 startSize = new Vector3(startX, startY, transform.localScale.z);
        Vector3 targetSize = new Vector3(targetX, targetY, transform.localScale.z);

        float currentTime = 0.0f;

        while (currentTime <= revertScaleDuration)
        {
            transform.localScale = Vector3.Lerp(startSize, targetSize, currentTime / revertScaleDuration);
            currentTime += Time.deltaTime;
            yield return null;
        }

        transform.localScale = targetSize;
    }
}

```
</details>
<details>
  <summary>Show PlayerController.cs</summary>
  
```cs

public class PlayerController : MonoBehaviour, IReset
{

# ...

    // Irrelevant code in the following method is omitted for brevity
    void Jump()
    {
        // Disable WallSquash
        if (WallSquashEnabler != null)
        {
            StopCoroutine(WallSquashEnabler);
        }

        squishAndSquash.JumpSquash();
    }

# ...

    void WallCollisionSquash()
    {
        if (!canMove || !wallCollisionSquash) return;

        if (Math.Abs(transform.position.x - prevPos.x) > deltaPosThreshold &&
            (!rayCastHandler.rightSide || !rayCastHandler.leftSide))
        {
            if ((prevRaycastLeft && !rayCastHandler.leftSide && rb.velocity.x < 0) ||
                (prevRaycastRight && !rayCastHandler.rightSide && rb.velocity.x > 0))
            {
                squishAndSquash.JumpSquash();
            }
        }

        prevRaycastRight = rayCastHandler.rightSide;
        prevRaycastLeft = rayCastHandler.leftSide;
        prevPos = transform.position;
    }

# ...

    // Irrelevant code in the following method is omitted for brevity
    private void LandingActions()
    {
        squishAndSquash.LandSquish();
    }

# ...

}
```

</details>

</details>

<br>

### Cutscene & VFX

|<img src="/_GIFs/VesperCutscene.gif?version=2" width="80%" />|
|---|

<details>
<summary>Show Cutscene & VFX</summary>

#### The Idea
The concept was to transition into a cinematic when the player encountered and picked up a power-up.

#### Cinematic logic
When the player picks up the power-up, control over movement is temporarily disabled, and the player character moves to a designated cutscene position. A list of GameObjects, including the player and vignettes, is triggered to play their respective animations. After a delay, the animations stop, and player control is restored.

#### Outline VFX Logic
The OutlineFxTrigger class spawns one or more outline GameObjects at a specified interval, while the OutlineFx class handles the growth and fading of each outline effect.
> *The VFX referenced here is only the outline effect that expands, the rest of the VFX are particles systems activated on trigger.*

<br>

*Click the dropdown arrows below to see the `code`!* <br>

<details>
<summary>Show PowerUp.cs</summary>
  
```cs
public class PowerUp : MonoBehaviour
{
    [Header("Power-Up")]
    public bool enableLargeSize;
    public bool enableSmallSize;

    [Header("Sprite Effects")]
    public bool spriteFade = true;
    public bool outlineFx = true;

    [Header("Movement")]
    public bool stopPlayerMovement = true;
    public bool movePlayerToPosition = true;
    public Transform targetPosition;
    public float moveDuration;

    [Header("Animation")]
    public bool playPlayerAnimation = true;
    public bool playObjectsAnimation = true;
    public float animationDuration;
    public List<GameObject> animationObjects;
    
    [Header("Puls")]
    [Range(0, 2)] public float sizeMulti = 1.02f;
    private Vector3 origiScale;
    private float pulsDuration = 1f;
    private float pulsingTimer = 0f;
    public float beatingDuration = 1f;

    // Private variables
    private bool cutscenePlayed = false;
    private bool isSizeChange = false;
    
    // References
    private FadeSprite fadeSprite;
    private OutlineFxTrigger outlineFxTrigger;
    private Rigidbody2D rb2d;

    void Start()
    {
        origiScale = transform.localScale;
        fadeSprite = GetComponent<FadeSprite>();
        outlineFxTrigger = GetComponent<OutlineFxTrigger>();
        rb2d = PlayerController.player.GetComponent<Rigidbody2D>();
    }

    void Update()
    {
        pulsingTimer += Time.deltaTime;

        if (pulsingTimer >= pulsDuration && !isSizeChange)
        {
            isSizeChange = true;
            Pulsing();
            pulsingTimer = 0f;
        }
    }

    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.CompareTag("Player") && !cutscenePlayed)
        {
            if (gameObject.CompareTag("GetSmaller"))
            {
                AudioManager.Instance.GameplaySFX(AudioManager.Instance.powerUpSmallSound, AudioManager.Instance.powerUpSmallVolume);
            }
            else if (gameObject.CompareTag("GetLarger"))
            {
                AudioManager.Instance.GameplaySFX(AudioManager.Instance.powerUpLargeSound, AudioManager.Instance.powerUpLargeVolume);
            }

            SpriteFade();
            OutlineFx();
            StopPlayerMovement();
            MovePlayerToCutscenePosition();
            PlayObjectsAnimations(true);
            PlayPlayerAnimation(true);

            cutscenePlayed = true;

            StartCoroutine(UnstopMovementAndStopAnimations());
        }
    }

    void SpriteFade()
    {
        if (!spriteFade) return;
        fadeSprite.FadeOut();
    }

    void OutlineFx()
    {
        if (!outlineFx) return;
        outlineFxTrigger.PlayFx();
    }

    void StopPlayerMovement()
    {
        if (!stopPlayerMovement) return;

        rb2d.velocity = Vector2.zero;
        PlayerController.instance.canMove = false;
        PlayerController.instance.bigEnabled = false;
        PlayerController.instance.smallEnabled = false;
    }

    IEnumerator UnstopMovementAndStopAnimations()
    {
        yield return new WaitForSeconds(animationDuration);
        PlayerController.instance.canMove = true;
        PlayObjectsAnimations(false);
        PlayPlayerAnimation(false);

        EnableSize();
    }

    public void Pulsing()
    {
        var newScale = origiScale * sizeMulti;
        Sequence sizeSeq = DOTween.Sequence();
        sizeSeq.Append(transform.DOScale(newScale, beatingDuration / 2).SetEase(Ease.InSine))
               .Append(transform.DOScale(origiScale, beatingDuration / 2).SetEase(Ease.InSine))
               .OnComplete(() => isSizeChange = false);
    }

    private void EnableSize()
    {
        PlayerController.instance.bigEnabled = enableLargeSize;
        PlayerController.instance.smallEnabled = enableSmallSize;
    }

    void MovePlayerToCutscenePosition()
    {
        if (!movePlayerToPosition) return;

        rb2d.transform.DOMove(targetPosition.position, moveDuration);
    }

    void PlayObjectsAnimations(bool boolean)
    {
        if (!playObjectsAnimation) return;

        // Enable animationObjects and their Animator
        foreach (GameObject animationObject in animationObjects)
        {
            animationObject.SetActive(boolean);
            Animator animator = animationObject.GetComponent<Animator>();

            if (animator != null)
            {
                animator.enabled = boolean;
            }
        }
    }

    void PlayPlayerAnimation(bool boolean)
    {
        if (!playPlayerAnimation) return;

        // Enable Player Animator
        Animator playerAnimator = PlayerController.player.GetComponent<Animator>();
        playerAnimator.enabled = boolean;
    }
}

```
</details>

<details>
<summary>Show OutlineFxTrigger.cs</summary>
  
```cs
public class OutlineFxTrigger : MonoBehaviour
{
    public float outlineSpawnDelay = 0.3f;
    public int numberOfOutlines = 1;
    public GameObject outline;

    public void PlayFx()
    {
        StartCoroutine(TriggerOutlineFx());
    }
    IEnumerator TriggerOutlineFx()
    {
        for (int i = 0; i < numberOfOutlines; i++)
        {
            Instantiate(outline, transform.position, Quaternion.identity);
            yield return new WaitForSeconds(outlineSpawnDelay);
        }
    }
}
```
</details>


<details>
<summary>Show OutlineFx.cs</summary>
  
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using DG.Tweening;

public class OutlineFx : MonoBehaviour
{
    public float growDuration;
    public float targetScale;
    public float fadeDelay;
    public Ease ease;
    bool playing;

    private void Start()
    {
        TriggerEffect();
    }
    
    private void Update()
    {
        if (!playing)
        {
            TriggerEffect();
        }
    }

    void TriggerEffect()
    {
        playing = true;
        transform.DOScale(targetScale, growDuration).SetEase(ease);
        StartCoroutine(TriggerFade());
    }

    IEnumerator TriggerFade()
    {
        yield return new WaitForSeconds(fadeDelay);
        GetComponent<FadeSprite>().FadeOut();
    }
}

```
</details>

</details>

<br>

### Trampoline

|<img src="/_GIFs/VesperTrampoline.gif" width="80%" />|
|---|
<details>
<summary>Show Trampoline</summary>

#### The Idea
The aim was to create a trampoline that provided a satisfying bounce that reacted to the player's size. We wanted to encourage the player to use the size-switch mechanic strategically to achieve the best possible bounce.

#### The Logic 
The trampoline applies a bounce force to the player on collision, with the force size determined by the player's gravity scale or Y-axis velocity. 
Seeing as the player's gravity scale changes with their size, using the gravity scale as a parameter for the bounce force worked pretty well.

Player movement is temporarily disabled to ensure a smoother bounce in the correct direction, and to prevent unwanted physics issues like double jumps or other glitches.

<br>

*Click the dropdown arrows below to see the `code`!* <br>

<details>
<summary>Show Trampoline.cs</summary>
  
```cs
public class Trampoline : MonoBehaviour
{
    [Header("Push mode")]
    public bool usingGravityMultiplier;
    public bool usingYVelocityMultiplier;

    [Header("Values")]
    public float gravityScaleMultiplier = 5;
    public float yVelocityMultiplier = 0.1f;
    public float maxBounceForce = 30;
    public float bounceDelay = 0.1f;
    
    [HideInInspector]
    public float bounceForce;

    PlayerController player;

    private void OnCollisionEnter2D(Collision2D other)
    {
        if (other.gameObject.CompareTag("Player"))
        {
            AudioManager.Instance.GameplaySFX(AudioManager.Instance.trampolineJump, AudioManager.Instance.trampolineJumpVolume);
            player = other.gameObject.GetComponentInParent<PlayerController>();
            var rb2d = player.GetComponent<Rigidbody2D>();

            if (usingGravityMultiplier)
            {
                bounceForce += rb2d.gravityScale * gravityScaleMultiplier;
            }

            if (usingYVelocityMultiplier)
            {
                bounceForce += player.GetAbsoluteYVelocity() * yVelocityMultiplier;
            }

            if (bounceForce > maxBounceForce)
            {
                bounceForce = maxBounceForce;
            }

            if (rb2d != null)
            {
                player.isBouncing = true;
                Vector2 bounceDirection = transform.up * bounceForce;
                rb2d.AddForce(bounceDirection, ForceMode2D.Impulse);

                StartCoroutine(EnableMovement());
            }

            bounceForce = 0;
        }
    }

    IEnumerator EnableMovement()
    {
        yield return new WaitForSeconds(bounceDelay);
        player.isBouncing = false;
    }
}

```
</details>

<details>
  <summary>Show PlayerController.cs</summary>
  
```cs
  public class PlayerController : MonoBehaviour, IReset
  {
	#...
		
	private float currentMagnitude;
	private float prevMagnitude;
	
	private List<float> yVelocities = new();
	private int numberOfVelocitiesToRecord = 10;
	
	#...
	
	// Irrelevant code in the following method is omitted for brevity
	void FixedUpdate()
	{
		RecordYVelocity();
		RecordMagnitude();
	}
    
	#...
	
	private void RecordMagnitude()
	{
		prevMagnitude = currentMagnitude;
		currentMagnitude = rb.velocity.magnitude;
	}
	
	internal float GetMagnitude()
	{
		if (prevMagnitude != 0) return prevMagnitude;
		else return currentMagnitude;
	}
	
	
	internal float GetAbsoluteYVelocity()
	{
		for (int i = yVelocities.Count - 1; i >= 0; i--)
		{
		    if (yVelocities[i] > 0.0001f)
		    {
			return yVelocities[i];
		    }
		}
	
		return 0f;
	}
	
	void RecordYVelocity()
	{
		yVelocities.Add(Mathf.Abs(rb.velocity.y));
	
		if (yVelocities.Count > numberOfVelocitiesToRecord)
		{
	    		yVelocities.RemoveAt(0);
		}
	}
	
	#...
}
```
  
</details>

</details>

<br>

### Platforms

|<img src="/_GIFs/VesperPlatformsShort.gif" width="80%" />|
|---|

<details>
<summary>Show Platforms</summary>

#### Platform Showcase
Below, you'll find dropdowns containing GIFs of different platforms along with their associated code.

<br>

Click the dropdown arrows below to see the platforms! <br>

<details>
  <summary>Show Moving</summary>

|<img src="/_GIFs/VesperPlatformMoving.gif" width="80%" />|
|---|

<br>

*Click the dropdown arrow below to see `code`!* <br>

 <details>
  <summary>Show Moving.cs</summary>
    
```cs
public class Moving : MonoBehaviour, IReset
{
    [Header("Movement Settings")]
    public float waitDuration;
    public float speed = 1f;
    public float percentageDistance;
    [Range(0,1)] public float startPercentageDistance;

    [Header("Coordinates")]
    public List<Transform> coordinates;
    Transform start;
    Transform end;
    int currentIndex;

    Coroutine waitCoroutine;
    bool move = true;

    void Start()
    {
        RegisterSelfToResettableManager();
        InitialValues();
    }

    void FixedUpdate()
    {
        if (move)
        {
            percentageDistance += Time.deltaTime * speed;
            transform.position = Vector3.Lerp(start.position, end.position, percentageDistance);
        }

        if (percentageDistance >= 1)
        {
            waitCoroutine = StartCoroutine(Wait());
            NextCycle();
        }
    }

    void NextCycle()
    {
        percentageDistance = 0;
        start = end;
        currentIndex++;

        if (currentIndex >= coordinates.Count)
        {
            currentIndex = 0;
        }

        end = coordinates[currentIndex];
    }

    IEnumerator Wait()
    {
        move = false;
        yield return new WaitForSeconds(waitDuration);
        move = true;
    }

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.gameObject.CompareTag("Player"))
        {
            var playerHandler = PlayerController.player.transform.parent.GetComponent<PlayerHandler>();
            PlayerController.instance.rb.velocity += Vector2.up * speed;
            if (playerHandler != null)
            {
                playerHandler.SetParent(transform);
            }
        }
    }

    private void OnTriggerExit2D(Collider2D other)
    {
        if (other.gameObject.CompareTag("Player"))
        {
            var playerHandler = PlayerController.player.transform.parent.GetComponent<PlayerHandler>();
            if (playerHandler != null)
            {
                playerHandler.SetParent(null);
            }
        }
    }

    public void Reset()
    {
        InitialValues();
    }

    public void RegisterSelfToResettableManager()
    {
        ResettableManager.Instance?.RegisterObject(this);
    }

    private void InitialValues()
    {
        start = coordinates[0];
        end = coordinates[1];
        currentIndex = 1;
        percentageDistance = startPercentageDistance;
        move = true;

        if (waitCoroutine != null)
        {
            StopCoroutine(waitCoroutine);
        }
    }
}
```
  </details>

---

<br>

</details>

<details>
<summary>Show Disappearing</summary>
	
|<img src="/_GIFs/VesperPlatformDisappearing.gif" width="80%" />|
|---|

<br>

*Click the dropdown arrow below to see `code`!* <br>

<details>
<summary>Show Disappearing.cs</summary>
	
```cs
public class Disappearing : MonoBehaviour, IReset
{
    public float sustainTime = 1f;
    public float cooldown = 0.5f;
    public float reapperingParticleDuration = 1f;
    public Color32 onTriggerColor;
    private Color32 defaultColor;
    
    bool platformActive = true;
    bool previousActive =  true;
    bool playerOverlapping;
    bool ongoingCoroutine;
    Coroutine myCoroutine;

    public GameObject platform;
    SpriteRenderer platformSpriteRenderer;

    public UnityEvent disappear;
    public UnityEvent reappear;
    public UnityEvent fadeIn;
    public UnityEvent fadeOut;

    private void Awake()
    {
        platformSpriteRenderer = platform.GetComponent<SpriteRenderer>();
        defaultColor = platformSpriteRenderer.color;   
    }

    private void Start()
    {
        RegisterSelfToResettableManager();
    }

    private void Update()
    {
        if (!platformActive)
        {
            platform.SetActive(false);
        }
        else if (!playerOverlapping)
        {
            platform.SetActive(true);

            if (previousActive == false)
            {
                fadeIn.Invoke();
            }
        }

        previousActive = platformActive;
    }

    public void Disappear()
    {
        if (!ongoingCoroutine)
        {
            myCoroutine = StartCoroutine(DisappearAndComeBack());
        }
    }
    IEnumerator DisappearAndComeBack()
    {
        ongoingCoroutine = true;
  
        platformSpriteRenderer.color = onTriggerColor;


        yield return new WaitForSeconds(sustainTime);
        platformActive = false;
        disappear.Invoke();
        AudioManager.Instance.GameplaySFX(AudioManager.Instance.disappearingPlatformSound, AudioManager.Instance.disappearingPlatformVolume);
        yield return new WaitForSeconds(cooldown);
        reappear.Invoke();
        yield return new WaitForSeconds(reapperingParticleDuration);
        platformActive = true;

        AudioManager.Instance.GameplaySFX(AudioManager.Instance.appearingPlatformSound, AudioManager.Instance.appearingPlatformVolume);
        platformSpriteRenderer.color = defaultColor;

        ongoingCoroutine = false;
    }

    public void SetPlayerOverlapping(bool boolean)
    {
        playerOverlapping = boolean;
    }

    public void Reset()
    {
        if (ongoingCoroutine)
        {
            StopCoroutine(myCoroutine);
            ongoingCoroutine = false;
        }
        platformActive = true;
        previousActive = true;
        platformSpriteRenderer.color = defaultColor;
    }

    public void RegisterSelfToResettableManager()
    {
        ResettableManager.Instance?.RegisterObject(this);
    }
}
```


</details>

---

<br>

</details>


<details>
<summary>Show Destructible</summary>
	
|<img src="/_GIFs/VesperPlatformDestructible.gif" width="80%" />|
|---|

<br>

*Click the dropdown arrow below to see `code`!* <br>

<details>
    <summary>Show Destructible.cs</summary>
  
```cs
public class Destructible : MonoBehaviour, IReset
{
    [Header("Terrain Objects")]
    public List<GameObject> terrainObjects;

    [Header("Particle Effects")]
    public ParticleSystem destructionParticles;

    private ScreenShakeHandler screenShakeHandler;
    private bool ongoingCoroutine;

    [Header("Respawn Settings")]
    public bool respawnEnabled = false;
    public float respawnDelay = 5f;

    private void Start()
    {
        RegisterSelfToResettableManager();
    }

    public void TriggerDestroy()
    {
        if (ongoingCoroutine) return;

        screenShakeHandler = Camera.main.GetComponent<ScreenShakeHandler>();
        AudioManager.Instance.GameplaySFX(AudioManager.Instance.destructiblePlatfrom, AudioManager.Instance.destructiblePlatfromVolume);
        
        destructionParticles.Play();
        screenShakeHandler.DestructionShake();
        StartCoroutine(DestroyCoroutine());
    }

    private IEnumerator DestroyCoroutine()
    {
        ongoingCoroutine = true;

        // Disable all terrain objects
        foreach (GameObject obj in terrainObjects)
        {
            obj.SetActive(false);
        }

        // Handle respawn if enabled
        if (respawnEnabled)
        {
            yield return new WaitForSeconds(respawnDelay);
            foreach (GameObject obj in terrainObjects)
            {
                obj.SetActive(true);
            }
        }

        ongoingCoroutine = false;
    }

    public void Reset()
    {
        // Ensure all terrain objects are active on reset
        foreach (GameObject obj in terrainObjects)
        {
            obj.SetActive(true);
        }
    }

    private void RegisterSelfToResettableManager()
    {
        ResettableManager.Instance.RegisterObject(this);
    }
}

```
  </details>

---

<br>

</details>


<details>
  <summary>Show Rising</summary>

|<img src="/_GIFs/VesperPlatformRising.gif" width="80%" />|
|---|

<br>

*Click the dropdown arrows below to see `code`!* <br>

  <details>
  <summary>Show RisingMovement.cs</summary>
    
  ```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class RisingMovement : MonoBehaviour, IReset
{
    private enum States { DOWN, UP };

    [Header("Values")]
    public float targetHeight; // Target height for rising
    public float TimerToTarget = 4; // Time to reach target height
    public float durationOnTarget = 2; // Time to stay at target height

    [HideInInspector]
    public ParticleSystem rocks;

    private Vector3 initialPosition;
    private States currentState;
    private float timer;
    private float riseSpeed;

    void Start()
    {
        rocks = GetComponentInChildren<ParticleSystem>();
        initialPosition = transform.position;
        currentState = States.DOWN;
        RegisterSelfToResettableManager();

        // Calculate the speed at which the object will rise
        riseSpeed = (targetHeight - initialPosition.y) / TimerToTarget;
    }

    void Update()
    {
        Movement();
    }

    private void Movement()
    {
        timer -= Time.deltaTime;

        switch (currentState)
        {
            case States.DOWN:
                if (timer <= 0 && transform.position.y > initialPosition.y)
                {
                    MoveDown();
                }
                break;

            case States.UP:
                if (transform.position.y < targetHeight)
                {
                    MoveUp();
                }
                break;
        }
    }

    private void MoveDown()
    {
        float step = riseSpeed * Time.deltaTime;
        transform.Translate(Vector3.down * step);
    }

    private void MoveUp()
    {
        float step = riseSpeed * Time.deltaTime;
        transform.Translate(Vector3.up * step);
    }

    public void Rise()
    {
        currentState = States.UP;
    }

    public void Descend()
    {
        if (currentState == States.DOWN) return;

        currentState = States.DOWN;
        timer = durationOnTarget;
    }

    public void ResetTimer()
    {
        timer = 0;
    }

    public void Reset()
    {
        timer = 0;
        transform.position = initialPosition;
        currentState = States.DOWN;
    }

    public void RegisterSelfToResettableManager()
    {
        ResettableManager.Instance?.RegisterObject(this);
    }
}
  ```
  </details>
  
  <details>
  <summary>Show RisingButton.cs</summary>
    
  ```cs
  public class RisingButton : MonoBehaviour, IReset
{
    // Takes in the moving platforms
    public List<RisingMovement> platforms;
   
    // Animator anim;
    public float pressedDistance;
    public float timer;
    private bool playerIsLarge;
    private bool onMe;
    private bool prevSize;
    public Transform box;
    private Vector3 stopPos;

    private void Start()
    {
        stopPos = Vector3.zero;
        RegisterSelfToResettableManager();
    }
    
    public void Update()
    {
        timer += Time.deltaTime;
        if(onMe)
        {
            if (playerIsLarge)
            {
                box.localPosition = Vector3.Lerp(Vector3.zero, Vector3.down * pressedDistance, timer);
            }
            else
            {
                box.localPosition = Vector3.Lerp(box.localPosition, Vector3.zero, Time.deltaTime * 4);
            }
        }
      
        else
        {
            box.localPosition = Vector3.Lerp(box.localPosition, Vector3.zero, Time.deltaTime * 4);
        }

        playerIsLarge = PlayerController.instance.currentSize == Sizes.BIG;
        
        if(prevSize != playerIsLarge)
        {
            if(playerIsLarge == false)
            {
                stopPos = box.transform.localPosition;
            }
            timer = 0;
        }
        prevSize = playerIsLarge;
    }
    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.gameObject.CompareTag("Player"))
        {
            if (onMe == false)
            {
                timer = 0;
                onMe = true;
            }

            foreach (var platform in platforms)
            {
                if (playerIsLarge)
                {
                    platform.Rise();
                }
                else
                {
                    platform.Descend();

                }
            }
        }
    }

    private void OnTriggerStay2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            foreach (var platform in platforms)
            {
                if (playerIsLarge)
                {
                    platform.Rise();
                }
                else
                {
                    platform.Descend();
                }
            }
        }
    }

    private void OnTriggerExit2D(Collider2D other)
    {
        if (other.gameObject.CompareTag("Player"))
        {
            if (onMe == true)
            {
                onMe = false;
                timer = 0;
                stopPos = box.transform.localPosition;
            }
            foreach (var platform in platforms)
                platform.Descend();
        }
    }

    public void DescendPlatformsPromptly()
    {
        foreach (var platform in platforms)
        {
            platform.Descend();
            platform.ResetTimer();
        }
    }

    public void Reset()
    {
        stopPos = Vector3.zero;
        box.transform.localPosition = Vector3.zero;
    }

    public void RegisterSelfToResettableManager()
    {
        ResettableManager.Instance?.RegisterObject(this);
    }
}
  ```
  </details>
</details>

<br>

#### IReset
All platforms and relevant classes implement the `IReset` interface, enabling the resetting of objects upon player death without reloading the level. Each class that inherits from this interface includes a method to register itself with the `ResettableManager`. When a reset is required, the `ResettableManager` iterates through its list and invokes the reset function for each object.

<details>
<summary>Show IReset.cs</summary>
  
```cs
using UnityEngine;

public interface IReset
{
    void Reset();

    void RegisterSelfToResettableManager();
}
```
</details>

</details>

<br>




