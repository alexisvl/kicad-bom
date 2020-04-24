# KiCad BOM script

This is a really rough simple script to extract BOMs KiCad files. It's mostly
just meant to dump out order lists (in particular, it can generate Digi-Key
bulk-add lists), and requires BOM lines to be specified in certain ways:

- Each line item starts with a _source_ (aka vendor, distributor, supplier),
  then a colon delimiter, then a part number. I use "DK" for Digi-Key, for
  example, so CP-063AH-ND is `DK:CP-063AH-ND`.

- To attach a line item to a symbol, put it in a field called `BOM`. Multiple
  can be attached to one symbol, delimited by a comma (and optionally a space).

- To list line items loose in the schematic, put them on a line by themselves
  in a text object, starting with `BOM ` and then formatted the same as a field
  (e.g. `BOM DK:CP-063AH-ND`). As before, commas are supported.

- To list loose line items with a quantity, put them on a line by themselves
  in a text object, formatted like `BOM <qty> <item>`. In this case, only one
  item specification per line is supported. For example, `BOM 4 DK:36-24438-ND`
  orders four of `36-24438-ND` from Digi-Key.

- In both fields and texts, `#` comments are supported.

KiCad has several built-in tools for BOM management, so you probably shouldn't
use this. Also it's a bit hacky and bad so you probably shouldn't use this. But
it fits my workflow well.

## Usage

```
usage: bom [-h] [--sort] [--collate] [--no-collate-refs]
           [--exclude-source SRC] [--only-source SRC] [--dk]
           SCH [SCH ...]

Extract BOM from KiCad schematics

positional arguments:
  SCH                   Files to extract, with optional colon-
                        delimited ref prefix

optional arguments:
  -h, --help            show this help message and exit
  --sort, -s            Sort items, first by BOM line, then by
                        reference
  --collate, -c         Combine same items onto lines. Implies --sort
  --no-collate-refs     List all references individually, even when
                        lines are collated
  --exclude-source SRC  Do not list SRC
  --only-source SRC     Only list SRC (can be given multiple times
  --dk                  Output in Digi-Key bulk add format. --only-
                        source=DK recommended
```

Output is tab-delimited raw text, with columns: quantity, source, item,
references.

I typically use it like this:

- `bom -sc --exclude-source=DK` — human-readable list of parts _not_ from
Digi-Key that I may have to manually source (consider piping into
`column -t -s $'\t'`)

- `bom -sc --dk --only-source=DK` — dump a bulk-add list to order from
Digi-Key

## Example output

```
bom -sc --only-source=DK *.sch
```

```
4   DK  1655-1504-1-ND               D2-3,D6-7
3   DK  1655-1817-1-ND               D1,D4-5
2   DK  2019-RN73R2BTTD4702B25CT-ND  Ra1,Rb1
1   DK  237-1887-ND                  ?
2   DK  296-35279-1-ND               U3-4
4   DK  311-2953-1-ND                Ra8,Ra12,Rb8,Rb12
2   DK  311-2967-1-ND                Ra4,Rb4
2   DK  311-4433-1-ND                Ca1,Cb1
4   DK  36-24438-ND                  ?,?,?,?
4   DK  399-15740-ND                 C5,C10-12
8   DK  399-C1206C104K5RAC7800CT-ND  C3,C8,C13-18
1   DK  563-1558-ND                  SW1
2   DK  565-3924-ND                  C1,C6
4   DK  732-12332-ND                 ?,?,?,?
12  DK  732-9268-1-ND                Ca3-4,Ca6-9,Cb3-4,Cb6-9
2   DK  732-9587-1-ND                C4,C9
1   DK  CP-063AH-ND                  J1
2   DK  CP-1401-ND                   Jb1-2
2   DK  CP-1402-ND                   Ja1-2
1   DK  LM317LDR2GOSCT-ND            U1
1   DK  LM337LMX/NOPBCT-ND           U2
5   DK  MH3261-601YCT-ND             E1-3,Ea1,Eb1
2   DK  P14435-ND                    C2,C7
6   DK  PCF1181CT-ND                 Ca2,Ca5,Ca10,Cb2,Cb5,Cb10
3   DK  RMCF1206FT10K0CT-ND          R5,Ra3,Rb3
4   DK  RMCF1206FT240RCT-ND          R1,R3,R6-7
2   DK  RMCF1206FT2K43CT-ND          R2,R4
2   DK  RMCF1206FT2K70CT-ND          Ra9,Rb9
2   DK  RMCF1206FT3K40CT-ND          Ra7,Rb7
4   DK  RMCF1206FT4K70CT-ND          Ra2,Ra5,Rb2,Rb5
2   DK  RMCF1206FT620RCT-ND          Ra13,Rb13
6   DK  RMCF1206JT470KCT-ND          Ra6,Ra10-11,Rb6,Rb10-11
```

## Missing BOM warnings

All non-virtual symbols (references do not start with `#`) are expected to
carry BOM lines; a warning is emitted for all that don't. To silence this
for a symbol, set its BOM line to `NOBOM`.

## Per-file prefixes

If two files in a set could have colliding references, you can add a prefix to
all references in a file when specifying that file on the command line. The
prefix is added to the filename, with a colon delimiter; this prefix will then
be prepended to all of that file's references.
