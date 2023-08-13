## Migrating OpenVAS Container and Data Volume

If you're looking to migrate an OpenVAS container and its associated data volume to a new host, here's a step-by-step guide based on my experience.

#### Creating Snapshot of Container's Filesystem
To capture the current state of your OpenVAS container, you can use the **docker export** command:

`docker export -o openvas_snapshot.tar openvas_on_azure`

#### Creating Snapshot of Data Volume
Your OpenVAS container has a volume attached to it. To back up this volume, you can follow these steps:

1. Create a "busybox" or "ubuntu" Docker container and mount the OpenVAS container's volume to it:

`docker run --rm --volumes-from openvas_on_azure -v /openvas/openvas_backup:/backup ubuntu tar cvf /backup/backup.tar /data`

#### Moving Snapshots to Destination Host
Use scp or winscp to transfer the two tar files (openvas_snapshot.tar and backup.tar) to your destination host.

#### Importing Docker Container from Snapshot
To import the OpenVAS container snapshot, use the docker import command:
`docker import openvas_snapshot.tar openvas_new_image_name`

#### Restoring Data Volume

Follow these steps to restore the data volume on the new host:

1. Move backup.tar to /data/openvas_backup location on the new host.

3. Create a new Docker container to hold the volume, and mount the data volume backup:
`docker run -v /data/openvas_data:/data --name dbstore2 ubuntu`

3. Extract the contents of backup.tar into the data volume:

`docker run --rm --volumes-from dbstore2 -v /data/openvas_backup:/backup ubuntu sh -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"`

#### Starting OpenVAS Container on New Host

Finally, you can start the OpenVAS container on your new host using the following command:
`docker run -d --name openvas_on_onprem -e PASSWORD=admin -e USERNAME=admin -e RELAYHOST=172.17.0.1 -e SMTPPORT=25 -e QUIET=false -e NEWDB=false -e SKIPSYNC=true -e RESTORE=false -e DEBUG=false -e HTTPS=false -p 8080:9392 -v /openvas/openvas_data:/data openvas_new_image_name`

Make sure to adjust the image name and any environment variables as needed.

**Note:** Maintain the same directory structure as on your old host to avoid complications during migration.

With these steps, you should be able to successfully migrate your OpenVAS container and data volume to your new host. Good luck!







