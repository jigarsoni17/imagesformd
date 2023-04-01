# **Title**

```
#!/bin/bash
set -eux 

FROMDIR="/home/ubuntu/airflow/logs/scheduler/"
S3URI="s3://airflow-poc-dev-s3/archived-logs/"
DIR_LIST=$(ls -dt ${FROMDIR}/*/ | tail -n +16 | sed "s%${FROMDIR}%%g" | tr -d '/') 
echo "started"
echo "finished"

for DIR in $DIR_LIST; do
    tar cvf ${DIR}.tar ${FROMDIR}${DIR};
    aws s3 cp ${DIR}.tar ${S3URI};
    rm -rf ${DIR}.tar; 
    rm -rf ${FROMDIR}${DIR};
done
```
&nbsp;

## let me explain the script to you:

#### `#!/bin/bash`

- The first line "#!/bin/bash" specifies that this is a bash script.

&nbsp;


#### `set -eux` 

- The next line "set -eux" sets some options for the script. "-e" option causes the script to exit immediately if any command fails. "-u" option causes the script to exit if it tries to use an undefined variable. "-x" option prints each command as it is executed.

&nbsp;

#### `FROMDIR="/home/ubuntu/airflow/logs/scheduler/"`

- The next line sets the value of the "FROMDIR" variable to "/home/ubuntu/airflow/logs/scheduler/" which is the directory containing the logs that need to be archived.

&nbsp;

#### `S3URI="s3://airflow-poc-dev-s3/archived-logs/"`

- The "S3URI" variable is set to "s3://airflow-poc-dev-s3/archived-logs/" which is the S3 bucket and prefix where the logs will be archived.

&nbsp;

#### `DIR_LIST=$(ls -dt ${FROMDIR}/*/ | tail -n +16 | sed "s%${FROMDIR}%%g" | tr -d '/')`

- The "DIR_LIST" variable is set to a list of directories that need to be archived. The "ls -dt" command lists all directories in the "FROMDIR" directory sorted by modification time (newest first). The "tail -n +16" command selects all directories except the 15 most recently modified (i.e. it keeps only the oldest directories). The "sed" command removes the "FROMDIR" prefix from each directory name. Finally, the "tr" command removes the trailing forward slashes from each directory name.

&nbsp;

#### `echo "started" & echo "finished"`

- The script then prints "started" and "finished" to indicate the beginning and end of the archiving process.

&nbsp;

#### `for DIR in $DIR_LIST; do`

- The for loop iterates over each item in the list of directories specified by the $DIR_LIST variable.

&nbsp;

#### `tar cvf ${DIR}.tar ${FROMDIR}${DIR};`

- For each directory, it creates a tar archive file of the directory using the "tar" command with the "cvf" options.The archive file is named after the directory with a ".tar" extension.

&nbsp;

#### `aws s3 cp ${DIR}.tar ${S3URI};`

- Next, it copies the archive file to the S3 bucket using the "aws s3 cp" command.

&nbsp;


#### ` rm -rf ${DIR}.tar; & rm -rf ${FROMDIR}${DIR};`

- After copying the archive file to S3, it deletes the archive file and the original directory using the "rm" command.The "-rf" option is used to recursively delete directories and files without prompting for confirmation.


This script is designed to archive logs older than a certain time period to an S3 bucket to free up space on the local filesystem.

&nbsp;

---
# **logrotate in linux**

Logrotate is a utility in Linux that manages the automatic rotation, compression, and deletion of log files. Its main purpose is to help manage disk space by compressing and deleting old log files, while preserving a certain amount of history for future troubleshooting and analysis.

```
/home/ubuntu/airflow/logs/webserver _access.log {
    daily
    rotate 31
    compress
    delaycompress
    copytruncate 
    lastaction
            mv /home/ubuntu/airflow/logs/webserver _access.log*.gz /home/ubuntu/airflow/logs/logrotate
            aws s3 sync /home/ubuntu/airflow/logs/logrotate /path/to/s3/bucket/folder
    endscript
 }
```


This file is a configuration file for logrotate that specifies how to rotate and manage the "/home/ubuntu/airflow/logs/webserver _access.log" file.

Here is an explanation of each of the directives in the configuration file:

&nbsp;

#### `/home/ubuntu/airflow/logs/webserver _access.log {`

- This line specifies the log file that is being managed by logrotate. In this case, it is "/home/ubuntu/airflow/logs/webserver _access.log".

&nbsp;

#### `daily`

- This directive tells logrotate to rotate the log file once a day.

&nbsp;

#### `rotate 31`

- This directive tells logrotate to keep 31 rotated logs before deleting the oldest log file. This means that the log files will be kept for 31 days.

&nbsp;

#### `compress`

- This directive tells logrotate to compress the rotated log files using gzip.

&nbsp;

#### `delaycompress`

- This directive tells logrotate to delay the compression of the rotated log file until the next rotation cycle. This is useful because it allows applications that may still be writing to the log file to complete their write operations before the file is compressed

&nbsp;

#### `copytruncate`

- This directive tells logrotate to create a copy of the log file before it is rotated, and then truncate the original log file so that it can continue to be written to by the application that is logging to it. This is necessary because if the original file is simply renamed or deleted, the application may continue writing to the old file descriptor, rather than the new file.

&nbsp;

```
lastaction
    mv /home/ubuntu/airflow/logs/webserver _access.log*.gz /home/ubuntu/airflow/logs/logrotate
    aws s3 sync /home/ubuntu/airflow/logs/logrotate /path/to/s3/bucket/folder
endscript
```

- This block of code specifies a shell script that will be executed after the log file has been rotated. In this case, it moves all compressed log files to the "/home/ubuntu/airflow/logs/logrotate" directory and then synchronizes them to an S3 bucket folder using the "aws s3 sync" command. This allows the log files to be backed up to S3 for long-term storage.

&nbsp;

 This configuration file for logrotate is used to manage the "/home/ubuntu/airflow/logs/webserver _access.log" file, rotating it once a day, compressing old log files, and backing up the compressed files to an S3 bucket.

