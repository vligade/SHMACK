# S.H.M.A.C.K

S. = Spark

H. = Hatch

M. = Mesos

A. = Akka

C. = Cassandra

K. = Kafka

## Trying some default stack for Big Data prototypes (May come out different from the above).

#<font color="red">WARNING: things can get expensive $$$$$ !</font>
When setting up the tutorial servers on Amazon AWS and letting them running, there will be monthly costs of approx **1700 $** !
Please make sure that servers are only used as required. See [FAQ](#avoidBill) section in this document.

# What do I need to read before working on this project? --> Exactly THIS! #
* Backlog is an Excel-File in order to prioritize and filter issues: **[here](#backlog)**
* Details for Backlog-Items are github issues with a link from the Backlog
* Any files used for work on specific issues can be found **[here](https://github.com/Zuehlke/SHMACK/tree/master/03_analysis_design/Issues)**, see also **[FAQ](#nonImplFiles)**
* Whatever can be automated shall be automated and checked in the `04_implementation` folder **[here](https://github.com/Zuehlke/SHMACK/tree/master/04_implementation)** (common sense may be applied ;-)
* The [Vision](#vision)
* [Installation instructions](#installation)

# Vision #
* We want like to be fast when ramping up cloud infrastructure.
* We do not want to answer "we never did this" to customers when asked.
* We want to know where the issues and traps are when setting up cloud infrastructure with the SHMACK stack.
* We want to create a RUA for Machine Learning to acquire customers and show competence.

* **@wgi: TODO Please correct / append this vision.**

### Installation

#### To be done once:
* Create AWS account **[here](https://aws.amazon.com/de/)**
* Create a Virtual Machine 
  * Recommended: **[Ubuntu >= 14.04.3 LTS](http://www.ubuntu.com/download/desktop)** with VMWare-Player
    * Optional: Install cinnamon desktop manager: 
      * http://www.webupd8.org/2014/12/install-cinnamon-24-stable-in-ubuntu.html
      * Un-Assign `Ctrl-Space` using `ibus-setup`  see **[here](http://askubuntu.com/questions/445676/ctrl-space-not-working-in-terminal-after-installing-cinnamon)**
        
  * Alternative: **[LinuxMint >= 17.02](http://www.linuxmint.com/download.php)** with VirtualBox 
  * **ATTENTION**: Do NOT only start the OS from the downloaded ISO image. INSTALL the OS to the virtual machine on the virtual machine's harddisk.
  * Hint: If Copy/Paste does not work, check whether VM-tools are installed.
* In the Virtual machine
  * `sudo apt-get install xsel git`
  * setup GIT (source of commands: https://help.github.com/articles/set-up-git/ )
    * `git config --global user.name "YOUR NAME"`
    * `git config --global user.email "your_GITHUB_email_address@example.com"`
    * `git config --global push.default simple`
    * Setup github Credentials
      * `git config --global credential.helper cache`
      * `git config --global credential.helper 'cache --timeout=43200'`  (cache 1 day)
  * `mkdir ${HOME}/shmack`
  * `cd ${HOME}/shmack && git clone https://<yourgithubusername>@github.com/Zuehlke/SHMACK.git repo`
  * `cd ${HOME}/shmack/repo/04_implementation/scripts && sudo -H bash ./setup-ubuntu.sh`
  * Setup AWS console (Source: http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html )
    * Create AWS user including AWS Access Key (can be deleted to revoce access from VM)
      * https://console.aws.amazon.com/iam/home?#users
      * Username: `shmack`
      * **DON'T TOUCH mouse or keyboard - LEAVE THE BROWSER OPEN** (credentials are shown only here, optionally download credentials and store them in a safe place only for you)
    * `aws configure`
      * `AWS Access Key ID [None]: [from browser page]`
      * `AWS Secret Access Key [None]: [from browser page]`
      * `Default region name [None]: us-west-1`  (VERY important, DO NOT change this!)
      * `Default output format [None]: json`
    * Assign Admin-Permissions to user `smack`: 
https://console.aws.amazon.com/iam/home?#users/shmack 
    * Create a AWS Key-Pair in region **us-west-1**: 
https://us-west-1.console.aws.amazon.com/ec2/v2/home?region=us-west-1#KeyPairs:sort=keyName
      * Key pair name: `shmack-key-pair-01` 
        **Attention: Use exactly this Key pair name as it is referenced in the scripts!**
      * Create .ssh directory `mkdir ${HOME}/.ssh`
      * Save the key pair (copy-paste is OK) `vim.tiny ${HOME}/.ssh/shmack-key-pair-01.pem`
        **Attention: Use exactly this filename as it is referenced in the scripts!**
      * `chmod 600 ${HOME}/.ssh/shmack-key-pair-01.pem`

  * Download and install eclipse
    * Download `Eclipse IDE for Java EE Developers ` from https://www.eclipse.org/downloads/ 
    * Extract eclipse: `cd ${HOME}; tar xvfz Downloads/eclipse-jee-mars-1-linux-gtk-x86_64.tar.gz` 
  * Append the following lines at the **end** of your `${HOME}/.bashrc`
```
alias cds='cd ${HOME}/shmack/repo/'
alias eclipse='nohup ${HOME}/eclipse/eclipse > /dev/null 2>&1 &'
PATH=${PATH}:${HOME}/shmack/repo/04_implementation/scripts
PATH=${PATH}:${HOME}/shmack/repo/04_implementation/scripts/target/dcos/bin
export PATH
```
  * Add gradle support to eclipse
    * open `eclipse`
    * Open `Help --> Eclipse Marketplace`
    * Install `Gradle IDE Pack`
  * Import Gradle projects from `${HOME}/shmack/repo/04_implementation` into eclipse

    
#### Stack Creation and Deletion 
##### Stack Creation
  * `${HOME}/shmack/repo/04_implementation/scripts/create-stack.sh`
    * Wait approx. 10 Minutes
    * **Do NOT interrupt the script!** (especially do **NOT** press Ctrl-C to copy the instructed URL!)
    * In case of failures see [Troubleshoting Section](#setupFailing)
  * Open URL as instructed in `Go to the following link in your browser:` and enter verification code.
  * `Modify your bash profile to add DCOS to your PATH? [yes/no]` --> yes (first time only)
  * Confirm all installations (several times): `Continue installing? [yes/no]` --> yes
  * <a name="confirmSsh"></a>Login once using ssh (in order to add mesos master to known hosts)
    * `${HOME}/shmack/repo/04_implementation/scripts/ssh-into-dcos-slave.sh 0`
    * Confirm SSH security prompts
    * Logout from the cluser (press `Ctrl-d` or type `exit` twice)
  * Optional: Check whether stack creation was successful, see **[here](#checkStackSetup)** 

  
<a name="stackDeletion"></a>
##### Stack Deletion
  * Option 1 (recommended):
    `${HOME}/shmack/repo/04_implementation/scripts/delete-stack.sh`
  * Option 2 (manual):
    * go to https://console.aws.amazon.com/cloudformation/ and delete the stack
  * Troubleshooting
    * Sometimes the deletion failes after approx. 20 minutes as a default VPC security group cannot be deleted. Reasons are likely race conditions. In this case the repetition of the stack deletion (either by Option 1 or Option 2) likely resolves the problem.
  * Verification (to avoid too high bills) make sure that...
	* ... the stack is deleted: https://console.aws.amazon.com/cloudformation/home?region=us-west-1#/stacks?filter=active
	* ... there are no autoscaling groups left: https://us-west-1.console.aws.amazon.com/ec2/autoscaling/home
	* ... there are no running EC2 instances or Volumes: https://us-west-1.console.aws.amazon.com/ec2/v2/home?region=us-west-1

#### Affiliate
* Focusgroup - Big Data / Cloud
* Team - TODO
* Initiator - wgi

# Links #
* [Mesosphere Homepage](https://mesosphere.com/)
* [Documentation](http://docs.mesosphere.com/)
* [Tutorials](https://docs.mesosphere.com/tutorials/)
* [DCOS Service Availability](https://docs.mesosphere.com/reference/servicestatus/)
* Articles
  * [MESOSPHERE DATACENTER OPERATING SYSTEM IS NOW GENERALLY AVAILABLE](https://mesosphere.com/blog/2015/06/09/the-mesosphere-datacenter-operating-system-is-now-generally-available/)
  * [MEET A NEW VERSION OF SPARK, BUILT JUST FOR THE DCOS](https://mesosphere.com/blog/2015/06/15/meet-a-new-version-of-spark-built-just-for-the-dcos/)
  * [EVERYTHING YOU NEED TO KNOW ABOUT SCALA AND BIG DATA](https://mesosphere.com/blog/2015/07/24/learn-everything-you-need-to-know-about-scala-and-big-data-in-oakland/)
  * [APPLE DETAILS HOW IT REBUILT SIRI ON MESOS](https://mesosphere.com/blog/2015/04/23/apple-details-j-a-r-v-i-s-the-mesos-framework-that-runs-siri/)
* Other Ressources
  * AMP Lab - Reference Architecture: https://amplab.cs.berkeley.edu/software/
  * AMP Lap Camp with exercices: http://ampcamp.berkeley.edu/5/  
  * Public Datasets (S3): https://aws.amazon.com/de/public-data-sets/
  * Reference Architecture for Netflix Style recommendation engines: https://github.com/fluxcapacitor
  * Apache Spark Example Use Cases (with Code): https://github.com/4Quant
  * Apache Spark Twitter Word Count: https://github.com/snowplow/spark-example-project


# Glossary
| Term | Definition |
|--------|--------|
| Issue  | = Can be a **"User Story"** (to be in sync with scrum and github terminology) or a **Bug**|


# Important Limitations / Things to consider before going productive
* As of 2015-10-28 the DCOS stack does **NOT work in AWS Region `eu-central-1` (Frankfurt)**. Recommended region to try is `us-west-1`. Take care of **regulatory issues** (physical location of data) when thinking about a real productive System.
* What if the number of client request "explodes". Is there a way to do autoscaling with DCOS / Mesophere WITHOUT human interaction?
* As of 2015-11-13 **all data in HDFS is lost** when scaling down, e.g. from 10 to 5 Slave nodes. This is a blocking issue. If unresolved productive use of the Stack is not possible. see **[here](https://github.com/Zuehlke/SHMACK/blob/master/03_analysis_design/Issues/Issue-10%20HDFS-Access/Scaling%20Test.docx)** According to the mesosphere development team (chat), this issue is addressed by **[maintenance primitives](https://mesosphere.com/blog/2015/10/07/mesos-inverse-offers/)**. But it is not clear when it will be finished.
* Make sure that admin access to the Mesos Master console is secure. As of 2015-11-27 only **passwordless** http acces is possible. https needs to be implemented.


# FAQ
<a name="avoidBill"></a>
## How do I avoid to be surprised by a monthly bill of **1700 $** ?
As of 2015-10-23 there is **no** officially supported way to suspend AWS EC2 instances.
see [Stackoverflow](http://stackoverflow.com/questions/31848810/mesososphere-dcos-cluster-on-aws-ec2-instances-are-terminated-and-again-restart) and [Issue](https://github.com/Zuehlke/SHMACK/issues/2)

The only official supported way to stop AWS bills is to completely delete the stack.
**ATTENTION**: 
* To delete a stack it is not sufficient to just terminate the EC2 instances as they are contained in an autoscaling group.
* To delete a stack see **[here](#stackDeletion)**

<a name="backlog"></a>
## Where is the Backlog?
The Backlog is an Excel-File which contains for each story
- the short name 
- the Category 
- the link to the Issue in github (which contains the details description of the story)

We use the Excel-Format due to the following reasons: 
1. we want to prioritize the issues
2. we want to filter for open issues only (otherwise the backlog would become too long)
3. we do not (yet) want to introduce another tool like trello to keep thing simple and together.

<a name="nonImplFiles"></a>
## Where do I put my notes / non-implementation files when working on an issue (including User-Stories) ?
Into the `03_analysis_design/Issues` folder, see https://github.com/Zuehlke/SHMACK/tree/master/03_analysis_design/Issues
````
<git-repo-root>
  |- 03_analysis_design
     |- Issues
        |- Issue-<ID> - <any short description you like>
           |- Any files you like to work on
````

## How do I scale up/down the number of slave nodes?
`${HOME}/shmack/repo/04_implementation/scripts/change-number-of-slaves.sh <new number of slaves>`
**Attention**: Data in HDFS is **destroyed** when scaling down 3 slave nodes or less!!

## Which Java Version can be used?
As of 2015-11-17 Spark-Jobs are failing because only Java 7 is available in the cluster.
Therefore Java 7 must be used until support for Java 8 is available.
Remark: as of 2015-11-17 the EC2 instance have Java 8!

<a name="checkStackSetup"></a>
## What should I do to check if the setup was successful?
Execute the testcase `ShmackUtilsTest` in eclipse.
If this testcase fails: see **[here](#inTestcasesFailing)**

## How can I execute Unit-Tests on a local Spark instance?
Look at the examples:
* `JavaSparkPiLocalTest`
* `WordCountLocalTest`

## How can I execute Unit-Tests on a remote Spark instance (i.e. in the Amazon EC2 cluster)?
Look at the examples:
* `JavaSparkPiRemoteTest`
* `WordCountRemoteTest`

Make sure that every thecase hase it's own `testcaseId`. This id needs only to be distinct only within one Test-Class.
```
String testcaseId = "WordCount-" + nSlices;
RemoteSparkTestRunner runner = new RemoteSparkTestRunner(JavaSparkPiRemoteTest.class, testcaseId);
```

To execute the tests do the following:
* invoke `gradle fatJarWithTests` (available as eclipse launch configuration)
* invoke the JUnit test from within eclipse (or your favorite IDE)

## How can I use `src/test/resources` on a remote Spark instance?
* Synchronize the  `src/test/resources` to the HDFS filesystem
  * `RemoteSparkTestBase#syncTestRessourcesToHdfs()`
  * Example: `WordCountRemoteTest#testWordcountRemote()`
  * Note that thanks to `rsync` **only changes** will be transferred from your laptop to the EC2 instance. This saves huge amounts of time and bandwith ;-). Nevertheless there is no efficient way to sync between the EC2-Master node and the EC2-HDFS Filesystem. But this should be no problem as bandwidth within the EC2 cluster is very high.
  
* Use the required ressource from within the Spark-Job (executed remotely) addressedb by the HDFS-URL , e.g.
```
   see WordCountRemoteTest#main():
	// resolves to hdfs://hdfs/spark-tests/resources/tweets/tweets_big_data_2000.json
    String inputFile = getHdfsTestRessourcePath("tweets/tweets_big_data_2000.json");
```

## How can I retrieve the results from a Unit-Test executed  on a remote Spark instance?
Use the `RemoteSparkTestRunner#`**`getRemoteResult()`** as follows:
* `executeSparkRemote(String...)`
* `waitForSparkFinished()`
* `getRemoteResult()`

Examples: 
* `JavaSparkPiRemoteTest`
* `WordCountRemoteTest`

## How does the JUnit-Test know when a Spark-Job is finished?
The `RemoteSparkTestRunner#executeWithStatusTracking()` is to be invoked by the spark Job. It writes the state of the spark job to the HDFS filesystem
The JUnit test uses the `RemoteSparkTestRunner` to poll the state, see `RemoteSparkTestRunner#waitForSparkFinished()`.

Examples: 
* `JavaSparkPiRemoteTest`
* `WordCountRemoteTest`

Nevertheless it can happen that due to a severe error, that the status in HDFS is not written.
In this case see **[here](#sparkJobsFailing)** 

## How can I execute command in the EC2 cluster from a local JUnit Test?

Use methods provided by `ShmackUtils`:
* `runOnMaster(CommandLine, ExecExceptionHandling)`
* `runOnMaster(ExecExceptionHandling, String, String...)`
* `runOnMaster(String, String...)`

These methods will typically throw an exception if the return code is not 0 (can be controlled using ExecExceptionHandling).


## How do I read / write files from / to the HDFS file system in the EC2 cluster?
You can do this ...
* ... either **locally** from your laptop:
  * from JUnit Tests: use  method provided by `ShmackUtils`, e.g.
    * `copyFromHdfs(File, File)`
    * `copyToHdfs(File, File)`
    * `syncFolderToHdfs(File, File)`
    * `syncFolderFromHdfs(File, File)`
    * `deleteInHdfs(File)`
    * `getHdfsURL(File)`
    * `readByteArrayFromHdfs(File)`
    * `readStringFromHdfs(File)`
    * Note that you can simply use a java.io.File to address files in HDFS, e.g. `/foo/bar.txt` will be written to the HDFS URL `hdfs://hdfs/foo/bar.txt`
  
  * from a bash:
    * `copy-from-hdfs.sh`
    * `copy-to-hdfs.sh`
    * `sync-from-hdfs-to-local.sh`
    * `sync-to-hdfs.sh`
    
* ... or from a Spark-Job executed **remote** in the EC2 cluster:
  *  use `com.zuehlke.shmack.sparkjobs.base.HdfsUtils`



# Troubleshooting
## I get a `SignatureDoesNotMatch` error in aws-cli.
Likely the clock of your virtual maching is wrong. 
To fix this:
* Shutdown VM completely (reboot is *not* enough in VirtualBox)
* Start VM
* Now the clock of the VM should be OK and aws-cli should work fine again.

<a name="setupFailing"></a>
## What should I do if the setup of the stack has failed?
* Try to understand the failure and fix it. Goal: As much as possible is automated and others do not fall into the same issue.
* Delete the stack to make sure there are no costs, see **[here](#stackDeletion)**
* If you still habe time: Try to create the stack again from scratch, but do not forget the **[running costs](#avoidBill)**...


## What should I do if ssh does not work?
In most cases the reason for this is that ssh is blocked by corporate networks.
Solution: Unplug network cable and use `zred` WiFi.

## What should I do to check if the setup was successful?
Execute the testcase `ShmackUtilsTest` in eclipse.
If this testcase fails: see **[here](#inTestcasesFailing)**

<a name="inTestcasesFailing"></a>
## What should I do if Integration testcases do not work?
Be sure to have confirmed idendity of hosts, see **[here](#confirmSsh)**


<a name="sparkJobsFailing"></a>
## What should I do if Spark-Jobs are failing?
* Open the mesos Web-UI `${HOME}/shmack/repo/04_implementation/scripts/open-shmack-mesos-console.sh`
* Click on the Link to the `sandbox` of your spark-job
* Click on `stderr`  
* Example see: **[here](https://github.com/Zuehlke/SHMACK/blob/master/03_analysis_design/Issues/Issue-7%20Spark%20Word%20Count/Issue-7%20Spark%20Word%20Count.docx)**

## To start with a clean state, you may delete the whole HDFS Filesystem as follows
`ssh-into-dcos-master.sh`
`hadoop fs -rm -r -f 'hdfs://hdfs/*'`
___
* [github] - See other project from Zühlke on github
* [bitbucket] - ee other project from Zühlke on bitbucket

[github]:https://github.com/zuehlke-ch
[bitbucket]:https://bitbucket.org/zuehlke/
