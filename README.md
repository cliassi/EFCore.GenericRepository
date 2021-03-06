# EF Core Generic Repository

This library is an almost perfect Generic Repository implementation for EF Core ORM which will remove developers' pain to write repository layer for each .NET Core and .NET project.

## This library includes following notable features:

1. This library can be run on any .NET Core or .NET application which has .NET Standard 2.0 and .NET Standard 2.1 support.

2. It’s providing the Generic Repository with and without Unit of Work pattern.

3. It has all the required methods to query your data in whatever way you want without getting IQueryable<T> from the repository.

4. It also has **`Specification<T>`** pattern support so that you can build your query dynamically i.e. differed query building.

5. It also has database level projection support for your query.

6. It also has support to run raw SQL command against your relational database.

7. It also has support to choose whether you would like to track your query entity/entities or not.

8. It also has support to reset your EF Core DbContext state whenever you really needed.

9. Most importantly, it has full Unit Testing support.

## Giving a star (⭐)

**If you find this library useful, please don't forget to encouraging me to do such more stuffs by giving a star to this repository. Thank you.**

## How do I get started?

First install the appropriate version of `TanvirArjel.EFCore.GenericRepository` [nuget](https://www.nuget.org/packages/TanvirArjel.EFCore.GenericRepository) package into your project as follows:

**For EF Core 2.x.x :**

    Install-Package TanvirArjel.EFCore.GenericRepository -Version 2.0.2
    
**For EF Core 3.0.x :**

    Install-Package TanvirArjel.EFCore.GenericRepository -Version 3.0.2
    
**For EF Core >= 3.1.x :**

    Install-Package TanvirArjel.EFCore.GenericRepository -Version 3.1.2
    
Then in the `ConfirugeServices` method of the `Startup` class:

    public void ConfigureServices(IServiceCollection services)
    {
        string connectionString = Configuration.GetConnectionString("RepositoryDbConnection");
        services.AddDbContext<DemoDbContext>(option => option.UseSqlServer(connectionString));
        
        services.AddGenericRepository<DemoDbContext>(); // Call it just after registering your DbConext.
    }
    
## Usage:

    public class EmployeeService
    {
         private readonly IRepository _repository;
         
         public EmployeeService(IRepository repository)
         {
             _repository = repository;
         }
         
         public async Task<Employee> GetEmployeeAsync(int employeeId)
         {
             Employee employee = await _repository.GetEntityByIdAsync<Employee>(1);
             return employee;
         }
    }
    
## More Details:

#### 1. To get all the data:

    var employeeList =  await _repository.GetEntityListAsync<Employee>();
    
    var noTrackedEmployeeList = await _repository.GetEntityListAsync<Employee>(asNoTracking: true);
    
#### 2. To get a filtered list of data:

    var employeeList =  await _repository.GetEntityListAsync<Employee>(e => e.EmployeeName.Contains("Tanvir") && e.DepartmentName == "Software");
                        
    var noTrackedEmployeeList = await _repository
                                .GetEntityListAsync<Employee>(e => e.EmployeeName.Contains("Tanvir") && e.DepartmentName == "Software", asNoTracking: true);

#### 3. To get a list of data by Specification<T>:
    
    Specification<Employee> specification = new Specification<Employee>();
    specification.Conditions.Add(e => e.EmployeeName.Contains("Tanvir"));
    specification.Includes = ep => ep.Include(e => e.Department);
    specification.OrderBy = sp => sp.OrderBy(e => e.EmployeeName).ThenBy(e => e.DepartmentName);
    specification.Skip = 0;
    specification.Take = 10;

    List<Employee> employeeList = await _repository.GetEntityListAsync<Employee>(specification);
                                  
    List<Employee> noTrackedEmployeeList = await _repository.GetEntityListAsync<Employee>(specification, true);
                                  
 #### 4. To get the projected entity list:
    
    Expression<Func<Employee, object>> selectExpression = e => new { e.EmployeeId, e.EmployeeName };
    var projectedList = await _repository.GetProjectedEntityListAsync<Employee, object>(selectExpression);
                      
 #### 5. To get filtered projected entity list:
 
    Expression<Func<Employee, object>> selectExpression = e => new { e.EmployeeId, e.EmployeeName };
    var filteredProjectedList = await _repository.GetProjectedEntityListAsync<Employee, object>(e => e.IsActive, selectExpression);
                                            
 #### 6. To get the projected entity list by `Specification<T>`:
 
    Specification<Employee> specification = new Specification<Employee>();
    specification.Conditions.Add(e => e.EmployeeName.Contains("Tanvir"));
    specification.Includes = ep => ep.Include(e => e.Department);
    specification.OrderBy = sp => sp.OrderBy(e => e.EmployeeName).ThenBy(e => e.DepartmentName);
    specification.Skip = 0;
    specification.Take = 10;
    
    Expression<Func<Employee, object>> selectExpression = e => new { e.EmployeeId, e.EmployeeName };
    var projectedList = await _repository.GetProjectedEntityListAsync<Employee, object>(specification, selectExpression);
                      
#### 7. To get an entity by Id (primary key):

    Employee employee = await _repository.GetEntityByIdAsync<Employee>(1);
    
    Employee noTrackedEmployee = await _repository.GetEntityByIdAsync<Employee>(1, true);
    
#### 8. To get a projected entity by Id (primary key):
    
    Expression<Func<Employee, object>> selectExpression = e => new { e.EmployeeId, e.EmployeeName };
    var projectedEntity = await _repository.GetProjectedEntityByIdAsync<Employee, object>(1, selectExpression);

#### 9. To get a single entity by any condition/filter:

    Employee employee = await _repository.GetEntityAsync<Employee>(e => e.EmployeeName == "Tanvir");
    
    Employee noTrackedEmployee = await _repository.GetEntityAsync<Employee>(e => e.EmployeeName == "Tanvir", true);
    
#### 10. To get a single entity by `Specification<T>`:
    
    Specification<Employee> specification = new Specification<Employee>();
    specification.Conditions.Add(e => e.EmployeeName == "Tanvir");
    specification.Includes = sp => sp.Include(e => e.Department);
    specification.OrderBy = sp => sp.OrderBy(e => e.Salary);
    
    Employee employee = await _repository.GetEntityAsync<Employee>(specification);
    
    Employee noTrackedEmployee = await _repository.GetEntityAsync<Employee>(specification, true);
    
#### 11. To get a single projected entity by any condition/filter:

    Expression<Func<Employee, object>> selectExpression = e => new { e.EmployeeId, e.EmployeeName };
    var projectedEntity = await _repository.GetProjectedEntityAsync<Employee, object>(e => e.EmployeeName == "Tanvir", selectExpression);
    
#### 12. To get a single projected entity by `Specification<T>`:
    
    Specification<Employee> specification = new Specification<Employee>();
    specification.Conditions.Add(e => e.EmployeeName == "Tanvir");
    specification.Includes = sp => sp.Include(e => e.Department);
    specification.OrderBy = sp => sp.OrderBy(e => e.Salary);
    
    Expression<Func<Employee, object>> selectExpression = e => new { e.EmployeeId, e.EmployeeName };
    var projectedEntity = await _repository.GetProjectedEntityAsync<Employee, object>(specification, selectExpression);

#### 13. To check if an entity exists:

    bool isExists = await _repository.IsEntityExistsAsync<Employee>(e => e.EmployeeName == "Tanvir");
    
#### 14. To create or insert a new entity:

    Employee employeeToBeCreated = new Employee()
    {
       EmployeeName = "Tanvir",
       ..........
    }
    
    await _repository.InsertEntityAsync<Employee>(employeeToBeCreated);
    await _repository.SaveChangesAsync();
    
#### 15. To create or insert a collection of new entities:

    List<Employee> employeesToBeCreated = new List<Employee>()
    {
       new Employee(){},
       new Employee(){},
    }
    
    await _repository.InsertEntitiesAsync<Employee>(employeesToBeCreated);
    await _repository.SaveChangesAsync();
    
#### 16. To update or modify an entity:

    Employee employeeToBeUpdated = new Employee()
    {
       EmployeeId = 1,
       EmployeeName = "Tanvir",
       ..........
    }
    
    _repository.UpdateEntity<Employee>(employeeToBeUpdated);
    await _repository.SaveChangesAsync();
    
#### 17. To update or modify the collection of entities:

    List<Employee> employeesToBeUpdated = new List<Employee>()
    {
       new Employee(){},
       new Employee(){},
    }
    
    _repository.UpdateEntities<Employee>(employeesToBeUpdated);
    await _repository.SaveChangesAsync();
    
#### 18. To delete an entity:

    Employee employeeToBeDeleted = new Employee()
    {
        EmployeeId = 1,
        EmployeeName = "Tanvir",
       ..........
    }
    
    _repository.DeleteEntity<Employee>(employeeToBeDeleted);
    await _repository.SaveChangesAsync();
    
#### 19. To delete a collection of entities:

    List<Employee> employeesToBeDeleted = new List<Employee>()
    {
       new Employee(){},
       new Employee(){},
    }
    
    _repository.DeleteEntities<Employee>(employeesToBeDeleted);
    await _repository.SaveChangesAsync();
    
#### 20. To get the count of entities with or without condition:

    int count =   await _repository.GetCountAsync<Employee>(); // Count of all
    int count =   await _repository.GetCountAsync<Employee>(e => e.EmployeeName = "Tanvir"); // Count with specified condtion
    
    long longCount =   await _repository.GetLongCountAsync<Employee>(); // Long count of all
    long longCount =   await _repository.GetLongCountAsync<Employee>(e => e.EmployeeName = "Tanvir"); // Long count with specified condtion
