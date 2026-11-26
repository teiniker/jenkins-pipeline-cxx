# Jenkins Pipeline C++

In this example we see a C++ project that is built using Jenkins.

* `Jenkinsfile`: The pipeline consists of two stages
    - build: Builds the C++ application using CMake
    - test: Runs the test executable

* `Dockerfile`: Jenkins uses an agent in the form of a Docker container 
    for the build process.


## Jenkins Pipeline (Jenkinsfile)

```yml
pipeline 
{
    agent { dockerfile true }
    
    stages 
    {
        stage('build') 
        {
            steps 
            {
                echo 'Build stage: compile all code and build an executable' 
                dir('file_service') 
                {
                    sh 'mkdir -p build && cd build && cmake .. && make'
                }
            }
        }
        stage('test') 
        {
            steps 
            {
                echo 'Test stage: run the test cases' 
                dir('file_service/build') 
                {
                    sh './test/test'
                }
            }
        }
    }
    post 
    {
        success 
        {
            echo 'Archive build artifacts'
            archiveArtifacts artifacts: 'file_service/build/test/test', fingerprint: true
        }
    }
}
```


## Jenkins Job Configuration

We need the following settings for a new Jenkins job:

* New Item
    - Enter an item Name "C++ Pipeline"
	- Pipeline
    - Description (optional)
    - Build Triggers 
	    - Poll SCM: `H/1 * * * *`
    - Pipeline / Definition: 
      - Pipeline Script from SCM: 
        - SCM: Git 
        - Repository URL: `https://github.com/teiniker/jenkins-pipeline-cxx.git` 
        - Branches to build: `*/main` 
      - Script Path: Jenkinsfile

Save these settings.

### What the Settings Mean

* **New Item / Pipeline**: Jenkins offers different job types (Freestyle, 
    Pipeline, Multibranch Pipeline, ...). A **Pipeline** job describes the 
    whole build process as code (Groovy DSL) instead of configuring each 
    build step in the web UI.

* **Item Name**: The name of the job. Jenkins also uses it as the name of 
    the job's workspace directory on the controller/agent.

* **Description**: Free text shown on the job's page. It has no effect on 
    the build.

* **Build Triggers / Poll SCM**: Jenkins periodically asks the Git 
    repository whether there are new commits. A build is only started 
    if something has changed since the last build. The schedule is 
    given in a cron-like syntax (see [Scheduling](#scheduling)).

* **Pipeline / Definition**: Defines where the pipeline script comes from:
    - *Pipeline script*: The script is typed directly into a text field 
        of the job configuration (stored only inside Jenkins).
    - *Pipeline script from SCM*: The script is loaded from the source 
        code repository. This is the preferred way ("Pipeline as Code"): 
        the `Jenkinsfile` is versioned together with the source code, 
        changes can be reviewed, and every branch can have its own pipeline.

* **SCM: Git**: The source code management system used to check out the 
    project. Jenkins clones the repository into the job's workspace 
    before the pipeline runs.

* **Repository URL**: The address of the Git repository. For private 
    repositories, credentials (e.g. a personal access token or SSH key) 
    must be added under *Credentials*.

* **Branches to build**: A branch specifier that selects which branch(es) 
    are checked out and monitored. `*/main` means the branch `main` from 
    any remote (usually `origin/main`).

* **Script Path**: The path of the pipeline script, relative to the root 
    of the repository. The default value is `Jenkinsfile`.

When the job runs, Jenkins clones the repository, reads the `Jenkinsfile`, 
builds the Docker image from the `Dockerfile` (because of 
`agent { dockerfile true }`), and executes the `build` and `test` stages 
inside that container. If all stages succeed, the `post { success { ... } }` 
block archives the test executable as a build artifact.


## Scheduling 

The *Poll SCM* trigger uses a syntax similar to the Unix **cron** utility. 
A schedule consists of five fields separated by whitespace:

```
MINUTE  HOUR  DOM  MONTH  DOW
```

| Field  | Meaning           | Allowed values         |
|--------|-------------------|------------------------|
| MINUTE | Minute of an hour | 0-59                   |
| HOUR   | Hour of a day     | 0-23                   |
| DOM    | Day of the month  | 1-31                   |
| MONTH  | Month             | 1-12                   |
| DOW    | Day of the week   | 0-7 (0 and 7 = Sunday) |

Within each field the following operators can be used:

* `*` matches all valid values.
* `M-N` specifies a range of values (e.g. `9-17`).
* `A,B,C` enumerates several values.
* `*/X` or `M-N/X` defines a step width: every X-th value (e.g. `*/15` 
    in the minute field = every 15 minutes).
* `H` ("hash") is a Jenkins-specific extension. Instead of a fixed value, 
    Jenkins computes a value from a hash of the job name. This spreads 
    the load: if many jobs use `H/15 * * * *`, they don't all poll at 
    exactly the same minute, but each job gets its own (stable) offset.

Our schedule **`H/1 * * * *`** therefore means:

* `H/1` (MINUTE): every minute (a step width of 1 makes the hash offset 
    irrelevant, so this is equivalent to `*`).
* `*` (HOUR, DOM, MONTH, DOW): every hour, every day, every month, 
    every weekday.

=> Jenkins checks the Git repository **once per minute** for new commits 
and starts the pipeline only if a change was found.

Polling every minute is useful in a lab environment because we get fast 
feedback after a `git push`. In production, a less frequent schedule 
(e.g. `H/15 * * * *` = every 15 minutes) or, even better, a **webhook** 
(GitHub notifies Jenkins on every push, *GitHub hook trigger for GITScm 
polling*) is recommended, because it avoids unnecessary load on Jenkins 
and the Git server.

Further examples:

| Schedule            | Meaning                                           |
|---------------------|---------------------------------------------------|
| `H/5 * * * *`       | every 5 minutes                                   |
| `H * * * *`         | once per hour (at a hashed minute)                |
| `H 2 * * *`         | once per day between 02:00 and 02:59              |
| `H 8-18/2 * * 1-5`  | every 2 hours between 08:00 and 18:00, Mon - Fri  |
| `@daily`, `@hourly` | Aliases (Jenkins applies `H` automatically)       |


## Docker Agent

The agent is described by the following Dockerfile:

```yml
# Use Alpine as the base image
FROM alpine:latest

# Install required packages: g++, make, cmake, and build tools
RUN apk add --no-cache git g++ make cmake

# Install Google Test framework
RUN git clone -q https://github.com/google/googletest.git /googletest \
  && mkdir -p /googletest/build \
  && cd /googletest/build \
  && cmake .. && make && make install \
  && cd / && rm -rf /googletest

# Set the working directory
WORKDIR /app
```

In addition to the build tools, the GoogleTest framework is also installed.


_Egon Teiniker, 2025, GPL v3.0_
