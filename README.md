# esphome-packages
Some useful packages for use in esphome

## battery_notes.yaml
This package is supposed to mimic the functionality offered by [andrew-codechimp's HA-Battery-Notes](https://github.com/andrew-codechimp/HA-Battery-Notes), ie. it adds sensors that show the battery type, quantity and to set timestamp when you exchange the battery.
Requires a time source to be configured and its id passed as one of package vars.
The timestamp is stored persistently.

Example usage:
```yaml
packages:
  battery_notes: !include
    file: battery_notes.yaml
    vars:
      battery_notes_time_id: homeassistant_time
      battery_notes_type: AAA
      battery_notes_count: 2
```

The above example will create for you 3 entites (visible to user) and a persistent global (battery_last_replaced_global).
Here are the ids of these entities:

* battery_last_replaced_sensor - a timestamp field that shows when the battery was last replaced
* battery_type_text - a text field showing battery type and quantity
* battery_replaced_button - a button for user to press to set the timestamp to current time.

## count_events.yaml
This package will help you keep track of the number of any events. It provides a sensor and a simple esphome script to be executed when an interesting even has occur.
Additionaly it creates a button that resets the events count.

Example usage:
```yaml
packages:
  event_counter: !include
    file: count_events.yaml
    vars:
      event_counter_name: count_triggers
      event_counter_friendly_name: "Total triggers"

...

sensor:
  ...
  on_turn_off:
    then:
        - script.execute:
            id: count_triggers
            by_value: 1

```

The above will create for you a sensor that shows the current count. The id of this sensor is ${event_counter_name}_sensor. The state of this sensor is stored persistently and it's restored on startup.


## button_enabling_switch.yaml
This package will create a template switch, which serves as a kind of "child lock". I add it in devices (plugs mainly) that have a physical button. The switch's state can control whether the button can actually switch the relay on/off (although this logic needs to be added in your main yaml). Since the plugs usually have also some led indicators, I use it to show the state of the switch - this is what the "light_id" is for (it is not created, it must be defined in your main yaml).

Example usage:
```
packages:
  button_enabler: !include
    file: ../shared_public/button_enabling_switch.yaml
    vars:
      switch_id: physical_button_enabled_switch
      switch_name: "Physical Button Enabled"
      light_id: button_active_led

light:
  - ...
    id: button_active_led


binary_sensor:
  - ...
    on_multi_click:
        - timing:
             - ON for at most 1s
             - OFF for at least 0.2s
          then:
            - lambda: |-
                if (id(physical_button_enabled_switch).state)
                {
                  id(switch_socket).toggle();
                }

```