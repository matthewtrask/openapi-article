Just like frameworks, API Documentation has seen its fair share of bright stars and over time those stars fading in exchange for something else. We have seen things like RAML pop up and fade, API Blueprint shine bright for a while, and Swagger keeping the lights on for a bit. While Swagger became more dominant, a group of people started working together as a consortium to build a foundation around API Specification including the group behind Swagger, SmartBear. This group, which included tech giants IBM, Google, Microsoft and more worked under the Linux Foundation to create the OpenAPI Specification and on January 1st 2016, the specification was officially launched. Swagger, which was donated by SmartBear to the foundation, was renamed the OpenAPI Specification and the community got to work building up the specification and tooling around it. It’s important to remember the following thing: Swagger is dead. Long live Swagger. I can’t tell you how many pull requests on [OpenAPI](https://openapi.tools) are not merged because of that wording. Words are important as we all know, and it’s equally important to point this out because Swagger is dead in everything but memory. 

### Documentation First
The biggest selling point to OpenAPI is that it is not just a singular source of tooling. Similar to a framework, a specification file can help you do a lot with ease and speed. Tooling exists now to automagically generate documentation that you can serve to consumers, check for attack vectors that may have not been discovered during code reviews, SDK generation, linting for local and CI/CD use and more. We will explore these topics in depth in a minute. 

To highlight the selling point though: writing an OpenAPI spec file before writing any code allows you to save money and time. Spending a day describing the API with front end developers, back end developers, ops, and leadership allows everyone to be on the same page and get input in up front (ideally, I know how the world works). Once the specification is agreed on, the contract is created. In this case the contract is delivering the API as expressed in your spec file. Similar to how we treat the production branch of Git and the production database as our sources of truth, our spec file is now the source of truth for consumers of your API. When your API is updated, the spec is to be updated and redistributed. 

### Anatomy of the Specification File
In order to make the file worth something, we need to follow some guidelines from the authors of the specification (https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md). First thing we have to decide what whether we want to use JSON or YAML to write our spec file. Thoughts aside, I’ve always found it easy to write the file in YAML if you are writing your spec files by hand. If you choose to not do it the hard way like I do, Stoplight.io has come out with an amazing tool called Stoplight Studio (https://stoplight.io/studio/) that will help you do a lot of what we will look at. 

At the core of it, the OpenAPI Spec file must have the following three objects defined:
* OpenAPI - Defining the version of the spec you are currently using. The current version of OpenAPI is 3.0.2 with 3.1.0 being worked on.
* Info - Meta information about the API being consumed. This includes things like a title, license information, terms of service, contact information incase a developer runs into an issue, etc.
* Paths - This is the true power of OpenAPI. This defines all the endpoints available to a developer consuming your API. 

As a former boss would say, this is a 30,000 feet look at the OpenAPI. Lets dive deeper. 

### What will we describe?

For this article I’ve decided to combine my love for America’s national parks and APIs together. You can find the example spec file at this [repo](https://github.com/matthewtrask/openapi-example). Over time I hope to continue working on it, even spinning up a usable API. For now though, we just need a spec file. Im going to keep it as basic as possible, and perhaps in a later article I will dive into things like `$refs` and splitting files apart so they are more managable. Today though, we are going to work in one file. As I mentioned earlier we need to tell the spec file what version we are using. This goes at the top of the file, similar to a PHP tag. 

```yaml
openapi: 3.0.2
```

After that, we need to define our info object. 

```yaml
info:
  version: 0.1.0
  title: "National Parks API"
  description: "This API covers data related to the national parks of the United States. It will allow you to read, create, update and soft delete data such as landmarks, lodging, trails, eating and more"
  contact: 
    name: Matt Trask
    email: matt@natparksapi.io (not a real email)
    url: https://natparksapi.com (not a real url)
  license: 
    name: MIT
    url: https://opensource.org/licenses/MIT
``` 

In this object we are describing information any developer will find useful. Things like how you would contact me in case you find a bug/security issue/have a question, what license covers this codebase, and what exactly can you get from using this API. This is also a place you can version your API (a topic for another time), which can hold you accountable to when you make changes to your API contract. 

These two are fairly straightforward right? Nothing ground breaking here but these two objects are essential to the process. Even if your API will never see the light of day for public consumption, it is still critical to have these objects completed. Your linting will break should you forget these two objects. 

Next comes the bulk of the OpenAPI specification: the `paths` object. I will keep this as chill as possible, but know that you can get incredibly detailed and complex with this object. 


```yaml
paths: 
  /parks:
    get: 
      summary: A data endpoint that returns all the parks in our API dataset.
      description: This returns the name, id, and links to each park in the American National Parks system. 
      operationId: getParks
      responses: 
        200: 
          description: Many Parks Response
          headers: 
            x-next: 
              description: A link to the next page of resources
              schema:
                type: string
          content: 
            application/json:
              schema:
                type: array 
                items: 
                  type: object
                  properties:
                    id:
                      type: integer
                      format: int64
                    name:  
                      type: string
                meta:
                  type: object
                  properties: 
                    totalParks:
                      type: integer
                      format: int64
                links: 
                  type: object
                  properties:
                    ref: 
                      type: string
                    next: 
                      type: string        
              examples: 
                Parks: 
                  data: {
                    { id: 1, name: 'Yosemite National Park' }, 
                    { id: 2, name: 'Grand Teton National Park' },
                    { id: 3, name: 'Zion National Park' },
                    { id: 4, name: 'Haleakala National Park' }, 
                    { id: 5, name: 'Crater Lake National Park' }
                  },
                  meta: {
                    totalCount: 61
                  }, 
                  links: {
                    ref: '/api/parks',
                    next: '/api/parks?page=2',
                  }       
        400:
          description: The request you made could not be fulfilled right now.
          content: 
            text/html:
              schema:
                type: object
                required:
                  - message
                  - code
                properties:
                  message:
                    type: string
                  code:
                    type: integer
                    minimum: 100
                    maximum: 600
```

These two resposnses are just a sample of what we can do. Going from the top `paths` object, we are doing to describe the endpoint `/parks`.  

The responses are quite expressive. OpenAPI gives us the ability to describes our success responses, redirect responses and error responses. Inside one of those responses, lets take the success response for example; we can describe the headers that will be returned with the response, the content type we will be returning and also what the response will look like. One thing to note is that OpenAPI does not officially support HATEOAS, or Hypermedia as the Engine of Application State, just yet. It is an open issue that is being worked on. Hopefully we will get that in the 3.1 release of the spec, since Hypermedia is considered crucial to the paradigm of REST. We can hack it in with the `links` object for the time being. 

At this point, you might be wondering what the difference between `summary` and `description`. Both are entirely optional for your spec file. The summary is a short explaination of what this endpoint does. The description is a more verbose explanation. Use either, both, or none. I would highly advise you to use at least one of these in your file. More information is always better for developers. The other thing to note is the `operationId` that we use. The `operationId` is a unique identifier to each operation you have and you will find that tools or libraries will use it to help identify operations in their tooling. 

Now, lets keep reading down our example, and into the `response` object. You will notice the next level keys are HTTP Status Codes. We describe a 200 OK Response and a 400 Bad Request response here. You do not need to go through every possible HTTP Code a developer will encounter, but do cover the most popular ones. The more you describe, the easier path a developer has to writing good code against your API. Inside the 200, we describe what this particular section is with the `description` key, what headers will be present with the `headers` key, and then what content a developer can expect. In this example, I only cover `application/json`, but you are free to describe whichever content types you return. In the content type, we next see a schema, which describes how the data will look when we send a request to the API endpoint. Since this is a general GET endpoint, all I will return is an id and name for each resource in the response. On top of the data there, I return a meta object that has a total count, and also links to show the developer that there is a link to page 2. If you are familiar with OpenAPI you will know there is a `header` object, but I omitted it for the sake of brevity. 

So, we have our schema, so our developer using this has an idea of what the data will look like, but now maybe we should give them, and some of the libraries that will spin up docs for us, some sample data to mock with. I picked 5 National Parks I absolutely adore and created an example response in the `example` object. One component of OpenAPI is that you can use mocking tools to create a quick feedback loop with example data you include in your specification document. This allows front end developers to have data to code against while the back end is all hooked up, thus speeding up devvelopment and testing. 

### Tools for the Spec

Since its initial release, OpenAPI has had a community flourish around it. People are writing OpenAPI 2 to OpenAPI 3 converters to help developers get up to the current spec, documentation tooling like ReDoc which automagically generate beautiful documentation around your spec files, mock servers like mentioned before to let developers get started sooner on their work instead of waiting around for the back end implementation to be completed and more. A full, up to date list of these tools can be found at [OpenAPI.tools](https://openapi.tools). 

Be warned, a lot of the tooling is offered as either a SaaS (which generally means you will pay for it), or written in various languages such as Go, Javascript, and Java to name a few. PHP isn't featured prominently as a language picked for building tools and thats ok! For most of this, you can find a JS version of the tool and use NPM or Yarn to execute the commands you wish to run. 

### Conclusion

This was a quick look at OpenAPI and what it can bring you. You don't need to wait for a brand new project to get started. OpenAPI can integrate right into existing projects with a small amount of work. Exciting things are coming in the API space going forward and OpenAPI looks to be right on the edge of the new horizon!