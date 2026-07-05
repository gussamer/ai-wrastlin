# Project Context

All Projects that I work on and searchs I perform

## About This Project

Projects will generally be Salesforce development mainly Apex and LWC projects and commandline devops tools also generally related to salesforce development

## Key Directories


## Standards

### SOLID Design Principles Explained

### 1. Single Responsibility Principle (SRP)

  ## BAD - Multiple responsibilities

  ```apex
  // User class with multiple unrelated responsibilities
  public class User {
      private String name;
      private String email;

      public User(String name, String email) {
          this.name = name;
          this.email = email;
      }

      // Persistence logic (database operations)
      public void save() {
          // Database insert/update logic here
          System.debug('Saving user: ' + name);
      }

      // Email functionality
      public void sendEmail(String message) {
          // Send email logic here
          System.debug('Sending email to: ' + email);
      }
  }
  ```

  ## GOOD - Single responsibility

  ```apex
  // User class - single responsibility: data storage only
  public class User {
      private String name;
      private String email;

      public User(String name, String email) {
          this.name = name;
          this.email = email;
      }

      // Getter and setter methods for encapsulation
      public String getName() { return name; }
      public void setName(String name) { this.name = name; }

      public String getEmail() { return email; }
      public void setEmail(String email) { this.email = email; }
  }
  
    // Repository class - single responsibility: persistence only
    public class UserRepository {
        // Persistence logic for user data
        public static void save(User user) {
            // Database insert/update logic here
            System.debug('Saving user to database: ' + user.getName());
        }

        // Additional CRUD operations
        public static User findById(Id id) {
            // Database query logic here
            return null;
        }
    }

    // Service class - single responsibility: email functionality
    public class EmailService {
        // Email sending logic
        public static void send(String recipient, String message) {
            // Email service implementation here
            System.debug('Sending email to: ' + recipient);
        }
    }
  ```

  Key Differences in Apex Implementation:

  1. Access Modifiers: Apex uses private, public, and static keywords for visibility and behavior
  2. Getters/Setters: Proper encapsulation with getter/setter methods following best practices
  3. Static Methods: Repository methods are typically static since they operate on the database layer
  4. System.debug(): Apex-specific debugging method used in examples
  5. String Types: Apex uses String class instead of Python's native string type

  Benefits of the SOLID Implementation:

  - Maintainability: Each class has a clear purpose, making changes easier and safer
  - Testability: Unit tests can focus on individual responsibilities without mocking multiple behaviors
  - Reusability: Components can be reused independently throughout your Apex code
  - Scalability: Adding new functionality won't require changing existing classes

  The SOLID principles work together to create robust, maintainable Salesforce code that follows best practices for
  enterprise-level development.

  ```apex
    class UserRepository:
        def save(self, user): pass  # Persistence only

    class EmailService:
        def send(self, recipient, message): pass  # Email only
  ```

  ### 2. Open/Closed Principle (OCP) in Apex

  Definition

  Software entities should be open for extension but closed for modification.

  When to apply

  When you need to add new functionality without changing existing code. In a multicurrency context, you can add support for
  new currency types without modifying the core CurrencyFormatter class.

  Example Implementation

  ## BAD - Modify every time you need a new currency type

  ```apex
    public class CurrencyFormatter {
        public String currencyCode;

        public Decimal formatAmount(Decimal amount) {
            if (currencyCode == 'USD') {
                return Math.round(amount * 100) / 100; // Standard rounding
            } else if (currencyCode == 'EUR') {
                return Math.round(amount * 100) / 100; // Same implementation
            } else if (currencyCode == 'JPY') {
                return Math.round(amount); // Different logic for JPY
            }
            return amount;
        }
    }
  ```

  In this example, every time you add a new currency code, you must modify the formatAmount method. This violates OCP.

  ## GOOD - Extend via inheritance and polymorphism

  ```apex
    // Abstract base class - represents any currency formatter
    public abstract class CurrencyFormatter {
        // Abstract method that enforces implementation in subclasses
        public abstract Decimal formatAmount(Decimal amount);
    }

    // USD formatter with specific rounding logic
    public class USDFormatter extends CurrencyFormatter {
        public override Decimal formatAmount(Decimal amount) {
            // Standard 2-decimal precision for USD
            return Math.round(amount * 100) / 100;
        }
    }

    // EUR formatter (can have different logic if needed)
    public class EURFormatter extends CurrencyFormatter {
        public override Decimal formatAmount(Decimal amount) {
            // Same as USD, but can be different without touching base code
            return Math.round(amount * 100) / 100;
        }
    }

    // JPY formatter with Yen-specific rounding
    public class JPYFormatter extends CurrencyFormatter {
        public override Decimal formatAmount(Decimal amount) {
            // Yen doesn't use decimal places in standard notation
            return Math.round(amount);
        }
    }
  ```

  Usage Example

  ```apex
    // Test class demonstrating the OCP benefit
    @isTest
    public class OCPExampleTests {

        @Test
        public void testDifferentCurrencyFormatters() {
            Decimal amount = 1234.567;

            // All formatters follow the same interface (Shape base class)
            CurrencyFormatter usdFormatter = new USDFormatter();
            CurrencyFormatter eurFormatter = new EURFormatter();
            CurrencyFormatter jpyFormatter = new JPYFormatter();

            // Each formatter can be extended without modifying existing code
            System.assertEquals(1234.57, usdFormatter.formatAmount(amount));
            System.assertEquals(1234.57, eurFormatter.formatAmount(amount));
            System.assertEquals(1235, jpyFormatter.formatAmount(amount));
        }

        @Test
        public void testAddingNewCurrencyWithoutBreakingExisting() {
            // NEW: You can add a new currency formatter WITHOUT touching existing code

            // Example of adding a GBP (British Pound) formatter
            CurrencyFormatter gbpFormatter = new GBPFormatter();

            Decimal amount = 987.654;
            System.assertEquals(987.65, gbpFormatter.formatAmount(amount));
        }

        // NEW: This class can be added without breaking any existing code
        public class GBPFormatter extends CurrencyFormatter {
            public override Decimal formatAmount(Decimal amount) {
                return Math.round(amount * 100) / 100; // British pounds also use 2 decimals
            }
        }
    }
  ```

  Benefits of this OCP Implementation in Apex

  1. Maintainability: Each currency formatter has a single responsibility (formatting logic)
  2. Extensibility: New currencies can be added by creating new classes without modifying existing code
  3. Testability: Each formatter is independently testable with its own test class
  4. Code Reusability: Formatters can be reused across different parts of your application

  Key SOLID Concepts Applied in this Apex Example:

  - Single Responsibility: Each formatter handles only one currency's formatting logic
  - Open/Closed: New currencies added as new classes, no modification to existing code needed
  - Liskov Substitution: Any CurrencyFormatter can be substituted for another
  - Interface Segregation: If you had multiple formatting methods, they'd be in focused interfaces
  - Dependency Inversion: High-level business logic depends on the abstract CurrencyFormatter base class

  This pattern is particularly useful in Salesforce development where currency handling and configuration changes are common
  requirements that may evolve over time.

  ### 3. Liskov Substitution Principle (LSP) - Apex Examples

  Definition: In Apex, subclasses must be substitutable for their base classes without breaking the application.

  ## BAD - Violates LSP

  ```apex
  public class FlyingBird {
      public String fly() {
          return "Flying high!";
      }
  }

  public class Sparrow extends FlyingBird {
      // Inherits fly() method
  }

  public class Penguin extends FlyingBird {  // Problem: penguins can't fly!
      public String fly() {
          throw new Exception("Penguins cannot fly!");
      }
  }

  // Using incorrectly - this will break with Penguins
  public String makeBirdFly(FlyingBird bird) {
      return bird.fly();
  }

  // Test cases
  Sparrow sparrow = new Sparrow();
  Penguin penguin = new Penguin();  // This bird violates LSP!

  System.debug(makeBirdFly(sparrow));  // Works: "Flying high!"
  // System.debug(makeBirdFly(penguin)); // Would throw exception - violates LSP!
  ```

  ## GOOD - Proper hierarchy with interfaces

  ```apex
  // Interface for flying capability only
  public interface ICanFly {
      String fly();
  }

  // Abstract base class for flightless birds
  public abstract class FlightlessBird {
      private String name;

      public FlightlessBird(String name) {
          this.name = name;
      }

      public String walk() {
          return name + ' is waddling!';
      }

      // Subclasses must implement their own behavior
      public abstract String getSpecies();
  }

  // Sparrow implements the flying interface - only has flight capability
  public class Sparrow implements ICanFly {
      public String fly() {
          return "Flying high!";
      }
  }

  // Penguin extends flightless bird - separate from flying hierarchy
  public class Penguin extends FlightlessBird {
      public Penguin(String name) {
          super(name);
      }

      public String speak() {
          return "Squawk!";
      }

      public String getSpecies() {
          return "King Penguin";
      }
  }
  ```


  ## Using correctly in Apex

  ```apex
  // Method that works with any bird type safely
  public String makeBirdFlyOrWalk(Object bird) {
      if (bird instanceof ICanFly) {
          return ((ICanFly) bird).fly();
      } else if (bird instanceof FlightlessBird) {
          return ((FlightlessBird) bird).walk();
      }
      throw new Exception('Unsupported bird type');
  }

  // Test cases
  Sparrow sparrow = new Sparrow();
  Penguin penguin = new Penguin('Penny');

  // These work correctly without breaking LSP
  System.debug(makeBirdFlyOrWalk(sparrow));  // Works: "Flying high!"
  System.debug(makeBirdFlyOrWalk(penguin));  // Works: "Penny is waddling!"
  ```

  Key Apex Differences:

  1. Type Checking: Use instanceof to safely cast and check types at runtime
  2. Interfaces: Define capabilities with interface keyword (e.g., ICanFly)
  3. Abstract Classes: Use abstract class for base functionality that subclasses extend
  4. Method Signatures: Apex requires explicit method signatures in interfaces
  5. Single Inheritance: Concrete classes can extend only one parent but implement multiple interfaces

  LSP Compliance Checklist:

  - Substitution: Sparrow and Penguin are now truly substitutable
  - Inheritance Hierarchy: Clear separation of concerns (flying vs. flightless)
  - Interface Segregation: Methods match actual capabilities
  - Dependency Inversion: High-level logic works with abstractions, not concrete types

  This design ensures that any object can be safely substituted for its base type without unexpected behavior or exceptions
  in the Salesforce application.

  ### 4. Interface Segregation Principle (ISP)

  Definition: Clients should not be forced to depend on interfaces they don't use.

  When to apply: When one large interface makes clients dependent on unnecessary methods.

  Example: Instead of having one big user interface:
  ## BAD - Single large interface forces all dependencies (Apex)

  ```apex
    public interface IUserRepository {
        void saveUser(User user); // Database ops only needed by some classes
        void updateUser(User user);
        void deleteUser(Id userId);
        List<User> findAdminUsers(); // Only for admin modules
        Boolean sendEmailNotification(User user, String message); // Only for user actions
    }
  ```

  ## GOOD - Multiple focused interfaces (Apex)

  ```apex
    // Repository interface with core CRUD operations
    public interface IUserRepository {
        User save(User entity);
        User update(Id id, User entity);
        void delete(Id id);
    }

    // Admin-specific interface
    public interface IAdminOperations {
        List<User> findAdmins();
        // Add other admin-specific methods here as needed
    }

    // User actions interface for non-repository operations
    public interface IUserActions {
        Boolean sendEmailNotification(User user, String
    message);
        // Add other action-specific methods here as needed
    }
  ```

  ### 5. Dependency Inversion Principle (DIP)

  Definition: High-level modules should depend on abstractions; both should depend on low-level modules.

  When to apply: When you have tight coupling between high-level business logic and low-level implementation details.

  ## BAD - High-level depends on low-level

  ```apex
  // Bad example: Hard dependency in constructor
  public with sharing class UserService {
      private DatabaseRepository repository; // Hard dependency

      public void saveUser(User user) {
          return this.repository.save(user);
      }

      // Constructor creates hard dependency
      public UserService() {
          this.repository = new DatabaseRepository();
      }
  }
  ```

  ## GOOD - Depends on abstraction

  ```apex
  // Good example: Dependency injection via interface
  public interface IUserRepository {
      SObject save(SObject entity);
  }

  public class DatabaseRepository implements IUserRepository {
      public void save(SObject entity) {
          // database logic here (e.g., DML operations)
          insert entity; // or other data manipulation logic
      }
  }

  public with sharing class UserService {
      private final IUserRepository repository; // Dependency injected

      public UserService(IUserRepository repo) {
          this.repository = repo;
      }

      public void saveUser(User user) {
          return this.repository.save(user);
      }
  }

  // Usage example:
  global set UserService service = new UserService(new DatabaseRepository());
  ```

  Key Differences for Apex:

  1. Interfaces: Use interface keyword instead of abc.ABC
  2. Abstract methods don't need @abstractmethod decorator in Apex
  3. Constructors use parameter injection rather than dependency injection via __init__
  4. Annotations: Add with sharing to controller/service classes for security
  5. Types: Use SObject as base type when appropriate for Salesforce data handling

  These examples maintain the same DIP principles while following Apex conventions and Salesforce development best
  practices.

  ### Key Benefits of SOLID Principles:

  1. Maintainability: Easier to modify and extend individual components
  2. Testability: Loosely coupled code is easier to unit test
  3. Reusability: Small, focused classes are more reusable across the codebase
  4. Flexibility: Systems can adapt to changing requirements without major refactoring
  5. Code Quality: Leads to cleaner, more understandable architecture

  ### Quick Checklist for SOLID Compliance:

  - SRP: Does this method/class do only one thing well?
  - OCP: Can new functionality be added via extension rather than modification?
  - LSP: Will replacing a subclass with its parent always work correctly?
  - ISP: Are clients forced to depend on methods they never use?
  - DIP: Do high-level modules depend on abstractions or concrete implementations?

  These principles work together to create robust, maintainable code that's easier to understand and modify over time.

## Common Commands

## Notes



