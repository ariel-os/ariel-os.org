+++
date = '2026-01-27T15:28:40+01:00'
draft = false
title = "Ariel OS 0.3.0: Smart sensors"
# tags = ["Release"]
+++

Today we are happy to announce a new version of Ariel OS. With over 6 months of work and 400 merged PRs, this new version brings shiny features and improvements. These include upgraded Embassy and esp-hal dependencies, UART support, a brand new way to write and access sensors, Bluetooth Low Energy support, structured board descriptions and much more!

On top of those new features, this release adds support to more MCUs and boards, some of those were added by the community!

You can check out the full changelog of [Ariel OS 0.3.0](https://github.com/ariel-os/ariel-os/releases/tag/v0.3.0).

Let's go over some of the highlights of this release.

## Upgraded Embassy and esp-hal

This was the major blocker for this new release, making everything work with as little breakage as possible, most applications built for Aiel OS v0.2.0 should work with Ariel OS v0.3.0 with little to no modification. You can check the full list of breaking changes in [the changelog](https://github.com/ariel-os/ariel-os/releases/tag/v0.3.0).

Ariel OS v0.3.0 uses `esp-hal` v1.0.0, `embassy-nrf` v0.8.0, `embassy-rp` v0.8.0, `embassy-stm32` v0.4.0.

With esp-hal becoming stable, upgrades to newer versions should come faster. Embassy isn't stable yet but it's now faster release schedule should make upgrades faster and easier for us.

## UART support

UART peripherals are now handled directly by Ariel OS when supported, you can check if your board is supported [in the hardware support chapter of our book](https://ariel-os.github.io/ariel-os/dev/docs/book/hardware-functionality-support.html). This allows you to easily write and read from UART, without having to worry about which hardware and HAL will be used.

Basic usage looks like this (full example in [`tests/uart-loopback`](https://github.com/ariel-os/ariel-os/tree/main/tests/uart-loopback)):

```rust
#[ariel_os::task(autostart, peripherals)]
async fn main(peripherals: pins::Peripherals) {
    let mut config = hal::uart::Config::default();
    config.baudrate = Baudrate::_115200;
    let mut rx_buf = [0u8; 32];
    let mut tx_buf = [0u8; 32];
    let mut uart = pins::TestUart::new(
        peripherals.uart_rx,
        peripherals.uart_tx,
        &mut rx_buf,
        &mut tx_buf,
        config,
    )
    .expect("Invalid UART configuration");
    const OUT: &str = "Test Message";
    let mut input = [0u8; OUT.len()];
    uart.write_all(OUT.as_bytes()).await.unwrap();
    uart.flush().await.unwrap();
    uart.read_exact(&mut input)).await;
}
```

## The sensor abstraction

After a few rounds of design, the new sensors abstraction API is now available for you to try!

All sensors using this system are available through a common registry and an universal API, want to know the current temperature? Just select any sensor in the registry that has the category `Temperature`!

```rust
let sensor = REGISTRY
    .sensors()
    .find(|s| s.categories().contains(&Category::Temperature))
    .unwrap();
sensor.trigger_measurement().unwrap();
let samples = sensor.wait_for_reading().await.unwrap();

for (reading_channel, sample) in samples
    .samples()
    .filter(|(reading_channel, _)| reading_channel.label() == Label::Temperature)
{
    info!(
        "Current temperature: {}{}",
        sample.value().unwrap(),
        reading_channel.unit()
    );
}
```

You can find a more detailed example in [`examples/thermometer`](https://github.com/ariel-os/ariel-os/tree/main/examples/thermometer) and read more about using and implementing sensor drivers in the [`sensors` module documentation](https://ariel-os.github.io/ariel-os/dev/docs/api/ariel_os/sensors/index.html).

Currently the sensors have to be initialized manually in the application. The long term goal is to have the sensor initialization code automatically integrated into the app and the list of sensors to be initialized would be handled in the board descriptions.

Speaking of board descriptions...

## Structured board descriptions

Structured Board Description or SBD is a new format created to describe how components on a board are wired to the MCU. Currently supported components are LEDs, buttons, USB ports and configuration of the software interrupt. This allows us to have some simple conditions to filter hardware: need an LED to display the status of your application? Select the "has_leds" module. Need some basic interaction? Select the "has_buttons" laze module.
The access to leds and buttons is also generated, you can access them using `ariel_os_boards::pins::LedPeripherals` and `ariel_os_boards::pins::ButtonPeripherals`, with a new macro `ariel_os::hal::group_peripherals!` for when your task needs peripherals from multiple sources.

Our [blinky example](https://github.com/ariel-os/ariel-os/tree/main/examples/blinky) is now even simpler, the pins.rs file with manual pins declaration is not needed anymore!

```rust
#[ariel_os::task(autostart, peripherals)]
async fn blinky(peripherals: ariel_os_boards::pins::LedPeripherals) {
    let mut led0 = Output::new(peripherals.led0, Level::Low);

    loop {
        led0.toggle();
        Timer::after_millis(500).await;
    }
}
```

## Bluetooth Low Energy

This release also adds supports for Bluetooth Low Energy, BLE. On [supported hardware](https://ariel-os.github.io/ariel-os/dev/docs/book/hardware-functionality-support.html) (check for the "Bluetooth Low Energy" column) the bluetooth hardware is automatically initialized when selecting a BLE-related laze module: `ble-peripheral` or `ble-central`, you can then obtain a [`trouble_host::Stack`](https://docs.embassy.dev/trouble-host/0.1.0/default/struct.Stack.html) using `ariel_os::ble::ble_stack().await;`, you can interact with this `Stack` to use the bluetooth functionality.

Here's a simple task that advertises a "peripheral", you can find more detailed examples in [`examples/ble-advertiser`](https://github.com/ariel-os/ariel-os/tree/main/examples/ble-advertiser) and [`examples/ble-scanner`](https://github.com/ariel-os/ariel-os/tree/main/examples/ble-scanner):

```rust
let mut host = ariel_os::ble::ble_stack().await.build();
let mut adv_data = [0; 31];
let len = AdStructure::encode_slice(&[AdStructure::CompleteLocalName(b"Ariel OS BLE"), AdStructure::Flags(LE_GENERAL_DISCOVERABLE | BR_EDR_NOT_SUPPORTED),],&mut adv_data[..],).unwrap();

let _ = join(host.runner.run(), async {
    let params = AdvertisementParameters::default();
    let _advertiser = host.peripheral.advertise(&params, Advertisement::NonconnectableScannableUndirected { adv_data: adv_data.get(..len).unwrap(), scan_data: &[],},).await;
    loop {
        info!("Advertising");
        Timer::after_secs(60).await;
    }
})
.await;
```

You can check the [dedicated page of the book](https://ariel-os.github.io/ariel-os/dev/docs/book/bluetooth.html) to learn more about BLE in Ariel OS.

## Hardware support

Hardware support is growing with addition from the core team and the community!

- [@royb3](https://github.com/royb3) added support for the STM32H753ZI MCU and the ST NUCLEO-H753ZI board.
- [@bugadani](https://github.com/bugadani) added support for the ESP32-S2, ESP32-S2Fx2, ESP32-S2Fx4, ESP32-S2Fx4R2 MCUs, the ESP32-S2-SOLO-2 hardware module, and the Espressif ESP32-S2-DevKitC-1,ESP32-C3-DevKit-RUST-1 boards
- [@baptleduc](https://github.com/baptleduc) added support for the Seeed Studio LoRa-E5 mini board.
- [@dgtlrift](https://github.com/dgtlrift) added support for the Heltec WiFi LoRa 32 V3 board.
- [@therealprof](https://github.com/therealprof) added support for the STM32F042K6 MCU and the ST NUCLEO-F042K6, BBC micro:bit V1 boards.
- [@Ollrogge](https://github.com/Ollrogge) added support for the FPU on compatible Cortex-M MCUs.
- [@SimonIT](https://github.com/SimonIT) added support for `getrandom` on MCUs that have an HWRNG.

Thank you for those contributions, we also want to thank [@KiyoLelou10](https://github.com/KiyoLelou10) and [@tshepang](https://github.com/tshepang) for their contributions to other parts of Ariel OS.

## Thank you

Thank you for reading this blog post, for being part of this community, we hope you will enjoy using this new version of Ariel OS!
