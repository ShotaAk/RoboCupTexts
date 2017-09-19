
# Definiton

## Play
- Applicable
- Done (Situation が一致した時にDone)
- Done_aborted (Situation が違うときにDone)
- Recent Done (Recent Situation が一致した時にDone)
- Recent Done_aborted (Recent Situationが違うときにDone)
- Timeout
- Roles

## Role
- Behavior Tree (Tasks) (Sequence)
- Loop Enable
- Task Parameters (set to Tasks)

## Tactic
- Robot ID
- Tactic Parameters
- Behavior Tree (Skills)
- Skill Parameters (set to Skills)

## Skill
- Robot ID
- Command
- Behavior Node (leaf)
- Global Info (only reference)
- Skill Parameters

## Command
- Target Pose (x, y, w)
- Aim (x, y)
- Target velocity (vx, vy, vw)
- Velocity Control Enable
- Kick Power
- Chip Enable
- Dribble Enable
- Navigation Enable



```python


    
    def update():
        WorldModel.update()
        self.select_play()
        
        result = self.execute_play()
        
        self.evaluate_play(result)
    
    def select_play():
        situation = WorldModel.situation
        recent_situation = WorldModel.recent_situation
        
        if self.play.done == situation or \
                self.play.done_aborted != situation or \
                self.play.recent_done == recent_situation or \
                self.play.recent_done_aborted != recent_situation:

            # Extract possible plays from playbook
            possible_plays = []
            for play in PlayBook:
                if play.applicable == situation:
                    possible_plays.append(play)

            # Select a play randomly
            self.play = randomly_select(possible_plays)

            WorldModel.reset_recent_situation()

    def execute_play():


```

```python

    class Skill(Task):
        def __init__(self, name, robot_id):
            super(Skill, self).__init__(name)
            self.robot_id = robot_id

        def run(self):
            pass



```


