# Hello Git sparse-checkout 

An experiment on using [git](https://git-scm.com/) as a minimal [configuration management](http://en.wikipedia.org/wiki/Configuration_management) system. 

## Use case - Requirements

1. Maintain configuration for a group of machines in a shared repository - including history, traceability, ...
	- The central location is represented by a [bare git repository](https://www.atlassian.com/git/tutorials/setting-up-a-repository/git-init) referred to as a [git remote](http://git-scm.com/docs/git-remote) with the name `origin`
2. Configuration can be changed from local peers in a decentralized manner
	- Clients can [checkout](http://git-scm.com/docs/git-checkout) from `origin`, [commit](http://git-scm.com/docs/git-commit) local changes, [push](http://git-scm.com/docs/git-push) them back and [pull](http://git-scm.com/docs/git-pull) updates using the well known [git workflows](https://www.atlassian.com/git/tutorials/comparing-workflows)  
3. Deploy changes from the central location to any or all machines - including multi-user concurrency, ...
    - A machine can update its configuration using git [pull](http://git-scm.com/docs/git-pull)
    - To trigger or push changes from master various [remote shell](http://en.wikipedia.org/wiki/Remote_Shell) technologies can be used, i.e. [RExec (Ant)](https://ant.apache.org/manual/Tasks/rexec.html), [PAExec](http://www.poweradmin.com/paexec/), [WinRM](https://msdn.microsoft.com/en-us/library/aa384426%28v=vs.85%29.aspx), just plain [ssh](http://en.wikipedia.org/wiki/Secure_Shell), etc.
4. Commit local changes back to the master
    - Local changes can be committed (author, email, time) and pushed back to `origin`  
5. Any machine should only see its own configuration, configuration specific to other machines should be transparent
    - The configuration for each machine or group of machines (role) is maintained in a specific subdirectory. On production all other role confiurations are skipped using [sparse-checkout](http://git-scm.com/docs/git-read-tree)

## Setup

1. Create master repository

	    mkdir master
	    cd master
		git init --bare --shared

2. Push this repo (`client`) to a master
		
		client> git remote add -f origin path/to/master
        git push origin master

3. Create production repositories: `prodA`, `prodB`, etc.

		mkdir prodA
        cd prodA
        git init
		git config core.sparseCheckout true
		git remote add -f origin path/to/master
		echo %COMPUTERNAME%/*>.git/info/sparse-checkout
		git pull origin master

## Workflows

2. Commit changes from `client` to `master`

		client> git add -A
		git commit -m "Message"
		git push origin master

3. Deploy changes from `master` to `production`

		prodA> git pull origin master

4. Commit local changes from `production` back to `master`

		prodA> git add -A
		git commit -m "Message"
		git push origin master
 
## Links

- [Configuration management (Wikipedia)](http://en.wikipedia.org/wiki/Configuration_management)
- [Matrix of Pain (Wikipedia)](http://en.wikipedia.org/wiki/Matrix_of_pain)
- [Is it possible to do a sparse checkout without checking out the whole repository first? (StackOverflow)](http://stackoverflow.com/questions/4114887/is-it-possible-to-do-a-sparse-checkout-without-checking-out-the-whole-repository)
- [Infrastructure as Code - A comprehensive overview (Patrick Debois, Jax2012))](http://www.jedi.be/blog/2013/05/24/Infrastructure%20as%20Code/)
- [git - the simple guide](https://rogerdudler.github.io/git-guide/)
