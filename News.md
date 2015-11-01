# What is new #

## 02.04.2011 - GenericRepository 1.1.0 released ##
Rebuilded with latest libraries
  * NHibernate 3.1.0 with build-in LINQ provider
  * Latest Autofac, LinFu DynamicProxy, NUnit
Bugfixes.
Clean up.

Entity Framework is still CTP4, EF 4.1 is now Release Candidate (RC), waiting for official release.

## 29.7.2010 - Extended ISpecificationResult ##
Extended ISpecificationResult with following useful common functionality:
  * Skip
  * SingleOrDefault
  * OrderByAscending
  * OrderByDescending
Added implementation for IQueryable based specification result and NHibernate ICriteria based specification result.
Added unit tests.

### Important note regarding _Skip_ functionality ###
See [GenericCustomerRepositoryFixture.cs](http://code.google.com/p/genericrepository/source/browse/Besnik.GenericRepository.Tests/Fixtures/GenericFixtures/GenericCustomerRepositoryFixture.cs), test method GetCustomersButSkipSome() for comments and play with it to experience the issues.

**NHibernate 2.1.2.4000 with NHibernate LINQ compiled for this version**
Only NH is able to correctly handle Skip command (also together with other commands like OrderByXXX) and has to dependencies on concrete command position in fluent interface call.

**Linq2Sql that ships with .NET 4.0**
Linq2Sql ignores OrderByXXX command if present AFTER Skip call.

**Entity Framework 4.0 with extensions CTP4**
EF is not capable of handling Skip command without preceeding OrderByXXX command. It will not handle Skip command if OrderByXXX is after the Skip, the OrderByXXX MUST GO BEFORE Skip command. Otherwise it throws.



## 25.7.2010 - Finalized Entity Framework 4.0 Features CTP4 implementation ##
Fixed unit tests and added test initialization code.
As CTP currently have no support for clearing tables before each test run, we have to clar the tables manually. Hopefully this will be resolved by MS team in future.

## 21.7.2010 - Added implementation for Entity Framework 4 with Features CTP4 ##
Features CTP4 assembly is included in the Lib folder, no need to install it explicitely. Only need is to have entity framework 4 that ships with .NET 4.0.
Feature only available in source code, release build of assembly haven't been done yet.