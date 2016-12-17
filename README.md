#python-inotify

The library exposes the events using a generator, which blocks until events occur.
After each cycle (all notified events were processed, or no events were received), you'll get a *None*. You may use this as an opportunity to perform other tasks, if your application is being primarily driven by *inotify* events. By default, we'll only block for one-second on queries to the kernel. This may be set to something else by passing a seconds-value into the constructor as *block_duration_s*.

```python
import inotify.adapters

inot = inotify.adapters.Inotify()
inot.add_watch(b'/tmp')

for event in inot.event_gen():
	if event:
		print(event)
```

You may also choose to pass the list of directories to watch via the *paths* parameter of the constructor. This would work best in situations where your list of paths is static. Also, the remove_watch() call is included in the example, but is not strictly necessary. The *inotify* resources is cleaned-up, which would clean-up any *inotify*-internal watch resources as well.

Note that the directories to pass to the add_watch() and remove_watch() functions must be bytestring (in Python 3).  The same holds for the contents of the events that are returned.  It's up to the user to encode and decode any strings.

##Recursive watching

Since the kernel doesn't provide a recursive directory watching functionality it is implemented
by this library. To recursively watch a directory simply use the `InotifyTree` adapter.

```python
inot = inotify.adapters.InotifyTree('/tmp/watch_tree')
```

This will immediately recurse through the directory tree and add watches on all subdirectories. New directories will automatically have watches added for them and deleted directories will be cleaned-up.

Differences to the regular adapter:

- You can't remove a watch since watches are automatically managed.
- Even if you provide a very restrictive mask that doesn't allow for directory create/delete events, the *IN_ISDIR*, *IN_CREATE*, and *IN_DELETE* flags will still be added.

##Notes

- *epoll* is used to audit for *inotify* kernel events. This is the fastest file-descriptor "selecting" strategy.

- Due to the GIL locking considerations of Python (or any VM-based language), it is strongly recommended that, if you need to be performing other tasks *while* you're concurrently watching directories, you use *multiprocessing* to put the directory-watching in a process of its own and feed information back [via queue/pipe/etc..]. This is especially true whenever your application is blocking on kernel functionality. Python's VM will remain locked and all other threads in your application will cease to function until something raises an event in the directories that are being watched.
