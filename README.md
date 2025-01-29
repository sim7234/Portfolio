# Portfolio
My Portfolio
<details>
<summary>Example</summary>
<pre>$ using System;
using UnityEngine;
using UnityEngine.SceneManagement;
using System.IO;
using System.Text;
using Firebase;
using Firebase.Auth;
using Firebase.Database;
using Firebase.Extensions;
using System.Collections;

public class SaveData : MonoBehaviour, ISaveObserver
{
    private static SaveData _instance;
    public static SaveData Instance { get { return _instance; } }

    // LevelSaveData[] levelDataArray = new LevelSaveData[SelectBuiildPieces.maxBuildPieces];

    public SaveContainer saveContainerArray = new SaveContainer(new Vector3[SelectBuildPieces.maxBuildPieces]);

    int selectedIndex;

    public Vector2 blockPos;
    public int blockId;

    FirebaseDatabase db;

    string levelName;
    string loadName;

    SelectBuildPieces buildScript;

    Vector3[] saveInfo;

    string fullLevelName;

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
        buildScript = FindAnyObjectByType<SelectBuildPieces>();

        Debug.Log(Application.persistentDataPath);
        if (SceneManager.GetActiveScene().name == "LevelBuilder")
        {
            SelectBuildPieces.saveObservers.Add(this);
        }
    }

    void Update()
    {
        if (Input.GetKeyUp(KeyCode.L))
        {
            LoadFromFirebase(loadName);
        }


    }

    public void LoadLevel(string nameToLoad)
    {
        fullLevelName = nameToLoad;
        SceneManager.LoadScene("PlayLevel");
        LoadFromFirebase(fullLevelName);
        //StartCoroutine(LoadSceneAsync());
    }

    public void LoadFromFirebase(string nameToLoad)
    {
        //nameToLoad + " UserId:" + FirebaseAuth.DefaultInstance.CurrentUser.UserId
        db.RootReference.Child("levels").Child(fullLevelName).GetValueAsync().ContinueWithOnMainThread(task =>
        {
            if (task.IsCompleted)
            {
                SaveContainer data = new(new Vector3[SelectBuildPieces.maxBuildPieces]);

                try
                {
                    data = JsonUtility.FromJson<SaveContainer>(task.Result.GetRawJsonValue());
                }
                catch (Exception ex)
                {
                    Debug.LogException(ex);
                }

                Debug.Log(data);

                saveInfo = data.buildPosAndID;


                for (int i = 0; i < saveContainerArray.buildPosAndID.Length; i++)
                {
                    saveInfo[i].z = Mathf.RoundToInt(saveInfo[i].z);

                    if (saveContainerArray.buildPosAndID[i] != null)
                    {
                        if (saveInfo[i] != Vector3.zero)
                        {
                            
                            try
                            {
                                Instantiate(SelectBuildPieces.staticBuildPieces[(int)saveInfo[i].z], new Vector2(saveInfo[i].x, saveInfo[i].y), Quaternion.identity);
                                //don't uncomment this debug line or the entire script will stop working without any errors :)
                                //D/O/N/'/T/ // // /!/ /?/  // Debug.Log(SelectBuildPieces.staticBuildPieces[(int)saveInfo[i].x]);
                            }
                            catch (ArgumentException ex)
                            {
                                Debug.LogError(ex);
                            }

                        }
                    }
                }
            }
        });
    }


    public void SaveLevelData(Vector2 blockPos, int id)
    {
        saveContainerArray.buildPosAndID[selectedIndex] = new Vector3(blockPos.x, blockPos.y, id);
        selectedIndex++;

        if (selectedIndex == saveContainerArray.buildPosAndID.Length)
        {

            if (db.RootReference.Child("levels") != null)
            {
                db.RootReference.Child("levels").Child(levelName + " UserId:" + FirebaseAuth.DefaultInstance.CurrentUser.UserId).SetRawJsonValueAsync(JsonUtility.ToJson(saveContainerArray, true));
            }

            SaveToFile(levelName, JsonUtility.ToJson(saveContainerArray, true));

            Debug.Log("Saved");
            selectedIndex = 0;
        }
    }

    void LoadSave(string levelNameToLoad)
    {
        SaveContainer loadedSave = JsonUtility.FromJson<SaveContainer>(LoadFromFile("Test Level"));
        Vector3[] saveInfo = loadedSave.buildPosAndID;

        for (int i = 0; i < saveContainerArray.buildPosAndID.Length; i++)
        {
            Debug.Log(saveInfo[i]);
            saveInfo[i].z = Mathf.RoundToInt(saveInfo[i].z);

            if (saveContainerArray.buildPosAndID[i] != null)
            {
                if (saveInfo[i] != Vector3.zero)
                {
                    Instantiate(SelectBuildPieces.staticBuildPieces[(int)saveInfo[i].z], new Vector2(saveInfo[i].x, saveInfo[i].y), Quaternion.identity);
                }
            }
        }
    }

    public void SaveToFile(string fileName, string jsonString)
    {
        using (var stream = File.OpenWrite(Application.persistentDataPath + '/' + fileName))
        {
            stream.SetLength(0);

            var bytes = Encoding.UTF8.GetBytes(jsonString);

            stream.Write(bytes, 0, bytes.Length);
        }
    }

    public string LoadFromFile(string fileName)
    {
        using (var stream = File.OpenText(Application.persistentDataPath + '/' + fileName))
        {
            return stream.ReadToEnd();
        }
    }

    public void SetLevelName(string name)
    {
        levelName = name;
    }

    public void SetLoadName(string name)
    {
        loadName = name;
    }

    IEnumerator LoadSceneAsync()
    {
        //if (SceneManager.GetActiveScene() == SceneManager.GetSceneByName("SelectLevel"))
        AsyncOperation aSync = SceneManager.LoadSceneAsync("PlayLevel");

        while (!aSync.isDone)
        {
            // SceneManager.LoadScene("PlayLevel");
            LoadFromFirebase(levelName);
            yield return null;
        }
    }
}


[Serializable]
public class SaveContainer
{
    public Vector3[] buildPosAndID;

    public SaveContainer(Vector3[] blockPosID)
    {
        buildPosAndID = blockPosID;
    }
}<br>a code block!</pre>
</details>
