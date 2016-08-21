# Consuming Hubstaff API In ASP.NET Web Application

This tutorial will go over how to integrate the Hubstaff ASP.NET API into your own ASP.NET application. The Hubstaff API allows you to easily link a User to their Hubstaff account and retrieve useful information such as custom team reports, project and activity details, screenshots, and much more.

The [Hubstaff ASP.NET App repository]()  for this tutorial includes `master branch` which contains the complete application you'll have when you finish this tutorial.

This tutorial will go over linking a User to their Hubstaff account and then show how to retrieve data.

We'll retrieve two core resources provided by the Hubstaff API,
custom team reports and screenshots.

Before we start, you need to set up a [Hubstaff account](https://hubstaff.com/). I also recommend creating some data so that your application will be able to
view data, specifically create an organization, project, notes, and a few screenshots.

After you've created some data you need to go to the [Hubstaff developer
page](https://developer.hubstaff.com/), click My Apps and create a new
application. You’re ready to dive in once you create an application and receive your `App-Token`.

Clone the `master branch` down and open the application in your editor of choice. 


First we'll be initializing hubstaff api into our project by calling the following

```cs
 hubstaff.client hubstaff_api = new hubstaff.client("< your hubstaff app token >");
```

Next, we'll generate our `App-Token` using your hubstaff account email address and password.

If we take a look into Views/dashboard/index.cshtml file we can see the connection form:

```html
<p>
	<a href = "#" class = "connect" >Connect to Hubstaff</a>
  <div class = "hubstaff-form" >
  <input type="text" class="text" value="@ViewBag.email" id="email" name="email" placeholder="Please add your hubstaff account email" />
  <input type="text" class="text" value="@ViewBag.password" id="email" name="password" placeholder="Please add your hubstaff account password" />
  <br>
  <input type="submit" class="submit" name = "submit" value="Connect" />
  </div>
</p>
```
Form submission will call the following ASP.NET code found in controllers/dashboard.cs to generate the authentication token.

```cs
hubstaff.client hubstaff_api = new hubstaff.client();
ViewBag.Data = hubstaff_api.auth(app_token,email,password);
ViewBag.email = email;
ViewBag.password = password;
```

Once we got all that done we can start requesting account related data like reports, users, organizations, notes and others from hubstaff.

Now let's start with fetching the team reports in a specific period of time.

First we need to specify all the parameters we're going to use for that operation.

```cs
Dictionary <string, string[]> param = new Dictionary <string, string[]>();
param["start_date"] = new string[] {"start_date"};
param["end_date"]   = new string[] {"end_date"};
param.Add("options", new string[] {"organizations","projects","users","show_tasks","show_notes","show_activity","include_archived"});

Dictionary <string, string> value_type = new Dictionary <string, string>();
value_type["organizations"] = "input";
value_type["projects"] = "input";
value_type["users"] = "input";
value_type["start_date"] = "datetime";
value_type["end_date"] = "datetime";
value_type["show_tasks"] = "select";
value_type["show_notes"] = "select";
value_type["show_activity"] = "select";
value_type["include_archived"] = "select";
```
We're going to have to required parameters "start_date" and "end_date" of type date ("YYYY-MM-DD").

Next we gonna generate our form using the following code.

```cs
@foreach (var item in ViewBag.param) {
  if(item.Key == "options")
  {
    foreach(var item2 in item.Value)
    {
      if(ViewBag.value_type[item2] == "input")
      {
        <div class = "input-container" ><span class = "title">@item2</span><input type = "text" name = "options[@item2]" ></div>
      }else if(ViewBag.value_type[item2] == "datetime")
      {
        <div class = "input-container" ><span class = "title">@item2</span><input type = "text" name = "options[@item2]" class="form-control time" ></div>
      }else
      {
        <div class = "input-container" ><span class = "title">@item2</span><select name = "options[@item2]" ><option>0</option><option>1</option></select></div>
      }
    }
  }else
  {
    if(ViewBag.value_type[item.Key] == "input")
    {
      <div class = "input-container" ><span class = "title">@item.Key</span><input type = "text" name = "@item.Key" ></div>
    }else if(ViewBag.value_type[item.Key] == "datetime")
    {
      <div class = "input-container" ><span class = "title">@item.Key</span><input type = "text" name = "@item.Key" class="form-control time" ></div>
    }else
    {
      <div class = "input-container" ><span class = "title">@item.Key</span><select name = "@item.Key" ><option>0</option><option>1</option></select></div>
    }
  }
}
```

Then we'll be requesting the report by calling custom_date_team function
```cs
hubstaff.client hubstaff_api = new hubstaff.client();
ViewBag.reports = hubstaff_api.custom_date_team(start_date, end_date, options);
```
Now let's print the output into our screen by iterating over the retured json string
```php
@if(ViewBag.reports != null){
	foreach(var org in ViewBag.reports["organizations"])
	{
		<h2>Organization Name: @org["name"] </h2>
		foreach(var dates in org["dates"])
		{
			foreach(var users in dates["users"])
			{
				<h3>User Name: @users["name"]</h3>
				<h4>Time Spent: @users["duration"]</h4>
				<br>
				foreach(var project in users["projects"])
				{
          <h3>User Name: @project["name"]</h3>
          <h4>Time Spent: @project["duration"]</h4>
          <br>
				}
			}
		}
	}
}	
```
And we're going to have something that looks like this

![asp_report](/images/asp_report.png)

And the same goes for screenshots functions, by changing the parameters to
```cs
Dictionary <string, string[]> param = new Dictionary <string, string[]>();
param["start_time"] = new string[] {"start_time"};
param["stop_time"]   = new string[] {"stop_time"};
param.Add("options", new string[] {"organizations","projects","users","offset"});

Dictionary <string, string> value_type = new Dictionary <string, string>();
value_type["organizations"] = "input";
value_type["projects"] = "input";
value_type["users"] = "input";
value_type["offset"] = "input";
value_type["start_time"] = "datetime";
value_type["stop_time"] = "datetime";
```
And generate the form like mentioned before, after that we gonna call the screenshots function using the following
```cs
hubstaff.client hubstaff_api = new hubstaff.client();
ViewBag.reports = hubstaff_api.custom_date_team(start_time, stop_time, offset, options);

```
And we'll have the following output:

![asp_screenshot](/images/asp_screenshot.png)

The `hubstaff-dotnet` client allows an ASP.NET application to easily retrieve useful information from the Hubstaff app, making it easy for your customers to access their Hubstaff data within your application.
