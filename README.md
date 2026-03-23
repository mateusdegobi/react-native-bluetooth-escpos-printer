# React Native Bluetooth ESC/POS & TSC Printer

React Native module for thermal Bluetooth printers using ESC/POS and TSC command sets.

> Android only. iOS support is not available.

---

## Table of Contents

- [Installation](#installation)
- [Setup](#setup)
- [BluetoothManager](#bluetoothmanager)
- [BluetoothEscposPrinter](#bluetoothescposprinter)
- [BluetoothTscPrinter](#bluetoothtscprinter)
- [Full Receipt Example](#full-receipt-example)
- [Acknowledgments](#acknowledgments)

---

## Installation

```bash
npm install @mateusdegobi/react-native-bluetooth-escpos-printer --save
```

Or directly from GitHub:

```bash
npm install https://github.com/mateusdegobi/react-native-bluetooth-escpos-printer.git --save
```

React Native 0.60+ handles autolinking automatically. Just rebuild:

```bash
npx react-native run-android
```

---

## Setup

```js
import {
  BluetoothManager,
  BluetoothEscposPrinter,
  BluetoothTscPrinter,
} from "@mateusdegobi/react-native-bluetooth-escpos-printer";
```

---

## BluetoothManager

Manages Bluetooth connectivity: enable/disable, scan, connect, disconnect, and device status.

### isBluetoothEnabled

```js
const enabled = await BluetoothManager.isBluetoothEnabled();
// true or false
```

### enableBluetooth

Enables Bluetooth and returns a list of already paired devices.

```js
const paired = await BluetoothManager.enableBluetooth();
const devices = paired.map((item) => JSON.parse(item));
```

### disableBluetooth

```js
await BluetoothManager.disableBluetooth();
```

### scanDevices

Scans for nearby Bluetooth devices. Returns paired and found devices.

```js
const result = JSON.parse(await BluetoothManager.scanDevices());
console.log(result.paired); // already paired
console.log(result.found);  // discovered nearby
```

### connect

```js
await BluetoothManager.connect("AA:BB:CC:DD:EE:FF");
```

### disconnect

```js
await BluetoothManager.disconnect("AA:BB:CC:DD:EE:FF");
```

### unpaire

Removes pairing with a device.

```js
await BluetoothManager.unpaire("AA:BB:CC:DD:EE:FF");
```

### isDeviceConnected

```js
const connected = await BluetoothManager.isDeviceConnected();
// true or false
```

### getConnectedDevice

Returns the currently connected device or `null`.

```js
const device = await BluetoothManager.getConnectedDevice();
// { name: "Printer", address: "AA:BB:CC:DD:EE:FF" } or null
```

### getConnectedDeviceAddress

Returns only the address of the connected device.

```js
const address = await BluetoothManager.getConnectedDeviceAddress();
```

### Events

Listen for Bluetooth events using `DeviceEventEmitter`:

```js
import { DeviceEventEmitter } from "react-native";

DeviceEventEmitter.addListener(BluetoothManager.EVENT_CONNECTED, (data) => {
  console.log("Connected to:", data.device_name);
});
```

| Event                         | Description                       |
| ----------------------------- | --------------------------------- |
| `EVENT_DEVICE_ALREADY_PAIRED` | Paired devices list               |
| `EVENT_DEVICE_FOUND`          | New device discovered             |
| `EVENT_DEVICE_DISCOVER_DONE`  | Scan completed                    |
| `EVENT_CONNECTED`             | Device connected                  |
| `EVENT_CONNECTION_LOST`       | Connection lost                   |
| `EVENT_UNABLE_CONNECT`        | Connection failed                 |
| `EVENT_BLUETOOTH_NOT_SUPPORT` | Bluetooth not supported on device |

---

## BluetoothEscposPrinter

For receipt/thermal printers using ESC/POS commands.

### printerInit

Initializes the printer. Call before printing.

```js
await BluetoothEscposPrinter.printerInit();
```

### printerAlign

Sets text alignment.

```js
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.CENTER);
```

### printText(text, options)

```js
await BluetoothEscposPrinter.printText("Hello World\n", {
  encoding: "GBK",
  widthtimes: 1,
  heigthtimes: 1,
  fonttype: 0,
});
```

### printColumn(widths, aligns, texts, options)

Prints text in a table-style column layout.

```js
await BluetoothEscposPrinter.printColumn(
  [12, 6, 6, 8],
  [
    BluetoothEscposPrinter.ALIGN.LEFT,
    BluetoothEscposPrinter.ALIGN.CENTER,
    BluetoothEscposPrinter.ALIGN.CENTER,
    BluetoothEscposPrinter.ALIGN.RIGHT,
  ],
  ["Item", "Qty", "Price", "Total"],
  {},
);
```

### printPic(base64, options)

Prints an image from a base64-encoded string.

| Option      | Type   | Description                           |
| ----------- | ------ | ------------------------------------- |
| `width`     | number | Image width in dots                   |
| `left`      | number | Left padding (ignored if `center`)    |
| `center`    | bool   | Center the image horizontally         |
| `autoCut`   | bool   | Auto-cut after printing (default `true`) |
| `paperSize` | number | Paper width in mm (58 or 80)          |

```js
await BluetoothEscposPrinter.printPic(base64Image, {
  width: 384,
  center: true,
  paperSize: 80,
  autoCut: false,
});
```

### printQRCode(content, size, correctionLevel)

```js
await BluetoothEscposPrinter.printQRCode("https://example.com", 200, 1);
```

### printBarCode(str, type, width, height, fontType, fontPosition)

```js
await BluetoothEscposPrinter.printBarCode(
  "1234567890",
  BluetoothEscposPrinter.BARCODETYPE.CODE128,
  3, 120, 0, 2,
);
```

### openDrawer

Opens a cash drawer connected to the printer.

```js
BluetoothEscposPrinter.openDrawer(0, 250, 250);
```

### Constants

```js
BluetoothEscposPrinter.ALIGN       // { LEFT: 0, CENTER: 1, RIGHT: 2 }
BluetoothEscposPrinter.ROTATION    // { OFF: 0, ON: 1 }

BluetoothEscposPrinter.BARCODETYPE
// { UPC_A: 65, UPC_E: 66, JAN13: 67, JAN8: 68,
//   CODE39: 69, ITF: 70, CODABAR: 71, CODE93: 72, CODE128: 73 }

BluetoothEscposPrinter.ERROR_CORRECTION
// { L: 1, M: 0, Q: 3, H: 2 }
```

---

## BluetoothTscPrinter

For label printers using TSC commands.

### printLabel(options)

```js
await BluetoothTscPrinter.printLabel({
  width: 40,
  height: 30,
  gap: 20,
  direction: BluetoothTscPrinter.DIRECTION.FORWARD,
  reference: [0, 0],
  tear: BluetoothTscPrinter.TEAR.ON,
  sound: 0,
  text: [
    {
      text: "Sample Text",
      x: 20,
      y: 0,
      fonttype: BluetoothTscPrinter.FONTTYPE.SIMPLIFIED_CHINESE,
      rotation: BluetoothTscPrinter.ROTATION.ROTATION_0,
      xscal: BluetoothTscPrinter.FONTMUL.MUL_1,
      yscal: BluetoothTscPrinter.FONTMUL.MUL_1,
    },
  ],
  qrcode: [
    {
      x: 20,
      y: 96,
      level: BluetoothTscPrinter.EEC.LEVEL_L,
      width: 3,
      code: "https://example.com",
    },
  ],
  barcode: [
    {
      x: 120,
      y: 96,
      type: BluetoothTscPrinter.BARCODETYPE.CODE128,
      height: 40,
      readable: 1,
      code: "1234567890",
    },
  ],
  image: [
    {
      x: 160,
      y: 160,
      mode: BluetoothTscPrinter.BITMAP_MODE.OVERWRITE,
      width: 60,
      image: base64Image,
    },
  ],
});
```

### Constants

```js
BluetoothTscPrinter.DIRECTION    // { FORWARD: 0, BACKWARD: 1 }
BluetoothTscPrinter.DENSITY      // { DNESITY0..DNESITY15 }
BluetoothTscPrinter.ROTATION     // { ROTATION_0, ROTATION_90, ROTATION_180, ROTATION_270 }
BluetoothTscPrinter.FONTMUL      // { MUL_1..MUL_10 }
BluetoothTscPrinter.BITMAP_MODE  // { OVERWRITE: 0, OR: 1, XOR: 2 }
BluetoothTscPrinter.TEAR         // { ON: "ON", OFF: "OFF" }
BluetoothTscPrinter.EEC          // { LEVEL_L, LEVEL_M, LEVEL_Q, LEVEL_H }

BluetoothTscPrinter.BARCODETYPE
// { CODE128, CODE128M, EAN128, CODE39, CODE93, EAN13, EAN8, CODABAR, ... }

BluetoothTscPrinter.FONTTYPE
// { FONT_1..FONT_8, SIMPLIFIED_CHINESE, TRADITIONAL_CHINESE, KOREAN }
```

---

## Full Receipt Example

```js
await BluetoothEscposPrinter.printerInit();
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.CENTER);
await BluetoothEscposPrinter.printText("My Store\n\r", {
  widthtimes: 3,
  heigthtimes: 3,
});
await BluetoothEscposPrinter.printText("Sales Receipt\n\r", {});
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.LEFT);
await BluetoothEscposPrinter.printText("Customer: Retail\n\r", {});
await BluetoothEscposPrinter.printText("Invoice: INV20251014\n\r", {});
await BluetoothEscposPrinter.printText("--------------------------------\n\r", {});

await BluetoothEscposPrinter.printColumn(
  [12, 6, 6, 8],
  [0, 1, 1, 2],
  ["Product", "Qty", "Price", "Total"],
  {},
);

await BluetoothEscposPrinter.printColumn(
  [12, 6, 6, 8],
  [0, 1, 1, 2],
  ["Widget A", "2", "$12.00", "$24.00"],
  {},
);

await BluetoothEscposPrinter.printColumn(
  [12, 6, 6, 8],
  [0, 1, 1, 2],
  ["Widget B", "4", "$10.00", "$40.00"],
  {},
);

await BluetoothEscposPrinter.printText("--------------------------------\n\r", {});
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.RIGHT);
await BluetoothEscposPrinter.printText("Total: $64.00\n\r", {});
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.CENTER);
await BluetoothEscposPrinter.printText("\n\rThank you!\n\r\n\r", {});
```

---

## Acknowledgments

This project builds upon the work of:

- [Janus J K Lu](https://github.com/januslo/react-native-bluetooth-escpos-printer) — original creator
- [Ccdilan](https://github.com/ccdilan/react-native-bluetooth-escpos-printer) — fork maintainer
- [Farid Fatkhurrozak (Vardrz)](https://github.com/vardrz/react-native-bluetooth-escpos-printer) — custom fork

This fork by [Mateus Degobi](https://github.com/mateusdegobi/react-native-bluetooth-escpos-printer) adds TypeScript support, native fixes, and additional features.

---

## License

MIT
