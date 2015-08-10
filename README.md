Web Sockets
============================
A simple nodeJS project that uses the socket.io NodeJS module to enable real time communication between clients and the development board via a web browser to toggle the state of the onboard LED.

This NodeJS application consists of a client side application accessible via a web browser in which users can view the number of connected users, send text messages between users as well as toggle the onboard LED (ON/OFF) of your development board.

You can view this application by inputting your development board's IP address as well as the port number (3000) specified in the main.js within the browser's address bar. For example, http://192.168.1.0:3000.

**Note:** You can find your board's IP address by typing the ```ifconfig``` command in the Linux terminal via a Serial or SSH connection.

###Intel(R) Edison
In order to leverage this project successfully, you will need to connect to a wireless network. For more information, visit https://software.intel.com/en-us/connecting-to-a-network-intel-edison-board.

###Intel(R) Galileo
In order to leverage this project successfully, you will need to connect to a wireless network. For more information, visit https://software.intel.com/en-us/articles/intel-galileo-getting-started-ethernet.


###Intel(R) Edison & Intel(R) Galileo
####(Intel XDK IoT Edition) Install node modules
Within the "manage your xdk daemon and IoT device" menu, check the following boxes:

* Clean '/node_modules' before building Run npm install directly on IoT
* Device (requires internet connection on device)

You can installed the required node modules for this project which are found in the package.json file by pressing the Build/Install button.

####(Intel XDK IoT Edition) Upload & Run project
After installing the neccessary node modules, press the:
    1. Upload button
    2. Run button to execute your project on your board.

####Getting Started with socket.io NodeJS Plug-in
#####Design Considerations
The necessary information needed for creating a web socket (socket.io) application:
```javascript
var mraa = require('mraa'); //require mraa
console.log('MRAA Version: ' + mraa.getVersion()); //write the mraa version to the Intel XDK console
//var myOnboardLed = new mraa.Gpio(3, false, true); //LED hooked up to digital pin (or built in pin on Galileo Gen1)
var myOnboardLed = new mraa.Gpio(13); //LED hooked up to digital pin 13 (or built in pin on Intel Galileo Gen2 as well as Intel Edison)
myOnboardLed.dir(mraa.DIR_OUT); //set the gpio direction to output
var ledState = true; //Boolean to hold the state of Led

var express = require('express');
var app = express();
var path = require('path');
var http = require('http').Server(app);
var io = require('socket.io')(http);
```
In order to setup the client side web page, you will need to identify the path to the index.html:
```javascript
app.get('/', function(req, res) {
    //Join all arguments together and normalize the resulting path.
    res.sendFile(path.join(__dirname + '/client', 'index.html'));
});
```
Files required for your client side web page(s) need to be explicitly identified.
```javascript
//Allow use of files in client folder
app.use(express.static(__dirname + '/client'));
app.use('/client', express.static(__dirname + '/client'));
```

#####Socket.io EventListeners
**Handling connection, and custom events**
```javascript
//Socket.io Event handlers
io.on('connection', function(socket) {
    console.log("\n Add new User: u"+connectedUsersArray.length);
    if(connectedUsersArray.length > 0) {
        var element = connectedUsersArray[connectedUsersArray.length-1];
        userId = 'u' + (parseInt(element.replace("u", ""))+1);
    }
    else {
        userId = "u0";
    }
    console.log('a user connected: '+userId);
    io.emit('user connect', userId);
    connectedUsersArray.push(userId);
    console.log('Number of Users Connected ' + connectedUsersArray.length);
    console.log('User(s) Connected: ' + connectedUsersArray);
    io.emit('connected users', connectedUsersArray);
    
    socket.on('user disconnect', function(msg) {
        console.log('remove: ' + msg);
        connectedUsersArray.splice(connectedUsersArray.lastIndexOf(msg), 1);
        io.emit('user disconnect', msg);
    });
    
    socket.on('chat message', function(msg) {
        io.emit('chat message', msg);
        console.log('message: ' + msg.value);
    });
    
    socket.on('toogle led', function(msg) {
        myOnboardLed.write(ledState?1:0); //if ledState is true then write a '1' (high) otherwise write a '0' (low)
        msg.value = ledState;
        io.emit('toogle led', msg);
        ledState = !ledState; //invert the ledState
    });
    
});
```

Starting the http server listening on port 3000
```javascript
http.listen(3000, function(){
    console.log('Web server Active listening on *:3000');
});
```

####(Web Browser) View Client side application
Input the IP address of your board plus the port number (3000) 
For example, http://192.168.1.0:3000

Intel(R) XDK IoT Edition
-------------------------------------------
This template is part of the Intel(R) XDK IoT Edition. 
Download the Intel(R) XDK IoT Edition at https://software.intel.com/en-us/html5/xdk-iot. To see the technical details of the sample, 
please visit the sample article page at https://software.intel.com/en-us/xdk/docs/intel-xdk-iot-edition-nodejs-templates.

Important App Files
---------------------------
* main.js
* package.json
* icon.png
* README.md
