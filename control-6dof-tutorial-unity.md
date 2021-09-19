---
id: ml1-control-6dof-tutorial-unity
title: 2.1 Control 6DOF - Unity
sidebar_position: 1
date: 09/04/2021
sdk: v0.26.0
engine: Unity
description: Initialize 6 DOF for the Control, receive the position and orientation data of the Control and design a pointer like a laser beam which starts from the Control and extends 2 meters away from it.
---

**6 DOF (Six Degrees of Freedom)** in Magic Leap 1 is the ability to track the position and orientation of the Control in space. This can be used to manipulate digital content, to easily indicate and select UI elements, and to allow generic motion tracking of the user’s hand holding the Control.

!(https://www.youtube.com/embed/ZfJqkIszODI?rel=0)

## What you will learn

By the end of this tutorial, you will be able to:

- **Initialize** 6 DOF for the Control.
- **Receive** the position and orientation data of the Control.
- **Design** a pointer like a laser beam which starts from the Control and extends *2 meters* away from it.

## Prerequisites

| For older Unity Editors and earlier versions of the Lumin SDK visit the [Unity Doc Archive](unity-doc-archive) guide.   |
| ---------- |

Before starting this tutorial, complete the following:
- [Unity Setup](get-started-developing-in-unity)
- [Hello, Cube!](gsg-create-your-first-unity-app)

## Output Poses for Apps

Output values sent to apps built in Unity follow the **frame definition** illustrated below. 

![control axis unity3](https://github.com/FaustianDeal/MLController6DOF/Images/control_axis_unity3.png)

The definition is as follows:

- The origin is located at the center of the Control Touchpad surface.
- The X axis points to the right along the Touchpad surface.
- The Y axis points "up", normal to X and Z.
- The Z axis points from the origin to the Control Bumper. It is angled 21.7 degrees below the Touchpad surface.


## Setup the Scene

In this part of the tutorial, we create a **Beam** GameObject using an **Line Renderer** and a material that will be applied to it. The beam will extend from the controller.

1. Right-click under your scene's **Hierarchy** in Unity.
2. Click **Create Empty** to create an empty GameObject.
3. Rename the new GameObject as __Beam__.
4. Under the Hierarchy tab, click on the Beam GameObject then, in the **Inspector** panel, click **Add Component** > **Line Renderer**.
5. Adjust the **Color** of the **Line Renderer** to whatever you like, then right-click the **Width** value and select **Edit Key**, then set your key value to  **0.001**.
6. Under the **Project** tab, right-click on the **Assets** folder, click **Create** > **Material** to create the material to be applied to the **Line Renderer**.
7. Name your new Material **Beam**.
8. Select the **Beam** object in the **Hierarchy** and in the **Inspector** tab, in the **Line Renderer**, drag the **Beam Material** to the **Line Renderer's Materials Element 0** .

## Control C# script

Next, create a script to enable 6 DOF.  

1. Select the **Beam** GameObject, under the **Hierarchy** tab.
2. Click **Add Component** > **New Script**. 
3. Name your script as **DynamicBeam** and click **Create and Add**.
4. Open the new script, paste the code below, and save the file.
5. In the **Inspector** tab, in our new script, adjust the **Start Color** and **End Color** to values you prefer. The **Start Color** will be where our beam starts at the controller, and the **End Color** will be the color of the end of the beam as it collides with an object. **Don't forget to adjust the ALPHA values of your colors as well!**

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.XR.MagicLeap;

public class DynamicBeam : MonoBehaviour
{
    private MLInput.Controller controller;
    private LineRenderer beamLine;
    public Color startColor;
    public Color endColor;
    // Start is called before the first frame update
    void Start()
    {
        controller = MLInput.GetController(MLInput.Hand.Left);
        beamLine = GetComponent<LineRenderer>();
        beamLine.startColor = startColor;
        beamLine.endColor = endColor;
    }

    // Update is called once per frame
    void Update()
    {
        transform.position = controller.Position;
        transform.rotation = controller.Orientation;

        RaycastHit hit;
        if (Physics.Raycast(transform.position, transform.forward, out hit))
        {
            beamLine.useWorldSpace = true;
            beamLine.SetPosition(0, transform.position);
            beamLine.SetPosition(1, hit.point);
        }
        else
        {
            beamLine.useWorldSpace = true;
            beamLine.SetPosition(0, transform.position);
            beamLine.SetPosition(1, transform.forward * 5);
        }
    }
}
```
####

Below we discuss the main concepts for this script.

### Variable

MLInputController **\_controller** is used to receive the Control's input.
LineRenderer **beamLine** is our attached **Line Renderer** component.
startColor and endColor will represent our beam's color.

### Start()

In our **Start()** method we start receiving some input from the **Control**. Typically, this means using a Control held in one of the two hands. Picking the left or the right-hand doesn't matter since we use a single Control. We also assign the **beamLine** to our **Line Renderer** component, and assign the colors to our beam here.

### Update method

In our **Update()** method, we attach the Beam GameObject to the Control (same position and orientation). 

**Note** the beam will dynamically change size if it collides with another gameobject in the scene.

---

## Run Your App

Next, [preview your app through Zero Iteration](sdk-play-mode-in-unity-with-ml-remote). 

## Next Step

Up next, [Raycast in Unity](raycast-in-unity).

[![Raycast development for Magic Leap](/legacy/kn6zp1o9zgwh/gWuwDrKs1iIuE54kMTLqX/b381db53128462f91fbc34bed244f82f/raycast_development_for_magic_leap.png)](raycast-in-unity)