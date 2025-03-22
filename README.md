<h1>Breaking a Monolithic Node.js Application into Microservices</h1>
<h2>Overview</h2>

Traditional monolithic architectures are hard to scale. As an application's code base grows, it becomes complex to update and maintain. Introducing new features, languages, frameworks, and technologies becomes challenging, which limits innovation and new ideas.

Within a microservices architecture, each application component runs as its own service and communicates with other services through a well-defined API. 
Microservices are built around business capabilities, and each service performs a single function. Microservices can be written by using different frameworks and programming languages, and you can deploy them as individual services, as a single service, or as a group of services.

In this lab, I migrated a monolithic application that runs in a standard Node.js server to a containerized Docker environment. I then refactor the application into microservices and deploy it to a containerized environment orchestrated by Amazon Elastic Container Service (Amazon ECS). 

The Node.js application implements the functions of a message board where users can create topic threads and post messages on each thread. The following diagram illustrates the evolution of the application's design as it moves from monolithic to microservices based.

<img width="710" alt="image" src="https://github.com/user-attachments/assets/b75a0c02-e6b3-4978-a10b-1a0adee3853b" />

The diagram highlights the following differences between the monolithic approach and the microservices design:
<ol>
<li>In a monolithic design, all of the functions of the Node.js application are packaged and run as a single service. </li>
<li>If one function fails, the entire application fails. Likewise, if one application function experiences a spike in demand, all functions in the service must be scaled together.</li>
<li>In a microservices architecture, each function of the Node.js application runs as a separate service. The services can scale and be updated independently of each other.</li>
</ol>

<h3>What I learned:</h3>
Throughout the lab, I:

✅Migrated a monolithic Node.js application to run in a Docker container.
      
✅Refactor a Node.js application from a monolithic design to a microservices architecture.
      
✅Deploy a containerized Node.js microservices application to Amazon ECS.

<h2>Task 1: Preparing the development environment</h2>

An AWS Cloud9 environment has been created for me as part of the creation process for the lab environment. AWS Cloud9 is a cloud-based integrated development environment (IDE) that you can use to write, run, and debug code with just a browser. It comes pre-packaged with essential tools for popular programming languages and provides access to the AWS Command Line Interface (AWS CLI) in a terminal session tab. 

AWS Cloud9 environment has access to the all of the Amazon Web Services (AWS) resources authorized for the user ID with which you signed in to the AWS Management Console.
To set up my development environment, I open the AWS Cloud9 IDE, and download and extract the required lab files.

<img width="875" alt="image" src="https://github.com/user-attachments/assets/30df8b68-a6f5-49c2-aa77-652d44ac8762" />

Next, download and extract the required lab files. In the bottom pane of the IDE, in the terminal tab labeled bash - "ip-nnn-nnn-nnn-nnn", run the following command:

    curl -s https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-2-91555/16-lab-mod13-guided-1-ECS/s3/lab-files-ms-node-js.tar.gz | tar -zxv

This command retrieves a compressed archive file containing the lab files and extracts its contents in the AWS Cloud9 ~/environment folder. The downloaded and extracted files are visible in the left pane. 

<img width="365" alt="image" src="https://github.com/user-attachments/assets/119f9cc2-6875-4ba7-bdbb-d5676066e38c" />

In the left pane, you should see the following folders:
       
✅ 1-no-container: This folder contains the files related to the monolithic implementation of the application intended to run directly on a Node.js server.
       
✅ 2-containerized-monolith: This folder contains the files related to the monolithic implementation of the application intended to run in a Docker-containerized environment orchestrated by Amazon ECS.

✅ 3-containerized-microservices: This folder contains the files related to the microservices implementation of the application intended to run in a Docker-containerized environment orchestrated by Amazon ECS.

You use the AWS Cloud9 IDE tab frequently throughout this lab, so keep this tab open.

 In the left pane, you should see the following folders:
       
✅ 1-no-container: This folder contains the files related to the monolithic implementation of the application intended to run directly on a Node.js server.
       
✅ 2-containerized-monolith: This folder contains the files related to the monolithic implementation of the application intended to run in a Docker-containerized environment orchestrated by Amazon ECS.
       
✅ 3-containerized-microservices: This folder contains the files related to the microservices implementation of the application intended to run in a Docker-containerized environment orchestrated by Amazon ECS.
       
  You use the AWS Cloud9 IDE tab frequently throughout this lab, so keep this tab open.

  <h2>Task 2: Running the application on a basic Node.js server</h2>
The base Node.js application is a monolithic service that has been designed to run directly on a server without a container. 

In this task, you deploy the application to the Node.js server that is installed on the instance running your AWS Cloud9 environment. You then test the application by using the command line terminal to invoke its Representational State Transfer (RESTful) API methods. The following diagram illustrates the deployment architecture and request flow.

<img width="668" alt="image" src="https://github.com/user-attachments/assets/994c9b80-d80e-40dd-8373-68976a66ebbb" />

The monolithic implementation of the application uses the Node.js cluster functionality to spawn one worker process for each CPU core. The processes share a single port and are invoked in a round-robin fashion by the load balancer built into Node.js. This feature increases scalability on servers that have multiple CPU cores.
In this task, you do the following:

 • Install the Node.js modules required by the application.
      
 • Review the application design and code.
      
`• Run the application.

<h3>Task 2.1: Installing the required Node.js modules</h3>
The message board application uses two modules from the Node.js koa framework in its implementation: koa and koa-router. 

Koa.js is a widely used Node.js web application framework that facilitates the building of asynchronous server-side JavaScript applications.

       In the AWS Cloud9 IDE terminal, to install the koa and koa-router modules, enter the following commands:
       cd ~/environment/1-no-container
       npm install koa
       npm install koa-router

<img width="785" alt="image" src="https://github.com/user-attachments/assets/016f2104-604d-43ca-8dec-e93f32b1be1c" />

The modules are downloaded and installed in the 1-no-container/node_modules folder of the AWS Cloud9 ~/environment folder. 

<h2>Task 2.2: Reviewing the application design and code</h2>
The components that implement the monolithic message board application are contained in the 1-no-container folder. Review them to gain an understanding of the application design and code.
In the AWS Cloud9 IDE terminal, in the left pane, expand the 1-no-container folder. The components of the application consist of the following:
         
<h4>node_modules: </h4>
This folder was created when you installed the required JavaScript modules in the previous sub-task and contains their source code.

<img width="167" alt="image" src="https://github.com/user-attachments/assets/e2bbc0e1-33d8-41f6-baf0-09c88c41a0c7" />

✅ db.json: This item is a JSON object that simulates the message board database. It contains attributes representing users, threads, and posts with corresponding sample values.
    
            {
      "users": [
        {
          "id": 1,
          "username": "marceline",
          "name": "Marceline Singer",
          "bio": "Cyclist, musician"
        },
        {
          "id": 2,
          "username": "finn",
          "name": "Finn Alberts",
          "bio": "Adventurer and hero, defender of good"
        },
        {
          "id": 3,
          "username": "pb",
          "name": "Paul Barium",
          "bio": "Scientist, cake lover"
        },
        {
          "id": 4,
          "username": "jake",
          "name": "Jake Storm",
          "bio": "Soccer fan, sky diver"
        }
      ],
    
      "threads": [
        {
          "id": 1,
          "title": "Did you see the Brazil game?",
          "createdBy": 4
        },
        {
          "id": 2,
          "title": "New French bakery opening in the neighborhood tomorrow",
          "createdBy": 3
        },
        {
          "id": 3,
          "title": "In search of a new guitar",
          "createdBy": 1
        }
      ],
    
      "posts": [
        {
          "thread": 1,
          "text": "That last goal was awesome!",
          "user": 4
        },
        {
          "thread": 1,
          "text": "Yes, the way the ball swerved... What talent!",
          "user": 2
        },
        {
          "thread": 2,
          "text": "I have to try their tarts!",
          "user": 3
        },
        {
          "thread": 2,
          "text": "I'm planning to stop by in the morning to try their croissants.",
          "user": 2
        },
        {
          "thread": 2,
          "text": "I could go for a chocolate eclair!",
          "user": 1
        },
        {
          "thread": 3,
          "text": "I need a new acoustic guitar at a good price.",
          "user": 1
        }
      ]
    }

✅ index.js: This item is the JavaScript program that is the application's entry point.

      const cluster = require('cluster');
      const http = require('http');
      const numCPUs = require('os').cpus().length;
      
      if (cluster.isMaster) {
        console.log(`Leader ${process.pid} is running`);
      
        // Fork workers.
        for (let i = 0; i < numCPUs; i++) {
          cluster.fork();
        }
      
        cluster.on('exit', (worker, code, signal) => {
          console.log(`worker ${worker.process.pid} died`);
        });
      } else {
        require('./server.js');
      
        console.log(`Worker ${process.pid} started`);
      }

✅ package-lock.json: This item is a JSON object that was automatically generated when you installed the required JavaScript modules in the node_modules folder. It is used by the npm installation utility to keep track of modifications made to the folder. 
<img width="959" alt="image" src="https://github.com/user-attachments/assets/57979e9d-c0a5-4c6d-abf3-054023a8979b" />

✅ package.json: This item is a JSON object describing the application, its entry point, and its dependencies.

<img width="482" alt="image" src="https://github.com/user-attachments/assets/e510bf90-9d6a-4e81-8ac2-e0ddb2051bf1" />

✅ server.js: This item is the JavaScript program that defines the application's RESTful API methods and implements their respective handlers.

            const app = require('koa')();
      const router = require('koa-router')();
      const db = require('./db.json');
      
      // Log requests
      app.use(function *(next){
        const start = new Date;
        yield next;
        const ms = new Date - start;
        console.log('%s %s - %s', this.method, this.url, ms);
      });
      
      router.get('/api/users', function *(next) {
        this.body = db.users;
      });
      
      router.get('/api/users/:userId', function *(next) {
        const id = parseInt(this.params.userId);
        this.body = db.users.find((user) => user.id == id);
      });
      
      router.get('/api/threads', function *() {
        this.body = db.threads;
      });
      
      router.get('/api/threads/:threadId', function *() {
        const id = parseInt(this.params.threadId);
        this.body = db.threads.find((thread) => thread.id == id);
      });
      
      router.get('/api/posts/in-thread/:threadId', function *() {
        const id = parseInt(this.params.threadId);
        this.body = db.posts.filter((post) => post.thread == id);
      });
      
      router.get('/api/posts/by-user/:userId', function *() {
        const id = parseInt(this.params.userId);
        this.body = db.posts.filter((post) => post.user == id);
      });
      
      router.get('/api/', function *() {
        this.body = "API ready to receive requests";
      });
      
      router.get('/', function *() {
        this.body = "Ready to receive requests";
      });
      
      app.use(router.routes());
      app.use(router.allowedMethods());
      
      app.listen(3000);
      
      Next, you examine the package.json object. 

<img width="624" alt="image" src="https://github.com/user-attachments/assets/f26aaa96-491f-4184-a54f-6b4fad3c9b7d" />

Next, you examine the package.json object. In the left pane, double-click package.json to open it in an editor tab. Notice the following attributes of the JSON object:

✅ Lines 2–5: The dependencies attribute defines the JavaScript module dependencies for the application. Note that the koa and koa-router modules that you installed in the previous sub-task are listed here.
         
✅ Lines 6–8: The scripts attribute indicates that the index.js program is the entry point to the application.

<img width="482" alt="image" src="https://github.com/user-attachments/assets/0b3124a7-42da-461b-82b8-322f59b8a838" />

Next, you examine the db.json object. In the left pane, double-click db.json to open it in an editor tab. Notice the following attributes of the JSON object: 

Lines 2–27: These lines define a users attribute that represents the registered users of the message board. The attribute value is a list of four sample users with the following names: Marcerline Singer, Finn Alberts, Paul Barium, and Jake Storm.

<img width="482" alt="image" src="https://github.com/user-attachments/assets/8b729a27-bd6c-4797-87ea-9fc0dd6e3cdc" />

Lines 29–45: These lines define a threads attribute that represents the current active threads on the message board. The attribute value is a list of three sample threads with the following titles:
<ol>
<li>Did you see the Brazil game?</li>
<li>New French bakery opening in the neighborhood tomorrow</li>
<li>In search of a new guitar</li>
</ol>

<img width="547" alt="image" src="https://github.com/user-attachments/assets/5556fb57-232f-4464-8d1e-9d6273b3b7fe" />

Lines 47–78: These lines define a posts attribute that represents the posted messages on the active threads. The attribute value is a list of six sample message posts.

<img width="626" alt="image" src="https://github.com/user-attachments/assets/cc8cbb20-6b9c-40b8-9b1d-a0c840232a08" />

Next, you review the code for the index.js object. In the left pane, double-click index.js to open it in an editor tab. Notice the following attributes:
Lines 1–3: These lines import the JavaScript modules that the program requires, specifically, cluster, http, and OS.

<img width="461" alt="image" src="https://github.com/user-attachments/assets/549ff463-41c1-4304-9f8e-082588d53345" />

Line 3: This line uses the os JavaScript module to check the number of CPU cores available on the server.

<img width="635" alt="image" src="https://github.com/user-attachments/assets/d1e3d1c1-6cee-4ba6-a63d-81e0752b1373" />

Lines 5 through 15: These lines are run the first time that the program is invoked (when the application is started). They create one leader thread for the cluster and one worker thread for each CPU core available on the server.

<img width="561" alt="image" src="https://github.com/user-attachments/assets/2e68127f-f951-4c6c-a270-9060503e2534" />

Lines 16–19: These lines handle each request to the application by invoking the server.js program in the current worker thread.

<img width="504" alt="image" src="https://github.com/user-attachments/assets/61fc1e42-e195-41df-89e4-5b876b32ffcb" />

Lastly, you review the code for the server.js object. In the left pane, double-click server.js to open it in an editor tab. Use the comments provided in the code to facilitate your understanding of the logic. In particular, notice the following: 
       
Line 3: This line imports the JSON object (db.json) that simulates the database.

<img width="670" alt="image" src="https://github.com/user-attachments/assets/5b137244-d2d8-47a6-afa6-c09dce1b4f1c" />

Lines 6–11: These lines define a generator function that gets run for every request. Its purpose is to print a line containing the HTTP method, resource path URL, and elapsed time for each request that is processed.

<img width="577" alt="image" src="https://github.com/user-attachments/assets/059bc13f-e680-4eaf-b885-6683ae694f74" />

Lines 13–47: These lines define the application's RESTful API methods and their implementation. Specifically, the application can respond to the following RESTful calls:

<img width="553" alt="image" src="https://github.com/user-attachments/assets/4313986a-c95c-4c0b-88de-6f176023f9dd" />


























