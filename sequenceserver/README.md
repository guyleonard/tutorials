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


