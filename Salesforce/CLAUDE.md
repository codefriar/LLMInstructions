# Salesforce Apex Development Guidelines for Claude

The following guidelines should be followed when helping with Salesforce Apex development:

## Language Limitations

- Apex has very limited reflection capabilities.
- When the need arises to test a private method, add the `@TestVisible` annotation to the line above the method to be tested's definition. This will effectively make the method `public` for the duration of a unit test.
- Use private constructors with the `@TestVisible` annotation to provide test-only dependency injection enabled constructors.

## Platform oddities

- Salesforce projects, while developed locally, must be deployed to an org for testing.
- Salesforce requires that All Apex code have an Aggregate code coverage of >= 75%. Writing or refactoring an Apex class therefore means also writing or refactoring the tests.
- More importantly, regardless of code coverage percentage, I require that _all logic branches_ in the code have coverage.
- Test classes should follow the naming convetion ClassNameTests. Example: IdServiceTests, or AccountServiceTests
- Salesforce provides a bash cli tool for interacting with project metadata called `sf`. you may safely assume `sf` is installed.
- Here are some common `sf` tasks, and when to use them
  - `sf project deploy start` - This will deploy all metadata in the project directory.
  - `sf project deploy start -m apexclass:SOMECLASS_WITHOUT_FILE_EXTENSION` - this deploys a specific apex class to the default org
    - adding `-o nameOfOrg` allows you to specify which org to deploy the metadata to.
  - `sf org list` - generates a table of orgs and their authentication status that are known to this computer.
  - adding `--json` to the end of any `sf` command returns the output in machine readable json
  - `sf project retrieve start -m apexClass:SomeClass_Without_File_Extension` - this retrieves metadata from the org.
  - Salesforce metadata is _generally_ found in the `force-app/main/default/` directory. `force-app` is in the same directory as this file.
  - `sf apex test run` - this is the high level command to run when you want to execute an Apex test in an org.
  - adding `-h` to a `sf` command, for example `sf apex test run -h` will teach you the options and flags needed to run a given command.
  - always run `sf apex test run` with the `-c` an `-v` parameters.
  - a full list of `sf` functions available through your use of bash can be found by running `sf commands` and you may find `sf commands --json` to be helpful
- Additionally, Salesforce has added some helper functionality through the projects' `package.json` file. Specifically, the `scripts` child object.
  - You may wish to use:
    - `npm run lint` - this will lint the Javascript for both Aura and LWC components
    - `npm run test:unit` - this will run the LWC Component Tests.
    - `npm run test:unit:watch` - this runs the unit tests whenever a components' js changes
    - `npm run test:unit:debug` - useful for attching a js debugger to the tests to debug logic in realtime.
    - `npm run test:unit:coverage` - This is the same as `npm run test:unit` but captures unit test coverage. Given a choice, always use the `test:unit:coverage` option.
    - `npm run prettier` - this runs prettier --write against all the metadata in the org. You should run this whenever you modify metadata files.

## Architecture Standards

### Naming Conventions and Locations

- No variable can have less than 3 characters in its name.
- Apex classes should be placed in relevant sub-folders under the standard `classes` folder based on their type.
  - Classes extending the `BaseRepo` class, or ending in `Repo` or `Repository` should be placed or moved to the `classes/repositories` folder.
  - Classes ending in `TriggerHandler` should be placed or moved to the `classes/triggerHandlers` folder.
  - Classes ending in `Service` should be placed or moved to the `classes/services` folder.
  - Classes ending in `Tests` should be placed or moved to the `classes/tests` folder.
  - If a class file contains nothing but an ENUM definiton, it should be palced or moved to the `classes/enums` folder.
- Class and Method names must be semantically meantiful, and should bias towards longer, more descriptive names rather than shorter ones.
  - Class names are limited to 40 characters.

### PreferGuardClauses

// Use early returns instead of nested if statements to reduce complexity.
**Bad:**
if (acc != null) {
if (acc.Name != null) {
// main logic
}
}
**Good:**
if (acc == null) {return;}
if (acc.Name == null) {return;}
// main logic

### The Repository Pattern

A repository pattern must be used for all Create, Update, Read(Query) and Delete actions:

- `BaseRepo.cls`: Base class that all repositories extend
- Security modes: USER_MODE (enforces permissions) and SYSTEM_MODE (bypasses permissions)
- Security context enum: ALLOWS_SAFE (default) and ALLOWS_UNSAFE (requires justification)
- Repositories handle CRUD operations with security enforcement
- Default to BaseRepo methods that work with SECURITY_ENFORCED

Example repositories:

- `AccountRepo.cls`: Handles account-related operations

Usage:

```apex
// Using with user context (safe)
AccountRepo repo = new AccountRepo();
List<Account> accounts = repo.getAccountsByType('Customer');

// Using with system context (unsafe, requires justification)
AccountRepo repo = new AccountRepo(BaseRepo.SECURITY_CONTEXT.ALLOWS_UNSAFE);
repo.updateAccounts(accountsToUpdate, 'Batch operation needs system access');
```

### Service Layer

Services encapsulate business logic and complex operations:

Example Services

- `IdService.cls`: Manages generation of IDs
- `SQIDService.cls`: Handles encoding/decoding of unique IDs
- `ContractIDService.cls`: Generates unique 12-digit contract numbers

Usage:

```apex
// Generate a contract ID
String contractId = ContractIDService.generateContractNumber();
```

### Trigger Framework

- Triggers must contain a bare minimum of logic, and should delgate actual work to a trigger handler class.
- Trigger's should contain very little more than the following code, regardless of object
- Use Custom metadata type (`Metadata_Driven_Trigger__mdt`) to enable/disable triggers
- Handler classes implement business logic and are referenced by the metadata.

Example:

```apex
trigger AccountTrigger on Account(after insert, before insert, after update, before update) {
    new MetadataTriggerFramework().run();
}
```

## Metadata Standards

- Use apiVersion 64.0 when creating or modifying \*.cls-meta.xml files

## Coding Conventions

- Apex annotations like `@TestVisible` and `@IsTest` are Pascal case
- Avoid the use of `@SuppressWarnings` annotation.

## Unit Testing Conventions and BestPractices - These assume the use of ApexKit's testing utitilites

- The Stub.Builder class is used to create mock implementations of classes for unit testing in Apex. Here's how and when to use it:

  When to Use Stub.Builder

  1. When writing isolated unit tests where you need to mock dependencies
  2. When testing error handling paths by simulating exceptions
  3. When you need to verify interaction with dependencies

  How to Use Stub.Builder

  Basic Pattern
  // Create the mock object
  MyClassName mockObj = (MyClassName) new Stub.Builder(MyClassName.class)
  .mockingMethodCall('methodNameToMock')
  .withParameterTypes(Type1.class, List<Type2>.class) // Define param types
  .withParameterValues(param1, param2) // Define expected params. You may also use PARAMETERMATCHER.ANYPARAMER to allow the mock to accept any parameter value.
  .returning(returnValue) // Define return value. This can be a previously constructed object, or hardcoded value
  .defineStub(true); // Create and return the stub

  Key Steps

  1. Initialize with class type: `new Stub.Builder(Class.class)`
  2. Specify the method you want to mock using `mockingMethodCall('methodNameHere')`
  3. Specify parameter types with `withParameterTypes(TypeOfParam.class, TypeOfParam2.class)`
  4. Specify expected parameter values with `withParameterValues(value1, 3, 'foo')`
  5. Define return behavior with one of:

  - returning(value) - Return a specific value
  - returning() - For void methods
  - throwingException() - Throw a generic exception
  - throwingException(customException) - Throw a specific exception

  6. Create the stub with defineStub(true) or defineStub().createStub()
  7. Cast to your interface/class type

  Parameter Matching

  - Use ParameterMatcher.ANYPARAMETER to match any value for a parameter:
    .withParameterValues(ParameterMatcher.ANYPARAMETER)

  Testing Method Calls

  - Use stubInstance.assertAllMockedMethodsWereCalled() to verify all mocked methods were called

### Use Test Factory for test data

- Use TestFactory to create test data:
  Account foo = (Account) TestFactory.createSObject(new Account()); // creates an Account object, but with required fields pre-filled out.
  Account foo = (Account) TestFactory.createSObject(new Account(), true); // Creates an account object and inserts it;
  List<Account> accounts = (List<Account>) TestFactory.createSObjectList(new Account(), 5, true); // Creates a list of 5 accounts with required fields populated and inserts them.

### use IdFactory for test

- Do not hardcode ids. Use the IdFactory.get('ObjectType') method to get a plausible, but non-hardcoded id:
  Id foo = IdFactory.get(Account.sObjectType);
  Id foo = IdFactory.get('Account');
  Id foo = IdFactory.get(Account.class);

### Use the HttpCalloutMockFactory

- Use the HttpCalloutMockFactory to create mock HTTP Callouts in apex:
  HttpCalloutMockFactory calloutMocks = new HttpCalloutMockFactory(200, 'OK', 'the response body', responseHeaderMap);
  Test.setMock(HttpCalloutMock.class, calloutMocks);
  // for multiple callouts
  HttpResponse firstResponse = new HttpResponse();
  firstResponse.setStatusCode(200);
  firstResponse.setStatus('OK');
  firstResponse.setBody('The response body');
  HttpResponse secondResponse = new HttpResponse();
  secondResponse.setStatusCode(200);
  secondResponse.setStatus('OK');
  secondResponse.setBody('The response body');
  List<HttpResponse> seriesOfResponses = new List<HttpResponse>{firstResponse,secondResponse}
  HttpCalloutMockFactory calloutMocks = new HttpCalloutMockFactory(seriesOfResponses);
  Test.setMock(HttpCalloutMock.class, seriesOfMocks);

- Always use the Arrange, Act, Assert pattern for creating unit tests
- Always use the Assert._ methods, instead of the System.Assert_ methods in Apex unit tests
- Always use Test.startTest() and Test.stopTest() calls to wrap the 'act' section of apex unit tests

## Best Practices

### ApexAssertionsShouldIncludeMessage

- Always include a message in Assert, Assert.areEqual, and Assert.areNotEqual for clarity in test failures.
  **Bad:**
  System.assert(o.isClosed);
  System.assertNotEquals('123', o.StageName);
  **Good:**
  System.assert(o.isClosed, 'Opportunity is not closed.');
  System.assertEquals('123', o.StageName, 'Opportunity stageName is wrong.');

### ApexUnitTestClassShouldHaveAsserts

- Every Apex unit test should include at least one assertion for robustness and clarity.
  **Bad:**
  @isTest
  public class Foo {
  public static testMethod void testSomething() {
  Account a = null;
  a.toString();
  }
  }
  **Good:**
  @isTest
  public class Foo {
  public static testMethod void testSomething() {
  Account a = null;
  System.assertNotEquals(a, null, 'account not found');
  }
  }

---

### ApexUnitTestClassShouldHaveRunAs

// Include at least one System.runAs in your test classes to test user context and permissions.
**Good:**
System.runAs(u) {
// test logic
}

---

### ApexUnitTestMethodShouldHaveIsTestAnnotation

// Use @isTest annotation for test methods, not the deprecated 'testMethod' keyword.
**Bad:**
static testmethod void methodCTest() { ... }
**Good:**
@isTest static void methodCTest() { ... }

---

### ApexUnitTestShouldNotUseSeeAllDataTrue

// Do not use @isTest(seeAllData=true); always create your own test data.
**Bad:**
@isTest(seeAllData = true)
public class Foo { ... }

---

### AvoidGlobalModifier

// Avoid using the global modifier unless absolutely necessary; prefer public or private.
**Bad:**
global class Unchangeable { ... }

---

### AvoidLogicInTrigger

// Avoid business logic in triggers; delegate to handler classes for maintainability and testability.
**Bad:**
trigger Accounts on Account (before insert) {
// logic here
}

---

### DebugsShouldUseLoggingLevel

// Always specify LoggingLevel in System.debug statements for better log filtering.
**Bad:**
System.debug('Some debug info');
**Good:**
System.debug(LoggingLevel.WARN, 'Something might be wrong.');

---

### QueueableWithoutFinalizer

// When using Queueable, always attach a Finalizer to handle post-processing or error recovery.
**Bad:**
public class MyJob implements Queueable {
public void execute(QueueableContext context) { ... }
}
**Good:**
public class MyJob implements Queueable, Finalizer {
public void execute(QueueableContext context) {
System.attachFinalizer(this);
// ...
}
public void execute(FinalizerContext ctx) { ... }
}

---

### UnusedLocalVariable

// Remove any local variables that are declared but never used.
**Bad:**
String x = 'unused';
String y = 'used';
return y;

---

## Code Style

### ClassNamingConventions

// Class, interface, and enum names should use PascalCase (e.g. MyClass, MyInterface).
**Bad:**
public class fooClass { }
**Good:**
public class FooClass { }

---

### FieldDeclarationsShouldBeAtStart

// Declare fields at the top of your class, before any methods.
**Bad:**
class Foo {
public void myMethod() { }
public Integer myField;
}
**Good:**
class Foo {
public Integer myField;
public void myMethod() { }
}

---

### FieldNamingConventions

// Fields should use camelCase. Constants use ALL_CAPS. Enum constants use ALL_CAPS.
**Bad:**
Integer INSTANCE_FIELD;
**Good:**
Integer instanceField;

---

### ForLoopsMustUseBraces

// Always use braces for for-loops, even if one statement.
**Bad:**
for (Integer i = 0; i < 10; i++)
foo();
**Good:**
for (Integer i = 0; i < 10; i++) {
foo();
}

---

### FormalParameterNamingConventions

// Method parameters should use camelCase.
**Bad:**
public void foo(Integer PARAM) { }
**Good:**
public void foo(Integer param) { }

---

### IfElseStmtsMustUseBraces

// Always use braces for if-else statements.
**Bad:**
if (foo)
x = x + 1;
else
x = x - 1;
**Good:**
if (foo) {
x = x + 1;
} else {
x = x - 1;
}

---

### IfStmtsMustUseBraces

// Always use braces for single-line if statements.
**Bad:**
if (foo)
x++;
**Good:**
if (foo) {
x++;
}

---

### LocalVariableNamingConventions

// Local variables should use camelCase.
**Bad:**
Integer LOCAL_VARIABLE;
**Good:**
Integer localVariable;

---

### MethodNamingConventions

// Method names should use camelCase.
**Bad:**
public void MY_METHOD() { }
**Good:**
public void myMethod() { }

---

### OneDeclarationPerLine

// Declare only one variable per line.
**Bad:**
Integer a, b;
**Good:**
Integer a;
Integer b;

---

### PropertyNamingConventions

// Properties should use camelCase.
**Bad:**
public Integer PROPERTY { get; set; }
**Good:**
public Integer property { get; set; }

---

### WhileLoopsMustUseBraces

// Always use braces for while-loops.
**Bad:**
while (true)
x++;
**Good:**
while (true) {
x++;
}

---

## Best Practices

### ApexAssertionsShouldIncludeMessage

// Always include a descriptive message as the second parameter in System.assert (or third in System.assertEquals / System.assertNotEquals).
System.assert(x > 0, 'x should be greater than zero');

### ApexUnitTestClassShouldHaveAsserts

// Every Apex unit test class must contain at least one assertion.
@isTest
private class MyTestClass {
static testMethod void testSomething() {
System.assert(true, 'Should always be true');
}
}

### AvoidDirectAccessTriggerMap

// Never access Trigger.new or Trigger.old directly inside loops.
for (Account acc : Trigger.new) {
// process
}

### AvoidDmlStatementsInLoops

// Avoid DML statements (insert, update, delete, etc.) inside loops.
List<Account> accsToUpdate = new List<Account>();
for (Account acc : accList) {
acc.Name = 'Updated';
accsToUpdate.add(acc);
}
update accsToUpdate;

### AvoidSoqlInLoops

// Never include SOQL queries inside loops.
List<Account> accs = [SELECT Id, Name FROM Account WHERE Name = 'Test'];

for (Account acc : accs) {
// process
}

### AvoidSoslInLoops

// Never include SOSL queries inside loops.
List<List<Account>> searchResults = [FIND 'Acme*' IN ALL FIELDS RETURNING Account (Id, Name)];
for (Account acc : searchResults[0]) {
// process
}

### MethodsMustBeGlobalOrPublic

// Methods must not have package-private or protected visibility; use public or global.
public void doSomething() { }

## Code Style

### ClassNamingConventions

// Class, interface, enum, and test class names must be in PascalCase.
public class MyTestClass { }

### FieldDeclarationsShouldBeAtStart

// Place all field declarations before any method declarations in a class.
public class MyClass {
Integer a;
void foo() {}
}

### FieldNamingConventions

// Field names must use camelCase, constants use UPPER_CASE.
public class Foo {
Integer instanceField;
static final Integer SOME_CONSTANT = 42;
}

### ForLoopsMustUseBraces

// Always use braces with for loops, even for single statements.
for (Integer i=0; i < 10; i++) {
doSomething();
}

### FormalParameterNamingConventions

// Method parameters must use camelCase.
public void processAccount(Integer accountId) { }

### IfElseStmtsMustUseBraces

// Use braces with if/else statements, even for single statements.
if (flag) {
x++;
} else {
x--;
}

### IfStmtsMustUseBraces

// Always use braces with if statements, even for single statements.
if (flag) {
x++;
}

### LocalVariableNamingConventions

// Local variable names must use camelCase.
Integer localVar = 1;

### MethodNamingConventions

// Method names must use camelCase.
public void processAccount() { }

### OneDeclarationPerLine

// Only declare one variable per line.
Integer a;
Integer b;

### PropertyNamingConventions

// Property names must use camelCase.
public Integer accountTotal { get; set; }

### WhileLoopsMustUseBraces

// Always use braces with while loops, even for single statements.
while (flag) {
x++;
}

## Design

### AvoidDeeplyNestedIfStmts

// Avoid creating deeply nested if-then statements; keep nesting to three levels or fewer.
public class Foo {
public void bar(Integer x, Integer y, Integer z) {
if (x > y) {
if (y > z) {
// do something
}
}
}
}

### CognitiveComplexity

// Avoid highly complex methods; break up methods so their cognitive complexity is less than 15.
public void simpleMethod() {
if (condition) {
doSomething();
}
}

### CyclomaticComplexity

// Keep cyclomatic complexity below 10 per method; refactor large methods into smaller ones.
public void process(Boolean flag) {
if (flag) {
doSomething();
} else {
doSomethingElse();
}
}

### ExcessiveClassLength

// Avoid classes with more than 1000 lines; split into smaller, more focused classes.
public class UserManager {
// less than 1000 lines
}

### ExcessiveParameterList

// Avoid methods with more than 4 parameters; use objects to group related parameters.
public void addPerson(Date birthdate, BodyMeasurements measurements, Integer ssn) {
// ...
}

### ExcessivePublicCount

// Avoid classes with more than 20 public methods, fields, or properties; break into smaller classes.
public class AccountHandler {
public void doWork() {}
public String value;
// ...less than 20 public members...
}

### NcssConstructorCount

// Constructors should not exceed 20 NCSS lines; keep constructors concise.
public Foo() {
super();
setup();
}

### NcssMethodCount

// Methods should not exceed 40 NCSS lines; split long methods into smaller helpers.
public Integer method() {
return 1;
}

### NcssTypeCount

// Classes should not exceed 500 NCSS lines; refactor into smaller types if needed.
public class SmallType {
// under 500 NCSS lines
}

### StdCyclomaticComplexity

// Keep cyclomatic complexity below 10 for methods; reduce nested and complex logic.
public void example() {
if (a) {
fiddle();
}
}

### TooManyFields

// Classes should have no more than 15 fields; group related fields into objects.
public class Person {
Date birthDate;
BodyMeasurements measurements;
}

### UnusedMethod

// Remove any methods that are not used anywhere in the codebase.
public class Triangle {
public Triangle(Double a, Double b, Double c) { /_ ... _/ }
public Double getPerimeter() { /_ ... _/ }
}

## Documentation

### ApexDoc

// Provide ApexDoc comments for all public/global classes, methods, and properties (excluding overrides and test code); include @description, @return for non-void methods, and @param for each parameter.
/\*\*

- @description Handles account logic.
  _/
  public class AccountHandler {
  /\*\*
  _ @description Calculates the sum.
  _ @param a The first number.
  _ @param b The second number.
  _ @return The sum of a and b.
  _/
  public Integer sum(Integer a, Integer b) {
  return a + b;
  }
  }

## Error Prone

### ApexCSRF

// Never perform DML operations (insert, update, delete, etc.) in constructors, initializers, or methods named "init".

### AvoidDirectAccessTriggerMap

// Never directly access Trigger.old or Trigger.new by index (e.g., Trigger.new[0]); always iterate using a loop.
for (Account a : Trigger.new) { /_ ... _/ }

### AvoidHardcodingId

// Do not hardcode Salesforce record IDs in code. Query or look up IDs dynamically.
Id myRecTypeId = [SELECT Id FROM RecordType WHERE ... LIMIT 1].Id;

### AvoidNonExistentAnnotations

// Do not use annotations that are not officially supported by Salesforce Apex.

### AvoidStatefulDatabaseResult

// Do not store Database._Result objects (like SaveResult) as instance variables in stateful batch classes; use a custom serializable type instead.
public class StatefulResult { /_ ... \*/ }
List<StatefulResult> results = new List<StatefulResult>();

### EmptyCatchBlock

// Do not leave catch blocks empty; always log or handle exceptions.
try { ... } catch (Exception e) { System.debug(e); }

### EmptyIfStmt

// Avoid empty if statements; remove or implement the block.

### EmptyStatementBlock

// Do not leave method or code blocks empty; remove or add meaningful code.

### EmptyTryOrFinallyBlock

// Avoid empty try or finally blocks.

### EmptyWhileStmt

// Avoid empty while loops.

### InaccessibleAuraEnabledGetter

// Properties exposed to Lightning (with @AuraEnabled) must have public getters.
@AuraEnabled public Integer counter { get; set; }

### MethodWithSameNameAsEnclosingClass

// Do not create methods with the same name as the class (except for constructors).

### OverrideBothEqualsAndHashcode

// If you override equals(), also override hashCode(), and vice versa.
public Boolean equals(Object o) { /_ ... _/ }
public Integer hashCode() { /_ ... _/ }

### TestMethodsMustBeInTestClasses

// All @IsTest or testMethod methods must reside in a class annotated with @IsTest.
@IsTest
private class TestClass { @IsTest static void myTest() {} }

### TypeShadowsBuiltInNamespace

// Do not name your classes, enums, or interfaces the same as System or Schema types.

## Performance

### AvoidDebugStatements

// Avoid System.debug() statements in production code; they impact performance and consume CPU time.
// Use Apex Replay Debugger, Checkpoints, or remove debug statements.

### AvoidNonRestrictiveQueries

// Avoid unfiltered SOQL/SOSL queries; always use WHERE clauses or LIMIT to restrict results.
Account[] accs2 = [SELECT Id FROM Account LIMIT 10];

### EagerlyLoadedDescribeSObjectResult

// Avoid calling SObjectType.getDescribe() or Schema.describeSObjects() without specifying SObjectDescribeOptions; use DEFERRED for lazy loading.
Account.SObjectType.getDescribe(SObjectDescribeOptions.DEFERRED);

### OperationWithHighCostInLoop

// Do not call expensive Schema methods (like Schema.getGlobalDescribe()) inside loops; call them once before the loop.
Map<String, Schema.SObjectField> map = Schema.getGlobalDescribe();
for (String f : fields) {
// use 'map'
}

### OperationWithLimitsInLoop

// Never perform DML, SOQL, SOSL, send emails, or queue async jobs inside a loop; batch outside loops to avoid governor limits.
insert accounts;

## Security

### ApexBadCrypto

// Never use hardcoded IVs or keys in Crypto calls; always generate them randomly.

### ApexCRUDViolation

// Always check CRUD and FLS permissions before SOQL, SOSL, or DML operations.
// Use WITH SECURITY_ENFORCED, isAccessible(), isUpdateable(), etc.
Contact c = [SELECT Status__c FROM Contact WHERE Id=:ID WITH SECURITY_ENFORCED];
if (Schema.sObjectType.Contact.fields.Status**c.isUpdateable()) {
c.Status**c = status;
update c;
}

### ApexDangerousMethods

// Never call methods that disable CRUD security checks or log sensitive data.

### ApexInsecureEndpoint

// Never use plain HTTP endpoints; always use HTTPS.
req.setEndpoint('https://example.com');

### ApexOpenRedirect

// Never redirect to user-controlled URLs without validation; avoid open redirects.

### ApexSharingViolations

// Always declare sharing mode (with/without sharing) on classes that perform DML operations.

### ApexSOQLInjection

// Never concatenate untrusted input into SOQL queries; use bind variables instead.
[SELECT Id FROM Account WHERE Name = :name];

### ApexSuggestUsingNamedCred

// Never hardcode credentials for callouts; use Named Credentials.
// Use Named Credentials configured in Salesforce.

### ApexXSSFromEscapeFalse

// Never use addError(message, false); always escape error messages.

### ApexXSSFromURLParam

// Always escape and validate values obtained from URL parameters before use.
