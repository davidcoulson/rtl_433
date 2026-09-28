# Managing Devices That Change IDs

Many battery-powered sensors pick a new random transmitter ID every time their
batteries are changed. rtl_433 identifies a device by that ID, so the sensor
comes back as a brand-new device with new entities and no history, while the
original stops updating and eventually goes unavailable.

rtl_433 can't tell the difference between a new device and one that changed its
ID. To re-link the device, go to the **Add or replace device** page.

Find the new device's card and click **Replace**, then pick the device it
replaces. Devices of the same model are listed first, since a battery swap does
not change the model. The button only appears once there is at least one added
device the candidate could stand in for.

The device is updated to take over the new ID. Entity IDs **do not change** even
if they contain the old ID. Automations, history, dashboards, and so on will all
keep working as they did before.

To confirm you are picking the right device, check the **Serial number** on the
device info card: it is the ID rtl_433 decoded for that device, plus its channel
and subtype when it has them. Unlike the device name, the serial number is not
affected by renaming the device, so it always shows the transmitter the device
is currently tracking — the old ID before a replace, the new one after.

## Noticing an ID change automatically

The integration also watches for this itself. When a device you added has been
silent for 10 minutes and a new, unadded device appears that looks like its
replacement, a **repair** shows up under **Settings → System → Repairs**. It names
the device, its old ID and the new one. Confirming the repair does exactly what
**Replace** does.

A candidate only counts as a replacement when every clue agrees:

- It is the **same model**, on the **same channel** (and subtype), with only the
  ID different.
- It has been **heard at least twice**, so a single bad decode never qualifies.
- It **first appeared after the old device went quiet**. A neighbour's sensor of
  the same model transmits while yours still does, which rules it out.
- When both report a temperature, the new reading is **within 10 °C** of the old
  device's last one.
- The match is **unambiguous**. If two candidates could replace one device, or
  one candidate could replace two quiet devices, nothing is suggested and you
  choose on the **Add or replace device** page as before.

The repair goes away by itself if the old device starts transmitting again.

### Following a device automatically

For a sensor whose batteries you change often, you can skip the repair. Open the
device's settings (the **rtl_433** panel, or the integration's **Configure** →
**Device settings**) and turn on **Follow ID changes automatically**. When a match
is found for that device, it is replaced straight away. The setting moves with the
device, so the next battery change is followed too.

Every replace made this way, whether automatic or confirmed from a repair, fires an
`rtl_433_device_id_changed` event. You can use it to be told when it happens:

```yaml
triggers:
  - trigger: event
    event_type: rtl_433_device_id_changed
actions:
  - action: persistent_notification.create
    data:
      title: "{{ trigger.event.data.name }} changed its ID"
      message: >-
        {{ trigger.event.data.old_key }} → {{ trigger.event.data.new_key }}
        ({{ 'automatically' if trigger.event.data.automatic else 'confirmed' }})
```

The event data carries `entry_id`, `device_id`, `name`, `old_key`, `new_key`, and
`automatic`.
