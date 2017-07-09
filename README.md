# testable-greeter
"The team hasn't implemented any automated tests or created a mechanism for automated deployments of this service to any of the existing environments (dev, test, demo, production) yet."

This looks like our first target. Getting a build pipeline going is good for morale, it allows developers to see the downstream effects. It reduces friction and distance from production.

Most of the shops I've been at have a Jenkins or Bamboo installed already, but maybe there is a more lightweight solution.

Maybe CI is something that is best hosted outside the company, but it's in fashion to have an in-house solution: an argument can be made from a security perspective.

I'm tempted to try out https://buildbot.net/, for its built in understanding of python issues, but in this scenario, I'm not sure if that would be the most useful choice for the team that has to maintain this install.

Nonetheless, I need something I can commit inside a four hour window, and buildbot seems simple enough.

Todo:
 * Configure to pull from git
 * Run unit tests
 * Run integration tests
 * making sure the buildbot service start up automatically on reboot, that we patch systemd to allow for processes owned by users not currently logged in.

* Tie login into to a shared authentication/authorization resource
* build in load testing with Tsung or apachebench some such.
* Fuzz testing the input with something like  boofuzz or OWASP's "Zed Attack Proxy Project"
* autoscaling

Building a build-bot master:
mkdir -p ~/tmp/bb-master
cd ~/tmp/bb-master

Assuming Python 3, because I know that short term, seemingly temporary hacks can persist for *years* (and EOL on 2.7 has been declared for 2020)

python3 -m venv sandbox

source sandbox/bin/activate

(Assumes Bash or Zsh)

pip install --upgrade pip

pip install 'buildbot[bundle]'

buildbot create-master master

mv master/master.cfg.sample master/master.cfg

and then a

 buildbot start master

 (Don't try this on an OSX box, you will experience disappointment. I ended up using an Ubuntu 16.04 box which is working out nicely)

Building a worker:

mkdir -p ~/tmp/bb-worker

cd ~/tmp/bb-worker

python3 -m venv sandbox

source sandbox/bin/activate

Install the buildbot-worker command:

pip install --upgrade pip

pip install buildbot-worker

# required for `runtests` build

pip install setuptools-trial

Now, create the worker:

buildbot-worker create-worker worker localhost example-worker pass

Note: If you decided to create this from another computer, you should replace localhost with the name of the
computer where your master is running.

The username (example-worker), and password (pass) should be the same as those in master/master.cfg;
verify this is the case by looking at the section for c['workers']:

cat ../bb-master/master/master.cfg

And finally, start the worker:

buildbot-worker start worker

