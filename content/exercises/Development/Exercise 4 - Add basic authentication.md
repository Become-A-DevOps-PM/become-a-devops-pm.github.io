+++
title = "4. Add Basic Authentication"
weight = 4
date = 2025-01-26
draft = false
+++

## Goal

In this tutorial, we’ll add cookie-based authentication to your ASP.NET MVC application. This allows users to log in, access restricted pages, and log out.

## Step-by-step Guide

### 1. Add Authentication Services in `Program.cs`

To enable cookie-based authentication, you need to configure the authentication middleware in your application.

Steps:

1. Open the `Program.cs` file in the root of your project.
2. Add the `AddAuthentication` and `AddCookie` services to the `builder.Services` configuration.

> Program.cs

```csharp
...

// Add support for basic authentication
builder.Services.AddAuthentication("Cookies")
    .AddCookie(options => options.LoginPath = "/Account/Login");

var app = builder.Build();

...

app.UseAuthentication();
app.UseAuthorization();

...

```

> Purpose:
> 
> - **`AddAuthentication("Cookies")`**: Enables cookie-based authentication.
> - **`AddCookie`**: Configures the behavior of cookie authentication, including the login path (`/Account/Login`).

### 2. Add a `LoginViewModel`

Use a `LoginViewModel` for validation and encapsulating user input.

Steps:

1. Open the `Models` folder. Create a new file `LoginViewModel.cs `:

> Models/LoginViewModel.cs

```csharp
using System.ComponentModel.DataAnnotations;

namespace Newsletter.Models;

public class LoginViewModel
{
   [Required(ErrorMessage = "Username is required.")]
   public string? Username { get; set; }

   [Required(ErrorMessage = "Password is required.")]
   [DataType(DataType.Password)]
   public string? Password { get; set; }
}
```

### 3. Add the `AccountController`

The `AccountController` handles login and logout actions.

Steps:

1. Open the `AccountController.cs` file in the `Controllers` folder (create it if it doesn’t exist).
2. Implement the following actions:

> Controllers/AccountController.cs

```csharp
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Newsletter.Models;

namespace Newsletter.Controllers;

public class AccountController : Controller
{
    [HttpGet]
    public IActionResult Login() => View(new LoginViewModel());

    [HttpPost]
    public async Task<IActionResult> Login(LoginViewModel model)
    {
        if (!ModelState.IsValid)
        {
            return View(model);
        }

        // Replace with proper user validation (e.g., check against a database)
        if (model.Username == "admin" && model.Password == "password")
        {
            var claims = new[] { new Claim(ClaimTypes.Name, model.Username) };
            var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
            var principal = new ClaimsPrincipal(identity);
            await HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme, principal);

            return RedirectToAction("Index", "Home");
        }

        ModelState.AddModelError(string.Empty, "Invalid username or password.");
        return View(model);
    }

    public async Task<IActionResult> Logout()
    {
        await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
        return RedirectToAction("Index", "Home");
    }
}
```

> Purpose:
> 
> - **`Login` (GET)**: Displays the login form.
> - **`Login` (POST)**: Authenticates the user and sets the authentication cookie.
> - **`Logout`**: Signs the user out and clears their authentication cookie.

### 4. Create the Login View

The login form allows users to enter their credentials.

Steps:

1. Navigate to the `Views/Account` folder (create it if it doesn’t exist).
2. Add a new Razor view named `Login.cshtml`.
3. Implement the form:

> Views/Account/Login.cshtml

```html
@model Newsletter.Models.LoginViewModel

<!-- Display validation summary for model-level errors -->
@if (!ViewData.ModelState.IsValid)
{
    <div class="alert alert-danger">
        @Html.ValidationSummary(false, null, new { @class = "text-danger" })
    </div>
}

<div class="container mt-5">
    <h2>Login</h2>
    <form method="post" asp-action="Login">
        <div class="mb-3">
            <label asp-for="Username" class="form-label"></label>
            <input asp-for="Username" class="form-control" />
            <span asp-validation-for="Username" class="text-danger"></span>
        </div>
        <div class="mb-3">
            <label asp-for="Password" class="form-label"></label>
            <input asp-for="Password" type="password" class="form-control" />
            <span asp-validation-for="Password" class="text-danger"></span>
        </div>
        <button type="submit" class="btn btn-primary">Login</button>
    </form>
</div>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}
```

> Purpose:
> 
> - Submits the credentials to the `Login` action in `AccountController`.
> - Use Tag Helpers (`asp-for` and `asp-validation-for`) for automatic data binding and error message display.
> - Include client-side validation via `_ValidationScriptsPartial`.

### 5. Update the Top NavBar for Authentication Links

The layout file provides links for logging in and out based on the user’s authentication state.

Steps:

1. Open the `_Layout.cshtml` file in the `Views/Shared` folder.
2. Add links for logging in and out:

> Views/Shared/_Layout.cshtml

```html
...

@if (User.Identity != null && User.Identity.IsAuthenticated)
{
    <li class="nav-item">
        <a class="nav-link text-dark" asp-controller="Account" asp-action="Logout">Sign Out</a>
    </li>
}
else
{
    <li class="nav-item">
        <a class="nav-link text-dark" asp-controller="Account" asp-action="Login">Sign In</a>
    </li>
}

...
```

> Purpose:
> 
> - Dynamically shows `Sign In` or `Sign Out` links based on the user's authentication state.


### 6. Secure Actions with Authentication

Protect sensitive actions by requiring authentication. I this step we will setup a _Who am I_ page that will be protected and show the logged in user´s - the principal´s - claims.

Steps:

1. Open `HomeController.cs` in the `Controllers` folder. Add the following action:

	> Controllers/HomeController.cs
	
	```csharp
	using Microsoft.AspNetCore.Authorization;
	
	...
	
	[Authorize]
	public IActionResult WhoAmI()
	{
	    return View();
	}
	```

2. Navigate to the `Views/Home` folder and create a new Razor view named `WhoAmI.cshtml`. Add the following code to display the user's claims:

	> Views/Home/WhoAmI.cshtml
	
	```html
	<h2>Who Am I?</h2>
	
	<table class="table">
	    <thead>
	        <tr>
	            <th>Claim Type</th>
	            <th>Claim Value</th>
	        </tr>
	    </thead>
	    <tbody>
	        @foreach (var claim in User.Claims)
	        {
	            <tr>
	                <td>@claim.Type</td>
	                <td>@claim.Value</td>
	            </tr>
	        }
	    </tbody>
	</table>
	```

3. Place a link in the footer of the page. Navigate to the `Views/Shared` folder and update the page footer in `_Layout.cshtml`.

	```html
	...
	
	<footer class="border-top footer text-muted">
	    <div class="container d-flex justify-content-between align-items-center">
	        <div>
	            &copy; 2025 - Newsletter - <a asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
	        </div>
	        <div>
	            <a asp-area="" asp-controller="Home" asp-action="WhoAmI" class="text-decoration-none">Who Am I?</a>
	        </div>
	    </div>
	</footer>
	
	...
	```

### 7. Run the Application

Steps:

1. Open a terminal in your project directory.
2. Run the application:
   ```bash
   dotnet run
   ```
3. Open your browser and navigate to:
   ```
   http://localhost:5000/Account/Login
   ```

	Expected Behavior:
	
	- Visit the `Subscribers` page: You’ll be redirected to the login page if unauthenticated.
	- Log in with `admin/password`: You’ll be redirected to the homepage, and the `Sign Out` link will appear.
	- Click `Sign Out`: You’ll be logged out and see the `Sign In` link.


## Summary

- **`Program.cs`**: Configured cookie authentication.
- **`AccountController`**: Added login and logout actions.
- **Login View**: Created a form for user authentication.
- **Layout**: Dynamically displayed links for login/logout.
- **Authorization**: Secured pages with the `[Authorize]` attribute.

# Basic authentication setup is complete! 🎉
