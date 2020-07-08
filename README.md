# Databases & API <!-- omit in toc -->
The concepts of Application Program Interfaces (APIs) and Databases are closely connected because it provides an *abstraction* for the developers to communicate with the backend - all using something called **end-points**. 

During this assignment, we will be focusing mainly on **RESTful APIs** that uses HTTP requests to carry out simple interactions with the data such as create/read/update/delete (CRUD) operations. This repository will contain notes for some of the key points, as well as screenshots of my work. <br>
<hr>

### Table Of Contents <!-- omit in toc -->
- [Model](#model)
- [Setup Azure Database](#setup-azure-database)
- [Model & Context Creation](#model--context-creation)
- [Migrations](#migrations)
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

### Migrations
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