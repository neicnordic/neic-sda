# Local development and testing guide

## Guide summary

This guide provides a brief introduction on how to locally install and run the Sensitive Data Archive (SDA) components on your development system, and run all the tests locally. 

This guide should get you started by setting up your environment to build and deploy the services of SDA, run all tests in the code base, and run several development related "shortcut" actions from the commandline using custom development helper scripts. 

In addition the guide includes a few tips and tricks on how to explore the services running locally after you've deployed them, to learn more about how they operate together and find more documentation.


## Local security / zone considerations

Normally a SDA deployment will consist of at least two different security zones, separating internet facing services from the services accessing the encrypted file archive with more sensitive config information. In this guide and the development deployment tools, these security zones are not implemented.

Hence your testing, staging and production deployments will most likely differ in deployment strategy and end-2-end testing approaches compared to what is documented in this guide.


## SDA local development and testing helpers

The SDA code itself contains some very useful development helpers, that let's you very easily:

 - Check that you have all required libraries, compilers, docker tools, and if not tries to install them
 - Build the services locally
 - Lint the code
 - Run the tests for each service
 - Run integration tests for services together by deploying the service containers
 - ...and some more useful actions

Once you have cloned the [neicnordic/sensitive-data-archive](https://github.com/neicnordic/sensitive-data-archive/) to your development system, you can start to
 explore these tools as explained in [this page](sda-dev-test-doc.md).
 
## What's next, once everything is up and running?

Once you've been able to automagically deploy and run all the tests and integration tests, thinking *"that was too easy, what really happened?"*, you could go in multiple direction from here to learn more:

 - List all the services running with ```$ docker ps``` 
 - Peek into logs of services with ```$ docker logs container-id-or-name```
 - Read more about how each service functions in the SERVICES section of the handbook
 - Read more about the dataflow and interactions between services in the COMMUNICATION section of the handbook




