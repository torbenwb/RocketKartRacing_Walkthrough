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

## Car Functionality: Longitudinal Force

```cs
using UnityEngine;

public class Player : MonoBehaviour
{
    [SerializeField] Car car;

    // Update is called once per frame
    void Update()
    {
        if (!car) return;
        car.Drive(Input.GetAxisRaw("Vertical"));
        car.Turn(Input.GetAxisRaw("Horizontal"));
        car.Brake(Input.GetKey(KeyCode.LeftShift) ? 1f : 0f);
    }
}
```

```cs
using UnityEngine;
using System.Collections.Generic;

public class Car : MonoBehaviour
{
    private Rigidbody rigidbody; // Rigidbody to apply forces to
    private float driveAxis, brakeAxis, turnAxis; // Save valid input values from public interface
    private bool grounded = false;

    [Header("Suspension")]

    [SerializeField] List<Transform> wheels;

    [Tooltip("Radius used for wheel raycasts.")]
    [Range(0.1f, 1f)]
    [SerializeField] float wheelRadius = 0.4f;

    [Tooltip("Spring force constant k. Applies upwards spring force proportional to wheel vertical offset.")]
    [Range(50f, 250f)]
    [SerializeField] float springStrength = 100f;

    [Tooltip("Spring damping value. Damps spring force proportional to point velocity.")]
    [Range(1f, 5f)]
    [SerializeField] float springDamping = 3f;

    [Header("Acceleration")]

    [Tooltip("Max longitudinal force output. Force output is proportional to (1 - (currentSpeed / maxSpeed)).")]
    [Range(15f, 35f)]
    [SerializeField] float maxSpeed = 25f;

    [Tooltip("Longitudinal friction coefficient. Used to apply oppositional longitudinal force proportional to velocity.")]
    [Range(1f, 5f)]
    [SerializeField] float longitudinalFriction = 2f;

    # region Public Interface
    /*  Accepts and validates external drive input.
        Clamps driveAxis between -1 and 1.
    */
    public void Drive(float driveAxis){
        this.driveAxis = Mathf.Clamp(driveAxis, -1f , 1f);
    }

    /*  Accepts and validates external braking input.
        Clamps brakeAxis between 0 and 1.
    */
    public void Brake(float brakeAxis){
        this.brakeAxis = Mathf.Clamp(brakeAxis, 0f, 1f);
    }

    /*  Accepts and validates external turn input.
        Clamps turn axis between -1 and 1.
    */
    public void Turn(float turnAxis){
        this.turnAxis = Mathf.Clamp(turnAxis, -1f, 1f);
    }
    #endregion

    #region MonoBehaviour Life Cycle
    private void Awake()
    {
        rigidbody = GetComponent<Rigidbody>();
    }

    private void FixedUpdate()
    {
        ApplySuspensionForce();

        if (!grounded) return;

        ApplyLongitudinalForce();
    }
    #endregion

    #region Forces
    private void ApplySuspensionForce()
    {
        bool tempGrounded = false;

        foreach(Transform wheel in wheels)
        {
            Vector3 origin = wheel.position;
            Vector3 direction = -wheel.up;
            RaycastHit hit;
            float offset = 0f;

            if (Physics.Raycast(origin,direction,out hit, wheelRadius)){
                tempGrounded = true;

                Vector3 end = origin + (direction * wheelRadius);
                offset = (end - hit.point).magnitude;

                float pointVelocity = Vector3.Dot(wheel.up, rigidbody.GetPointVelocity(wheel.position));
                float suspensionForce = (springStrength * offset) + (-pointVelocity * springDamping);
                rigidbody.AddForceAtPosition(wheel.up * suspensionForce, wheel.position);
            }
        }

        grounded = tempGrounded;
    }

    
    private void ApplyLongitudinalForce()
    {
        float forwardVelocity = Vector3.Dot(transform.forward, rigidbody.velocity);
        float maxSpeedRatio = Mathf.Abs(forwardVelocity) / maxSpeed;

        Vector3 force;

        if (Mathf.Abs(driveAxis) > 0f){
            force = transform.forward * driveAxis * maxSpeed * (1 - maxSpeedRatio);
        }
        else{
            force = transform.forward * -forwardVelocity * longitudinalFriction;
        }

        rigidbody.AddForce(force);
    }
    #endregion
}

```
