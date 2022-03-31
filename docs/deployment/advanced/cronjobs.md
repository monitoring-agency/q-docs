# Cronjobs
There are certain cronjobs that should be executed regularly.

## Database cleanup
As there are some objects that don't get deleted but de-referenced, there is some cleanup necessary.

- `GenericKVP`: all unreferenced KVPs can be deleted
- `Labels`: all unreferenced labels can be deleted
- `ScheduledObject`: if their reference to `object_id` doesn't exist anymore, it can be deleted
- `OrderedListItems` they get unreferenced each time a new item must be inserted, instead of appended
