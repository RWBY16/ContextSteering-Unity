# ContextSteering Unity
A research project regarding "context steering behaviors" for use in games with a practical implemention in the Unity game engine.

## Introduction
Steering behaviors are movement algorithms that determine where an AI agent should be next.
These algorithms use basic information about the AI agent (current position, velocity, direcetion, ...) and the world to make a decision on where to go next.
The steering behavior will calculate a direction vector to adjust the movement of the AI agent.

These simple steeringbehaviors can then be combined in combined steering behaviors the create more complex movement and make the agent seem more intelligent

## The need for context steering
The need for context steering arises when players are able to inspect individual AI agents and observe them closesy avoiding collision with other agents and with the static world. The more steering behaviors are combined the harder it will become for the developer to tune the parameters of each and every one of those steering behaviors to achieve the behavior needed. This could also possibly mean that the behavior components itself will grow in size and become tightly coupled. These tightly coupled behaviors can then cause problems in terms of maintainability of the codebase.

Context steering combines small context behaviors that can be combinded together without tightly coupling them.
## Context steering overview
Think of context steering as a steering behavior that wants to go in a certain amount of directions equally divided over a circle.
If it was just this, the agent would stand still because all directions have an equal length thus they all apply the same amount of force to the agent negating eachother. We alter the force applied by these directions by the desire of the behavior to go in a certain direction. 
These scalar values of how desired or undesired a certain direction is are stored in Context maps.

### Context maps
Each context behavior has 2 context maps, an interest context map and a danger context map. The context steeringbehavior uses the interst map to represent its desire to go into a certain direction while the danger map represents the oposite.

For example a chase or seek context behavior will fill the slots of the interest map with higher scaler values relative the amount the corresponding direction of the slot is pointing in the same direction as the direction vector to the target of the chase behavior (think Dot product).

An avoid context steering behavior will do the exact oposite this behavior will fill the dangermap with scalar values.
Each slot of the danger map again corresponds to a direction the agent can move in and the value in the slot itself represents the behaviors desire to NOT go into that direction.

Keep in mind that there should always be an equal amount of slots in both the interest map and danger map as there are directions the agent can move in.

### Context Merger
The context merger will gather all interest and danger maps from all context behaviors active on the AI agent and merge them together to get to a final direction vector result to move the agent with.

#### How the context maps are merged

First all context maps are gathered by the context merger to build a final interest and danger map to calculate the final direction.

For both the interest and the danger map, the merger loops over all slots and picks the highest value it can find for that particular slot from all its corresponding maps (interest maps for final interest map, danger maps for final danger maps). We could also calculate the average to have for example even less of a desire to move to a spot where 2 avoid targets are but this is unneccesary because the avoidance of the first obstacle will already keep us safe from the obstacle behind it.

When we then have calculated both the final interest and danger map we subtract each interest slot of our final interest map by its corresponding slot in the final danger map. This way the interests towards our goal are altered if there is and obstacle on our path.

## Implementation
This section will describe the implemetion of the context steering in the unity application.
### Context Merger
#### Memeber variables
````
    [SerializeField] private int m_MapResolution;
    [SerializeField] private float m_MovementSpeed;

    [SerializeField] private BaseContextBehavior[] m_Behaviors;

    private List<Vector2> m_Directions;
    private List<float> m_InterestMap;
    private List<float> m_DangerMap;
    Rigidbody2D m_Rigibody2D;
````
- m_MapResolution: Determines the amount of directions used for calculating the final direction, also determines size of the interest and danger maps
- m_MovementSpeed: Movement speed of the agent
- m_Behaviors: Array of all Context behaviors associated to this agent
- m_Directions: list of direction vectors
- m_InterestMap: final interest map
- m_DangerMap: final danger map

#### Initializing the directions
````
    void InitializeDirections()
    {
        float twoPi = Mathf.PI * 2;
        float directionInterval = twoPi / m_MapResolution;
        for (int i = 0; i < m_MapResolution; i++)
        {
            float currentAngle = i * directionInterval;

            m_Directions.Add(new Vector2(Mathf.Cos(currentAngle), Mathf.Sin(currentAngle)));

        }

        m_InterestMap = new List<float>(new float[m_Directions.Count]);
        m_DangerMap = new List<float>(new float[m_Directions.Count]);

    }
````

This function initializes all directions equally divided on a circle these are the directions that will be altered by the desires from the context maps



## Result
### ContextSteering behaviour using Chase and avoid
![ContextSteeringPathFinding](https://user-images.githubusercontent.com/41028126/151200242-e4261247-d152-46fb-8299-14b755f4c060.gif)

### ContextSteering behaviour Directional steering and avoid
![ContextSteeringRacing](https://user-images.githubusercontent.com/41028126/151201575-8f0ae3fe-27a4-4245-b2f1-cb11f022bc0a.gif)

## Conclusion / Future Work

### Usage in games
## Sources
