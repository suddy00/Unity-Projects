Artless 
<img width="2765" height="1237" alt="Shards SS" src="https://github.com/user-attachments/assets/b990a1a7-0368-4198-beab-0046c2e5ac06" />


Player Controller:

```csharp
using TMPro;
using UnityEditor.Timeline;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.InputSystem;
using UnityEngine.InputSystem.XR;


public class PlayerController : MonoBehaviour
{
    public float speed = 5.0f;
    //Rotation speed has to be extremely fast for instantaneous direction change or there will be glitching when changing directions
    public float rotSpeed = 20000.0f; 

    private float horizontalInput;
    private float verticalInput;
    private Vector3 moveDirection;
    private Quaternion targetRotation;
    private float gravity = 9.8f;

    public TextMeshProUGUI timerText;
    public float timeRemaining = 5.0f;

    //Variable to limit the bounds of the player
    public float xzRange = 8f;

    public TextMeshProUGUI shardText;
    public int shardsCollected;
    public int shardsRemaining = 20;

    public TextMeshProUGUI gameOverText;
    private Vector3 front;

    public GameObject attackPrefab;

    //Using Rigidbody to control direction facing
    private Rigidbody rb;
    private CharacterController characterController;

    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {
        rb = GetComponent<Rigidbody>();
        characterController = GetComponent<CharacterController>();

        shardsCollected = 0;
        setShardsText();
    }

    void SetTimerText()
    {
        if (timeRemaining > 0)
        {
            timerText.text = "Time Remaining: " + Mathf.FloorToInt(timeRemaining -= Time.deltaTime).ToString();
        }
        else { timeRemaining = 0; gameOverText.text = "Game Over!"; }
    }

    // Update is called once per frame
    void Update()
    {
        horizontalInput = Input.GetAxis("Horizontal");
        verticalInput = Input.GetAxis("Vertical");

        // Camera's direction vectors
        Vector3 cameraForward = Camera.main.transform.forward;
        Vector3 cameraRight = Camera.main.transform.right;

        // Movement is based off camera view, rather than player
        // This ensures movement stays flat on the ground and prevents vertical movement
        cameraForward.y = 0f;
        cameraRight.y = 0f;
        cameraForward.Normalize();
        cameraRight.Normalize();

        // Calculate world-space movement direction relative to the camera
        moveDirection = cameraForward * verticalInput + cameraRight * horizontalInput;
        moveDirection.Normalize(); // Normalize to ensure consistent speed when moving diagonally

        // Apply movement
        if (moveDirection != Vector3.zero)
        {

            if (targetRotation != Quaternion.LookRotation(moveDirection))
            {
                // Rotate the player model to face the direction of movement
                targetRotation = Quaternion.LookRotation(moveDirection);
                transform.rotation = Quaternion.Slerp(transform.rotation, targetRotation, rotSpeed * Time.deltaTime);
            }
            else
            {
                characterController.Move(moveDirection * speed * Time.deltaTime);
            }
            // Keeps character on the ground
            characterController.Move(Vector3.down * gravity * Time.deltaTime);

        }


        if (Input.GetKeyDown(KeyCode.Space))
        {
            //Launch a projectile from the player
            Instantiate(attackPrefab, (transform.position+transform.forward), transform.rotation);
        }

    }

    private void LateUpdate()
    {
        SetTimerText();
    }

    void OnTriggerEnter(Collider other)
    {
        if (other.gameObject.CompareTag("Shard"))
        {
            other.gameObject.SetActive(false);
            updateShards(1);
        }
    }

    private void setShardsText()
    {
        shardText.text = "Shards Collected: " + shardsCollected.ToString() + "/" + shardsRemaining.ToString();

        if (shardsCollected >= shardsRemaining)
        {
            gameOverText.text = "You Win!";
        }
    }

    private void updateShards(int shards)
    {
        shardsCollected += shards;
        setShardsText();
    }
}

```

Spawn Manager: Responsible for spawning prefabs on the stage

```csharp
using UnityEngine;

public class SpawnManager : MonoBehaviour
{
    public GameObject[] spawnPrefab;
    private float spawnRangeX = 8;
    private float spawnPosZ = 8;

    private float startDelay = 2;
    private float spawnInterval = 1.5f;

    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {
        InvokeRepeating("SpawnRandomItem", startDelay, spawnInterval);
    }

    // Update is called once per frame
    void Update()
    {
    }

    void SpawnRandomItem()
    {
        int spawnItem = Random.Range(0, spawnPrefab.Length);

        //Randomly generate animal index and spawn position
        Vector3 spawnPos = new Vector3(Random.Range(-spawnRangeX, spawnRangeX), 1, Random.Range(-spawnPosZ, spawnPosZ));


        Instantiate(spawnPrefab[spawnItem], spawnPos, spawnPrefab[spawnItem].transform.rotation);
    }
}

```

Attack: Attached to attacks made to destory boxes
```csharp
using System.Collections;
using UnityEngine;


public class Attack : MonoBehaviour
{
    public GameObject[] spawnPrefab;
    public float lifetime = .01f;

    void Start()
    {
        StartCoroutine(SelfDestruct());
    }

    private IEnumerator SelfDestruct()
    {
        yield return new WaitForSeconds(lifetime);
        Destroy(gameObject);
    }

    void OnTriggerEnter(Collider other)
    {        
        if(other.tag == "Box")
        {
            SpawnRandomItem();
            Destroy(other.gameObject);
        } 
    }

    void SpawnRandomItem()
    {
            if (Random.Range(1, 1000) % 2 == 0)
            {
                int spawnItem = Random.Range(0, spawnPrefab.Length);

                //Potentially spawn item
                Instantiate(spawnPrefab[spawnItem], transform.position, spawnPrefab[spawnItem].transform.rotation);
            }
    }
}
```
