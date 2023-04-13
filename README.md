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
* The state of the API will be stored in a DynamoBD table.
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
interface with DynamoBD easily.

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
   │               │                                             AWS │
   │     ┌──────────▼──────────┐    ┌─────────────────────┐          │
   │     │ << CloudFront >>    │    │ << API Gateway >>   │          │
   │     │                     ├────►                     ├───────┐  │
   │     │       hello         │    │       hello         │       │  │
   │     └─────────────────────┘    └─────────────────────┘       │  │
   │                                                              │  │
   │         ┌────────────────────────────────────────────────────┘  │
   │         │                                                       │
   │         │   ┌───────────────────┐    ┌────────────────────┐     │
   │         │   │   << Lambda >>    │    │  << DynamoBD >>    │     │
   │         └───►                   ├────►                    │     │
   │             │       hello       │    │       hello        │     │
   │             └───────▲──────▲────┘    └────────────────────┘     │
   │                     │      │                                    │
   └──────────────────── │ ──── │ ───────────────────────────────────┘
                         │      │
            ┌────────────┘      │
            │                   │

            0                   0
            ┼ GitHub Actions    ┼  API Developer (zappa)
            ┴                   ┴
```

## Proposal caveats and further discussion

### Caveats

### Future improvement

### SRE considerations (part of point 3 in code test)
