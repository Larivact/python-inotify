#python-inotify

*inotify* functionality is available from the Linux kernel and allows you to register one or more directories for watching, and to simply block and wait for notification events. This is obviously far more efficient than polling one or more directories to determine if anything has changed. This is available in the Linux kernel as of version 2.6 .

We've designed this library to act as a generator. All you have to do is loop, and you'll see one event at a time and block in-between. After each cycle (all notified events were processed, or no events were received), you'll get a *None*. You may use this as an opportunity to perform other tasks, if your application is being primarily driven by *inotify* events. By default, we'll only block for one-second on queries to the kernel. This may be set to something else by passing a seconds-value into the constructor as *block_duration_s*.

    import inotify.adapters

    i = inotify.adapters.Inotify()

    i.add_watch(b'/tmp')

        for event in i.event_gen():
            if event:
                print(event)

You may also choose to pass the list of directories to watch via the *paths* parameter of the constructor. This would work best in situations where your list of paths is static. Also, the remove_watch() call is included in the example, but is not strictly necessary. The *inotify* resources is cleaned-up, which would clean-up any *inotify*-internal watch resources as well.

Note that the directories to pass to the add_watch() and remove_watch() functions must be bytestring (in Python 3).  The same holds for the contents of the events that are returned.  It's up to the user to encode and decode any strings.

##Recursive Watching

We also provide you with the ability to add a recursive watch on a path. It turns out that there's no low-cost way of doing this; That's the reason that this functionality isn't provided by the kernel. However, we recognize that this is, nonetheless, sometimes necessary.

Example::

    i = inotify.adapters.InotifyTree('/tmp/watch_tree')

    for event in i.event_gen():
        # Do stuff...

        pass

The only substantial difference is the type of object that was instantiated. Everything else is the same.

This will immediately recurse through the directory tree and add watches on all subdirectories. New directories will automatically have watches added for them and deleted directories will be cleaned-up.

The other differences from the standard functionality:

- You can't remove a watch since watches are automatically managed.
- Even if you provide a very restrictive mask that doesn't allow for directory create/delete events, the *IN_ISDIR*, *IN_CREATE*, and *IN_DELETE* flags will still be added.

##Notes

- *epoll* is used to audit for *inotify* kernel events. This is the fastest file-descriptor "selecting" strategy.

- Due to the GIL locking considerations of Python (or any VM-based language), it is strongly recommended that, if you need to be performing other tasks *while* you're concurrently watching directories, you use *multiprocessing* to put the directory-watching in a process of its own and feed information back [via queue/pipe/etc..]. This is especially true whenever your application is blocking on kernel functionality. Python's VM will remain locked and all other threads in your application will cease to function until something raises an event in the directories that are being watched.
