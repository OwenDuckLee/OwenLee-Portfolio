---
title: Winbod SIO Driver read voltage value issue in OS power saving mode
parent: Debug Cases
layout: default
nav_order: 3
---

# Problem Description
## System Environment
- OS: Windows10
- Driver: Winbond SIO Driver customized by Client

## Issue Description
- When system boot-up, driver can read voltage value from SIO correctly
- OS go to power saving mode S3 about 1 min, then turn back to S0: looping this cycle and can read voltage correctly
- After 1hr: driver read voltage value is out of range
- After 4hr: driver read voltage value is out of range

# Verifying Test

## Suspection of bug in SIO driver
  1. Driver internal state machine abnormal, or queue buffer overflow/corruption
  2. driver.c or device.c , where control power management missing
  3. device.c PortIOEvtDeviceCreate()中的power management object was marked
  4. device.c D0Entry() need to remapping SIO register 

## Read Winbond SIO W83627DHG-P data sheet 
- confirm how to control SIO behavior in D0 D3 mode
- Below are reference screen shot from W83627DHG-P Data Sheet#
 ![image](https://github.com/user-attachments/assets/f866454d-987e-4a61-a304-d155755e6fc6) #p.28.29
 ![image](https://github.com/user-attachments/assets/dbe7e28a-f1f1-4c7b-b13a-5e538ed10798) #p.28
 ![image](https://github.com/user-attachments/assets/f459a8f5-909a-4b25-8dd6-caa3a4c74b44) #p.88
 ![image](https://github.com/user-attachments/assets/b1f7332e-70c8-4473-bd0a-928e59b78bf1) #p.78


## Measure HW signal when driver read abnormal value
  Measure HW ADC signal(12V, 5V, 3.3V) by Oscilloscope, 
  -	monitor each voltage signal if is wrong while voltage values(output from SIO driver) is out of range?
  >> if ADC signal all correct while switching between D0 – D3, we should dedicated investigate on driver level firstly.
  >> if ADC signal occurs abnormal, then we should direction to investigate on HW/BIOS firstly.
  ![image](https://github.com/user-attachments/assets/4c5cf763-82f1-4194-b823-39d31414eb9d)


# Solution
- Submit Winbond SIO W83627DHG-P data sheet section for how to re-init SIO HWM LDN
- Help client re-tracing dirver code
- Submit SIO driver modified code for client(re-add power management object / add reset SIO registers to LDN and bank0)

## Final modification in SIO Driver device.c 
- PortIODeviceCreate() -> re-added the initialization of the power management object.
- PortIOEvtDeviceD0Entry() -> add reset SIO registers to LDN and bank0 while each time go back to D0 state.

```c
NTSTATUS
PortIODeviceCreate(
    PWDFDEVICE_INIT DeviceInit
    )
/*++

Routine Description:

    Worker routine called to create a device and its software resources.

Arguments:

    DeviceInit - Pointer to an opaque init structure. Memory for this
        structure will be freed by the framework when the WdfDeviceCreate
        succeeds. So don't access the structure after that point.

Return Value:

    NTSTATUS

--*/
{
    WDF_OBJECT_ATTRIBUTES           deviceAttributes;
    PDEVICE_CONTEXT                 deviceContext;
    WDF_PNPPOWER_EVENT_CALLBACKS    pnpPowerCallbacks;
    WDFDEVICE                       device;
    NTSTATUS                        status;
    UNICODE_STRING                  ntDeviceName;
    UNICODE_STRING                  win32DeviceName;
    WDF_FILEOBJECT_CONFIG           fileConfig;
    WDF_OBJECT_ATTRIBUTES attributes;

    PAGED_CODE();
    
    WDF_PNPPOWER_EVENT_CALLBACKS_INIT(&pnpPowerCallbacks);

    //
    // Register pnp/power callbacks so that we can start and stop the timer as 
    // the device gets started and stopped.
    //
    pnpPowerCallbacks.EvtDevicePrepareHardware = PortIOEvtDevicePrepareHardware;
    pnpPowerCallbacks.EvtDeviceReleaseHardware = PortIOEvtDeviceReleaseHardware;
    pnpPowerCallbacks.EvtDeviceD0Entry = PortIOEvtDeviceD0Entry;
    pnpPowerCallbacks.EvtDeviceD0Exit = PortIOEvtDeviceD0Exit;

    //
    // Register the PnP and power callbacks. Power policy related callbacks will 
    // be registered later in SotwareInit.
    //
    WdfDeviceInitSetPnpPowerEventCallbacks(DeviceInit, &pnpPowerCallbacks);

    WDF_OBJECT_ATTRIBUTES_INIT_CONTEXT_TYPE(&deviceAttributes, DEVICE_CONTEXT);    

    WDF_FILEOBJECT_CONFIG_INIT(
                    &fileConfig,
                    WDF_NO_EVENT_CALLBACK, 
                    WDF_NO_EVENT_CALLBACK, 
                    WDF_NO_EVENT_CALLBACK // not interested in Cleanup
                    );
    
    // Let the framework complete create and close
    fileConfig.AutoForwardCleanupClose = WdfFalse;
    
    WdfDeviceInitSetFileObjectConfig(DeviceInit,
                                     &fileConfig,
                                     WDF_NO_OBJECT_ATTRIBUTES);
    //
    // Create a named deviceobject so that legacy applications can talk to us.
    // Since we are naming the object, we wouldn't able to install multiple
    // instance of this driver. Please note that as per PNP guidelines, we
    // should not name the FDO or create symbolic links. We are doing it because
    // we have a legacy APP that doesn't know how to open an interface.
    //
    RtlInitUnicodeString(&ntDeviceName, GPD_DEVICE_NAME);
    
    status = WdfDeviceInitAssignName(DeviceInit,&ntDeviceName);
    if (!NT_SUCCESS(status)) {
        return status;
    }
    
    WdfDeviceInitSetDeviceType(DeviceInit, GPD_TYPE);
    // Set device object as an exclusive device.
    // Only one device object can be opened at the same time.
    //WdfDeviceInitSetExclusive (DeviceInit, TRUE);

    //
    // Call this if the device is not holding a pagefile
    // crashdump file or hibernate file.
    //
    WdfDeviceInitSetPowerPageable(DeviceInit);

    status = WdfDeviceCreate(&DeviceInit, &deviceAttributes, &device);
    if (!NT_SUCCESS(status)) {
        return status;
    }

    //
    // Get the device context and initialize it. WdfObjectGet_DEVICE_CONTEXT is an
    // inline function generated by WDF_DECLARE_CONTEXT_TYPE macro in the
    // device.h header file. This function will do the type checking and return
    // the device context. If you pass a wrong object  handle
    // it will return NULL and assert if run under framework verifier mode.
    //
    deviceContext = PortIOGetDeviceContext(device);

    //
    // This values is based on the hardware design.
    // I'm assuming the address is in I/O space for our hardware.
    // Refer http://support.microsoft.com/default.aspx?scid=kb;en-us;Q323595
    // for more info.
    //        
    deviceContext-> PortMemoryType = 1; 
    deviceContext->PortBase = NULL;
    deviceContext->PortWasMapped = FALSE;

    //
    // Create a device interface so that application can find and talk
    // to us.
    //
    RtlInitUnicodeString(&win32DeviceName, DOS_DEVICE_NAME);
    
    status = WdfDeviceCreateSymbolicLink(
                device,
                &win32DeviceName);

    if (!NT_SUCCESS(status)) {
        return status;
    }

    //Power management initialization
    WDF_DEVICE_POWER_POLICY_IDLE_SETTINGS_INIT(&idleSettings, IdleCannotWakeFromS0);
    idleSettings.IdleTimeout = 300000;
    idleSettings.UserControlOfIdleSettings = IdleDoNotAllowUserControl;
    idleSettings.Enabled = WdfTrue;
    status = WdfDeviceAssignS0IdleSettings(device, &idleSettings);
    if (!NT_SUCCESS(status))
    {
        KdPrint(("WdfDeviceAssignS0IdleSettings failed 0x%x\n", status));
        return status;
    }

    WDF_DEVICE_POWER_CAPABILITIES_INIT(&powerCaps);
    powerCaps.DeviceD1 = WdfFalse;
    powerCaps.DeviceD2 = WdfFalse;
    powerCaps.DeviceWake = PowerDeviceD3;
    powerCaps.DeviceState[PowerSystemWorking] = PowerDeviceD0;
    powerCaps.DeviceState[PowerSystemSleeping1] = PowerDeviceD3;
    powerCaps.DeviceState[PowerSystemSleeping2] = PowerDeviceD3;
    powerCaps.DeviceState[PowerSystemSleeping3] = PowerDeviceD3;
    powerCaps.DeviceState[PowerSystemHibernate] = PowerDeviceD3;
    powerCaps.DeviceState[PowerSystemShutdown] = PowerDeviceD3;
    powerCaps.SystemWake = PowerSystemSleeping1; //Set wake capabilities.
    status = WdfDeviceSetPowerCapabilities(device, &powerCaps);
    if (!NT_SUCCESS(status)) {
        KdPrint(("WdfDeviceSetPowerCapabilities failed 0x%x\n", status));
        return status;
    }

    WDF_OBJECT_ATTRIBUTES_INIT(&attributes);
    attributes.ParentObject = device;
    status = WdfSpinLockCreate(&attributes, &deviceContext->PowerLock);
    if (!NT_SUCCESS(status)) {
        KdPrint(("WdfSpinLockCreate failed 0x%x\n", status));
        return status;
    }

    // Initialize the I/O Package and any Queues
    status = PortIOQueueInitialize(device);

    return status;
}
```

```c
NTSTATUS
PortIOEvtDeviceD0Entry(
    __in WDFDEVICE Device,
    __in WDF_POWER_DEVICE_STATE PreviousState
    )
{
   PDEVICE_CONTEXT devContext = PortIOGetDeviceContext(Device);
   NTSTATUS status = STATUS_SUCCESS;

   PAGED_CODE();

   KdPrint(("Entering D0 from %s\n", DbgDevicePowerString(PreviousState)));

   WdfSpinLockAcquire(devContext->PowerLock);

   if (PreviousState != WdfPowerDeviceD0)
   {
      if (devContext->PortBase == NULL)
      {
         KdPrint(("Remapping resources after D3->D0\n"));

         if (devContext->PortIsMemory)
         {
            // Remap memory resources
            devContext->PortBase = MmMapIoSpace(
               devContext->PortStartAddress,
               devContext->PortLength,
               MmNonCached);

            if (devContext->PortBase == NULL) {
               KdPrint(("Failed to remap memory resources\n"));
               status = STATUS_INSUFFICIENT_RESOURCES;
            }
            else {
               devContext->PortWasMapped = TRUE;
               KdPrint(("Successfully remapped memory resources at 0x%p\n",
                  devContext->PortBase));
            }
         }
         else
         {
            // For I/O ports, just set the base address
            devContext->PortBase = ULongToPtr(devContext->PortStartAddress.LowPart);
            devContext->PortWasMapped = FALSE;
            KdPrint(("Restored I/O port base address to 0x%p\n",
               devContext->PortBase));
         }
      }

      if (NT_SUCCESS(status) && devContext->PortBase != NULL)
      {
        //every time get into D0 state, re-init SIO register, e.g LDN, Bank0
        PUCHAR indexPort = (PUCHAR)devContext->PortBase;
        PUCHAR dataPort  = indexPort + 1;
        KdPrint(("Reinitializing SIO at IndexPort=0x%p, DataPort=0x%p\n", indexPort, dataPort));
        
        //get into SIO config mode
        WRITE_PORT_UCHAR(indexPort, 0x87);
        WRITE_PORT_UCHAR(indexPort, 0x87);

        //select LDN 0x0B(Hardware Monitor)
        WRITE_PORT_UCHAR(indexPort, 0x07);
        WRITE_PORT_UCHAR(dataPort, 0x0B); 

        //select Bank0
        WRITE_PORT_UCHAR(indexPort, 0x4E);
        WRITE_PORT_UCHAR(dataPort, 0x00);

        //exit config mode 
        WRITE_PORT_UCHAR(indexPort, 0xAA); // Exit config mode

        KdPrint(("Reinitialized SIO LDN to 0x0B and Bank0\n"));
        KdPrint(("Hardware reinitialized after D3->D0\n"));
      }
   }
   WdfSpinLockRelease(devContext->PowerLock);
   return status;
}
```
