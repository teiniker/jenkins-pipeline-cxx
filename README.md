# jenkins-pipeline-cxx


## Docker Agent

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


## Jenkins Pipeline

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
}
```

_Egon Teiniker, 2025, GPL v3.0_