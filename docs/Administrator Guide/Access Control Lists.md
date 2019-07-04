# POSIX Access Control Lists

POSIX Access Control Lists (ACLs) allows you to assign different
permissions for different users or groups even though they do not
correspond to the original owner or the owning group.

For example: User john creates a file but does not want to allow anyone
to do anything with this file, except another user, antony (even though
there are other users that belong to the group john).

This means, in addition to the file owner, the file group, and others,
additional users and groups can be granted or denied access by using
POSIX ACLs.

## Activating POSIX ACLs Support

To use POSIX ACLs for a file or directory, the partition of the file or
directory must be mounted with POSIX ACLs support.

### Activating POSIX ACLs Support on Server

To mount the backend export directories for POSIX ACLs support, use the
following command:

`# mount -o acl`

For example:

`# mount -o acl /dev/sda1 /export1`

Alternatively, if the partition is listed in the /etc/fstab file, add
the following entry for the partition to include the POSIX ACLs option:

`LABEL=/work /export1 ext3 rw,acl 14`

### Activating POSIX ACLs Support on Client

To mount the glusterfs volumes for POSIX ACLs support, use the following
command:

`# mount –t glusterfs -o acl`

For example:

`# mount -t glusterfs -o acl 198.192.198.234:glustervolume /mnt/gluster`

## Retrieving POSIX ACLs

View the existing access ACLs of a file using the following command:

`# getfacl <file>`

For example, to view the existing POSIX ACLs for sample.jpg

        # getfacl /mnt/gluster/data/test-file

Output:

        # owner: antony
        # group: antony
        user::rw-
        group::rw-
        other::r--

View the ACLs of a directory using the following command:

`# getfacl <directory>`

For example, to view the existing ACLs for `data/`

        # getfacl /mnt/gluster/data/

Output:

        # owner: antony
        # group: antony
        user::rw-
        user:john:r--
        group::r--
        mask::r--
        other::r--
        default:user::rwx
        default:user:antony:rwx
        default:group::r-x
        default:mask::rwx
        default:other::r-x

## Type of POSIX ACLs

You can set two types of POSIX ACLs: access ACLs and default ACLs.
You can use access ACLs to grant permission for a specific file or
directory. You can use default ACLs only on a directory, and if a file
inside that directory does not have access ACLs, it inherits the
permissions of the default ACLs of the directory.

You can set ACLs for per user, per group, for users not in the user
group for the file, and via the effective rights mask.

## Modifying Access ACLs

You can modify access ACLs to grant permission for both files and
directories using the following command:

`# setfacl -m <permission> <file/directory>`

The ACL entry types are the POSIX ACLs representations of owner, group,
and other.

Permissions must be a combination of the characters `r` (read), `w`
(write), and `x` (execute). You must specify the ACL entry in the
following format and can specify multiple entry types separated by
commas.

  ACL Entry | Description
  --- | ---
  `u:uid:<permission>` | Sets the access ACLs for a user. You can specify user name or UID
  `g:gid:<permission>` | Sets the access ACLs for a group. You can specify group name or GID.
  `m:<permission>` | Sets the effective rights mask. The mask is the combination of all access permissions of the owning group and all of the user and group entries.
  `o:<permission>` | Sets the access ACLs for users other than the ones in the group for the file.

If a file or directory already has a POSIX ACLs, and the setfacl
command is used, the additional permissions are added to the existing
POSIX ACLs or the existing rule is modified.

For example, to give read and write permissions to user antony:

`# setfacl -m u:antony:rw /mnt/gluster/data/test-file`

## Modifying Default ACLs

You can only apply default ACLs to directories. They determine the
permissions of file system objects that inherits from its parent
directory when it is created. You can modify default ACLs for
directories using the following command:

`# setfacl -d -m <permission> <directory>`

For example, to set the default ACLs for the /data directory to read for
users not in the group assigned to the directory:

`# setfacl -d -m o:r /mnt/gluster/data`

> **Note**
>
> Access ACLs set for an individual file can override the default
> ACLs permissions.

### Effects of a Default ACLs

The following are the ways in which the permissions of a directory's
default ACLs are passed to the files and subdirectories in it:

-   A subdirectory inherits the default ACLs of the parent directory
    both as its default ACLs and as an access ACLs.
-   A file inherits the default ACLs as its access ACLs.

## Setting ACLs

You can also set ACLs, which, unlike modifying, removes all existing
ACLs and replaces them with the specified onces. This works for both
file and directories, and for both default and access ACLs. You can
set ACLs with the following command:

`# setfacl --set [-d] <permission> <file/directory>`

For example, to remove existing access ACLs and replace them with one
that allows the user antony read access:

`# setfacl --set u:antony /mnt/gluster/data`

## Removing POSIX ACLs

To remove all the permissions for a user, groups, or others, use the
following command:

`# setfacl -x <permission> <file/directory>`

Unlike modifying ACLs, the section with `r`, `w` or `x` must be
excluded.

For example, to remove all permissions from the user antony:

`# setfacl -x u:antony /mnt/gluster/data/test-file`

## Samba and ACLs

If you are using Samba to access GlusterFS FUSE mount, then POSIX ACLs
are enabled by default. Samba has been compiled with the
`--with-acl-support` option, so no special flags are required when
accessing or mounting a Samba share.

## NFS and ACLs

Currently GlusterFS supports POSIX ACL configuration through NFS mounts,
i.e. setfacl and getfacl commands work through NFS mounts.

## Further information

For more information on the `setfacl` command set, see the man page for
your platform of choice.
