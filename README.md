# Databases & API <!-- omit in toc -->
The concepts of Application Program Interfaces (APIs) and Databases are closely connected because it provides an *abstraction* for the developers to communicate with the backend - all using something called **end-points**. 

During this assignment, we will be focusing mainly on **RESTful APIs** that uses HTTP requests to carry out simple interactions with the data such as create/read/update/delete (CRUD) operations. This repository will contain notes for some of the key points, as well as screenshots of my work. <br>
<hr>

### Table Of Contents <!-- omit in toc -->
- [Model](#model)
- [Setup Azure Database](#setup-azure-database)
- [Model & Context Creation](#model--context-creation)
<hr>

### Model
Before we start to program our database, it is important to design its structure because the cost of changing it in the future would be expensive. We refer to this structure as a **model**.

Our initial (and very simple) model will contain the following data fields -
- StudentID
- FirstName
- LastName
- EmailAddress

### Setup Azure Database
Here are basic steps for the process of creating SQL instances in Azure. The **resource group** is a collection of resources which are used for a particular application, and you need to create one for this particular application.

Thereafter you create a server which will contain your SQL database, and you can configure it according to your future-scaling needs. In real world cases, you need to restrict the access to your server from permitted IPs only - but simplicity's sake we can allow all ranging from `0.0.0.0` to `255.255.255.255`. This can be done from the server settings.

### Model & Context Creation
To get started with the actual programming, firstly we need to create a **ASP.NET<span> Core Web Application** project in Visual Studio, and using **API** as the template. Thereafter we delete the default *WeatherForecastController.cs* and *WeatherForecast.cs* files which come with the template.

To further help us create our API, we need to install some libraries/extensions using the Nuget Package Manager.
- Microsoft.EntityFrameworkCore: it's an ORM (Object-Relational Mapper) which allows us to work with a database using `.NET` objects.
- Microsoft.EntityFrameworkCore.Tools: used for migration.
- Microsoft.EntityFrameworkCore.SqlServer: used for database communications.

Now after this configuration, we are ready to actually start defining the data layout for our database. Create a file called `Student.cs` which will contain the following code.

```cpp
public class Student
    {
        /* Denotes as ID for the data row, and by default it will be set 
            as the primary key.
        */
        [Key]

        /* Telling the API that the identifier above is supposed to be generated 
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