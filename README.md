# RocketKartRacing_Walkthrough

## OOP: Class Design

```cs
using UnityEngine;

public class Car : MonoBehaviour
{
    private float driveAxis, brakeAxis, turnAxis;

    public void Drive(float driveAxis){
        this.driveAxis = Mathf.Clamp(driveAxis, -1f , 1f);
    }

    public void Brake(float brakeAxis){
        this.brakeAxis = Mathf.Clamp(brakeAxis, 0f, 1f);
    }

    public void Turn(float turnAxis){
        this.turnAxis = Mathf.Clamp(turnAxis, -1f, 1f);
    }
}

```

## Car Functionality: Suspension

```cs
using UnityEngine;
using System.Collections.Generic;

public class Car : MonoBehaviour
{
    private Rigidbody rigidbody;
    private float driveAxis, brakeAxis, turnAxis;
    [SerializeField] List<Transform> wheels;
    [SerializeField] float wheelRadius = 0.4f;
    [SerializeField] float springStrength = 100f;
    [SerializeField] float springDamping = 3f;

    public void Drive(float driveAxis){
        this.driveAxis = Mathf.Clamp(driveAxis, -1f , 1f);
    }

    public void Brake(float brakeAxis){
        this.brakeAxis = Mathf.Clamp(brakeAxis, 0f, 1f);
    }

    public void Turn(float turnAxis){
        this.turnAxis = Mathf.Clamp(turnAxis, -1f, 1f);
    }

    private void Awake()
    {
        rigidbody = GetComponent<Rigidbody>();
    }

    private void FixedUpdate()
    {
        ApplySuspension();
    }

    private void ApplySuspension()
    {
        foreach(Transform wheel in wheels)
        {
            Vector3 origin = wheel.position;
            Vector3 direction = -wheel.up;
            RaycastHit hit;
            float offset = 0f;

            if (Physics.Raycast(origin,direction,out hit, wheelRadius)){
                Vector3 end = origin + (direction * wheelRadius);
                offset = (end - hit.point).magnitude;

                float pointVelocity = Vector3.Dot(wheel.up, rigidbody.GetPointVelocity(wheel.position));
                float suspensionForce = (springStrength * offset) + (-pointVelocity * springDamping);
                rigidbody.AddForceAtPosition(wheel.up * suspensionForce, wheel.position);
            }
        }
    }
}

```
