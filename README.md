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
  in a text object, formatted like `BOM qty item`, for example
  `BOM 1 DK:CP-063AH-ND`. Currently, the quantity is mandatory, and line items
  cannot be combined with commas (use separate lines).

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

## Missing BOM warnings

All non-virtual symbols (references do not start with `#`) are expected to
carry BOM lines; a warning is emitted for all that don't. To silence this
for a symbol, set its BOM line to `NOBOM`.
