# iCue_Nexus_RevEng
How the Corsair iCue Nexus works (as of SDK v3.0.378)

# Corsair Nexus & iCUE: Developer's Reference Guide

## 1. Hardware Specifications

- **Display**: 640×48 pixel touchscreen (5" diagonal)
- **Interface**: USB (Vendor ID: 0x1B1C, Product ID: 0x1B8E)
- **Connection**: Direct USB or via compatible Corsair keyboard
- **Input**: Capacitive touch

## 2. Software Architecture

- **iCUE**: Central software that manages all Corsair devices
- **Device Classification**: Nexus is classified as CDT_Touchbar (0x0800) in the SDK
- **LED Group**: CLG_Touchbar (14) for display control
- **Key Components**:
  - `NexusHub.dll` - Core functionality
  - `nexusImages.dll` - Image processing and display

## 3. Display Control Methods

### Official SDK Approach
```cpp
// 1. Connect to iCUE
CorsairConnect(onStateChanged, context);

// 2. Find Nexus device
CorsairDeviceFilter filter = { CDT_Touchbar };
CorsairGetDevices(&filter, CORSAIR_DEVICE_COUNT_MAX, devices, &deviceCount);

// 3. Update display pixels
CorsairLedColor pixels[640*48]; // Each LED represents a pixel
// Set pixel colors...
CorsairSetLedColors(nexusId, pixelCount, pixels);
```

### Display Addressing
- Pixels are addressed as LEDs with unique IDs
- Format: `(CLG_Touchbar << 16) | pixelIndex`
- Display buffer uses standard RGB color format

## 4. Touch Input Handling

```cpp
// Register for touch events
CorsairSubscribeForEvents(onEvent, context);

// In the event handler:
void onEvent(void* context, const CorsairEvent* event) {
    if (event->id == CEI_KeyEvent) {
        // Handle touch event
        auto keyEvent = event->keyEvent;
        // Touch position data is encoded in the keyId
        processTouchInput(keyEvent->keyId, keyEvent->isPressed);
    }
}
```

## 5. Creating Custom Widgets

### Widget Structure
1. **Screen**: Collection of buttons/elements
2. **Button**: Interactive area with visual representation
3. **Action**: What happens when button is touched

### Implementation Options
1. **Official SDK**: Create C++ application using iCUE SDK
2. **Export Files**: Create .cuescreens files with custom buttons
3. **Direct USB**: Bypass iCUE for full control (advanced)

## 6. File Formats

- **Screens Export**: .cuescreens file (ZIP-based container)
- **Structure**:
  - screen_settings: Configuration data
  - button_attachments: Action definitions
  - nexus_images: PNG images for buttons (typically 100×48px)

## 7. Technical References

### SDK Headers
- `iCUESDK.h`: Main SDK interface
- `iCUESDKLedIdEnum.h`: LED/pixel addressing
- `iCUESDKGlobal.h`: Platform-specific definitions

### Key DLLs
- `iCUESDK.x64_2019.dll`: Main SDK library
- `NexusHub.dll`: Nexus device management
- `nexusImages.dll`: Image processing for display

## 8. Development Best Practices

1. **Resolution Awareness**: Always design for 640×48px display
2. **Touch Target Size**: Minimum 48px wide for comfortable touch
3. **Text Legibility**: Use fonts at least 16px high
4. **Battery Impact**: Custom widgets can increase power consumption
5. **Compatibility**: Test across iCUE versions

## 9. Advanced Techniques

### Direct USB Control
For bypassing iCUE completely:
1. Use Zadig to install WinUSB driver
2. Use USBlyzer to capture communication protocol
3. Implement protocol in custom software
4. Note: This disables normal iCUE functionality

### Custom Screen Creation
1. Design screen layout (640×48px)
2. Slice into button images (100×48px)
3. Create screen configuration
4. Package and export as .cuescreens

## 10. Useful Tools

- **USBlyzer**: USB protocol analysis
- **Process Monitor**: Track iCUE file/registry operations
- **Ghidra/IDA Pro**: DLL analysis if needed
- **Wireshark + USBPcap**: USB packet capture

---

This guide represents current knowledge based on SDK analysis. Corsair may update functionality in future iCUE releases.
