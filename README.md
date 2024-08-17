# Unity-Scripts---Effects---Aim---EffectsUnityTechnologies-
This is the scripts where all the unity game basic scripts are available - Unity Technologies Shared Scripts - Effects also included 
![image](https://github.com/user-attachments/assets/737a1c11-b8aa-4d8a-a43d-56436ddb48ca)


# DecalDestroyer
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DecalDestroyer : MonoBehaviour {

	public float lifeTime = 5.0f;

	private IEnumerator Start()
	{
		yield return new WaitForSeconds(lifeTime);
		Destroy(gameObject);
	}
}

```

# ExtinguishableFire
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;


/// <summary>
/// This simulate an extinguishable fire, 
/// </summary>
public class ExtinguishableFire : MonoBehaviour
{
    public ParticleSystem fireParticleSystem;
    public ParticleSystem smokeParticleSystem;

    protected bool m_isExtinguished;

    const float m_FireStartingTime = 2.0f;

    private void Start()
    {
        m_isExtinguished = true;

        smokeParticleSystem.Stop();
        fireParticleSystem.Stop();

        StartCoroutine(StartingFire());
    }

    public void Extinguish()
    {
        if (m_isExtinguished)
            return;

        m_isExtinguished = true;
        StartCoroutine(Extinguishing());
    }

    IEnumerator Extinguishing()
    {
        fireParticleSystem.Stop();
        smokeParticleSystem.time = 0;
        smokeParticleSystem.Play();

        float elapsedTime = 0.0f;
        while (elapsedTime < m_FireStartingTime)
        {
            float ratio = Mathf.Max(0.0f, 1.0f - (elapsedTime / m_FireStartingTime));

            fireParticleSystem.transform.localScale = Vector3.one * ratio;

            yield return null;

            elapsedTime += Time.deltaTime;
        }

        yield return new WaitForSeconds(2.0f);

        smokeParticleSystem.Stop();
        fireParticleSystem.transform.localScale = Vector3.one;

        yield return new WaitForSeconds(4.0f);

        StartCoroutine(StartingFire());
    }

    IEnumerator StartingFire()
    {
        smokeParticleSystem.Stop();
        fireParticleSystem.time = 0;
        fireParticleSystem.Play();

        float elapsedTime = 0.0f;
        while (elapsedTime < m_FireStartingTime)
        {
            float ratio = Mathf.Min(1.0f, (elapsedTime / m_FireStartingTime));

            fireParticleSystem.transform.localScale = Vector3.one * ratio;

            yield return null;

            elapsedTime += Time.deltaTime;
        }

        fireParticleSystem.transform.localScale = Vector3.one;
        m_isExtinguished = false;
    }
}
```
# GunAim
```
using UnityEngine;
using System.Collections;

public class GunAim:MonoBehaviour
{
	public int borderLeft;
	public int borderRight;
	public int borderTop;
	public int borderBottom;

	private Camera parentCamera;
	private bool isOutOfBounds;

	void Start () 
	{
		parentCamera = GetComponentInParent<Camera>();
	}

	void Update()
	{
		float mouseX = Input.mousePosition.x;
		float mouseY = Input.mousePosition.y;

		if (mouseX <= borderLeft || mouseX >= Screen.width - borderRight || mouseY <= borderBottom || mouseY >= Screen.height - borderTop) 
		{
			isOutOfBounds = true;
		} 
		else 
		{
			isOutOfBounds = false;
		}

		if (!isOutOfBounds)
		{
			transform.LookAt(parentCamera.ScreenToWorldPoint (new Vector3(mouseX, mouseY, 5.0f)));
		}
	}

	public bool GetIsOutOfBounds()
	{
		return isOutOfBounds;
	}
}

```
# GunShoot
```
using UnityEngine;
using System.Collections;
using Random = UnityEngine.Random;

public class GunShoot : MonoBehaviour {

	public float fireRate = 0.25f;										// Number in seconds which controls how often the player can fire
	public float weaponRange = 20f;										// Distance in Unity units over which the player can fire

	public Transform gunEnd;
	public ParticleSystem muzzleFlash;
	public ParticleSystem cartridgeEjection;

	public GameObject metalHitEffect;
	public GameObject sandHitEffect;
	public GameObject stoneHitEffect;
	public GameObject waterLeakEffect;
    public GameObject waterLeakExtinguishEffect;
	public GameObject[] fleshHitEffects;
	public GameObject woodHitEffect;

	private float nextFire;												// Float to store the time the player will be allowed to fire again, after firing
	private Animator anim;
	private GunAim gunAim;

	void Start () 
	{
		anim = GetComponent<Animator> ();
		gunAim = GetComponentInParent<GunAim>();
	}

	void Update () 
	{
		if (Input.GetButtonDown("Fire1") && Time.time > nextFire && !gunAim.GetIsOutOfBounds()) 
		{
			nextFire = Time.time + fireRate;
			muzzleFlash.Play();
			cartridgeEjection.Play();
			anim.SetTrigger ("Fire");

			Vector3 rayOrigin = gunEnd.position;
			RaycastHit hit;
			if (Physics.Raycast(rayOrigin, gunEnd.forward, out hit, weaponRange))
			{
				HandleHit(hit);
			}
		}
	}

	void HandleHit(RaycastHit hit)
	{
		if(hit.collider.sharedMaterial != null)
		{
			string materialName = hit.collider.sharedMaterial.name;

			switch(materialName)
			{
				case "Metal":
					SpawnDecal(hit, metalHitEffect);
					break;
				case "Sand":
					SpawnDecal(hit, sandHitEffect);
					break;
				case  "Stone":
					SpawnDecal(hit, stoneHitEffect);
					break;
				case "WaterFilled":
					SpawnDecal(hit, waterLeakEffect);
					SpawnDecal(hit, metalHitEffect);
					break;
				case "Wood":
					SpawnDecal(hit, woodHitEffect);
					break;
				case "Meat":
					SpawnDecal(hit, fleshHitEffects[Random.Range(0, fleshHitEffects.Length)]);
					break;
				case "Character":
					SpawnDecal(hit, fleshHitEffects[Random.Range(0, fleshHitEffects.Length)]);
					break;
                case "WaterFilledExtinguish":
                    SpawnDecal(hit, waterLeakExtinguishEffect);
                    SpawnDecal(hit, metalHitEffect);
                    break;
            }
		}
	}

	void SpawnDecal(RaycastHit hit, GameObject prefab)
	{
		GameObject spawnedDecal = GameObject.Instantiate(prefab, hit.point, Quaternion.LookRotation(hit.normal));
		spawnedDecal.transform.SetParent(hit.collider.transform);
	}
}
```

# ParticleCollision
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/// <summary>
/// This script demonstrate how to use the particle system collision callback.
/// The sample using it is the "Extinguish" prefab. It use a second, non displayed
/// particle system to lighten the load of collision detection.
/// </summary>
public class ParticleCollision : MonoBehaviour
{
    private List<ParticleCollisionEvent> m_CollisionEvents = new List<ParticleCollisionEvent>();
    private ParticleSystem m_ParticleSystem;


    private void Start()
    {
        m_ParticleSystem = GetComponent<ParticleSystem>();
    }


    private void OnParticleCollision(GameObject other)
    {
        int numCollisionEvents = m_ParticleSystem.GetCollisionEvents(other, m_CollisionEvents);
        for (int i = 0; i < numCollisionEvents; ++i)
        {
            var col = m_CollisionEvents[i].colliderComponent;

            var fire = col.GetComponent<ExtinguishableFire>();
            if (fire != null)
                fire.Extinguish();
        }
    }
}

```
# ParticleExamples
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[System.Serializable]
public class ParticleExamples {

	public string title;
	[TextArea]
	public string description;
	public bool isWeaponEffect;
	public GameObject particleSystemGO;
	public Vector3 particlePosition, particleRotation;
}

```
# ParticleMenu
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class ParticleMenu : MonoBehaviour {

	// our ParticleExamples class being turned into an array of things that can be referenced
	public ParticleExamples[] particleSystems;

	// the gun GameObject
	public GameObject gunGameObject;

	// a private integer to store the current position in the array
	private int currentIndex;

	// the currently shown prefab game object
	private GameObject currentGO;

	// where to spawn prefabs 
	public Transform spawnLocation;

	// references to the UI Text components
	public Text title;
	public Text description;
	public Text navigationDetails;

	// setting up the first menu item and resetting the currentIndex to ensure it's at zero
	void Start()
	{
		Navigate (0);
		currentIndex = 0;
	}


	// our public function that gets called by our menu's buttons
	public void Navigate(int i){

		// set the current position in the array to the next or previous position depending on whether i is -1 or 1, defined in our button event
		currentIndex = (particleSystems.Length + currentIndex + i) % particleSystems.Length;

		// check if there is a currentGO, if there is (if its not null), then destroy it to make space for the new one..
		if(currentGO != null)
			Destroy (currentGO);

		// ..spawn the relevant game object based on the array of potential game objects, according to the current index (position in the array)
		currentGO = Instantiate (particleSystems[currentIndex].particleSystemGO, spawnLocation.position + particleSystems[currentIndex].particlePosition, Quaternion.Euler(particleSystems[currentIndex].particleRotation)) as GameObject;

		// only activate the gun GameObject if the current effect is a weapon effect
		gunGameObject.SetActive (particleSystems[currentIndex].isWeaponEffect);

		// setup the UI texts according to the strings in the array 
		title.text = particleSystems [currentIndex].title;
		description.text = particleSystems [currentIndex].description;
		navigationDetails.text = "" + (currentIndex+1) + " out of " + particleSystems.Length.ToString();

	}
}

```

