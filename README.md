# Performance Testing Using JMeter

For the performance testing of CUBA-based web application we use the following tools:

[Apache JMeter](https://jmeter.apache.org/)

[Mozilla Firefox](https://www.mozilla.org/en-US/firefox/) 

#### Application configuration 
Add `cuba.performanceTestMode = true` to the web-app.properties file to disable “CSRF” security.

#### Firefox configuration
Need to set `network.proxy.allow_hijacking_localhost` to `true` in Firefox `about:config` in order to proxy localhost (it’s false by default)

#### JMeter scenario
##### Adding Elements to Test Plan
Add “Thread Group” to “Test Plan”:
```
Test Plan -> Add -> Threads (Users) -> Thread Group
```
Add “HTTP(S) Test Script Recorder” to “Test Plan”:
```
Test Plan -> Add -> Non-Test Elements -> HTTP(S) Test Script Recorder
```
Change the default port of the “HTTP(S) Test Script Recorder” proxy to 9090 
or another required one. 8080 is used for our application.

![](/img/70f94592fc1ee6634f3f9da224f083b66103eba4.png)

On the ‘Requests Filtering’ tab, specify the following code to exclude unnecessary requests (JS, CSS, etc.):
```
(?i).*\.(bmp|css|js|gif|ico|jpe?g|png|swf|woff|txt)
(?i).*\.(bmp|css|js|gif|ico|jpe?g|png|swf|woff|txt)[\?;].*
```
Or you can add it separately:
```
.*\.png
.*\.jpg
.*\.gif
```

![](/img/a4b620d81b8f0269ce17a54e009553c663eb0e3e.png)

Also, add “View Results Tree” to “HTTP(S) Test Script Recorder”
```
HTTP(S) Test Script Recorder -> Add -> Listener -> View Results Tree
```
It shows data for all recorded requests.

##### Adding Elements to Thread Group
Add “HTTP Cookie Manager” to “Thread Group” :
```
Thread Group -> Add -> Config Element -> HTTP Cookie Manager
```
Check “Clear cookies each iteration” in this manager.

![](/img/bab80e708b14e744e629be05b7cc22f85857f222.png)

Add “User Defined Variables” to “Thread Group”:
```
Thread Group -> Add -> Config Element -> User Defined Variables
```

We will use two variables:

* host address: `${__P(host, localhost)}`
* port: `${__P(port, 8080)}`

![](/img/f579e01a75f53eb693807068e0e568181dac92d0.png)

Then add “Recording Controller” to “Thread Group”:
```
Thread Group -> Add -> Logic Controller -> Recording Controller
```
Requests from the application will be saved in this controller. Also, add a tree which 
shows info about all recorded requests when they are played.
```
Recording Controller -> Add -> Listener -> View Results Tree
```
So, the final scenario tree is the following:

![](/img/cbb6b094730b0e36feb3f3c576d737eae49df7c8.png)

##### Configure Firefox proxy
Configure Firefox to use localhost on port 9090 (like in HTTP(S) Test Script Recorder proxy):

* Open Firefox
* Open browser options
* Go to proxy settings
* Select the “Manual proxy configuration”
* Set HTTP Proxy to “localhost” and Port to “9090”
* Check “Use this proxy server for all protocols” option
* Check that localhost and 127.0.0.1 NOT in “No Proxy for” list, otherwise remove them

![](/img/bf4dc75af9ece05c4cebfcc12357642690de163a.png)

* Click OK and exit from the menu
* Restart Firefox

##### Recording Requests
Our test will do 3 simple steps:

1. Login to the application.
2. Open “Users” screen.
3. Logout.
To record request do the following steps:
* In the “HTTP(S) Test Script Recorder” click “Start” button. We will see the information dialog: “Root CA certificate ApacheJMeterTemporaryRootCA created in bin directory”. Click the OK button.
* In Firefox open [http://localhost:8080/app](). We can see that some requests have already saved in “Recording Controller”.

![](/img/6befc1e6a1e27ec743263b8133e13b2421e68d28.png)

* Login to the application. For the sake of convenience, rename the resulting request.

![](/img/53ea269aa842cd89b557ee011ae8f41af0b81da6.png)

* Open “Users” screen.
* Logout.
* Stop recording.

You can find the resulting scenario [here](/scenario/cuba-app-test-scenario.jmx).

Now we can delete unnecessary requests, for instance, `app/Push`. After that, the scenario looks as follows:

![](/img/ed7b17af7b574336d6f1472636699229ef5e8117.png)

Play our recorded requests to test it. Click “Start” button on the top buttons panel.

The dialog will appear. It will ask you to save the test plan. Save it with the name “cuba-app-test-scenario”.

In the tomcat’s log, we can see that the admin user ‘logged in’ and ‘logged out’. “View result tree” shows info about request and response.

![](/img/2ff5aa5059349733794034260eafb73e05215d2f.png)