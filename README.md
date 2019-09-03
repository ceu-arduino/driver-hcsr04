# HC-SR04 - Ultrasonic Sensor

Ultrasonic ranging sensor:

https://www.sparkfun.com/products/13959

## API

### Macros

These macros must be defined before including the driver:

```
#define HCSR04_ECHO     <fill-with-input-pin>
#define HCSR04_TRIGGER  <fill-with-output-pin>
```

*NOTE: The current driver only supports a single device.*

### Includes

```
#include "hcsr04.ceu"
```

### Code Abstractions

#### HCSR04_Read

Measures the distance from an obstacle:

```
code/await HCSR04_Read (var int?, var int?) -> int?;
```

Parameters:

- `int`: energy port used to turn on and off the sensor
- `int`: wake up time. Time, in ms, the driver waits for the circuit to stabilize

Return:

- `int?`: the distance found (if any)

The measures takes at most `25ms`. If passing a energy port as first parameter, the measure takes at most `25 + 80`, where `80` is the default wake up time.

The recommended period between measures is `60ms`.

#### HCSR04_Init

Turns on the Ultrasonic Sensor

```
code/await HCSR04_Init (var int energyPort, var int? time) -> NEVER;
```

Parameters:

- `int`: energy port used to turn on and off the sensor
- `int`: wake up time. Time, in ms, the driver waits for the circuit to stabilize

Return:

- `NEVER`: never returns

## Examples

### Periodic measures

Measures the distance continuously with a period of at least `60ms`:

```
#include "out.ceu"
#include "int0.ceu"
#include "wclock.ceu"

output high/low OUT_07;
output high/low OUT_13;

#define HCSR04_ECHO    INT0
#define HCSR04_TRIGGER OUT_07
#include "hcsr04.ceu"

loop do
    var int? d = await HCSR04_Read(_,_);
    emit OUT_13(d? and ((d! <= 30) as high/low));
    await 60ms;     // minimum recommended time between measures
end
```

### Single measure (turning the Ultrasonic Sensor on and off)
With Ultrasonic Sensor VCC connect to pin 8 of Arduino UNO

```
#include "out.ceu"
#include "int0.ceu"

output high/low OUT_07;
output high/low OUT_13;

#define HCSR04_ECHO    INT0
#define HCSR04_TRIGGER OUT_07
#include "hcsr04.ceu"

var int? d = await HCSR04_Read(8,_);
emit OUT_13(d? and ((d! <= 30) as high/low));

await FOREVER;
```

### Periodic measures (turning the Ultrasonic Sensor on)
```
#include "out.ceu"
#include "int0.ceu"
#include "wclock.ceu"

output high/low OUT_07;
output high/low OUT_13;

#define HCSR04_ECHO    INT0
#define HCSR04_TRIGGER OUT_07
#include "hcsr04.ceu"

spawn HCSR04_Init(8,_);

loop do
    var int? d = await HCSR04_Read(_,_);
    emit OUT_13(d? and ((d! <= 30) as high/low));
    await 60ms;     // minimum recommended time between measures
end
```