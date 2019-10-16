# Challenge

## Introduction

Aim is to draw / show how you would build a production environment for a service like HaveIBeenPwnd.

Some details:

* It does not have uniform access, with peaks at expected heights (Addition of a new  dataleak) or unexpected (Mentioned in a news story);
* Thousands of millions of data;
* There are workers to process files of new leaks;
* Include monitoring and logging in the design;

### The software

The software is built from several services, some services communicate with each other, some depend on an SQL database others on queuing message brokers or in memory databases,  tßhe components are as follows:

* **Frontend** is a single-page application that includes an API client that issues requests to the **Webserver** from the user's browser.
* **Webserver** provides a web API that **Frontend** uses. It needs an SQL database and sends job requests to the **Controller**.
* **Controller** manages jobs that are currently running, and starts jobs when it receives a request from a **Webserver** and sends tasks to be done by **Workers**. Controllers use the database to save state.
* **Scheduler** receives tasks from **Controller** and sends them to a **Dispatcher**.
* **Dispatcher** provides an endpoint that one or more **Worker** connects to to receive Tasks. **Dispatchers** send task results back to **Controllers**
* **Workers** connect to a **Dispatcher** and process tasks sent to them.

