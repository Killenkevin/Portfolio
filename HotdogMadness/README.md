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

### Radio

<details>
<summary>Show Dialogue</summary>

#### The Idea
The aim was to create a working radio that the player could start, pause, resume, skip to the next song.

#### The Logic 
The radio has a list of songs it goes through. The two buttons on the radio dictates if the player wants to skip to the next song, pause, resume or play.

The buttons has to be kinematic since they have the grabable element on them. Makes it so the player can interact with it.

<br>

*Click the dropdown arrows below to see the `code`!* <br>

<details>
<summary>Show Radio.cs</summary>
  
```cs
public class Radio : MonoBehaviour
{
    [SerializeField] List<AudioClip> musicList = new();
    [SerializeField, Range(0f,1f)] float volume = 0.8f;
    bool mono = false;

    SoundObject currentLoop;
    int lastIndex = -1;
    bool isPaused = false;

    public void PlayNext()
    {
        if (musicList == null || musicList.Count == 0) return;

        isPaused = false;

        int index = lastIndex + 1;
        if (index >= musicList.Count) index = 0;
        lastIndex = index;

        if (currentLoop != null) currentLoop.StopPlaying();

        currentLoop = SoundManager.Instance.PlayMusic(
            musicList[index],
            transform.position,
            transform,
            volume,
            looping: true,
            pitch: 1f,
            fadeOutLength: 0f,
            mono: mono
        );
    }

    public void TogglePause()
    {
        if (currentLoop == null) return;

        if (!isPaused)
        {
            currentLoop.Pause();
            isPaused = true;
        }
        else
        {
            currentLoop.Resume();
            isPaused = false;
        }
    }
}
```
</details>

<details>
  <summary>Show Radiobutton.cs</summary>
  
```cs
public class RadioButton : MonoBehaviour
{
    public enum ButtonType { Next, PauseToggle }

    [SerializeField] Radio radio;
    [SerializeField] ButtonType buttonType;

    XRSimpleInteractable grab;

    void Awake()
    {
        grab = GetComponent<XRSimpleInteractable>();
        var rb = GetComponent<Rigidbody>();
        rb.isKinematic = true;
    }

    void OnEnable()
    {
        grab.selectEntered.AddListener(OnActivated);
    }

    void OnDisable()
    {
        grab.selectEntered.RemoveListener(OnActivated);
    }

    void OnActivated(SelectEnterEventArgs _)
    {
        if (buttonType == ButtonType.Next)
            radio.PlayNext();
        else if (buttonType == ButtonType.PauseToggle)
            radio.TogglePause();
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






