# TAMU Professor Info UI Overhaul
PR Author: Tiernan Lindauer

## Change Summary
This PR contributes the following features to the TAMU Professor Info UI:
- App elements updated using `container mt-4 theme-light`
- Added page header
- Changed icon to black TAMU logo
- Condensed the Subject and Course input boxes into one input box
- Animations for table loading/unloading
- Red text below the text input box when the user enters an invalid input (doesn't include when the input is blank)
- Added disclaimer to the bottom of the page to inform people that this isn't a TAMU endorsed project
- Added a link to the issues page to reccomend people to submit issues if they find any or request features

## Future Thoughts
- Could add line graphs to chart the average GPA over time for a given professor (potentially other stats too)
- See about adding older data to a database ([Astra DB](https://astra.datastax.com/) is a good choice and shoud have more than enough free credits to use for a long time)
