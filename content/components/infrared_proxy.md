---
description: "Instructions for setting up the Infrared Proxy in ESPHome."
title: "Infrared Proxy"
params:
  seo:
    description: Instructions for setting up the Infrared Proxy in ESPHome.
    image: infrared.svg
---

ESPHome's infrared proxy component works with Home Assistant to expand its remote control capabilities. This component
provides a unified API-accessible interface for transmitting and receiving infrared and RF signals, acting as a bridge
between Home Assistant (or other API clients) and ESPHome's existing [remote_receiver](/components/remote_receiver) and
[remote_transmitter](/components/remote_transmitter) components. Note that at least one of these components is required
in your device's configuration if you wish to use the infrared proxy component.

The infrared proxy enables runtime signal transmission without recompiling and/or reinstalling (flashing) firmware,
making it ideal for learning and replaying IR/RF commands, creating universal remote controls, and integrating with
Home Assistant's remote control features.

```yaml
# Example configuration entry
infrared_proxy:
  # IR transmitter instance
  - id: living_room_ir
    name: Living Room IR Transmitter
    remote_transmitter_id: ir_tx

  # IR receiver instance
  - id: ir_learner
    name: IR Receiver
    remote_receiver_id: ir_rx
```

### Configuration variables:

- **id** (**Required**, [ID](/docs/configuration-types#id)): Unique identifier for the infrared proxy instance.
- **name** (*Optional*, string): The name of the infrared proxy instance.
- **remote_transmitter_id** (*Optional*, [ID](/docs/configuration-types#id)): The ID of the
  [remote_transmitter](/components/remote_transmitter) component to use for sending signals. Either this or
  `remote_receiver_id` must be specified, but not both.
- **remote_receiver_id** (*Optional*, [ID](/docs/configuration-types#id)): The ID of the 
  [remote_receiver](/components/remote_receiver) component to use for receiving signals. Either this or
  `remote_transmitter_id` must be specified, but not both.
- **frequency** (*Optional*, int): The operating frequency in Hz. Set to a non-zero value for RF hardware. Defaults to
  `0` (infrared). This value is passed back to Home Assistant, allowing it to identify integrations this infrared proxy
  instance can potentially support; it has no impact on the hardware or device configuration itself.
- **icon** (*Optional*, icon): Manually set the icon for this entity.
- **entity_category** (*Optional*, string): The category of the entity. See 
  [entity categories](/docs/configuration-types#entity-categories) for more information.
- **disabled_by_default** (*Optional*, boolean): If the entity should be disabled by default. Defaults to `false`.

## How It Works

The infrared proxy component creates API-accessible entities that can be controlled from Home Assistant or other API
clients. Each instance can be configured as either a transmitter or receiver by specifying the appropriate hardware
component ID.

### Transmitting Signals

The infrared proxy supports two methods for transmitting signals:

#### 1. Protocol-Based Encoding (High-Level)

This method uses simple JSON protocol specifications to transmit signals. The component automatically handles
protocol-specific encoding details.

**Supported Protocols** (37 total): abbwelcome, aeha, beo4, byronsx, canalsat, canalsatld, coolix, dish, dooya,
drayton, dyson, gobox, haier, jvc, keeloq, lg, magiquest, midea, mirage, nec, nexa, panasonic, pioneer, pronto,
raw, rc5, rc6, rc_switch_raw, rc_switch_type_a, rc_switch_type_b, rc_switch_type_c, rc_switch_type_d, roomba,
samsung, samsung36, sony, symphony, toshiba_ac, toto

**Example JSON format:**
```json
{"protocol": "nec", "address": 255, "command": 187}
```

Home Assistant integrations may use this method to send commands via the infrared proxy.

#### 2. Pulse Width Encoding (Low-Level)

This method provides full control over timing parameters, typically to support protocols not (yet) implemented by
ESPHome. You can specify:

- Header/footer timings
- Bit encoding (mark/space durations for logical 1 and 0)
- Repeat handling with configurable idle time
- MSB/LSB bit ordering
- Carrier frequency

### Receiving Signals

When configured with a `remote_receiver_id`, the infrared proxy captures raw IR/RF timings and sends them to API
clients for decoding or analysis. This enables:

- Learning IR/RF commands from existing remotes
- Analyzing unknown protocols
- Creating universal remote controls

Reception is non-blocking, allowing other listeners to also process signals simultaneously.

## Hardware Support

The infrared proxy can work with both infrared and RF hardware:

- **Infrared**: Do not set `frequency` (or set `frequency: 0`, which is the default) and use standard IR LEDs/receivers
- **RF**: Set `frequency` to match your RF hardware (for example, `315000000` for 315 MHz or `433920000` for 433.92 MHz)

You can create separate instances for different purposes:
- Transmit-only (specify only `remote_transmitter_id`)
- Receive-only (specify only `remote_receiver_id`)
- Multiple instances of any of the above for different hardware or frequencies

## See Also

- [Remote Transmitter](/components/remote_transmitter)
- [Remote Receiver](/components/remote_receiver)
- {{< apiref "infrared_proxy/infrared_proxy.h" "infrared_proxy/infrared_proxy.h" >}}
- [Home Assistant Remote Integration](https://www.home-assistant.io/integrations/remote/)
