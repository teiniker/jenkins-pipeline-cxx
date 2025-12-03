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
            sh '''
                mkdir -p artifacts
                cp file_service/build/test/test artifacts/ 
            '''
            archiveArtifacts artifacts: 'artifacts/*', fingerprint: true
        }
    }    
}
```

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


## Jenkins Job Configuration

We need the following settings for a new Jenkins job:

* New Item
    - Enter an item Name "C++ Pipeline"
	- Pipeline
    - Description (optional)
    - Build Triggers 
	    - Poll SCM: `H/1 * * * *`
    - Pipeline / Definition: 
      - Pipeleine: Pipeline Script from SCM: 
        - SCM: Git 
        - Repository URL: `https://github.com/teiniker/jenkins-pipeline-cxx.git` 
        - Branches to build: `*/main` 
      - Script Path: Jenkinsfile

Save these settings.

_Egon Teiniker, 2025, GPL v3.0_