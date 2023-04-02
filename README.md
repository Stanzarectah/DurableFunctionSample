## Webhooks and Durable Functions

## The problem
Currently all HTTP triggered actions in Power Automate and HTTP triggered Azure functions have a 230 timeout limit. Typically, some scenarios involve the execution of many processing steps or the updating of thousands of records which causes the Azure function to run way past the limit,

## Solution
The best option is to use a **Webhook** action in Power Automate that subcribes to a **Durable function** which provides a callback url.
During this time the Webhook waits until it receives a pot back from the durable function
The logic that contains the long running process is implemented within the **Activity** function of the Durable function and then the callback url is "called back" once the long running logic has completed.

Refer to articles about Durable functions [here](https://medium.com/asos-techblog/getting-started-with-durable-functions-1382adf1d6ac). They are ideally used to implement long running processes as they provide the polling and queue archtiecture required by long running processes out of the box. 

## Setup
 
1. Create a Durable function based on the code provided in the code provided. You will implement your long running login within the Activity function.
2. Publish the function app and get the key for the **HTTP trigger** of the Durable function in the Azure portal
    ![httptrigger](https://user-images.githubusercontent.com/17443786/229332516-2e569909-3503-4bd5-86d5-f1351f5e070a.JPG)

    ![image](https://user-images.githubusercontent.com/17443786/229332642-11487274-d44a-4004-b2c4-f8ea8e41fbc8.png)
3. In Power Automate create a WebHook action and:

     - Paste the url of the Durable function with the function key in the previous step in the **Subscribe - URI** field
     - Specify the callback url request body value from the dynamic content selector in the **Subscribe - Body** field
     
     ![image](https://user-images.githubusercontent.com/17443786/229332891-da36495c-0083-4801-bdbb-fb42687c6838.png)

## How it works
 ![image](https://user-images.githubusercontent.com/17443786/229335123-b021876b-084c-48b4-8f89-805fe13542e6.png)

1. HTTP trigger of the Durable function receives the callback url when it gets triggered from the WEbhook in Power Automate and then starts the Orchestrator

    ![image](https://user-images.githubusercontent.com/17443786/229333652-b6908cb2-f3f0-4860-92b7-6f4c306545b8.png)

3. The Orchestrator receives the callback url form the HTTP trigger and passes it to the Activity function when calling it 
    ![image](https://user-images.githubusercontent.com/17443786/229333712-f5b4a1eb-67ae-4489-b986-14800bf29823.png)

4. The Activity function receives the callback url from Orchestrator, executes the long running code and makes a POST request to the callback url to indicate that process has completed in turn notifying the webook in Power Automate.

     ![image](https://user-images.githubusercontent.com/17443786/229335582-225dbb88-f7ae-40ba-b7df-1af1bbfdbf11.png)

     ![image](https://user-images.githubusercontent.com/17443786/229336037-a0b160f4-b663-431e-bf14-5cdf6ef28227.png)

6. The rest of the flow continues to execute
