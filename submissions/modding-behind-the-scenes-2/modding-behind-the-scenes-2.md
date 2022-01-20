## Modding Behind The Scenes <author>stringweasel</author>

It's [worth repeating](https://alt-f4.blog/ALTF4-21/) that Factorio has an absolutely amazing modding community. That said, you might not realize just how much effort some modders need to go through to create their masterpieces. Especially when they start running into engine limitations. The Factorio developers do [try and expand](https://forums.factorio.com/viewforum.php?f=28) the engine to give modders more flexibility, but it's impractical to allow modders to do _everything_ they want. So, when modders reach a limit of what Factorio allows them to do, they need to get creative to achieve their vision. What they come up with are usually interesting and crazy ~~hacks~~ workarounds which the player often doesn't even notice. I always find these workarounds fascinating, so I reached out to a few modders and asked them about their favourite stories of how they bent Factorio to their will.

### Logistic Train Network: Something <author>Opterra</author>

LTN is 5 years old, remembering the more insane hacks back in 0.14 feels a bit like a grandfather complaining how easy everything is now.
Circuit Network didn't support detecting neighbours, so I had to use a signal to check for short circuited input and output. 
Accessing prototype information like wagon capacity from control stage was another thing that didn't exist until 0.15. The flying-text prototype was one of the few that could be read during control stage. I used name = "ltn-inventories["..wagon.name.."]" as index and order string for capacity. During control capacity checks tried to access a cached lookup table only reading the flying-text once. Caching often used prototypes like this is still good practice today.

### Some stories <author>GotLag</author>

In the mod Explosive Excavation, the blasting charges are simply an item that places a tile of water. To create the explosive effect, a run-time script creates a custom projectile with an explosion effect at the tile location. At the time I made the mod, projectiles required an entity as a target (rather than simply a location), so the script spawned a fish in the new water tile and then immediately blew it up. 
In a similar vein, the horn sound in Honk used to be made by an invisible explosion that did no damage and sounded like a horn. The API later had a function added to play a sound directly, so it uses that now.
The only interesting workaround I recall from Reactors was spawning an invisible dummy burner type entity for the cooling tower steam (as the cooling tower is an electric-powered assembler and those can't emit smoke). 

### <author>Pyanodon</author>
So I was asked to write an article about some of the things we modders do to create the mods you all suffer through for our amusement. Well I am going to talk today about the farms in Pyanodons AlienLife but before we get to the how I believe I should explain the why of the decisions that we made that lead us to do it this way. When kingarthur and i started working on AlienLife we wanted to add farms that would require the animals to work and similar to real life as the number of creatures in a farmed increased they would become more productive. Its easier to get more milk when you have more korlex after all. 

Now on to the how. We had a few ideas but each had some issues we needed to overcome. The farms could have used the furnace entity type with the animals as fuel. That came with the issue that they would consume the animal as it worked as at the time burners could not have waste products which was not desired. So one idea was discarded and we moved on to using looking at making the farms assembling machines. First was the idea of just adding them to the recipe but then we couldn't really make it variable as you either have enough or the recipe doesnt cycle. That wasnt want we wanted so we needed a different approach and in came the idea of using the module slots since speed modules make the machines faster with the benefit of being able to still limit animals to a given farm with module categories. This solved the problem of them consuming the animal but then created more issues the biggest of which was that even without modules the farms would work and normally an assembly machine doesn't really need a module to process a recipe cycle at a decent speed. 

Unable to solve the problem in the data stage alone we started discussing ideas of how to solve these issues of working without modules and the speed being too fast to make the need of having a full set of modules unnecessary. Starting with the data stage and the prototypes we looked into reducing the machine speed to zero.(insert screenshot of zero speed error)

The game engine wouldn't allow that option at all so we went with the next best thing. Crafting speed is a double which means it allows decimal numbers like say the assembling machine 1 zero point five crafting speed. Well taking that to the extreme we tried numbers like this. (insert crafting speed.png) yes that is a very small number. After the decimal there is thirteen zeros. Assuming you had a recipe with a one second craft time it would take thirty one thousand seventy millenniums give or take a few centuries. Have fun letting your computer run long enough to finish that craft. With a crafting speed that low we ran into the next issue. Module effects have a max possible effect cap of thirty two thousand seven hundred sixty seven percent. Way too low to work with numbers that small. Even at the max amount you would only save a few millenniums and still never finish in a human lifetime. At that point we redid the math using the effect cap as a starting point and with a lot of maths determined that the module bonus formula rewritten out looks like this. module bonus formula = crafting speed * ( 1 + ( module bonus [in decimal] * number of modules ). With the knowledge of how many module slots we wanted and how much we could cap them at we plugged in numbers till we arrived at the crafting speeds that we found would work well enough for our needs. Which in the case of the arqad farm from the earlier example came out to zero point zero four. With that same one second recipe it would take twenty five seconds to complete. Given the starting recipes for the arqads is over 100 seconds with that speed it would take almost an hour. Better but pyanodons isn't a mod for speed running it was actually fast enough people could choose to just wait it out.

Enter the second major problem and where we moved onto the control stage to solve these issues. As we now needed to disable a modueless machine we needed to start tracking them and checking them periodically to make sure they were still empty or if a module had been inserted to enable the machine to function. First every time the player places down an entity we check if that entity is one of the farms using a list of names long enough i cant fit them all on my screen at the same time. (insert check if its a farm.png)  ignore the commented out logs they are from debugging and sprinkled throughout the code. With pyal started before the tips and tricks were added we created our own version and the top copy of the entity name checker displays a how to use message the first time a farm is placed down. After that in both versions we travel up to the disable_machine function where we add the entity to a list of farms to disable it and check if it's an animal farm, plant farm or fungus farm and display one of 3 custom warning icons to show the machine is disabled. (insert icons). That takes care of the machine working without modules but now how to enable them after they are inserted.

In comes the on_tick event to cycle through all the farms to check if they have modules in them. If so they can be enabled and have the disabled icon removed or if they was active before and if the modules were removed then they can be disabled again. and in that moment a disturbance is felt as if thousands of voices all yelled out bemoaning the death of ups that cycling through all those entities would cost late game. Never fear knowing that with pyanodons mods that a mega base is all but required to finish the mod pack we took steps to avoid eating all the ups doing this one thing. (insert on tick loop.png). We use a running counter that gets updated each loop so that it only checks ten farms per tick which is small enough to go unnoticed ups wise but fast enough that the time delay late game isn't to big. Given that normally ups should be sixty ticks per second then in one minute the loop can process thirty six thousand six hundred farms per minute. That's a lot of farms even in the end game. After all that the only thing left is to breed your giant murder hornets and make some meat honey.

### <author>PreyLooZero</author>

Whats a KI core?

One main mechanic of 248k is the KI core alongside its co-workers (special beacons). 
It alows you to "brodcast" the effects of modules to linked beacons. 
Allowing massive module savings and some insane beacons stacking - eventually.
Sounds a bit OP? Well it kinda is but also not.


So what's it doing?

Basically it's a machine that feeds on a constant input of memory and computing power
and in return will power up your machines when they are in range of an linked KI beacon.
I added some ways to make the whole thing better over time so instead of just building more 
the player is rewarded to keep the thing running as long as possible - or so my idear was in the beginning.

Turns out implementing such a thing as a very inexperienced modder was kinda difficult.
But ill walk you though anyways:


Where to start?

So for me at least the first logical step was to start with a blank assembling machine and a beacon.
Just add a fixed void recipe to the assembly machine and a quick and dirty animation and the first step was complete.

After that things got - well difficult. I needed ways to link my special beacon to this exact assembling machine, 
someway for the player to pick which module effects should get brodcasted, some sort of logic to keep track of all my beacons/assembly machines
and of course the mechanic that put the effects from the main assembly machine - lets call it KI core - into my linked beacons.


A lot to todo then: 

At first I wrote a registry script for the beacons/KI cores. So whenever one of them was placed down my script would add a registry entry for it.
Getting this to work the first time was a real pain due to my own bad structure and so many edge cases 
f.e. if a core was placed by another script or a god damn biter wrecked a core and beacon at the same time things where just crashing left and right - 
so to say bugs (literally) caused a lot of problems...

But in the end this worked fine so for the next task I added a little invisible beacon which would get spawned right in the middle of the KI core.
That way the player had somewhere to put his modules and I could read them aswell.
Added the little fellow to the registry and done as well. Nice.
Almost done then eh - next was the linking problem.

These of you who have played my mod in the early stages know how I linked beacons and KI core in the beginning - eventhough it was pretty trashy tbh.
So basically I made an linker item which was spawned by the KI core and you had to manually put it in the beacon to establish a connection.
Yeah pretty impractical if you ask me so things had to change. 
Now we have a fancy GUI for the KI mechanic aswell as a selection tool to change the channels of many beacons at once!


"Das I-Tüpfelchen"

(Here you can see why it's called KI core instead of AI core.)
So to finish things I wannted the whole thing to get better over time but also make it not that OP in the early game.
How to do that? Quite simple in facorio - add some techs!
Said and done. Through research the KI core consumes less and can link to more beacons, so that early it is pretty balanced and late game gets stupid OP.

There you go sir a working KI core with some nice beacons to go.


### Share your story!

What you read above is only a handful of the stories I heard, and we will release another article with even more scary modding stories from other modders in the future. If you're a modder with an interesting ~~hacking~~ workaround story, or anything to do with Factorio, then let us know! We would love to tell the world about it. And it's important to talk about such traumatic events, because keeping it all inside is never good. It might slow down the growth of your factory, after all. Wouldn't want that.
