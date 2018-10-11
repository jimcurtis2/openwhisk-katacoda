# Introduction to Serverless Concepts and Knative Design

During this step of the course, we will be discussing some basic concepts of Serverless and cover the design of Knative.

## Serverless Concepts

The term Serverless is often used interchangeably with the term FaaS (Function-as-a-Service).  Serverless platforms provide APIs
that allow users to run code functions (also called actions) and return the results of each function.  Serverless platforms provide
HTTPS endpoints to allow the developer to retrieve function results.  These endpoints can be used as inputs for other functions, 
thereby providing a sequence (or chaining) of related functions.

On most Serverless platforms, the user deploys (or creates) the functions first before executing them.  The Serverless platform 
then has all the necessary code to execute the functions when it is told to.  The execution of a Serverless function can be invoked
manually by the user via a command or it may be triggered by an event source that is configured to activate the function in
response to events such as cron job alarms, file upload, or any of the many events that can be chosen.

## Knative Design

## I need some good text here describing the design along with any png/gif/jpeg that illustrates this design.  If you can
## point me at something, please e-mail me or catch me on slack.
