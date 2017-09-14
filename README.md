# WiMNET WebRTC-analyzer
### A simple video server app to measure and record performance metrics of webRTC calls

Want to see it in action? Check out the demo: https://webrtc-wim.net

## Getting started
1. Install dependencies via `npm install`

2. Start video server: `node server.js`

## Setting up an experiment
#### Local
To initiate a webRTC call, you need two clients. A client is simply a webRTC-compatible browser pointing to the URL where the server is running.
Each client must be running on a machine with a webcam, or on a browser instance with a video file injected in place of a webcam.
In local testing (one machine), this will be `localhost:8000` by default. If the browser is on a computer with a webcam, the app will query to access the webcam. Custom video files can also be injected, detailed in a later section.

In either client window, enter a room name, and click "create". This creates a 'room' which other clients can join by entering the same room name or entering the room URL into the browser.
Room URLs follow this format: `localhost:8000/?<room_name>`

As soon as more than one client is in the room, a webRTC call begins, and the app begins recording call statistics. To save a call's trace, use the "log stats" button. This feature requires configuring a database, which is outlined in the database section.

#### Over a network
To run experiments over a local network, the app must be running on a network-addressable location. This can be done within a LAN by using a web server such as Apache or NGINX to map incoming traffic on port 80 to the server machine's port 8000.

To run experiments over the internet, the app must be running on a publically reachable address, and a webserver is required for port-forwarding as it is in the LAN use case. A very simple way to achieve these steps at once is to use a service such as AWS Elastic Beanstalk, Google's Firebase, or any hosting service that allows you to upload and host a Node.js project.
Our experiments utilized an ElasticBeanstalk Node.js deployment. See [their docs](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_nodejs.html) for setup instructions.

NOTE: Most browsers require a HTTPS connection for webRTC calling by default.

#### Setting up a database
Out of the box, this app requires a database connection to log call stats. This app was built using a [Firebase](https://firebase.google.com/products/database/) Realtime Database instance.
After creating a Firebase project (free tier worked just fine for our experimental purposes), you will be given a configuration script to paste into your webapp.
In `charts.js`, replace the config object with the one Firebase gives you. It should look something like this:

```js
  // Initialize Firebase
  var config = {
    apiKey: "<YOUR_API_KEY>",
    authDomain: "<your-project>.firebaseapp.com",
    databaseURL: "https://<your-project>.firebaseio.com",
    projectId: "<your-project>",
    storageBucket: "phidio-c8b8a.appspot.com",
    messagingSenderId: "123456789"
  };
```

## Running experiments
#### Basic usage
To run an experiment, simply add two clients to a room. The moment two clients join the same room, the call will begin and the app will begin recording stats. To record call stats, clicking 'log stats' will upload the call metrics to the database. The call will continue after clicking the 'log stats' button. A call ends when one of the clients leaves the room.

#### Remote testing
Clients on remote servers can be initiated using the `start-phidio.py` script provided in the `scripts` directory. The script can be run on any server that has both Google Chrome and Google's ChromeDriver installed. See https://sites.google.com/a/chromium.org/chromedriver/ for instructions.
Assuming this remote server does not have a webcam, you will need to supply a local video file to inject into the chrome instance. In `start-phidio.py`, modify the `LOCAL_PATH_TO_VIDEO` variable to point to a local video file on the remote server. You will also need to modify the `APP_URL` variable to point to where your instance of this app is hosted.
The script takes one command line argument, the room name. Run the script on the remote server as follows: `python start-phidio.py <room_name>`.
This will boot an instance of chrome, inject the local video file into it for fake video capture, and point it to your instance of the app at the specified room name. If no room is specified, it will default to a room 'testroom'.

To log stats remotely, add the `-log` flag on the command line. This will automatically 'click' the Log Stats button after 3 minutes, thus uploading call data from your remote server to the database.

Executing this process on two remote machines will initiate a webRTC call between them. It can be fully monitored from another machine by simply navigating to that room URL from any other computer.

## Viewing and collecting experiment data
All experimental results can be viewed in-browser via the `/charts` resource. i.e. in our demo app, this is at `https://webrtc-wim.net/charts`. All calls can be viewed using the dropdown selector. It may take a moment for the call data to load from the database. Below the dropdown selector is a button to download the selected call's data in JSON format. A script to compute statistics on a given call's data is provided in `scripts/get-call-stats.py`.


This experimental tool was built using the [SimpleWebRTC](https://github.com/andyet/SimpleWebRTC) library.

