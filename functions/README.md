![](images/fn.png)

## Introduction

In this guide, we'll show you a few simple steps to launch an Oracle Linux instance on Oracle Cloud Infrastructure, and then proceed to launch your [Fn Project](https://fnproject.io/) Functions server & run your cloud functions. The path that we will take is as follows:

 - Launch your Oracle Linux instance in the Oracle cloud
 - Install Oracle Container Runtime for Docker
 - Install the Fn Functions Server
 - Configure and run your Oracle Code Card Fn functions
 - Create a Fn function for your Code Card

## Create an Oracle Linux instance on the Oracle Cloud Infrastructure
Follow [these instructions](oci.md) to create an Oracle Linux instance and the come back after you are done.

## Configuring your Fn Server

Your Oracle Linux instance is now running, and ready to be configured to host your cloud functions.

### Install and configure Docker

While connected to your Oracle Linux instance, run the following commands to install and configure the Docker container runtime.

```bash
sudo yum -y install docker-engine-18.03.1.ol-0.0.9.el7.x86_64 -y
sudo usermod -aG docker opc
sudo systemctl enable docker
sudo systemctl start docker
```

Before proceeding further, logout of the current SSH session, & then reconnect to your Oracle Linux instance and log back in vis SSH. _This is to ensure group membership configured in the previous step is correctly applied and in effect._
Once you have reconnected to the instance, run the following command to install the Fn CLI tool (this will download a shell script and execute it).

### Configure Oracle Linux for Fn

```bash
curl -LSs https://raw.githubusercontent.com/fnproject/cli/master/install | sh
```

At completion, the installation will output the Fn CLI version - per the below example output.

```bash
fn version 0.5.16

        ______
       / ____/___
      / /_  / __ \
     / __/ / / / /
    /_/   /_/ /_/`

```

#### SELinux constraints
Before you can start Fn you must relax SELinux constraints by running this command:

```bash
sudo setenforce permissive
```

### Start your Fn Server

Run the following command which will start Fn in the background as a single server mode, using an embedded database and message queue.

```bash
fn start -d
```

Your Fn server is now instantiated and running in the background.

## Configure and run your Fn functions

Functions are small but powerful blocks of code that generally do one simple thing. Forget about monoliths when using functions, just focus on the task that you want the function to perform. Our CLI tool will help you get started super quickly.

To create a hello world function, run the following command.

```bash
fn init --runtime go --trigger http hello
```

This will create a simple function in the directory hello, so let's cd into it:

```bash
cd hello
```

### Deploy your functions to your local Fn server

```bash
fn deploy --app codecard --create-app --local
```

Now you can call your function locally using curl:

```bash
curl http://localhost:8080/t/codecard/hello
```

or, using the Fn client:

```bash
fn invoke codecard hello
```

or in a browser: http://<linux-instance-public-ip>:8080/t/codecard/hello-trigger

That's it! You just deployed your first function and called it. You are now ready to configure your Code Card to access your cloud function!

## Create a Fn function for your Code Card
The Code Card needs to receive the following JSON format:

Required fields:

```bash
{
	"template": "template[1-11]",
	"title": "Hello World",
	"subtitle": "This is a subtitle",
	"bodtext": "This is the body",
	"icon": "[see list of named icons| BMP url]",
	"backgroundColor": "[white|black]"
}
```

**Check out the list of available named icons [here](icons.md)*.

Optional fields:

```bash
{	...
	"badge": [0-100] It will override the icon
	"backgroundImage": "[oracle|codeone | BMP url]" Only for templates that have backgrounds
	"fingerprint": "" The SHA-1 signature of the server containing the custom icon or backgroundImage URL.
	...
}
```

To checkout all available templates go to Oracle Events App -> Code One --> Code Card Designer.

Let's create our first Code Card function!

```bash
fn init --runtime node --trigger http button1
cd button1
```bash

Now lets edit the func.js file using `nano` or `vi`.

```bash
nano func.js
```bash

Modify the handle function to look like this:

```bash 
fdk.handle(function(input){
    let codeCardJson = {
      template: 'template1',
      title: 'Hello there!',
      subtitle: 'How are you?',
      bodytext: 'This is my first Fn function from the Oracle Cloud.',
      icon: 'opensource',
      backgroundColor: 'white'
    }
    return codeCardJson
})
```

In nano `Ctrl` + O and `Ctrl` + X (WriteOut and Exit.)

In vi `ESC`  `:wq` (write and quit.)

Now deploy your new function

```bash
fn deploy --app codecard --local
```

And test on your browser

```bash
http://<linux-instance-public-ip>:8080/t/codecard/button1
```

Now you are ready to configure your Code Card to point to your new function!

### Configure Code Card
In this example, we will program the `shortpress` action for button `B` on the card.

#### Establish serial connection with Code Card
In order to configure our Code Card, we need to establish a serial connection over USB to the CodeCard CLI. Follow [this guide](https://github.com/cameronsenese/codecard/blob/master/terminal/README.md#connect-via-terminal-emulator) to establish the serial over USB connection. Remember to ensure that the Code Card WiFi settings are configured correctly also! (Direction available from the referenced guide).

#### Configure `buttonb1` button action
*In the Code Card CLI, `buttonb1` correlates to button B shortpress action.*

In your terminal session you should now see the Code Card CLI Menu, as follows.

```bash
***************************************************************************************
  Code Card v1.0
  Oracle Groundbreakers
  developer.oracle.com/codecard
***************************************************************************************
Commands:
  ls                Show all stored key/values
  help              Show this help
  shortpress[a|b]   Simulate the press of a button
  longpress[a|b]    Simulate the long press of a button
  connect           Connect to wifi
  disconnect        Disconnect wifi
  restart           Restart wifi
  status            Show wifi status
  home              Show home screen
  reset             Reset to factory settings

Usage:
  Read saved key value:
    key
  Save new key value:
    key=[value]

Available keys:
  ssid, password, buttona1, buttona2, buttonb1, buttonb2, fingerprinta1, fingerprinta2,
  fingerprintb1, fingerprintb2, methoda1, methoda2, methodb1, methodb2,
>>>
```

First, we will set the HTTP method for the B shortpress by entering the following command.
*Keep in mind that pausing for 2 seconds while typing will automatically enter the command. It may be easier to pre-type the commands elsewhere and copy-paste them into the window.*

```bash
methodb1=GET
```

Code Card will confirm setting update as follows.
```bash
>>>
Value saved for methodb1: GET
>>>
```

Next configure the HTTP endpoint for the B shortpress by entering the following command. Be sure to substitute values in `<brackets>` as appropriate.

```bash
buttonb1=http://<linux-instance-public-ip>:8080/t/codecard/button1
```

Code Card will confirm setting update as follows.

```bash
>>>
Value saved for buttonb1: http://<linux-instance-public-ip>:8080/t/codecard/button1
>>>
```

### Invoke the function from the Code Card
Ok, so now our cloud function and Code Card are ready to Go! Powercycle your Code Card and perform a button B shortpress. If your card is still connected via the serial connection, you will see output similar to the following.

```bash
Button b - short pressed
>>>
Connecting to 'pmac851' ...................connected!
IP address: 192.168.43.13
MAC address: 84:0D:8E:A7:89:5B
>>>
Request:
  host: 129.213.19.161
  port: 32690
  url: http://`linux-instance-public-ip`:8080/t/codecard/button1
  method: GET
text/plain;charset=UTF-8
Response:
  {"template":"template1","title":"Hello there!","subtitle":"How are you?","bodytext":"This is my first Fn function from the Oracle Cloud.","icon":"opensource","backgroundColor":"white"}
>>>
```
