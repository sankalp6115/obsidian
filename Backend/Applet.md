1. Applet is a Java program that can be embedded into a web page. 
2. It runs inside the web browser and works at client side. 
3. Applets are used to make the website more dynamic and entertaining. 
4. All applets are sub-classes (either directly or indirectly) of java.applet.Applet class. 
5. Applets can run within a web browser or an applet viewer (standard applet viewer tool). 
 
Life cycle of an Applet : 
Life cycle of an applet use five methods which are as follows : 
a. init( ) : This method is intended for whatever initialization is needed for our applet. 
b. start( ) : This method is automatically called after the browser calls the init method. 
c. stop( ) : This method is automatically called when the user moves off the page on which the applet sits. 
d. destroy( ) : This method is only called when the browser shuts down normally. 
e. paint( ) : Invoked immediately after the start( ) method, and also any time the applet needs to repaint itself in the browser.


Applet program for displaying image
![[Pasted image 20260512193149.png]]
![[Pasted image 20260512193200.png]]
![[Pasted image 20260512193250.png]]


Advantages of applets : 
1. Applets are platform independent. 
2. Applets are quite secure and safe to use. 
3. Applets cache quickly. 
4. Applet increase interactivity for users. 
5. Database integration is another important advantage of applets. 

Drawbacks of applets : 
1. Applets do not access client-side resources, like such as file, operating system. 
2. Applet cannot work with native methods. 
3. Applet can only extract information about client-machine i.e., its name, Java version, OS, version etc. 
4. Mobile browsers which are running on IOS or Android do not support applets.

IF TIME REMAINS, LEARN TO WRITE APPLETS