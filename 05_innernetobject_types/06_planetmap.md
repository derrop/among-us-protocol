# `PlanetMap`

The `PlanetMap` object (which is a variant of the [`ShipStatus`](00_shipstatus.md) object) is spawned at the start of a game on *Polus* and despawned at the end of the game. It holds the map for *Polus* and is responsible for controlling the interactive systems throughout the map.

##### All `SystemType`s for Polus

The [`SystemType`s](../01_packet_structure/06_enums.md#systemtype) in the table below are used on *Polus* and should be handled when being spawned, serialized, and deserialized.

| Type | Implementation | On Spawn | On Data |
| --- | --- | --- | --- |
| `ELECTRICAL` | [`SwitchSystem`](../06_the_systemtype_implementations/02_switchsystem.md) | &#x2714; | &#x2714; |
| `MEDBAY` | [`MedScanSystem`](../06_the_systemtype_implementations/04_medscansystem.md) | &#x2714; | &#x2714; |
| `SECURITY` | [`SecurityCameraSystem`](../06_the_systemtype_implementations/05_securitycamerasystem.md) | &#x2714; | &#x2714; |
| `COMMUNICATIONS` | [`HudOverrideSystem`](../06_the_systemtype_implementations/06_hudoverridesystem.md) | &#x2714; | &#x2714; |
| `DOORS` | [`DoorsSystem`](../06_the_systemtype_implementations/11_doorssystem.md) | &#x2714; | &#x2714; |
| `SABOTAGE` | [`SabotageSystem`](../06_the_systemtype_implementations/08_sabotagesystem.md) | &#x2714; | &#x2714; |
| `DECONTAMINATION` | [`DeconSystem`](../06_the_systemtype_implementations/10_deconsystem.md) | &#x2716; | &#x2714; |
| `LABORATORY` | [`ReactorSystem`](../06_the_systemtype_implementations/01_reactorsystem.md) | &#x2714; | &#x2714; |
| `DECONTAMINATION_2` | [`DeconSystem`](../06_the_systemtype_implementations/10_deconsystem.md) | &#x2716; | &#x2714; |

##### Serialize

> **Note**: Because `PlanetMap` is an alias for [`ShipStatus`](00_shipstatus.md) specific to *Polus*, the `serialize` method is identical to that of `ShipStatus` except for the system types.

When the `ShipStatus` is being spawned, the game loops over all [`SystemType`s](../01_packet_structure/06_enums.md#systemtype) in order and checks if the type is part of *Polus*. For each type that is a part of *Polus*, the Hazel message writer containing the [component data](../03_gamedata_and_gamedatato_message_types/04_spawn.md#the-component-structure) is passed in to the type's `serialize` method.

When sending data (via [`0x01` Data](../03_gamedata_and_gamedatato_message_types/01_data.md)), the game first writes the information in the table below.

| Type | Name | Description |
| --- | --- | --- |
| `packed uint32` | Systems Mask | A bitfield mask used to tell the game which [`SystemType`s](../01_packet_structure/06_enums.md#systemtype) are receiving data |

After writing the mask, the game loops over all [`SystemType`s](../01_packet_structure/06_enums.md#systemtype) in order and checks if the type's ID is set on the mask. For each type that does have its ID set on the mask, the Hazel message writer containing the update data is passed in to the type's `serialize` method. Refer to the pseudocode below for an example.

```java
writer.writePackedUInt32(shipStatusNetId);
writer.startMessage(0);

// Loop through all SystemTypes
for (SystemType system : SystemType.values()) {
    // If the ShipStatus is being spawned
    if (isSpawning) {
        // If Polus has this SystemType...
        if (polusSystems.containsKey(system)) {
            // ...then we should write (serialize) the data from the system
            polusSystems.get(system).serialize(writer);
        }
    // If the ShipStatus is receiving data updates
    } else {
        writer.writePackedUInt32(systemsMask);

        // If this system is set on the systemsMask...
        if ((systemsMask & (1 << system.id())) != 0) {
            // ...then we should write (serialize) the data from the system
            polusSystems.get(system).serialize(writer);
        }
    }
}

writer.endMessage();
```

##### Deserialize

> **Note**: Because `PlanetMap` is an alias for [`ShipStatus`](00_shipstatus.md) specific to *Polus*, the `deserialize` method is identical to that of `ShipStatus` except for the system types.

When the `ShipStatus` has been spawned, the game loops over all [`SystemType`s](../01_packet_structure/06_enums.md#systemtype) in order and checks if the type is part of *Polus*. For each type that is a part of *Polus*, the Hazel message containing the [component data](../03_gamedata_and_gamedatato_message_types/04_spawn.md#the-component-structure) is passed in to the type's `deserialize` method.

When receiving data (via [`0x01` Data](../03_gamedata_and_gamedatato_message_types/01_data.md)), the game first reads the information in the table below.

| Type | Name | Description |
| --- | --- | --- |
| `packed uint32` | Systems Mask | A bitfield mask used to tell the game which [`SystemType`s](../01_packet_structure/06_enums.md#systemtype) are receiving data |

After reading the mask, the game loops over all [`SystemType`s](../01_packet_structure/06_enums.md#systemtype) in order and checks if the type's ID is set on the mask. For each type that does have its ID set on the mask, the Hazel message containing the update data is passed in to the type's `deserialize` method. Refer to the pseudocode below for an example.

```java
long shipStatusNetId = reader.readPackedUInt32();
MessageReader shipStatus = reader.readMessage();

// Loop through all SystemTypes
for (SystemType system : SystemType.values()) {
    // If the ShipStatus is being spawned
    if (isSpawning) {
        // If Polus has this SystemType...
        if (polusSystems.containsKey(system)) {
            // ...then we should read (deserialize) the data from the component message
            polusSystems.get(system).deserialize(shipStatus);
        }
    // If the ShipStatus is receiving data updates
    } else {
        long systemsMask = shipStatus.readPackedUInt32();

        // If this system is set on the systemsMask...
        if ((systemsMask & (1 << system.id())) != 0) {
            // ...then we should read (deserialize) the data from the component message
            polusSystems.get(system).deserialize(shipStatus);
        }
    }
}
```

---

> Previous section: [`Headquarters`](05_headquarters.md)<br>
> Next section: [`AprilShipStatus`](07_aprilshipstatus.md)
