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
This system simulates realistic pouring in VR, detecting when a held object is tilted and spawning a fluid stream with sound, while spawning sauce “splat” decals on surfaces when particles collide.

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
<summary>Show Radio</summary>

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


<br>

I implemented all the soundeffects/music used in the game, some of the effects I recorded myself. <br>
<br>Some 3D modeling<br>
<br>Helped with alot of small fixes around the game<br>

<br>

<br>






