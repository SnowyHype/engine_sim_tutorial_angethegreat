# How to Build an Engine on AngeTheGreat's Engine Sim

## Step 1: Creating your engine file
1. Go into your current used build of the engine simulator, then into the `assets` folder then into the `engines` folder and right click to create a file.
2. Select create new text file, and name it what you want it to be named, for example, `mycustomengine.mr`

## Step 2: Using VSC or Notepad to Edit your Engine File

### Using VSC (Visual Studio Code):
Press open file and find the directory in the popup, then begin editing! Sometimes you can double click on the file and it will ask you what to open it with, choose visual studio code.

### Using Notepad:
Double click the file and it will either automatically use notepad, or it will ask you. If not, you can right click and press `Open With...` and it will ask you what program you'd like to use.

## Step 3: Your Engine Code

First and foremost, you should define everything needed such as the engine sim library, impulse response library (irlib), units, and constants.

This can be done like this:
```
import "engine_sim.mr"
impulse_response_library ir_lib()
units units()
constants constants()

label cycle((360 * 2) * units.deg)
label rpmlimit(7500 * units.rpm)
```

### __Wires__

Next you should make a wire node. Note these wires will only be used in `this file` so you can make it a `private node`. 

Here's an example for a 4 cylinder engine:
```
private node wires {
    output wire1: ignition_wire();
    output wire2: ignition_wire();
    output wire3: ignition_wire();
    output wire4: ignition_wire();
}
```

Each output defines a new wire used for your engine, 4 wires for a 4 cylinder. These will be called upon more later in the code.\
When defining your node, you should pay attention to a couple of things.

 Think, is this going to be used across multiple files?\
 What am I going to name this node?\
 What am I using this node for?

 If it's just going to be called upon by this file then you should make it a `private node`, if it's going to be called upon by another file make it a `public node`.\
 The name of the node can be really whatever you want. Generally, you cannot put a number as the first character of the name (ex. `private node 4cylinder {}` will throw an error)\
 Lastly you should know what you are going for when making your node. This engine will be made up of them, they technically are just big variables.

### __Camshafts__

You should have a camshaft node as well inside of your engine folder. Now, yes this can be stored in the `camshafts.mr` file found in the `parts-library` folder, but generally if you plan to make this engine public, you should put it in the engine file.

Your camshaft node should look something like this.
```
private node i4_camshaft {
    input lobe_profile;
    input intake_lobe_profile: lobe_profile;
    input exhaust_lobe_profile: lobe_profile;

    input lobe_separation: 114 * units.deg;
    input intake_lobe_center: lobe_separation;
    input exhaust_lobe_center: lobe_separation;
    input advance: 0 * units.deg; 
    input base_radius: 1.0 * units.inch;

    output intake_cam: _intake_cam;
    output exhaust_cam: _exhaust_cam;

    camshaft_parameters params (
        advance: advance,
        base_radius: base_radius
    )

    camshaft _intake_cam(params, lobe_profile: intake_lobe_profile)
    camshaft _exhaust_cam(params, lobe_profile: exhaust_lobe_profile)

    label rot180(180 * units.deg)
    label rot360(360 * units.deg)

    _exhaust_cam
        .add_lobe(rot360 - exhaust_lobe_center)
        .add_lobe(rot360 - exhaust_lobe_center + 3 * rot180)
        .add_lobe(rot360 - exhaust_lobe_center + 1 * rot180)
        .add_lobe(rot360 - exhaust_lobe_center + 2 * rot180)
    _intake_cam
        .add_lobe(rot360 + intake_lobe_center)
        .add_lobe(rot360 + intake_lobe_center + 3 * rot180)
        .add_lobe(rot360 + intake_lobe_center + 1 * rot180)
        .add_lobe(rot360 + intake_lobe_center + 2 * rot180)
}
```
As of right now (ver. 0.1.9a) every engine is dual overhead cam (DOHC) so you will have 2 cams per engine bank. 

As you can see, for each cam, there is 4 lobes. This is a 4 cylinder after all. Each lobe is numbered as you see them, the top one is for cylinder 1, next is for cylinder 2 and so on. The math you see for each lobe, calculates the correct angle that lobe should be when your engine `spawns`. You see `rot360`, `intake_lobe_center`, `exhaust_lobe_center`, `0123` and `rot180`. Lets go through what all of these things mean.

>`rot360`: This defines half of a 4 stroke cycle or 360°\
`intake_lobe_center` and `exhaust_lobe_center`: These define the centers or in this case the lobe separation between the intake and exhaust cam lobe angles\
`0123`: This basically sets the order of which cams move when, these should be set in the firing order of your engine. See my [guide](https://github.com/SnowyHype/angethegreat_engine_sim_guide#finding-cam-and-crank-orders) on how to set these.\
`rot180`: This is the firing interval of your motor. There is more info on this in my [guide](https://github.com/SnowyHype/angethegreat_engine_sim_guide#finding-firing-intervals).

### __Heads__

Next in line of your engine file, you should have a head node. This could also be placed into `heads.mr` as a public node, but, like cams, if you want to make this engine public you should keep it as a private node in your engine file.

Here's an example of a head node:
```
private node i4_head {
    input intake_camshaft;
    input exhaust_camshaft;
    input chamber_volume: 41.6 * units.cc;
    input intake_runner_volume: 149.6 * units.cc;
    input intake_runner_cross_section_area: 1.35 * units.inch * 1.35 * units.inch;
    input exhaust_runner_volume: 50.0 * units.cc;
    input exhaust_runner_cross_section_area: 1.25 * units.inch * 1.25 * units.inch;

    input flow_attenuation: 1.0;
    input lift_scale: 1.0;
    input flip_display: false;
    alias output __out: head;

    function intake_flow(50 * units.thou)
    intake_flow
        .add_flow_sample(0, 0)
        .add_flow_sample(50, 25)
        .add_flow_sample(100, 76)
        .add_flow_sample(150, 100)
        .add_flow_sample(200, 146)
        .add_flow_sample(250, 175)
        .add_flow_sample(300, 212)
        .add_flow_sample(350, 230)
        .add_flow_sample(400, 255)
        .add_flow_sample(450, 275)
        .add_flow_sample(500, 294)
        .add_flow_sample(550, 300)
        .add_flow_sample(600, 314)
        .add_flow_sample(650, 314)
        .add_flow_sample(700, 314)

    function exhaust_flow(50 * units.thou)
    exhaust_flow
        .add_flow_sample(0, 0)
        .add_flow_sample(50, 25)
        .add_flow_sample(100, 70)
        .add_flow_sample(150, 100)
        .add_flow_sample(200, 132)
        .add_flow_sample(250, 140)
        .add_flow_sample(300, 156)
        .add_flow_sample(350, 170)
        .add_flow_sample(400, 181)
        .add_flow_sample(450, 191)
        .add_flow_sample(500, 207)
        .add_flow_sample(550, 214)
        .add_flow_sample(600, 228)
        .add_flow_sample(650, 228)
        .add_flow_sample(700, 228)

    cylinder_head head(
        chamber_volume: chamber_volume,
        intake_runner_volume: intake_runner_volume,
        intake_runner_cross_section_area: intake_runner_cross_section_area,
        exhaust_runner_volume: exhaust_runner_volume,
        exhaust_runner_cross_section_area: exhaust_runner_cross_section_area,

        intake_camshaft: intake_camshaft,
        exhaust_camshaft: exhaust_camshaft,
        intake_port_flow: intake_flow,
        exhaust_port_flow: exhaust_flow,
        flip_display: flip_display
    )
}
```

There's really not that much here to explain, there's a couple of values that will change performance such as `chamber_volume` and `intake_runner_volume` but nothing much else.

### __Engine__

Your engine node is the heart and soul of this file and is what everything leading up to this section goes toward. In this node you'll see some things referenced in other nodes and some things that are defined within the node.

Here's an example of an engine node:
```
public node i4 {
    alias output __out: engine;

    wires wires()

    engine engine(
        name: "Inline 4",
        starter_torque: 200 * units.lb_ft,
        starter_speed: 350 * units.rpm,
        redline: rpmlimit,
        throttle_gamma: 1.5,
	fuel: fuel(
            max_turbulence_effect: 3.0,
            burning_efficiency_randomness: 0.5,
            max_burning_efficiency: 0.75)
    )

    crankshaft c0(
        throw: 38.5 * units.mm,
        flywheel_mass: 10 * units.lb,
        mass: 30 * units.lb,
        friction_torque: 5.0 * units.lb_ft,
        moment_of_inertia: 0.22986844776863666 * 0.4,
        position_x: 0.0,
        position_y: 0.0,
        tdc: 90  * units.deg
    )

    rod_journal rj0(angle: 0 * 180 * units.deg)
    rod_journal rj1(angle: 3 * 180 * units.deg)
    rod_journal rj2(angle: 1 * 180 * units.deg)
    rod_journal rj3(angle: 2 * 180 * units.deg)

    c0
        .add_rod_journal(rj0)
        .add_rod_journal(rj1)
        .add_rod_journal(rj2)
        .add_rod_journal(rj3)

    piston_parameters piston_params(
        mass: 565 * units.g,
        blowby: 0,
        compression_height: 30.5 * units.mm,
        wrist_pin_position: -1.0 * units.mm,
        displacement: -7.4 * units.cc
    )

    connecting_rod_parameters cr_params(
        mass: 508 * units.g,
        moment_of_inertia: 0.0015884918028487504,
        center_of_mass: 0.0,
        length: 122 * units.mm
    )

    cylinder_bank_parameters bank_params(
        bore: 81 * units.mm,
        deck_height: 191 * units.mm
    )

    intake intake(
        plenum_volume: 1.325 * units.L,
        plenum_cross_section_area: 20.0 * units.cm2,
        intake_flow_rate: k_carb(800.0),
        runner_flow_rate: k_carb(250.0),
        runner_length: 7.0 * units.inch,
        idle_flow_rate: k_carb(0.0),
        idle_throttle_plate_position: 0.9985,
        velocity_decay: 0.5
    )

    exhaust_system_parameters es_params(
        outlet_flow_rate: k_carb(1000.0),
        primary_tube_length: 20.0 * units.inch,
        primary_flow_rate: k_carb(200.0),
        velocity_decay: 1.0, //0.5
        volume: 50.0 * units.L
    )

    exhaust_system exhaust0(
        es_params,
        audio_volume: 0.75 * 0.1,
        impulse_response:
            impulse_response(
                filename: "../../sound-library/smooth/smooth_39.wav",
                volume: 0.01)
    )

    exhaust_system exhaust1(
        es_params,
        audio_volume: 1.0 * 0.1,
        impulse_response:
            impulse_response(
                filename: "../../sound-library/smooth/smooth_39.wav",
                volume: 0.01)
    )

    cylinder_bank b0(bank_params, angle: 0 * units.deg)
    b0
        .add_cylinder(
            piston: piston(piston_params, blowby: k_28inH2O(0.1)),
            connecting_rod: connecting_rod(cr_params),
            rod_journal: rj0,
            intake: intake,
            exhaust_system: exhaust0,
            ignition_wire: wires.wire1
        )
		.add_cylinder(
            piston: piston(piston_params, blowby: k_28inH2O(0.1)),
            connecting_rod: connecting_rod(cr_params),
            rod_journal: rj1,
            intake: intake,
            exhaust_system: exhaust1,
            ignition_wire: wires.wire2
        )
		.add_cylinder(
            piston: piston(piston_params, blowby: k_28inH2O(0.1)),
            connecting_rod: connecting_rod(cr_params),
            rod_journal: rj2,
            intake: intake,
            exhaust_system: exhaust0,
            ignition_wire: wires.wire3
        )
		.add_cylinder(
            piston: piston(piston_params, blowby: k_28inH2O(0.1)),
            connecting_rod: connecting_rod(cr_params),
            rod_journal: rj3,
            intake: intake,
            exhaust_system: exhaust0,
            ignition_wire: wires.wire4
        )

    engine
        .add_cylinder_bank(b0)

    engine.add_crankshaft(c0)

    harmonic_cam_lobe intake_lobe(
        duration_at_50_thou: 210 * units.deg,
        gamma: 1.0,
        lift: 6.9 * units.mm,
        steps: 100
    )

    harmonic_cam_lobe exhaust_lobe(
        duration_at_50_thou: 190 * units.deg,
        gamma: 1.0,
        lift: 6.5 * units.mm,
        steps: 100
    )

    i4_camshaft camshaft(
        lobe_profile: "N/A",

        intake_lobe_profile: intake_lobe,
        exhaust_lobe_profile: exhaust_lobe,
        intake_lobe_center: 116 * units.deg,
        exhaust_lobe_center: 116 * units.deg,
        base_radius: 500 * units.thou
    )

    b0.set_cylinder_head (
        i4_head(
            intake_camshaft: camshaft.intake_cam,
            exhaust_camshaft: camshaft.exhaust_cam
        )
    )

    function timing_curve(1000 * units.rpm)
    timing_curve
        .add_sample(0000 * units.rpm, 14 * units.deg)
        .add_sample(1000 * units.rpm, 14 * units.deg)
        .add_sample(2000 * units.rpm, 30 * units.deg)
        .add_sample(3000 * units.rpm, 30 * units.deg)
        .add_sample(4000 * units.rpm, 34 * units.deg)
        .add_sample(5000 * units.rpm, 34 * units.deg)
        .add_sample(6000 * units.rpm, 40 * units.deg)
        .add_sample(7000 * units.rpm, 40 * units.deg)

    ignition_module ignition_module(
        timing_curve: timing_curve,
        rev_limit: rpmlimit, 
        limiter_duration: 0.05
    )

    ignition_module
            .connect_wire(wires.wire1, (0.0/4) * cycle)
            .connect_wire(wires.wire3, (1.0/4) * cycle)
            .connect_wire(wires.wire4, (2.0/4) * cycle)
            .connect_wire(wires.wire2, (3.0/4) * cycle)

    engine.add_ignition_module(ignition_module)
} 
```

In the engine node you see a ***lot*** of new things and some re-used things. Lets start from the top

> **Wires:** This calls on the wires node defining the wires in the engine node.\
**Engine:** This defines things that make drastic changes to the engine's look, starting speed/torque, and fuel\
**Crankshaft:** This makes changes to the crankshaft and can change things like flywheel weight, crankshaft weight and more.\
**Rod Journals:** This defines the individual journals that each connecting rod bolts onto when assembled onto the crankshaft.\
**Piston Parameters:** This edits things like the displacement on top of the piston, weight, compression height etc.\
**Connecting Rod Parameters:** This edits things like length and weight of the connecting rods.\
**Cylinder Bank Parameters:** This edits the bore and deck height of each cylinder.\
**Intake:** This edits things like the plenum volume, runner volume and more.\
**Exhaust System Parameters:** This edits things like, flow rate of the exhaust, exhaust velocity decay, header volume and a few other things.\
**Exhaust System:** This defines the impuse responses. Add a couple more of these to improve the sound of the exhaust but the more exhaust systems, the more the sim *will* lag. Try to keep it under 3 or 4.\
**Cylinder Banks:** This defines each cylinder. Add multiple banks is how you create certain engines such as v engines, radial engines, and more!\
**Harmonic Cam Lobes:** This is how *most* edit their cam lobes. If you are using a VTEC valvetrain, you should have at least 4 of these in your engine node.\
**Camshaft:** This is where you take all of the camshaft parameters from the camshaft node and put them to work.\
**Timing Curve:** This is what defines the ignition timing advance at different RPM ranges.\
**Ignition Module:** This is where you connect your wires to each cylinder and set your firing order. Here you can also set a hard rev limiter and the duration of said limiter. 
>

### __Transmissions__

Transmissions are almost as important as your engine node. The sim will run without a transmission node set but it will use the original transmission meant for a 400ish hp v8. Your transmission node can be `public` or `private` but this will effect how you toggle them in your `main.mr` Generally it is easier to make them private and make public `main` node. 

Here's an example of what your transmission node should look like: 
```
public node i4_transmission {
    input max_clutch_torque: 300 * units.lb_ft;
    alias output __out:
        transmission(max_clutch_torque)
            .add_gear(3.166)
            .add_gear(2.050)
            .add_gear(1.481)
            .add_gear(1.166)
            .add_gear(0.916)
            .add_gear(0.725);
}
```

You can set the amount of torque the clutch can handle by changing the `max_clutch_torque`.\
To add more gears, add another `.add_gear()`. Keep in mind that only the last `.add_gear()` should have a semicolon (;) at the end of the line.

### __Vehicles__

Vehicles are just as important as transmissions and can be set the same way. Again though, if none is set it will set the vehicle to the original 6000lb pickup.

Here's an example of what your vehicle node should look like:
```
public node i4_vehicle {
   input mass: 1000 * units.lb;
   input diff_ratio: 4.529;
   input tire_radius: 12.65 * units.inch;

   alias output __out: vehicle;

   vehicle vehicle(
    mass: mass,
    diff_ratio: diff_ratio,
    tire_radius: tire_radius
   )
}
```

There's a bunch of variables included in this node. Most of them are not edited and kept default which are those that are not listed.\
The `mass` variable is the weight of the simulated vehicle.\
The `diff_ratio` is the final drive of the rear differential of the simulated vehicle.\
And the `tire_radius` is the radius of the wheel used on the simulated vehicle.

### __Main__
This node should be public and is what is used to set your engine, transmission, and vehicle in your `main.mr`

A main node is fairly simple. This can be used if your engine, transmission, and vehicle nodes are all public ***or*** private. 

Here's a quick example of a main node: 
```
public node main {
    set_engine(i4())
    set_vehicle(i4_vehicle())
    set_transmission(i4_transmission())
}
main()
```
> **Set Engine:** This is used to define the engine node in your `main.mr` file.\
**Set Vehicle:** This is used to define the vehicle node in your `main.mr` file.\
**Set Transmission:** This is used to define the transmission node in your `main.mr` file.
>

__**Last step is to run the `engine_sim_app.exe` file and see if you messed anything up. Nobody gets it right on the first try! :P**__
