
# Creality CP-01 3D Printer with Klipper
Since no one else is providing support, I decided to take matters into my own hands.

## Before You Start
If you don’t already own a 3D printer, do not buy this model. Creality has discontinued all support for the CP-01, and you won’t even find it listed in their own slicer software. If you already own one and need help modifying it, this repository will be useful.
Make sure you have a Raspberry Pi or another single-board computer (SBC) ready to use as the host.

I must say, if your hobby is fixing or modifying 3D printers, Creality is one of the best brands you can find. You'll end up spending more time repairing it than actually printing STL files.

## Flashing Klipper
Flashing Klipper onto your printer is straightforward: simply select avrmega2560 as the target. The challenging part is creating a configuration file for your printer. Unfortunately, there are no pre-made configurations for the CP-01 available online. However, you can now use the configuration I’ve provided: CP-01 Config ([configs/printer-cp01.cfg](configs/printer-cp01.cfg)).

## Bed Leveling
If you’re new to Klipper, note that there is no screen interface for bed leveling. Instead, you’ll need to use the ```BED_SCREWS_ADJUST``` command.
For more details, refer to the official Klipper documentation: Manual Leveling.

## TMC Driver Feature
It’s recommended to solder a jumper wire to enable this feature. Klipper prints models at higher speeds, which can increase pressure on the extruder and potentially cause the TMC driver to crash. If you’re running in standalone mode, you won’t receive any error reports; the printer will continue idling, and you may notice the extruder stops extruding material. For more information, see this GitHub issue: Klipper Issue [#3738](https://github.com/Klipper3d/klipper/issues/3738#issuecomment-757184030).
![CP-01 UART](images/UART.png)

## Measuring Resonances

![X](images/shaper_calibrate_x.webp)
![Y](images/shaper_calibrate_y.webp)
I’ve measured the resonances on my own CP-01 and used the data to configure the input shaper. I’m not sure if these measurements will work for your machine, but I’ve included them in the configuration. If you’re not satisfied or have your own ADXL345 accelerometer, you can measure the resonances yourself.


## Pressure Advance
Unfortunately, the CP-01 cannot enable the Pressure Advance feature. The Creality Silent Board v2.5.2 uses TMC2208 drivers, and it’s unclear whether the MCU or the drivers are unable to support this feature.
There are several reports about this issue:

- Klipper Issue [#870](https://github.com/Klipper3d/klipper/issues/870)
- Marlin Issue [#20688](https://github.com/MarlinFirmware/Marlin/issues/20688)

In my experience, even when the system doesn’t crash, the extruder behaves erratically and often over-extrudes.

If you know how to enable this feature and without any quality issue, please let me know. I really wannt know how to enable without any quality issue.


## Extruder Issues
### Stepper

The default CP-01 extruder stepper is outdated and difficult to source, making direct replacement challenging. If your setup lacks the TMC UART feature, replacing the stepper can be risky. If the replacement stepper has a lower resistance, the TMC driver will detect it and issue a warning. If you haven't soldered a jumper wire to enable the UART function, it's best to skip this step to avoid damaging the driver.

If you decide to replace the stepper, use a multimeter to verify the wiring of the original stepper. Adjust the wiring order as needed to ensure proper functionality.

The stock extruder is highly inefficient. Replacing the extruder stepper can significantly improve the printer's maximum acceleration (`max_accel`) and maximum velocity (`max_velocity`). While the XYZ axes can handle higher speeds, the extruder remains a bottleneck. The most effective solution is to replace the entire extruder rather than just upgrading its stepper motor.

### TMC Driver failed

```
TMC 'extruder' reports error: DRV_STATUS: 40130020 s2vsb=1(ShortToSupply_B!) cs_actual=19 stealth=1
```

No matter whether it's ShortToSupply_B or ShortToSupply_A, the issue occurs randomly in layer 2 or layer 3. I have no idea why! All I know is that disabling `Pressure Advance` and disable `stealthchop` can improve the situation, but it doesn't completely prevent it from happening.

According characpter `Pressure Advance` 's github issue, uddenly stopping the stepper motor generates a strong back EMF. Since stealthchop mode introduces a delay, it may amplify this back EMF even further. The TMC driver detects this back EMF and misinterprets it as a short circuit. However, I don’t understand why this issue always occurs with the extruder. I just have to say—the CP-01's extruder is garbage! The worst extruder ever.

### The Peculiar Fan
After a prolonged period of TMC driver failures (which I'll simply refer to as crashes), I finally identified the root cause of ShortToSupply_A/B. Whenever the PWM fan was activated, it emitted a sharp noise, and print failures consistently occurred above the third layer. Then, in a moment of unexpected insight while adjusting slicer settings, everything suddenly clicked!
To pinpoint the exact cause, I disabled all features and halted printing. First, I preheated the nozzle and set the fan's PWM to 10%, which immediately triggered the noise—but the machine was not under heavy load. When I pressed "extrude," something bizarre happened—Klipper crashed without warning! This completely contradicted my initial assumption that high load was overwhelming the power supply or the MCU’s CPU resources. Clearly, that wasn’t the issue at all.

So, I adjusted the slicer settings to enforce 100% minimum fan speed, meaning the fan could now only be 0% or 100%. With this setup, I successfully completed a 150mm/s print with excellent quality—no crashes, and none of that annoying electrical noise (though, of course, the fan itself is still loud).
That means my previous hypothesis was incorrect. The problem wasn't with the extruder, but rather the mainboard—though this revelation unfortunately doesn’t absolve the extruder of its infamous reputation as the worst extruder ever.
I suspect that with the fan locked at 100%, I might now be able to enable Pressure Advance or StealthChop—though I haven’t tested that yet. But for now, things are running smoothly, and I can finally take a break from fixing my 3D printer. Time to print some parts and get back to my journey!

### PWM Cause crash
After many research, I totaly confirm all of crash just because the pwm fan causing the gnd voltage not stable thus cause the tmc driver failed (EMI). The normal way to prevent EMI is split gnd. The Easy way is use EBB36.

## Extruder

![new_extruder.png](images/new_extruder.png)

Printer now can running on 100mm/s with good quality.