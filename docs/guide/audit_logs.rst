:orphan:

.. currentmodule:: discord

.. _guide_audit_logs:

Audit Logs
============

Audit logs log various administrative actions taken in a guild. Examples may include:

- A channel being created, updated or deleted
- A member being kicked, pruned, banned, unbanned, updated thier roles, or moved from a voice channel.
- An invite being created, updated or deleted

.. warning:: 

    There are a couple of limitations with audit logs that need to be understood:

    1. There is no guarantee on audit logs arriving when expected (if they arrive at all)
    2. No event is triggered when an audit log entry is created.
    3. Audit logs are limited to 45 days.
    4. No entry is created for message deletions if:

      - It is a bot account deleting a single message
      - It is the message author deleting their own message

Depending on the action taken, an entry in the audit log may contain additional info about that specific action.
Each entry is an instance of :class:`AuditLogEntry`, which contains some metadata like:

- Reason
- When it was created
- What category the action falls into
- A list of changes in this entry
- The state of the target before and after this action occurred
- Any extra data provided about the action, for example:

  - The id of a channel, member, message, or role involved in the action
  - The number of entities targeted by the action
  - A number representing the number of days a member needed to be inactive for to be pruned
  - The number of members removed by a prune action
  - A role name
  - A type representing what the changed entity is.

Getting Audit Logs
~~~~~~~~~~~~~~~~~~~~

Audit logs can be retrieved via :func:`Guild.audit_logs`, assuming you have the :attr:`~Permissions.view_audit_log` permission. 

Note that this function returns an :term:`asynchronous iterator` and to properly go through the audit logs, you will need to iterate over them.

Reacting to Audit Log entries being created
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Audit log entries being created fires the :func:`on_audit_log_entry_create` event.

You must have the :attr:`Permissions.view_audit_log` permission to receive this.

This event will also not fire if you don't have :attr:`Intents.moderation` enabled.

Examples
==========

A handful of examples follow that walk through some use cases for looking at audit logs.

Getting and displaying all audit log entries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python3

    async for entry in guild.audit_logs():
        print(f'{entry.user} did {entry.action} to {entry.target}')

Getting a specific number of entries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python3

    async for entry in guild.audit_logs(limit=50):
        print(f'{entry.user} did {entry.action} to {entry.target}')

Getting and displaying a certain type of action
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We may want to know how many times users have been kicked to look for certain users being kicked repeatedly instead of being banned:

.. code-block:: python3

    async for entry in guild.audit_logs(action=discord.AuditLogAction.kick):
        print(f'{entry.user} kicked {entry.target}')

Getting all entries made by a specific user
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Perhaps we wanted to check to see if a specific mod in our guild is possibly inactive. We could 
achieve this by finding out how many actions they have performed in the past 45 days, like so:

.. code-block:: python3

    entries = [entry async for entry in guild.audit_logs(user=message.author)]
    await channel.send(f'{message.author} has made {len(entries)} moderation actions in the last 90 days.')

Sorting entries from oldest to newest
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python3
    
    async for entry in guild.audit_logs(oldest_first=True):
        print(f'{entry.user} did {entry.action} to {entry.target}')

Filtering to a specific time range
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This could be helpful if you are trying to find actions that occurred in a specific time frame. For 
example, maybe we want to see how many messages were deleted in a specific timeframe:

.. code-block:: python3

    deletes = [entry async for entry in guild.audit_logs(before=message.id, after=othermessage.id, action=discord.AuditLogAction.message_delete)]
    await channel.send(f'{len(deletes)} messages deleted before {discord.utils.format_dt(message.created_at)}')

.. note:: 

    If choosing to filter the audit log to entries after a specific time, the logs will be sorted from oldest to newest.

Log actions when they are created
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This could be useful when you want to log actions that are occurring in real time.

.. code-block:: python3

    @bot.event
    async def on_audit_log_entry_create(entry: discord.AuditLogEntry):
        print(f'{entry.user} did {entry.action} to {entry.target}')

Further reading
==================

You can find more information on the audit logs in the documentation for :meth:`~Guild.audit_logs`, :class:`AuditLogEntry`, :class:`AuditLogAction`, :class:`AuditLogActionCategory`, :class:`AuditLogChanges`, and :class:`AuditLogDiff`.