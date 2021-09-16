---
id: ml1-raycast-in-unity
title: 2.2 Control Raycast - Unity
sidebar_position: 2
date: 09/04/2020
sdk: v0.24.1
engine: Unity
description: Learn how to write a Unity app to cast rays to find fast intersections with the world geometry. Raycast is a feature of World Reconstruction for low-latency determination of surface intersections from the world reconstruction model.
---

In this guide we cast rays from the center of our camera. When a ray hits a surface, we create one cube at that position. Additionally, we orient the cube depending on the direction (normal) of the surface we hit.

## What will you learn?

This guide is for those who want the fastest path to see the __Raycast__ feature working. 

### Prerequisites

| For older Unity Editors and earlier versions of the Lumin SDK visit the [Unity Doc Archive](unity-doc-archive) guide.   |
| ---------- |

Before starting this tutorial, complete the following:
- [Unity Setup](get-started-developing-in-unity)
- [Control Input](control-6dof-tutorial-unity)

## Scene Setup

Let's setup our scene.

1. Under the Assets folder, click __Create__ > __C# Script__.
2. Name the new script as **Raycast**.
3. Click __GameObject__ > __Create Empty__.
4. Rename the new GameObject as **Raycast** in the Inspector tab.
5. Drag and drop the **Raycast** script on to the **Raycast** GameObject.
6. Click __GameObject__ > __3D Object__ > __Cube__.
7. In the Transform property, set the Cube's *Position* and *Rotation* to (0, 0, 0) and *Scale* to (0.05, 0.05, 0.05).
8. Drag the Cube from the Hierarchy tab to the __Assets__ folder in the Project tab. This creates a Cube prefab within our project.
9. Back in the Hierarchy, delete the Cube instance (in blue text). We will create multiple instances of the Cube prefab later on (at runtime).

![Cube](/legacy/kn6zp1o9zgwh/1jFrJXFiGC2cMeEUQs8MqC/0415fb0bebbc99f4bcce11d0a617b318/cube.jpg)

## Raycast Script Setup

Double-click on our **Raycast** C# script inside the __Assets__ folder of the Project tab. This opens it up into your chosen IDE.

__Note:__ The full script is available at the bottom of this page.

### Add the Magic Leap Namespace

To get access to the Magic Leap SDK APIs, add the following namespace to the script underneath the __using UnityEngine;__ line.

```C#
using UnityEngine.XR.MagicLeap;
```

### Variables

Make two public variables at the top of our MLRaycast class.

- The first will be used as a reference for our Cube prefab.
- The second will be used for a reference to our Controller's transform. We will use the controller's transform as the location to shoot our Raycast from.

```C#
    public GameObject prefab;    // Cube prefab
    public Transform controllerTransform; //controller gameobject in scene
```

### NormalMaker()

The __NormalMaker__ method will be used to instantiate and rotate a new cube exactly where the raycast hits. If the prefab is already instantiated, we just move the prefab to our raycast hit location.

```C#
    // Instantiate the prefab if one doesn't exist at the given point, otherwise move the prefab to the point
    // Rotate the prefab to match given normal.
    private void NormalMarker(Vector3 point, Vector3 normal)
    {
        Quaternion rotation = Quaternion.FromToRotation(Vector3.up, normal);
        if (!prefabCreated)
        {
            cubeObject = Instantiate(prefab, point, rotation);
            prefabCreated = true;
        }
        cubeObject.transform.position = point;
        cubeObject.transform.rotation = rotation;
    }
```

### HandleOnReceiveRaycast()

Below our NormalMaker, create a method to handle a Raycast __result__ that takes in as parameters:
- a MLWorldRaycastResultState
- two Vector3s
- a float

```C#
    // Use a callback to know when to run the NormalMaker() function.
    void HandleOnReceiveRaycast(MLRaycast.ResultState state, UnityEngine.Vector3 point, Vector3 normal, float confidence)
    {
        if (state == MLRaycast.ResultState.HitObserved)
        {
            NormalMarker(point, normal);
        }
    }
```

## Update()

Finally, we tie all this together with the __Update()__ method. 

We do the following:
- First, we only raycast when the trigger is pulled. We check for a trigger pull against **controller.TriggerValue**
- Create a raycast parameter. 
- Adjust some properties. 
- Feed them into our raycast results request.
- Finally, if the trigger is released, allow use to pull the trigger again at a later time by resetting our **triggerPulled** bool
This keeps us from raycasting unnecessarily every frame. 
 
Add the following lines inside.

```C#
    void Update()
    {
        //Perform the raycast if we're pulling the trigger and the trigger isn't already pulled
        if ((controller.TriggerValue > 0.8f) && !triggerPulled)
        {
            //Get the orientation of the controller
            controllerTransform.rotation = controller.Orientation;
            // Create a raycast parameters variable
            MLRaycast.QueryParams _raycastParams = new MLRaycast.QueryParams
            {
                // Update the parameters with our controller's transform
                Position = controller.Position,
                Direction = controllerTransform.forward,
                UpVector = controllerTransform.up,
                // Provide a size of our raycasting array (1x1)
                Width = 1,
                Height = 1
            };
            // Feed our modified raycast parameters and handler to our raycast request
            MLRaycast.Raycast(_raycastParams, HandleOnReceiveRaycast);
            triggerPulled = true;
        }
        //Trigger is released, reset our bool so we can raycast again
        if((controller.TriggerValue < 0.2f))
        {
            triggerPulled = false;
        }
    }
```
### Save your progress

Be sure to __save__ what you've done in Raycast.cs.

## Back in Unity

Now that our MLRaycast script is setup, lets go ahead and add a reference to our scene's __Main Camera__ transform and a reference to our Cube prefab.

With MLRaycast selected in the Hierarchy tab, do the following:

1. Click and drag whatever game object you'd like to use as a controller model into the public Transform variable.
2. In the Assets folder, click and drag the Cube prefab into our public prefab variable.

Save the scene by clicking **File** > **Save Scenes**.

![Script](/legacy/kn6zp1o9zgwh/laWAjG9R4xL73DTdktpJ4/c002c293e9ed26e016fbfeeb2f904dcd/script.png)

---

## Run your app

Next, [preview your app through Zero Iteration](sdk-play-mode-in-unity-with-ml-remote).

**Note on Raycasting using the Simulator of ML Remote**: The Interaction window of Magic Leap Remote must load a Virtual Room in order to simulate Raycast.

## Next Step

Up next, [Control Buttons](control-buttons-unity).

[![Control Buttons development for Magic Leap](/legacy/kn6zp1o9zgwh/6SHyWRGGSrE2IRnYXUm15Y/6569d660e90700a3d4eddac85d3086b0/control_buttons_development_for_magic_leap.png)](control-buttons-unity)

### Conclusion

You have learned how to use the Lumin SDK Raycast API to get access to world reconstruction data generated by Magic Leap 1.

## Raycast.cs Full Script

Use this as a cross-reference to what you went through above or to simply copy-paste from.

```C#
using System.Collections;
using UnityEngine;
using UnityEngine.XR.MagicLeap;

public class Raycast : MonoBehaviour
{
    public GameObject prefab;    // Cube prefab
    public Transform controllerTransform; //controller gameobject in scene
    private bool prefabCreated = false; //Did we already make a cube?
    private bool triggerPulled = false; //Is the trigger currently pulled?
    private GameObject cubeObject; //instantiated cube
    private MLInput.Controller controller; //ML Controller


    private void Start()
    {
        controller = MLInput.GetController(MLInput.Hand.Left);
    }

    // Update is called once per frame
    void Update()
    {
        //Perform the raycast if we're pulling the trigger and the trigger isn't already pulled
        if ((controller.TriggerValue > 0.8f) && !triggerPulled)
        {
            //Get the orientation of the controller
            controllerTransform.rotation = controller.Orientation;
            // Create a raycast parameters variable
            MLRaycast.QueryParams _raycastParams = new MLRaycast.QueryParams
            {
                // Update the parameters with our controller's transform
                Position = controller.Position,
                Direction = controllerTransform.forward,
                UpVector = controllerTransform.up,
                // Provide a size of our raycasting array (1x1)
                Width = 1,
                Height = 1
            };
            // Feed our modified raycast parameters and handler to our raycast request
            MLRaycast.Raycast(_raycastParams, HandleOnReceiveRaycast);
            triggerPulled = true;
        }
        //Trigger is released, reset our bool so we can raycast again
        if((controller.TriggerValue < 0.2f))
        {
            triggerPulled = false;
        }
    }

    // Instantiate the prefab if one doesn't exist at the given point, otherwise move the prefab to the point
    // Rotate the prefab to match given normal.
    private void NormalMarker(Vector3 point, Vector3 normal)
    {
        Quaternion rotation = Quaternion.FromToRotation(Vector3.up, normal);
        if (!prefabCreated)
        {
            cubeObject = Instantiate(prefab, point, rotation);
            prefabCreated = true;
        }
        cubeObject.transform.position = point;
        cubeObject.transform.rotation = rotation;
    }

    // Use a callback to know when to run the NormalMaker() function.
    void HandleOnReceiveRaycast(MLRaycast.ResultState state, UnityEngine.Vector3 point, Vector3 normal, float confidence)
    {
        if (state == MLRaycast.ResultState.HitObserved)
        {
            NormalMarker(point, normal);
        }
    }
}
```