+++
title = "3. Create a Form with Validation and Feedback"
weight = 3
date = 2025-01-26
draft = false
+++

## Goal

Enhance your ASP.NET MVC application by creating a subscription form that validates user input on the client side. Follow these steps to implement client-side validation and provide immediate feedback to users.

## Step-by-step Guide

### 1. Update Your Model

The **model** defines the data structure for your application. In this step, you’ll add validation attributes to enforce rules for the `Name` and `Email` fields.

Steps:

1. Open the `Subscriber.cs` file in the `Models` folder.
2. Add the necessary validation attributes to the `Subscriber` class.

```csharp
using System.ComponentModel.DataAnnotations;

namespace Newsletter.Models;

public class Subscriber
{
    public string? Id { get; set; }
    [Required]
    public string? Name { get; set; }
    [Required, EmailAddress]
    public string? Email { get; set; }
}
```

> Purpose:
> 
> - `[Required]`: Ensures the `Name` and `Email` fields are not empty.
> - `[EmailAddress]`: Validates that the email field contains a properly formatted email address.

### 2. Add Controller Actions

The **controller** handles both displaying the form and processing form submissions. Here, you will create two `Subscribe` actions to manage `GET` and `POST` requests respectively.

Steps:

1. Open the `NewsletterController.cs` file in the `Controllers` folder.
2. Implement the `Subscribe` actions:

```csharp
using Microsoft.AspNetCore.Mvc;
using Newsletter.Models;

namespace Newsletter.Controllers;

public class NewsletterController : Controller
{
    public IActionResult Index()
    {
        // Create a subscriber object and pass it to the view
        var subscriber = new Subscriber
        {
            Id = Guid.NewGuid().ToString(),
            Name = "John Doe",
            Email = "john.doe@email.com"
        };
        Console.WriteLine($"Subscriber {subscriber.Name}, {subscriber.Email} with ID: {subscriber.Id} created");
        return View(subscriber);
    }

    [HttpGet]
    public IActionResult Subscribe()
    {
        return View(new Subscriber());
    }

    [HttpPost]
    public async Task<IActionResult> Subscribe(Subscriber subscriber)
    {
        // Validate the model state
        if (ModelState.IsValid)
        {
            // Log the subscriber details
            Console.WriteLine($"Name: {subscriber.Name}, Email: {subscriber.Email}");

            // TODO: Implement the subscription logic
            await Task.Delay(100); // Simulate an operation

            // Set a result message in TempData in order to display it in the view
            TempData["SuccessMessage"] = "You have successfully subscribed to our newsletter!";

            // Redirect back to the Subscribe GET action to clear the form
            return RedirectToAction("Subscribe");
        }

        // If the model state is invalid, redisplay the form with validation errors
        return View(subscriber);
    }
}
```

> Purpose:
> 
> - The `Subscribe` actions renders the subscription form (`GET`) and processes the submitted data (`POST`).
> - `ModelState.IsValid`: Ensures the submitted data meets validation rules defined in the model.
> - `TempData["SuccessMessage"]`: Displays a confirmation message after a successful subscription.


### 3. Create Your View

The **view** displays the subscription form and provides immediate validation feedback.

Steps:

1. Navigate to the `Views/Newsletter` folder (create it if it doesn’t exist).
2. Add a new Razor view named `Subscribe.cshtml`.
3. Use ASP.NET Core tag helpers for validation and display a success message.

```html
@model Newsletter.Models.Subscriber

<!-- Display message from the TempData sent by the controller -->
@if (TempData["SuccessMessage"] != null)
{
    <div class="alert alert-success alert-dismissible fade show" role="alert">
        @TempData["SuccessMessage"]
        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"><i class="fas fa-times"></i></button>
    </div>
}

<h2>Subscribe to our newsletter</h2>

<!-- The form to subscribe to the newsletter. Uses ASP tag helpers for validation -->
<form asp-action="Subscribe" method="post">
    <div class="form-group">
        <label asp-for="Name" class="control-label"></label>
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>
    <div class="form-group">
        <label asp-for="Email" class="control-label"></label>
        <input asp-for="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>
    <button type="submit" class="btn btn-primary">Subscribe</button>
</form>

<!-- This script is used to display the validation error messages on the client side -->
@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}
```

> Purpose:
> 
> - **Form fields**: Automatically bind to the `Name` and `Email` properties of the model.
> - **Validation messages**: Display error messages when input is invalid.
> - **Success message**: Notify users of successful subscriptions.

### 4. Update the Layout

Update the link in the top NavBar

Steps:

1.	Open the _Layout.cshtml file in the Views/Shared folder.
2.	Update the navigation link to the Newsletter subscribe page:

```html
<li class="nav-item">
    <a class="nav-link text-dark" asp-area="" asp-controller="Newsletter" asp-action="Subscribe">Newsletter</a>
</li>
```

### 5. Run Your Application

Steps:

1. Open a terminal in your project directory.
2. Run the application:
   ```bash
   dotnet run
   ```
3. Open your browser and navigate to:
   ```
   http://localhost:5000/Newsletter/Subscribe
   ```

	Expected Result:
	
	- The subscription form validates `Name` and `Email` inputs on the client side.
	- Invalid fields display error messages without submitting the form.
	- Successful submissions redirect users with a confirmation message.


## Summary

- **Model**: Added validation attributes for `Name` and `Email`.
- **Controller**: Handled form display and submission using `Subscribe` actions.
- **View**: Created a form with error messages and success feedback.
- **Validation**: Included client-side validation scripts for real-time feedback.

# You're done! 🎉 Test your form!
