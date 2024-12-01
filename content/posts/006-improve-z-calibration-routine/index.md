---
title: "Sovol SV08: Improve Z-Calibration Routine"
date: 2024-11-30T12:00:00
draft: false
description: "The default Z-offset calibration isn't perfect, but it's easy to tweak it. Here's how to do it."
summary: "The default Z-offset calibration isn't perfect, but it's easy to tweak it. Here's how to do it."
---

Before each print runs the Sovol SV08 calibrates its Z-offset. This is needed to make sure the extruder has the perfect height, else prints easily get messy:
* A bit too high and the filament won't stick together
* A bit low and the nozzle goes through the filament

Normally this process is either done manually or by using a [Klicky probe](https://www.printables.com/model/849409-klicky-probe-for-sovol-sv08-abl). Sovol went a different route and added [an inductive probe](https://www.sovol3d.com/products/proximity-sensor-switch-for-sv08) which finds the height by checking for the inductive field of the bed.

This is a good idea in general, but Sovol seemingly went a bit cheap on their solution. Other than the SV08s predecessors the inductive probe has issues when not at the perfect temperature. As a result printing with a freshly turned on printer is a game of luck - sometimes a print works good because the Z-offset was just right by accident, sometimes you get the spaghetti monster.

## How to solve that?

Besides modding the printer and putting in a Klicky probe or even using an adapter so you can reuse the predecessors inductive probe it's possible to just tweak the config and reduce the random factor.

The SV08 runs with Klipper which is an Open Source printer firmware. The [config reference](https://www.klipper3d.org/Config_Reference.html#probe) is very helpful. The probe is configured like this by default:

```yaml
[probe]
pin: extra_mcu:PB6    
x_offset: -17                  
y_offset: 10             
#z_offset : 0
speed: 15.0
speed: 5.0
samples: 2
sample_retract_dist: 2.0
lift_speed: 50
samples_result: average
samples_tolerance: 0.016
samples_tolerance_retries: 2
```

These are the important settings:
* `speed`: This is how fast the probe moves over the bed. With higher speeds there's more room for errors.
* `samples`: This is how many times each point is checked for height. With just 2 samples it's easy to have an outlier in there.
* `sample_retract_dist`: This is how far up the probe starts checking for height. The default of 2mm is pretty near the bed and it does not hurt to increase the value a bit so the probe can come in cleanly each time.
* `samples_result`: This is by far the biggest gamechanger. Per default this is configured as `average`, but it's also possible to set this to `median`. You need more samples to perform a median-calculation, but it's a lot more accurate as it removes outliers from the equation.
* `samples_tolerance`: This is the tolerance which gets accepted as a good result. When measured values are outside of this range the probe is considered to be miscalculated and the process restarts. You want this at a very low value, as otherwise you won't get good Z-offset measurements. Better measure often than to measure quick have a broken print in the end.
* `samples_tolerance_retries`: This is how often the printer retries probing when the sample values are outside of the given tolerance.

> *Note #1: No idea why speed is given twice, but for some reason it did not really work when I removed one of these occurences... Whatever, won't hurt to leave it just in.*

> *Note #2: Don't ask why z-offset is in, but commented out. I guess because they want the user to put in his own value? As this is all about getting a proper Z-Offset anyways I just left it commented out...*

With these in mind I changed the config to the following:

```yaml
[probe]
pin: extra_mcu:PB6    
x_offset: -17                  
y_offset: 10             
#z_offset : 0
speed: 10                     # original 15.0
speed: 10.0                   # original 5.0
samples: 3                    # original 2
sample_retract_dist: 5.0      # original 2.0
lift_speed: 25                # original 50
samples_result: median        # original average
samples_tolerance: 0.0125     # original 0.016
samples_tolerance_retries: 10 # original 2
```

I saw online that people went even lower with the `samples_tolerance` but so far it works good for me. I also think increasing the number of `samples` won't hurt, but it works good enough with a value of 3 for me.

One of the major things to look out for is not to print with a cool printer. The sensor needs to warm up before it returns usable values, so before each print I heat up the bed to 65°C and home the printing head to the center of the bed with a height of 1mm. Having it like that for some minutes definitely helps getting a good Z-offset.

For each print I also check the filament stripe that gets printed to the front of the bed before the print begins. If it's easy to pull apart I know that I can cancel the print immediately, as the Z-offset is set too high.

I hope this helps someone else as well. This info is not new and there are numerous sources out there giving similar advice, but I thought having it condensed might help someone (at least it will help future me for sure!).
