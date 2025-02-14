+++
title = "2. Intro to MVC"
weight = 2
date = 2025-01-26
draft = false
+++

**Create your first Model View Controller in an ASP.NET MVC Application**

## Goal

To implement a basic newsletter feature in an ASP.NET MVC application with functionality to display a subscriber’s information.

## Step-by-step Guide

### 1. Create the Model

The model represents the data structure used in your application.

Steps:

1.	Navigate to the Models folder in your project.
2.	Add a new class file named Subscriber.cs.
3.	Define the following properties in the Subscriber class:

> Models/Subscriber.cs

```csharp
namespace Newsletter.Models;

public class Subscriber
{
    public required string Id { get; set; }   // Unique identifier for the subscriber
    public string? Name { get; set; }        // Name of the subscriber
    public required string Email { get; set; } // Email of the subscriber
}
```
> Purpose: The model defines the structure of a subscriber, encapsulating Id, Name, and Email.

### 2. Create the Controller

The controller handles incoming requests, processes data, and sends it to the view.

Steps:

1.	Navigate to the Controllers folder.
2.	Add a new controller named NewsletterController.cs.
3.	Implement the Index action in the controller:

> Controllers/NewsletterController.cs

```csharp
using Microsoft.AspNetCore.Mvc;
using Newsletter.Models;

namespace Newsletter.Controllers;

public class NewsletterController : Controller
{
    public IActionResult Index()
    {
        // Generate a mock subscriber for demonstration
        var subscriber = new Subscriber
        {
            Id = Guid.NewGuid().ToString(), // Generate unique GUID
            Name = "John Doe",
            Email = "john.doe@email.com"
        };
        
        // Log the creation of the subscriber (for debugging)
        Console.WriteLine($"Subscriber {subscriber.Name}, {subscriber.Email} with ID: {subscriber.Id} created");

        // Pass the subscriber to the view
        return View(subscriber);
    }
}
```

> Purpose: The controller fetches a mock Subscriber instance and passes it to the view.

### 3. Create the View

The view renders the UI based on the data passed from the controller.

Steps:

1.	Navigate to Views/Newsletter (create the folder if it doesn’t exist).
2.	Add a new Razor view named Index.cshtml.
3.	Implement the view to display the subscriber data:

> Views/Newsletter/Index.cshtml

```html
@model Newsletter.Models.Subscriber

<h1>Example Subscriber</h1>

<p>ID: @Model.Id</p>
<p>Name: @Model.Name</p>
<p>Email: @Model.Email</p>
```

> Purpose: The view presents the subscriber’s Id, Name, and Email data.

### 4. Update the Layout

The layout defines the shared structure of your application.

Steps:

1.	Open the _Layout.cshtml file in the Views/Shared folder.
2.	Add a navigation link to the Newsletter feature:

```html
<li class="nav-item">
    <a class="nav-link text-dark" asp-area="" asp-controller="Newsletter" asp-action="Index">Newsletter</a>
</li>
```

> Purpose: This ensures users can navigate to the newsletter page via the top navigation bar.

### 5. Run the Application

Steps:

1.	Open a terminal in your project directory.
2.	Use the following commands to run the application:

	```bash
	dotnet run
	```

3.	Open a browser and navigate to http://localhost:5000/Newsletter/Index.

	Expected Result:
	
	- The page displays the mock subscriber details:

	```
	Example Subscriber
	ID: <generated GUID>
	Name: John Doe
	Email: john.doe@email.com
	```

### Summary

• Model: Defined the Subscriber structure.
• Controller: Created an action to pass subscriber data to the view.
• View: Rendered the subscriber’s information.
• Layout: Updated the navigation to include a link to the Newsletter page.

# Happy coding! 🚀