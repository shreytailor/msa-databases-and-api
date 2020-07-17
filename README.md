# Databases & API <!-- omit in toc -->
The concepts of Application Program Interfaces (APIs) and Databases are closely connected because it provides an *abstraction* for the developers to communicate with the backend - all using something called **end-points**. 

During this assignment, we will be focusing mainly on **RESTful APIs** that uses HTTP requests to carry out simple interactions with the data such as create/read/update/delete (CRUD) operations. This repository will contain notes for some of the key points, as well as screenshots of my work. <br>
<hr>

## Table Of Contents <!-- omit in toc -->
- [Model](#model)
- [Setup Azure Database](#setup-azure-database)
- [Model & Context Creation](#model--context-creation)
- [Migrations](#migrations)
- [API Controllers](#api-controllers)
- [Swagger](#swagger)
- [Updating Our Model](#updating-our-model)
- [Deploy .NET Core Web API To Azure](#deploy-net-core-web-api-to-azure)
- [Assignment For API + Database Module](#assignment-for-api--database-module)
<hr>

## Model
Before we start to program our database, it is important to design its structure because the cost of changing it in the future would be expensive. We refer to this structure as a **model**.

Our initial (and very simple) model will contain the following data fields -
- StudentID
- FirstName
- LastName
- EmailAddress
<hr>

## Setup Azure Database
Here are basic steps for the process of creating SQL instances in Azure. The **resource group** is a collection of resources which are used for a particular application, and you need to create one for this particular application.

Thereafter you create a server which will contain your SQL database, and you can configure it according to your future-scaling needs. In real world cases, you need to restrict the access to your server from permitted IPs only - but simplicity's sake we can allow all ranging from `0.0.0.0` to `255.255.255.255`. This can be done from the server settings.
<hr>

## Model & Context Creation
To get started with the actual programming, firstly we need to create a **ASP.NET<span> Core Web Application** project in Visual Studio, and using **API** as the template. Thereafter we delete the default *WeatherForecastController.cs* and *WeatherForecast.cs* files which come with the template.

To further help us create our API, we need to install some libraries/extensions using the Nuget Package Manager.
- Microsoft.EntityFrameworkCore: it's an ORM (Object-Relational Mapper) which allows us to work with a database using `.NET` objects.
- Microsoft.EntityFrameworkCore.Tools: used for migration.
- Microsoft.EntityFrameworkCore.SqlServer: used for database communications.

Now after this configuration, we are ready to actually start defining the data layout for our database. Create a file called `Student.cs` which will contain the following code.

```cpp
public class Student
    {
        /* 
        Denotes as ID for the data row, and by default it will be set as the 
        primary key.
        */
        [Key]

        /* 
        Telling the API that the identifier above is supposed to be generated 
        automatically when we add a student
        */
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]

        // Database fields that define a student.
        public int studentId { get; set; }
        public String firstName { get; set; }
        public string lastName { get; set; }
        public string emailAddress { get; set; }
    }
```

Now we must introduce **DbContext** which is the way to incorporate EntityFramework-based data access into the application. An instance of this class represents a session with the database which can be used to query and save instances of entities into our database. Previously, we had just defined a schema for our entity instances.

To create our context, create a folder called *Data* within the project directory and then create a class within it called `StudentContext.cs` which will contain the following code.

```cpp
public class StudentContext: DbContext
    {
        /*
        An empty constructor.
        */
        public StudentContext() { }

        /*
        The base(options) code calls the base class's contructor, and in our case
        it is DbContext.
        */
        public StudentContext(DbContextOptions<StudentContext> options) :base(options) { }

        /*
        Here, we link our querying and creating process to do something relating
        to the Student schema defined in the other class.
        */
        public DbSet<Student> Student { get; set; }
        public static System.Collections.Specialized.NameValueCollection AppSettings { get; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            IConfigurationRoot configuration = new ConfigurationBuilder()
                .SetBasePath(AppDomain.CurrentDomain.BaseDirectory)
                .AddJsonFile("appsettings.json")
                .Build();

            /*
            The string 'schoolSIMSConnection' is the name of the key which contains 
            the connection string produced by our database/server system earlier.
            */
            optionsBuilder.UseSqlServer(configuration.GetConnectionString("schoolSIMSConnection"));
        }
    }
```
<hr>

## Migrations
To connect our database/server system in Azure to our project, we need to modify our `appsettings.json` file to contain our connection string. Add the following code to this file.

```json
"AllowedHosts": "*",
  "ConnectionStrings": {
    "schoolSIMSConnection": "YOUR_OWN_CONNECTION_STRING"
  }
```

We now need to generate the files required to update the database - the process of *Migration*. If we try to rename or drop a table without migration, it will delete all the existing data currently in the database. It is like a *git* for our model.

Open the package console manager and type the following commands. On successful execution, it will create a folder called *Migration* in the project directory.

```
Add-Migration InitialCreate
Update-Database
```
<hr>

## API Controllers
The **controller** is where all our API endpoints are created. We are basically generating a boilerplate to work with, in order to get all of our basic methods to work with.

Firstly, open the `Startup.cs` file and add the following code to the `ConfigureServices` method.

```cpp
var connection = Configuration.GetConnectionString("schoolSIMSConnection");
// ... where 'schoolSIMSConnection' is the key name usued in the previously created JSON file.

services.AddDbContext<StudentContext>(options => options.UseSqlServer(connection));
```

After doing this, view the hidden folders by clicking on the icon below in order to view the `Controllers` folder.

![](./images/1.PNG?)

Within that folder, select *Add -> New Scaffold Item -> Select API Controller with actions, using Entity Framework.*. Here you are supposed to select the files which you created previously.

> Before doing the above, make sure all your NuGet packages are updated to the latest versions - otherwise, you will have errors.

![](./images/2.PNG?)
<hr>

## Swagger
Swagger UI is used so that it is easier to work and interact with our created API. To initialize it, install *Swashbuckle.AspNetCore* from NuGet package manager.

Now adding the following code to `ConfigureServices` method in the `Startup.cs` file.

```cpp
services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "StudentSIMS", Version = "v1" });
});
```

Then add the following code to the `Configure` method in the same file.

```cpp
// Setting up Swagger.
app.UseSwagger();

// Enabling middleware to serve swagger-ui (HTML, JS, CSS)
// specifying the endpoint for our API.
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "My first API V1");
    c.RoutePrefix = string.Empty; // launch swagger from the root.
});
```

In `Properties/launchsetting.json` file, you much now change the launchURL to be an empty string. If everything was done correctly, it should now be configured and now test the application by pressing the F5 button. You should get a nice UI to deal with our API endpoints.

It is important to test if everything is working now, so change the field values for your first name, last name, and email address and then press the **Execute** button. Note that you don't have to change the ID because it is automatically generated.

![](./images/3.PNG?)

If we are returned with `201` and `200` HTTP response codes, everything is running fine with your API and server.
<hr>

## Updating Our Model
If our model ever needs to change, we can update the existing model which we created earlier.

```cpp
public class Student 
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int studentID {get; set;}
    [Required, MaxLength(100)]
    public string firstName {get; set;}
    public string middleName {get; set;}
    [Required]
    public string lastName {get; set;}
    public string emailAddress {get; set;}
    public int phoneNumber {get; set;}
    [TimeStamp]
    public DateTime timecreated {get; set;}
}
```

After doing so, we must run the following commands in the package manager to consider those updates made.

```
Add-Migration UpdatedStudentModel
Update-Database
```

To revert changes, simply call `Update-Database` along with the name of the previous migration.
<hr>

## Deploy .NET Core Web API To Azure
We have completed building our API, and now we can move onto the publishing phase. Firstly, we must configure the CORS policy - so that we can host the Swagger application on Azure. Add the following code to the `Startup.cs` file.

```cpp
public void ConfigureServices(IServiceCollection services)
{
    // Setting the CORS policy.
    services.AddCors();
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Setting the CORS policy.
    app.UseCors(builder => builder
        .AllowAnyHeader()
        .AllowAnyMethod()
        .SetIsOriginAllowed(host => true)
        .AllowCredentials()
    );
}
```

We have configured everything, and are ready for final deployment now. In Azure, go to the **App Servicies** section and create a new one. For the settings, set everything as below and leave the *App Services Plan* as default.

![](./images/4.PNG?)

After creating the service on Azure, we have to publish our application from Visual Studio, on it. In order to do this, right click on our project and select *Publish*. Select **New > Azure > Azure App Service (Windows)** and then select your application which you just created from the Azure Portal.

Correctly carrying out the steps above will ensure that your API is now published on the internet.
<hr>

## Assignment For API + Database Module
