# Add Quick Exceeding Quota Check to Lustre
An informal and incomplete description and specification for a feature to be added to lustre.

### Terminology
I frequently use the term "user", as in a "lustre user". This can often be interpreted as 
"user, or group, or project", because these are the things for which lustre cas set quotas.

## Basic Description
A new feature for lustre that allows for checking the status of the `edquot` flag for a user, group, or project. This flag 
determines if a quota is exceeded and additional writing to the file system is allowed. Currently, there is a way to get 
this value indirectly, but it involves either attempting to write a file and failing, or qureyinig the system and inferring 
it from more detailed quota information.

## Purpose
Programs running lustre may want to check whether or not they are able to write additional files. There are several reasons
that an application could attempt to write and fail, but the cause that this feature addresses is that of lustre quotas.
Lustre quotas can put limits on how much data or how many files a user, group, or project can create. If any attempt to write 
to a lustre file system is then made, the write will fail and the error code `EDQUOT` (exceeding quota) will be returned.
Depending on how the application is designed, it may be easier to check if a write will be permitted instead of attempting to write
and failing, and then having to recover.

Another case for checking the quota status of users, groups, and projects is for the administrators of the lustre file system.
Admins can check the status of users and send them notifications that they are exceeding a limit, or are near a limit.

In both cases, a mechanism for checking quota status already exists. The `lfs` utility has a `quota` option that can be used
to get quota information for a user, group, or project. However, in some situations this method gives more information, and 
consequently involves more work, than is neccesary. The existing method causes the `QMS` (Quota Master Server) to talk to all of the
`QSDs` (Quota Slave Devices). In cases where there only information needed is "exceeding quota or not", it's not necesary for the `QMS` 
to talk to the `QSDs`. The `QMS` keeps track of the quota status for each user, group, and project locally, and talks to the `QSDs`
to get more up to date and granular information.

The proposed feature that gets only the "exceeding quota or not" information can get the information faster and reduce network traffic
between the `QMS` and `QSDs` when more detailed information is not needed.

## User Experience
The users are as mentioned above: People who write and run applications that use lustre and administrators of a lustre file system.
Because this feature aims to reduce the time it takes for the user check if they are exceeding a quota, there needs to be a way to 
do this check that does not involve forking a process. The current method involves spawing an new `lfs` process. For adminitrators who
are getting the quota status of many users, groups, and projects at a time, it may be acceptable to spawn a new process, but that 
process will get the quota statuses for multiple entities.

Lustre has recently added a new feature that allows for having quotas for `OST pools`. `OST pools` are groups of OSTs from the same lustre
file system. If attempting to write to an OST that is part of one or more OST groups, a user should also be able to check their quota
status on any OST pool, and not just the global pool that consists of all OSTs.

### LFS Interface
The current way to query quota status uses the `lfs` command line tool. `lfs` already has many subcommands a flags, including a
`quota` subcommand that is used to get quota data. 
The plan is to add a flag to the `lfs quota` command. THe flag is `-e` meaning `edquot?`. 
The current usage of `lfs quota` is:
not defined for OST quota pools 




## Non-Goals
Don't create a new `ioctl` that returns the quota data for multiple lustre users.


