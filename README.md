# auditbeat

[![Build Status](https://cloud.drone.io/api/badges/rolehippie/auditbeat/status.svg)](https://cloud.drone.io/rolehippie/auditbeat)

Ansible role to configure auditbeat

## Table of content

* [Default Variables](#default-variables)
  * [auditbeat_service_enabled](#auditbeat_service_enabled)
  * [auditbeat_service_state](#auditbeat_service_state)
* [Dependencies](#dependencies)
* [License](#license)
* [Author](#author)

---

## Default Variables

### auditbeat_service_enabled

Switch the service into enabled state

#### Default value

```YAML
auditbeat_service_enabled: false
```

### auditbeat_service_state

State for the service definition

#### Default value

```YAML
auditbeat_service_state: stopped
```

## Dependencies

None.

## License

Apache-2.0

## Author

Thomas Boerger
