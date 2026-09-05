Concept:
We save a lot of images as resources and screenshots from various places like instagram, youtube, reddit etc, but when it comes to referring them in future, we cant search for the screenshot, it is basically lost in the large sea of screenshots, and it would lie there indefinitely until removed permanently. 
To solve this i have made a image analysis and clustering system that clusters similar screenshots together and shows them in a organized UI, where you can search by semantic meaning, exact word match, and visuals. 

Here you can see all images in a unified interface and be able to easily find out the screenshot that was lost, just by knowing what that screenshot could have contained, the exact or surrounding text, how it looked etc which is much easier to figure out.

The system:
I have devised a client-server architecture where a central server takes in all the screenshots from all the connected clients, accumulates in an organized manner, runs a complete python pipeline on it to get required info from the images and retains them in a SQLite DB.

Now this retained data is fetched from server, and shown on a UI, this data shown is clustered using semantic meaning matching and visual matching.




![](Assets/Screenshot%20analyser.pdf)