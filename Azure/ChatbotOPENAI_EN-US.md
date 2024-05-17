Here is the English translation of your Markdown:

# How to create a chatbot for your own files using Azure OpenAI in 2024

This is a new process, and it uses Azure's "Preview" functions, so there may be some differences when you create yours. But it's still functional.

This solution can also be created directly using OPENAI's APIs, without the need for an Azure subscription.

But I confess that Azure greatly facilitates the service. In addition to already delivering a chatbot application that authenticates in your AD (or your company's) and is ready to be customized. Plus, if you're just starting out, you can use some of your free credits without weighing heavily on your pocket.

The step-by-step will be very detailed, those who already know the ropes can skip some parts.


## Step 1: Authentication and creation of the OpenAI resource

First, enter the Azure Portal. This is the same for everyone.

![Screenshot of 2024-04-16 18-20-03](https://github.com/pedropberger/tutorials/assets/98188778/5622f5b4-59b5-4f05-a4c7-fa3d749ea1da)

Log in with your credentials (personal or your company's)

![Screenshot of 2024-04-16 18-20-16](https://github.com/pedropberger/tutorials/assets/98188778/bb284728-0370-4625-80a2-8d40ce0eaa1a)

Usually, you will see this bar. Click on the "+" and look for the resource, or just search for "OpenAI" in the top bar.

![Screenshot of 2024-04-16 18-21-32](https://github.com/pedropberger/tutorials/assets/98188778/13c84f15-cb96-4631-817c-3e667ea13768)

I clicked on the "Azure OpenAI" icon to enter the resource options.

![Screenshot of 2024-04-16 18-22-30](https://github.com/pedropberger/tutorials/assets/98188778/a506b056-e261-468c-b569-42c98c74b329)

On this screen, you manage all options about the resource, including options about deployments of models that you will make in the future (GPT 3.5 turbo, GPT 4, DALL-e, etc).

You can create one or more OpenAI resources. The operation of creating the resource does not have a cost, and by my tests. So you can create an OpenAI resource for each application and chatbot, or you can use just one for all your solutions. This will depend on the number of services and the way your team/organization works. To create the resource, just click on "Create" in the upper left corner.

![Screenshot of 2024-04-16 18-23-03](https://github.com/pedropberger/tutorials/assets/98188778/9e8361d2-1c63-4604-bc5a-ebbbe923b840)

You will arrive at the screen below. Here begin the important definitions that will follow forever with the resource. Some can be modified after the Deploy, others cannot. But you don't need to worry too much about making a mistake if it's your first time, if it goes bad just delete everything and create again.

Select your Subscription and your Resource group. If you can't, look for someone who can give you permission. Sometimes it's not enough to have access to the Resource Group, you need permission to create the resource within that resource group.

Select the region. Important tip about the region: You will need to create other resources in this tutorial or in your life, such as a Storage Account and a Search Service, it is interesting that they are in the same network as the OpenAI resource, and for that sometimes it is necessary that they are created in the same region, and not all regions have all resources. So I advise you to choose a region and create all your resources and network in it. And choose a region that has all resources. Sorry if it sounded verbose.

Some regions (like Brazil-South, for example) still do not have certain models that will be necessary in this tutorial.

![Screenshot of 2024-04-16 18-26-01](https://github.com/pedropberger/tutorials/assets/98188778/2f459747-93fa-455d-a8f9-5bf81df954ca)

Here in the network I recommend you can create or use a virtual network and subnet for your security. Remembering that all resources will need to be created in the same network.

You can also use the first option and release to all networks, including the internet, to access the resource. Is it risky? it is. But for someone to consume your resource will need your Keys that will be created in the future. So in my opinion of non-expert in IT security, this is a controlled risk, since it will be a development environment.

Are you going to do a production deploy? use a closed network and consult the network administrator of your team.

![Screenshot of 2024-04-16 18-26-34](https://github.com/pedropberger/tutorials/assets/98188778/2f28cb2d-4f2d-4f23-b988-f6255bc58439)

TAGs!

Here you put the TAGs

What are tags?
- Tags are key-value pairs that you can associate with resources in Azure.
- Each resource or resource group can have up to 15 tags.
- Example of tag format: Department: IT, Status: Test, CostCenter: HR.

Benefits of using tags:
- Organization: Helps to organize and categorize resources.
- Management: Facilitates the search and filtering of resources based on specific criteria.
- Cost tracking: Allows tracking the costs associated with specific projects or departments.

See how your team an organized way to use them. It is optional, each team has its policies and rest assured, you can change/add new ones later.

![Screenshot of 2024-04-16 18-27-40](https://github.com/pedropberger/tutorials/assets/98188778/a2ab9113-9f34-4057-9a8d-0c2b36ec788a)

On the final screen, you review if there is any pending and just go for it, CREATE.

![Screenshot of 2024-04-16 18-29-21](https://github.com/pedropberger/tutorials/assets/98188778/d6369ef6-11e3-4855-9dda-ab08d689be7d)

On the right side will appear the screen warning that the resource is being created, just wait.

![Screenshot of 2024-04-16 18-29-35](https://github.com/pedropberger/tutorials/assets/98188778/d4c5019f-06b8-4f8b-b962-314efb23dae9)

.. and done, resource working. Now it's just joy.

![Screenshot of 2024-04-16 18-30-57](https://github.com/pedropberger/tutorials/assets/98188778/e4cc0205-4593-44b6-8f09-638283c93222)


# Step 2: Azure OpenAI Studio

Here begins the development of the chatbot. For this, Azure now provides Azure OpenAI Studio, which brings the following benefits:

- Azure OpenAI Studio unifies capabilities from different Azure AI services.
- Simplified support for models like ChatGPT, DALL-E, Ada, and others in the Azure OpenAI service.
- Various options for security, compliance, and pricing.
- It allows you to create, evaluate, and deploy AI-generated solutions.
- Whether for chatbots, virtual assistants, or other applications, the studio offers an integrated experience.

There you control which models, how you will use them, and even receive examples of how to consult the API (but that is a subject for another tutorial).

Let's go. To start, look for your newly created resource and click on it:

![Screenshot of 2024-04-16 18-31-42](https://github.com/pedropberger/tutorials/assets/98188778/fded6ea5-72cb-4e84-b250-c71620bf103e)

You will go to a screen similar to the screen below. Here you control everything that is generated by your resource.

![Screenshot of 2024-04-16 18-32-44](https://github.com/pedropberger/tutorials/assets/98188778/f0642943-88fa-421c-a7e5-e580b4061cfc)

In particular, the Develop tab brings the usage keys, region where the resource was created, and the Endpoint. These parameters are not mutable, and are of great importance in the deployment of applications or consumption of models via API. When in doubt, always remember how to find them and be aware of the risks of exposing the Keys.

![Screenshot of 2024-04-16 18-32-52](https://github.com/pedropberger/tutorials/assets/98188778/4e537ca5-e685-420d-9287-dd604d6639e6)

In the Monitor tab, you control the use and consumption of your models and applications. There are several metrics with different purposes. You can monitor from the use of the resource, or tokens (which impacts the price) to training usage time for fine-tuning.

![Screenshot of 2024-04-16 18-33-08](https://github.com/pedropberger/tutorials/assets/98188778/773fd484-03e3-41ef-b507-80fa1936aa93)

Now click on "Go to Azure OpenAI Studio" in the upper left corner and let's go to what really matters.

![Screenshot of 2024-04-22 16-08-19](https://github.com/pedropberger/tutorials/assets/98188778/cc74e5d5-3f9a-49be-a051-4a801fce60b3)

You will bump into a screen similar to this one:

![Screenshot of 2024-04-22 16-36-18](https://github.com/pedropberger/tutorials/assets/98188778/78a249e9-aca0-46cf-aa8c-cab591032391)

There are several ready-made tutorials for you to play and learn. But what I advise to pay attention to is this side menu and its functionalities:

Playground
- Chat: The chat feature allows you to interact with language models, such as ChatGPT, to create application experiences based on conversations. You can send prompts and receive responses generated by the model. It is useful for creating virtual assistants, chatbots, and other natural language processing applications.
- Completions: Completions refer to autocomplete models, like GPT-4. These models can predict and continuously generate text based on an initial context. They are useful for tasks such as autocompleting sentences, generating code, or writing stories.
- DALL-E: DALL-E is an AI model that generates images from text descriptions. It is capable of creating visual art based on text prompts, allowing you to explore creativity in image generation.
- Assistants (Preview): The assistants functionality is in preview and allows you to create your own custom virtual assistants using AI models, such as GPT-4 and DALL-E. You can integrate your own data, experiment with prompts, and create custom dialogue flows to create specific assistants for your needs.

Management
- Deployments: Deployments are where you deploy trained models for production use. You can deploy language models, computer vision models, and other types of models to meet the needs of your application.
- Models: Here you will find a variety of AI models available for use. These models include features such as natural language processing, computer vision, and much more. You can choose the appropriate model for your specific application.
- Data Files: Data files are used to train and evaluate models. You can upload your own data for training or use pre-existing datasets to create custom models.
- Quotas: Quotas refer to usage limits for model deployment and inference. Each model has an associated quota, and you can adjust these quotas as needed to meet the demands of your application.
- Content Filters: Content filters are used to ensure that responses generated by models are within acceptable limits. This is especially important to avoid inappropriate or offensive content in production applications.

You feel like starting in the Playground, right? But the first step is to click on Deployment.

![Screenshot of 2024-04-16 18-34-24](https://github.com/pedropberger/tutorials/assets/98188778/51919801-3d69-473f-a46f-5c641d9ef763)

And why start with Deployment? Simple, without a model you don't have AI. The chatbot or whatever you want to do will need a model to base on (and consume resources).

I like to emphasize that the model is the car engine. Your Azure credits are the fuel. The Chatbot is just the bodywork. Car without engine doesn't run.

To create one, click on "Create new deployment" and a screen with several options will appear:

![Screenshot of 2024-04-22 16-37-35](https://github.com/pedropberger/tutorials/assets/98188778/1c7d631c-a7e3-40c7-b4d2-b06399facb63)

Understanding the screen above and defining the parameters:

Select a model: Here you decide your model. Will it be a Generative AI? select a GPT that you like. Is it an image you want? DALL-E. Embedding? Ada. To keep the coherence of this tutorial, select a GPT. I'll go for the cheapest one, gpt-35-turbo. The idea of this chatbot is just to read a library of documents and answer about it. I don't need recent information or the ability to pass the entrance exam. But everyone knows what they need. If you want to know more about which model, [search this site and you'll find everything](https://www.google.com).

Model version: Choose the version. It's hard to say how this will impact. Choose the one you like the most or stay in the standard that you can't go wrong.

Deployment type: Standard, always Standard.

Deployment name: Choose a name for the model. You can have more than one model of the same type, or of different types. The future belongs to God, so name wisely.

Content filter: For now, you can't select anything other than the default.

Tokens per Minute Rate Limit: This is important. Here you restrict how many tokens you can consume at most per minute. I recommend leaving the minimum reasonable that you think you will use and enable the dynamic quota. But if you are going to deploy several models it is important to think about the limits before. Currently per Subscription you have the limit of 240k tokens per minute, and this quota is valid FOR ALL MODELS, that is, if you have another application that consumes tokens, it is important to limit it so as not to impact your chatbot. You can view the quotas and their limits in the Quotas tab.

Ready. Then just do the deploy and see the screen below.

Important tip: like any Azure resource, you can create and delete as many times as you want, but the model's quotas take 48 hours to purge. So if you go out creating and deleting models, pay attention to the quotas provided for each model, so you don't have to wait a long time for them to return. This happened to me and the solution I [found here](https://learn.microsoft.com/en-us/azure/ai-services/recover-purge-resources?tabs=azure-portal#purge-a-deleted-resource).

![Screenshot of 2024-04-22 16-37-45](https://github.com/pedropberger/tutorials/assets/98188778/23840a85-a8f1-4c9d-80a5-03b91be3a508)

Deploy done my friends, let's create the chatbot.


## Step 3: Indexing the Data

Obviously, click on the second item in the sidebar, "Chat". And welcome to the Chat Playground. Here you can already chat, test parameters, and familiarize yourself with your chat that already has the model you deployed above as its engine.

![Screenshot of 2024-04-22 16-38-03](https://github.com/pedropberger/tutorials/assets/98188778/cb2d218c-b88d-4c50-9843-309e7f0d9621)

Our goal is a chatbot that reads files, so let's get straight to the point. Click on Add your data. And let's add your texts.

![Screenshot of 2024-04-22 16-38-25](https://github.com/pedropberger/tutorials/assets/98188778/ada23fc3-62c1-4867-ab3d-12365cccf943)

Right off the bat, you define the Data Source, and choose the option to upload your files. You can import from other options, but we won't cover that in this tutorial.

![Screenshot of 2024-04-22 16-38-50](https://github.com/pedropberger/tutorials/assets/98188778/57f191a6-9bae-4ce0-87e5-fed5c7fc77ed)

Next, you select the Subscription, and then select the Blob storage resource where the files will be stored. If you don't have this resource, just click on "Create a new Azure Blob storage resource", follow the flow, and create a Storage Account. Remember that it needs to be on the network and region of the Azure OpenAI resource.

![Screenshot of 2024-04-22 16-39-02](https://github.com/pedropberger/tutorials/assets/98188778/83731d18-458f-47e7-9fe7-8b6b8141a267)

The next step is AI Search. Here you will create the Indexers and Indexes.

Indexers and Indexes are essential components in Azure OpenAI On Your Data, which allows developers to connect, ingest, and base their corporate data to quickly create custom copilots. Let's understand what these components are:

Indexers:
- Indexers are responsible for processing the data that the service monitors. When all data is processed, Azure OpenAI triggers the second indexer.
- This second indexer stores the processed data in an Azure AI Search service.
- Indexers are fundamental for efficient search and retrieval of relevant information from corporate data.

Indexes:
- Indexes are data structures that organize and optimize access to data.
- They allow the system to execute queries efficiently, speeding up the search and retrieval of information.
- In the context of Azure OpenAI, indexes are used to improve the accuracy of the model's responses and speed up interaction with the data.

In summary, indexers process the data and store it in optimized indexes to facilitate search and analysis. These components work together to allow Azure OpenAI On Your Data to offer more accurate and efficient responses based on corporate data.

Click to create an Indexer and go to the following screen:

![Screenshot of 2024-04-22 16-41-38](https://github.com/pedropberger/tutorials/assets/98188778/ea9a8ebd-4b8f-4dbf-905d-9079950fb898)

Here you select your options and give a name to the search service. Select the right region. It is optional (and recommended) to change the Pricing tier according to the amount of data you plan to index. But be aware that the "Free" option will not work with the chatbot. At least not today. The first time I did it, it only accepted the Standard which was very expensive. Now you can do it with less burdensome options.

![Screenshot of 2024-04-22 16-41-55](https://github.com/pedropberger/tutorials/assets/98188778/6d5bf62c-53fe-4102-8751-d9d5bef9d886)

Define the number of replicas and partitions. If you don't understand or don't need scalability, skip this part, adjust the Network, the Tags, and be happy after Review+Create

![Screenshot of 2024-04-22 16-42-25](https://github.com/pedropberger/tutorials/assets/98188778/abe99d57-7f94-4e39-83bb-7109d43fd0e2)

If the screen below appears, click on Turn on CORS to continue. The explanation is in the warning.

![Screenshot of 2024-04-22 16-40-55](https://github.com/pedropberger/tutorials/assets/98188778/1f9d32da-49ac-440c-9106-88a5871c5651)

Put a name for the Index and then just create and follow the dance.

![Screenshot of 2024-04-22 16-56-12](https://github.com/pedropberger/tutorials/assets/98188778/c0cf0ed9-1338-498a-be8d-9052913e62b3)

In the following screen, you upload your files. It's basically a drag and drop. It only accepts text files in the formats presented. Upload and proceed.

![Screenshot of 2024-04-22 16-56-25](https://github.com/pedropberger/tutorials/assets/98188778/8ad16fab-3456-4a52-bdef-a75ad862d800)

In the screen below you manage how your data will be used. You can't select anything other than Keyword. But you can change the Chunk Size.

You can write a master's dissertation about Chunk Size. In my experience, I leave the following tips:

- You have small files and few pages: I recommend size 256 or 512.
- The answers will be related to small sections of the documents: 256 or 512.
- Large text files: 1024 or 1536.
- Want answers that incorporate various parts of the text or summarize entire pages: 1024 or 1536.

When in doubt, go to 1024 that it's good too.

It's funny that there is a lot of subjectivity, and the best references on the subject say that you should adjust according to your "feeling" and refine based on the responses you have received and user feedback. We don't yet have optimization functions or that automate the best fit.

![Screenshot of 2024-04-22 17-00-27](https://github.com/pedropberger/tutorials/assets/98188778/4b468949-f24e-4ce7-8367-feab35ec46e8)

Proceed and then just wait for the ingestion process of your files. They will be pre-processed and indexed one by one. If there is any problem, just redo this step. The most common problems are files in incompatible formats, existing index from other deploys, and problems accessing your Storage Account.

![Screenshot of 2024-04-22 17-00-40](https://github.com/pedropberger/tutorials/assets/98188778/e0e0480d-64c9-4a23-9e29-0c4447f04e06)

And that's it, now all that's left is to refine the parameters.


# Step 4: The Chatbot

Now we already have indexed files linked to our Azure OpenAI resource. It's time to adjust the details about the prompt and the parameters. At any time during this step, you can already test the chatbot on the main screen, which will give the answers according to the adjusted parameters in real-time.

The first step before anything else is to make adjustments to how your chatbot will use the data. For this, click on "Advanced settings" as shown in the screen below and 3 options will be presented.

![Screenshot of 2024-04-22 17-18-29](https://github.com/pedropberger/tutorials/assets/98188778/b4dfc65e-e018-4f64-be6d-4083b300663c)

The first checkbox is where you define whether the chatbot will be able to answer questions on any topic (a root ChatGPT) in addition to those related to the documents. I strongly suggest leaving it checked if your interest is indeed to create a chatbot that answers questions about the indexed documents. There are parameters to limit how much it can go out of the scope of the documents, but all my tests with users where I didn't leave this box checked were frustrating. Users hardly follow a manual and sometimes even forget

The second step is to define the "role" of your chatbot. Some are pre-configured for you to view and test. But as this goes into production, you should set it as Default (or leave it blank) and set the System Message, as shown in the screen below. In the System Message, you tell your chatbot what it is and how it should respond. Those who have studied the very basics of the prompt know the classic phrase "Speak like an expert" before chatting with the GPT. Here the function will be similar, but you say in what it will be an expert. In our case, we put "Speak like an HR employee. Always answer in Brazilian Portuguese". After entering the information, just click save.

![Screenshot of 2024-04-22 17-18-21](https://github.com/pedropberger/tutorials/assets/98188778/7b80b2eb-225a-4ebd-a2a6-83cf7a4930a2)

In the "Configuration" session, you can select the Model that you deployed and select the number of messages that will be taken into account in the prompt. Remembering that they will be considered in the total of tokens.

![Screenshot of 2024-04-22 17-18-41](https://github.com/pedropberger/tutorials/assets/98188778/99d20512-a980-4d81-8ab7-67cdd7ea0dc1)

In the parameters session is where you can play with the results of the text generation. Quickly what each one means:

Temperature:
- The temperature controls the randomness of the responses generated by the model.
- A higher temperature value (for example, 0.8-1.0) makes the responses more diverse and creative, as it allows the model to explore different possibilities.
- On the other hand, a lower value (for example, 0.2-0.5) makes the responses more focused and deterministic, being useful for fact-based queries or precise responses.

Top P:
- The Top P parameter controls the diversity of the result generated, limiting the probability distribution of words.
- When the Top P value is set to 0.4, the model considers only 40% of the most likely words or phrases.
- Higher values (for example, 0.9-1.0) ensure a wider range of options, resulting in more diverse responses.
- Lower values (for example, 0.1-0.5) limit the options to the most likely ones, making the responses more focused and coherent.

Frequency Penalty:
- It is a penalty applied to the model to prevent it from repeating words too often.
- When configured, the model is encouraged to vary its word choices, making the responses more natural and less repetitive.
- It is especially useful to avoid overly monotonous or redundant responses.

Presence Penalty:
- The Presence Penalty penalizes the model when it generates words that are not present in the provided context.
- This helps to avoid responses that include irrelevant information or that are not directly related to the question.
- The presence of penalty encourages the model to stick to the context and produce more cohesive and relevant responses.

I, personally, for a chatbot that has to be restricted and conservative, leave the temperature at full, top P low, and penalties at 0.

After finishing the settings, just do the deploy.

![Screenshot of 2024-04-22 17-18-51](https://github.com/pedropberger/tutorials/assets/98188778/876eea6a-140e-488a-83a4-fc424d4ed065)

Then two options appear. Web app is the idea of this tutorial and the one we will select. This option creates a generic interface for your chatbot (which in the future I intend to list how to edit) that can be distributed internally and externally. By default, it is restricted to your users registered in AD.

Deploying in the copilot studio will take you to another service, which is a universe apart.

![Screenshot of 2024-04-22 17-19-19](https://github.com/pedropberger/tutorials/assets/98188778/fe162f61-f757-4bee-8113-919ca7e3f562)

After selecting you will go to the screen below:

![Screenshot of 2024-04-22 17-20-17](https://github.com/pedropberger/tutorials/assets/98188778/8bd333f6-d41c-4726-bd9a-623d73b469ec)

Here you create your application, or, if you have already created, you can upgrade the existing application. You can enable chat saving if you think it's important, but then you will have to create a CosmosDB service.

![Screenshot of 2024-04-22 17-26-05](https://github.com/pedropberger/tutorials/assets/98188778/d11bee24-2132-4651-b6da-184d26194fd1)

Once this is done, just wait for the deploy (which is a bit slow) and then voila:

![Screenshot of 2024-04-22 17-42-16](https://github.com/pedropberger/tutorials/assets/98188778/c427a7d1-7e8c-4bf5-8d88-a16bc264f058)

Chatbot operating.
