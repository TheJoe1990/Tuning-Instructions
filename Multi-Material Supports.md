Please note that these instructions were writen mostly for my mom, for the current version of SnapMaker Orca fork in Augest of 2026 and Im sure things will change.  These steps are not exactly the same in Bambu or Mainline Orca, and are only recomended for nozzle changers at this time.  I recomend using dramatically different colors for your pla vs your petg.  Thank you.  

![Selecting filaments](photos/Supports%20step%201.png)

Start by selecting your filaments on the left side, in this screenshot we have PLA for 1,3, and 4 with PETG in 2.  Either PLA or PETG can be used to support the other, so if you had a PETG model that needed support you could use PLA.  

![Editing the support filament](photos/Supports%20step%202.png)

Then we edit the filament we are using as a support material, in this case the PETG.  

Check the box for it being a support material (Not soluble material, that's different) and then save it as something like, "Petg Support".

![Modified support profile](photos/Supports%20step%203.png)

Make sure that it now shows your modified "Support" profile.  In our example that is filament 2, now our PETG Support, where I put the top red arrow.  Then select that same filament for the support raft interface, the lower red arrow.   If everything has worked correctly you will get the popup and be able to select yes.  If you do not get the popup, something didnt work as expected.  

tower material.png

The next step is making sure the prime tower doesnt come apart while you are printing.  Youll need to set the prime tower to do the walls in your primary material so it still has structure.  In the photo above you can see my red arrow.  

https://github.com/TheJoe1990/Tuning-Instructions/blob/main/photos/Support%20Speeds.png

We found the supports were being thrown off of the build sheet so we halved the speed the supports were printed at.  You may need to tune these further, and if your supports are nice and short you may not need to touch this setting at all, but this worked for us.  

And here is our final result!  Im sure that this can be tuned to be even better, but Im pretty jazzed at where we arrived for now. Mom did have to scrape some of the PETG off the model with her fingernail, but it all came away cleanly leaving no sags or rough areas.  

Happy printing!

https://github.com/TheJoe1990/Tuning-Instructions/blob/main/photos/frontside.jpg
https://github.com/TheJoe1990/Tuning-Instructions/blob/main/photos/backside.jpg
https://github.com/TheJoe1990/Tuning-Instructions/blob/main/photos/final%20result.jpg
