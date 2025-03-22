<h1>Breaking a Monolithic Node.js Application into Microservices</h1>
<h2>Overview</h2>

Traditional monolithic architectures are hard to scale. As an application's code base grows, it becomes complex to update and maintain. Introducing new features, languages, frameworks, and technologies becomes challenging, which limits innovation and new ideas.

Within a microservices architecture, each application component runs as its own service and communicates with other services through a well-defined API. 
Microservices are built around business capabilities, and each service performs a single function. Microservices can be written by using different frameworks and programming languages, and you can deploy them as individual services, as a single service, or as a group of services.

In this lab, I migrated a monolithic application that runs in a standard Node.js server to a containerized Docker environment. I then refactor the application into microservices and deploy it to a containerized environment orchestrated by Amazon Elastic Container Service (Amazon ECS). 

The Node.js application implements the functions of a message board where users can create topic threads and post messages on each thread. The following diagram illustrates the evolution of the application's design as it moves from monolithic to microservices based.
