+++
date = '2026-05-26T14:26:12+02:00'
draft = false
title = 'Ariel OS at RustWeek 2026'
+++

Last week Ariel OS was present at [RustWeek 2026] with a community booth.
You could find us opposite to the main track room,
trying to lure visitors to the booth with our live climate measurement demo on the screen.
Plenty of visitors came to our booth to ask about Ariel OS
and the demos we had prepared.
<!--more-->

## Demos

Nils brought his [CDC Badge] with his own firmware based on Ariel OS.
[The firmware][ble-scan] showcased a Bluetooth device scanner based on the [TrouBLE] Bluetooth Low Energy (BLE) stack,
with the graphics using [Ratatui] on the e-paper display.
In the crowded venue the screen was filled with all kinds of headsets and smart watches most of the time.

Our other demo showcased how easy it is to get a simple sensor measurement setup going with Ariel OS.
It consisted of a Nordic [nRF52840 development kit], connected to the laptop via its USB peripheral.
Hooked up to the board was a Bosch [BME280] humidity, temperature and pressure sensor together with an [SCD40] CO2 sensor from Sensirion.
The board pushed its measurements to the laptop over [CoAP],
which could then display it live on the television screen provided to us.

{{< figure
  src="nrf52840dk.jpg"
  alt="nRF52840 Development Kit with sensors"
  caption="nRF52840 Development Kit with sensors"
  height=620
  width=674
>}}

## Experience

We had a great time together with [Espressif], [Infineon], [OxidOS] and the other booths present at RustWeek.
Meeting in person with the developers behind these companies allowed us to discuss what to expect from them regarding Rust support.

Of course also a great thank you to the RustWeek organization
and the generous opportunity to have us present with the community booth.

Looking for the next opportunity to meet us?
We're organizing the first Ariel OS community day at September 1, 2026 in Grenoble.
See our page on the [1AD] for more information!

[RustWeek 2026]: https://2026.rustweek.org/
[CDC Badge]: https://github.com/riatlabs/cdc-badge
[ble-scan]: https://github.com/ariel-os/cdc-badge-demo/tree/feat/ble-scan
[TrouBLE]: https://github.com/embassy-rs/trouble
[Ratatui]: https://ratatui.rs/
[nRF52840 development kit]: https://www.nordicsemi.com/Products/Development-hardware/nRF52840-DK
[BME280]: https://www.bosch-sensortec.com/en/products/environmental-sensors/humidity-sensors-bme280
[SCD40]: https://sensirion.com/products/catalog/SCD40
[CoAP]: https://coap.space/
[Espressif]: https://www.espressif.com/
[Infineon]: https://www.infineon.com/
[OxidOS]: https://oxidos.io/
[1AD]: https://ariel-os.org/1ad/
