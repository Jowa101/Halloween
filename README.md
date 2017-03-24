# Halloween
Just Fix the problems

Player Control

using UnityEngine; using System.Collections;

public class PlayerControl : MonoBehaviour { public GameControlScript control; CharacterController controller; bool isGrounded= false; public float speed = 6.0f; public float jumpSpeed = 8.0f; public float gravity = 20.0f; private Vector3 moveDirection = Vector3.zero; private Animation animate;

//start 
void Start () {
    controller = GetComponent<CharacterController>();
    animate = GetComponent<Animation>();
}

// Update is called once per frame
void Update ()
{
    if (Input.GetKey (KeyCode.LeftArrow)) {
        moveDirection = new Vector3 (Input.GetAxis ("Horizontal"), 0, 0);
        moveDirection = transform.TransformDirection (moveDirection);
        moveDirection *= speed;
    }

    if (Input.GetKey (KeyCode.RightArrow)) {
        moveDirection = new Vector3 (Input.GetAxis ("Horizontal"), 0, 0);
        moveDirection = transform.TransformDirection (moveDirection);
        moveDirection *= speed;

        if (controller.isGrounded) {
            animate.Play ("HumanoidRun");            //play "run" animation if spacebar is not pressed
            moveDirection = new Vector3 (Input.GetAxis ("Horizontal"), 0, 0);  //get keyboard input to move in the horizontal direction
            moveDirection = transform.TransformDirection (moveDirection);  //apply this direction to the character
            moveDirection *= speed;            //increase the speed of the movement by the factor "speed" 

            if (Input.GetKey (KeyCode.Space)) {          //play "Jump" animation if character is grounded and spacebar is pressed
                animate.Stop ("HumanoidRun");
                animate.Play ("HumanoidJumpUp");
                moveDirection.y = jumpSpeed;         //add the jump height to the character
            }
            if (controller.isGrounded)           //set the flag isGrounded to true if character is grounded
            isGrounded = true;
        }

        moveDirection.y -= gravity * Time.deltaTime;       //Apply gravity  
        controller.Move (moveDirection * Time.deltaTime);      //Move the controller
    }
}

//check if the character collects the powerups or the snags
void OnTriggerEnter(Collider other)
{               
    if(other.gameObject.name == "Powerup(Clone)")
    {
        control.PowerupCollected();
    }
    else if(other.gameObject.name == "Obstacle(Clone)" && isGrounded == true)
    {
        control.AlcoholCollected();
    }

    Destroy(other.gameObject);

}
}

the game logic

using UnityEngine; using System.Collections; using UnityEngine.SceneManagement;

public class GameControlScript : MonoBehaviour {

float timeRemaining = 10;   //Pre-earned time
float timeExtension = 3f;   //time to extend by on collecting powerup
float timeDeduction = 2f;   //time to reduce, on collecting the snag
float totalTimeElapsed = 0;   
float score=0f;      //total score
public bool isGameOver = false;

public void PowerupCollected()
{
    timeRemaining += timeExtension;   //add time to the time remaining
}

public void AlcoholCollected()
{
    timeRemaining -= timeDeduction;   // deduct time
}

// Use this for initialization
void Start(){
    Time.timeScale = 1;  // set the time scale to 1, to start the game world. This is needed if you restart the game from the game over menu
}

// Update is called once per frame
void Update () { 
    if(isGameOver)     //check if isGameOver is true
        return;      //move out of the function

    totalTimeElapsed += Time.deltaTime; 
    score = totalTimeElapsed*100;  //calculate the score based on total time elapsed
    timeRemaining -= Time.deltaTime; //decrement the time remaining by 1 sec every update
    if(timeRemaining <= 0){
        isGameOver = true;    // set the isGameOver flag to true if timeRemaining is zero
    }
}

void OnGUI()
{
    //check if game is not over, if so, display the score and the time left
    if(!isGameOver)    
    {
        GUI.Label(new Rect(10, 10, Screen.width/5, Screen.height/6),"TIME LEFT: "+((int)timeRemaining).ToString());
        GUI.Label(new Rect(Screen.width-(Screen.width/6), 10, Screen.width/6, Screen.height/6), "SCORE: "+((int)score).ToString());
    }
    //if game over, display game over menu with score
    else
    {
        Time.timeScale = 0; //set the timescale to zero so as to stop the game world

        //display the final score
        GUI.Box(new Rect(Screen.width/4, Screen.height/4, Screen.width/2, Screen.height/2), "GAME OVER\nYOUR SCORE: "+(int)score);

        //restart the game on click
        if (GUI.Button(new Rect(Screen.width/4+10, Screen.height/4+Screen.height/10+10, Screen.width/2-20, Screen.height/10), "RESTART")){
            SceneManager.LoadScene (SceneManager.GetActiveScene ().buildIndex);
        }

        //load the main menu, which as of now has not been created
        if (GUI.Button(new Rect(Screen.width/4+10, Screen.height/4+2*Screen.height/10+10, Screen.width/2-20, Screen.height/10), "MAIN MENU")){
            SceneManager.LoadScene(1);
        }

        //exit the game
        if (GUI.Button(new Rect(Screen.width/4+10, Screen.height/4+3*Screen.height/10+10, Screen.width/2-20, Screen.height/10), "EXIT GAME")){
            Application.Quit();
        }
    }
}
public GameControlScript control;
}

Object spawn script

using UnityEngine; using System.Collections;

public class SpawnScript : MonoBehaviour { public GameObject obstacle; public GameObject powerup;

float timeElapsed = 0;
float spawnCycle = 0.5f;
bool spawnPowerup = true;

void Update () {
    timeElapsed += Time.deltaTime;
    if(timeElapsed > spawnCycle)
    {
        GameObject temp;
        if(spawnPowerup)
        {
            temp = (GameObject)Instantiate(powerup);
            Vector3 pos = temp.transform.position;
            temp.transform.position = new Vector3(Random.Range(-3, 4), pos.y, pos.z);
        }
        else
        {
            temp = (GameObject)Instantiate(obstacle);
            Vector3 pos = temp.transform.position;
            temp.transform.position = new Vector3(Random.Range(-3, 4), pos.y, pos.z);
        }

        timeElapsed -= spawnCycle;
        spawnPowerup = !spawnPowerup;
    }
}
}

candy spawn script

using UnityEngine; using System.Collections;

public class PowerupScript : MonoBehaviour { public float objectSpeed = -0.5f;

void Update () {
    transform.Translate(0, 0, objectSpeed);
}
}

obstacle script //i made the obstacle a bottle of alcohol for now, its the only objects i could find

using UnityEngine; using System.Collections;

public class ObstacleScript : MonoBehaviour { public float objectSpeed = -0.5f;

void Update () {
    transform.Translate(0, objectSpeed, 0);
}
}

Ground control

using UnityEngine; using System.Collections;

public class GroundControl: MonoBehaviour {

// Use this for initialization
void Start () {
}

float speed = .5f;
// Update is called once per frame
void Update () {
    float offset = Time.time * speed;                             
    GetComponent<Renderer>().material.mainTextureOffset = new Vector2(0, -offset); 
}
}
