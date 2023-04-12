# didactic-umbrella
Naranjo's DevOps Engineer submission.

## Disclaimer
A quick search on GitHub shows many replies to other home tasks sent by your
company. To avoid future leaks of my response and allow you to reuse the home
task, I will avoid including your name and other key details in the submission
contents.

## Submission structure, compatibility and requirements
This submission consists in 3 different repositories.

* This one: Giving a general defense for the decisions I took and keeping the
code for the requested diagram.
* https://github.com/fjfnaranjo/improved-couscous: With the API.
* https://github.com/fjfnaranjo/fuzzy-chainsaw: With the configuration code.

### Compatibility
The code in the last two repositories should be easy to run in any POSIX shell
with a properly configured `docker` command. A Makefile will be provided as
reference for the commands needed.

I'm developing using Debian stable and I haven't taken portability with MacOS
into account, but if you are using Mac you should be able to run it without any
issues. Feel free to contact me with the details if something fails to run.

Also, I'm not considering WSL compatibility. Again, feel free to ask me if you
find any problems.

### AWS credentials
Running the configuration files in the fuzzy-chainsaw repository will require a
set of valid AWS credentials.

The tooling assumes the credentials will be available in the environment. Check
https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html .
