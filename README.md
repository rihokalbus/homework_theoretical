## Architecture of error handling software
### New components to be added to the system
The following components are required to implement error handling for the [Described System](./C%20homework.pdf):
1. Registry of the systems, located in the Central PC 
2. Systems central health component, located in the Central PC
3. Registry of COTS devices located in each subsystem
4. Embedded controller's health components, located in each subsystem.

Central health component and embedded controllers health components exchange information over the shared bus.
### Embedded controllers health component
Embedded controllers health component relies on the controller's COTS devices registry which is created
from the configuration on the subsystem's startup. COTS devices registry contains device's ID, current status
and information about the last error.

Device's status in the registry can be "Up", "Error l1" or "Error l2". The embedded controller's driver software
is responsible for updating the device status. The health component of an embedded controller determines the state
of the subsystem based on the states of all COTS devices in the controller. Subsystem's health state can be:
* "Up" - all COTS devices are functional
* "Warning" - at least one device has level 1 error 
* "Down" - at least one device has level2 error or is unresponsive (timeout)

If an error occurs in the device, the controller software updates the status of the device in the COTS device
register and notifies the health component.

The health component of the subsystem provides information to the health component of the central PC using
[health reports](https://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/rihokalbus/homework_theoretical/master/health_report.puml).
Health reports are provided either in response to a central component's health
request, or asynchronously, upon receiving a notification from the embedded controller. The health component has
its own common bus listener to exchange messages with the central PC.

The health component must have the ability to initiate a hardware restart of the controller.
### Central health component
The central health component relies on a registry of subsystems which is created from configuration at system startup.
Subsystems registry contains subsystem's ID, current status and subsystem's latest health report data.
The central health component regularly sends a health request to all subsystems and updates the subsystem registry
according to the received data. Subsystems registry data is also changed when asynchronous notification is received
from a subsystem. 

The central health component determines the state of the system based on the states of all subsystems.
System's health state can be:
* "Up" - all subsystems are functional
* "Warning" - at least one subsystem has status "Warning"
* "Down" - at least one subsystem has status "Down" or is unresponsive (timeout)

When the system status changes to "Warning" or "Down", the central computer notifies the user.

## Restart procedures
Restart procedures are initiated by the user. Restart procedures are performed by sending restart messages over the
common bus.
### Function restart
Function restart command RestartFunctionRequest is sent from central PC to subsystem's embedded controller.
Subsystem's embedded controller processes the message and replies with RestartFunctionResponse message. 
### Subsystem restart
Subsystem restart is carried out by sending RestartSubsystemRequest message from central PS to subsystem. Subsystems health
component processes the message and replies with RestartSubsystemResponse message.
### System restart
A complete restart of the system is performed by sending a RestartSubsystemRequest message from the central computer
to all subsystems.