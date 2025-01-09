# Chapter 1

### **Detailed Explanation of the Reward Function**

The reward function is the heart of reinforcement learning. It translates the results of an agent's actions into numerical feedback that the agent uses to learn. In this scenario, the reward function is designed to:
1. Encourage the aircraft to reduce its distance to the target location.
2. Align its heading with the optimal trajectory toward the goal.
3. Penalize erratic heading changes to ensure smooth movement.

---

### **1. Distance Progress Reward**

#### **Purpose**
This reward encourages the aircraft to minimize its distance to the target. The closer it gets to the goal, the higher the reward.

#### **How It Works**
1. **Calculate Distance**:
   - The function calculates the Euclidean distance from the aircraft’s previous position to the goal (`previous_distance`) and from its current position to the goal (`current_distance`).

   ```python
   previous_distance = euclidean_distance(
       [previous_latitude, previous_longitude], goal_coordinates
   )
   current_distance = euclidean_distance(
       [aircraft.latitude, aircraft.longitude], goal_coordinates
   )
   ```

2. **Compute Reward**:
   - The difference between the previous and current distances determines progress:
     ```python
     distance_progress_reward = scaling_factors.get("distance") * (
         previous_distance - current_distance
     )
     ```
   - If the aircraft moves closer to the goal, the reward is positive. Moving away results in a negative reward.

3. **Scaling Factor**:
   - A scaling factor (`scaling_factors.get("distance")`) adjusts the reward’s magnitude to influence the agent's learning.

#### **Why It’s Important**
- **Encourages Goal-Oriented Behavior**: Guides the agent to prioritize moving toward the target.
- **Reward Alignment**: Reflects progress in terms of simulation objectives.

---

### **2. Heading Alignment Reward**

#### **Purpose**
This reward ensures the aircraft aligns its heading with the optimal trajectory toward the goal. Misaligned headings waste time and fuel, so this reward encourages efficiency.

#### **How It Works**
1. **Calculate Target Heading**:
   - The target heading is the angle between the aircraft’s current position and the goal:
     ```python
     target_heading = get_bearing_between_two_points(
         aircraft.latitude, aircraft.longitude, goal_coordinates[0], goal_coordinates[1]
     )
     ```

2. **Calculate Heading Difference**:
   - The difference between the aircraft’s current heading (`previous_heading`) and the target heading is normalized to a range of 0°–180°:
     ```python
     heading_difference = abs((previous_heading - target_heading + 180) % 360 - 180)
     ```

3. **Compute Reward**:
   - The reward decreases linearly with the heading difference:
     ```python
     heading_alignment_reward = (1 - heading_difference / 180.0) * scaling_factors.get("heading_align")
     ```
   - A perfectly aligned heading yields the maximum reward, while a completely opposite heading (180° off) results in no reward.

4. **Scaling Factor**:
   - A scaling factor (`scaling_factors.get("heading_align")`) adjusts the importance of alignment relative to other rewards.

#### **Why It’s Important**
- **Promotes Efficiency**: Aligning with the optimal trajectory minimizes travel time and fuel consumption.
- **Prevents Erratic Behavior**: Discourages the agent from taking unnecessary detours.

---

### **3. Heading Smoothness Penalty**

#### **Purpose**
This component penalizes erratic heading changes, encouraging the agent to maintain a smooth trajectory.

#### **How It Works**
1. **Extract Heading History**:
   - The function retrieves a history of the aircraft’s headings from its black box logs:
     ```python
     headings = [
         log_entry["heading"]
         for log_entry in aircraft.black_box._logs
         if "heading" in log_entry
     ]
     ```

2. **Calculate Smoothness**:
   - The mean squared change in heading is calculated to quantify smoothness:
     ```python
     differences = np.abs(np.diff(headings))
     differences = np.where(differences > 180, 360 - differences, differences)
     mean_squared_change = np.mean(differences**2)
     ```

3. **Compute Penalty**:
   - The penalty is proportional to the mean squared change:
     ```python
     heading_smoothness_penalty = -mean_squared_change * scaling_factors.get("heading_smoothness")
     ```

#### **Why It’s Important**
- **Encourages Stability**: Prevents abrupt heading changes that could indicate inefficient or erratic behavior.
- **Promotes Realism**: Smooth trajectories are more realistic for an aircraft.

---

### **4. Exponential Reward (Optional)**

#### **Purpose**
This optional reward component emphasizes proximity to the goal, providing higher rewards as the aircraft nears the target.

#### **How It Works**
1. **Calculate Exponential Reward**:
   - The reward exponentially increases as the distance to the goal decreases:
     ```python
     exponential_reward = np.exp(-current_distance) * scaling_factors.get("exp_progress")
     ```

2. **Incentivizing the Goal**:
   - This component makes it highly rewarding for the aircraft to approach the target quickly.

#### **Why It’s Important**
- **Fine-Tuned Precision**: Adds another layer of guidance near the goal.
- **Optional Flexibility**: Can be toggled on or off depending on the desired behavior.

---

### **5. Combining Reward Components**

The total reward is a weighted sum of all components:
```python
total_reward = (
    distance_progress_reward
    + heading_alignment_reward
    + heading_smoothness_penalty
    # + exponential_reward (optional)
)
```

Each component is scaled to balance its influence:
- **Distance**: Encourages consistent progress toward the goal.
- **Alignment**: Promotes heading efficiency.
- **Smoothness**: Discourages abrupt or erratic movements.

---

### **7. Why This Reward Function is Effective**

1. **Multi-Faceted Guidance**:
   - Combines different aspects of aircraft behavior into a single numerical feedback loop.

2. **Realism and Efficiency**:
   - Simulates real-world considerations like fuel efficiency and stable trajectories.

3. **Customizability**:
   - Developers can adjust scaling factors or toggle components (e.g., exponential reward) to fine-tune agent behavior.

4. **Debug-Friendly**:
   - Breaks the reward into understandable parts, making it easier to identify and fix issues.

---