This isn't actually a readme, sorry. 

# Bomem DA3 .SPC / .LOG header cheat sheet

Offsets are bytes from the start of the block, in decimal and in octal (octal matches the DMP dumps).
"?" means my best guess.

## Basics

- File is 512-byte blocks. Block 1 and 2 are header. Block 3 is empty, or has some unknown extra numbers in SPC files. Data starts at block 4 (byte 1536).
- Numbers are PDP-11: 16-bit words, low byte first.
- Floats are DEC format, not normal IEEE. Easy trick: take the two words in order, read them as a normal 32-bit
  float, divide by 4.
- Doubles (D) are the same idea but 4 words long.

## Block 1

| Offset | Octal | What | Type | Example (GL1KB1 / ATM077011 / K17X001) |
|---|---|---|---|---|
| 0 | 000 | File size in blocks | int | 11 / 23 / 127 |
| 2 | 002 | Number of points | int | 1024 / 2542 / 15808 |
| 6 | 006 | First wavenumber (MNX) | double | 0 / 124.97 / 442.29 |
| 14 | 016 | Last wavenumber (MXX) | double | 7899 / 370.09 / 2005.63 |
| 22 | 026 | Step between points | double | 7.721 / 0.0964 / 0.0989 |
| 30 | 036 | Range limit 1, low (display?) | double | 0 / 125 / 450 |
| 38 | 046 | Range limit 1, high | double | 7899 / 370 / 2000 |
| 46 | 056 | Range limit 2, low (same values so far) | double | 0 / 125 / 450 |
| 54 | 066 | Range limit 2, high | double | 7899 / 370 / 2000 |
| 62 | 076 | Plot min amplitude (MNY) | float | 0.0144 / 0 / 0 |
| 66 | 102 | Point number of the min (counting from 1) | int | 628 / 236 / 718 |
| 70 | 106 | Plot max amplitude (MXY) | float | 459.97 / 2.0 / 2.0 |
| 74 | 112 | Point number of the max (counting from 1) | int | 229 / 207 / 15 |
| 78 | 116 | Resolution? (or plot step) | float | 15.44 / 0.5 / 0.2 |
| 92 | 134 | Number of scans? (matches block 2, byte 138) | int | 10 / 200 / 2000 |
| 120 | 170 | Apodization | 8 chars | " Bartlet" / " HAMMING" |
| 128 | 200 | Date and time | 17 chars | "04-OCT-9612:16:37" |
| 364 | 554 | Source file name | text | "SY0:GL13M2C .SPC" |

- Wavenumber of point i = first + i x step (i starts at 0).
- On LOG files the min/max point numbers are flipped, because they come from the ratio before the log.
- On SPC files the min/max amplitude is just the data's min and max. On LOG files it's 0 to 2 for plotting.
- The min/max amplitude are 2-word floats, each followed by its point number. Reading them as 4-word doubles gives
  the same value but hides the point number.
- Resolution vs plot step: on GL1KB1 it's exactly 2 x step (normal for resolution), and the instrument's file
  line shows it next to the date and range. On the LOG files 0.5 and 0.2 would also be sensible tick spacings for
  a 0 to 2 plot, so not settled.

## Block 2 (starts at byte 512)

| Offset | Octal | What | Type | Example |
|---|---|---|---|---|
| 16 | 020 | Optics: detector, beamsplitter, source, filter? | 4 x 4 chars | " MCT KBR GL1 FI1", " BO1 3M1 GL1 FI1" |
| 76 | 114 | Unknown | float | 200 / 100 / 100 |
| 88 | 130 | 4 x HeNe laser wavenumber | float | 63192 |
| 138 | 212 | Number of scans? (matches block 1, byte 92) | int | 10 / 199 / 2000 |
| 92 | 134 | Range setting? | float | 450 / 125 / 450 |
| 96 | 140 | Range setting? | float | 5000 / 850 / 5000 |
| 100 | 144 | Range setting? | float | 500 / 10 / 330 |
| 104 | 150 | Range setting? | float | 5000 / 370 / 2000 |
| 452 | 704 | Related files: data, .SET, .REF | text | "GL13M2C  GL13M2C .SET  GL13M2C09.REF" |

## Data (starts at byte 1536)

- One float per point, as many as the point count. The rest of the last block is zeros.
- SPC = single beam, raw intensity.
- LOG = -log(sample / reference), so absorbance. Probably log10.
- Exactly 8.0 in a LOG file means "too dark to measure", a clip value.

## Odd one out

ATM077011's header doesn't quite add up: last - first is one step longer than the point count allows. So its last
point could be off by one step (0.096 cm-1). The other two files add up exactly.
