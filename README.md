# ADF4351
ESP32 to ADF4351
The right PLL math (for 10 MHz → 1 GHz)

Using to make a CW beaken and other things.

Reference (REFIN): 10 MHz
R counter: 1 (no reference doubler/divider)
PFD (phase detector): 10 MHz
Target RFOUT: 1 GHz
ADF4351 VCO range: ~2.2–4.4 GHz → use Output Divider = 4
VCO frequency: 1 GHz × 4 = 4.000 GHz
Integer‑N operation:

Prescaler: 8/9 (required since INT ≥ 75; we’ll have 400)
INT = VCO / PFD = 4000 MHz / 10 MHz = 400
FRAC = 0, MOD = 1 (pure integer‑N)


Feedback select: Fundamental (feedback from VCO, not the divided output)
Lock Detect (MUXOUT): Digital LD (so you can read it on a GPIO)

These values are robust, low‑spurs, and easy to lock.

Practical register field choices
Below are field choices you’d set in the six 32‑bit registers (R5 → R0). I’m giving the intent for each relevant field so you can use any register calculator or library to produce the exact 32‑bit words:
R0 (INT/FRAC):

INT = 400
FRAC = 0
Control bits tag = R0

R1 (MOD / Phase / Prescaler):

MOD = 1
PHASE = 1
Prescaler = 8/9
Phase Adjust = disabled
Tag = R1

R2 (R counter / CP / LD):

R = 1
Reference doubler = off
Reference divide-by-2 = off
Charge pump current = (e.g.) 2.50 mA (typical to start; you can tune later)
Lock Detect = Digital
PD polarity = positive
CP three‑state = disabled (normal)
Double buffer = on
Tag = R2

R3 (Clock dividers / CSR / timeout):

Clock divider mode = direct
CSR, CHARGE CANCEL, ABP = defaults (CSR off, ABP = 10 ns typical)
Tag = R3

R4 (Output / Feedback / Band select):

Feedback select = Fundamental
RF divider select = ÷4
Band select clock:

If using Low band-select mode: set Band Select Clock Divider ~200 (with 10 MHz PFD, this yields 50 kHz)


RF output A = enabled
RF output power = 0 dBm (adjust to taste)
Mute till lock detect = optional (I often enable this)
Aux output = disabled
MTLD = optional
Tag = R4

R5 (Control):

Reserved per datasheet (standard value used in most examples)
Tag = R5


If you want, I can output the exact 32‑bit hex words (R5 → R0) tailored to these choices. Just say the word and I’ll post them.


ESP32 wiring (example)

CLK → GPIO 18
DATA → GPIO 23
LE → GPIO 5
CE → GPIO 4 (hold LOW at power‑up, then HIGH to enable)
MUXOUT (LD) → GPIO 34 (input, optional)
VCC → Clean 3.3 V (or per your module; cleaner = better phase noise)
GND → GND

Keep SPI traces short and add good decoupling on the PLL power pins.
