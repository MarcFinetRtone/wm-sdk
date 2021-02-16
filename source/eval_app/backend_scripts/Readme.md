# Evaluation application python script

## Script scope

This script allows to send and received specific Evaluation app's messages to/from nodes, part of a Wirepas Mesh network
via a Wirepas gateway connected to the mesh using the **Wirepas-gateway-API**.

## Script building blocks
The script is built on several python modules listed below:
- **paho-mqtt** (from Eclipse Foundation) to handle everything related to the MQTT protocol, broker and client (i.e client creation and connection to broker,
topic publishing/subscribing).
- **wirepas-mesh-messaging** (from Wirepas) to implement the Wirepas gateway API to communicate with a Wirepas gateway (i.e encode and decode packets).
- **Cmd** (from python built-in modules) to create an interactive shell which allows users to list all script commands, their help and enter
commands to send to the network.

## Usage example
In order to be able to receive and send messages at the same time, the script needs to be run
in two separate shells. One will only act as a viewer (i.e it receives, decodes and display messages coming from
a Wirepas Mesh network) and the other as a controller (i.e it asks user input, parses the given command, encodes
it and send the command to a Wirepas Mesh network).

The general and full script command line format is:
```bash
    python script_evaluation_app.py -s \<MQTT broker host address\> -p \<MQTT broker port> -u \<username (optional) \> -pw \<password (optional)\> -fu \<select unsecure connection mode to MQTT broker (i.e disable SSL/TLS) (optional)\>
```
Note: full command line interface can also be obtained with:
```bash
    python script_evaluation_app.py -h
```

The following sections describe how to use the script to receive or/and send messages from/to the Evaluation application message set.

### Receive messages
To receive messages:
1. open a shell either bash or Window PowerShell
2. go to where this script is located on your machine using the "cd" shell command
3. enter a command which follows the format given in the previous section. An example is given (connection to a local MQTT broker with unsecure connection mode):
```bash
    python script_evaluation_app.py -s 192.168.1.30 -h 1883 -fu
```
Note: the parameters given must be identical to the one defined in gateway.env file on the Wirepas gateway.<br>
4. in the new prompt enter:
```
    (cmd) enable_messages_reception
```
Note: this command subscribes the MQTT client to the "gw-event/send_data/+/+/+/1/1" topic where messages sent by nodes running
the Evaluation app are published.

From there the script is endlessly waiting for messages to be received or "Enter" key pressed input to stop reception.

### Send messages
To send messages:
1. Use the same steps as described in the previous section to execute the script
2. In the interactive shell, enter the following command to display the list of supported commands:
```
    (cmd) ?
```
Note: Here only the send_* and bye commands must be used.<br>
3. Enter the command to send to the network. An example is given below (echo command is sent):
```
    (cmd) send_echo_command wp_eval_app_gw sink1 348791499
```
Note: To get each message format see the associated HowTo documentation or enter "help \<command name\>" (e.g help send_echo_command).

From there the script is endlessly waiting for user input. To stop the script, just enter the following command
which will disconnect the client from the MQTT broker:
```
    (cmd) bye
```

## Script execution flow
The script works as follow:
1. get MQTT client parameter from command line (thanks to the mqtt_client_get_parameters() function)
2. create a MQTT client
3. connect to the specified MQTT broker
4. start an interactive shell (thanks to the Backendshell class)
    - if *enable_messages_reception* command is selected:
        - subscribe to the topic where messages are published
        - wait for the **on_message_event_data_received_callback** to be called. <br>
        This is in this function that all the message handling is done i.e decoding thanks to wirepas-mesh-messaging module
        and some custom decoding code (here implemented by the RxMsg class) to parse the data payload
    - if *send_\** command are selected:
        - wait for user input
        - encode the message thanks to the wirepas-mesh-messaging module
        - publish it to the right topic
