# Introduction #

The project aims to unify the way we are implementing and using the repository pattern. It also provides implemented functionality for various data mappers like NHibernate, Entity Framework or LinqToSql. The plan is to support as much data mappers as possible; This should be doable as the layer is very lightweight and thin.

# Motivation #

Most of the application that are using repository pattern or it's variation are implementing the same thing - the repository and queries for the data. The GenericRepository library tries to provide that functionality out of the box and in unified way. It also uses so called Filter pattern together with Service Locator pattern for resolving queries, all in DDD manner.
It is a good practice not to tightly couple 3rd party libraries to your projects; Generic repository wants to help here.

# Design #
The library puts no requirements on your domain model. You are free to use POCO (plain old clr object) style of programming. Currently the only restriction is that the interface is working only with reference types (not value types - structs). This might be removed in the future.

# Example #
Assume trivial domain model with following entity:
```
public class Customer
{
	public virtual int Id { get; set; }
	public virtual string Name { get; set; }
	public virtual int Age { get; set; }
}
```
We would like to do CRUD operations (create, read, update, delete) for this domain entity from the data storage (e.g. sql database). Generic repository provides us predefined interface for those operations:
```
public interface IGenericRepository<TEntity, TPrimaryKey> where TEntity : class
{
  void Insert(TEntity entity);
  void Update(TEntity entity);
  void Delete(TEntity entity);

  TEntity GetById(TPrimaryKey id);

  IList<TEntity> GetAll();

  TSpecification Specify<TSpecification>() where TSpecification : class, ISpecification<TEntity>;
}
```
Last method of the interface might not be that clear, we will get back to it later. For now remember, that it allows you to specify query, that will be used to load entities from the data storage (e.g. load only customers with _Age_ higher than 18 and _Name_ starting with "Peter".

We can work directly with the interface, but it is more suitable and domain oriented to define dedicated interface for our entity:
```
public interface ICustomerRepository : IGenericRepository<Customer, int>
{
}
```
Second type parameter on System.Int32 specifies type of primary key used to store the entity.

Generic Repository contains generic class that already implements all methods:
```
public class GenericRepository<TEntity, TPrimaryKey> : IGenericRepository<TEntity, TPrimaryKey>
		where TEntity : class
{
   ...
}
```

You can work directly with the `GenericRepository<>` and `IGenericRepository<>` OR it is more convenient and does not cost much effort to create dedicated implementation for the entity:
```
public class CustomerRepository : GenericRepository<Customer, int>, ICustomerRepository
{
  public CustomerRepository(IUnitOfWork unitOfWork, ISpecificationLocator specificationLocator)
    : base(unitOfWork, specificationLocator)
  {
  }
}
```
Together with IoC container (inversion of control, dependency injection) you can get implementation of `ICustomerRepository`. This also improves readability.

Each repository works with two things:
  * a unit of work that wrapps connection to the data storage, manages loaded entities and transactions.
  * specification locator. it is basically wrapper over a IoC container that resolves your domain specifications (domain specific queries with strong names of the methods). Specifications = filters.

For creating unit of works we have so called unit of work factory in generic repository:
```
public interface IUnitOfWorkFactory : IDisposable
{
  IUnitOfWork BeginUnitOfWork();
  void EndUnitOfWork(IUnitOfWork unitOfWork);
}
```
Generic repository comes packed with predefined implementations for data mappers available out there. Lets assume we decided to work with NHibernate behind the scene. Then we need to create new instance of `NHibernateUnitOfWorkFactory`. The constructor expect path to the config file with settings like connection string and assembly where mappings .hbm.xml files are embedded.

`IUnitOfWork` implementations are nothing else than wrappers over concrete data mappers unit of works objects (e.g. `ISession` in NHibernate). They are most important as they can read and write the data using underlying data mapper. And finally they have one additional important feature. `IUnitOfWork` is able to create transactions (see example below).

Lets put everything together, that means `IUnitOfWork`, `IUnitOfWorkFactory`, `IGenericRepository`, `ITransaction` and we have following straighforward code that works with transactions:
```
var specificationLocator = this.IoC.Resolve<ISpecificationLocator>();

using ( var unitOfWork = this.IoC.Resolve<IUnitOfWorkFactory>().BeginUnitOfWork() )
{
  ICustomerRepository cr = this.IoC.Resolve<ICustomerRepository>(unitOfWork, specificationLocator);

  using ( var transaction = unitOfWork.BeginTransaction() )
  {
    cr.Insert(customer);

    transaction.Commit();
  }
}
```

The above demostrates clean unified way how to deal with CRUD operations. Concrete data mappers are hidden behind interfaces of generic repository. More examples can be found in unit tests of the generic repository, please see source code for more details.

Switching to other data mapper like LinqToSql is not a problem, all you need todo is to configure IoC container to load apropriate implentations of the above interfaces.

# Summary #
Generic Repository provides you with all important interfaces and their generic implementations. Clients of the library are responsible just for plumbing functionality, e.g. instantiating `IUnitOfWorkFactory`, resolving repositories and opening transactions (if needed). It is suggested to create and use concrete classes instead of generic (e.g. `ICustomerRepository` instead of `IGenericRepository<>`.