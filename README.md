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

<ol>
<li>GET /api/users returns the collection of users in the database.</li>
<li>GET /api/users/:userId returns the information for the user identified by :userId. </li>
<li>GET /api/threads returns the collection of threads in the database.</li>
<li>GET /api/threads/:threadId returns the information for the thread identified by :threadId.</li>
<li>GET /api/posts/in-thread/:threadId returns the collection of post messages for the thread identified by :threadId.</li>
<li>GET /api/posts/by-user/:userId returns the collection of post messages for the user identified by :userId.</li>
<li>GET /api/ returns the message "API ready to receive requests."</li>
<li>GET / returns the message "Ready to receive requests."</li>
</ol>

Line 52: This line defines the port number on which the application is listening for requests.

<h2>Task 2.3: Running the application</h2>
In this task, you start the Node.js server and run the message board application. You then test some of its RESTful API methods. In the terminal tab, to start the Node.js server and the application, enter the following command:
       
       npm start

<img width="446" alt="image" src="https://github.com/user-attachments/assets/0d4bea4c-1294-447e-8236-a651db16f88e" />

The server is started, and the application's entry point, index.js, is started. The first time it is invoked, index.js creates two cluster threads—Leader and Worker—to process requests.
Next, you need to keep the current terminal session active and open a second terminal tab to test the application's RESTful API.

In the bottom pane, choose (+), and choose New Terminal to open a new terminal tab. You now have two terminals in which you can enter commands. In the right terminal tab, to retrieve the /api/users resource, enter the following command:

      curl localhost:3000/api/users
      
The RESTful invocation returns a JSON object containing the list of users in the message board database. 

<img width="955" alt="image" src="https://github.com/user-attachments/assets/b4bfaa9e-1332-4942-8054-5227de1b0677" />

Choose the left terminal tab. You see the message that is output from server.js indicating that it has processed a GET method request on the resource identified by the path /api/users. The request took 4 milliseconds to process. 

<img width="700" alt="image" src="https://github.com/user-attachments/assets/6d58cda0-82ab-4899-9193-8310d13301b3" />

Similarly, you retrieve information about threads, posts etc and observe the output and the messages in left terminal tab. 

Retrieve information about 4th user: In the right terminal tab, enter the following command:

      curl localhost:3000/api/users/4

<img width="627" alt="image" src="https://github.com/user-attachments/assets/9692a524-bde4-4cd4-89ce-5b8e590309ed" />

Retrieve threads: In the right terminal tab, enter the following command: 

      curl localhost:3000/api/threads

<img width="797" alt="image" src="https://github.com/user-attachments/assets/bdce18ec-e695-4fb5-8fdb-67c71f949729" />

Retrieve thread 1 : In the right terminal tab, enter the following command:

      curl localhost:3000/api/posts/in-thread/1
<img width="805" alt="image" src="https://github.com/user-attachments/assets/0417fbbf-5513-4ff8-9320-c46ff1f2146b" />

Now you stop the Node.js server. In the left terminal tab, press Ctrl+C to shut down the server process. You have validated that the application responds properly to RESTful GET requests and can now proceed to containerize it.

<img width="554" alt="image" src="https://github.com/user-attachments/assets/d72321a3-7bd1-4da7-bdba-08cd8d794904" />

<h2>Task 3: Containerizing the monolith for Amazon ECS</h2>

Containers wrap application code in a unit of deployment that captures a snapshot of the code and its dependencies. They can help ensure that applications deploy quickly, reliably, and consistently regardless of the deployment environment. In this task, you build a container image for the monolithic message board application and push it to Amazon Elastic Container Registry (Amazon ECR). These steps are in preparation for deploying the application to Amazon ECS.

Specifically, you perform the following steps:

✅ Prepare the application for Docker containerization.

✅ Provision a repository.
      
✅ Build and push the Docker image to the repository.

<h3>Task 3.1: Preparing the application for Docker containerization</h3>

To put the message board application into a Docker container, you need to make the following changes to the application:
    
•  Remove the use of the Node.js cluster feature, and convert the application to a single process design. 
      
With Docker containers, the goal is to run a single process for each container rather than a cluster of processes.

•  Create a Docker file for the application. This file is a build script that contains instructions on how to build a container image for the application.

The 2-containerized-monolith folder of your AWS Cloud9 environment includes a container-ready version of the application. In this task, you review the files so that you understand the changes that were made to prepare the application for containerization. In the left pane, expand the 2-containerized-monolith folder, and double-click package.json to open it in an editor tab. 

<img width="482" alt="image" src="https://github.com/user-attachments/assets/4db89db2-2e8f-48bd-894d-8c268c59211f" />

Notice in line 7 that the entry point into the application has been changed from index.js to server.js. The index.js file is no longer present in the application folder because it contains the initialization logic for the Node.js cluster feature, which you are no longer using. 

<img width="551" alt="image" src="https://github.com/user-attachments/assets/fb43bc24-3cb3-4e26-b102-78e189b59822" />

In the left pane, in the 2-containerized-monolith folder, double-click server.js to open it in an editor tab. The only difference between this object and the non-containerized version is the addition of line 54, which prints the message "Worker started" when the application is first started.

<img width="663" alt="image" src="https://github.com/user-attachments/assets/e8b14c14-2f1d-4372-a9d5-432dfabd14ad" />

In the left pane, in the 2-containerized-monolith folder, double-click Dockerfile to open it in an editor tab. This file contains the instructions on how to build the container image for the application.

<img width="594" alt="image" src="https://github.com/user-attachments/assets/06bf8847-97de-4924-a665-3dce9c83da41" />

Understand the following:
<ol>
<li>Line 1: The base image on which the container image is to be built is alpine-node, which is a Node.js image.
<li>Line 3: This line sets the working directory of the filesystem on the image to /srv.</li>
<li>Line 4: This line adds the contents of the 2-containerized-monolith folder (the application folder) to the current working directory of the file system of the image (set in the previous line).</li>
<li>Line 5: This line invokes the npm install command to install all of the application's library dependencies declared in the package.json file.</li>
<li>Line 7: This line informs Docker that the container listens on port 3000 at runtime.</li>
<li>Line 8: This line asks Docker to run the node server.js command, which starts the application when the image is started.</li>
</ol>

Now that you understand how the container image for the application will be built, you look at where to put the image once it is built.

<h3>Task 3.2: Provisioning the repository</h3>

Docker container images are intended to be stored in a repository for sharing, version control, and convenient management purposes. Developers can use the Amazon ECR service to store, manage, and deploy Docker container images. In addition, Amazon ECR is integrated with Amazon ECS, allowing Amazon ECS to pull container images directly for production deployments.
In this sub-task, you create a repository in Amazon ECR to house the container image for the message board application.

On the AWS Management Console, in the search box, enter and choose Elastic Container Registry. In the Create a repository section, choose Get Started.  If you are shown the options menu under the Repositories menu, choose hyperlink Private repositores

For the Repository name enter mb-repo
       
Note: Ensure that under Visibility settings, Private is chosen.

Choose Create repository.
       
A message is displayed at the top of the page indicating that the repository was successfully created.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/5740fe18-b7be-4284-a758-ed8cf896789b" />

<h2>Task 3.3: Building and pushing the Docker image</h2>

You are now ready to build the container image for the application and push it to the Amazon ECR repository that you just created.
One of the useful features of the Amazon ECR console is that it provides ready-to-use command templates to build and push an image to the new repository that you just created. 

You use these provided AWS CLI commands in the next steps. In the message window at the top of the page, choose View push commands. A pop-up window titled Push commands for mb-repo opens. This window lists four AWS CLI commands that are customized for the mb-repo and purposely built to do the following:

<ol>
<li>Authenticate your Docker client to your Amazon ECR registry.</li>
<li>Build your Docker image.</li>
<li>Tag your Docker image.</li>
<li>Push you Docker image to the repository.</li>
</ol>

Notice also that the pop-up window offers two versions of the commands: macOS/Linux and Windows. Make sure that the macOS/Linux tab is selected because you are going to run these commands in your AWS Cloud9 environment.

<img width="542" alt="image" src="https://github.com/user-attachments/assets/4b98efab-7d5a-4d1a-8eef-14eec2835529" />

First, you copy and run the command to log your Docker client in to your registry. In the pop-up window, for the first command, choose the Copy icon to copy it to the clipboard. The command looks like the following:
            
            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 371458831472.dkr.ecr.us-east-1.amazonaws.com

Switch to the Cloud9-IDE - AWS Cloud9 browser tab. Paste the copied command into the left terminal tab, and press Enter to run the command. If the command completes successfully, it returns the message "Login Succeeded." You can ignore the displayed warnings.

<img width="828" alt="image" src="https://github.com/user-attachments/assets/e3e3b7e4-bad3-47f8-bd38-8bff65aadb85" />

Next, you build the Docker image for your application. Note: When a specific terminal tab is not mentioned in an instruction step, use the left terminal tab.
In the terminal tab, to change the directory to the 2-containerized-monolith folder, enter the following command:

    cd ~/environment/2-containerized-monolith

Switch to the Elastic Container Registry browser tab. In the pop-up window, for the second command, choose the Copy icon to copy it to the clipboard. The command looks like the following: 
       
       docker build -t mb-repo .

Note: Make sure to include the period at the end of the command. Switch to the Cloud9-IDE - AWS Cloud9 browser tab. Paste the copied command into the terminal tab, and press Enter to run the command: The build command produces a lot of output as it runs the instructions contained in the application's Docker file. When it is finished, you see the message.. 

<img width="683" alt="image" src="https://github.com/user-attachments/assets/0fc9ed44-e6b9-4f06-be93-be668c7f1abf" />

Next, you tag the image with the repository URI so that it can be pushed to the repository. Switch to the Elastic Container Registry browser tab. In the pop-up window, for the third command, choose the Copy icon to copy it to the clipboard. The command looks like the following:
       
      docker tag mb-repo:latest 371458831472.dkr.ecr.us-east-1.amazonaws.com/mb-repo:latest

Switch to the Cloud9-IDE - AWS Cloud9 browser tab. Paste the copied command into the terminal tab, and press Enter to run the command: If the command completed successfully, it does not return anything

<img width="688" alt="image" src="https://github.com/user-attachments/assets/fade1ab7-9dbf-42f5-b3bf-545665e5375d" />

Finally, you push the container image to the application's repository. Switch to the Elastic Container Registry browser tab. In the pop-up window, for the fourth command, choose the Copy icon to copy it to the clipboard. The command looks like the following:

            docker push 371458831472.dkr.ecr.us-east-1.amazonaws.com/mb-repo:latest

<img width="734" alt="image" src="https://github.com/user-attachments/assets/24ab3678-f77f-47e4-bc03-8080fb4db36b" />

Switch to the Cloud9-IDE - AWS Cloud9 browser tab. Paste the copied command into the terminal tab, and press Enter to run the command: The command outputs several messages as each layer of the image is pushed to the repository. Next, you verify that the image was successfully uploaded. Next, you verify that the image was successfully uploaded. Switch to the Elastic Container Registry browser tab. In the Push commands for mb-repo pop-up window, choose Close. In the Private registry > Repositories list, choose mb-repo. In the images list, you see the container image that you pushed, which is identified by the latest tag.

<img width="850" alt="image" src="https://github.com/user-attachments/assets/ed6e0a55-8706-46f7-b055-b474ee17b9a0" />

Next, you record the image URI. In the Images list, next to Copy URI, choose the Copy icon. Paste the value in a text editor. You use it in a subsequent step.
       
Great job! You have successfully created a container image for the message board application and pushed it to an Amazon ECR repository.

<h2>Task 4: Deploying the monolith to Amazon ECS</h2>
In this task, you deploy the containerized monolithic application to an Amazon ECS runtime environment. Specifically, you use Amazon ECS to create a managed cluster of Amazon Elastic Compute Cloud (Amazon EC2) instances on which to deploy your application container image. The cluster is configured as the target group of an Application Load Balancer to provide failover and scalability. The following diagram shows the deployment architecture of the containerized monolithic application. It also displays the resources that you create in this task.

<img width="699" alt="image" src="https://github.com/user-attachments/assets/cfcb7917-02cc-4c0f-9151-002d858faee6" />

In this task, you perform the following steps:
<ol>
<li>Create an Amazon ECS cluster.</li>
<li>Create a task definition for the application container image.</li>
<li>Deploy the monolithic application as an Amazon ECS service.</li>
<li>Test the containerized monolithic application.</li>
</ol>

<h2>Task 4.1: Creating an Amazon ECS cluster</h2>
An Amazon ECS cluster is a logical grouping of EC2 instances on which you can run tasks or services representing your containerized application. In this sub-task, you create an Amazon ECS cluster by using the Amazon ECS console. The console's cluster creation wizard facilitates the creation of all of the infrastructure components needed to create the Amazon ECS cluster environment, including the virtual private cloud (VPC), subnets, security groups, internet gateway, and AWS Identity and Access Management (IAM) roles. On the AWS Management Console, in the search box, enter and select Elastic Container Service 

<img width="935" alt="image" src="https://github.com/user-attachments/assets/d799e780-7495-49ca-9a4b-a5ae0bcc343a" />

 From the left menu, choose Clusters. Choose Create cluster, and configure the following options:
<ol>
<li>In the Cluster configuration section, for the Cluster name, enter mb-ecs-cluster</li>
<li>In the Infrastructure section, select Amazon EC2 instances, and configure the following options:</li>
<li>Leave the Auto Scaling group (ASG) settings at defaults.</li>
</ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/83fb75ab-2d23-4617-b3e9-f605fa615d4d" />

 ▪  For Operating system/Architecture, choose Amazon Linux 2023.
              
 ▪  For EC2 instance type, choose t2.medium.

 <img width="959" alt="image" src="https://github.com/user-attachments/assets/4c50c076-8350-4c08-9348-1dffb9fe890c" />

 ▪  For Desired capacity, for Minimum and Maximum, enter 2

 <img width="949" alt="image" src="https://github.com/user-attachments/assets/3441a52a-cf19-4486-b00b-59701f1c7ae4" />

 ◦ In the Network settings for Amazon EC2 instances, configure the following options:
          
 ▪  VPC: choose Lab VPC.
              
 ▪  Subnets: Reconfirm that, all public subnets chosen.

 <img width="959" alt="image" src="https://github.com/user-attachments/assets/c4254260-c6ee-4a45-ba6c-9b250150082c" />

▪ Security group: Choose Use an existing security group.
              
▪ Security group name dropdown list, select the security group that has ECSSG in the name.
              
▪ Clear the default security group.         

 <img width="953" alt="image" src="https://github.com/user-attachments/assets/0530040b-3041-469a-be53-7254431e1c58" />

 Choose Create.

 It takes a few minutes for the cluster to be created. Choose the cluster you created, e.g. mb-ecs-cluster. The details page for mb-ecs-cluster is displayed. Notice that the Status shows a value of Active.

 <img width="919" alt="image" src="https://github.com/user-attachments/assets/2165fee0-401b-4344-8b06-07e46bcc95d4" />

 <img width="953" alt="image" src="https://github.com/user-attachments/assets/5a872262-1792-4374-a9e0-9106a4862fec" />

 Choose the Infrastructure tab.

 The Container instances pane shows that two EC2 instances for the cluster were created.

 <img width="900" alt="image" src="https://github.com/user-attachments/assets/74dadb88-7dff-42c0-b526-8917a2707766" />

 <h3>Task 4.2: Creating a task definition for the application container image</h3>
A task definition is a list of configuration settings for how to run a Docker container on Amazon ECS. The following are examples of the information that a task definition provides to Amazon ECS:
<ol>
<li>Which container image to run</li>
<li>How much CPU and memory the container needs</li>
<li>On which ports the container listens to traffic</li>
</ol>

In this sub-task, you create a task definition for the container image of the message board application. On the Amazon ECS console, from the left menu, choose Task definitions.
Choose Create new task definition, and configure the following options:

• In the Task definition configuration section, for Task definition family, enter mb-task
    
• In the Infrastructure requirements, select Amazon EC2 instances, and clear AWS Fargate.

<img width="825" alt="image" src="https://github.com/user-attachments/assets/816ea704-a961-4f76-be8c-e840d9cb64b7" />

◦  For the Task size, choose CPU: .5vCPU, Memory: 1GB

◦  For Task execution role, choose Create new role.

<img width="839" alt="image" src="https://github.com/user-attachments/assets/f497ef33-365c-4ff8-b769-aeee74fff3e4" />

 ◦ In the Container - 1 section, configure the following options:
            
▪ For Container details, for Name, enter mb-container
           
▪ For Image URI, paste the URI of the users container image that you copied to a text editor earlier.
           
▪ For Port mappings, for Container port, enter 3000. This option specifies the port on which the container receives requests.

<img width="873" alt="image" src="https://github.com/user-attachments/assets/dd347024-4d94-4c8f-a8a4-627d42ff1a1d" />

Choose create
A message is displayed at the top that says, "Task definition successfully created." You now have a task definition that tells Amazon ECS how to deploy your application container across the cluster.

<img width="842" alt="image" src="https://github.com/user-attachments/assets/a1ddd904-8e82-48a0-98a5-b1fbdf26fc52" />

<h3>Task 4.3: Creating and deploying the service</h3>
All of the required Amazon ECS infrastructure components are created, and you can now deploy the containerized monolithic application to the cluster as an Amazon ECS service.
You can use Amazon ECS to run and maintain a specified number of instances of a task definition simultaneously in an Amazon ECS cluster. If one of the tasks fails or stops for any reason, the Amazon ECS service scheduler launches another instance of the task definition to replace it and maintains the desired number of tasks specified in the service.
In this sub-task, you use the Amazon ECS console to create an Amazon ECS service for the message board application's task definition. On the same screen where you created mb-task, choose Deploy, then choose Create service. In the Environment section, configure the following options:

• For Compute options, choose Launch type.
    
• For Launch type, choose EC2.

<img width="781" alt="image" src="https://github.com/user-attachments/assets/c454e194-c1c1-4988-a0cb-15aa28e706fd" />















 


 





























































