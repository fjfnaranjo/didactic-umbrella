# didactic-umbrella
Naranjo's DevOps Engineer code test submission defense.

## Disclaimer
A quick search on GitHub shows many replies to other code tests sent by your
company. To avoid future leaks of my response and allow you to reuse the home
task, I will avoid including your name and other key details in the submission
contents.

## Submission structure, compatibility and requirements
The submission consists of 3 different repositories.

* This one: Giving a general defense for the decisions I took and keeping the
code for the requested diagram.
* https://github.com/fjfnaranjo/improved-couscous: With the API.
* https://github.com/fjfnaranjo/fuzzy-chainsaw: With the configuration code.

### Running the code in the respositories
The code in the last two repositories should be easy to run in any POSIX shell
with access to the `docker` and `docker-compose` commands. A `Makefile` file
will be provided that can be use directly or as a reference for the commands
needed. Check for specific running instructions in each repository `README.md`.

I'm developing using Debian stable and I haven't taken portability with MacOS
into account, but if you are using Mac you should be able to run it without any
issues. Feel free to contact me with the details if something fails to run (you
can also create an issue in the repository).

Also, I'm not considering WSL compatibility. Again, feel free to ask me if you
find any problems.

### AWS credentials
Running the configuration files in the 'fuzzy-chainsaw' repository will require
a set of valid AWS credentials.

The tooling assumes the credentials will be available in the environment. Check
https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html .

## Proposal design and general considerations

> NOTE: Instead of following the order for the points in the code test, I'm
> following a more natural order that fits better the proposal defense. I will
> indicate the proper correspondence between the defense sections and the code
> test order in the section headers.

### Configuration and deployment part (point 3 in the code test)
The requested task fits very well into the serverless case. The API is simple,
it has a straightforward business logic and the time restriction associated to
any code test encourages an easy to deploy solution.

Said that, the approach I suggest has its own caveats. I will elaborate on them
later. But for now, the general description of the proposal goes as follow:

* The API will be deployed as a Lambda cloud function.
* The state of the API will be stored in a DynamoDB table.
* In front of the API, an API Gateway will be configured.
* Finally, in front of the API Gateway, a CloudFront distribution.

On the configuration/deployment side, this should be enough for now. Future
improvements will be also discussed later, but for now, the absence of VPN
dependant services will make the things even easier.

The 'fuzzy-chainsaw' repository will handle this first part.

### API part (point 1 in the code test)
I'm mainly a Python developer, so the function will be developed using Python
with just two runtime dependencies:

* Bottle: Its the perfect fit for the API endpoints. I need the lightest
possible solution and I don't have to deal with the extra complexity of
asynchronous code (because each call to the API will be serve as a single call
to a function).
* Boto3: Is already needed by Zappa and it will allow the application to
interface with DynamoDB easily.

Zappa will handle the packaging of the function code and the deployment to
Lambda. It's a very popular tool and integrates very well with other AWS
services.

Basic inversion of control will be implemented in the API views. Combined with
pytest and static analysis tools, the quality requirements for the task will be
easy to achieve.

CI/CD will be implemented as GitHub actions.

Find the code for this part in the 'improved-couscous' repository.

### Deployment diagram (point 2 in the code test)
```
                   0
                   ┼  API Client (https)
                   ┴
                   │
   ┌────────────── │ ────────────────────────────────────────────────┐
   │               │                                       AWS Cloud │
   │     ┌─────────▼───────────┐    ┌─────────────────────┐          │
   │     │ << CloudFront >>  □ │    │ << API Gateway >> □ │          │
   │     │                     ├────►                     ├───────┐  │
   │     │       hello         │    │       hello         │       │  │
   │     └─────────────────────┘    └─────────────────────┘       │  │
   │                                                              │  │
   │         ┌────────────────────────────────────────────────────┘  │
   │         │                                                       │
   │         │   ┌───────────────────┐    ┌────────────────────┐     │
   │         │   │   << Lambda >>  □ │    │  << DynamoDB >>  □ │     │
   │         └───►                   ├────►                    │     │
   │             │       hello       │    │       hello        │     │
   │             └──▲─────────────▲──┘    └────────────────────┘     │
   │                │             │                                  │
   └─────────────── │ ─────────── │ ─────────────────────────────────┘
                    │             │
       ┌────────────┘             │
       │                          │

       0                          0
       ┼ GitHub Actions (zappa)   ┼  API Developer (zappa)
       ┴                          ┴
```

## Proposal caveats and further discussion

### Caveats and future improvements
The most concerning problem about this solution is that the developer has
direct access to the production infrastructure. A botched Zappa deploy can take
the service down. With the GitHub action automation, this is even more
dangerous because creating a new release in GitHub is pretty easy. The proposal
is presented as it is for simplicity but this problem has a simple fix. The
infrastructure definitions can be modified to have two (or more) stages and
sharing only the appropriate credentials. Also, the credentials themselves can
reside only on the GitHub action secrets and proper permissions can be set to
key people in the team to stablish manual verification steps in the CI/CD
chain. Further integration tests and other checks can be implemented to make
the deployments safer.

The second big caveat is how tied to AWS is this solution. Migrating to EC2
instances or Compute VMs would be the next step to make the API independent
from Amazon. A simple host with a MongoDB server and as many hosts as needed
with the API (under some autoscale solution) combined with asynchronous code
will keep the cost efficiency down. Packing the API as a OCI container image
and using an existing K8s platform can keep the implementation of the CI/CD
straightforward. We will have to replace the DynamoDB backup service with a
custom implementation to save the MongoDB data with the desired frequency.

Another point to take into account is that AWS deployments are not usually this
easy. Some AWS services require configuration of VPNs, virtual network devices
and a variety of bridges to keep everything connected. This components require
extra know-how and maintenance/upgrade considerations. For this case, only
non-VPN services are used, but if cost efficiency becomes a problem in the
future other services will eventually appear in the equation.

Finally, I'm not exactly sure about the usage/cost curve of this solution
versus Google related services like Firebase. Without concrete data about the
expected loads I cannot conduct further research. In any case, if the existing
team is more experienced and tied to Google, using Firebase Realtime Database
and Cloud Functions is a similar option to consider if the usage is expected to
be low.

### SRE considerations (part of point 3 in code test)
As a SRE, the first thing you need to consider is how much information do you
have on the platform behavior. AWS Lambda can be easily integrated with
CloudWatch to have detailed graphs about its behavior. AWS X-Ray integration is
also easy, for APMs. Proper use of CloudWatch alarms and SNS will keep the
support team up to the date with any impaired performance. All of this are just
extra definitions/parameters in the Terraform module files.

About formal SLAs, its very difficult to outperform AWS/Google/Azure in this
field. And this solution doesn't introduce points of failure that cannot be
mitigated with proper testing/backups.

Future scaling of the platform is already discussed, but to reiterate, if this
goes up to several MAUs per day, the serverless approach is not going to be
enough (or cheap). Leveraging managed hosts and K8s is going to be the most
reasonable path. The solution its so easy in its design that the only criteria
we can use to shard the data is the API username. If other headers are sent by
the clients we can use them too (like geolocation, for regional base sharding).

Finally, compliance with data regulations is always a difficult task. A
parallel system that loads, modifies, test and stores the backups will be
needed for GDPR as a minimum. If you give me a more complex case I can
elaborate more, but legal compliance should be always checked with an
specialized legal team that follows the developments in the field and review
the processes with the SRE team.
