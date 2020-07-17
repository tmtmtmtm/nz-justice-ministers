Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item for the [Minister of Justice](https://www.wikidata.org/wiki/Q6866227)
looks mostly good, except it wasn't connected (in either direction) to
the [Ministry of Justice](https://www.wikidata.org/wiki/Q7015504), so I
fixed that.

Step 2: Tracking page
=====================

Initial PositionHolderHistory list set up at https://www.wikidata.org/w/index.php?title=Talk:Q6866227&oldid=1232880210

Current status: knows of 44 historic officeholders, but with 12
warnings.

Step 3: Set up the metadata
===========================

The first step now in a new repo is always to edit [add_P39.js script](add_P39.js)
to set up the Item ID and source URL.

Step 4: Scrape
==============

Comparison/source = [Minister of Justice (New Zealand)](https://en.wikipedia.org/wiki/Minister_of_Justice_(New_Zealand))

    wd ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

Scraped cleanly on first pass.

Step 5: Get local copy of Wikidata information
==============================================

We now get the argument to this from the JSON, so call it as:

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json


Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

12 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/6a436ebed5945

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

43 additions (entirely of 'series ordinal') -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/84e7875362ab8

Step 8: Refresh the Tracking Page
=================================

Those updates give us
https://www.wikidata.org/w/index.php?title=Talk:Q6866227&oldid=1232888734,
which is much more complete, but a bit muddled recently. It seems that
this is because Chris Finlayson was an acting minister, so I've added
'nature of statement: acting' to his P39, and will accept the
modifications new-qualifiers.rb suggests (including a couple of tweaks
to some earlier dates too):

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json 2>&1 >/dev/null |
      fgrep -v \* | wd uq --batch --summary \
        "Update qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

-> https://tools.wmflabs.org/editgroups/b/wikibase-cli/72fb8c82c6d88/

That gives us a final version as https://www.wikidata.org/w/index.php?title=Talk:Q6866227&oldid=1232905308

