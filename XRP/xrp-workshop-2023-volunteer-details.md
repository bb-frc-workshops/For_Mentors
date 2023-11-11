# XRP Workshop Volunteer Details

Thank you for helping us support growing the interest and knowledge of our younger generation in STEM. For the XRP Workshop, your main duties will be helping the kids/teams with their programming (Java based). Below is some information that should help guide you through that process. If you are ever in doubt, please reach out to one of the head mentors (Zhiquan Yeo or Joe Pokorny) and they can provide next steps.

## Get to know the hardware

We are using the [XRP (Experiential Robotics Platform)](https://www.sparkfun.com/products/22230). This comes with a chassis, wheels, roller balls, drivetrain motors (and encoders for them), one servo motor with arm, color sensor, range finder, and a raspberry pi pico W microcontroller. We will also supply 4 AA Batteries and a usb cord to connect to the laptop (for robot program uploading).

## Steps for setting up and connecting the XRP

Physically build the robot using the steps here (note the video is really useful): https://xrpusersguide.readthedocs.io/en/latest/course/building.html

In order to upload your code, you will need to follow the instructions here:
https://docs.wpilib.org/en/latest/docs/xrp-robot/index.html

Most important is to hold “BOOTSEL” button, then press and release “RESET” button. This should enter the XRP into upload mode. Drag and drop the uf2 file (https://github.com/wpilibsuite/xrp-wpilib-firmware/releases/download/v1.0.0/xrp-wpilib-firmware-1.0.0.uf2) onto the robot. Once this is done, remove the usb cable, and cycle the robot’s power. A new access point should show in your list of networks starting with XRP (if in doubt, enter upload mode with USB cable connected to your laptop and open xrp-status.txt on the PICODISK drive. The text file will have the exact SSID).

Note: The latest release of the firmware can always be found at https://github.com/wpilibsuite/xrp-wpilib-firmware/releases.

## Programming

We will be using Java and a style called command-based programming, see here:
https://docs.wpilib.org/en/2022/docs/software/commandbased/index.html
The XRPReference example is in this style already and we plan to build upon it.

There are several classes in WPILib that you can leverage. You can find the documentation on them here:
https://first.wpi.edu/wpilib/allwpilib/docs/release/java/edu/wpi/first/wpilibj/package-summary.html

And the complete list here:
https://first.wpi.edu/wpilib/allwpilib/docs/release/java/index.html

Note, because the XRP has limited hardware capabilities, we have special classes for our motors (XRPMotor) and servos (XRPServo).

Our suggestion is to leverage the example (XRPReference documented steps to obtain it here https://docs.wpilib.org/en/latest/docs/xrp-robot/programming-xrp.html) as a starting point. 

Note we leverage Command-Based Programming (more on that here
https://docs.wpilib.org/en/stable/docs/software/commandbased/index.html). Command-Based Programming is based on the premise that you have a set of commands that you want to take on some subsystems (hardware). For example, you may want a robot to drive forward for 5 seconds. In command-based programming you would construct a “drive forward” command that could take time as an input. This command would act on the subsystem drivetrain which is the hardware that will be used to move the robot. This can then be extended to have a sequence of commands (drive forward, turn right, drive forward, etc) or commands in parallel which act on various parts of your hardware (drive forward and lift arm concurrently for example). Note, a subsystem can only act on one command at a time (or in other words, only one command can have control of the subsystem at a time). Additionally, many more examples are given in the above link if you’d like to learn more.

### General Robot Program Structure

**RobotContainer.java** - Where you will instantiate the subsystem(s) and specify the default command(s) for each.

**Robot.java** - skeleton of the different robot states you can be in based on the Mode that is active (Disabled, Teleoperated, Autonomous, or Test). There is an init and periodic for each. The init is run only once when you enter that Mode where the periodic is repeatedly ran in a loop while remaining in that state. You can specify the command(s) you want to run for each mode (note, cancelling a command restores the default to that subsystem).

**subsystem folder** - where all your “hardware” components will be. For example, Drivetrain.java represents the interfacing with the hardware for your wheels. It is recommended that the students create a new subsystem here if they decided to add an actuator (Arm.java, Lifter.java, Intake.java, etc)

**commands folder** - where all the commands you want to provide to the robot are. These commands can be independent or done as a grouping. For example, AutonomousDistance.java shows how you can create a set of commands to be run in sequence.

## Troubleshooting

1. **When in doubt, try changing the batteries** - (4 AAs on bottom of XRP). New batteries solve a lot of issues that just don’t make sense (including spotty or dropped wifi connection).

2. **Simulation build error - “WPIHaljni” dependency path not found** - 
We are still working on this. If you see it, please call a head mentor over to capture the details. In the meantime, the workaround is to not simulate from the option in the drop-down, but rather after the failure, in the terminal in VS Code, type: `./gradlew simulateJavaRelease`

3. **Simulation build error - All others** - Something is wrong with the Java code. Please investigate the error outputs/stack trace

4. **Robot not moving**
    - First check to see if the robot is connected using http://192.168.42.1:5000
        - If you see a webpage, you can reach your robot
    - Ensure that these are set in `build.gradle`
        - `wpi.sim.envVar("HALSIMXRP_HOST", "192.168.42.1")`
        - `wpi.sim.addXRPClient().defaultEnabled = true`
    - Ensure you are using `Simulate Robot Code`
    - In the Simulator GUI, check that you are in `Teleoperated` mode
    - In the Simulator GUI, check that your Joystick is mapped to `Joystick[0]`. If it says `Unassigned`, drag and drop the Logitech/XBox controller from the `System Joysticks` on the left side of the screen
    - In the Simulator GUI, verify that pressing buttons and moving joystick/gamepad axes do in fact light up in your `Joystick[0]` area. If not, try unplugging and switching the game interface mode on the back of the controller from `X` to `D` (or vice versa). This switch changes the button/axis mapping for the controller
    - Verify the code. Did you map the buttons correctly in code (most likely in `RobotContainer` or in a command file)? Did you make sure that the command is executed (either directly or via a defaultCommand)? If it is to run in a loop continuously, did you ensure that your command has an overridden `isFinished()` method returning false (like `ArcadeDrive`)?
    - Make sure you are using `XRPMotor` for your motors and not `Spark` or any "standard" motor controllers for the roboRIO
    - See point #1 again

5. **Servo not moving** -
    - Make sure you are using the `XRPServo` class and not the regular `Servo` class
    - Make sure that the physical servo is connected to the XRP board (black wire goes to Ground)
    - Make sure the exact servo physical channel is specified correctly on object construction (either channel 4 or 5)
    - Make sure to set the default command of the servo/arm subsystem is set up correctly in `RobotContainer`
    - Make sure the servo/arm command `isFinished()` method returns false so that it is always running
    - If using the `setAngle()` method from the example, make sure to use values between 0 and 180 (degrees)
    - See point #1 again

## FMS and Match Play

When in "match play", all XRPs and controlling computers are connected to a Field Management System (FMS) network. By default, the FMS network name is `XRP-FMS-NET` with a password of `xrp-fms-network`. The actual FMS server is located at `192.168.0.2`, and all connected devices will be given an IP address in the range of `192.168.0.10 - 192.168.0.199`.

Some configuration modifications will need to be made:

In the `build.gradle` file, the following lines need to be added/replaced.
```
wpi.sim.envVar("HALSIMWS_HOST", "192.168.0.2")
wpi.sim.envVar("HALSIMWS_URI", "/xrp-fms-blue2") // This should match whichever station you are assigned to
wpi.sim.envVar("HALSIMWS_FILTERS", "DriverStation")
wpi.sim.addWebsocketsClient().defaultEnabled = true

//Sets the XRP Client Host
wpi.sim.envVar("HALSIMXRP_HOST", "192.168.0.2")
wpi.sim.addXRPClient().defaultEnabled = true
```
Note that the `HALSIMWS_URI` should match whichever driver station position you are assigned to. Valid options here are `/xrp-fms-blue1`, `/xrp-fms-blue2`, `/xrp-fms-red1` or `/xrp-fms-red2`.

On the XRP, the WiFi set up should be configured so that:
* You add the `XRP-FMS-NET` network (and password) to the network list
* Switch the WiFi Mode to `STA` (instead of `AP`)

When in match play and connected to the FMS, you should not need to manually switch between disabled, autonomous or teleop. Instead, the FMS will signal to your robot code and the XRP which mode it should be in. Additionally, in the "Driver Station" section of the simulation GUI, you should also see the assigned station reflected.
