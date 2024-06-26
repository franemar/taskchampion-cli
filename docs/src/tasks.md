# Tasks

Tasks are stored internally as a key/value map with string keys and values.
All fields are optional: the `Create` operation creates an empty task.
Display layers should apply appropriate defaults where necessary.

## Atomicity

The synchronization process does not support read-modify-write operations.
For example, suppose tags are updated by reading a list of tags, adding a tag, and writing the result back.
This would be captured as an `Update` operation containing the amended list of tags.
Suppose two such `Update` operations are made in different replicas and must be reconciled:
 * `Update("d394be59-60e6-499e-b7e7-ca0142648409", "tags", "oldtag,newtag1", "2020-11-23T14:21:22Z")`
 * `Update("d394be59-60e6-499e-b7e7-ca0142648409", "tags", "oldtag,newtag2", "2020-11-23T15:08:57Z")`

The result of this reconciliation will be `oldtag,newtag2`, while the user almost certainly intended `oldtag,newtag1,newtag2`.

The key names given below avoid this issue, allowing user updates such as adding a tag or deleting a dependency to be represented in a single `Update` operation.

## Representations

Integers are stored in decimal notation.

Timestamps are stored as UNIX epoch timestamps, in the form of an integer.

## Keys

The following keys, and key formats, are defined:

* `status` - one of `P` for a pending task (the default), `C` for completed or `D` for deleted
* `description` - the one-line summary of the task
* `modified` - the time of the last modification of this task
* `start` - the most recent time at which this task was started (a task with no `start` key is not active)
* `tag_<tag>` - indicates this task has tag `<tag>` (value is an empty string)
* `wait` - indicates the time before which this task should be hidden, as it is not actionable

The following are not yet implemented:

* `dep.<uuid>` - indicates this task depends on `<uuid>` (value is an empty string)
* `annotation.<timestamp>` - value is an annotation created at the given time
