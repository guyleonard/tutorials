# Sequenceserver
Sequenceserver is a graphical user interface to blast, available from [here](https://sequenceserver.com/) and published in MBE (2109) [here](https://academic.oup.com/mbe/article/36/12/2922/5549819). You can read more about it from those two sources, we'll get stuck right in here.

## Installation
For this tutorial we will install the 'pre' or pre-release version of Sequenceserver as it has a few extra features that will be useful. This will install it to your user account, however it might have been previously installed by your admin, please check - if so you can continue to the next section. To quickly check you can type:
```bash
sequenceserver --version
```

And if you receive the error "sequenceserver: command not found" then follow the instructions below, or if you see a version number you can skip to the next section.
```bash
gem install --user-install --pre sequenceserver
```

You will see some output, like this:
```bash
WARNING:  You don't have /home/username/.gem/ruby/2.X.X/bin in your PATH,
	  gem executables will not run.

------------------------------------------------------------------------
  Thank you for installing SequenceServer :)

  To launch SequenceServer execute 'sequenceserver' from command line.

    $ sequenceserver


  Visit http://sequenceserver.com for more.
------------------------------------------------------------------------

Successfully installed sequenceserver-2.0.0.rc4
Parsing documentation for sequenceserver-2.0.0.rc4
Installing ri documentation for sequenceserver-2.0.0.rc4
Done installing documentation for sequenceserver after 0 seconds
1 gem installed
```

Paying attention to the warning, take the path (the one on your terminal, not the one in this tutorial) to the bin folder and add it to your PATH.
```bash
export PATH=$PATH:/home/username/.gem/ruby/2.X.X/bin
```

### Dependencies
You will also need BLAST+ exectuables in your PATH. We won't go through installation of those here, but you can refer to instructions in the tutorial [here](https://github.com/guyleonard/tutorials/tree/main/blast) or install them via 'conda'. You can get to see if they are installed with "blastn --version". Sequenceserver will also offer to download them for you if it cannot find them in PATH.

## Running Sequenceserver
In this tutorial there is one 'fasta' file in the directory "saccharomycese_cerevisiae.fas", let's see what happens when we try and run sequenceserver in the current directory.
```bash
sequenceserver -d .
```

```bash
[2020-06-19 14:55:14] INFO  Reading configuration file: /home/user/.sequenceserver.conf.

Could not find BLAST+ databases in: /home/user/tutorials/sequenceserver.

Search for FASTA files (.fa, .fasta, .fna) in '/home/user/tutorials/sequenceserver' and try
creating BLAST+ databases? [y/n] (Default: y).

>>
```

As you can see, sequenceserver detects that there are no blast databases available, however it is capable of creating one for you from files it can detect with the right extensions (e.g. .fa, .fasta, .fna). Let's say 'yes'.
```bash
Searching ...

FASTA file: /home/cs02gl/Desktop/git/tutorials/sequenceserver/saccharomyces_cerevisiae.fasta
FASTA type: protein
Proceed? [y/n] (Default: y): 
```

It detects the 'fasta' file and asks if you wish to create a database. Say 'yes' again, and click 'enter' to accept the defaults for 'database title' and 'taxid'.
```bash
Enter a database title or will use 'saccharomyces cerevisiae': 
Enter taxid (optional): 

Building a new DB, current time: 06/19/2020 15:02:35
New DB name:   /home/cs02gl/Desktop/git/tutorials/sequenceserver/saccharomyces_cerevisiae.fasta
New DB title:  saccharomyces cerevisiae
Sequence type: Protein
Keep MBits: T
Maximum file size: 1000000000B
Adding sequences from FASTA; added 6002 sequences in 0.21314 seconds.
```
Congrats, a new protein database has been created. You can now reissue the "sequenceserver -d ." command to start the server. You should see the message:
```bash
[2020-06-19 15:03:37] INFO  Reading configuration file: /home/user/.sequenceserver.conf.
[2020-06-19 15:03:37] WARN  Will listen on all interfaces (0.0.0.0). Consider using 127.0.0.1 (--host option).
** SequenceServer is ready.
   Go to http://localhost:4567 in your browser and start BLASTing!
   To share your setup, please try one of the following: 
     -  http://XXX.XXX.X.XX:4567
   Press CTRL+C to quit.
```

If you are running this on your local machine it will also open your default browser to 'localhost:4567' and show you the blast server interface. If you are running remotely, connect to the server's address on 'ip-address:4567'.




