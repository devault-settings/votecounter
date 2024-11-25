# votecounter

A ranked pairs vote counting code compatible with google forms.

This code is adapted from a votecounting code developed for use in elections at Ricketts Hovse, one of the 8 undergraduate houses at Caltech.
It exists thanks to the contributions of Coby Abrahams, Alejandro López, Alison Dugas, Umesh Padia, Jack Briones, Audrey DeVault, and other Skurves.

To make sure the code is working correctly, try running `RunTest()` . This will create and count ballots matching the Tennessee example in the [Ranked Pairs Wikipedia Page](https://en.wikipedia.org/wiki/Ranked_pairs). If errors arise, ensure all required packages are properly installed and up to date, specifically glob, copy, os, networkx, matplotlib, collections and warnings. The winner should be Nashville and the graphical representation should look like this, with arrows pointing to the losers: 

<img src="https://github.com/user-attachments/assets/9176093c-790f-47ad-8acf-34d61e32ef54" width="300">


To utilize the code with a google form, first make a google form using the "Multiple Choice Grid" format, as seen in the [example balllot](https://docs.google.com/forms/d/e/1FAIpQLSfqx1SwrUv0cPKBTYrf01hfVWlrvuUeNCWjlGBjRGQR9zXr_Q/viewform?usp=pp_url&entry.1241028677=7&entry.607893604=3&entry.1623747979=2&entry.622707023=8&entry.936978487=6&entry.2121588736=5&entry.1845818087=1&entry.1079487710=4). Ensure no candidate titles contain commas, and that there are as many number columns as there are candidates.

When voting is complete, navigate to the 'Responses' tab of your form, click the three dots, then click 'Download responses (.csv)'. Place the resulting your_filename_here.csv in the same directory as this votecounter code, then run `Run('your_filename_here.csv')`.

As an additional worked example, I have included a sample output .csv from a google form vote to determine the best acronym for the High Energy Density Physics group at MIT's Plasma Science and Fusion Center. Running `Run('acronym_votes.csv')` should yield the winning acronym, CANDELA (Center for Advanced Nuclear Diagnostics and Experimental Laboratory Astrophysics), and the following graphical representation:

<img src="https://github.com/user-attachments/assets/f559cb32-cea2-43e4-af66-ca6f0ef7c579" width="300">


RIP JEDI (Junction for High Energy Density Investigations)

