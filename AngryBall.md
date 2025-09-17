# Angry Ball : [Back](https://github.com/sim7234/Portfolio/blob/main/README.md) <-- Click to go back.
* School solo project 2025 Jan - Feb
* Engine: Unity
* Genre: Mobile, level builder
* 1 programmer
 <td ><img width="512" height="
" src="AngryBall\LevelBuild.png"/></td>

# Goal
This project was all about learning Firebase and making something with online capabilities, so my goal was to make a Level editor that was playable on every platform that had access to the game. <br>

### How i started

After making a simple angry birds clone i started with the level builder, i began by learning how to save level data in a json format and for that i had to learn how to use Serializable classes that could store multiple variables for me. <br> <br>
 After learning how that works i had to decide what data to store, after much trial and error and thinking i got to storing a Vector4, x and y for position, z for block ID and w for the blocks rotation on 1 axis and i simply made an array for that and stored each block in the level editor in json format. <br>


Here is the script on how i handled save data.
<details>

<summary>SaveData script</summary>
        
```csharp
using System;
using UnityEngine;
using UnityEngine.SceneManagement;
using Firebase.Auth;
using Firebase.Database;
using Firebase.Extensions;

public class SaveData : MonoBehaviour
{
    private static SaveData _instance;
    public static SaveData Instance { get { return _instance; } }

    public SaveContainer saveContainerArray = new SaveContainer(new Vector4[SelectBuildPieces.maxBuildPieces], new int());

    FirebaseDatabase db;

    int selectedIndex;

    string levelName;
    string loadName;
    string fullLevelName;

    public int blockId;
    public Vector2 blockPos;
    Vector4[] saveInfo;

    public int totalObjectivesLeft;

    void Awake()
    {
        if (_instance == null)
        {
            _instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    void Start()
    {
        db = FirebaseDatabase.DefaultInstance;
    }

    public void LoadLevel(string nameToLoad)
    {
        fullLevelName = nameToLoad;
        SceneManager.LoadScene("PlayLevel");
        LoadFromFirebase(fullLevelName);
    }

    public void LoadFromFirebase(string nameToLoad)
    {
        db.RootReference.Child("levels").Child(nameToLoad).GetValueAsync().ContinueWithOnMainThread(task =>
        {
            if (task.IsCompleted)
            {
                SaveContainer data = new(new Vector4[SelectBuildPieces.maxBuildPieces], new int());

                data = JsonUtility.FromJson<SaveContainer>(task.Result.GetRawJsonValue());

                saveInfo = data.buildPosAndID;

                totalObjectivesLeft = 0;
                Score.instance.LoadHighScore(data.highScore);

                for (int i = 0; i < saveContainerArray.buildPosAndID.Length; i++)
                {
                    saveInfo[i].z = Mathf.RoundToInt(saveInfo[i].z);

                    if (saveContainerArray.buildPosAndID[i] != null && saveInfo[i] != Vector4.zero)
                    {
                        if (saveInfo[i].z == 1)
                        {
                            totalObjectivesLeft++;
                        }
                        Instantiate(SelectBuildPieces.staticBuildPieces[(int)saveInfo[i].z], new Vector2(saveInfo[i].x, saveInfo[i].y),
                            new Quaternion(Quaternion.identity.x, Quaternion.identity.y, saveInfo[i].w, Quaternion.identity.w));
                    }
                }
            }
        });
    }

    public void SaveToFirebase(Vector2 blockPos, int id, float rotation)
    {
        saveContainerArray.buildPosAndID[selectedIndex] = new Vector4(blockPos.x, blockPos.y, id, rotation);
        selectedIndex++;

        if (selectedIndex == saveContainerArray.buildPosAndID.Length)
        {
            saveContainerArray.highScore = 0;

            if (db.RootReference.Child("levels") != null)
            {
                db.RootReference.Child("levels").Child(levelName + " UserId:" + FirebaseAuth.DefaultInstance.CurrentUser.UserId).
                    SetRawJsonValueAsync(JsonUtility.ToJson(saveContainerArray, true));
            }

            selectedIndex = 0;
        }
    }

    public void SaveHighScore(int score)
    {
        saveContainerArray.highScore = score;

        db.RootReference.Child("levels").Child(WinStateAndDataHolder.selectedLevel).GetValueAsync().ContinueWithOnMainThread(task =>
        {
            if (task.IsCompleted)
            {
                SaveContainer data = new(new Vector4[SelectBuildPieces.maxBuildPieces], new int());

                data = JsonUtility.FromJson<SaveContainer>(task.Result.GetRawJsonValue());

                data.highScore = score;
                db.RootReference.Child("levels").Child(WinStateAndDataHolder.selectedLevel).
                       SetRawJsonValueAsync(JsonUtility.ToJson(data, true));

                Invoke(nameof(SwitchScene), 3f);
            }
        });
    }

    void SwitchScene()
    {
        SceneManager.LoadScene("Menu");
    }

    public void SetLevelName(string name)
    {
        levelName = name;
    }

    public void SetLoadName(string name)
    {
        loadName = name;
    }
}


[Serializable]
public class SaveContainer
{
    public Vector4[] buildPosAndID;
    public int highScore;

    public SaveContainer(Vector4[] blockPosID, int _highScore)
    {
        buildPosAndID = blockPosID;
        highScore = _highScore;
    }
}

```

</details>

<br>

Another function i spent some extra time on was the trajectory line that shows where the ball is going to land.
I stated with using physics simulations about 100 - 1000 ticks into the future to get a perfectly accurate trajectory, this was incredibly accurate and satisfying to use, it was also very laggy and came with a few bugs so i decided to scrap that.

 Instead i googled and found some math i could use for my trajectory, so i did what any great programer does, i copied and pasted it into my own script, then just tweaked it a bit to fit my needs and bam i got a working trajectory.

<details>

<summary> Trajectory line </summary>

``` CSharp

using UnityEngine;
[RequireComponent(typeof(Rigidbody2D))]
[RequireComponent(typeof(LineRenderer))]
public class Trajectory : MonoBehaviour
{
    private Rigidbody2D rb;
    private LineRenderer lineRenderer;

    private int steps = 1400;

    private Vector2[] linePos;

    private Camera cam;

    [SerializeField] private PhysicsMaterial2D PMaterial2D;
    [SerializeField] private bool doGroundBounce;

    private float defaultGravityScale;

    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
        lineRenderer = GetComponent<LineRenderer>();
        lineRenderer.positionCount = steps;
        linePos = new Vector2[steps];
        cam = Camera.main;
        defaultGravityScale = rb.gravityScale;
    }

    public void CalculateTrajectory(Vector2 totalForce)
    {
        float timeStep = Time.fixedDeltaTime / Physics2D.velocityIterations;

        Vector2 gravityAccel = Physics2D.gravity * defaultGravityScale * timeStep * timeStep;

        Vector2 velocity = totalForce;

        Vector2 pos = transform.position;

        float drag = 1f - timeStep * rb.linearDamping;
        Vector2 moveStep = velocity * timeStep;

        for (int i = 0; i < steps; i++)
        {
            moveStep += gravityAccel;
            moveStep *= drag;
            pos += moveStep;

            if (pos.x >= cam.ViewportToWorldPoint(new Vector3(1, 1f)).x)
            {
                pos.x = cam.ViewportToWorldPoint(new Vector3(0.9999f, 0.9999f)).x;
                moveStep.x *= -1;
            }

            if (pos.x < cam.ViewportToWorldPoint(new Vector3(0, 0f)).x)
            {
                pos.x = cam.ViewportToWorldPoint(new Vector3(0f, 0f)).x;
                moveStep.x *= -1;
            }

            //-4.56f is ground level
            if (pos.y <= -4.56f && doGroundBounce == true)
            {
                moveStep.y *= -1 * PMaterial2D.bounciness;
            }

            linePos[i] = pos;

            lineRenderer.SetPosition(i, linePos[i]);
        }
    }
}

```

</details>

<br>

 <td ><img width="512" height="
" src="AngryBall\Traj.gif"/></td>