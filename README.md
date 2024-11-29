

from homeassistant.config_entries import ConfigEntry
from homeassistant.const import Platform
from homeassistant.core import HomeAssistant
from homeassistant.helpers.device import (
    async_remove_stale_devices_links_keep_entity_device,
)

CONF_HEATER = "heater"
DOMAIN = "generic_thermostat"
PLATFORMS = [Platform.CLIMATE]
from .const import CONF_HEATER, PLATFORMS


async def async_setup_entry(hass: HomeAssistant, entry: ConfigEntry) -> bool:
‎homeassistant/components/generic_thermostat/climate.py
+ 16
- 27
Orijinal dosya satır numarası	Farklı satır numarası	Fark satırı değişikliği
@@ -14,13 +14,7 @@
from homeassistant.components.climate import (
    ATTR_PRESET_MODE,
    PLATFORM_SCHEMA as CLIMATE_PLATFORM_SCHEMA,
    PRESET_ACTIVITY,
    PRESET_AWAY,
    PRESET_COMFORT,
    PRESET_ECO,
    PRESET_HOME,
    PRESET_NONE,
    PRESET_SLEEP,
    ClimateEntity,
    ClimateEntityFeature,
    HVACAction,
@@ -64,36 +58,31 @@
from homeassistant.helpers.restore_state import RestoreEntity
from homeassistant.helpers.typing import ConfigType, DiscoveryInfoType, VolDictType

from . import CONF_HEATER, DOMAIN, PLATFORMS
from .const import (
    CONF_AC_MODE,
    CONF_COLD_TOLERANCE,
    CONF_HEATER,
    CONF_HOT_TOLERANCE,
    CONF_MIN_DUR,
    CONF_PRESETS,
    CONF_SENSOR,
    DEFAULT_TOLERANCE,
    DOMAIN,
    PLATFORMS,
)

_LOGGER = logging.getLogger(__name__)

DEFAULT_TOLERANCE = 0.3
DEFAULT_NAME = "Generic Thermostat"
CONF_SENSOR = "target_sensor"
CONF_INITIAL_HVAC_MODE = "initial_hvac_mode"
CONF_KEEP_ALIVE = "keep_alive"
CONF_MIN_TEMP = "min_temp"
CONF_MAX_TEMP = "max_temp"
CONF_TARGET_TEMP = "target_temp"
CONF_AC_MODE = "ac_mode"
CONF_MIN_DUR = "min_cycle_duration"
CONF_COLD_TOLERANCE = "cold_tolerance"
CONF_HOT_TOLERANCE = "hot_tolerance"
CONF_KEEP_ALIVE = "keep_alive"
CONF_INITIAL_HVAC_MODE = "initial_hvac_mode"
CONF_PRECISION = "precision"
CONF_TARGET_TEMP = "target_temp"
CONF_TEMP_STEP = "target_temp_step"

CONF_PRESETS = {
    p: f"{p}_temp"
    for p in (
        PRESET_AWAY,
        PRESET_COMFORT,
        PRESET_ECO,
        PRESET_HOME,
        PRESET_SLEEP,
        PRESET_ACTIVITY,
    )
}

PRESETS_SCHEMA: VolDictType = {
    vol.Optional(v): vol.Coerce(float) for v in CONF_PRESETS.values()
‎homeassistant/components/generic_thermostat/config_flow.py
+ 1
- 1
Orijinal dosya satır numarası	Farklı satır numarası	Fark satırı değişikliği
@@ -16,7 +16,7 @@
    SchemaFlowFormStep,
)

from .climate import (
from .const import (
    CONF_AC_MODE,
    CONF_COLD_TOLERANCE,
    CONF_HEATER,
‎homeassistant/components/generic_thermostat/const.py
+ 34
Orijinal dosya satır numarası	Farklı satır numarası	Fark satırı değişikliği
@@ -0,0 +1,34 @@
"""Constants for the Generic Thermostat helper."""
from homeassistant.components.climate import (
    PRESET_ACTIVITY,
    PRESET_AWAY,
    PRESET_COMFORT,
    PRESET_ECO,
    PRESET_HOME,
    PRESET_SLEEP,
)
from homeassistant.const import Platform
DOMAIN = "generic_thermostat"
PLATFORMS = [Platform.CLIMATE]
CONF_AC_MODE = "ac_mode"
CONF_COLD_TOLERANCE = "cold_tolerance"
CONF_HEATER = "heater"
CONF_HOT_TOLERANCE = "hot_tolerance"
CONF_MIN_DUR = "min_cycle_duration"
CONF_PRESETS = {
    p: f"{p}_temp"
    for p in (
        PRESET_AWAY,
        PRESET_COMFORT,
        PRESET_ECO,
        PRESET_HOME,
        PRESET_SLEEP,
        PRESET_ACTIVITY,
    )
}
CONF_SENSOR = "target_sensor"
DEFAULT_TOLERANCE = 0.3
‎tests/components/generic_thermostat/test_climate.py
+ 1
- 1
Orijinal dosya satır numarası	Farklı satır numarası	Fark satırı değişikliği
@@ -21,7 +21,7 @@
    PRESET_SLEEP,
    HVACMode,
)
from homeassistant.components.generic_thermostat import (
from homeassistant.components.generic_thermostat.const import (
    DOMAIN as GENERIC_THERMOSTAT_DOMAIN,
)
from homeassistant.const import (
‎tests/components/generic_thermostat/test_config_flow.py
+ 2
- 2
Orijinal dosya satır numarası	Farklı satır numarası	Fark satırı değişikliği
@@ -6,12 +6,11 @@
from syrupy.filters import props

from homeassistant.components.climate import PRESET_AWAY
from homeassistant.components.generic_thermostat.climate import (
from homeassistant.components.generic_thermostat.const import (
    CONF_AC_MODE,
    CONF_COLD_TOLERANCE,
    CONF_HEATER,
    CONF_HOT_TOLERANCE,
    CONF_NAME,
    CONF_PRESETS,
    CONF_SENSOR,
    DOMAIN,
@@ -21,6 +20,7 @@
from homeassistant.const import (
    ATTR_DEVICE_CLASS,
    ATTR_UNIT_OF_MEASUREMENT,
    CONF_NAME,
    STATE_OFF,
    UnitOfTemperature,
)
‎tests/components/generic_thermostat/test_init.py
+ 1
- 1
Orijinal dosya satır numarası	Farklı satır numarası	Fark satırı değişikliği
@@ -2,97 +2,97 @@

from __future__ import annotations

from homeassistant.components.generic_thermostat import DOMAIN
from homeassistant.components.generic_thermostat.const import DOMAIN
from homeassistant.core import HomeAssistant
from homeassistant.helpers import device_registry as dr, entity_registry as er

from tests.common import MockConfigEntry


async def test_device_cleaning(
    hass: HomeAssistant,
    device_registry: dr.DeviceRegistry,
    entity_registry: er.EntityRegistry,
) -> None:
    """Test cleaning of devices linked to the helper config entry."""

    # Source entity device config entry
    source_config_entry = MockConfigEntry()
    source_config_entry.add_to_hass(hass)

    # Device entry of the source entity
    source_device1_entry = device_registry.async_get_or_create(
        config_entry_id=source_config_entry.entry_id,
        identifiers={("switch", "identifier_test1")},
        connections={("mac", "30:31:32:33:34:01")},
    )

    # Source entity registry
    source_entity = entity_registry.async_get_or_create(
        "switch",
        "test",
        "source",
        config_entry=source_config_entry,
        device_id=source_device1_entry.id,
    )
    await hass.async_block_till_done()
    assert entity_registry.async_get("switch.test_source") is not None

    # Configure the configuration entry for helper
    helper_config_entry = MockConfigEntry(
        data={},
        domain=DOMAIN,
        options={
            "name": "Test",
            "heater": "switch.test_source",
            "target_sensor": "sensor.temperature",
            "ac_mode": False,
            "cold_tolerance": 0.3,
            "hot_tolerance": 0.3,
        },
        title="Test",
    )
    helper_config_entry.add_to_hass(hass)
    assert await hass.config_entries.async_setup(helper_config_entry.entry_id)
    await hass.async_block_till_done()

    # Confirm the link between the source entity device and the helper entity
    helper_entity = entity_registry.async_get("climate.test")
    assert helper_entity is not None
    assert helper_entity.device_id == source_entity.device_id

    # Device entry incorrectly linked to config entry
    device_registry.async_get_or_create(
        config_entry_id=helper_config_entry.entry_id,
        identifiers={("sensor", "identifier_test2")},
        connections={("mac", "30:31:32:33:34:02")},
    )
    device_registry.async_get_or_create(
        config_entry_id=helper_config_entry.entry_id,
        identifiers={("sensor", "identifier_test3")},
        connections={("mac", "30:31:32:33:34:03")},
    )
    await hass.async_block_till_done()

    # Before reloading the config entry, 3 devices are expected to be linked
    devices_before_reload = device_registry.devices.get_devices_for_config_entry_id(
        helper_config_entry.entry_id
    )
    assert len(devices_before_reload) == 3

    # Config entry reload
    await hass.config_entries.async_reload(helper_config_entry.entry_id)
    await hass.async_block_till_done()

    # Confirm the link between the source entity device and the helper entity
    helper_entity = entity_registry.async_get("climate.test")
    assert helper_entity is not None
    assert helper_entity.device_id == source_entity.device_id

    # After reloading the config entry, only one linked device is expected
    devices_after_reload = device_registry.devices.get_devices_for_config_entry_id(
        helper_config_entry.entry_id
    )
    assert len(devices_after_reload) == 1

    assert devices_after_reload[0].id == source_device1_entry.id
