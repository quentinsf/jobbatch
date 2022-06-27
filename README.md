# Run batches of commands

**A design plan: not actually written yet!**

A simple tool for queuing up batches of commands and running N of them at a time.

Our initial use case was scheduling software updates on a number of remote devices without getting all the support queries back in the same day! So we wanted to update 100 devices each night.  But you might want to send 10 emails per minute, transcode 5 videos per hour, or whatever.  You can set upa  cron job or similar to run `qbatch`, and it will split the task into these chunks for you. 

## Here's how it works.

The flow is managed using the filesystem.  You have a set of subdirectories:

* queue
* batch
* successes
* failures
* output

You can put all the commands you want to run as individual files into the 'queue' directory.  (These are probably created by some external system.)  If you have them as lines in a file rather than a bunch of individual files, you can use `qbatch split` to convert it into one file per line.
Often, each file is a shell script, but it doesn't have to be (see below).

Each time you want to run the next chunk of jobs, you do `qbatch select` to take a selection of the files in the `queue` directory and move them into the `batch` directory.  You can then take a look to check that things look right, and if you don't like what you see, you can just move all the files back into the queue directory.  If the `queue` directory is empty, this command will do nothing.

Then you run `qbatch run`, which will execute those the jobs in the `batch` directory one at a time.  After execution, each file is moved to either `successes` or `failures` depending on its exit code.   If the `batch` directory is empty, this command will do nothing.

Any standard output or error output from a job will be put in a file with a '.stdout' or '.stderr'  extension within the `output` directory.

## Not just for scripts

Typically, each job file will be a shell script or program to be executed.  If the file has the 'executable' bit set, it will be executed directly.  If it doesn't, it will be passed as the argument to a processor such as `/bin/bash`, but you can change this with a command-line option.

This means, therefore, that the jobs do not have to be scripts!   Suppose you had a script for emailing thumbnails of your photographs to your friends, you could put the photographs into the 'queue' directory and specify your script as the processor, and then send them eight thumbnails per day.
