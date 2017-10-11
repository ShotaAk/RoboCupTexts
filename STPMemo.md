
# References
- [STP: SKills, Tactics and Plays](https://pdfs.semanticscholar.org/5087/460f31babc3fbafafddbb480f216ea72832e.pdf)


# ルール
- RoleとRobot_IDの変換はWorldModel内のAssignmentのみで行う
- SkillはRoleを頼りに見方ロボットの情報を得る

# Definiton

## WorldModel
- Assignment[6]
- Commands[6]
- ロボットとボールの位置情報を供給できる何か


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

## Assignment

```python
    unassigned_roles, unassigned_ids = check_unassignment(assignment, existing_friends_id)

    numerical_assignment(deleted_roles, unassigned_ids)
    

```


```python
    class WorldModel():
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

        if (self.play.done and WorldModel.situations[self.play.done]) or \
                (self.play.done_aborted and 
                        not WorldModel.situations[self.play.done_aborted]) or \
                (self.play.recent_done and 
                        WorldModel.recent_situations[self.play.recent_done]) or \
                (self.play.recent_done_aborted and 
                        not WorldModel.recent_situations[self.play.recent_done_aborted]):

                self.play_termination = True
```

``` python
    # in playbook file

    class PlayHalt(Play):
        def __init__(self):
            super(PlayHalt, self).__init__("PlayHalt")

            self.applicable = "HALT" # RefereeがHALTなら実行する
            self.done_aborted = "HALT" # RefereeがHALTじゃなくなったらやめる

            for i in range(6):
                self.roles[i].loop_enable = True
                self.roles[i].behavior.add_child(
                        Tactics.Halt("Halt", self.roles[i].my_role))

    
    class PlayStop(Play):
        def __init__(self):
            super(PlayStop, self).__init__("PlayStop")

            self.applicable = "STOP" # RefereeがSTOPなら実行する
            self.done_aborted = "STOP" # RefereeがSTOPじゃなくなったらやめる

            self.roles[0].loop_enable = True
            self.roles[0].behavior.add_child(
                    Tactics.Goalie(
                        "StopGoalie", 
                        self.roles[0].my_role))

            self.roles[1].loop_enable = True
            self.roles[1].behavior.add_child(
                    Tactics.StopCenter(
                        "StopCenter", 
                        self.roles[1].my_role))

            self.roles[2].loop_enable = True
            self.roles[2].behavior.add_child(
                    Tactics.StopLeft(
                        "StopLeft",
                        self.roles[2].my_role))

    class PlayOneAttack(Play):
        def __init__(self):
            super(PlayOneAttack, self).__init__("PlayOneAttack")

            self.applicable = "OFFENSE" # RefereeがOFFENSEなら実行する
            self.done_aborted = "OFFENSE" # RefereeがOFFENSEじゃなくなったらやめる
            self.recent_done = "BALL_KICKED" # ボールが蹴られたらやめる

            self.timeout = 15 # 15秒たったらやめる
            


            self.roles[0].loop_enable = True
            self.roles[0].behavior.add_child(
                    Tactics.Goalie(
                        "Goalie",
                        self.roles[0].my_role))

            # role_2はloop_enableしないので、Shootが成功or失敗するとPlayがおわる
            self.roles[1].behavior.add_child( # ボールまで移動する
                    Tactics.DriveToBall(
                        "DriveToBall",
                        self.roles[1].my_role))
            self.roles[1].behavior.add_child( # Shootする
                    Tactics.Shoot(
                        "DriveToBall",
                        self.roles[1].my_role,
                        "Aim")) # ゴールを狙う NoAim: とりあえずける Deflect: 跳ね返す


            self.roles[2].behavior.add_child(
                    Tactics.Defend(
                        "Defend",
                        self.roles[2].my_role,
                        Coordinate() # Coordinateっていう素晴らしいクラスを作る予定
                        )



                        
```

```python
    class Play():
        def __init__(self, name):
            self.name = name
            self.applicable = None
            self.done = None
            self.done_aborted = None
            self.recent_done = None
            self.recent_done_aborted = None
            self.timeout = None
            self.roles = [
                    Role("Role_0"), Role("Role_1"), Role("Role_2"),
                    Role("Role_3"), Role("Role_4"), Role("Role_5")]

```

```python
    class Role():
        def __init__(self, my_role):
            self.behavior = Sequence(my_role) # my_role is  like a "Role_0".
            self.loop_enable = False
            
            

```

```python
    # Tactcについては何も縛りを設けなくて良い
    class Tactic():
        def __init__(self, name, my_role):
            self.my_role = my_role

```

```python

    # Abstract Base Classを使うよ
    class Skill(Task):
        def __init__(self, name, my_role):
            super(Skill, self).__init__(name)
            self.my_role = my_role

        def run(self):
            pass



```


