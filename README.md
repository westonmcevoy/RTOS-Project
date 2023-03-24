# RTOS-Project
Real Time Operating Systems video game implemented using advanced ITC

Crack open the fortress, Free the prisoners!
We have found the legendary Wolfenstein Castle, where prisoners are being held on a fortified cliff top.  Our advanced artillery unit has assembled a cutting-edge firing platform for you (that includes an on-demand force-field shield and an anti-gravity sled) with a slightly-underpowered, fixed-elevation rail gun.  Unfortunately, everything was a trade-off!  We are supplying you with a virtually unlimited ammo supply and fuel for the generator, but could only acquire an underpowered generator and fewer capacitors than we’d have liked.  Either hit the castle just once (we don’t want collateral damage once it’s been breached!) or the space just below the castle enough to crack the foundation to provide escape, before the defenders destroy your platform with hand-thrown satchel charges.
Good luck, noble defenders!    I’m just going to back up a bit here to give you room to work…  Don’t mind me… I’ve just got some really important work to do back at our heavily-armored base…
Basic idea:
You will model the physics for a couple simple mechanics problems (parabolic projectiles, and a  linear-motion platform), with some “environmental/configuration” inputs that will affect the physics, general purpose inputs from our SDK affecting the simulation, and with the current physical state periodically updated to the LCD and LEDs.
One of the great advantages to our simulating physics is that we can have ideal conditions (and thereby avoid “real physics” when it would be very difficult (e.g. friction that complicates things); instant response to inputs; configurable mass, forcing capability, etc).
The basic challenge:
Using the slider, apply a force to the platform to move it left or right in an attempt to position your firing platform where you want to shoot from, or to avoid thrown satchel charges.   
Use the left button to instantaneously discharge the force field shield, using energy from the capacitors.  As long as the satchel charge is close enough to any part of the sled AND the required energy is available at that instant, the satchel will be harmlessly destroyed.
Use the right button to indicate how powerful a rail gun discharge you’d like, as long as the capacitors can provide it (start at zero when the button is pressed, and continue increasing the energy until the button is either released, or until the maximum energy is indicated).  Fire the rail gun with the maximum of the available capacitive energy or the indicated shot energy when the button is released. Rail gun ammunition will not bounce at all.  Upon first impact, it takes a chunk out of the cliff face, or sinks into the mud that the rail gun is hovering above.  Beware though of weak shots that might hit the platform!
Track how much capacitive energy is available.
The construction of the platform allows it to bounce off of the canyon walls (100% elastic, as long as it is moving slowly enough; else it is destroyed).
The satchel charges will bounce (100% elastic) off the canyon walls, but will detonate automatically when hitting any horizontal surface (the sled or ground).
Choices, and bells-and-whistles left to your discretion:
1.	Choices about game priorities when energy is not sufficient for shields and cannon, e.g.
•	Will you force an under-powered shot to retain a shield charge, once the shield requirement is buffered?
•	Will you allow the shot even if the capacitors will then be insufficient for the shield?
2.	Graphical enhancements that assist in play, e.g.
•	A status bar showing the capacitive charge, with other bars shown nearby to illustrate how much charge would be needed for your indicated rail gun power level, and for a force field pulse
•	A status bar showing the magnitude of the sled’s velocity, with an indicator of the speed at which a canyon wall collision would be catastrophic
3.	Allow modification of configurable data after defaults are loaded via some sort of menu
4.	Choose how you will represent damage to the castle, escaping prisoners, exploding satchels, shield operation, etc.
5.	Possible end-game reporting, such as:
•	Victory(castle hit)/Victory(escape complete)/Defeated(by satchel)/Defeated(by collateral damage)
•	Number of satchels thrown
•	Number of shield activations
•	Number of useful shield activations
•	Number of rail gun shots
Baseline project expectations:
•	Physics gets serviced every τPhysics in its own task.  This task will receive information about button changes (both presses AND releases will need to generate unique messages for one of the buttons, while the other will only need presses to be noted) and CapSense position (or “no position” when finger is not on it), and will communicate information that will affect the LCD and LEDs to a display task.  (Part of your work in a later week on your project may be to push up the physics update frequency to find out when you no longer have uniformly-periodic updates to physics)
o	In each time increment, you will:
	Apply force to the platform (possibly zero) and update its horizontal velocity based on the resulting acceleration, possibly reverse its horizontal velocity direction if it bounces off a canyon wall, possibly adjust its horizontal velocity for the projected impulse imparted by a rail gun shot, and update its horizontal position
	Update the satchel charge’s path based on bounces (which reverse the direction of its X-velocity) and Earth-normal gravity.
•	CapSense Slider will be sampled every τSlider
o	Commit to a number of positions (4 may have some benefit due to better discrimination), and decide how the positions map to force values within your configured limits (finger off the CapSense means no force applied to the platform)
o	Choose and justify your value of τSlider.
•	Pushbutton controls 
o	Configure the General Purpose Inputs to appropriately generate interrupts only when your design needs them.  Send information to the Physics task as soon as these transitions are noticed.
•	Left LED
o	When sufficient damage has been caused to the castle or its foundation, blink on/off with a 50% duty cycle at 1Hz to indicate that evacuation is under way. Once evacuation is complete, continuously light the LED.
•	Right LED
o	Pulse width modulated to show (via human-perceived brightness) the current force magnitude, as a % of MAX_FORCE.
•	Display Task will be serviced every τDisplay, at which point it should process any information necessary to update of the LCD.  LED controls should be updated as well, but you may have other OS mechanisms performing the low-level control of the LEDs—they are not required to be updated by the Display Task.
•	Graphics shall show satchel positions within the canyon (bounded by the cliff on the left, and the screen edge on the right), the position of the platform on the bottom of the canyon, and any rail gun shots in flight.  Only draw double lines on the left and a single line on the right edges to show the cliff and canyon, and draw only the platform at the bottom of the canyon. The picture should update every τDisplay  (As with the physics update, you may be finding out how fast you can push your solution in later weeks.)   Figure out the pixel geometry (NxM pixels) of your SDK’s LCD screen from documentation and make the canyon width (including the cliff width) and depth scale to it (see below) assuming that the pixels are physically square.  Place the origin in the center of the floor.
•	Satchel throwing: Use a uniform random number generator to determine where on the canyon floor the satchel will impact.  The satchel will be thrown with the cm/s horizontal velocity (and zero vertical velocity) to hit this point.  You must, at a minimum support the “AlwaysOne” satchel method.  If you do not support others and they are indicated by the configuration data, your code must detect this mal-configuration at runtime and indicate it.  Each of the methods is described as follows:
o	AlwaysOne: There will always be one in flight.  As soon as it is resolved, another is immediately thrown.
o	MaxInFlight: Once the first set get thrown (spaced out in time by τThrow), every time one gets destroyed or hits the ground, another is immediately thrown. 
o	PeriodicThrowTime: Regardless of how many are in-flight, new satchels will be thrown on a periodic interval, τThrow.  Therefore, depending on CanyonSize, the number will grow from zero to approximately a maximum, with random variation.
•	Configurable data to drive the game, with clear and appropriate units noted in comments for each 32b quantity, as shown below.   (Your data structure must have space set aside for each of these items in specified groupings (i.e. data structures of data structures), even if you don’t implement anything for a particular datum (e.g. StachelCharges has some elements that you may not use).  Your code should check for settings with respect to your implementation and fail if values are not supported).   The suggested default values for these (shown below, after “=”) are merely SWAGs.   Be prepared to explain your chosen values at your demo.  At Demo time, expect to be able to substitute in numbers supplied by instructional staff.

o	Data Structure Version = 1 (for the data structure shown below)
o	TauPhysics [ms] = 50
o	TauDisplay [ms] = 150 
o	TauSlider [ms] = 100
o	CanyonSize [cm] = 100000
o	Wolfenstein
	CastleHeight[cm] = 5000
	FoundationHitsRequired [-] = 2
	FoundationDepth[cm] = 5000
o	SatchelCharges
	LimitingMethod [enum: AlwaysOne=0, MaxInFlight=1, PeriodicThrowTime=2] = 0
	DisplayDiameter [pixels] = 10
	Union (usage based on LimitingMethod):
•	N/A, TauThrow, TauThrow
	Union (usage based on LimitingMethod):
•	N/A, MaxNumInFlight, N/A
o	Platform
	MaxForce [N] = 20000000
	Mass [kg] = 100
	Length [cm] = 10000
	MaxPlatformBounceSpeed [cm/s] = 50000
o	Shield
	EffectiveRange[cm] = 15000
	ActivationEnergy [kJ] = 30000
o	RailGun
	ElevationAngle [mrad] = 800
	ShotMass[kg] = 50
	ShotDisplayDiameter[pixels] = 5
o	Generator
	EnergyStorage [kJ] = 50000
	Power [kW] = 20000
