# Getting Started on Connected Environment with .NET Core

Previous step: [Debugging containers in Kubernetes](get-started-netcore-04.md)

In this section we're going to create a second service, `mywebapi`, and have `webfrontend` call it. Each service will run in separate containers. We'll then debug across both containers.

![](media/multi-container.png)

## Download Sample Code for mywebapi
For the sake of time, let's download some sample code from a GitHub repository. Navigate to https://github.com/johnsta/vsce-samples and select **Clone or Download** to download the GitHub repository. The code for this section is in `vsce-samples/dotnetcore/getting-started/mywebapi`.


## Run mywebapi
1. Open the folder `mywebapi` in a *separate VS Code window*.
1. Hit F5, and wait for the service to build and deploy. You'll know it's ready when the VS Code debug bar appears.
1. Take note of the endpoint URL, it will look something like http://localhost:\<portnumber\>. It may seem like the container is running locally, but actually it is running in our development environment in Azure. The reason for the localhost address is because `mywebapi` has not defined any public endpoints and can only be accessed from within the Kubernetes instance. For your convenience, and to facilitate interacting with the private service from your local machine, Connected Environment creates a temporary SSH tunnel to the container running in Azure.
1. When `mywebapi` is ready, open your browser to the localhost address. Append `/api/values` to the URL to invoke the default GET API for the `ValuesController`. 
1. If all the steps were successful, you should be able to see a response from the `mywebapi` service.


## Make a Request from 'webfrontend' to 'mywebapi'
Let's now write code in `webfrontend` that makes a request to `mywebapi`.
1. Switch to the VS Code window for `webfrontend`.
1. *Replace* the code for the About method:

```csharp
public async Task<IActionResult> About()
{
    ViewData["Message"] = "Hello from webfrontend";
    
    // Use HeaderPropagatingHttpClient instead of HttpClient so we can propagate
    // headers in the incoming request to any outgoing requests
    using (var client = new HeaderPropagatingHttpClient(this.Request))
        {
            // Call 'mywebapi', and display its response in the page
            var response = await client.GetAsync("http://mywebapi/api/values/1");
            ViewData["Message"] += " and " + await response.Content.ReadAsStringAsync();
        }

    return View();
}
```

Note how Kubernetes' DNS service discovery is employed to simply refer to the service as `http://mywebapi`. **Code in our development environment is running the same way it will run in production**.

The code example above also makes use of a `HeaderPropagatingHttpClient` class. This helper class was added to your code folder at the time you ran `vsce init`. `HeaderPropagatingHttpClient` is dervied from the well-known `HttpClient` class - the only functionality it adds to `HttpClient` is to propagate specific headers from an existing ASP .NET HttpRequest object into an outgoing HttpRequestMessage object. We'll see later how this facilitates a more productive development experience in team scenarios.


## Debug Across Multiple Services
1. At this point, `mywebapi` should still be running with the debugger attached. If it is not, hit F5 in the `mywebapi` project.
1. Set a breakpoint in the `Get(int id)` method that handles `api/values/{id}` GET requests.
1. In the `webfrontend` project, set a breakpoint just before it sends a GET request to `mywebapi/api/values`.
1. Hit F5 in the `webfrontend` project.
1. Invoke the web app, and step through code in both services.
1. In the web app, the About page will display a message concatenated by the two services: "Hello from webfrontend and Hello from mywebapi".


Well done! You now have a multi-container application where each container can be developed and deployed separately.

> [!div class="nextstepaction"]
> [Learn about team development](get-started-netcore-06.md)

