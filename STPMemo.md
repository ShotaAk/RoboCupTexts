
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
    class GlobalInfo():
        def __init__(self):
            assignmnet = {"Role_0":None,"Role_1":None,
                    "Role_2":None,"Role_3":None,
                    "Role_4":None,"Role_5":None} # assignment

            
```


```python
    
    def update(self):
        WorldModel.update()
        self.select_play()
        
        self.execute_play()
        
        self.evaluate_play()

    
    def select_play(self):
        
        if self.play_termination:

            # Extract possible plays from playbook
            possible_plays = []
            for play in PlayBook:
                if play.applicable == situation:
                    possible_plays.append(play)

            # Select a play randomly
            self.play = randomly_select(possible_plays)

            WorldModel.reset_recent_situation()
            self.play_past_time = CurrentTime()
            self.play_termination = False
            RoleAssignemnt(self.play.roles)


    def execute_play(self):
        for role in self.play.roles:
            role.behavior.run()


    def evaluate_play(self):
        for role in self.play.roles:
            status = role.behavior.get_status()

            if role.loop_enable:
                if status != TaskStatus.RUNNING:
                    role.behavior.reset()
            else:
                if status != TaskStatus.RUNNING:
                    self.play_termination = True

        if self.play.timeout:
            if CurrentTime() - self.play_past_time > self.play.timeout:
                self.play_termination = True

        if WorldModel.situations[self.play.done] or \
                not WorldModel.situations[self.play.done_aborted] or \
                WorldModel.recent_situations[self.play.recent_done] or \
                not WorldModel.recent_situations[self.play.recent_done_aborted]:

                self.play_termination = True
```

```python
    class Play():
        def __init__(self, name):
            self.name = name
            self.applicable = None
            

```

```python
    class Role():
        def __init__(self, name):
            self.behavior = Sequence(name) # name like a "Role_0".
            
            

```

```python

    class Skill(Task):
        def __init__(self, name, robot_id):
            super(Skill, self).__init__(name)
            self.robot_id = robot_id

        def run(self):
            pass



```


