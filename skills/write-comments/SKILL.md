---
name: write-comments
description: Write concise, beginner-friendly documentation comments for embedded C/C++ module headers and functions. Use when documenting an interface, adding call examples, explaining ranges, units, return values, hardware mapping, or usage limits. Suitable for ADC, frequency, LCD, EEPROM, GPIO, PWM, DAC, and similar modules.
---

# Write Comments

## Goal

Help a beginner understand what a module does, where it is connected, what the framework already initializes, where to call it, what data it returns, and when it must not be used.

Use short explanations, real calls, and end-of-line annotations. Do not turn a header file into a long tutorial. The generated project comments may be Chinese when the project documentation is Chinese; this Skill file and its metadata are English for tool compatibility.

## Verify Before Writing

1. Read the target header and implementation. Check initialization, call sites, timers, clocks, and hardware mapping when needed.
2. Base every statement on the current code. Do not guess ranges, units, return values, or pin assignments from a function name.
3. Distinguish the C type range, the driver accepted or returned range, the hardware operating range, and measured accuracy. Never present an unverified range or accuracy as a guarantee.
4. If the user requests comments only, change comments only. Do not change interfaces, drivers, configuration, or application logic, and do not add features.

## Comment Structure

Use this order for the module-level comment; omit sections that do not apply:

1. Purpose: one sentence describing the input and output.
2. Hardware mapping: channel, component, pin, jumper, and required wiring.
3. Framework-provided work: initialization, periodic update, interrupt capture, or cached values already handled.
4. Common calls: one real call per line, with type, range, and unit at the end.
5. One small example: show the task or function where it belongs and when it runs.
6. Important limits: invalid arguments, stale data, failure behavior, safety limits, and call-context restrictions.

Keep a normal module comment around 20–40 lines. Add detail only when the module is genuinely complex. Function declarations usually need one to three lines; do not repeat the complete module explanation before every function.

## Interface Requirements

- Explain valid argument values and which physical item each channel represents. Distinguish a software channel number from a timer or peripheral channel number.
- Explain return type, effective range, unit, and meaning. For example, frequency is in Hz and duty cycle is 0–100%, not 0–1.
- Explain whether the return value is data or status. Describe HAL_OK, HAL_ERROR, HAL_TIMEOUT, and 1/0 separately.
- State what happens when data is invalid: returns zero, keeps the old value, or leaves output arguments unchanged.
- State whether a read returns a cache, clears an event, waits for new data, or starts a conversion.
- For write APIs, explain allowed values, clamping behavior, when to write, and why repeated writes are unsafe.
- State refresh periods, measurement resolution, and timeout behavior when they matter. Do not confuse float decimal digits with measurement accuracy.

## Example Rules

- Use only interfaces that actually exist in the project. Never invent a function.
- Prefer: type value = Function(argument); followed by a short explanation of meaning, range, and unit.
- Use matching variable types and the correct ampersand for output arguments. Explain this only when useful.
- Keep examples short and suitable for the current framework. Say where the example belongs; do not insert application logic unless requested.
- Check return values when needed, and use results only after the operation succeeds.
- Give one small example by default. Add more only when the user asks for more types or cases.
- Explain only concepts that are easy to misunderstand, such as output addresses, sizeof, bytes versus displayed characters, or cached data.

## Storage Module Rules

For EEPROM, Flash, or similar storage modules, include a compact table of type, byte count, and matching read/write functions. Use non-overlapping example addresses.

State byte counts based on the current target and toolchain; do not claim that every C platform has the same sizes. For this STM32 project, float is a 32-bit value and occupies 4 bytes; this has nothing to do with the number of decimal digits.

Count the terminating null character in C strings. Do not infer byte count from the number of Chinese characters. State that reads and writes must use the same type, address, and length, that storage should not be written every millisecond, and that successful communication does not prove stored content is valid.

## Style Example

Verify the actual pins, ranges, and initialization in the target project before using this example:

~~~c
/*
Potentiometer / ADC: converts input voltage to a digital value. Calibration and initialization are already done.
Channel 1: R37, J11 installed, PB15 / ADC2_IN15.
ADC_Proc() updates the cache every 20 ms; the application only reads the cache:
  uint16_t raw = Analog_GetRaw(1);        // raw value 0–4095
  uint16_t mv  = Analog_GetMillivolts(1); // 0–3300, unit: mV
  float v      = Analog_GetVoltage(1);    // 0.000–3.300, unit: V
  uint8_t p    = Analog_GetPercent(1);    // 0–100, unit: %
  if (Analog_IsValid()) { ... }           // use only after a valid 3-channel update
Small example, in UART_Task() every 500 ms:
  UART_SendFloat(Analog_GetVoltage(1));
  UART_SendString(" V\r\n");
Channel must be 1, 2, or 3. An invalid channel returns 0.
Before the first successful sample, the cache is 0; check Analog_IsValid().
The default reference is 3300 mV and may be adjusted to the measured VDDA.
Only 0–VDDA may be applied; never connect 5 V. Update in the main loop, not an interrupt.
*/
~~~

## Final Checklist

- Can the reader understand how to call the interface and the returned type, range, and unit without opening the implementation?
- Is it clear whether zero means valid data or invalid data?
- Were channel mappings, pins, timing, and limits verified instead of guessed?
- Is the example short and relevant, without unrelated tutorial text?
- When comments only were requested, was only comment text changed?
- Do not claim physical hardware validation when only source inspection or compilation was performed.
- Report the changed path and main additions briefly; do not paste the entire comment back to the user.
