# Exercise-04c-Hide-and-Seek

Exercise for MSCH-C220

A demonstration of this exercise is available at [https://youtu.be/B1kE3ffBK5M](https://youtu.be/B1kE3ffBK5M).

This exercise is designed to continue our creation of a 2D Platformer, by creating and experimenting with two enemy types. The concepts behind this exercise will be outlined in class.

Fork this repository. When that process has completed, make sure that the top of the repository reads [your username]/Exercise-04c-Hide-and-Seek. *Edit the LICENSE and replace BL-MSCH-C220-F22 with your full name.* Commit your changes.

Clone the repository to a Local Path on your computer.

Open Godot. Import the project.godot file and open the "Hide and Seek" project.

You should see a very basic 2D Platformer with tiles made from the Godot Logo. The player is very similar to the one we developed in exercise 04a. *Notice it now jumps with the w character instead of the space bar.* Now, we will create some enemies for the player to contend with.

One of those enemies will be a bat that searches for the player. We will implement this using a new (to us) node: Navigation2D

In the Game.tscn scene, as a child of the Game node, create a new Navigation2D. As a child of Navigation2D, add a NavigationPolygonInstance. Select the NavigationPolygonInstance and add points to the polygon until the empty space inside the level are green. Feel free to leave a little margin around the Godot icon blocks. Add points to the polygon to exclude each of the platforms.

Now we will create a flying bat that will use the Navigation2D node to find the player. The bat will use a Raycast2D node to determine whether it has line-of-sight to the player; when it can see the player, the bat will fly faster.

Create a new Scene. Create Root Node: Other Node. Select KinematicBody2D, and rename it Bat.

As children of the Bat node, add AnimatedSprite, CollisionShape2D, Area2D, and RayCast2D nodes.

Select the RayCast2D node. In the Inspector panel, make sure Enabled is checked.

Select the AnimatedSprite. In the Inspector Panel, select Frames->New SpriteFrames. Then select Frames again, and choose Edit.

The Animate Frames panel should appear at the bottom of the window. Editing the default animation, press the Sprite Sheet icon and choose res://Assets/bat.png. All three of the images should be included in the default animation. In the Inspector panel, make sure Playing is checked.

In the Scene panel, select the Bat's CollisionShape2D. Create a CircleShape2D collision shape with a radius of 10.

In the Scene panel, select the Bat's Area2D. Add a CollisionShape2D as a child node of the Area2D. Create a CircleShape2D collision shape with a radius of 25.

Now add a script to the Bat. Save it as res://Enemy/Bat.gd:

```
extends KinematicBody2D

var player = null
onready var ray = $RayCast2D
export var speed = 350
export var looking_speed = 25
var line_of_sight = false

export var looking_color = Color(0.455,0.753,0.988,0.25)
export var los_color = Color(0.988,0.753,0.455,0.5)

var points = []
const margin = 1.5

func _ready():
	position = Vector2(2000,100)

func _physics_process(_delta):
	var velocity = Vector2.ZERO
	player = get_node_or_null("/root/Game/Player_Container/Player")
	if player != null:
		ray.cast_to = ray.to_local(player.global_position)
		var c = ray.get_collider()
		line_of_sight = false
		if c and c.name == "Player":
			line_of_sight = true
		points = get_node("/root/Game/Navigation2D").get_simple_path(get_global_position(), player.global_position, true)
		if points.size() > 1:
			var distance = points[1] - get_global_position()
			var direction = distance.normalized()
			if distance.length() > margin or points.size() > 2:
				if line_of_sight:
					velocity = direction*speed
				else:
					velocity = direction*looking_speed
			else:
				velocity = Vector2(0, 0)
			move_and_slide(velocity, Vector2(0,0))
		update()

func _draw():
	var c = looking_color
	if line_of_sight:
		c = los_color
	if points.size() > 1:
		var prev_point = get_global_position()
		for p in points:
			draw_line(p - get_global_position(), prev_point - get_global_position(), c, 2)
			prev_point = p


func _on_Area2D_body_entered(body):
	if body.name == 'Player':
		body.die()
		queue_free()


```

Select the Area2D node, and add a body_entered Signal. Connect it to the `_on_Area2D_body_entered` method in the Bat.gd script.

Save the scene as res://Enemy/Bat.tscn and close it. Open the Game scene.

Open the Enemy_Container script (res://Enemy/Enemy_Container.gd) and paste in the following code:
```
extends Node2D

onready var Bat = load("res://Enemy/Bat.tscn")

func _physics_process(_delta):
	if not has_node("Bat"):
		var bat = Bat.instance()
		add_child(bat)
		bat.name = 'Bat'

```

Test the game, and watch the bat's behavior. Notice how it flies faster when it has a clear line of sight to the player.

I have created a Turret scene that uses a state machine to cycle through three different modes: scanning, searching, and found. Instance and place four turrets in the scene as children of the Enemy_Container node (note, if the turret can see the player's starting position, it will kill it continuously). You can rotate and place them on the ceiling, walls, or floor.

Take a look at the Turret scene. Do you understand what is happening? What questions do you have? How might you want to modify it in the future?

Test the project. You should now see the turrets cycling through their modes.

Quit Godot. In GitHub desktop, add a summary message, commit your changes and push them back to GitHub. If you return to and refresh your GitHub repository page, you should now see your updated files with the time when they were changed.

Now edit the README.md file. When you have finished editing, commit your changes, and then turn in the URL of the main repository page (https://github.com/[username]/Exercise-04c-Hide-and-Seek) on Canvas.

The final state of the file should be as follows (replacing the "Created by" information with your name):
```
# Exercise-04c-Hide-and-Seek

The second exercise for the 2D Platformer project, exploring two enemy types.

## Implementation

Built using Godot 3.5

The player sprite is an adaptation of [MV Platformer Male](https://opengameart.org/content/mv-platformer-male-32x64) by MoikMellah. CC0 Licensed.

The bat sprite was originally created by bagzie for Stendhal and was [reworked by AntumDeluge](https://opengameart.org/content/bat-rework). 

The turret was created by Jason Francis and used with permission.

The pathfinding code was adapted from the good work done by FEDEOD: [https://github.com/FEDE0D/godot-pathfinding2d-demo](https://github.com/FEDE0D/godot-pathfinding2d-demo). Special thanks goes to GDQuest for their excellent pathfinding tutorial: [https://www.gdquest.com/tutorial/godot/2d/navigation2d-tilemap-pathfinding/](https://www.gdquest.com/tutorial/godot/2d/navigation2d-tilemap-pathfinding/).

## References

None

## Future Development

None

## Created by 

Jason Francis
```
