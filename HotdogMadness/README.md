# __Hot Dog Madness__

```
Developed:  11/2024 - 12/2024
Duration:   5 weeks
Engine:     Unity
Genre:      Virtual Reality, work simulator
Team:       4 programmers
```

<table>
  <tr>
    <td width="50%"><img src="/PortfolioBilder/hotdog1.png" /></td>
    <td width="50%"><img src="/PortfolioBilder/hotdog2.png" /></td>
  </tr>
  <tr>
    <td width="50%"><img src="/PortfolioBilder/hotdog3.png" /></td>
    <td width="50%"><img src="/PortfolioBilder/hotdog4.png" /></td>
  </tr>
</table>

## _A Brief Game Description_
Hot Dog Madness, an emerging VR game where you work as a hot dog man. Fast paced game with a variation of orders and levels.


## _My Roles in the Project_

I took it upon myself to make various mechanics such as the condiments, radio and all the sound in the game. 

My most significant contribution was the development of the condiments aswell as finding, mixing and implementing sounds to everything in the game.

Below you’ll find a list of a few features I created and implemented myself, along with summaries of my code.

# Contributions 

### Condiments
	
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="/PortfolioGifs/Condiments.gif" alt="juice1" width="800" height="auto"> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; |
|:---:|


> *In the GIF, you can see the player pouring condiments over a hotdog.*

<details>
  <summary>Condiments</summary>

#### The Idea
Making an easy to use ketchup + mustard bottle that recognizes when you have the bottle upside down.

#### The Logic
To have the frog assist you in combat you have to water it which is a core mechanic in Sun Seed. After you have watered it to the required number it starts shooting out its tounge at the nearest "Enemy" tagged object. The frog launches its toungue as a linerenderer, grabs enemy, eats it (destroys enemy) and it resets the timer.

<br>

*Click the dropdown arrows below to see the `code`!* <br>

<details>
<summary>Show PourDetector.cs</summary>

 ```cs
public class PourDetector : MonoBehaviour
{
    [Header("Pour Settings")]
    [Range(0, 180f)] public float pourThreshold = 100f; 
    public Transform origin;
    public Stream streamPrefab;

    private Stream currentStream;

    private XRGrabInteractable grab;
    private bool isHeld;
    private bool isTriggerPressed;

    public AudioClip pourSound;
    SoundObject currentPourSound;

    private void Awake()
    {
        grab = GetComponent<XRGrabInteractable>();
    }

    private void OnEnable()
    {
        if (!grab) return;

        grab.selectEntered.AddListener(OnSelectEntered);
        grab.selectExited.AddListener(OnSelectExited);

        grab.activated.AddListener(OnActivated);
        grab.deactivated.AddListener(OnDeactivated);
    }

    private void OnDisable()
    {
        if (!grab) return;

        grab.selectEntered.RemoveListener(OnSelectEntered);
        grab.selectExited.RemoveListener(OnSelectExited);
        grab.activated.RemoveListener(OnActivated);
        grab.deactivated.RemoveListener(OnDeactivated);
    }

    private void OnSelectEntered(SelectEnterEventArgs args)
    {
        isHeld = true;
    }

    private void OnSelectExited(SelectExitEventArgs args)
    {
        isHeld = false;
        isTriggerPressed = false;
        EndPour();
    }

    private void OnActivated(ActivateEventArgs args)
    {
        isTriggerPressed = true;
    }

    private void OnDeactivated(DeactivateEventArgs args)
    {
        isTriggerPressed = false;
        EndPour();
    }

    private void Update()
    {
        bool shouldPour = isHeld && isTriggerPressed && IsTiltPastThreshold();

        if (shouldPour && currentStream == null)
        {
            StartPour();
        }
        else if (!shouldPour && currentStream != null)
        {
            EndPour();
        }

        if (currentStream != null && origin != null)
            currentStream.transform.position = origin.position;
    }

    private bool IsTiltPastThreshold()
    {
        float angleFromUp = Vector3.Angle(transform.up, Vector3.up);
        return angleFromUp > pourThreshold;
    }

    private void StartPour()
    {
        if (origin == null || streamPrefab == null) return;
        currentStream = Instantiate(streamPrefab, origin.position, Quaternion.identity, transform);
        currentStream.SetOrigin(origin);
        currentStream.Begin();
        currentPourSound = SoundManager.Instance.PlaySoundFX(pourSound, transform.position, transform, 1);
    }

    private void EndPour()
    {
        if (currentStream == null) return;
        Destroy(currentStream.gameObject);
        currentStream = null;

        if (currentPourSound != null)
        {
            currentPourSound.StopPlaying(); 
            currentPourSound = null;
        }
    }
}

```
</details>

<details>
<summary>Show SauceBlobStainer.cs</summary>

 ```cs
[RequireComponent(typeof(ParticleSystem))]
public class SauceBlobStainer : MonoBehaviour
{
    public GameObject blobPrefab;
    public LayerMask stainLayers = ~0;
    public string blobLayerName = "SauceSplat";

    [Header("Spawn")]
    public int blobsPerCollision = 3;      
    public float minIntervalPerTarget = 0.01f; 
    public float jitterRadius = 0.0025f;   
    public float surfaceOffset = 0.0006f;   
    public bool parentToHitObject = true;

    private ParticleSystem ps;
    private readonly List<ParticleCollisionEvent> buf = new();
    private readonly Dictionary<Transform, float> lastTime = new();
    private int blobLayer = -1;

    private List<GameObject> spawnedBlobs = new List<GameObject>();
    private bool hitFoodObject;

    void Awake()
    {
        ps = GetComponent<ParticleSystem>();
        blobLayer = LayerMask.NameToLayer(blobLayerName);
    }

    void OnParticleCollision(GameObject other)
    {
        if (ps == null || blobPrefab == null) return;
        if (((1 << other.layer) & stainLayers) == 0) return;

        int n = ParticlePhysicsExtensions.GetCollisionEvents(ps, other, buf);
        if (n == 0) return;

        Transform target = other.transform;
        float now = Time.time;
        if (!lastTime.TryGetValue(target, out float last)) last = 0f;
        if (now - last < minIntervalPerTarget) return;

        var e = buf[n - 1];
        Vector3 pos = e.intersection;
        Vector3 normal = e.normal;

        SpawnBlobs(target, pos, normal);

        lastTime[target] = now;
    }

    void SpawnBlobs(Transform target, Vector3 center, Vector3 normal)
    {
        Vector3 t = Vector3.Cross(normal, Vector3.up);
        if (t.sqrMagnitude < 1e-4f) t = Vector3.Cross(normal, Vector3.right);
        t.Normalize();
        Vector3 b = Vector3.Cross(normal, t);

        Transform parent = parentToHitObject ? target : null;
        Quaternion rot = Quaternion.LookRotation(normal);

        for (int i = 0; i < blobsPerCollision; i++)
        {
            Vector2 j2 = Random.insideUnitCircle * jitterRadius;
            Vector3 j = t * j2.x + b * j2.y;
            var go = Instantiate(blobPrefab, center + normal * surfaceOffset + j, rot);
            if (parentToHitObject && target != null)
            {
                go.transform.SetParent(target, true);
            }
                
            if (blobLayer != -1) SetLayerRecursively(go, blobLayer);
        }
    }

    static void SetLayerRecursively(GameObject obj, int layer)
    {
        obj.layer = layer;
        foreach (Transform c in obj.transform) SetLayerRecursively(c.gameObject, layer);
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
  <summary>Show Enemy</summary>
<br>
*Click the dropdown arrow below to see `code`!* <br>

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
  </details>

---

<br>

</details>

<details>
<summary>Show PlayerAttack</summary>

<br>

*Click the dropdown arrow below to see `code`!* <br>

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

---

<br>

</details>


<details>
<summary>Show PlayerJoined</summary>
	

<br>

*Click the dropdown arrow below to see `code`!* <br>

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

---

<br>

</details>






