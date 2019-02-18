---
title: "Train plugin API (IRuntimeTrain) and Route plugin API (IRuntimeRoute)"
hidden: true
---

This is a document of train plugin and route plugin. To create a train plugin, implement the IRuntimeTrain interface from the namespace OpenBveApi.Runtime. To create a route plugin, implement the IRuntimeRoute interface from the namespace OpenBveApi.Runtime.

{{% warning %}}

#### IRuntime vs. IRuntimeTrain and IRuntimeRoute

IRuntime and IRuntimeTrain are not compatible. IRuntimeTrain and IRuntimeRoute can be used with openBVE 1.5.4.x or later. Please be aware that an error will occur when reading in the previous version.

{{% /warning %}}

## ■ Overview

When the plug-in is loaded, the following functions are called in this order:

- IRuntimeRoute
  - Load
  - Initialize
- IRuntimeTrain
  - Load
  - SetVehicleSpecs
  - Initialize
  - SetPower
  - SetBrake
  - SetReverser

In every frame, the following functions are called in this order:

- IRuntimeRoute
  - Elapse
- IRuntimeTrain
  - Elapse

The following functions are called at any time in the following order:

- IRuntimeRoute
  - SetBeacon
- IRuntimeTrain
  - SetBeacon

The following functions can be called at any time.

- IRuntimeTrain
  - SetPower
  - SetBrake
  - SetReverser
  - KeyDown
  - KeyUp
  - HornBlow
  - DoorChange
  - SetSignal
  - PerformAI

NOTE by S520: Route plugins are not called before these functions at the moment. Please consult me if it is better to implement.

When the plug-in is unloaded, the following functions are called in this order:

- IRuntimeTrain
  - Unload
- IRuntimeRoute
  - Unload

## ■ Function calls - IRuntimeTrain

Since IRuntimeTrain is an interface that extends IRuntime, here is a list of only different functions and their operation. Functions not described are the same as IRuntime, so please see here.

------

**void Elapse(ElapseData data, byte[] recieveData)**

This function is called in all frames. It sends the current state of the train to the vehicle plug-in and set the state of the steering wheel. In addition, it can receive data from the route plugin.

arguments:

{{% table-nonheader %}}

| ElapseData | data        | Data passed from the OpenBVE to the plug-in  |
| ---------- | ----------- | -------------------------------------------- |
| byte[]     | recieveData | Acquire data received from the route plug-in |

{{% /table-nonheader %}}

------

**event EventHandler\<TxEventArgs> Transmitter**

This calls the IRuntimeRoute function Receiver.

TxEventArgs (class):

{{% table-nonheader %}}

| byte[] | SendData | Set data to be sent to the route plug-in. |
| ------ | -------- | ----------------------------------------- |
|        |          |                                           |

{{% /table-nonheader %}}

<br/>

{{% code "Implementation example" %}}

```c#
public event EventHandler<TxEventArgs> Transmitter;

protected virtual void UseTransmitter(TxEventArgs e) {
    if (Transmitter != null) {
        Transmitter(this, e);
    }
}

// usage
using System;
using System.Linq;
bool A = true;
int B = 123;
byte[] sendData = BitConverter.GetBytes(A).Concat(BitConverter.GetBytes(B)).ToArray();
UseTransmitter(new TxEventArgs(sendData));
```

{{% /code %}}

## ■ Function calls - IRuntimeRoute

The following is a list of all functions and explanations on their operation.

------

**bool Load()**

This function will be called the first time after the plugin is loaded.

NOTE by S520: The argument is temporary. Please point out if there is something better to get at the time of loading.

------

**void Unload()**

This function is the last one to be called before the plugin is unloaded.

------

**void Initialize()**

This function is called after Load to initialize the plugin. When the user uses the "Jump to Station" function, this function will also be called before moving the vehicle to a new location.

NOTE by S520: The argument is temporary. Please point out if there is something better to get at initialization.

------

**void Elapse(ElapseDataRoute data, out byte[] sendData)**

This function is called in all frames. It sends the vehicle plug-in of the current state of the route and set the signal index. In addition, it can send data to the vehicle plug-in.

arguments:

{{% table-nonheader %}}

| ElapseDataRoute | data     | Data passed to the plug-in from the OpenBVE. |
| --------------- | -------- | -------------------------------------------- |
| byte[]          | sendData | Set data to be sent to the vehicle plug-in.  |

{{% /table-nonheader %}}

ElapseDataRoute (class):

{{% table-nonheader %}}

| Section[] | Sections | Information on all blocks ahead. |
| --------- | -------- | -------------------------------- |
|           |          |                                  |

{{% /table-nonheader %}}

Section (struct):

{{% table-nonheader %}}

| SectionAspect[] | Aspects | Information on all signal indices of each block. |
| --------------- | ------- | ------------------------------------------------ |
|                 |         |                                                  |

{{% /table-nonheader %}}

SectionAspect (struct):

{{% table-nonheader %}}

| int    | Number | Get and set the number of each signal index.                 |
| ------ | ------ | ------------------------------------------------------------ |
| double | Speed  | Acquires and sets the speed limit corresponding to each signal index. |

{{% /table-nonheader %}}

NOTE by S520: Members of classes and structures are temporary. Please point out if there is something better to make it available.

------

**void SetBeacon(BeaconDataEx data)**

This function is called when the train passes over Beacon and you can set the type and options of Beacon.

argument:

{{% table-nonheader %}}

| BeaconDataEx | data | Data of a Beacon |
| ------------ | ---- | ---------------- |
|              |      |                  |

{{% /table-nonheader %}}

BeaconDataEx (class):

{{% table-nonheader %}}

| int        | Type     | Get and set the type of Beacon.           |
| ---------- | -------- | ----------------------------------------- |
| int        | Optional | Get and set optional data sent by Beacon. |
| SignalData | Signal   | Obtain a block associated with Beacon.    |

{{% /table-nonheader %}}

SignalData (class):

{{% table-nonheader %}}

| int    | Aspect   | Obtain signal indices of block.  |
| ------ | -------- | -------------------------------- |
| double | Distance | Obtain the distance to blockage. |

{{% /table-nonheader %}}

------

**void Receiver(byte[] receiveData)**

This function is called on IRuntimeTrain event Transmitter. You can receive data from the tarin plug-in.

引数:

{{% table-nonheader %}}

| byte[] | recieveData | Acquire data received from the vehicle plug-in. |
| ------ | ----------- | ----------------------------------------------- |
|        |             |                                                 |

{{% /table-nonheader %}}

<br/>

{{% code "Implementation example" %}}

```c#
using System;

public void Receiver(byte[] receiveData) {
    if (receiveData.Length < 5) {
        return;
    }
    bool A = BitConverter.ToBoolean(receiveData, 0);
    int B = BitConverter.ToInt32(receiveData, 1);
}
```

{{% /code %}}