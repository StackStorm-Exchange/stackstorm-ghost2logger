# Change Log

## 1.1.2

- Python 3 fixups
- Add explicit support for Python 2 and 3

## 1.1.1

- Bug fixes to Ghost2logger-X Go binary: Gracefully handles regular expression faults without faulting, better logging for internal goings on
- Bug fixes to ghostloggersensor - casting string to int for port number and turned down logging verbosity
- Compatible with 2.5x of ST2 and retains backwards compatibility

## 1.1.0

- New changes made to Ghost2logger binary that allows a regular expression as a host entry as well as message.
- Config changes made, rendering this version not backwards compatible. Please generate a new configuration (or at least copy and paste across the items.)

## 1.0.2

- Added example config, ghost2logger.yaml.example
