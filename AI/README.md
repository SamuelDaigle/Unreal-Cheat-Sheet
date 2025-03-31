[Home](../README.md) > [AI](README.md)  
# Artificial Intelligence

Best Practices : Use separate behavior trees for each enemy, even if they share behaviors. Reuse tasks and decorators to share similar behaviors.

### Behavior Tree Components
**Selectors** : Go through children from left to right looking for a successful node. If a node fails, it tries the next one. When successful, the node is completed and we can go back up the tree.

**Sequence** : Execute from left to right, until a node fails. If the node is successful, do the next one. If the node fails, go back up the tree.

**Blackboard** : Big data structure to share data between each task.

**Task (BTT)** : Press New Task top right, make logic from ‘Event Receive Execute AI’ to ‘Finish Execute’. Add an instance editable variable of type ‘Blackboard Key Selector’ to set the blackboard key. For example:
- Task FindClosestPlayer: Sets blackboard key ‘TargetLocation’.
- Task MoveTo: Uses blackboard key ‘TargetLocation.

**Decorator** : Condition on a child task. Mostly used for selector.
- Type: Blackboard (check blackboard value), Cooldown (execute at intervals)
- Observer aborts: On result change, abort self (or lower priority/task number, both) to go to the next selector child. ‘Both’ option is usually safer because it will go back to the parent and reevaluate.

### AI Controller
***AIPerception*** : Uses ‘Senses Config’ (sight, hearing) to see enemies, friendlies with config like view distance, etc.

***AIPerceptionStimuliSource*** : Only actors with this component can be seen by ‘AIPerception’ components. Set ‘Auto Register as Source’ and the senses that they emit.

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 