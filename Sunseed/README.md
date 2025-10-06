# __Sun Seed Vegan Knights__

![sunseed_banner](/_Images/ZmxUP5.png)


## [___Sun Seed Vegan Knights___]

```
Developed:  11/2024 - 12/2024
Duration:   7 weeks
Engine:     Unity
Genre:      2D Action Game / Hack-and-Slash
Team:       3 Programmers, 4 Artist
```

<table>
  <tr>
    <td width="50%"><img src="/PortfolioBilder/MenuScreenTemp.png" /></td>
    <td width="50%"><img src="/PortfolioBilder/sunseed2.png" /></td>
  </tr>
  <tr>
    <td width="50%"><img src="/PortfolioBilder/sunseed3.png" /></td>
    <td width="50%"><img src="/PortfolioBilder/sunseed1.png" /></td>
  </tr>
</table>

## _A Brief Game Description_
Sun Seed Vegan Knights, a multiplayer mission based hack & slash game. You and your friends gets placed in a level where your mission is to eliminate the enemy and protect the objective

[Webpage (itch.io)](https://yrgo-game-creator.itch.io/sun-seed) <br>
[Trailer](https://www.youtube.com/watch?v=URABgAn8mho)

## _My Roles in the Project_

I co-developed the player controls and mechanics alongside the other programmers, contributed to adding polish and “juice” to enhance the gameplay experience.

My most significant contribution was the development of the tutorial aswell as some interactive elements within the game world. I took it upon myself to design and create the tutorial level which is a very important part of a game. Our game has some complex elements to it which makes it very important that the players understand the core mechanics before leaving the level.

Below you’ll find a list of a few features I created and implemented myself, along with summaries of my code.

# Contributions 

### Frog
	
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
<summary>Show WaterObjective.cs</summary>

 ```cs
public class WaterObjective : MonoBehaviour
{
    [SerializeField] private float maxWater = 1500f;
    private float currentWater = 0f;

    [SerializeField] private float waterDepletionRate = 5f;
    private bool isComplete = false;

    [SerializeField] private TMP_Text waterProgressText;

    [SerializeField] private LineRenderer tongueLine;
    [SerializeField] private Transform shootPoint;
    [SerializeField] private float shootInterval = 0.5f;
    [SerializeField] private float detectionRadius = 10.0f;

    [SerializeField] private Animator frogAnimator;
    [SerializeField] private SpriteRenderer frogSpriteRenderer;
    [SerializeField] private Sprite frogOpenMouthSprite;

    private bool isTurretActive = false;

    private void Start()
    {
        currentWater = 1; // starting water
        UpdateProgressUI();
        StartCoroutine(DepleteWater());
    }

    private void UpdateProgressUI()
    {
        if (waterProgressText != null)
        {
            waterProgressText.text = $"{Mathf.FloorToInt(currentWater)} / {Mathf.FloorToInt(maxWater)}";
        }
    }

    public void AddWater(float amount)
    {
        if (!isComplete)
        {
            currentWater += amount;
            currentWater = Mathf.Clamp(currentWater, 0, maxWater);
            UpdateProgressUI();

            if (currentWater >= maxWater)
            {
                CompleteObjective();
            }
        }
    }

    private void CompleteObjective()
    {
        isComplete = true;
        StopCoroutine(DepleteWater());
        ActivateTurret();
    }

    private IEnumerator DepleteWater()
    {
        while (!isComplete)
        {
            yield return new WaitForSeconds(1f);

            if (currentWater >= maxWater || currentWater <= 0)
            {
                continue;
            }
            currentWater -= waterDepletionRate;
            currentWater = Mathf.Max(currentWater, 0);
            UpdateProgressUI();

            if (currentWater <= 0)
            {
                break;
            }
        }
    }

    private void ActivateTurret()
    {
        isTurretActive = true;
        StartCoroutine(Shoot());
    }

    private IEnumerator Shoot()
    {
        while (isTurretActive)
        {
            GameObject target = FindNearestEnemy();

            if (target != null)
            {
                yield return StartCoroutine(ShootAtTarget(target));
            }

            yield return new WaitForSeconds(shootInterval);
        }
    }

    private GameObject FindNearestEnemy()
    {
        EnemyHealthDisplay[] enemies = FindObjectsOfType<EnemyHealthDisplay>();
        GameObject nearestEnemy = null;
        float shortestDistance = detectionRadius;

        foreach (EnemyHealthDisplay enemy in enemies)
        {
            float distanceToEnemy = Vector3.Distance(transform.position, enemy.transform.position);

            if (distanceToEnemy < shortestDistance)
            {
                shortestDistance = distanceToEnemy;
                nearestEnemy = enemy.gameObject;
            }
        }

        return nearestEnemy;
    }
    private IEnumerator ShootAtTarget(GameObject target)
    {
        if (tongueLine != null && shootPoint != null)
        {
            if (frogAnimator != null)
            {
                frogAnimator.enabled = false;
            }
            if (frogSpriteRenderer != null && frogOpenMouthSprite != null)
            {
                frogSpriteRenderer.sprite = frogOpenMouthSprite;
            }

                float shootSpeed = 120f;
                Vector2 startPosition = shootPoint.position;
                Vector2 endPosition = target != null ? target.transform.position : startPosition;
                float distance = Vector2.Distance(startPosition, endPosition);
                float time = 0;

                tongueLine.SetPosition(0, startPosition);

                float maxTongueDuration = 5f; 
                float elapsedTongueTime = 0f;

                while (time < distance / shootSpeed && elapsedTongueTime < maxTongueDuration)
                {
                    if (target == null || !target.activeInHierarchy)
                    {
                        break;
                    }

                    time += Time.deltaTime;
                    elapsedTongueTime += Time.deltaTime;
                    Vector2 currentPoint = Vector2.Lerp(startPosition, endPosition, time / (distance / shootSpeed));
                    tongueLine.SetPosition(1, new Vector3(currentPoint.x, currentPoint.y, 0));
                    yield return null;
                }

            tongueLine.SetPosition(0, Vector3.zero);
            tongueLine.SetPosition(1, Vector3.zero);

            if (frogAnimator != null)
            {
                frogAnimator.enabled = true;
            }

            if (target != null && target.activeInHierarchy)
            {
                Health targetHealth = target.GetComponent<Health>();
                if (targetHealth != null)
                {
                    if (targetHealth.GetCurrentHealth() <= 5)
                    {
                        targetHealth.TakeDamage(150);
                    }
                    else if (targetHealth.GetCurrentHealth() <= 150)
                    {
                        yield return StartCoroutine(DragTargetToFrog(target));
                        targetHealth.TakeDamage(150);
                    }
                    else
                    {
                        targetHealth.TakeDamage(150);
                    }
                }
            }
        }
    }
    private IEnumerator DragTargetToFrog(GameObject target)
    {
        if (target == null || !target.activeInHierarchy)
        {
            yield break; 
        }

        BloomRecipient bloomRecipient = target.GetComponent<BloomRecipient>();
        if (bloomRecipient != null)
        {
            bloomRecipient.ResetForDrag();
        }
        
        if (frogAnimator != null)
        {
            frogAnimator.enabled = false;
        }
        if (frogSpriteRenderer != null && frogOpenMouthSprite != null)
        {
            frogSpriteRenderer.sprite = frogOpenMouthSprite;
        }

        Vector3 startPosition = target.transform.position;
        Vector3 endPosition = shootPoint.position;
        float dragSpeed = 0.6f;
        float time = 0;

        if (target.TryGetComponent(out Rigidbody2D rb))
        {
            rb.velocity = Vector2.zero;
            rb.isKinematic = true;
        }

        Collider2D frogCollider = GetComponent<Collider2D>();
        Collider2D targetCollider = target.GetComponent<Collider2D>();
        if (frogCollider != null && targetCollider != null)
        {
            Physics2D.IgnoreCollision(frogCollider, targetCollider, true);
        }

        while (time < 1f)
        {
            if (target == null || !target.activeInHierarchy)
            {
                yield break; 
            }

            time += Time.deltaTime * dragSpeed;
            Vector3 currentTargetPosition = Vector3.Lerp(startPosition, endPosition, time);
            target.transform.position = currentTargetPosition;

            tongueLine.SetPosition(0, shootPoint.position);
            tongueLine.SetPosition(1, currentTargetPosition);

            yield return null;
        }

        if (frogCollider != null && targetCollider != null)
        {
            Physics2D.IgnoreCollision(frogCollider, targetCollider, false);
        }

        tongueLine.SetPosition(0, Vector3.zero);
        tongueLine.SetPosition(1, Vector3.zero);

        if (frogAnimator != null)
        {
            frogAnimator.enabled = true;
        }
    }
}
```
</details>

</details>

<br>

### Dialogue

|<img src="/PortfolioBilder/dialogue3.jpg" width="80%" />|
|---|
<details>
<summary>Show Dialogue</summary>

#### The Idea
The aim was to create a dialogue system for tutorial and also use the system in the hub for an interactable NPC. 

#### The Logic 
This dialogue system automatically moves the camera between stages when all enemies in a stage are defeated, triggering stage-specific dialogues using a dynamic dialogue system that manages player input and action maps.

Player movement is temporarily disabled to ensure that they read through the dialogue aswell as doesn't accidently skip anything.

<br>

*Click the dropdown arrows below to see the `code`!* <br>

<details>
<summary>Show Dialogue.cs</summary>
  
```cs
public class Dialogue : MonoBehaviour
{
    public TextMeshProUGUI textComponent;
    [TextArea(3, 10)]
    public string[] lines;

    private int index;
    private PlayerInput playerInput;
    public UnityEvent<int> onDialogueLineChanged;
    public string actionMapToDisable = "ControlActions1"; 
    public bool IsDialogueActive { get; private set; } 

    private void Start()
    {
        textComponent.text = string.Empty;
        StartDialogue(lines);
    }

    public void OnPlayerJoined(PlayerInput playerInput)
    {
        if (this.playerInput == null)
        {
            this.playerInput = playerInput;

            playerInput.actions["NextDialogue"].performed += OnNextDialoguePerformed;
            playerInput.actions["PreviousDialogue"].performed += OnPreviousDialoguePerformed; 
        }
    }

    private void OnDisable()
    {
        if (playerInput != null)
        {
            playerInput.actions["NextDialogue"].performed -= OnNextDialoguePerformed;
            playerInput.actions["PreviousDialogue"].performed -= OnPreviousDialoguePerformed;
        }
    }

    private void OnEnable()
    {
        if (playerInput != null)
        {
            playerInput.actions["NextDialogue"].Enable();
            playerInput.actions["PreviousDialogue"].Enable();
        }
    }

    private void OnNextDialoguePerformed(InputAction.CallbackContext context)
    {
        NextLine();
    }

    private void OnPreviousDialoguePerformed(InputAction.CallbackContext context)
    {
        PreviousLine();
    }

    public void StartDialogue(string[] newLines)
    {
        if (newLines == null || newLines.Length == 0)
        {
            return;
        }

        lines = newLines;
        index = 0;
        IsDialogueActive = true; 
        gameObject.SetActive(true); 
        DisplayLine();

        DisableActionMap();
    }

    private void DisplayLine()
    {
        if (index >= 0 && index < lines.Length)
        {
            textComponent.text = lines[index];
            onDialogueLineChanged?.Invoke(index);
        }
    }

    private bool canAdvanceDialogue = true;

    public void NextLine()
    {
        if (!canAdvanceDialogue) return;

        StartCoroutine(DebounceDialogueAdvance());

        if (index < lines.Length - 1)
        {
            index++;
            DisplayLine();
        }
        else
        {
            EndDialogue();
        }
    }

    private IEnumerator DebounceDialogueAdvance()
    {
        canAdvanceDialogue = false;
        yield return new WaitForSeconds(0.01f); 
        canAdvanceDialogue = true;
    }

    public void PreviousLine()
    {
        if (index > 0)
        {
            index--;
            DisplayLine();
        }
    }

    private void EndDialogue()
    {
        IsDialogueActive = false;
        StartCoroutine(EndDialogueWithDelay(0.05f));
    }

    private IEnumerator EndDialogueWithDelay(float delay)
    {
        yield return new WaitForSeconds(delay);
        gameObject.SetActive(false);
        textComponent.text = string.Empty;
        EnableActionMap();
    }
    private PlayerInput dialogueControllerPlayer; 

    private void DisableActionMap()
    {
        var players = FindObjectsOfType<PlayerInput>();

        foreach (PlayerInput player in players)
        {
            if (player == dialogueControllerPlayer)
            {
                player.SwitchCurrentActionMap("UI"); 
            }
            else
            {
                player.SwitchCurrentActionMap("Disabled"); 
            }
        }
    }

    private void EnableActionMap()
    {
        foreach (PlayerInput player in FindObjectsOfType<PlayerInput>())
        {
            if (player == dialogueControllerPlayer)
            {
                player.SwitchCurrentActionMap(actionMapToDisable); 
            }
            else
            {
                player.SwitchCurrentActionMap("ControlActions1"); 
            }
        }
    }
}

```
</details>

<details>
  <summary>Show CameraMoverOnEnemyDeath.cs</summary>
  
```cs
[System.Serializable]
public class StageDialogue
{
    [TextArea(3, 10)]
    public string[] dialogues;
}

public class CameraMoverOnEnemyDeath : MonoBehaviour
{
    public GameObject[] enemiesStage1;
    public GameObject[] enemiesStage2;
    public GameObject[] enemiesStage3;
    public Transform[] cameraPositions;
    public float cameraSpeed = 2f;
    public Dialogue dialogueSystem;

    public StageDialogue[] stageDialogues;

    private int currentStage = 0;
    private bool moveCamera = false;

    void Start()
    {
        if (dialogueSystem == null || stageDialogues == null || stageDialogues.Length == 0)
        {
            return;
        }
        
        dialogueSystem.gameObject.SetActive(false);
    }
    void Update()
    {
        if (currentStage < cameraPositions.Length && AreAllEnemiesDead(GetCurrentEnemies()))
        {
            moveCamera = true;
        }

        if (moveCamera)
        {
            MoveCameraToTarget();
        }
    }

    public IEnumerator ShowDialogueAfterDelay(float delay)
    {
        yield return new WaitForSeconds(delay);

        dialogueSystem.gameObject.SetActive(true);

        if (stageDialogues.Length > 0)
        {
            dialogueSystem.StartDialogue(stageDialogues[currentStage].dialogues);
        }
    }

    void MoveCameraToTarget()
    {
        transform.position = Vector3.MoveTowards(transform.position, cameraPositions[currentStage].position, cameraSpeed * Time.deltaTime);

        if (Vector3.Distance(transform.position, cameraPositions[currentStage].position) <= 0.1f)
        {
            transform.position = cameraPositions[currentStage].position;
            moveCamera = false;
            TriggerNextStage();
        }
    }

    void TriggerNextStage()
    {
        currentStage++;

        if (currentStage < stageDialogues.Length && dialogueSystem != null)
        {
            dialogueSystem.StartDialogue(stageDialogues[currentStage].dialogues);
        }
    }

    GameObject[] GetCurrentEnemies()
    {
        switch (currentStage)
        {
            case 0: return enemiesStage1;
            case 1: return enemiesStage2;
            case 2: return enemiesStage3;
            default: return new GameObject[0];
        }
    }

    bool AreAllEnemiesDead(GameObject[] enemies)
    {
        foreach (GameObject enemy in enemies)
        {
            if (enemy != null)
            {
                return false;
            }
        }
        return true;
    }
}
```
  
</details>

<details>
  <summary>Show EnableEnemyOnDialogue.cs</summary>
  
```cs
public class EnableEnemyOnDialogue : MonoBehaviour
{
    public Dialogue dialogueSystem;
    public GameObject enemyToEnable; 
    public int dialogueIndexToEnableEnemy = 1; 

    private bool enemyEnabled = false; 

    private void Start()
    {
        if (dialogueSystem != null)
        {
            dialogueSystem.onDialogueLineChanged.AddListener(OnDialogueLineChanged);
        }

        if (enemyToEnable != null)
        {
            enemyToEnable.SetActive(false); 
        }
    }

    private void OnDialogueLineChanged(int index)
    {
        if (!enemyEnabled && index == dialogueIndexToEnableEnemy)
        {
            if (enemyToEnable != null)
            {
                enemyToEnable.SetActive(true); 
                enemyEnabled = true; 
            }
        }
    }

    private void OnDestroy()
    {
        if (dialogueSystem != null)
        {
            dialogueSystem.onDialogueLineChanged.RemoveListener(OnDialogueLineChanged);
        }
    }
}
```
  
</details>

</details>


<br>

### Extras

<details>
<summary>Some other stuff I did</summary>

#### Extra Showcase
Below, you'll find some contributions I did aswell as some example code

<br>

I also worked on: Learning and implementing the new unity input system, enemies, sound, multiplayer, balancing, game design  <br>

<br>

Click the dropdown arrows below to see some example code! <br>

 <details>
  <summary>Show Enemy.cs</summary>
    
```cs
public class EnemyAttacks : MonoBehaviour
{
    Pathfinding pathfindingScript;

    protected Vector3 targetPosition;
    [HideInInspector] public float distenceToTarget;

    public float distanceToAttack = 20;

    [HideInInspector] public bool isAttacking = false;
    [HideInInspector] public bool withinDistance = false;

    NavMeshAgent agent;

    private void Start()
    {
        agent = GetComponent<NavMeshAgent>();
        pathfindingScript = GetComponent<Pathfinding>();
    }

    private void Update()
    {
        if (!(pathfindingScript.target.Count <= 0) && pathfindingScript.target[pathfindingScript.finalTarget] != null)
        {
            targetPosition = pathfindingScript.target[pathfindingScript.finalTarget].transform.position - transform.position;
        }
        
     
        distenceToTarget = targetPosition.sqrMagnitude;

        if (distenceToTarget < distanceToAttack)
        {
            withinDistance = true;
            pathfindingScript.followTarget = false;
            if (pathfindingScript.trackTarget == true)
            {
                agent.velocity = Vector3.zero;
            }
        }

        if (distenceToTarget > distanceToAttack && isAttacking == false)
        {
            pathfindingScript.followTarget = true;
            withinDistance = false;
        }
    }
}

```
<br>

</details>

<details>
<summary>Show PlayerAttack.cs</summary>
	
```cs
public class PlayerAttack : MonoBehaviour
{
    private GameObject weapon;

    private Collider2D weaponCollider;

    private Animator weaponAnimator;

    private PlayerInput playerInput;
    private InputAction fireAction;

    private void Awake()
    {
        playerInput = GetComponent<PlayerInput>();
        if (playerInput != null)
        {
            fireAction = playerInput.actions["Fire"];
        }
        else
        {
            Debug.LogError("PlayerInput component is missing on this GameObject.");
        }
    }

    private void Start()
    {
        weaponCollider = weapon.GetComponent<Collider2D>();
        weaponAnimator = weapon.GetComponent<Animator>();

        if (fireAction != null)
        {
            fireAction.performed += OnFirePerformed;
        }
        else
        {
            Debug.LogError("Fire action could not be found. Check the Input Action Asset.");
        }
    }

    private void Update()
    private void OnDestroy()
    {
        if (Input.GetKeyDown(KeyCode.Joystick1Button5))
        if (fireAction != null)
        {
            weaponAnimator.SetTrigger("PressedR1");
            fireAction.performed -= OnFirePerformed;
        }
    }

    private void OnFirePerformed(InputAction.CallbackContext context)
    {
        weaponAnimator.SetTrigger("PressedR1");
    }
}
```
</details>


<details>
    <summary>Show PlayedJoined.cs</summary>
  
```cs
public class PlayerJoined : MonoBehaviour
{
    public Dialogue dialogueSystem;
    public TextMeshProUGUI messageText;
    private PlayerInputManager playerInputManager;
    public CameraMoverOnEnemyDeath cameraMover;

    private PlayerInput firstPlayerInput; 

    private void Start()
    {
        if (messageText != null)
        {
            messageText.gameObject.SetActive(true);
        }
    }

    void OnEnable()
    {
        playerInputManager = FindObjectOfType<PlayerInputManager>();
        if (playerInputManager != null)
        {
            playerInputManager.onPlayerJoined += OnPlayerJoined;
        }
    }

    void OnDisable()
    {
        if (playerInputManager != null)
        {
            playerInputManager.onPlayerJoined -= OnPlayerJoined;
        }
    }

    public void OnPlayerJoined(PlayerInput playerInput)
    {
        if (playerInput.devices.Count > 0 && 
            (playerInput.devices[0] is Keyboard || playerInput.devices[0] is Mouse))
        {
            Destroy(playerInput.gameObject);
            return;
        }

        DontDestroyOnLoad(playerInput.gameObject);

        if (firstPlayerInput == null)
        {
            firstPlayerInput = playerInput;
            playerInput.SwitchCurrentActionMap("ControlActions1"); 
        }
        else
        {
            playerInput.SwitchCurrentActionMap("ControlActions1");
        }

        InputAction pauseAction = playerInput.actions["Pause"];
        if (pauseAction != null)
        {
            pauseAction.Disable();
            StartCoroutine(ReenablePauseAction(pauseAction));
        }

        PlayerAttack playerAttack = playerInput.GetComponent<PlayerAttack>();
        if (playerAttack == null)
        {
            playerAttack = playerInput.gameObject.AddComponent<PlayerAttack>();
        }
        playerAttack.Initialize(playerInput);

        if (messageText != null)
        {
            messageText.text = "Use   <voffset=0.3em><sprite=3></voffset>to move and   <voffset=0.3em><sprite=0></voffset>to rotate";
            Invoke(nameof(HideMessage), 5f);
        }

        if (dialogueSystem != null && playerInput == firstPlayerInput) 
        {
            dialogueSystem.OnPlayerJoined(playerInput);

            playerInput.actions["NextDialogue"].performed += context =>
            {
                if (dialogueSystem.IsDialogueActive)
                {
                    dialogueSystem.NextLine();
                }
            };

            playerInput.actions["PreviousDialogue"].performed += context =>
            {
                if (dialogueSystem.IsDialogueActive)
                {
                    dialogueSystem.PreviousLine();
                }
            };
        }
    }

    private IEnumerator ReenablePauseAction(InputAction pauseAction)
    {
        yield return null;
        pauseAction.Enable();
    }

    private void HideMessage()
    {
        if (messageText != null)
        {
            messageText.gameObject.SetActive(false);
        }
        if (cameraMover != null)
        {
            StartCoroutine(StartDialogueCoroutine());
        }
        StartCoroutine(ShowSecondMessageCoroutine());
    }

    private IEnumerator StartDialogueCoroutine()
    {
        yield return cameraMover.ShowDialogueAfterDelay(3.5f);
    }

    private IEnumerator ShowSecondMessageCoroutine()
    {
        yield return new WaitForSeconds(0f);
        if (messageText != null)
        {
            messageText.gameObject.SetActive(true);
            messageText.text = "Press   <voffset=0.3em><sprite=2></voffset>to dash.";
            Invoke(nameof(HideSecondMessage), 3.5f);
        }
    }

    private void HideSecondMessage()
    {
        if (messageText != null)
        {
            messageText.gameObject.SetActive(false);
        }
    }
}

```

</details>






