This isn't actually a readme, sorry. 

# Bomem DA3 .SPC / .LOG header cheat sheet

Offsets are bytes from the start of the block, in decimal and in octal (octal matches the DMP dumps).
"?" means my best guess.

## Basics

- File is 512-byte blocks. Block 1 and 2 are header. Block 3 is empty, or has 128 unknown numbers in phase scans.
  Data starts at block 4 (byte 1536).
- The file can be bigger than the header's block count (SIMPLG000.TRA has 43 extra blocks of leftover junk), so go
  by the block count, not the file size.
- Numbers are PDP-11: 16-bit words, low byte first.
- Floats are DEC format, not normal IEEE. Easy trick: take the two words in order, read them as a normal 32-bit
  float, divide by 4.
- Doubles (D) are the same idea but 4 words long.

## File types

| Type code (block 1, byte 110) | What | Extension |
|---|---|---|
| 3 | Single beam (sample, reference or phase scan) | .SPC, .REF |
| 4 | Settings file, with the phase correction for the run | .SET |
| 5 | Absorbance, -log10(sample / reference) | .LOG |
| 8 | Transmittance, sample / reference as a 0 to 1 fraction | .TRA |

## Block 1

| Offset | Octal | What | Type | Example (GL1KB1 / ATM077011 / SIMPLG000.LOG) |
|---|---|---|---|---|
| 0 | 000 | Block count | int | 11 / 23 / 138 |
| 2 | 002 | Number of points | int | 1024 / 2542 / 17236 |
| 6 | 006 | First wavenumber (MNX) | double | 0 / 124.97 / 3949.53 |
| 14 | 016 | Last wavenumber (MXX) | double | 7899 / 370.09 / 12526.92 |
| 22 | 026 | Step between points | double | 7.721 / 0.0964 / 0.4977 |
| 30 | 036 | Requested range, low | double | 0 / 125 / 4000 |
| 38 | 046 | Requested range, high | double | 7899 / 370 / 12500 |
| 46 | 056 | Requested range again, low (always the same so far) | double | 0 / 125 / 4000 |
| 54 | 066 | Requested range again, high | double | 7899 / 370 / 12500 |
| 62 | 076 | Plot min amplitude (MNY) | float | 0.0144 / 0 / 0 |
| 66 | 102 | Point number of the min (counting from 1) | int | 628 / 236 / 1158 |
| 70 | 106 | Plot max amplitude (MXY) | float | 459.97 / 2.0 / 2.0 |
| 74 | 112 | Point number of the max (counting from 1) | int | 229 / 207 / 16900 |
| 78 | 116 | Resolution, cm-1 | float | 15.44 / 0.5 / 1.0 |
| 88 | 130 | Unknown, 0 on phase scans | int | 0 / 131 / 2932 |
| 92 | 134 | Number of scans asked for | int | 10 / 200 / 40 |
| 100 | 144 | Phase points? (128 = the .SET kernel length) | int | 8 / 1960 / 128 |
| 102 | 146 | Phase points again? | int | 8 / 98 / 128 |
| 110 | 156 | File type (see above) | int | 3 / 5 / 5 |
| 120 | 170 | Apodization | 8 chars | " Bartlet" / " HAMMING" / "  BOXCAR" |
| 128 | 200 | Date and time | 17 chars | "18-DEC-9112:59:57" |
| 148 | 224 | 0 on phase scans, 1.0 on everything else | float | 0 / 1 / 1 |
| 152 | 230 | Phase correction mode? 2 = phase scan, 1 = uses the .SET, 3 = other | int | 2 / 3 / 1 |
| 364 | 554 | Phase scan the run used (a phase scan names itself) | text | "SY0:QZ3QU1 .SPC" |
| 384 | 600 | Notes typed in when setting up the scan, 128 chars, space padded | text | "hello from the other side" |

- Wavenumber of point i = first + i x step (i starts at 0).
- Min/max amplitude: on single-beam files it's the data's own min and max. On LOG it's 0 to 2, on TRA 0 to 1.
- The min/max amplitude are 2-word floats, each followed by its point number. Reading them as 4-word doubles gives
  the same value but hides the point number.
- On LOG files the two point numbers are flipped, because they come from the ratio before the log.
- Resolution: on phase scans it's 2 x step (15.44 for a "16 cm-1" run). On the others it's the nominal value
  (1.0 for a 1 cm-1 run).

## Block 2 (starts at byte 512)

| Offset | Octal | What | Type | Example (GL1KB1 / ATM077011 / SIMPLG000.LOG) |
|---|---|---|---|---|
| 16 | 020 | Optics: detector, beamsplitter, source, filter? | 4 x 4 chars | " MCT KBR GL1 FI1" / " BO1 3M1 GL1 FI1" / " INA QU2 QZ1 INM" |
| 32 | 040 | Beamsplitter code? (KBr 6, Mylar 1, quartz 7) | int | 6 / 1 / 7 |
| 34 | 042 | Source code? (globar 1, quartz lamp 2) | int | 1 / 1 / 2 |
| 76 | 114 | Unknown, maybe electronics filter low (Hz)? | float | 200 / 100 / 3000 |
| 80 | 120 | Unknown, maybe electronics filter high (Hz)? | float | 10000 / 10000 / 28000 |
| 88 | 130 | 4 x HeNe laser wavenumber | float | 63192 |
| 92 | 134 | Beamsplitter range, low? | float | 450 / 125 / 8500 |
| 96 | 140 | Beamsplitter range, high? | float | 5000 / 850 / 50000 |
| 100 | 144 | Detector range, low? | float | 500 / 10 / 3800 |
| 104 | 150 | Detector range, high? | float | 5000 / 370 / 12500 |
| 114 | 162 | Instrument readings for the sample? (11 words, not a timestamp) | ints | |
| 138 | 212 | Number of scans actually done | int | 10 / 199 / 40 |
| 146 | 222 | Instrument readings for the reference? (11 words) | ints | |
| 452 | 704 | Related files: phase scan, settings (.SET), reference (.REF), 20 chars each | text | "QZ3QU1  QZ3QU1 .SET  SIMPLG000.REF" |

- Scans done vs asked for: SIMPLG000.REF asked for 40 and did 39, ATM077011 asked for 200 and did 199.
- The ranges follow the optics: both KBr files have 450-5000, the Mylar one 125-850, the bolometer 10-370.
- The two 11-word records: the .SET has an exact copy of the phase scan's, and the REF has the first one only half
  filled. They don't change with time, so they're not timestamps.

## Data (starts at byte 1536)

- One float per point, as many as the point count. The rest of the last block is zeros.
- SPC / REF = single beam, raw detector signal, no real unit.
- LOG = -log10(sample / reference) = absorbance. Confirmed: SIMPLG000.LOG matches -log10(SIMPLG000.TRA).
- TRA = transmittance as a plain fraction, 0 to 1. Not x10, not percent.
- Exactly 8.0 in a LOG file means "too dark to measure", a clip value.
- .SET: after the header, 128 values that are the phase correction kernel for the run (flat across exactly the
  requested 4000-12500 range), then 1580 more values for...something (a narrow burst near 5315 cm-1, purpose unknown).
