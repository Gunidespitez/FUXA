# OT WRITEUP
## Task: Create a Smart Valve System Using FUXA SCADA
This system will allow monitoring and controlling a valve through a graphical interface. The steps include:
* Installing and configuring FUXA SCADA.
* Creating a basic HMI (Human-Machine Interface) to display and control the valve.
* Simulating valve operation to visualize its behavior.
* Developing a simulated PLC (in Python) and integrating it with FUXA SCADA for real-time control and monitoring.
## WHAT IS A SCADA BASED SYSTEM?
SCADA (Supervisory Control and Data Acquisition) is a computer-based system used for monitoring, controlling, 
and analyzing industrial processes in real-time. It provides control over collecting
data from field devices (like sensors and PLCs) and presenting it to operators via an HMI (Human-Machine Interface).
## WHY USE SCADA AND NOT DCS?
SCADA is a centralized system for monitoring and control, often used in distributed industrial environments.
While, DCS (Distributed Control System) is more suited for continuous processes like chemical plants, 
where control logic is distributed across multiple controllers.
## WHAT IS FUXA?
FUXA is an open-source, web-based SCADA system designed for monitoring and controlling industrial automation processes.
It is lightweight, easy to use, and supports multiple communication protocols like Modbus, OPC UA,etc. 
We choose to use FUXA over other SCADA based systems like Ignition and WinCC because we do not require licensing and 
it is also lightweighted to use.
## What is Modbus?
Modbus is a communication protocol used in industrial automation to exchange data between PLCs, SCADA systems, sensors, and other devices. It was developed by Modicon 
(now Schneider Electric) in 1979 and is one of the most widely used open protocols in industrial settings.
Its basic working principle is : 
* A Master (SCADA/PLC) sends requests to a Slave (sensor/device).
* The Slave responds with the requested data.
* Uses a register-based addressing system for data exchange.
## WHY TCP?
For our project, we use Modbus TCP/IP because:
* It is simple & well-supported in SCADA systems like FUXA.
* It allows direct communication between PLCs and SCADA.
* Modbus is simpler, and we don’t need advanced security or IIoT features.
* While MODBUS RTU is best for short distances, real-time control ; TCP is best for long distances & modern networks
## Basic Valve System in FUXA SCADA
* Switch – Controls the valve (ON/OFF).
* Valve – Changes state based on the switch.
Closed (Red) when OFF
Open (Green) when ON
*pipes – Show flow animation when the valve is ON.
We add the following elements and then configure them:
* For the switch:
* Set it as a toggle switch (ON/OFF).
* Bind it to a tag which is same for all three components, named valve_state
* ON → Writes 1 (Valve Opens).
* OFF → Writes 0 (Valve Closes).
* Similar configuration for valve and pipes
## Pymodbus to control the valve
### MY CODE:
from pymodbus.server.sync import StartTcpServer
from pymodbus.datastore import ModbusSequentialDataBlock
from pymodbus.datastore import ModbusSlaveContext, ModbusServerContext
import threading
import time

reg = ModbusSlaveContext(co=ModbusSequentialDataBlock(0, [0, 0, 0]))
serv = ModbusServerContext(slaves=reg, single=True)

def coil():
    while 1>0:
        status = reg.getValues(1, 0, count=1)[0] 
        reg.setValues(1, 1, [status])
        reg.setValues(1, 2, [status])
        print(f"Switch: {status}, Valve: {status}, Pipe Flow: {status}")
        time.sleep(1)


threading.Thread(target=coil,daemon=True).start()
print("Modbus server running on port 5025...")
StartTcpServer(serv, address=("0.0.0.0", 5028))

![Screenshot from 2025-03-03 17-00-34](https://github.com/user-attachments/assets/6f63f302-ed49-4952-9154-54c8a6d3155f)



### WHAT DOES MY SYSTEM DO?

https://github.com/user-attachments/assets/f2fab2f0-7db6-448f-8ccc-048e2f636819

User clicks the sitch in FUXA
SCADA → Sends a signal to the Modbus server.
Modbus server updates the coil register (00001) → Changes value from 0 to 1.
FUXA reads the coil value → If 1, the valve opens and pipes start flowing.
If user turns the switch OFF → Coil value changes to 0, valve closes, and pipes stop flowing.
The system keeps running in real-time through continuous communication between FUXA and the Modbus server.

## What is PyModbus?
PyModbus is a Python library for Modbus communication, allowing you to:
Create Modbus clients (to send requests).
Create Modbus servers (to respond to requests).
Read and write data to coils (ON/OFF) and registers (numeric values).
### KEY CONCEPTS:
* ModbusTcpClient-
* This class is used to connect to a Modbus server (my Python script)
* ModbusServerContext
* This class stores and manages Modbus data (coils, registers).
* client.connect()
* Establishes a connection to the Modbus server.
* client.read_coils(address, count)
* Reads the ON/OFF state of coils (like switches & valves).
* client.write_coil(address, value)
* Writes a new value (ON/OFF) to a coil.
* client.read_holding_registers(address, count)
* Reads numeric values stored in holding registers.
* client.write_register(address, value)
* Writes a numeric value to a holding register.
* client.close()
* Closes the connection to the server.
## SMART VALVE SYSTEM
https://github.com/user-attachments/assets/26963173-78bb-4777-962d-162f719119f0
* Initial Setup
    - Tank 1 starts full (100L), Tank 2 is empty.
    - The valve is initially closed (RED).
    - If the switch is turned ON, the valve opens (GREEN), and water starts flowing from Tank 1 to Tank 2.
    - As Tank 2 fills up, Tank 1’s level decreases.
    - When Tank 2 reaches 100L, the valve automatically closes to prevent overflow.
    -  Tank levels update in real-time on FUXA SCADA.
    - The graph shows water level trends over time.

## SUMMARY
The simple valve system uses FUXA SCADA and Python (PyModbus) to control a valve and pipe flow based on a switch. When the switch is turned ON in FUXA, a Modbus command is sent to the Python-based Modbus server, which updates the valve state to open (green) and starts the pipe flow animation. If the switch is turned OFF, the valve closes (red) and the pipe stops flowing. The system uses Modbus coils to store the switch, valve, and pipe flow states, updating them in real time

