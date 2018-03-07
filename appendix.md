# APPENDIX

---

<A name="part-A"> </A>

## Part A. Helpful `bash` prompt annotations

The following lines, appended to the `~/.bashrc` file, can be useful when working on the `bash` command line to do software development, and can particularly expedite tasks while using `git` to manage version control:  

```
function tgrps1 {
  local BOLDRED="\[\033[1;31m\]"
  local GREEN="\[\033[0;32m\]"
  local BOLDYELLOW="\[\033[1;33m\]"
  local BOLDCYAN="\[\033[1;36m\]"
  local NOCOLOR="\[\033[0m\]"
  export GIT_PS1_SHOWDIRTYSTATE=1
  PS1=$BOLDRED'`echo "ERROR($?) " | sed '\''s/^ERROR(0) \$//'\''`'
  PS1+="$GREEN\$(date +%H:%M%p)"
  PS1+="$BOLDYELLOW\$(__git_ps1) "
  PS1+="$BOLDCYAN\u$NOCOLOR@\h:\w \$ "
}
tgrps1
```  

Once these lines are added, continue by bringing up a new command window or doing an `exec bash` within the current one, and the command prompt should now reflect:

- the name of the branch currently checked out in the `git`-enabled repository in which you sit - that is, the machine-local folder aka directory aka "repo" closest to and at-or-above your current working directory
- the status relative to changes that might need to be `add`ed or `commit`ed to save work done in the repository
- the return code (highlighted in red) from the last command issued on the command line.

---

<A name="part-B"> </A>
## Part B. Running `openvpn` for access to firewall-protected RENCI servers and VM's

OpenVPN is available on a variety of platforms. Once installed and enabled, it will allow you to connect to RENCI-internal servers which would otherwise be unavailable from outside the office spaces.  In order to download and use OpenVPN, the first step is to navigate to  https://vpn.renci.org , and enter your credentials (RENCI username and password.)  

Then follow the instruction below that corresponds to your choice of computing platform:

- **MacOS**
    1. On first accessing the above URL, you will be prompted to download a OpenVPN Connect `.dmg` file.
    1. Once it is downloaded, clicking on this package installer will most likely fail to install the software the first time, as MacOS guards against software from developers it doesn't recognize. Solve this problem by opening the `System Preferences` dialog and going to the `General` settings tab.  Your computer will remember the attempt to install the OpenVPN package, and will offer the option to override the OS's initial barrier to installing.  Select it and click through, as the folks at OpenVPN are certainly not rogue developers....
    1. Subsequent openings of the above-mentioned URL should offer a 'connect' or 'login', and you'll want to choose the former after entering your credentials.

- **Microsoft Windows**
    * The process for enabling the VPN under Windows is almost identical to that given for MacOS above, except that the product being offered for download may be PrivateTunnel instead of OpenVPN Connect. Either will work.  You will most likely need administrative privileges associated with your windows login.

- **Linux**
    1. Once logged into the above-mentioned URL, click on the bottom-most link to download your connection profile. It will probably be called `client.ovpn`, but it contains your own specific credentials for logging in past RENCI firewalls.   
    1. Assuming you have `sudo` access on your machine, open up a terminal window and  issue the command : `sudo apt-get install openvpn`
    1. In the same terminal window command `openvpn -config [/path/to/client.ovpn]` to enable VPN access. You'll be prompted again for your RENCI user name and password.

---
<A name="part-C"> </A>
<A name="dpkg-heck"> </A>

## Part C. Getting out of "dpkg heck"

It is possible to arrive at an impasse with the `dpkg` system, in which (for example) `irods-server` package is partially installed and may neither be fully removed nor successfully installed using `dpkg` and `apt` commands. This might happen if, for example, the /var/lib/irods directory tree is removed before any attempt to uninstall the irods-server package itself, which owns and depends on some of the files under that directory. 

When this happens, one should do the following:
```
sudo su -
mkdir -p /var/lib/irods/packaging
cd /var/lib/irods/packaging ; touch  postinstall.sh  preremove.sh 
chmod +x postinstall.sh  preremove.sh
chown irods:iords postinstall.sh  preremove.sh
cd ..
dpkg -r irods-server
```

<A name="part-D"> </A>

## Part D. Jenkins and Build-a-Bear for Testing and CI  in a collaborative environment

Thorough testing and Continuous Integration (CI) of new features encourages -- or demands, depending on the number of developers and/or active feature branches on a project -- an automated approach in order to ensure that these code changes do not interact destructively.  Once it's decided to automate testing of such things, it's a straightforward decision to extend the testing to ensuring the software, including projected changes, runs as expected on all of the different computing platforms for which it has been promised to run.

And so it is also for the iRODS core software: before a feature or code fix is added to the official code-base on GitHub, it is first staged and tested on a [Jenkins](http://jenkins.io) server where a system of automatic tests vet the change automatically - not only against the built-in system of unit tests, but also across the comprehensive list of operating systems iRODS supports. (These are currently Centos6 and 7, and Ubuntu 12,14, and 16).

Enter "Build-a-Bear".

Build-a-Bear, as it's known to the core iRODS team, is a system of scripts that allows developers to push a custom build or set of changes from a local directory (ie git repo), to be build and tested prior to making them potentially official with a pull request. Usually the main repo pushed is one in which prospective changes have been made to iRODS server code for testing.  Presumably unit tests will also have been added to the code-base , in order to verify that the changes support the desired function (perhaps a new feature) and do not cause problems in pre-existing tests.

The changes need to have been committed in the local repository; then the repo can be synced to a shared directory visible to the Jenkins server (http://172.25.14.125:8080). (*Note* if using the site outside of RENCI spaces, follow OpenVPN instructions above to get past the firewall first!) This is done with a pre-formulated script customarily kept by team members in [*~/bin/syncbuildabear*](./syncbuildabear.sh). The lines in this script can be duplicated as necessary to sync more than one repo.  One SHA code will be printed for every repo as it is synchronized.

The SHA printed out for each of your repos synced to B-a-B can be entered into the fields presented under **Jenkins -> Personal -> irods-build-and-test-workflow -> Build With Parameters** . In place of the "commit-ish" SHA you can also enter a version number such as "4-2-stable", for example.  You might do this for iRODS client icommands, if the local commit(s) you are testing correspond to that version but don't include a custom build for the icommands themselves.

Keep in mind versions (latest production version at the last revision of this document is 4.2.2) currently must be equal across components. So the build ID you enter for different components of a complete iRODS system -- the icommands , the ICAT server and its plugins (soon also to be a part of Build-a-Bear tests !) in the fields pictured below need to be for matching versions.

<A> <IMG src="./Jenkins.png">
</A>  

...

## NFS link to /projects directory
```
na-projects.edc.renci.org:/ /projects nfs vers=3,hard,intr,rw,bg,timeo=600,rsize=65536,wsize=65536 0 0
```

...

  (*This section not yet finished*)
