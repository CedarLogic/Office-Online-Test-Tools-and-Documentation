
:orphan:

..  _coauth:

Co-authoring using Office Online
================================

Office Online supports multiple users editing a document at the same time. There are no unique WOPI host requirements
beyond those described in this documentation in order to support co-authoring. However, the guidelines around file IDs
and locks, as described in the :ref:`key concepts` section, are very important in order to enable co-authoring.


How co-authoring works in Office Online
---------------------------------------

When multiple users edit a single document using Office Online, the Office Online service manages the document
changes and any necessary merges internally.

When a user attempts to edit a document, Office Online takes a lock using the :ref:`Lock` operation and the access
token of the current user. When additional users then attempt to edit the same document, Office Online will verify
those users have permission to edit the document by calling :ref:`CheckFileInfo` using each user's access token. If
the CheckFileInfo call succeeds and the user has permissions to edit, they will join the editing session already in
progress.

As users make changes, Office Online will call :ref:`PutFile` periodically with the updated document contents. There
are three critical questions to consider with respect to co-authoring sessions:

#. **Auto-save frequency**: How frequently will the application call :ref:`PutFile`?
#. **PutFile access token**: Which user's access token will be used when :ref:`PutFile` is called?
#. **Permissions check frequency**: How often will the application verify that a user still has appropriate permissions
   to edit the document?

Question 3 is important because :ref:`PutFile` is called using a single user's access token, so a WOPI host will only
be able to check permissions of the user whose access token is used. Hosts rely on Office Online to verify users
still have edit permissions periodically.

The answers to these questions differ between the Office Online applications. The table below summarizes the
behavior for each application, and the following sections describe this behavior in more detail.

..  csv-table:: Summary of co-authoring behavior for Office Online applications
    :file: ../tables/coauth_behavior.csv
    :header-rows: 1


Word Online co-authoring behavior
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Auto-save frequency**: Every 30 seconds if document is updated. In other words, if users are actively editing a
document, :ref:`PutFile` will be called every 30 seconds. However, if users stop editing for a period of time, Word
Online will not call :ref:`PutFile` until document changes are made again.

**PutFile access token**: For each auto-save interval, Word Online will use the access token of the user who made the
most recent change to the document. In other words, if User A and B both make changes to the document within the
same auto-save interval, but User B made the last change, Word Online will use User B's access token when calling
:ref:`PutFile`. The file will have both users' changes, but the PutFile request will use User B's access token.

If, on the other hand, User A made a change in one auto-save interval, and User B made a change in another auto-save
interval, then Word Online will make two PutFile requests, each using the access token of the user who made the change.

**Permissions check frequency**: Word Online will verify that a user has permissions by calling CheckFileInfo at
least every 5 minutes while the user is in an active session.


Excel Online co-authoring behavior
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Auto-save frequency**: Every 2 minutes.

**PutFile access token**: Excel Online will always use the access token of the user who joined the editing session
first. This user is called the *principal user*. If the principal user leaves the session, then the next user who
joined the session becomes the principal user. In other words, if User A starts editing and then User B joins the
session, User A is the principal user, and Excel Online will use User A's access token when calling :ref:`PutFile`.
The file will have both users' changes, but the PutFile request will use User A's access token.

If User A leaves the session, then User B becomes the principal user, and User B's access token will be used when
calling PutFile.

**Permissions check frequency**: Excel Online will verify that a user has permissions by calling :ref:`RefreshLock` at
least every 15 minutes while the user is in an active session.


PowerPoint Online co-authoring behavior
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Auto-save frequency**: TODO

**PutFile access token**: TODO

**Permissions check frequency**: TODO


Scenarios
---------

The following scenarios illustrate the behavior WOPI hosts can expect for each Office Online application when
users are co-authoring.

All scenarios described here assume the following baseline flow.

..  note::

    The pattern of WOPI calls described below is not meant to be absolutely accurate. Office Online may make
    additional WOPI calls beyond those described below. These scenarios are meant only to illustrate the key behavioral
    aspects of the Office Online applications; they are not an absolute transcript of WOPI traffic between Office
    Online and a WOPI host.

Scenario baseline
~~~~~~~~~~~~~~~~~

#. User A begins editing a document.
#. Office Online calls :ref:`CheckFileInfo` using User A's access token to verify the user has edit permissions.
#. Office Online calls :ref:`Lock` using User A's access token.
#. User B tries to edit the same document.
#. Office Online calls :ref:`CheckFileInfo` using User B's access token to verify the user has edit permissions.

Key points
^^^^^^^^^^

* Office Online will always verify each user has appropriate edit permissions to the document by calling
  CheckFileInfo before allowing them to join the edit session.
* Lock will always be called using the access token of the first user to start editing the document.

Scenario 1
~~~~~~~~~~

#. User A continues editing the document.
#. User B makes no changes.

..  csv-table:: Co-authoring scenario 1
    :file: ../tables/coauth_scenario_1.csv
    :header-rows: 1


Scenario 2
~~~~~~~~~~

#. User A continues editing the document.
#. User B also edits the document.

..  csv-table:: Co-authoring scenario 2
    :file: ../tables/coauth_scenario_2.csv
    :header-rows: 1


Scenario 3
~~~~~~~~~~

#. User A leaves the editing session by closing the Office Online application or navigating away.
#. User B continues editing the document.
#. User C tries to edit the same document.
#. Office Online calls :ref:`CheckFileInfo` using User C's access token to verify the user has edit permissions.

..  csv-table:: Co-authoring scenario 3
    :file: ../tables/coauth_scenario_3.csv
    :header-rows: 1


Scenario 4
~~~~~~~~~~

#. User A continues editing the document.
#. User B is in the session but is not editing the document.
#. While the editing session is in progress, User B's permissions to edit the document are removed.

..  csv-table:: Co-authoring scenario 4
    :file: ../tables/coauth_scenario_4.csv
    :header-rows: 1


Scenario 5
~~~~~~~~~~

#. User A continues editing the document.
#. User B also continues editing the document.
#. While the editing session is in progress, User B's permissions to edit the document are removed.

..  csv-table:: Co-authoring scenario 5
    :file: ../tables/coauth_scenario_5.csv
    :header-rows: 1


Scenario 6
~~~~~~~~~~

#. User A continues editing the document.
#. User B also continues editing the document.
#. While the editing session is in progress, User A's permissions to edit the document are removed.

..  csv-table:: Co-authoring scenario 6
    :file: ../tables/coauth_scenario_6.csv
    :header-rows: 1
